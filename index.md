---
layout: splash
title: "Sungwon Kim"

excerpt: >
  **성능 병목을 추적하고, 수치로 증명하는 백엔드 개발자.**<br><br>
  TimescaleDB P95 &nbsp;7,247ms&nbsp; → &nbsp;<strong>235ms</strong>&nbsp; (28× 개선)<br>
  WebSocket &nbsp;0.38%&nbsp; → &nbsp;<strong>99.97%</strong>&nbsp; ≤200ms<br><br>
  JFR/JMC 기반 GC 패턴 분석 · Kafka 기반 <strong>장애 복구(Failback) 설계</strong>

header:
  overlay_image: /assets/images/hero-bg-1.png
  overlay_filter: 0.6
  overlay_color: "#000000"
  actions:
    - label: "대표 리포트 보기"
      url: "#highlights"

intro:
  - excerpt: |
      ## Performance Highlights {#highlights}
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
  - title: "JFR/JMC 기반 Hot Path 제거 · Fetch 구조 PoC 후 성능 악화 확인 및 미채택"
    excerpt: |
      쿼리 튜닝 이후에도 SLO 미달 → JFR/JMC로 추가 분석 수행.
      JMC Stack Trace를 통해 JWT 중복 검증이 hot path에 위치함을 확인하고 제거.

      이후 Fetch Join → 2-step 구조로 PoC를 진행했으나,
      P95 지연 증가와 Old GC 시간 약 36% 증가를 확인.
      이는 병목 제거가 아닌 GC 부담 이동으로 판단되어 최종 미채택.
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
