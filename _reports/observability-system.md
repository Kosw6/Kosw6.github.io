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

> **시리즈**: 장애 대응과 운영 검증
> <br>**1. Observability System**
> <br>[2. SLO 기반 인프라 사양 검증](/reports/slo-operating-capacity/)
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

서비스를 운영하려면 다음 질문에 답할 수 있어야 한다.

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

로그 수집 도구를 선택할 때 기능 목록만 비교하지 않고, 현재 인프라에서 감당할 수 있는 운영 비용을 함께 확인했다.

### 비교 방법

ELK와 Loki를 동일한 인프라에 구성하고 같은 형식의 로그를 수집했다.
k6로 50, 150, 500 RPS의 트래픽을 발생시키며 CPU, 메모리, Disk Write 변화를 비교했다.

| 비교 조건 | ELK | Loki | 확인한 차이 |
|---|---|---|---|
| 유휴 상태 | CPU와 메모리 기본 사용량이 높음 | 상대적으로 낮음 | 로그가 적을 때도 발생하는 운영 비용 차이 |
| 150 RPS | CPU 사용량이 크게 변동 | CPU와 메모리가 비교적 안정적 | 본문 색인 과정의 리소스 비용 |
| 500 RPS | CPU와 메모리 사용량이 빠르게 증가 | 상대적으로 안정적 | 트래픽 증가 시 필요한 인프라 여유 |
| Disk Write | 색인과 분석 후 저장 | 유입량에 따라 선형 증가 | 저장 방식에 따른 비용의 위치 |

Loki는 로그 본문을 전문 색인하지 않고 압축된 chunk에 append하는 방식이어서 트래픽이 증가할수록 Disk Write가 선형적으로 늘었다.
반면 Elasticsearch는 저장 전에 inverted index 생성과 분석을 수행하므로 CPU와 메모리 사용량이 더 크게 증가했다.

### 선택 기준과 tradeoff

| 비교 항목 | ELK | Loki |
|---|---|---|
| CPU, 메모리 사용량 | 상대적으로 높음 | 상대적으로 낮음 |
| 디스크 쓰기 | 색인 처리 이후 증가 | 로그 유입량에 따라 선형 증가 |
| 검색 기능 | 전문 검색과 복잡한 분석에 강함 | 라벨과 필터 중심으로 제한적 |
| 운영 복잡도 | 상대적으로 높음 | 비교적 단순함 |
| Grafana 연동 | 별도 구성이 필요함 | 로그, 메트릭, 알람을 한 화면에서 연결하기 쉬움 |

이 시스템에서 로그의 주된 목적은 장기 데이터 분석보다 **실시간 장애 감지, traceId 기반 요청 추적, 알람 이후 원인 확인**이었다.
따라서 검색 기능보다 CPU와 메모리 사용량, Grafana 연동, 운영 복잡도를 우선해 Loki, Promtail, Grafana 조합을 선택했다.

다만 로그 유입량에 따른 Disk Write 증가와 제한적인 전문 검색은 Loki의 tradeoff로 남겼다.
장기 로그 분석이나 복잡한 검색이 핵심 요구사항이 된다면 ELK가 더 적합하다고 판단했다.

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

## 개선 과정에서 배운 점

이번 개선을 통해 장애 대응에서 중요한 것은 로그를 많이 남기는 것이 아니라, 하나의 요청이 어느 서비스와 이벤트를 거쳐 실패했는지 추적할 수 있는 기준을 만드는 것임을 배웠다.

또한 알람 발생 여부만으로는 복구가 끝났다고 판단할 수 없었다. 이후 장애 검증에서는 로그와 지표를 함께 확인하고, 복구 작업 이후 서비스 상태가 정상으로 돌아왔는지까지 확인하는 것을 완료 기준으로 삼았다.
