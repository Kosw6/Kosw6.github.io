---
layout: collection
title: "Engineering Reports"
permalink: /reports/
collection: reports
entries_layout: grid
---

## Engineering Reports

성능 분석, 아키텍처 설계, 부하 테스트 결과를 정리한 엔지니어링 문서입니다.

각 보고서는 다음 구조로 작성됩니다.

- Problem (문제)
- Analysis (분석)
- Solution (개선)
- Result (정량 결과)

대표 주제

- TimescaleDB 시계열 조회 성능 튜닝
- JPA Fetch 전략별 성능 비교 및 Payload 영향 분석
- JFR / JMC 기반 Allocation Hotspot 분석
- WebSocket 실시간 브로드캐스트 성능 개선
- WebSocket 수평 확장 PoC 시리즈
  - PoC 1: 그룹 샤딩 기반 부하 분산 (fanout locality + JVM GC 감소)
  - PoC 2: Fallback 환경 상태 동기화 및 필드 단위 충돌 제어
  - PoC 3: Kafka Replay 기반 Failback & 무중단 서버 전환