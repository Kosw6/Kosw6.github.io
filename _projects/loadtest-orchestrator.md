---
title: "부하·장애 검증 시나리오 오케스트레이터"
layout: single
classes: wide
excerpt: "로컬 UI와 실행 엔진을 통합하고 Terraform, k6, AWS SSM, WebSocket, Redis 장애 주입을 하나의 검증 시나리오로 자동화하는 Go 기반 오케스트레이터"
tags: [go, wails, k6, terraform, redis, websocket, ssm, loadtest]
---

> **부하 테스트 경험이 적어도 그래프에서 시나리오를 구성하고, 각자의 PC에서 인프라 준비부터 장애 주입과 복구 확인까지 반복 실행할 수 있도록 만든 도구입니다.**

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
    <strong>장애 시나리오와 정상 기준 비교</strong>
    <span>최종 상태 확인과 결과 수집으로 복구 확인</span>
  </div>
</div>

---

## 프로젝트 개요

### 해결하려던 문제

Trader-replay 프로젝트에서 Redis/Kafka 장애, WebSocket 부하, 복구 검증을 반복하면서 매번 같은 수동 작업을 되풀이하는 불편함을 느꼈습니다.

기존 방식은 Terraform output을 직접 확인하고, k6를 실행하고, Redis 장애를 수동 주입하고, health check와 로그를 다시 확인해야 했습니다.

한 번의 검증보다 같은 순서로 다시 실행하는 과정이 더 번거로웠고, 단계가 늘어날수록 실행 순서나 확인 항목이 누락될 가능성도 커졌습니다. 또한 부하 테스트 경험이 적은 사용자는 JMeter나 k6 같은 도구의 사용법과 설정 방식을 먼저 익혀야 테스트를 시작할 수 있었습니다.

### 선택한 방향

시나리오를 만드는 과정과 실제 부하를 발생시키는 과정을 분리했습니다. 사용자는 단일 설정 서버의 그래프 UI에서 단계, 의존성, 인프라와 실행 명령을 구성하고 ZIP 파일을 내려받습니다. 실제 테스트는 공용 부하 실행 서버가 아니라 각 사용자의 로컬 UI에서 ZIP 파일을 불러와 실행합니다.

공용 실행 서버에서 여러 사용자의 테스트가 동시에 실행되면 CPU, 메모리와 네트워크 대역폭을 공유하게 됩니다. 이 경우 대상 API의 변화가 아니라 부하 발생 서버의 자원 경합이 응답시간과 처리량에 섞일 수 있습니다. 각자의 PC에서 실행하도록 분리해 다른 사용자의 테스트가 현재 측정에 미치는 영향을 줄였습니다.

Load Test Orchestrator는 내려받은 ZIP 시나리오를 다음 순서로 실행합니다.

```text
scenario.zip
  -> Terraform apply
  -> output variable replacement
  -> health/auth check
  -> k6 WebSocket load
  -> AWS SSM 장애 주입
  -> 복구 상태 최종 확인
  -> result comparison
```

---

## 사용자 실행 흐름

시나리오 설정 서버는 테스트를 직접 실행하지 않고, 사용자가 실행할 ZIP 파일을 만드는 역할만 담당합니다. 실제 부하와 장애 검증은 로컬 앱이 수행합니다.

| 단계 | 사용자 경험 | 내부 처리 |
|---|---|---|
| 1. 시나리오 구성 | 설정 서버에 접속해 그래프에서 테스트 단계와 연결 순서를 구성 | 단계, 의존성, 인프라 설정과 CLI 명령을 시나리오로 저장 |
| 2. ZIP 다운로드 | 완성된 시나리오를 ZIP 파일로 내려받음 | YAML, k6 템플릿과 실행 설정을 하나의 파일로 패키징 |
| 3. 로컬 앱 불러오기 | 로컬 UI에서 내려받은 ZIP 파일을 선택 | 시나리오 구조와 실행 순서를 확인하고 임시 작업 공간에 준비 |
| 4. 테스트 실행 | 실행 버튼으로 전체 검증 흐름을 시작 | Go 엔진이 의존성에 따라 Terraform, k6와 AWS SSM 단계를 실행 |
| 5. 결과 확인 | 실행 로그와 단계별 성공 여부, 복구 결과를 확인 | 최종 상태를 수집하고 장애 시나리오와 정상 기준을 비교 |

```text
[시나리오 설정 서버]
그래프 기반 설정 -> ZIP 다운로드
                         |
                         v
[사용자 PC]
로컬 UI -> ZIP 불러오기 -> 인프라 준비 -> 부하 실행 -> 장애 주입 -> 복구 확인
```

로컬 실행은 공용 실행 서버에서 발생할 수 있는 테스트 간 자원 경합을 피하기 위한 선택입니다. 다만 사용자마다 PC 사양과 네트워크 환경이 다르므로 서로 다른 PC의 절대 수치를 직접 비교하지는 않습니다. 같은 실행 환경에서 개선 전후 또는 장애 시나리오와 정상 기준을 반복 비교하는 용도로 사용합니다.

---

## 기술 구성

| 영역 | 기술 |
|---|---|
| Scenario builder | React 기반 그래프 UI, YAML/ZIP 생성 |
| Local runner | Go, Wails, React UI |
| Load test | k6, WebSocket |
| Infra | Terraform, AWS EC2 |
| Fault injection | AWS SSM command |
| Target system | Redis Pub/Sub, Spring Boot backend |
| Report | Markdown/HTML result collection |

---

## UI와 실행 엔진 통합

### 통합 전

초기에는 로컬 UI와 실행 엔진을 별도 프로젝트처럼 다뤘지만, 현재는 Load Test Orchestrator 안에 통합했습니다.

### 통합 후

```text
loadtest-orchestrator/
  -> frontend/       # Wails 기반 로컬 UI
  -> app.go          # UI와 Go 엔진 연결
  -> engine/         # scenario 실행, k6, auth, final_check, terraform, chaos
  -> templates/      # k6 script templates
```

따라서 현재 포트폴리오에서는 `LoadTest Desktop`을 별도 프로젝트로 분리하지 않고, **Load Test Orchestrator = 로컬 UI + 실행 엔진 + 장애 검증 자동화**로 정리합니다.

시나리오 설정 서버는 실행 엔진과 분리해 그래프 편집과 ZIP 생성에 집중하고, 로컬 UI는 실행 상태와 로그를 보여주면서 Go 엔진을 제어합니다.

---

## Redis 장애 검증

### 검증 조건

가장 최근 검증에서는 Redis 장애 주입 시나리오(v2)와 Redis 장애가 없는 정상 기준(v3)을 비교했습니다.

### 검증 결과

| 항목 | v2 Redis 장애 | v3 정상 기준 |
|---|---:|---:|
| sent | 약 26,988 | 26,990 |
| received | 약 34,020 | 89,564 |
| errors | 0 | 0 |

### 결과 해석

- WebSocket 연결 자체는 Redis 장애 중에도 유지되었습니다.
- 다만 Redis Pub/Sub fanout 경로가 끊기면서 received 메시지 수가 감소했습니다.
- 정상 기준에서는 장애 시나리오 대비 약 2.6배 많은 메시지를 수신했습니다.

[Redis 장애 검증 리포트](/reports/loadtest-orchestrator-redis-fault-validation/)

---

## 반복 검증을 자동화한 이유

이 프로젝트의 목적은 JMeter나 k6 같은 부하 테스트 도구를 대체하는 것이 아닙니다. 실제 요청 생성과 측정은 기존 도구가 담당하고, 오케스트레이터는 이를 더 쉽게 설정해 여러 API를 정해진 순서로 호출하고 부하 실행, 장애 주입과 복구 확인을 하나의 시나리오로 연결합니다.

목표는 부하 테스트 경험이 적은 사용자도 전체 흐름을 구성할 수 있게 하고, 백엔드 개발자가 다음 과정을 반복 가능한 조건으로 검증하도록 돕는 것입니다.

- 인프라를 같은 조건으로 생성한다.
- health/auth를 먼저 검증한다.
- 부하 중 장애를 주입한다.
- 장애 주입 명령이 실제 성공했는지 확인한다.
- 복구 후 상태를 최종 확인 단계에서 점검한다.
- 장애 시나리오와 정상 기준을 비교한다.

즉, 부하 테스트를 "한 번 실행한 결과"가 아니라 **운영 검증 시나리오**로 다루는 도구입니다.

---

## 소스 코드

- [loadtest-orchestrator](https://github.com/Kosw6/loadtest-orchestrator) — 로컬 UI와 Go 실행 엔진
- [loadtest-converter](https://github.com/Kosw6/loadtest-converter) — 그래프 기반 시나리오 설정과 ZIP 생성
