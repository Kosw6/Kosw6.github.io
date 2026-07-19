---
layout: single
title: "Resume"
permalink: /resume/
classes: wide
toc: true
toc_sticky: true
---

## Profile

**Sungwon Kim — Backend Platform Engineer**

성능을 측정하고 병목을 추적하는 데서 시작해, 실시간 이벤트와 데이터 파이프라인을 장애 이후에도 복구 가능한 운영 구조로 확장하는 엔지니어입니다. k6 부하 테스트, JFR/JMC 런타임 분석, Kafka 기반 이벤트 처리, Go Controller와 Python Worker 제어를 실측과 상태 기록으로 검증합니다.

- **GitHub**: [github.com/Kosw6](https://github.com/Kosw6)
- **Portfolio**: [kosw6.github.io](https://kosw6.github.io)

---

## 기술 스택

| 분류 | 기술 |
|------|------|
| **Backend** | Spring Boot, Spring MVC, JPA/Hibernate, Spring Security |
| **Language** | Java, **Go** (net/http, embed, goroutine, io.Writer), Python |
| **Database** | PostgreSQL, TimescaleDB (하이퍼테이블 · 청크 튜닝 · CAGG), 인덱싱 전략 |
| **Cache / MQ** | Redis (TTL 설계, JSON Serializer), Kafka (Consumer Group, offset replay) |
| **Realtime** | WebSocket (RAW), 샤딩 · Fallback · Failback 설계 |
| **Data Platform** | DB Outbox, Kafka lag, raw/ETL 분리, lineage, idempotency |
| **Performance** | k6, JFR/JMC, Prometheus/Grafana, Slack 알람 |
| **DevOps** | Docker, Terraform, GitHub Actions, AWS (EC2/S3/CloudFront/SSM/ASG/Lambda) |
| **Frontend** | React, Vite, React Flow, WebGL 차트 |

---

## 성과 중심 경험

### LoadTest Converter / Desktop (개인 프로젝트, 2025)

k6 부하테스트 시나리오를 GUI로 작성하고 실행하는 도구 시리즈. Go로 직접 구현.

**LoadTest Converter** — k6 시나리오 YAML 빌더 (Go + React, Render 배포)
- Go `net/http` 표준 라이브러리 기반 REST API 서버 구현
- `//go:embed` 로 k6 템플릿을 단일 바이너리에 내장, `archive/zip` 으로 ZIP 패키징
- Preview / Export / Import 3-endpoint 설계 (YAML 생성 · ZIP 내보내기 · 역파싱)

**LoadTest Desktop** — k6 부하테스트 데스크탑 실행기 (Go + Wails)
- 별도 엔진 바이너리 제거 → `engine/` 패키지로 흡수, 단일 실행파일 배포
- `io.Writer` 인터페이스 기반 실시간 로그 스트리밍 설계 (`sync.Mutex` 스레드 안전 보장)
- k6 출력 `io.MultiWriter(logFile, w)` 로 파일 저장 + UI 실시간 전달 동시 처리
- `depends_on` 기반 wave 병렬 스텝 실행 (goroutine + WaitGroup + channel 에러 수집)

---

### Trader Platform (개인 프로젝트, 2025–현재)

주식투자 학습을 위한 복기 플랫폼. 25–30M+ OHLCV 조회 성능 개선에서 시작해 실시간 협업, 장애 복구, KIS/BLS/SEC 데이터 파이프라인과 worker control plane까지 확장했습니다.

**성능 개선**
- k6 부하 테스트 + TimescaleDB 하이퍼테이블·청크 튜닝 → P95 **7,247ms → 235ms** @ 300 RPS **(28배 개선, SLO 달성)**
- 복합 인덱스 단독 적용만으로 P95 **342ms → 32ms (10배)** — 원인을 단계별로 분리해 측정
- JPA Fetch 전략 4차 비교 실험 (Lazy N+1 / Fetch Join / Projection / DB preview) → 10K payload 기준 붕괴 RPS **5배 차이** 수치 확인
- JFR/JMC Stack Trace로 JWT 중복 검증 hot path 발견 → 제거 후 Old GC **기준치 대비 36% 감소**

**실시간 시스템**
- WebSocket TEXT_PARTIAL_WRITING 동시성 버그 분석 → Dirty Flag 최신값 전송으로 재설계 → ≤200ms 수신율 **0.38% → 99.97%**
- groupId 해시 샤딩: totalSendAttempts **159K → 79K + 79K** 균등 분배, GC **3회 → 1회**, byte[] Allocation **205MiB → 93+111MiB** (JFR 실측)
- shard 장애 시 Redis Draft 편집 상태 보존 + `dirtyFields ∩ serverChangedFields` 기반 **AUTO_MERGE / CONFLICT 자동 판별**
- Kafka Catch-up Consumer offset replay → catchupCompleted 후 Broadcast 전환, **이벤트 유실 없는 무중단 Failback** E2E 검증

**아키텍처 및 운영**
- TimescaleDB Continuous Aggregate (CAGG) 활용한 1W/1M/1Y 조회 전략 설계
- KIS 주가, BLS 거시경제, SEC 재무 데이터를 raw로 보존하고 Python ETL Worker가 정규화 테이블에 적재하는 파이프라인 구현
- DB outbox, `source_object`, `record_lineage`, `processed_event`로 Kafka 발행부터 raw, ETL, consumer commit까지 추적
- Go Controller가 Kafka lag와 worker heartbeat를 판단해 AWS ASG desired capacity를 **0 -> 1 -> 0**으로 조절하는 흐름 검증
- ETL worker 중지 중 BLS raw lag **2** 누적, worker 재개 후 lag **0**과 `PROCESSED` 전환 확인
- Prometheus/Grafana 모니터링 + Slack 알람, GitHub Actions CI/CD, JaCoCo ≥70% 기준
- Google/Kakao/Naver JWT SSO 구현

---

### SIC Club Portal (팀 프로젝트, 2025.09–2025.11)

FE/BE/AI/Design 14명 팀 리딩, 동아리 운영 웹 서비스 구축

- 역할 기준(FE/BE 분리) 스프린트에서 작업 추적 및 API 의존성 문제 발생  
  → 기능 단위로 FE-BE를 묶는 스프린트 구조로 재설계하여 개발 흐름 개선

- GitHub Actions 기반 CI/CD 및 JaCoCo ≥70% 테스트 기준 도입  
  → 병합 전 오류 사전 검출 및 배포 안정성 확보

- 학업 병행으로 스프린트 지연 발생 시 테스트 기준을 일시 완화하고 기능 개발 중심으로 전략 전환  
  → MVP 일정 내 완성 및 개발 속도 회복

- EC2 t3.micro 환경에서 OOM 장애 발생 → free -h 기반 원인 분석  
  → t3.medium + 1GB swap 적용으로 안정화 및 인프라 운영 기준 수립

- AWS EC2/S3/CloudFront/RDS + SSM 기반 인프라 설계 및 배포 자동화  
  → EventBridge + Lambda로 운영 시간 제어(09–18시)하여 비용 최적화


---

## 학력

- **세종대학교** (Sejong University)

---

## 링크

| | |
|--|--|
| GitHub | [github.com/Kosw6](https://github.com/Kosw6) |
| Portfolio | [kosw6.github.io](https://kosw6.github.io) |
| Engineering Reports | [kosw6.github.io/reports](https://kosw6.github.io/reports/) |
