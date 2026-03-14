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
  - excerpt: '## Performance Highlights'

highlights_row:
  - title: "27x Database Improvement"
    excerpt: "TimescaleDB 하이퍼테이블 최적화를 통해 대규모 시계열 데이터 조회 성능을 27배 개선"
    url: "/reports/timescaledb-27x/"
    btn_label: "View Report"

  - title: "JPA Query Optimization"
    excerpt: "Canvas Node 조회 성능 개선을 위해 JPA fetch 전략과 쿼리 구조를 분석 및 튜닝"
    url: "/reports/jpa-tuning/"
    btn_label: "View Report"

  - title: "JFR / JMC Hot Path Analysis"
    excerpt: "JVM recording과 JMC 분석을 통해 핫패스 및 allocation bottleneck을 추적하고 개선"
    url: "/reports/jfr-jmc-hotpath/"
    btn_label: "View Report"

  - title: "Realtime WebSocket Architecture"
    excerpt: "Group Canvas 브로드캐스트 병목을 분석하고 Redis Pub/Sub 기반 확장 전략 설계"
    url: "/reports/websocket-group-canvas/"
    btn_label: "View Report"

projects_intro:
  - excerpt: '## Featured Projects'

projects_row:
  - title: "Trader Platform"
    excerpt: "40–50M+ OHLCV 시계열 데이터를 처리하는 트레이딩 분석 플랫폼"
    url: "/projects/trader/"
    btn_label: "View Project"

  - title: "SIC Club Portal"
    excerpt: "팀 리딩 기반 웹 서비스 프로젝트 · CI/CD 및 운영 자동화 경험"
    url: "/projects/sic-portal/"
    btn_label: "View Project"

---

{% include feature_row id="intro" type="center" %}
{% include feature_row id="highlights_row" %}
{% include feature_row id="projects_intro" type="center" %}
{% include feature_row id="projects_row" %}