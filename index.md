---
layout: splash
title: "Sungwon Kim"

excerpt: >
  **성능 병목을 분석하고, 수치로 개선하는 백엔드 개발자**<br><br>
  P95 7,247ms → <strong>235ms (28× 개선)</strong><br>
  WebSocket ≤200ms <strong>99.97%</strong><br>
  성능 분석 기반 병목 해결 · 분산 환경 장애 복구 설계

header:
  overlay_image: /assets/images/hero-bg-1.png
  overlay_filter: 0.6
  overlay_color: "#000000"
  actions:
    - label: "성능 개선 과정 보기"
      url: "#highlights"

intro:
  - excerpt: |
      ## Performance Highlights {#highlights}
      실측 기반으로 병목을 분석하고, 수치로 개선을 검증했습니다.

highlights_row1:
  - title: "TimescaleDB — 28× 개선"
    excerpt: |
      **P95 7,247ms → 235ms**<br>
      원인: 인덱스 + 하이퍼테이블 미적용<br>
      해결: 인덱스 → 하이퍼테이블 → 청크 튜닝
    url: "/reports/timescaledb-27x/"
    btn_label: "Report 보기"
    btn_class: "btn--primary"

  - title: "JPA Fetch — RPS 5배 차이"
    excerpt: |
      **Lazy vs Fetch Join vs Projection 비교**<br>
      핵심: N+1 + 객체 생성 수 + GC 영향 분석<br>
      결과: Fetch Join + preview 구조 선택
    url: "/reports/jpa-tuning/"
    btn_label: "Report 보기"
    btn_class: "btn--primary"

highlights_row2:
  - title: "JFR/JMC — GC 병목 추적"
    excerpt: |
      **Hot Path 개선으로 GC 개선**<br>
      JWT 중복 검증 제거 → GC 부담 감소<br>
      2-step 구조 PoC → GC 증가로 미채택
    url: "/reports/jfr-jmc-hotpath/"
    btn_label: "Report 보기"
    btn_class: "btn--primary"

  - title: "WebSocket — 실시간 안정성 개선"
    excerpt: |
      **0.38% → 99.97% ≤200ms**<br>
      동시성 문제 해결 + Dirty Flag 구조 적용 <br>
      (STOMP vs RAW 비교 기반 설계)<br>
    url: "/reports/websocket-group-canvas/"
    btn_label: "Report 보기"
    btn_class: "btn--primary"

poc_intro:
  - excerpt: |
      ## WebSocket 수평 확장 PoC
      WebSocket 안정성 개선 이후, 단일 서버의 구조적 한계를<br>
      성능 → 분산 → 정합성 → 복구 순서로 확장 설계·검증했습니다.

poc_row:
  - title: "PoC 1 — 샤딩"
    excerpt: |
      **fanout 부하 50% 분산**<br>
      159K → 79K + 79K<br>
      Gateway + 슬롯 기반 샤딩
    url: "/reports/websocket-poc1-sharding/"
    btn_label: "PoC 1"
    btn_class: "btn--primary"

  - title: "PoC 2 — Fallback"
    excerpt: |
      **충돌 감지 및 자동 병합**<br>
      편집 상태 유실 없이 장애 복구<br>
      Redis 기반 상태 유지 + Gateway fallback
    url: "/reports/websocket-poc2-conflict/"
    btn_label: "PoC 2"
    btn_class: "btn--primary"

  - title: "PoC 3 — Failback"
    excerpt: |
      이벤트 유실 없이 서버 상태를 복구하고 전환<br>
      → Kafka replay + 상태 기반 lifecycle 제어
    url: "/reports/websocket-poc3-failback/"
    btn_label: "PoC 3"
    btn_class: "btn--primary"

projects_intro:
  - excerpt: |
      ## Featured Projects
      성능, 확장성, 운영을 중심으로 설계한 프로젝트입니다.

project_row:
  - title: "Trader Platform"
    excerpt: |
      **40M+ 시계열 데이터 처리**
      TimescaleDB + k6 + Grafana
    url: "/projects/trader/"
    btn_label: "Project"
    btn_class: "btn--primary"

  - title: "SIC Club Portal"
    excerpt: |
      **팀 리딩 & CI/CD 구축**
      GitHub Actions + JaCoCo
    url: "/projects/sic-portal/"
    btn_label: "Project"
    btn_class: "btn--primary"

---

{% include feature_row id="intro" type="center" %}

{% include feature_row id="highlights_row1" %}
{% include feature_row id="highlights_row2" %}

{% include feature_row id="poc_intro" type="center" %}
{% include feature_row id="poc_row" %}

{% include feature_row id="projects_intro" type="center" %}
{% include feature_row id="project_row" %}