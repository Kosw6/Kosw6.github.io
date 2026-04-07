---
layout: single
title: "Resume"
permalink: /resume/
classes: wide
toc: true
toc_sticky: true
---

## Profile

**Sungwon Kim — Backend Engineer**

성능을 측정하고, 병목을 추적하고, 구조로 해결하는 엔지니어.
단순 기능 구현을 넘어 k6 부하 테스트 · JFR/JMC 런타임 프로파일링 · 분산 시스템 설계까지 실측 기반으로 접근합니다.

- **GitHub**: [github.com/Kosw6](https://github.com/Kosw6)
- **Portfolio**: [kosw6.github.io](https://kosw6.github.io)

---

## 기술 스택

| 분류 | 기술 |
|------|------|
| **Backend** | Spring Boot, Spring MVC, JPA/Hibernate, Spring Security |
| **Database** | PostgreSQL, TimescaleDB (하이퍼테이블 · 청크 튜닝 · CAGG), 인덱싱 전략 |
| **Cache / MQ** | Redis (TTL 설계, JSON Serializer), Kafka (Consumer Group, offset replay) |
| **Realtime** | WebSocket (RAW), 샤딩 · Fallback · Failback 설계 |
| **Performance** | k6, JFR/JMC, Prometheus/Grafana, Slack 알람 |
| **DevOps** | Docker, GitHub Actions, JaCoCo (≥70% 기준), AWS (EC2/S3/CloudFront/RDS) |
| **Frontend** | React, Vite, React Flow, WebGL 차트 |

---

## 성과 중심 경험

### Trader Platform (개인 프로젝트, 2024–현재)

40–50M+ OHLCV 시계열 데이터 처리 플랫폼. 성능 측정 → 원인 분석 → 구조 개선 사이클을 반복하며 운영 중.

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
