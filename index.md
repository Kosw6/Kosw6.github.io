---
layout: splash
title: "Sungwon Kim"

excerpt: >
  **성능 병목을 분석하고, 수치로 개선하는 백엔드 개발자**<br><br>
  P95 7,247ms → <strong>235ms (28× 개선)</strong><br>
  WebSocket ≤200ms <strong>99.97%</strong><br>
  Go 기반 Load Test Orchestrator로 Terraform, k6, AWS SSM 장애 주입 자동화<br>


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

  - title: "PoC 2 — Failover & Fallback"
    excerpt: |
      **충돌 감지 및 자동 병합**<br>
      편집 상태 유실 없이 장애 복구<br>
      Redis 기반 상태 유지 + Gateway failover
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

reliability_intro:
  - excerpt: |
      ## Reliability & Operations
      관측 → 운영 기준 → degraded mode → 자동 복구 → 검증 자동화 순서로 운영 안정성을 설계하고 검증했습니다.

reliability_row:
  - title: "1. Observability System"
    excerpt: |
      **TraceId + Loki/Grafana 관측 체계**<br>
      구조화 로그, error count/rate alert<br>
      장애 원인 추적을 위한 운영 로그 설계
    url: "/reports/observability-system/"
    btn_label: "Report"
    btn_class: "btn--primary"

  - title: "2. SLO 기반 운영 사양 산정"
    excerpt: |
      **p95 <= 300ms 기준 인프라 산정**<br>
      App/DB 2vCPU/4GB 구성 검증<br>
      Linux context switch와 DB pending 분석
    url: "/reports/slo-operating-capacity/"
    btn_label: "Report"
    btn_class: "btn--primary"

  - title: "3. Realtime Degraded Mode"
    excerpt: |
      **Redis/Kafka/PubSub 장애 대응**<br>
      Hot path와 durable replay 경로 분리<br>
      Kafka replay 설계 근거까지 연결
    url: "/reports/realtime-degrade-overview/"
    btn_label: "Report"
    btn_class: "btn--primary"

orchestrator_ops_row:
  - title: "4. Auto Recovery & Scale-out"
    excerpt: |
      **Grafana Alert -> Lambda -> SSM/ASG**<br>
      App restart, scale-out, 신규 인스턴스 합류 검증<br>
      Gateway health 기반 failover/failback
    url: "/reports/auto-recovery-scaleout/"
    btn_label: "Report"
    btn_class: "btn--primary"

  - title: "5. Load Test Orchestrator"
    excerpt: |
      **Terraform + k6 + Redis 장애 주입 자동화**<br>
      WebSocket errors 0, received 34k -> 89k<br>
      SSM polling과 recovery health check
    url: "/reports/loadtest-orchestrator-redis-fault-validation/"
    btn_label: "Report"
    btn_class: "btn--primary"

projects_intro:
  - excerpt: |
      ## Featured Projects
      성능, 확장성, 운영 검증을 중심으로 설계한 대표 프로젝트입니다.

project_row:
  - title: "Trader Platform"
    excerpt: |
      **25~30M+ 시계열 데이터 처리**<br>
      TimescaleDB + k6 + Grafana<br>
      성능 병목, 장애 대응, 관측 체계 검증
    url: "/projects/trader/"
    btn_label: "Project"
    btn_class: "btn--primary"

  - title: "Load Test Orchestrator"
    excerpt: |
      **Terraform + k6 + Redis 장애 검증**<br>
      ZIP 시나리오 기반 E2E 자동화<br>
      SSM command polling + recovery check
    url: "/projects/loadtest-orchestrator/"
    btn_label: "Project"
    btn_class: "btn--primary"

  - title: "SIC Club Portal"
    excerpt: |
      **팀 리딩 & CI/CD 구축**<br>
      Nginx reverse proxy + CloudFront<br>
      GitHub Actions + AWS 배포 인프라
    url: "/projects/sic-portal/"
    btn_label: "Project"
    btn_class: "btn--primary"

---

{% include feature_row id="intro" type="center" %}

{% include feature_row id="highlights_row1" %}
{% include feature_row id="highlights_row2" %}

{% include feature_row id="poc_intro" type="center" %}
{% include feature_row id="poc_row" %}

{% include feature_row id="reliability_intro" type="center" %}
{% include feature_row id="reliability_row" %}
{% include feature_row id="orchestrator_ops_row" %}

{% include feature_row id="projects_intro" type="center" %}
{% include feature_row id="project_row" %}
