---
title: "Observability System - TraceId, Loki, Grafana Alert"
layout: single
permalink: /reports/observability-system/
toc: true
toc_sticky: true
classes: wide
excerpt: "TraceId 기반 구조화 로그와 Loki/Grafana 관측 체계를 구성해 장애 원인 추적과 error rate alert를 가능하게 만든 과정"
tags: [observability, logging, loki, grafana, prometheus, spring]
---

> **핵심 질문**: 장애가 발생했을 때 "어느 요청, 어느 도메인, 어느 이벤트에서 실패했는지"를 빠르게 찾을 수 있는가?
>
> 원문: [관측 체계 도입.md](https://github.com/Kosw6/engineering-notes/blob/main/reports/%EA%B4%80%EC%B8%A1%20%EC%B2%B4%EA%B3%84%20%EB%8F%84%EC%9E%85.md)
<br>

> **시리즈**: Reliability & Operations
> <br>**1. Observability System**
> <br>[2. SLO 기반 운영 사양 산정](/reports/slo-operating-capacity/)
> <br>[3. Realtime Degraded Mode](/reports/realtime-degrade-overview/)
> <br>[4. Auto Recovery & Scale-out](/reports/auto-recovery-scaleout/)
> <br>[5. Load Test Orchestrator](/reports/loadtest-orchestrator-redis-fault-validation/)


---

## Summary

| 항목 | 내용 |
|---|---|
| Problem | 일반 문자열 로그만으로는 장애 원인, 요청 흐름, 도메인별 실패 지점을 빠르게 추적하기 어려움 |
| Solution | TraceIdFilter, MDC, AOP 기반 구조화 로그와 Loki/Grafana 관측 체계 구성 |
| Stack | Spring Boot, Prometheus, Grafana, Loki, Promtail |
| Result | error count/rate dashboard와 alert 기반 운영 관측 체계 구성 |

---

## 문제

기능이 정상 동작하는 것만으로는 운영 안정성을 설명할 수 없다.

실제 운영에서는 다음 질문에 답할 수 있어야 한다.

- 특정 요청이 어느 API와 도메인을 거쳐 실패했는가?
- Redis, Kafka, DB 중 어떤 경로에서 지연 또는 오류가 발생했는가?
- 장애가 재현될 때 같은 traceId로 로그를 따라갈 수 있는가?
- 에러율이 임계치를 넘었을 때 알림을 받을 수 있는가?

이를 위해 로그를 단순 문자열이 아니라 검색과 집계가 가능한 필드 구조로 남기도록 설계했다.

---

## 설계

구조화 로그에는 다음 필드를 포함했다.

| 필드 | 목적 |
|---|---|
| `traceId` | 하나의 요청 흐름 추적 |
| `domain` | 장애가 발생한 업무 도메인 구분 |
| `instance` | 어느 서버에서 발생했는지 구분 |
| `event` | 관측 이벤트 이름 |
| `level` | INFO, WARN, ERROR 등 심각도 |
| `errorCode` | 장애 유형 필터링 |
| `elapsedMs` | 요청 또는 작업 소요 시간 |
| `uri` | HTTP 요청 경로 |

구현 방향은 다음과 같다.

```text
HTTP Request
  -> TraceIdFilter
  -> MDC에 traceId 저장
  -> ObservedLogAspect
  -> JSON/structured log
  -> Promtail
  -> Loki
  -> Grafana dashboard / alert
```

---

## ELK 대신 Loki를 선택한 이유

운영 PoC에서는 ELK와 Loki를 비교했고, 현재 단계에서는 Loki를 선택했다.

| 비교 항목 | ELK | Loki |
|---|---|---|
| 설치/운영 복잡도 | 높음 | 낮음 |
| 리소스 사용량 | 상대적으로 큼 | 상대적으로 작음 |
| Grafana 연동 | 가능 | 자연스러움 |
| 현재 목표 | 전문 검색/분석에 강함 | 로그-메트릭-알림 연결에 적합 |

목표가 대규모 검색 엔진 구축이 아니라 **장애 원인 추적과 운영 알림**이었기 때문에 Loki + Promtail + Grafana 조합을 선택했다.

---

## 결과

Grafana에서 다음 지표와 로그를 함께 볼 수 있게 구성했다.

- 인스턴스별 로그 수집 상태
- 도메인별 ERROR count
- error rate 기반 alert
- 요청별 traceId 검색
- API latency와 로그 이벤트 연결
- Prometheus resource metric과 로그 이벤트 비교

이 구조를 통해 장애 대응 문서에서 단순히 "장애가 발생했다"가 아니라, **어떤 이벤트와 지표가 함께 변했는지**를 근거로 설명할 수 있게 되었다.

---

## 포트폴리오 관점의 의미

이 문서는 단순 모니터링 도구 설치 기록이 아니다.

백엔드 시스템을 운영 환경에 가깝게 검증하려면 다음이 필요하다는 것을 보여준다.

- 장애를 재현할 수 있는 부하/시나리오
- 장애를 설명할 수 있는 로그 구조
- 장애를 감지할 수 있는 metric과 alert
- 복구 전후 상태를 비교할 수 있는 dashboard

즉, 기능 구현 이후의 단계인 **운영 가능한 백엔드 시스템 만들기**를 다룬 문서다.
