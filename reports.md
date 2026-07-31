---
layout: single
title: "문제 해결 기록"
permalink: /reports/
classes:
  - wide
  - portfolio-index
  - portfolio-index--wide
---

문제가 발생한 조건, 대안을 선택한 이유와 검증 결과를 정리했습니다. 각 문서에는 측정 수치와 실행 기록도 함께 남겼습니다.

## 데이터 처리와 운영

<div class="report-grid">
  <a class="report-card report-card--featured" href="/reports/trader-data-platform/">
    <span class="report-card__eyebrow">데이터 파이프라인</span>
    <h3>중단 후에도 처리 위치를 확인하는 구조</h3>
    <p>서로 다른 원천 데이터를 먼저 보관하고 수집과 변환을 분리해, 중단 후에도 재처리 범위를 확인할 수 있게 했습니다.</p>
    <span class="report-card__result">ETL 재개: lag 2 → 0 / 별도 ASG 검증: 0 → 1 → 0</span>
  </a>
  <a class="report-card" href="/reports/realtime-degrade-overview/">
    <span class="report-card__eyebrow">메시지 시스템 장애</span>
    <h3>Redis, Kafka 장애 대응</h3>
    <p>메시지 시스템이 중단돼도 요청과 변경 내용을 보관하고, 복구 후 다시 처리하는 흐름을 검증했습니다.</p>
    <span class="report-card__result">Redis 검증: HTTP 5xx 0건 / Kafka 5분 38초 장애 검증: 유실, 중복 0건</span>
  </a>
  <a class="report-card" href="/reports/auto-recovery-scaleout/">
    <span class="report-card__eyebrow">자동 복구</span>
    <h3>장애 감지에서 복구 작업까지</h3>
    <p>애플리케이션 중단에는 컨테이너 재시작을, 자원 부족에는 인스턴스 확장을 실행하도록 알람과 복구 작업을 연결했습니다.</p>
    <span class="report-card__result">App 재시작 / CPU 확장 / Gateway 약 4초 전환</span>
  </a>
  <a class="report-card" href="/reports/observability-system/">
    <span class="report-card__eyebrow">관측</span>
    <h3>장애가 발생한 요청과 서비스 추적</h3>
    <p>요청 식별자를 구조화 로그에 남기고 오류 수와 비율을 함께 확인해 장애 발생 지점과 요청 흐름을 찾았습니다.</p>
    <span class="report-card__result">관측 → 판단 → 복구</span>
  </a>
  <a class="report-card" href="/reports/slo-operating-capacity/">
    <span class="report-card__eyebrow">운영 사양</span>
    <h3>SLO 기반 운영 사양 산정</h3>
    <p>목표 응답시간 300ms를 기준으로 애플리케이션과 DB 자원 설정을 같은 부하에서 비교했습니다.</p>
    <span class="report-card__result">2vCPU, 4GB, Thread 30, Connection 8</span>
  </a>
  <a class="report-card" href="/reports/loadtest-orchestrator-redis-fault-validation/">
    <span class="report-card__eyebrow">검증 자동화</span>
    <h3>같은 장애 시나리오 반복 실행</h3>
    <p>인프라 준비부터 부하 실행, 장애 주입과 복구 확인까지 정해진 순서로 다시 실행할 수 있게 했습니다.</p>
    <span class="report-card__result">수동 컨테이너 조작 -> 오케스트레이터 시작 1회</span>
  </a>
</div>

## 성능 개선

<div class="report-grid">
  <a class="report-card report-card--featured" href="/reports/timescaledb-27x/">
    <span class="report-card__eyebrow">시계열 DB</span>
    <h3>90일 차트 조회 지연 개선</h3>
    <p>동일 인덱스 조건에서 일반 테이블과 하이퍼테이블을 비교하고, 조회 범위에 맞는 청크 구성을 검증했습니다.</p>
    <span class="report-card__result">p95 7,247ms → 235ms</span>
  </a>
  <a class="report-card" href="/reports/jpa-tuning/">
    <span class="report-card__eyebrow">JPA</span>
    <h3>조회 방식별 처리량 비교</h3>
    <p>네 가지 조회 구조를 같은 부하에서 비교한 뒤, 1만 자 원문과 DB 20자 프리뷰의 p95, GC, 객체 할당량을 검증했습니다.</p>
    <span class="report-card__result">동일 p95 구간 유지 가능 RPS 약 5배</span>
  </a>
  <a class="report-card" href="/reports/jfr-jmc-hotpath/">
    <span class="report-card__eyebrow">런타임 분석</span>
    <h3>반복되는 JWT 검증 제거</h3>
    <p>실행 중 객체 할당이 집중되는 경로를 추적해 같은 요청에서 반복되던 검증을 제거했습니다.</p>
    <span class="report-card__result">Old GC 총 시간 약 36% 감소</span>
  </a>
</div>

## 실시간 처리와 복구

<div class="report-grid">
  <a class="report-card report-card--featured" href="/reports/websocket-group-canvas/">
    <span class="report-card__eyebrow">실시간 전송</span>
    <h3>동시 편집 메시지 전송 개선</h3>
    <p>세션별 전송을 직렬화해 쓰기 충돌을 제거한 뒤, 반복 전송을 줄이고 최신 상태만 전달하도록 바꿨습니다.</p>
    <span class="report-card__result">200ms 이내 수신 0.38% → 99.97%</span>
  </a>
  <a class="report-card" href="/reports/websocket-poc1-sharding/">
    <span class="report-card__eyebrow">서버 분산 검증</span>
    <h3>그룹별 연결 분산</h3>
    <p>그룹별 연결을 두 인스턴스에 나눠 한 서버에 집중되던 메시지 전송 부하를 분산했습니다.</p>
    <span class="report-card__result">서버당 전송 시도: 159K → 약 79K씩 분산</span>
  </a>
  <a class="report-card" href="/reports/websocket-poc2-conflict/">
    <span class="report-card__eyebrow">편집 충돌 검증</span>
    <h3>장애 중 작성한 내용의 병합 판단</h3>
    <p>사용자가 작성한 내용과 서버 변경 내용을 필드별로 비교해 자동 병합 가능 여부를 판단했습니다.</p>
    <span class="report-card__result">안전, 자동 병합, 충돌로 구분</span>
  </a>
  <a class="report-card" href="/reports/websocket-poc3-failback/">
    <span class="report-card__eyebrow">서버 복귀 검증</span>
    <h3>누락된 변경을 처리한 뒤 서버 복귀</h3>
    <p>복구된 서버가 장애 중 누락된 변경을 모두 처리한 뒤 다시 요청을 받도록 전환 순서를 정했습니다.</p>
    <span class="report-card__result">Kafka 3건을 목표 offset까지 처리 후 서버 전환</span>
  </a>
</div>

## 읽는 순서

처음 방문했다면 [Trader 프로젝트](/projects/trader/)에서 서비스 목적과 전체 구성을 확인한 뒤, 관심 있는 문제 해결 기록을 살펴보세요. 각 문서 하단에는 전체 실험 조건과 로그를 정리한 원문을 연결했습니다.
