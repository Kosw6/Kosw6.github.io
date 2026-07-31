---
title: "Trader - 주식투자 복기 서비스"
layout: single
classes: wide
excerpt: "투자 복기 서비스, TimescaleDB p95 SLO 달성, WebSocket 99.97%, Kafka/ETL lineage, AWS worker scaling"
tags: [spring, timescaledb, redis, kafka, websocket, k6, go, python, aws, etl]
---

Trader는 차트, 투자 일지와 실시간 캔버스를 연결해 투자 판단과 결과를 다시 살펴볼 수 있게 만든 주식투자 복기 서비스입니다.

투자 판단 근거는 React Flow 기반 노드와 엣지로 시각화하고, 주가와 거시경제, 재무 데이터를 함께 조회할 수 있게 했습니다. 서비스가 커지면서 발생한 조회 지연과 동시 편집 문제를 측정해 개선하고, 데이터 수집 중단 이후에도 처리 상태를 확인하고 복구할 수 있는 구조로 확장했습니다.

<nav class="page-quick-nav" aria-label="핵심 섹션 바로가기">
  <strong>빠르게 보기</strong>
  <a href="#프로젝트-개요">프로젝트 개요</a>
  <a href="#product-screen">구현 화면</a>
  <a href="#성과-한눈에-보기">주요 결과</a>
  <a href="#data-platform">데이터 파이프라인</a>
  <a href="#timescaledb-performance">성능 개선</a>
  <a href="#websocket-realtime">실시간 처리</a>
  <a href="#reliability-operations">장애 복구와 운영</a>
</nav>

---

## 프로젝트 개요

| 항목 | 내용 |
|------|------|
| **개발 기간** | 2025.01 - 현재, 개인 개발 |
| **사용자 기능** | 그룹 및 개인 캔버스, 투자 일지, 차트 마커, 재무와 거시경제 데이터 조회 |
| **Backend** | Spring Boot, JPA, PostgreSQL, TimescaleDB, Redis, Kafka |
| **Frontend** | React, React Flow, WebSocket 기반 실시간 캔버스 |
| **데이터 처리** | Go Controller, Python Worker, 원본 보존과 ETL 처리 추적 |
| **데이터 규모** | 약 1만 종목, 2,600만 행 이상의 OHLCV, KIS, BLS, SEC 데이터 |
| **검증** | k6, JFR/JMC, Grafana, AWS ASG Worker 조절 |

## 사용자 조회 화면 {#product-screen}

<figure class="report-figure">
  <a href="/assets/images/trader/trader-chart.png" target="_blank" rel="noopener">
    <img src="/assets/images/trader/trader-chart.png" alt="일봉 주가 캔들 차트와 핵심 통계, 투자 일지 마커를 제공하는 Trader 사용자 화면" loading="lazy">
  </a>
  <figcaption>90일 단위 OHLCV 조회 결과와 캔들 차트, 핵심 통계, 투자 일지 마커를 연결한 사용자 화면입니다. <a href="/assets/images/trader/trader-chart.png" target="_blank" rel="noopener">원본 크기로 보기</a></figcaption>
</figure>

---

## 성과 한눈에 보기

| 문제 | 개선 전 | 개선 후 | 결과 |
|------|--------|---------|------|
| TimescaleDB 시계열 쿼리 (p95, 300 RPS) | 7,247ms | **235ms** | **목표 300ms 충족** |
| WebSocket 200ms 이내 수신율 (쓰기 충돌 제거 후) | 0.38% | **99.97%** | **99.6%p 증가** |
| Old GC 총 시간 (90초 본부하) | 3.47초 | **2.22초** | 약 36% 감소 |
| BLS ETL 중단 복구 | Worker 중단 중 2건 대기 | **재기동 후 2건 모두 처리** | 중단 지점부터 작업을 이어서 처리 |
| AWS Python Worker 기동 검증 | 테스트 이전에는 수동 기동,종료 | **Kafka lag 감지 후 자동 확장, 유휴 시 자동 축소** | ASG desired를 0 → 1 → 0으로 자동 조절 |

---

## Data Pipeline & Control Plane 확장 {#data-platform}

차트와 일지만으로는 투자 복기에 필요한 재무·매크로·시세 데이터를 안정적으로 제공하기 어렵다고 판단했습니다. 그래서 수집, raw 보존, 정규화, 재처리, worker 제어를 별도 데이터 처리 파이프라인과 제어 구조로 확장했습니다.

<figure class="report-figure">
  <img src="/assets/images/data-platform/trader-data-architecture.svg" alt="Go Controller, Kafka, Python Collector와 ETL Worker, raw storage와 PostgreSQL로 구성한 데이터 파이프라인">
  <figcaption>Go Controller가 정책과 상태를 제어하고 Python Worker가 수집과 ETL을 실행합니다.</figcaption>
</figure>

### 데이터 운영 화면

<figure class="report-figure">
  <a href="/assets/images/trader/trader-data-inventory.png" target="_blank" rel="noopener">
    <img src="/assets/images/trader/trader-data-inventory.png" alt="BLS 데이터를 카테고리, 시리즈, 연도별로 확인하는 Data Inventory 관리자 화면" loading="lazy">
  </a>
  <figcaption>BLS 데이터를 category, series, year 단위로 내려가며 적재 범위와 마지막 수집 시점을 확인하는 Data Inventory 화면입니다. <a href="/assets/images/trader/trader-data-inventory.png" target="_blank" rel="noopener">원본 크기로 보기</a></figcaption>
</figure>

<figure class="report-figure">
  <a href="/assets/images/trader/trader-worker-control.png" target="_blank" rel="noopener">
    <img src="/assets/images/trader/trader-worker-control.png" alt="Worker 정책과 heartbeat, 작업량, AWS ASG scale command를 확인하는 Worker Control 관리자 화면" loading="lazy">
  </a>
  <figcaption>Go Controller가 worker policy, heartbeat, 작업량과 AWS ASG scale command를 조회하고 제어하는 Worker Control 화면입니다. <a href="/assets/images/trader/trader-worker-control.png" target="_blank" rel="noopener">원본 크기로 보기</a></figcaption>
</figure>

| 설계 결정 | 내용 |
|---|---|
| Go Controller | job 생성, outbox relay, Kafka lag 측정, worker scale-out/in 제어 |
| Python Worker | KIS/BLS/SEC 수집과 ETL 처리 담당 |
| raw 보존 | 외부 API 응답을 source_object와 storage_key로 추적 |
| ETL lineage | record_lineage로 raw와 정규화 레코드 연결 |
| 중복 방지 | processed_event와 idempotency_key로 Kafka 재전달 대응 |
| 비용/부하 제어 | Kafka lag와 worker heartbeat 기반 AWS ASG scale-out/in 검증 |

### 검증 결과

| 검증 항목 | 결과 |
|---|---|
| BLS job end-to-end | JOB_ITEM_QUEUED -> raw 수집 -> RAW_OBJECT_READY -> ETL 완료 |
| raw lag 처리 | trader.raw.bls.ready lag 2 -> ETL worker 재기동 후 lag 0 |
| AWS worker scale-out | job lag 1 발생 시 Python worker ASG desired 0 -> 1 |
| AWS worker scale-in | 전체 lag 0 + idle heartbeat 120초 후 desired 1 -> 0 |
| Kafka/outbox 복구 | DB outbox를 기준으로 Kafka 상태 불일치 구간 재발행 복구 |

핵심은 데이터를 단순 적재하는 것이 아니라, **raw부터 정규화 결과, Kafka commit, worker 상태까지 운영자가 추적하고 제어할 수 있는 구조**로 확장한 것입니다.

→ [Trader 데이터 파이프라인 Report 보기](/reports/trader-data-platform/)

---

## 성능 개선 1 — TimescaleDB 시계열 쿼리 p95 SLO 달성 {#timescaledb-performance}

### 상황

특정 종목의 90일치 OHLCV 데이터를 조회하는 주 기능.
목표 SLO: **P95 < 300ms @ 300 RPS.**

8 RPS 테스트에서 이미 응답이 느려지기 시작했다. 300 RPS는 커녕 버티지 못하는 수준.

### 원인 추적

```sql
-- 1) Sequential Scan 확인
EXPLAIN ANALYZE SELECT * FROM ohlcv WHERE symb = 'AAPL' ORDER BY timestamp DESC LIMIT 1000;
-- → Seq Scan, rows=26,000,000

-- 2) 하이퍼테이블 여부 확인
SELECT hypertable_name FROM timescaledb_information.hypertables;
-- → 결과 없음. 덤프/복원 과정에서 메타데이터 손실
```

두 가지 원인이 겹쳐 있었다.
1. `(symb, timestamp)` 복합 인덱스 없음 → **2,600만 행 전체 스캔**
2. 하이퍼테이블 누락 → TimescaleDB의 청크 기반 파티셔닝 미작동

### 단계별 개선 결과

| 검증 | 조건 | P95 결과 |
|------|------|----------|
| 인덱스 효과 | 일반 테이블, 인덱스 없음 | **342ms** @ 10 RPS |
| 인덱스 효과 | 일반 테이블, `(symb, timestamp)` 적용 | **32ms** @ 10 RPS (**약 10배**) |
| 하이퍼테이블 기준선 | 동일 인덱스의 일반 테이블 | **7,247ms** @ 300 RPS |
| 청크 비교 | 하이퍼테이블, 90일 인터벌 + 공간 파티션 8 | **332ms** @ 300 RPS |
| 최종 설정 | 하이퍼테이블, 90일 인터벌 + 공간 파티션 4 | **235ms** @ 300 RPS, SLO 달성 |

> **인덱스가 쿼리 경로를 결정하고, 하이퍼테이블이 스캔 범위를 제한한다.**
> 두 조건이 동시에 충족되어야 대규모 시계열 조회가 성립한다.

**추가: 조회 프레임별 전략 분리**

| 프레임 | 데이터 소스 |
|--------|------------|
| 1D | Hypertable 직접 조회 |
| 1W / 1M / 1Y | TimescaleDB CAGG 적용 가능성 검증 및 조회 전략 설계 |

→ [상세 Report 보기](/reports/timescaledb-27x/)

현재 사용자 화면에는 1D 조회를 연결했으며, 1W/1M/1Y는 CAGG의 성능과 정합성 특성을 확인한 설계·검증 범위입니다.

---

## 성능 개선 2 — JPA 조회 구조와 본문 크기 비교

### 상황

캔버스 노드 목록 API에서 어떤 조회 구조가 안정적인지, 목록에 필요하지 않은 본문 전체를 가져오는 비용이 얼마나 큰지 확인해야 했습니다.

### 실험 설계

네 가지 조회 구조를 동일한 k6 부하로 실행하고 p95, GC, 객체 할당량을 비교했습니다. Fetch Join이 부모 엔티티를 중복 생성하지 않아 Projection보다 안정적인 조건을 확인한 뒤, 같은 Fetch Join 흐름에서 본문 처리 위치를 추가로 비교했습니다.

| 비교 | 결과 |
|------|------|
| 네 가지 조회 구조 | Fetch Join이 p95와 GC 측면에서 가장 안정적 |
| 1만 자 원문 또는 앱 20자 절단 | 약 26 RPS부터 p95 붕괴 |
| DB에서 20자만 조회 | 120 RPS에서 안정적으로 처리 |
| 동일 p95 구간의 유지 가능 RPS | DB 20자 프리뷰가 **약 5배 높음** |

그래프 목록에서는 본문 전체가 필요하지 않았습니다. 따라서 목록은 DB에서 20자 프리뷰만 조회하고, 원문은 상세 조회에서 불러오도록 분리했습니다.

→ [상세 Report 보기](/reports/jpa-tuning/)

---

## 성능 개선 3 — JFR / JMC 런타임 프로파일링

### 상황

쿼리 튜닝 이후 응답 속도는 개선됐는데 **GC가 예상보다 많이 발생**했다.
어디서 오는지 알 수 없었다.

### 원인 추적

JMC Stack Trace 분석으로 hot path를 추적.

```
Allocation Hotspot:
  BaseNCodec.decode()
  → Base64.decodeBase64()
  → JWT.require().verify(token)
```

`JwtFilter`와 `JwtTokenProvider`가 각각 검증하던 구조를 한 번만 검증하고 `DecodedJWT`를 인증 단계에서 재사용하도록 바꿨습니다. 인증 경로의 불필요한 DB 조회도 제거해 **90초 본부하의 Old GC 총 시간을 약 36% 줄였습니다.**

쿼리 최적화만으로는 보이지 않는 병목이었다.
런타임 프로파일러가 없었으면 발견하지 못했을 지점.

→ [상세 Report 보기](/reports/jfr-jmc-hotpath/)

---

## 실시간 시스템 1 — WebSocket 브로드캐스트 재설계 {#websocket-realtime}

### 상황

Group Canvas의 실시간 노드 업데이트 기능.
로컬에서는 잘 됐지만, 동시 접속 200명 부하에서 같은 세션에 여러 작업이 전송하며 쓰기 충돌과 세션 이탈이 발생했습니다.

세션별 전송을 직렬화해 오류를 제거한 뒤에도 송신자 20명, 20Hz 조건에서 ≤200ms 수신율은 **0.38%**에 머물렀습니다.

### 원인

멀티스레드 fanout 구조의 동시 `sendMessage()` 호출은 `TEXT_PARTIAL_WRITING`을 일으켰습니다.
이를 직렬화한 뒤에는 변경 여부와 관계없이 반복되는 flush가 큐를 쌓아 지연을 만들었습니다.

### 해결

`ConcurrentWebSocketSessionDecorator`로 세션별 쓰기를 직렬화했습니다.
이후 **Dirty Flag 기반 최신값 단건 전송**으로 재설계해, 변경이 있을 때만 최신 스냅샷을 전송했습니다.

- ≤200ms 수신율: 0.38% → **99.97%**

→ [상세 Report 보기](/reports/websocket-group-canvas/)

---

## 실시간 시스템 2 — Redis Pub/Sub 누락 보정

### 문제

Redis Pub/Sub은 빠르게 이벤트를 전파하지만, 특정 서버의 구독이 잠시 끊기면 그동안의 이벤트를 다시 받을 수 없다. 캔버스 잠금과 노드 변경처럼 놓치면 사용자별 화면 상태가 달라지는 이벤트에는 별도의 복구 경로가 필요했다.

### 해결

- Volatile 이벤트와 Reliable 이벤트의 즉시 전파는 Redis Pub/Sub으로 유지
- Reliable 이벤트만 Kafka에도 발행해 누락 보정과 재기동 후 catch-up에 사용
- `serverId + eventId` Redis key로 서버별 전파 여부 기록
- 서버별 Kafka Consumer Group이 전체 이벤트를 소비한 뒤, 자기 서버에서 누락된 이벤트만 WebSocket으로 재전파

Redis는 정상 상태의 빠른 전파를 담당하고 Kafka는 Reliable 이벤트의 복구 경로를 담당하도록 역할을 분리했다.

→ [Redis Pub/Sub 누락과 Kafka 보정 구조 보기](/reports/realtime-degrade-overview/#reliable-event-recovery)

---

## 실시간 시스템 3 — 수평 확장 PoC 시리즈

단일 인스턴스 최적화 이후, **"인스턴스가 2개 이상이면 어떻게 되는가"** 를 3단계로 설계, 검증

### PoC 1 — 그룹 샤딩

**문제**: 단순 연결 분산만으로는 같은 그룹의 fanout 비용이 여러 서버에 남아, 서버별 처리 책임을 나누기 어려웠습니다.
**해결**: groupId % shard 수 기반 라우팅으로 인스턴스별 담당 그룹 분리.

| 항목 | Before (단일) | After (샤딩 2대) |
|------|--------------|------------------|
| totalSendAttempts | 159,317 | **79,776 + 79,759** (50% 균등) |
| GC 횟수 / 인스턴스 | 3회 | **1회** |
| byte[] Allocation | 205MiB | **93 + 111MiB** |

→ [PoC 1 보기](/reports/websocket-poc1-sharding/)

### PoC 2 — Fallback & 편집 충돌 제어

**문제**: 우회 서버는 기존 서버의 메모리 편집 상태를 공유하지 않아 사용자 수정과 서버 변경을 비교하기 어려웠습니다.
**해결**: Redis Draft에 편집 상태 보존 + dirtyFields 기반 충돌 감지.

- 내가 편집 중인 사이 서버에서 변경된 필드를 추적
- `dirtyFields ∩ serverChangedFields ≠ ∅` → **CONFLICT**, 아니면 **AUTO_MERGE**
- SAFE / AUTO_MERGE / CONFLICT 3 케이스 E2E 전체 검증

→ [PoC 2 보기](/reports/websocket-poc2-conflict/)

### PoC 3 — Failback & Kafka Replay

**문제**: 복구 서버가 장애 구간 이벤트를 따라잡기 전에 다시 연결을 받으면 서버별 상태가 달라질 수 있었습니다.
**해결**: Kafka Consumer Group 분리 (Broadcast / Catch-up) + offset replay.

1. 목표 offset 기록 → Catch-up Consumer가 replay
2. `catchupCompleted = true` 확인 후 Broadcast Group으로 전환
3. 구 서버 Drain → 클라이언트 재연결 유도 → 세션 전환

검증 시나리오에서 Kafka offset 3건을 목표 offset까지 처리한 뒤 ready 전환을 확인했고, 기존 서버는 재연결 안내 후 drain했습니다.

→ [PoC 3 보기](/reports/websocket-poc3-failback/)

---

## 아키텍처

```
[Client]
   │ HTTPS / WebSocket
[Spring Boot]
   ├── TimescaleDB  → Hypertable
   ├── Redis        → Cache, Draft State (편집 상태 보존)
   ├── Kafka        → Event replay, Failback Consumer Group
   └── Prometheus / Grafana
```

---

## 운영

| 항목 | 내용 |
|------|------|
| **Monitoring** | Prometheus + Grafana |
| **Load Test** | k6 constant-arrival-rate 시나리오 |

---
## AWS 배포 환경 구성과 SLO 기반 사양 검증

### 보고서 링크

>→ [AWS 인프라 구축 및 SLO기반 성능 검증과 다운사이징 문서 보기](https://github.com/Kosw6/engineering-notes/blob/main/reports/SLO%20%EA%B8%B0%EB%B0%98%20%EC%9A%B4%EC%98%81%20%EC%82%AC%EC%96%91%20%EC%82%B0%EC%A0%95%20%EC%8B%A4%ED%97%98.md)

### 상황

성능 개선 이후 AWS에 App과 DB를 분리한 검증 환경을 구성해 목표 부하에서 필요한 사양을 확인했습니다.
단순히 빠른 것이 아니라, 현재 워크로드에서 SLO를 만족하면서 자원을 어디까지 줄일 수 있는지 확인하는 단계였습니다.

### 접근방식

* SLO 정의: 목표 RPS 환경에서 p95 ≤ 300ms
* AWS 환경 구성 (App: Public / DB: Private)
* SSM + VPC Endpoint 기반 인터넷 없는 내부 통신 구조
* k6 기반 부하 테스트

### 결과

| 구성                        | Node p95 | Stock p95 |
| ------------------------- | -------- | --------- |
| 2core / 4GB + 2core / 8GB | 10.54ms  | 10.85ms   |
| 2core / 4GB + 2core / 4GB | 23.35ms  | 11.83ms   |

- 해당 검증 부하에서 App 2core/4GB, DB 2core/4GB 구성도 p95 300ms 목표 충족

### 핵심

기존 개발 단일 서버(APP+DB+etc)와 다른 APP,DB서버 분리를 통한 성능 검증을 통해<br>
성능은 단순 인프라 사양이 아니라, 구조와 자원 사용 방식에 의해 결정된다는 것을 검증하였다.<br>

---
## Reliability & Operations {#reliability-operations}

Trader는 단순 기능 구현 이후 Redis/Kafka 장애, 운영 관측, 자동 복구, 데이터 파이프라인 제어 흐름까지 검증했습니다.

| 영역 | 검증 내용 | Report |
|------|----------|--------|
| Observability | TraceId/MDC/AOP 기반 구조화 로그, Loki/Grafana dashboard, error rate alert | [Observability System](/reports/observability-system/) |
| Redis Degrade | lock/autosave/version hint DB fallback, API 5xx 없이 기능 지속 | [Realtime Degraded Mode](/reports/realtime-degrade-overview/) |
| Kafka Degrade | Outbox 기반 durable log, 5분 38초 장애 검증에서 손실 0건/중복 0건 | [Realtime Degraded Mode](/reports/realtime-degrade-overview/) |
| Auto Recovery | Grafana Alert -> Lambda -> SSM/ASG 복구 및 scale-out | [Auto Recovery & Scale-out](/reports/auto-recovery-scaleout/) |
| Load Validation | Terraform, k6, AWS SSM 기반 Redis 장애 주입과 baseline 비교 | [Load Test Orchestrator Validation](/reports/loadtest-orchestrator-redis-fault-validation/) |
| Data Pipeline | raw 보존, Kafka outbox, ETL lineage, worker ASG scale-out/in | [Trader 데이터 파이프라인](/reports/trader-data-platform/) |

핵심은 장애를 숨기는 것이 아니라, 장애 범위를 제한하고 복구 흐름을 검증 가능한 형태로 만드는 것입니다. 최근 확장에서는 ETL 처리 비용과 부하를 즉시 발생시키지 않고, raw와 Kafka lag를 기준으로 필요한 시점에 worker를 기동하는 control plane 구조까지 검증했습니다.

---

## Source Code {#source-code}

| Repository | Responsibility |
|---|---|
| [trader-backend](https://github.com/Kosw6/trader-backend) | Spring Boot REST API, WebSocket, 실시간 협업 및 도메인 서비스 |
| [trader-controller](https://github.com/Kosw6/trader-controller) | Go Control Plane, Kafka Outbox, lag 측정, Worker 및 AWS ASG 제어 |
| [trader-data](https://github.com/Kosw6/trader-data) | Python 기반 KIS, BLS, SEC 수집, raw 보존, ETL 정규화 및 lineage 처리 |
