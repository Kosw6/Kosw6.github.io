---
title: "부하·장애 검증 시나리오 오케스트레이터"
layout: single
classes: wide
excerpt: "로컬 UI와 실행 엔진을 통합하고 Terraform, k6, AWS SSM, WebSocket, Redis 장애 주입을 하나의 검증 시나리오로 자동화하는 Go 기반 오케스트레이터"
tags: [go, wails, k6, terraform, redis, websocket, ssm, loadtest]
---

> **부하 테스트를 실행하는 것에서 끝나지 않고, 인프라 생성, 장애 주입, 복구 확인, baseline 비교까지 하나의 검증 워크플로우로 묶는 도구입니다.**

<div class="proof-strip">
  <div class="proof-item">
    <span class="proof-item__label">SCENARIO</span>
    <strong>단계·의존성·인프라 정의</strong>
    <span>UI에서 YAML/ZIP 시나리오 구성</span>
  </div>
  <div class="proof-item">
    <span class="proof-item__label">EXECUTION</span>
    <strong>Terraform → k6 → SSM</strong>
    <span>준비, 부하, 장애, 복구를 순서대로 실행</span>
  </div>
  <div class="proof-item">
    <span class="proof-item__label">VALIDATION</span>
    <strong>fault와 baseline 비교</strong>
    <span>final check와 결과 수집으로 복구 확인</span>
  </div>
</div>

---

## Project Overview

Trader-replay 프로젝트에서 Redis/Kafka 장애, WebSocket 부하, 복구 검증을 반복하면서 수동 검증의 한계를 느꼈습니다.

기존 방식은 Terraform output을 직접 확인하고, k6를 실행하고, Redis 장애를 수동 주입하고, health check와 로그를 다시 확인해야 했습니다.

Load Test Orchestrator는 이 과정을 ZIP 시나리오 기반으로 실행합니다.

```text
scenario.zip
  -> Terraform apply
  -> output variable replacement
  -> health/auth check
  -> k6 WebSocket load
  -> AWS SSM fault injection
  -> recovery final check
  -> result comparison
```

---

## Stack

| 영역 | 기술 |
|---|---|
| App | Go, Wails, React UI |
| Load test | k6, WebSocket |
| Infra | Terraform, AWS EC2 |
| Fault injection | AWS SSM command |
| Target system | Redis Pub/Sub, Spring Boot backend |
| Report | Markdown/HTML result collection |

---

## UI + Engine 통합

초기에는 로컬 UI와 실행 엔진을 별도 프로젝트처럼 다뤘지만, 현재는 Load Test Orchestrator 안에 통합했습니다.

```text
loadtest-orchestrator/
  -> frontend/       # Wails 기반 로컬 UI
  -> app.go          # UI와 Go 엔진 연결
  -> engine/         # scenario 실행, k6, auth, final_check, terraform, chaos
  -> templates/      # k6 script templates
```

따라서 현재 포트폴리오에서는 `LoadTest Desktop`을 별도 프로젝트로 분리하지 않고, **Load Test Orchestrator = 로컬 UI + 실행 엔진 + 장애 검증 자동화**로 정리합니다.

---

## Redis Fault Validation

가장 최근 검증에서는 Redis 장애 주입 시나리오(v2)와 Redis 장애 없는 baseline(v3)을 비교했습니다.

| 항목 | v2 Redis fault | v3 baseline |
|---|---:|---:|
| sent | 약 26,988 | 26,990 |
| received | 약 34,020 | 89,564 |
| errors | 0 | 0 |

해석:

- WebSocket 연결 자체는 Redis 장애 중에도 유지되었습니다.
- 다만 Redis Pub/Sub fanout 경로가 끊기면서 received 메시지 수가 감소했습니다.
- 정상 baseline은 장애군 대비 약 2.6배 많은 메시지를 수신했습니다.

[Redis fault validation report](/reports/loadtest-orchestrator-redis-fault-validation/)

---

## Why It Matters

이 프로젝트의 목적은 k6를 대체하는 것이 아닙니다.

목표는 백엔드 개발자가 다음 흐름을 반복 가능하게 검증하도록 돕는 것입니다.

- 인프라를 같은 조건으로 생성한다.
- health/auth를 먼저 검증한다.
- 부하 중 장애를 주입한다.
- 장애 주입 명령이 실제 성공했는지 확인한다.
- 복구 후 상태를 final check로 확인한다.
- 장애군과 정상 baseline을 비교한다.

즉, 부하 테스트를 "한 번 실행한 결과"가 아니라 **운영 검증 시나리오**로 다루는 도구입니다.

---

## GitHub

- [loadtest-orchestrator](https://github.com/Kosw6/loadtest-orchestrator)
- [loadtest-converter](https://github.com/Kosw6/loadtest-converter)
