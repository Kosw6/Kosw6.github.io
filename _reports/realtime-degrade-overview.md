---
title: "Realtime Degraded Mode - Redis, Kafka, Pub/Sub 장애 대응"
layout: single
permalink: /reports/realtime-degrade-overview/
classes: wide
excerpt: "Redis, Kafka, Pub/Sub 장애 상황에서 WebSocket 실시간 흐름과 durable event log를 분리해 장애 범위를 제한한 검증"
tags: [redis, kafka, websocket, degrade, fallback, spring]
---

> **핵심 질문**: Redis 또는 Kafka가 장애 상태가 되어도 실시간 협업 흐름과 이벤트 보존을 분리해 장애 범위를 제한할 수 있는가?
>
> **근거 문서**
> <br>[전체 장애 대응 구조](https://github.com/Kosw6/engineering-notes/blob/main/reports/degrade-overview.md)
> · [Redis 상태 저장소 fallback](https://github.com/Kosw6/engineering-notes/blob/main/reports/redis-degrade.md)
> · [gRPC / HTTP / Redis relay 비교](https://github.com/Kosw6/engineering-notes/blob/main/reports/pubsub-degrade.md)
> <br>[Kafka Outbox 복구](https://github.com/Kosw6/engineering-notes/blob/main/reports/kafka-degrade.md)
> · [Kafka 도입 근거](https://github.com/Kosw6/engineering-notes/blob/main/reports/kafka-necessity.md)
> · [장애 경로별 UX 비교](https://github.com/Kosw6/engineering-notes/blob/main/reports/kafka-ux-tradeoff.md)
<br>

<nav class="page-quick-nav" aria-label="핵심 섹션 바로가기">
  <strong>빠르게 보기</strong>
  <a href="#summary">결과 요약</a>
  <a href="#reliable-event-recovery">누락 보정</a>
  <a href="#redis-장애-대응">Redis fallback</a>
  <a href="#kafka-장애-대응">Kafka Outbox</a>
  <a href="#pubsub-장애-대응">Relay 비교</a>
</nav>

<div class="proof-strip">
  <div class="proof-item">
    <span class="proof-item__label">REDIS FALLBACK</span>
    <strong>50 VU -> 성공 1 / 거부 49</strong>
    <span>DB UNIQUE 경합 보장, HTTP 5xx 0</span>
  </div>
  <div class="proof-item">
    <span class="proof-item__label">RELAY P95</span>
    <strong>gRPC 14.5ms / Redis 22.7ms</strong>
    <span>HTTP 비교군 59.2ms</span>
  </div>
  <div class="proof-item">
    <span class="proof-item__label">KAFKA RECOVERY</span>
    <strong>loss 0 / duplicate 0</strong>
    <span>5분 38초 장애 검증 후 Outbox 전량 SENT</span>
  </div>
</div>

---

## Summary

| 장애 | 대응 전략 | 검증 결과 |
|---|---|---|
| Redis state/cache 장애 | DB fallback | lock/autosave/version hint 경로에서 5xx 없이 기능 지속 |
| Redis Pub/Sub 장애 | gRPC relay fallback | volatile event 전달 경로 유지 |
| Kafka 장애 | Outbox DB 저장 후 recovery replay | 5분 38초 장애 검증에서 손실 0건, 중복 0건, PENDING/FAILED 0건 |
| Redis latency 500ms | timeout 기준 DB fallback | Redis 완전 다운이 아니어도 degrade 전환 |

---

## 설계 원칙

Realtime 시스템에서는 모든 이벤트를 같은 경로로 처리하면 장애가 크게 전파된다.

따라서 Trader에서는 경로를 분리했다.

```text
Hot path
  - WebSocket 즉시 반응
  - Redis Pub/Sub fanout
  - 장애 시 gRPC fallback

Durable path
  - Kafka event log
  - Kafka 장애 시 Outbox DB 저장
  - 복구 후 replay

State fallback
  - Redis cache/lock/session
  - 장애 시 PostgreSQL fallback
```

핵심은 Kafka를 WebSocket hot path로 보지 않는 것이다. Kafka는 즉시 반응 경로가 아니라 **복구와 감사 가능한 durable event log**로 사용했다.

Kafka를 왜 기본 실시간 전파 경로가 아니라 recovery path로 두었는지는 [Kafka replay 설계 근거](https://github.com/Kosw6/engineering-notes/blob/main/reports/kafka-necessity.md)에 따로 정리했다. Redis Pub/Sub은 정상 상태의 낮은 latency hot path로 유지하고, Kafka는 Pub/Sub miss와 app 재기동 이후의 catch-up을 보정하는 역할로 분리했다.

---

## Reliable 이벤트 누락 보정 {#reliable-event-recovery}

### 해결하려던 문제

Redis Pub/Sub은 낮은 지연으로 여러 서버에 이벤트를 전파하기 좋지만, 구독이 잠시 끊긴 동안의 이벤트를 보관하지 않는다.

캔버스의 잠금과 노드 변경처럼 놓치면 다른 사용자의 화면 상태가 달라지는 Reliable 이벤트는 빠르게 전달하는 것뿐 아니라, 특정 서버의 누락을 나중에 확인하고 보정할 수 있어야 했다.

### 선택한 구조

Volatile 이벤트와 Reliable 이벤트의 정상 전파는 모두 Redis Pub/Sub을 사용했다. Reliable 이벤트만 Kafka에도 비동기로 발행해 복구 가능한 이벤트 로그를 남겼다.

```text
Volatile 이벤트
  -> Redis Pub/Sub
  -> 각 서버 WebSocket 전파

Reliable 이벤트
  +-> Redis Pub/Sub
  |    -> 즉시 WebSocket 전파
  |    -> 서버별 처리 키 기록
  |
  +-> Kafka
       -> 각 서버가 전체 이벤트를 독립 소비
       -> 처리 키가 있으면 이미 전파된 이벤트이므로 skip
       -> 처리 키가 없으면 Pub/Sub 누락으로 판단
       -> WebSocket 재전파 후 처리 키 기록
```

Kafka를 모든 WebSocket 이벤트의 기본 전파 경로로 사용하지 않은 이유는 정상 상태의 반응 속도와 장애 복구 책임을 분리하기 위해서였다.

### 서버별 처리 여부 확인

처리 여부는 전역 값이 아니라 서버별 Redis key로 기록했다.

```text
processed:reliable:{serverId}:{eventId}  TTL 5분
```

app-1이 이벤트를 정상 전파했더라도 app-2가 Pub/Sub 이벤트를 놓쳤다면 app-2의 처리 key만 존재하지 않는다. app-2의 Kafka Consumer는 이를 누락으로 판단해 자신의 WebSocket 세션에만 이벤트를 다시 전파한다.

모든 수신 경로는 같은 inbound handler를 거치게 해 Redis Pub/Sub, gRPC, HTTP, Kafka 중 어느 경로로 도착해도 동일한 중복 방지 규칙을 적용했다.

### 서버별 Consumer Group

WebSocket 누락 보정용 Consumer는 서버별로 고유한 groupId를 사용했다.

```text
reliable-replay-app-1 -> app-1이 전체 Reliable 이벤트 소비
reliable-replay-app-2 -> app-2가 전체 Reliable 이벤트 소비
```

공통 groupId를 사용하면 Kafka가 이벤트를 서버 사이에 분배하므로, 이벤트를 받지 못한 서버가 자신의 누락을 확인할 수 없다. 각 서버가 전체 이벤트를 독립적으로 소비한 뒤 `serverId + eventId` 처리 key로 자기 서버의 누락만 골라냈다.

### 검증 결과

| 검증 상황 | 확인한 동작 | 결과 |
|---|---|---|
| app-2의 Redis Pub/Sub listener 중단 | app-2 처리 key가 없는 이벤트를 Kafka Consumer가 감지 | 잠금 상태를 app-2 WebSocket에 재전파 |
| Redis Pub/Sub 정상 전파 | Kafka Consumer가 처리 key를 확인 | 중복 WebSocket 전파 없이 skip |
| app-1 중단 후 재기동 | 고정된 서버별 groupId의 이전 offset부터 catch-up | 중단 구간 Reliable 이벤트 순차 복구 |

Kafka replay는 Redis나 DB의 현재 상태를 대체하지 않는다. Pub/Sub 누락 이후 서버별 화면 상태가 다시 수렴하는 시간을 줄이고, 재기동한 서버가 중단 구간의 이벤트를 따라잡게 하는 복구 경로로 사용했다.

Redis 자체가 중단되어 처리 key를 확인할 수 없는 경우에는 재전파를 허용해 at-least-once로 동작하고, 클라이언트의 eventId 기준 중복 제거로 중복 렌더링을 막았다.

> 실제 잠금 상태 복구 화면과 Consumer 로그는 [Redis Pub/Sub 기반 실시간 전파의 한계와 Reliable 이벤트 복구 설계](https://github.com/Kosw6/engineering-notes/blob/main/reports/kafka-necessity.md)에 정리했다.

---

## Redis 장애 대응

Redis는 lock, autosave, version hint에 사용된다.

장애 시에는 도메인별로 fallback 경로를 분리했다.

| 도메인 | Redis 정상 경로 | 장애 시 fallback |
|---|---|---|
| Lock | Redis Lua / TTL key | PostgreSQL `canvas_lock` UNIQUE |
| Autosave | Redis TTL draft | PostgreSQL `edit_session` upsert |
| Version Hint | Redis MGET | PostgreSQL `node_history` query |

검증 과정에서 Redis 정상 경로의 비효율도 발견했다.

```text
Version Hint: N GET -> MGET
p95 약 44% 개선
```

---

## Kafka 장애 대응

Kafka 장애 시 실시간 WebSocket 반응이 멈추지 않도록 Kafka를 hot path에서 분리했다.

```text
Client event
  -> Redis/WebSocket immediate path
  -> Kafka durable path
      -> Kafka down
      -> Outbox DB PENDING
      -> Kafka recovery
      -> replay
      -> SENT
```

검증 결과:

| 항목 | 결과 |
|---|---|
| Kafka 장애 시간 | 약 5분 38초 |
| 장애 중 이벤트 처리 | Outbox DB PENDING 저장 |
| 복구 후 replay | 전량 SENT |
| 최종 PENDING | 0 |
| 최종 FAILED | 0 |
| 이벤트 손실 | 0 |
| 이벤트 중복 | 0 |

---

## Pub/Sub 장애 대응

Redis Pub/Sub은 기본 volatile relay 경로로 사용했다.

노드 수가 늘어날 때 Redis Pub/Sub은 broker 기반 fanout 구조라 운영이 단순하다. 다만 Redis 장애 시에는 volatile event가 끊길 수 있으므로 gRPC fallback을 검증했다.

| 경로 | p95 max | 누락·drop 관측 | 결정 |
|---|---:|---:|---|
| gRPC relay | 14.5ms | 약 0.5 ops/s | Redis 장애 시 fallback |
| Redis Pub/Sub | 22.7ms | 약 0 ops/s | 기본 broker fanout |
| HTTP relay | 59.2ms | 약 6 ops/s | 비교군, 운영 경로에서 제외 |

동일한 약 500 ops/s 조건에서 gRPC는 persistent connection과 Protobuf를 사용해 가장 낮은 p95를 보였다. Redis는 broker hop 비용이 있지만 노드가 늘어날 때 fanout 구조가 단순하므로 기본 경로로 유지했다. HTTP는 요청별 connection과 JSON 처리 비용으로 latency와 누락이 모두 불리했다.

인프라 지표에서도 app-1 기준 context switching이 gRPC 약 30,000, Redis 약 45,000, HTTP 약 55,000 순서로 관찰되어 latency 순서와 일치했다. 따라서 단일 수치가 가장 낮은 경로만 고르지 않고, 정상 상태의 확장성과 장애 상태의 직접 relay 책임을 분리했다.

---

## 개선 과정에서 배운 점

장애 중에도 정상 상태와 완전히 같은 동작을 유지하려 하면 모든 경로의 책임이 복잡해졌다. 대신 즉시 전달해야 하는 상태와 반드시 보존해야 하는 이벤트를 나누고, 장애 중 무엇을 포기하고 무엇을 지킬지 먼저 정하는 편이 명확했다.

이후 외부 의존성 장애를 검증할 때는 연결 유지 여부만 보지 않고 사용자 영향, 저장된 이벤트 수, 복구 후 replay 결과까지 확인했다. Kafka 복구 후 outbox의 `PENDING`과 `FAILED`가 모두 0이 된 시점을 복구 완료 기준으로 삼았다.
