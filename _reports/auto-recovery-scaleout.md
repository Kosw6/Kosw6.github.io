---
title: "Auto Recovery & Scale-out - Grafana Alert, Lambda, SSM/ASG"
layout: single
permalink: /reports/auto-recovery-scaleout/
toc: true
toc_sticky: true
classes: wide
excerpt: "Grafana Alert에서 Lambda를 거쳐 SSM 복구와 ASG scale-out을 실행하고 신규 인스턴스의 서비스 합류까지 검증한 운영 자동화"
tags: [aws, grafana, lambda, ssm, autoscaling, recovery]
---

> **핵심 질문**: 장애를 감지하는 데서 끝나지 않고, 복구 명령 실행과 신규 인스턴스 합류까지 자동화할 수 있는가?
>
> 원문: [auto-recovery-scaleout-verification.md](https://github.com/Kosw6/engineering-notes/blob/main/reports/auto-recovery-scaleout-verification.md)

---

## Summary

| 장애 유형 | 감지 | 자동 조치 | 검증 |
|---|---|---|---|
| App Down | Grafana alert | Lambda -> SSM -> docker compose restart | Prometheus scrape UP 복귀 |
| High CPU | Grafana alert | Lambda -> ASG desired capacity 1 -> 2 | 신규 EC2 생성 및 scrape 합류 |
| Gateway routing | health polling | healthy/draining 상태 기반 라우팅 | 장애 인스턴스 회피 및 failback |

---

## 문제

관측 체계를 구성해도 알림만 있고 복구가 수동이면 MTTR은 여전히 사람의 반응 속도에 의존한다.

따라서 다음 흐름을 검증했다.

```text
Metric anomaly
  -> Grafana Alert
  -> Webhook
  -> Lambda
  -> SSM or ASG
  -> Recovery / Scale-out
  -> Prometheus scrape 확인
```

---

## App Down 자동 복구

App Down은 Prometheus scrape 실패를 기반으로 감지했다.

중요한 점은 일시적인 scrape 흔들림을 장애로 오판하지 않도록 prior normal condition과 pending period를 둔 것이다.

```text
app down
  -> Grafana alert pending
  -> Lambda webhook
  -> AWS SSM command
  -> cd /data/app && docker compose restart app
  -> app scrape UP
  -> alert Normal
```

이 검증은 단순히 Lambda가 호출되는지 보는 것이 아니라, **복구 후 Prometheus가 다시 app을 scrape할 수 있는지**까지 확인했다.

---

## High CPU Scale-out

CPU 부하가 높아지는 경우에는 같은 인스턴스를 재시작하는 것보다 capacity를 늘리는 것이 적절하다.

검증 흐름:

```text
High CPU alert
  -> Lambda
  -> ASG desired capacity 1 -> 2
  -> EC2 launch
  -> app container start
  -> Prometheus scrape target 추가
  -> Redis/Kafka/Gateway 경로 합류
```

중복 scale-out을 막기 위해 max capacity 상태도 확인했다.

---

## Gateway Failover / Failback

Gateway는 Eureka와 `/internal/health`를 함께 사용해 backend instance 상태를 확인했다.

상태 모델:

| 상태 | 의미 |
|---|---|
| HEALTHY | 라우팅 가능 |
| DRAINING | 신규 연결 차단, 기존 연결 정리 |
| DOWN | 라우팅 제외 |

Failover는 health poller 주기에 맞춰 약 1 cycle 내 전환되도록 설계했다.

기존 세션은 강제로 failback하지 않고 reconnect churn을 줄이는 방향을 선택했다.

---

## 결론

이 검증은 "알림을 받았다"에서 끝나지 않는다.

운영 자동화의 기준을 다음처럼 잡았다.

- 장애 유형별로 soft recovery와 scale-out을 구분한다.
- 복구 명령이 실제 대상 인스턴스에서 실행되는지 확인한다.
- 신규 인스턴스가 Prometheus, Redis, Kafka, Gateway 경로에 합류하는지 확인한다.
- Gateway 라우팅이 health 상태에 따라 장애 인스턴스를 회피하는지 확인한다.

따라서 이 문서는 백엔드 시스템을 운영 가능한 상태로 만들기 위한 **자동 복구와 scale-out 검증 기록**이다.
