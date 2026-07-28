---
layout: single
title: "Engineering Reports"
permalink: /reports/
classes:
  - wide
  - portfolio-index
  - portfolio-index--wide
---

## Engineering Reports

문제를 지표와 상태로 드러내고, 원인을 좁힌 뒤, 같은 조건에서 개선과 복구 결과를 검증한 기록입니다. 프로젝트 소개보다 깊은 설계 근거, 실패 과정, 수치와 로그를 확인할 수 있습니다.

## Data Platform & Operations

<div class="report-grid">
  <a class="report-card report-card--featured" href="/reports/trader-data-platform/">
    <span class="report-card__eyebrow">DATA PLATFORM</span>
    <h3>Trader Data Platform</h3>
    <p>KIS, BLS, SEC raw 보존부터 Kafka outbox, ETL lineage, 장애 복구와 worker control plane까지 연결했습니다.</p>
    <span class="report-card__result">raw lag 2 -> 0 · AWS ASG 0 -> 1 -> 0</span>
  </a>
  <a class="report-card" href="/reports/realtime-degrade-overview/">
    <span class="report-card__eyebrow">DEGRADED MODE</span>
    <h3>Redis, Kafka 장애 대응</h3>
    <p>저지연 hot path와 durable replay 경로를 분리하고 DB fallback과 outbox 복구를 검증했습니다.</p>
    <span class="report-card__result">Redis lock 1/49 · gRPC p95 14.5ms · loss 0</span>
  </a>
  <a class="report-card" href="/reports/auto-recovery-scaleout/">
    <span class="report-card__eyebrow">AUTO RECOVERY</span>
    <h3>Alert에서 복구 액션까지</h3>
    <p>Grafana Alert, Lambda, SSM, ASG를 연결해 컨테이너 재시작과 신규 인스턴스 합류를 검증했습니다.</p>
    <span class="report-card__result">App restart · 4초 failover · ASG scale-out</span>
  </a>
  <a class="report-card" href="/reports/observability-system/">
    <span class="report-card__eyebrow">OBSERVABILITY</span>
    <h3>TraceId, Loki, Grafana</h3>
    <p>구조화 로그와 error count/rate alert로 장애 발생 지점과 요청 흐름을 추적했습니다.</p>
    <span class="report-card__result">관측 -> 판단 -> 복구</span>
  </a>
  <a class="report-card" href="/reports/slo-operating-capacity/">
    <span class="report-card__eyebrow">CAPACITY</span>
    <h3>SLO 기반 운영 사양 산정</h3>
    <p>p95 300ms를 기준으로 App/DB 사양과 thread, pool 설정을 부하 테스트로 결정했습니다.</p>
    <span class="report-card__result">2vCPU / 4GB · Thread 30 · Hikari 8</span>
  </a>
  <a class="report-card" href="/reports/loadtest-orchestrator-redis-fault-validation/">
    <span class="report-card__eyebrow">VALIDATION</span>
    <h3>Load Test Orchestrator</h3>
    <p>Terraform, k6, AWS SSM을 묶어 환경 준비, 장애 주입, 복구 확인을 반복 실행했습니다.</p>
    <span class="report-card__result">WebSocket 34k -> 89k received</span>
  </a>
</div>

## Performance Engineering

<div class="report-grid">
  <a class="report-card report-card--featured" href="/reports/timescaledb-27x/">
    <span class="report-card__eyebrow">TIMESCALEDB</span>
    <h3>시계열 조회 p95 SLO 달성</h3>
    <p>인덱스, 하이퍼테이블, chunk 구성을 단계별로 분리해 같은 부하에서 비교했습니다.</p>
    <span class="report-card__result">p95 7,247ms -> 235ms</span>
  </a>
  <a class="report-card" href="/reports/jpa-tuning/">
    <span class="report-card__eyebrow">JPA</span>
    <h3>Fetch 전략 비교</h3>
    <p>Lazy, Fetch Join, Projection, DB preview를 payload와 객체 생성 비용까지 포함해 비교했습니다.</p>
    <span class="report-card__result">붕괴 RPS 5배 차이</span>
  </a>
  <a class="report-card" href="/reports/jfr-jmc-hotpath/">
    <span class="report-card__eyebrow">RUNTIME</span>
    <h3>JFR/JMC Hot Path 분석</h3>
    <p>JWT 중복 검증을 allocation hotspot으로 추적하고 필터 경로를 재구성했습니다.</p>
    <span class="report-card__result">Old GC 36% 감소</span>
  </a>
</div>

## Realtime Architecture

<div class="report-grid">
  <a class="report-card report-card--featured" href="/reports/websocket-group-canvas/">
    <span class="report-card__eyebrow">WEBSOCKET</span>
    <h3>브로드캐스트 재설계</h3>
    <p>동시 쓰기 충돌을 Dirty Flag 기반 최신값 전송 구조로 바꿨습니다.</p>
    <span class="report-card__result">&lt;=200ms 수신율 0.38% -> 99.97%</span>
  </a>
  <a class="report-card" href="/reports/websocket-poc1-sharding/">
    <span class="report-card__eyebrow">POC 1</span>
    <h3>그룹 샤딩</h3>
    <p>groupId 기반 라우팅으로 fanout 부하를 두 인스턴스에 분리했습니다.</p>
    <span class="report-card__result">159K -> 79K + 79K</span>
  </a>
  <a class="report-card" href="/reports/websocket-poc2-conflict/">
    <span class="report-card__eyebrow">POC 2</span>
    <h3>Fallback 충돌 제어</h3>
    <p>Redis Draft와 필드 단위 변경 비교로 AUTO_MERGE와 CONFLICT를 판별했습니다.</p>
    <span class="report-card__result">SAFE · AUTO_MERGE · CONFLICT</span>
  </a>
  <a class="report-card" href="/reports/websocket-poc3-failback/">
    <span class="report-card__eyebrow">POC 3</span>
    <h3>Kafka Replay Failback</h3>
    <p>Catch-up consumer, offset replay, drain 순서로 복구 서버를 운영 경로에 재편입했습니다.</p>
    <span class="report-card__result">이벤트 유실 없는 서버 전환</span>
  </a>
</div>

## 읽는 순서

처음 방문했다면 [Trader Platform 프로젝트](/projects/trader/)에서 서비스 목적과 전체 성장 흐름을 확인한 뒤, 관심 있는 문제의 Engineering Report로 들어가는 것을 권장합니다. 각 리포트 하단에는 전체 실험과 검증 로그가 담긴 engineering-notes 원문을 연결했습니다.
