---
layout: splash
title: "Sungwon Kim"

header:
  overlay_filter: 0.5
  overlay_color: "#0b0f19"
  excerpt: >
    **성능 병목을 추적하고, 수치로 증명하는 백엔드 개발자.**<br><br>
    TimescaleDB P95 &nbsp;<del>7,247ms</del>&nbsp; → &nbsp;<strong>235ms</strong>&nbsp; (28배)
    &nbsp;·&nbsp;
    WebSocket &nbsp;<del>0.38%</del>&nbsp; → &nbsp;<strong>99.97%</strong>&nbsp; ≤200ms<br>
    JFR/JMC Old GC <strong>36% 감소</strong> &nbsp;·&nbsp; Kafka 기반 무중단 Failback 설계

intro:
  - excerpt: |
      ## Performance Highlights
      추측이 아닌 실측 기반으로 병목을 찾고, 수치로 개선 결과를 검증했습니다.

highlights_row1:
  - title: "TimescaleDB — P95 7,247ms → 235ms"
    excerpt: |
      **300 RPS 기준 28배 개선, SLO 달성**
      하이퍼테이블 누락 + 인덱스 미적용을 원인으로 확인. 인덱스(10배) → 하이퍼테이블 → 청크 튜닝 3단계로 순차 개선.
    url: "/reports/timescaledb-27x/"
    btn_label: "Report 보기"
    btn_class: "btn--primary"
  - title: "JPA Fetch — 붕괴 RPS 5배 차이 실측"
    excerpt: |
      **Lazy N+1 → DB preview 전환**
      4가지 전략(Lazy / Fetch Join / Projection / DB preview)을 동일 조건 4차 비교. 10K payload 기준 붕괴 RPS 5배 차이 확인. GC Pause 수치까지 측정.
    url: "/reports/jpa-tuning/"
    btn_label: "Report 보기"
    btn_class: "btn--primary"

highlights_row2:
  - title: "JFR / JMC — Old GC 횟수 36% 감소"
    excerpt: |
      **쿼리 튜닝 후 남은 GC를 프로파일러로 추적**
      GC가 예상보다 많이 발생 → JMC Stack Trace 분석 → JWT 중복 검증이 hot path 확인 → 제거. 수치 개선 검증.
    url: "/reports/jfr-jmc-hotpath/"
    btn_label: "Report 보기"
    btn_class: "btn--primary"
  - title: "WebSocket — 0.38% → 99.97% ≤200ms"
    excerpt: |
      **실시간 수신 실패율 99.6%p 개선**
      TEXT_PARTIAL_WRITING 동시성 버그 분석 → Dirty Flag 기반 최신값 단건 전송으로 재설계. 부하 100명 기준 E2E 검증.
    url: "/reports/websocket-group-canvas/"
    btn_label: "Report 보기"
    btn_class: "btn--primary"

poc_intro:
  - excerpt: |
      ## WebSocket 수평 확장 PoC 시리즈
      단일 인스턴스 최적화 이후, **분산 환경에서 어떻게 확장하고 장애 없이 복구할 것인가**를 단계별로 설계·검증했습니다.
      각 PoC는 이전 결과 위에서 다음 문제를 다룹니다. **PoC 1부터 순서대로 읽기를 권장합니다.**

poc_row:
  - title: "PoC 1 — 그룹 샤딩 & 부하 분산"
    excerpt: |
      **fanout 부하 인스턴스별 50% 분산 실측**
      groupId 해시 기반 샤딩. totalSendAttempts 159K → 79K+79K 균등 분배.
      JFR 실측: GC 3회 → 1회, byte[] Allocation 205MiB → 93+111MiB.
    url: "/reports/websocket-poc1-sharding/"
    btn_label: "PoC 1 보기"
    btn_class: "btn--primary"
  - title: "PoC 2 — Fallback & 편집 충돌 제어"
    excerpt: |
      **CONFLICT / AUTO_MERGE 자동 판별 검증**
      shard 장애 시 다른 인스턴스로 우회 + Redis Draft로 편집 상태 보존.
      dirtyFields ∩ serverChangedFields 기반 충돌 감지 → E2E 3 케이스 전체 검증.
    url: "/reports/websocket-poc2-conflict/"
    btn_label: "PoC 2 보기"
    btn_class: "btn--primary"
  - title: "PoC 3 — Failback & Kafka Replay"
    excerpt: |
      **이벤트 유실 없는 무중단 서버 복구**
      Kafka offset 기반 누락 이벤트 replay → catchupCompleted 후 broadcast 전환.
      Drain → 재연결 요청 → 세션 전환 전 과정 E2E 검증.
    url: "/reports/websocket-poc3-failback/"
    btn_label: "PoC 3 보기"
    btn_class: "btn--primary"

projects_intro:
  - excerpt: |
      ## Featured Projects
      성능, 확장성, 운영 경험을 중심으로 설계하고 구현한 프로젝트들입니다.

project_row:
  - title: "Trader Platform"
    excerpt: |
      **40–50M+ OHLCV 시계열 데이터 처리 플랫폼**
      TimescaleDB 하이퍼테이블 + 청크 튜닝, k6 부하 테스트, Prometheus/Grafana 모니터링, Google/Kakao/Naver SSO.
    url: "/projects/trader/"
    btn_label: "Project 보기"
    btn_class: "btn--primary"
  - title: "SIC Club Portal"
    excerpt: |
      **FE/BE/AI/Design 14–15명 팀 리딩**
      GitHub Actions CI/CD, JaCoCo ≥70% 커버리지 기준 수립, Jira/Slack/Notion 워크플로우 구축.
    url: "/projects/sic-portal/"
    btn_label: "Project 보기"
    btn_class: "btn--primary"

---

{% include feature_row id="intro" type="center" %}

{% include feature_row id="highlights_row1" %}
{% include feature_row id="highlights_row2" %}

{% include feature_row id="poc_intro" type="center" %}
{% include feature_row id="poc_row" %}

{% include feature_row id="projects_intro" type="center" %}
{% include feature_row id="project_row" %}
