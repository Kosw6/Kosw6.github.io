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
      실측 기반 성능 개선 사례를 중심으로, 병목 분석 → 구조 개선 → 결과를 정리했습니다.

highlight_1:
  - title: "27× Faster Time-Series Query"
    excerpt: |
      **TimescaleDB 기반 시계열 조회 성능 27배 개선**  
      하이퍼테이블 적용과 쿼리 플랜 비교를 통해 대규모 시계열 데이터 조회 성능을 개선했습니다.
    url: "/reports/timescaledb-27x/"
    btn_label: "View Report"

highlight_2:
  - title: "JPA Query Performance Optimization"
    excerpt: |
      **Canvas Node 조회 성능 개선**  
      fetch 전략, 쿼리 구조, 데이터 반환 형태를 비교하며 JPA 조회 성능을 튜닝했습니다.
    url: "/reports/jpa-tuning/"
    btn_label: "View Report"

highlight_3:
  - title: "JFR / JMC Allocation Hotspot Analysis"
    excerpt: |
      **JVM allocation bottleneck 추적 및 개선**  
      JFR/JMC 분석으로 Hibernate 내부 hot path와 allocation 병목을 확인하고 불필요한 흐름을 개선했습니다.
    url: "/reports/jfr-jmc-hotpath/"
    btn_label: "View Report"

highlight_4:
  - title: "≤200ms Realtime Delivery Improved (48% → 99%)"
    excerpt: |
      **WebSocket fan-out 병목 분석 및 실시간성 개선**  
      Group Canvas 브로드캐스트 구조를 분석하고 RAW WebSocket 기반으로 개선해 200ms 이내 수신율을 향상시켰습니다.
    url: "/reports/websocket-group-canvas/"
    btn_label: "View Report"

projects_intro:
  - excerpt: |
      ## Featured Projects
      성능, 확장성, 운영 경험을 중심으로 설계하고 구현한 프로젝트들입니다.

project_1:
  - title: "Trader Platform"
    excerpt: |
      **40–50M+ OHLCV 시계열 데이터 처리 플랫폼**  
      Spring Boot, PostgreSQL/TimescaleDB 기반으로 트레이딩 데이터 분석 서비스를 설계하고 성능을 최적화했습니다.
    url: "/projects/trader/"
    btn_label: "View Project"

project_2:
  - title: "SIC Club Portal"
    excerpt: |
      **팀 리딩 기반 웹 서비스 프로젝트**  
      팀원 모집, 서비스 기획, 백엔드 개발, AWS 기반 인프라 및 CI/CD 구축을 수행했습니다.
    url: "/projects/sic-portal/"
    btn_label: "View Project"

---

{% include feature_row id="intro" type="center" %}

{% include feature_row id="highlight_1" type="left" %}
{% include feature_row id="highlight_2" type="left" %}
{% include feature_row id="highlight_3" type="left" %}
{% include feature_row id="highlight_4" type="left" %}

{% include feature_row id="projects_intro" type="center" %}

{% include feature_row id="project_1" type="left" %}
{% include feature_row id="project_2" type="left" %}