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
| WebSocket 200ms 이내 수신율 | 0.38% | **99.97%** | **99.6%p 증가** |
| Old GC 총 시간 (90초 본부하) | 3.47초 | **2.22초** | 약 36% 감소 |
| BLS ETL 중단 복구 | Worker 중단 중 2건 대기 | **재기동 후 2건 모두 처리** | 중단 지점부터 작업을 이어서 처리 |
| AWS Python Worker 운영 | 작업 발생 시 수동 기동,종료 | **Kafka lag 감지 후 자동 확장, 유휴 시 자동 축소** | 수동 운영을 0 → 1 → 0 자동 조절로 전환 |

---

## Data Pipeline & Control Plane 확장 {#data-platform}

차트와 일지만으로는 투자 복기에 필요한 재무·매크로·시세 데이터를 안정적으로 제공하기 어렵다고 판단했습니다. 그래서 수집, raw 보존, 정규화, 재처리, worker 제어를 별도 데이터 플랫폼 구조로 확장했습니다.

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

→ [Trader Data Platform Report 보기](/reports/trader-data-platform/)

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

| 단계 | 조치 | P95 변화 |
|------|------|----------|
| Before | 인덱스 없음, 일반 테이블 | **342ms** @ 10 RPS |
| 1단계 | `(symb, timestamp)` 복합 인덱스 적용 | **32ms** @ 10 RPS (**10배**) |
| 2단계 | 하이퍼테이블 생성 + 청크 구조 분석 | 7,247ms @ 300 RPS (인덱스 있어도 대용량에서 한계) |
| 3단계 | 90일 인터벌 + 공간 파티션 4 튜닝 | **235ms** @ 300 RPS, SLO 달성 |

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

## 성능 개선 2 — JPA Fetch 전략 4차 비교 실험

### 상황

캔버스 노드 조회 API. JPA 전략에 따라 성능 차이가 얼마나 나는지 모르는 상태.
**"Fetch Join이 당연히 빠르다"는 추측을 검증하려 했는데, 결과가 달랐다.**

### 실험 설계

Lazy N+1 / Fetch Join / Projection / DB 레벨 preview 4가지를
동일한 k6 부하 환경에서 4차 비교 실험.

| 항목 | Fetch Join | DB preview |
|------|------------|------------|
| 10K payload 붕괴 RPS | 기준 | **5배 높음** |
| GC Pause | 측정값 있음 | 더 낮음 |

Projection이 Fetch Join보다 느린 케이스도 확인.
단순 "전략 선택"이 아니라 payload 크기와 GC 영향까지 같이 봐야 한다는 결론.

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
  org.springframework.security.oauth2.jwt.JwtDecoder.decode()
  → 요청마다 JWT를 중복 검증 중
  → SecurityContext에 이미 파싱된 값이 있는데도 재파싱
```

**필터 순서 재조정으로 중복 검증 제거 → Old GC 총 시간 약 36% 감소.**

쿼리 최적화만으로는 보이지 않는 병목이었다.
런타임 프로파일러가 없었으면 발견하지 못했을 지점.

→ [상세 Report 보기](/reports/jfr-jmc-hotpath/)

---

## 실시간 시스템 1 — WebSocket 브로드캐스트 재설계 {#websocket-realtime}

### 상황

Group Canvas의 실시간 노드 업데이트 기능.
로컬에서는 잘 됐다. **부하를 올리자 수신 실패율이 폭증했다.**

- 100명 부하 기준 ≤200ms 성공률: **0.38%**

### 원인

멀티스레드 fanout 구조에서 `TEXT_PARTIAL_WRITING` 에러 발생.
상태 누적 방식 전송이 동시 접근 시 충돌.

### 해결

**Dirty Flag 기반 최신값 단건 전송**으로 재설계.
부분 상태를 누적하지 않고 변경이 감지된 시점에 항상 최신 스냅샷을 전송.

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

**문제**: 단일 인스턴스에 모든 fanout 집중 → 스케일아웃이 불가능한 구조.
**해결**: groupId % shard 수 기반 라우팅으로 인스턴스별 담당 그룹 분리.

| 항목 | Before (단일) | After (샤딩 2대) |
|------|--------------|------------------|
| totalSendAttempts | 159,317 | **79,776 + 79,759** (50% 균등) |
| GC 횟수 / 인스턴스 | 3회 | **1회** |
| byte[] Allocation | 205MiB | **93 + 111MiB** |

→ [PoC 1 보기](/reports/websocket-poc1-sharding/)

### PoC 2 — Fallback & 편집 충돌 제어

**문제**: shard 장애 시 다른 인스턴스로 우회되면 편집 중이던 상태가 사라짐.
**해결**: Redis Draft에 편집 상태 보존 + dirtyFields 기반 충돌 감지.

- 내가 편집 중인 사이 서버에서 변경된 필드를 추적
- `dirtyFields ∩ serverChangedFields ≠ ∅` → **CONFLICT**, 아니면 **AUTO_MERGE**
- SAFE / AUTO_MERGE / CONFLICT 3 케이스 E2E 전체 검증

→ [PoC 2 보기](/reports/websocket-poc2-conflict/)

### PoC 3 — Failback & Kafka Replay

**문제**: 장애 서버 복구 후 재진입 시 그동안 발생한 이벤트가 유실됨.
**해결**: Kafka Consumer Group 분리 (Broadcast / Catch-up) + offset replay.

1. 목표 offset 기록 → Catch-up Consumer가 replay
2. `catchupCompleted = true` 확인 후 Broadcast Group으로 전환
3. 구 서버 Drain → 클라이언트 재연결 유도 → 세션 전환

이벤트 유실 없이 서비스 중단 없이 복구.

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
## 운영환경 인프라 구축 및 성능 검증 & 다운사이징

### 보고서 링크

>→ [AWS 인프라 구축 및 SLO기반 성능 검증과 다운사이징 문서 보기](https://github.com/Kosw6/engineering-notes/blob/main/reports/SLO%20%EA%B8%B0%EB%B0%98%20%EC%9A%B4%EC%98%81%20%EC%82%AC%EC%96%91%20%EC%82%B0%EC%A0%95%20%EC%8B%A4%ED%97%98.md)

### 상황

성능 개선 이후, 실제 운영 환경에서도 안정적으로 동작하는지 검증이 필요했다.
단순히 빠른 것이 아니라, 얼마나 버틸 수 있고, 어디까지 줄일 수 있는지를 확인하는 단계였다.

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

- 저사양에서도 SLO 안정적으로 만족

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
| Kafka Degrade | Outbox 기반 durable log, 장애 기간 이벤트 손실 0건/중복 0건 | [Realtime Degraded Mode](/reports/realtime-degrade-overview/) |
| Auto Recovery | Grafana Alert -> Lambda -> SSM/ASG 복구 및 scale-out | [Auto Recovery & Scale-out](/reports/auto-recovery-scaleout/) |
| Load Validation | Terraform, k6, AWS SSM 기반 Redis 장애 주입과 baseline 비교 | [Load Test Orchestrator Validation](/reports/loadtest-orchestrator-redis-fault-validation/) |
| Data Pipeline | raw 보존, Kafka outbox, ETL lineage, worker ASG scale-out/in | [Trader Data Platform](/reports/trader-data-platform/) |

핵심은 장애를 숨기는 것이 아니라, 장애 범위를 제한하고 복구 흐름을 검증 가능한 형태로 만드는 것입니다. 최근 확장에서는 ETL 처리 비용과 부하를 즉시 발생시키지 않고, raw와 Kafka lag를 기준으로 필요한 시점에 worker를 기동하는 control plane 구조까지 검증했습니다.

---

## Source Code {#source-code}

| Repository | Responsibility |
|---|---|
| [trader-backend](https://github.com/Kosw6/trader-backend) | Spring Boot REST API, WebSocket, 실시간 협업 및 도메인 서비스 |
| [trader-controller](https://github.com/Kosw6/trader-controller) | Go Control Plane, Kafka Outbox, lag 측정, Worker 및 AWS ASG 제어 |
| [trader-data](https://github.com/Kosw6/trader-data) | Python 기반 KIS, BLS, SEC 수집, raw 보존, ETL 정규화 및 lineage 처리 |
