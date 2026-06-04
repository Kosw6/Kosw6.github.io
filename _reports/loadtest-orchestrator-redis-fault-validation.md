---
title: "Load Test Orchestrator - Redis Fault Validation"
layout: single
permalink: /reports/loadtest-orchestrator-redis-fault-validation/
toc: true
toc_sticky: true
classes: wide
excerpt: "Terraform apply, WebSocket load, Redis stop/start, SSM polling, recovery health check를 하나의 시나리오로 자동화한 장애 검증"
tags: [go, k6, terraform, redis, websocket, ssm, loadtest]
---

> **핵심 질문**: 부하 테스트 중 Redis 장애를 주입하고, 장애 효과와 복구 상태를 자동화된 시나리오 안에서 검증할 수 있는가?
>
> 원문: [loadtest-orchestrator-redis-fault-validation.md](https://github.com/Kosw6/engineering-notes/blob/main/reports/loadtest-orchestrator-redis-fault-validation.md)

---

## Summary

| 항목 | 결과 |
|---|---|
| Terraform apply | AWS 리소스 생성 및 output 변수 치환 |
| WebSocket load | 50 VU, 약 90초 |
| Redis fault | AWS SSM으로 stop/start 실행 |
| SSM validation | command-id 기반 Success polling |
| v2 fault scenario | sent 약 26,988 / received 약 34,020 / errors 0 |
| v3 baseline | sent 26,990 / received 89,564 / errors 0 |
| 차이 | baseline received가 약 2.6배 많음 |

---

## 검증 시나리오

기존 방식은 사람이 직접 Terraform output을 확인하고, k6를 실행하고, Redis 장애를 주입하고, health를 확인해야 했다.

오케스트레이터는 이 흐름을 시나리오로 묶었다.

```text
Terraform apply
  -> output 조회 및 변수 치환
  -> app1/app2 health check
  -> k6 WebSocket load
  -> AWS SSM Redis stop
  -> SSM command Success polling
  -> AWS SSM Redis start
  -> recovery health check
  -> result comparison
```

---

## v2: Redis 장애 주입

WebSocket 부하가 진행되는 동안 Redis stop/start를 실행했다.

| 항목 | 결과 |
|---|---|
| WebSocket sent | 약 26,988 |
| WebSocket received | 약 34,020 |
| WebSocket errors | 0 |
| Redis stop | SSM command Success |
| Redis start | SSM command Success |
| Recovery health | Redis 복구 후 503에서 200으로 회복 |

해석:

- WebSocket 연결 자체는 유지되었다.
- Redis Pub/Sub fanout이 끊기면서 수신 메시지 수가 감소했다.
- Redis stop/start가 실제로 실행됐는지 SSM command result로 확인했다.

---

## v3: 장애 없는 Baseline

동일한 WebSocket 부하를 Redis 장애 없이 실행했다.

```text
duration: 90.47s
open: 50 / close: 50 / errors: 0
sent: 26990 / received: 89564
sent/s: 298.33 / recv/s: 989.98
connect(ms) avg=291.8 p95=454.3 p99=462.5
```

---

## 비교

| 항목 | v2 Redis fault | v3 baseline | 해석 |
|---|---:|---:|---|
| sent | 약 26,988 | 26,990 | 거의 동일한 송신 부하 |
| received | 약 34,020 | 89,564 | baseline이 약 2.6배 많음 |
| errors | 0 | 0 | 연결 안정성은 양쪽 모두 유지 |

이번 결과는 Redis 장애가 WebSocket 연결을 즉시 끊는 형태가 아니라, **연결은 유지되지만 Pub/Sub fanout 수신량이 감소하는 형태**로 나타난다는 점을 보여준다.

---

## 오케스트레이터의 의미

이 검증은 단순 k6 실행 결과가 아니다.

오케스트레이터가 다음을 자동화했다.

- 인프라 생성
- Terraform output 변수 치환
- 사전 health check
- WebSocket 부하 실행
- Redis 장애 주입
- SSM command 성공 확인
- 복구 health check
- 장애군/baseline 비교

따라서 이 프로젝트는 부하 테스트 실행기가 아니라, 백엔드 운영 환경의 장애와 복구를 재현하는 **검증 워크플로우 도구**에 가깝다.
