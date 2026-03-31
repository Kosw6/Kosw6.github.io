---
title: "TimescaleDB 시계열 조회 성능 28배 개선"
layout: single
permalink: /reports/timescaledb-27x/
toc: true
toc_sticky: true
classes: wide
---

> **원본 분석 노트**: [GitHub에서 보기](링크_입력)

---

## 요약

| 항목 | Before | After |
|------|--------|-------|
| P95 (300 RPS) | 7,247 ms | 235 ms |
| 개선율 | — | **약 28배** |

- **문제**: 하이퍼테이블 누락 + 인덱스 미적용 → 대용량 시계열 전테이블 스캔
- **개선**: 인덱스 최적화 → 하이퍼테이블 적용 → 시간/공간 청크 튜닝
- **환경**: 2,600만 행 OHLCV, PostgreSQL 17 + TimescaleDB, k6 부하 테스트

---

## 배경

트레이딩 플랫폼의 주 기능은 특정 종목(symb)의 90일치 OHLCV 데이터를 차트로 렌더링하는 것이다.
읽기 위주(append-only) 워크로드이며, 업데이트·삭제는 거의 없고 조회 요청이 집중된다.
목표 SLO: **p95 < 300ms**

---

## 1단계: 인덱스 최적화

### 문제 확인

초기 테스트에서 8 RPS만으로도 과부하가 발생하여 constant-arrival-rate를 5~7 RPS로 낮춰야 했다.
DB 쿼리 성능 저하 및 인덱스 미적용으로 진단하였다.

### 결과

| 항목 | RPS | P95 | Throughput | FailRate |
|------|-----|-----|------------|---------|
| 인덱스 X | 10 | 342.14 ms | 10.01 req/s | 0.00% |
| 인덱스 O `(symb, timestamp)` | 10 | 32.06 ms | 10.01 req/s | 0.00% |

→ 인덱스 적용만으로 **약 10배 개선**

---

## 2단계: 하이퍼테이블 적용

### 문제 확인

덤프/복원 과정에서 TimescaleDB 하이퍼테이블 메타데이터가 누락되어, 테이블이 일반 PostgreSQL 테이블로 운영 중이었다.

```sql
SELECT hypertable_name, num_chunks, compression_enabled
FROM timescaledb_information.hypertables;
-- 결과: 조회되지 않음 → 하이퍼테이블 미적용 확인
```

### 청크 구조 이해

TimescaleDB 하이퍼테이블은 데이터를 **시간(Time) → 공간(Space)** 순으로 분할한다.

- **시간 분할**: `chunk_time_interval` 기준으로 timestamp를 주기 단위로 나눔
- **공간 분할**: `hash(symb) % num_partitions` 로 동일 종목이 항상 같은 버킷에 저장

### 청크 설정 비교 (EXPLAIN 기반, 웜/콜드 캐시 각 5종목 테스트)

| 항목 | 평균 planningTime (콜드) | 평균 executionTime |
|------|--------------------------|-------------------|
| 30일 인터벌, 공간 없음 | 49 ms | 3 ms |
| 90일 인터벌, 공간 8 | 31 ms | 3 ms |
| 90일 인터벌, 공간 4 | 23 ms | 3 ms |

- 실행 시간은 인터벌 설정과 무관하게 동일 (0.3~3 ms)
- 조회 범위(90일)와 인터벌을 맞출수록 스캔 청크 수 감소 → cold planning time 단축
- 공간 파티션 수를 줄일수록 같은 이유로 planner 오버헤드 감소

> 웜 캐시 구간에서는 buffer hit + plan cache로 planning time이 3 ms로 수렴하여,
> 차이는 콜드 플래닝 구간에서만 유의미하다.

---

## 3단계: 하이퍼테이블 전/후 부하 테스트 (동일 인덱스 조건)

| 항목 | RPS | P95 | Throughput | FailRate |
|------|-----|-----|------------|---------|
| 일반 PostgreSQL 테이블 | 300 | 7,247.28 ms | 213.40 req/s | 0.00% |
| 하이퍼테이블 (90d, 공간 8) | 300 | 331.99 ms | 300.01 req/s | 0.00% |
| 하이퍼테이블 (90d, 공간 4) | 300 | **235.32 ms** | 300.01 req/s | 0.00% |

→ 하이퍼테이블 튜닝 후 **약 28배 개선**, 공간 파티션 4 기준 SLO 충족

---

## 최종 설정 및 선택 이유

| 항목 | 설정 |
|------|------|
| 시간 분할 | 90일 |
| 공간 분할 | 4 |

**선택 근거**:
- 단순 90일 종목 조회가 주 기능 → 시간 인터벌을 조회 범위와 일치
- append-only 구조로 VACUUM / 인덱스 락 경합 없음
- 하루 1회 배치 쓰기(≈10,000건) → 쓰기 병렬도 중요하지 않음
- 읽기 성능 최적화 우선 → num_partitions=4 (2보다 안정성, 8보다 낮은 planner 오버헤드)

---

## 추가: 1D/1W/1M/1Y 조회 전략

| 프레임 | 데이터 소스 |
|--------|------------|
| 1D | 원본 hypertable 직접 조회 |
| 1W / 1M / 1Y | TimescaleDB Continuous Aggregate (CAGG) materialized view |

- CAGG 인덱스: `(symb, bucket DESC)` → 커서 기반 페이징 최적화
- Refresh: 일봉 배치 완료 후 수동 호출 (`refresh_continuous_aggregate()`) — 최근 구간만 갱신하여 DB 부하 최소화

---

## 핵심 인사이트

> 인덱스가 쿼리 경로를 결정하고, 하이퍼테이블이 스캔 범위를 제한한다.
> 두 조건이 모두 충족되어야 대규모 시계열 조회 성능이 확보된다.
