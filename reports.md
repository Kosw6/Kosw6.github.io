---
layout: collection
title: "Engineering Reports"
permalink: /reports/
collection: reports
entries_layout: grid
---

## Engineering Reports

성능 병목 분석뿐 아니라 장애 대응, 관측 체계, 자동 복구, 부하 테스트 자동화까지 실제 운영 환경을 기준으로 검증한 문서들입니다.

각 보고서는 다음 관점으로 정리합니다.

- Problem: 어떤 운영 또는 성능 문제가 있었는가
- Analysis: 어떤 지표, 로그, 부하 테스트로 원인을 확인했는가
- Solution: 어떤 구조나 설정을 선택했는가
- Result: 수치와 로그 기준으로 무엇이 개선되었는가

주요 주제:

### Reliability & Operations 시리즈

- Observability System: TraceId, 구조화 로그, Loki/Grafana alert
- SLO 기반 운영 사양 산정: p95 기준 App/DB 사양과 Thread/Hikari 설정 결정
- Realtime Degraded Mode: Redis/Kafka/PubSub 장애 대응과 Kafka replay 설계 근거
- Auto Recovery & Scale-out: Grafana Alert, Lambda, SSM, ASG 기반 복구/확장 검증
- Load Test Orchestrator: Terraform, k6, AWS SSM 기반 장애 검증 자동화

- Performance tuning: TimescaleDB, JPA fetch strategy, JFR/JMC hotspot analysis
- Realtime architecture: WebSocket broadcast, sharding, failover, failback
- Reliability: Redis/Kafka degraded mode, Pub/Sub fallback, DB fallback, Outbox replay
- Observability: TraceId logging, Loki/Grafana dashboard, error rate alert
- Automation: Terraform, k6, AWS SSM 기반 장애 검증 오케스트레이션

