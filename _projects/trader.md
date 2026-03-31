---
title: "Trader Platform"
layout: single
sidebar:
  nav: "main"
toc: true
toc_sticky: true
classes: wide
excerpt: "40–50M+ OHLCV · TimescaleDB 28배 개선 · WebSocket 99.97% · Kafka Failback"
tags: [spring, timescaledb, redis, kafka, websocket, k6, aws]
---

## 프로젝트 개요

40–50M+ OHLCV 시계열 데이터를 다루는 개인 퀀트 트레이딩 플랫폼.
기능 구현보다 **"얼마나 버티는가"와 "왜 느린가"** 를 중심으로 개발했습니다.

| 항목 | 내용 |
|------|------|
| **Stack** | Spring Boot · JPA · PostgreSQL / TimescaleDB · Redis · Kafka · React · AWS |
| **Scale** | ~10K 종목 × 40–50M+ OHLCV 행 |
| **k6 결과** | p90 57ms, avg 18.6ms (캐시 없이, 캐시 적용 전 원본) |
| **SSO** | Google / Kakao / Naver JWT |

---

## 성능 엔지니어링

### 1. TimescaleDB 시계열 쿼리 28배 개선

> **P95 7,247ms → 235ms @ 300 RPS**

#### 문제

- 하이퍼테이블 누락: 덤프/복원 과정에서 TimescaleDB 메타데이터 손실 → 일반 테이블로 운영
- 인덱스 미적용: `(symb, timestamp)` 복합 인덱스 없음 → 전체 테이블 스캔

#### 단계별 개선

| 단계 | 조치 | P95 변화 |
|------|------|----------|
| 1단계 | `(symb, timestamp)` 복합 인덱스 적용 | 342ms → **32ms** @ 10 RPS (10배) |
| 2단계 | 하이퍼테이블 생성 + 청크 구조 분석 | — |
| 3단계 | 90일 인터벌 + 공간 파티션 4 튜닝 | 7,247ms → **235ms** @ 300 RPS (28배) |

> 인덱스가 쿼리 경로를 결정하고, 하이퍼테이블이 스캔 범위를 제한한다.
> 두 조건이 동시에 충족되어야 대규모 시계열 조회 성능이 확보된다.

→ [상세 Report 보기](/reports/timescaledb-27x/)

---

### 2. JPA Fetch 전략 비교 실험

> **붕괴 RPS 5배 차이 확인**

Lazy N+1 / Fetch Join / Projection / DB 레벨 preview 4가지 전략을 동일 조건에서 비교.
10K payload 기준 Fetch Join vs preview 간 붕괴 RPS가 5배 차이 발생.

→ [상세 Report 보기](/reports/jpa-tuning/)

---

### 3. JFR / JMC 런타임 프로파일링

> **Old GC 36% 감소**

쿼리 튜닝 이후 수치가 개선되었음에도 GC가 예상보다 많이 발생.
JMC Stack Trace 분석으로 JWT 중복 검증이 hot path임을 확인하고 제거.

→ [상세 Report 보기](/reports/jfr-jmc-hotpath/)

---

## 실시간 시스템

### WebSocket Group Canvas

> **≤200ms 수신율 0.38% → 99.97%**

#### 문제

멀티스레드 fanout 구조에서 `TEXT_PARTIAL_WRITING` 동시성 버그 발생.
부하가 높을수록 수신 실패율이 급증하는 구조적 문제.

#### 해결

- Dirty Flag 기반 최신값 단건 전송 전략으로 재설계
- 부분 상태 누적이 아닌 항상 최신 스냅샷을 전송

→ [상세 Report 보기](/reports/websocket-group-canvas/)

---

### WebSocket 수평 확장 PoC 시리즈

단일 인스턴스 최적화 이후, 분산 환경 확장성을 3단계로 설계·검증.

| PoC | 주제 | 핵심 결과 |
|-----|------|----------|
| PoC 1 | 그룹 샤딩 & 부하 분산 | fanout 50% 균등 분배, GC 3→1회, byte[] 205MiB→93+111MiB |
| PoC 2 | Fallback & 편집 충돌 제어 | dirtyFields 기반 AUTO_MERGE / CONFLICT 자동 판별 |
| PoC 3 | Failback & Kafka Replay | 이벤트 유실 없는 무중단 서버 복구 E2E 검증 |

→ [PoC 1 보기](/reports/websocket-poc1-sharding/) &nbsp;·&nbsp; [PoC 2 보기](/reports/websocket-poc2-conflict/) &nbsp;·&nbsp; [PoC 3 보기](/reports/websocket-poc3-failback/)

---

## 아키텍처

```
[Client]
   │ HTTP / WebSocket
[Spring Boot]
   ├── TimescaleDB (Hypertable + CAGG)
   ├── Redis (Cache, Draft State)
   ├── Kafka (Event replay, Failback)
   └── Prometheus / Grafana / Slack
[AWS]
   EC2 · S3 · CloudFront · RDS
```

### 조회 전략

| 프레임 | 데이터 소스 |
|--------|------------|
| 1D | 원본 Hypertable 직접 조회 |
| 1W / 1M / 1Y | Continuous Aggregate (CAGG) materialized view |

---

## 운영 및 관측성

- **CI/CD**: GitHub Actions — 빌드 · 테스트 · 배포 자동화
- **Coverage**: JaCoCo ≥70% 기준
- **Monitoring**: Prometheus + Grafana 대시보드 + Slack 알람
- **Load Test**: k6 constant-arrival-rate 시나리오

---

## GitHub

- [trader-backend](https://github.com/Kosw6/trader-backend)
