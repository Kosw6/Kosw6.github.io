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
- k6 부하 테스트 설계 및 TimescaleDB 하이퍼테이블 도입 → **P95 7,247ms → 235ms @ 300 RPS (28배 개선)**
- 인덱스 단계에서만 **P95 342ms → 32ms (10배)**, 이후 하이퍼테이블 + 청크 튜닝으로 최종 SLO 달성
- JPA Fetch 전략 4차 비교 실험 (Lazy N+1 / Fetch Join / Projection / DB preview) → 10K payload 기준 붕괴 RPS **5배 차이** 확인
- JFR/JMC 런타임 프로파일링으로 쿼리 튜닝 이후 숨어있던 JWT 중복 검증 발견 → **Old GC 36% 감소**

**실시간 시스템**
- WebSocket 브로드캐스트 동시성 버그(TEXT_PARTIAL_WRITING) 분석 → Dirty Flag 기반 최신값 전송 전략으로 재설계 → **≤200ms 수신율 0.38% → 99.97%**
- groupId 해시 기반 WebSocket 샤딩 설계 · JFR 실측 검증 (GC 3회 → 1회, byte[] Allocation 205MiB → 93+111MiB)
- shard 장애 시 Redis Draft 기반 편집 상태 보존 + dirtyFields 기반 **AUTO_MERGE / CONFLICT 자동 판별** 구현
- Kafka offset replay 기반 **이벤트 유실 없는 무중단 Failback** 설계 및 E2E 검증

**아키텍처 및 운영**
- TimescaleDB Continuous Aggregate (CAGG) 활용한 1W/1M/1Y 조회 전략 설계
- Prometheus/Grafana 모니터링 + Slack 알람, GitHub Actions CI/CD, JaCoCo ≥70% 기준
- Google/Kakao/Naver JWT SSO 구현

---

### SIC Club Portal (팀 프로젝트, 2025–현재)

14–15명(FE/BE/AI/Design) 팀을 리딩한 동아리 운영 관리 플랫폼.

- Atlassian-style 워크플로우 수립: CODEOWNERS, PR 규칙, Jira/Slack/Notion 연동
- GitHub Actions CI/CD 파이프라인 구축, JaCoCo ≥70% 테스트 커버리지 기준 팀 전체 적용
- 출결/퀴즈/계약 모듈 기획 및 백엔드 구현
- AWS 기반 인프라 구성 및 배포 자동화 (EC2, S3, CloudFront, RDS)
- React + Vite PWA, Grafana 대시보드 구성

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
