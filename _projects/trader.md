---
title: "Trader Platform"
layout: single
sidebar:
  nav: "main"
header:
  image: /assets/images/trader-demo.png
excerpt: "40–50M+ OHLCV, TimescaleDB, WebGL Candles, JWT SSO, k6"
tags: [spring, timescaledb, redis, aws, k6]
---

## Overview

- **Stack**: Spring Boot, JPA, PostgreSQL/TimescaleDB, Redis, React, AWS, Docker
- **Scale**: 40–50M+ OHLCV across ~10K tickers
- **Perf**: k6 결과 — p90 57ms, 평균 18.6ms (캐시 없이)
- **SSO**: Google/Kakao/Naver JWT

## Architecture

- Hypertable & chunking, composite indexes, Redis optional cache
- CI/CD: GitHub Actions · Test coverage (JaCoCo ≥70%) · k6 load tests
- Observability: Prometheus/Grafana · Slack 알람

## Highlights

- WebGL 차트 + 노트/엣지 그래프(React Flow)
- 백테스트 기초 모듈(Sharpe/MDD) & 향후 PatchTST/LSTM 계획

## Link

- 부하테스트 결과

  | 단계 | 문제점                               | 해결요약                     | 링크                                                                              |
  | ---- | ------------------------------------ | ---------------------------- | --------------------------------------------------------------------------------- |
  | 1차  | 서버와 로컬의 매우 큰 성능 차이 존재 | 모니터링을 통한 DB 설정 변경 | [결과 보기](https://github.com/Kosw6/trader-backend/blob/master/k6/2025-10-17.md) |
