---
layout: splash
title: "Sungwon Kim"

header:
  overlay_filter: 0.5
  overlay_color: "#0b0f19"
  excerpt: >
    **성능을 측정하고, 병목을 추적하고, 구조로 해결하는 백엔드 엔지니어.**<br><br>
    TimescaleDB 쿼리 **28배 개선** &nbsp;·&nbsp; WebSocket 실시간성 **0.38% → 99.97%**<br>
    JFR/JMC 런타임 분석 &nbsp;·&nbsp; Kafka 기반 무중단 복구 설계

intro:
  - excerpt: |
      ## Performance Highlights
      추측이 아닌 실측 기반으로 병목을 찾고, 수치로 개선 결과를 검증했습니다.

highlights_row1:
  - title: "28× Faster Time-Series Query"
    excerpt: |
      **P95 7,247ms → 235ms @ 300 RPS**
      인덱스 미적용 + 하이퍼테이블 누락을 원인으로 확인. 인덱스 → 하이퍼테이블 → 청크 튜닝 3단계 개선으로 SLO 달성.
    url: "/reports/timescaledb-27x/"
    btn_label: "Report 보기"
    btn_class: "btn--primary"
  - title: "JPA Fetch Strategy Tuning"
    excerpt: |
      **붕괴 RPS 5배 차이 실측**
      Lazy N+1 → Fetch Join → DB 레벨 preview 전환. 10K payload 기준 4차 비교 실험, GC Pause 수치까지 측정.
    url: "/reports/jpa-tuning/"
    btn_label: "Report 보기"
    btn_class: "btn--primary"

highlights_row2:
  - title: "JFR / JMC Allocation Hotspot"
    excerpt: |
      **Old GC 36% 감소**
      쿼리 튜닝 이후 남은 병목을 JMC Stack Trace로 추적. JWT 중복 검증이 hot path임을 확인하고 제거.
    url: "/reports/jfr-jmc-hotpath/"
    btn_label: "Report 보기"
    btn_class: "btn--primary"
  - title: "WebSocket 실시간성 0.38% → 99.97%"
    excerpt: |
      **≤200ms 수신율 99.97% 달성**
      TEXT_PARTIAL_WRITING 동시성 버그 → Dirty Flag 기반 최신값 전송 전략으로 재설계. 브로드캐스트 구조 분석 포함.
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
