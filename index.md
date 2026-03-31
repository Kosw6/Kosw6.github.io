---
layout: splash
title: "Sungwon Kim"

header:
  overlay_filter: 0.45
  overlay_color: "#0b0f19"
  excerpt: >
    Backend Engineer focused on performance optimization, realtime architecture, and scalable systems.<br>
    Spring Boot · PostgreSQL · Redis · AWS · Performance Engineering

intro:
  - excerpt: |
      ## Performance Highlights
      실측 기반 성능 개선 사례를 중심으로, 병목 분석 → 구조 개선 → 정량 결과를 정리했습니다.

highlights_row1:
  - title: "27× Faster Time-Series Query"
    excerpt: |
      **TimescaleDB 시계열 조회 성능 28배 개선**
      인덱스 미적용 + 하이퍼테이블 누락으로 P95 7,247ms → 하이퍼테이블 + 청크 튜닝으로 **235ms (300 RPS)** 달성.
    url: "/reports/timescaledb-27x/"
    btn_label: "View Report"
    btn_class: "btn--primary"
  - title: "JPA Fetch Strategy Tuning"
    excerpt: |
      **Canvas Node 조회 성능 개선**
      Lazy N+1 → Fetch Join + DB 레벨 preview 전환. 10K payload 기준 붕괴 RPS **5배 차이** 확인. GC Pause 수치까지 측정.
    url: "/reports/jpa-tuning/"
    btn_label: "View Report"
    btn_class: "btn--primary"

highlights_row2:
  - title: "JFR / JMC Allocation Hotspot"
    excerpt: |
      **JVM 런타임 분석으로 인증 핫패스 발견**
      쿼리 튜닝 이후 보이지 않던 병목을 JMC Stack Trace로 추적. JWT 중복 검증 제거 → Old GC **36% 감소**.
    url: "/reports/jfr-jmc-hotpath/"
    btn_label: "View Report"
    btn_class: "btn--primary"
  - title: "≤200ms Realtime Delivery (48% → 99%)"
    excerpt: |
      **WebSocket 브로드캐스트 실시간성 개선**
      TEXT_PARTIAL_WRITING 동시성 버그 → Dirty Flag 기반 최신값 전송 전략으로 ≤200ms 성공률 **0.38% → 99.97%**.
    url: "/reports/websocket-group-canvas/"
    btn_label: "View Report"
    btn_class: "btn--primary"

poc_intro:
  - excerpt: |
      ## WebSocket 수평 확장 PoC 시리즈
      단일 인스턴스 최적화 이후, **분산 환경에서 어떻게 확장하고 장애를 복구할 것인가**를 단계별로 설계·검증했습니다.
      순서대로 읽으면 샤딩 → 충돌 제어 → 무중단 복구로 이어집니다.

poc_row:
  - title: "PoC 1 — 그룹 샤딩 & 부하 분산"
    excerpt: |
      groupId 기반 샤딩으로 fanout을 인스턴스 단위로 분리.
      totalSendAttempts **159K → 79K+79K**, GC **3회 → 1회**, byte[] Allocation **205MiB → 93+111MiB** (JFR 실측).
    url: "/reports/websocket-poc1-sharding/"
    btn_label: "PoC 1 보기"
    btn_class: "btn--primary"
  - title: "PoC 2 — Fallback & 편집 충돌 제어"
    excerpt: |
      shard 장애 시 다른 인스턴스로 우회 + Redis Draft로 편집 상태 유지.
      내가 편집 중 발생한 서버 변경을 필드 단위로 추적 → **AUTO_MERGE / CONFLICT** 자동 판별.
    url: "/reports/websocket-poc2-conflict/"
    btn_label: "PoC 2 보기"
    btn_class: "btn--primary"
  - title: "PoC 3 — Kafka Replay & 무중단 Failback"
    excerpt: |
      장애 서버 복구 시 Kafka offset 기반 누락 이벤트 replay → 이벤트 유실 없이 상태 복구.
      Drain → 재연결 요청 → 서버 전환까지 **사용자 서비스 중단 없이** 처리.
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
      Spring Boot, PostgreSQL/TimescaleDB 기반으로 트레이딩 데이터 분석 서비스를 설계하고 성능을 최적화했습니다.
    url: "/projects/trader/"
    btn_label: "View Project"
    btn_class: "btn--primary"
  - title: "SIC Club Portal"
    excerpt: |
      **팀 리딩 기반 웹 서비스 프로젝트**
      팀원 모집, 서비스 기획, 백엔드 개발, AWS 기반 인프라 및 CI/CD 구축을 수행했습니다.
    url: "/projects/sic-portal/"
    btn_label: "View Project"
    btn_class: "btn--primary"

---

{% include feature_row id="intro" type="center" %}

{% include feature_row id="highlights_row1" %}
{% include feature_row id="highlights_row2" %}

{% include feature_row id="poc_intro" type="center" %}
{% include feature_row id="poc_row" %}

{% include feature_row id="projects_intro" type="center" %}
{% include feature_row id="project_row" %}
