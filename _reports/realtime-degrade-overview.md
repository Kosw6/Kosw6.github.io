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
    <span>5분 38초 장애 후 Outbox 전량 SENT</span>
  </div>
</div>

---

## Summary

| 장애 | 대응 전략 | 검증 결과 |
|---|---|---|
| Redis state/cache 장애 | DB fallback | lock/autosave/version hint 경로에서 5xx 없이 기능 지속 |
| Redis Pub/Sub 장애 | gRPC relay fallback | volatile event 전달 경로 유지 |
| Kafka 장애 | Outbox DB 저장 후 recovery replay | 이벤트 손실 0건, 중복 0건, PENDING/FAILED 0건 |
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

## 결론

이 검증의 목적은 "장애가 나도 모든 것이 완벽히 동일하게 동작한다"가 아니었다.

목표는 다음과 같았다.

- 장애 범위를 제한한다.
- 5xx와 사용자 영향 범위를 줄인다.
- durable event는 손실하지 않는다.
- 복구 후 정상 경로로 돌아온다.
- Redis/Kafka의 역할을 hot path와 durable path로 분리한다.

즉, 이 문서는 기능 구현 이후 운영 환경에서 필요한 **degraded mode 설계와 검증**을 다룬다.
