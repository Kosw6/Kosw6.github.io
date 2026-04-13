---
title: "Trader Platform"
layout: single
sidebar:
  nav: "main"
toc: true
toc_sticky: true
classes: wide
excerpt: "20–30M+ OHLCV, TimescaleDB 28배 개선, WebSocket 99.97%, Kafka Failback"
tags: [spring, timescaledb, redis, kafka, websocket, k6]
---

> **성능 수치를 가설로 세우고, k6·JFR·JMC로 측정하고, 구조로 개선했습니다.**
> 이 페이지는 "무엇을 만들었냐"가 아니라 "어떻게 개선했냐"의 기록입니다.

---

## 성과 한눈에 보기

| 문제 | Before | After | 개선 |
|------|--------|-------|------|
| TimescaleDB 시계열 쿼리 (P95 @ 300 RPS) | 7,247ms | **235ms** | **28배** |
| 인덱스 단독 효과 (P95 @ 10 RPS) | 342ms | **32ms** | **10배** |
| WebSocket ≤200ms 수신율 | 0.38% | **99.97%** | **+99.6%p** |
| Old GC 횟수 (JFR 실측) | 기준치 | **36% 감소** | — |
| WebSocket fanout 부하 (샤딩 후) | 159K 단일 | **79K + 79K** | **50% 분산** |

---

## 프로젝트 개요

20–30M+ OHLCV 시계열 데이터를 다루는 개인 퀀트 트레이딩 플랫폼.
**"기능이 동작하는가"보다 "얼마나 버티는가"와 "왜 느린가"를 먼저 물었습니다.**

| 항목 | 내용 |
|------|------|
| **Stack** | Spring Boot,  JPA, PostgreSQL / TimescaleDB, Redis, Kafka, React|
| **Scale** | ~10K 종목 × 20–30M+ OHLCV 행 |
| **Load Test** | k6 constant-arrival-rate, p90 57ms, avg 18.6ms |

---

## 성능 개선 1 — TimescaleDB 시계열 쿼리 28배

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
| 3단계 | 90일 인터벌 + 공간 파티션 4 튜닝 | **235ms** @ 300 RPS (**28배**, SLO 달성) |

> **인덱스가 쿼리 경로를 결정하고, 하이퍼테이블이 스캔 범위를 제한한다.**
> 두 조건이 동시에 충족되어야 대규모 시계열 조회가 성립한다.

**추가: 조회 프레임별 전략 분리**

| 프레임 | 데이터 소스 |
|--------|------------|
| 1D | Hypertable 직접 조회 |
| 1W / 1M / 1Y | TimescaleDB CAGG (materialized view) |

→ [상세 Report 보기](/reports/timescaledb-27x/)

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

**필터 순서 재조정으로 중복 검증 제거 → Old GC 36% 감소.**

쿼리 최적화만으로는 보이지 않는 병목이었다.
런타임 프로파일러가 없었으면 발견하지 못했을 지점.

→ [상세 Report 보기](/reports/jfr-jmc-hotpath/)

---

## 실시간 시스템 1 — WebSocket 브로드캐스트 재설계

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

## 실시간 시스템 2 — 수평 확장 PoC 시리즈

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
## GitHub

- [trader-backend](https://github.com/Kosw6/trader-backend)
