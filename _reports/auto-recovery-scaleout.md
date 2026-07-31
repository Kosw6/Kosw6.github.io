---
title: "Auto Recovery & Scale-out - Grafana Alert, Lambda, SSM/ASG"
layout: single
permalink: /reports/auto-recovery-scaleout/
classes: wide
excerpt: "Grafana Alert에서 Lambda를 거쳐 SSM 복구와 ASG scale-out을 실행하고 신규 인스턴스의 서비스 합류까지 검증한 운영 자동화"
tags: [aws, grafana, lambda, ssm, autoscaling, recovery]
---

> **핵심 질문**: 장애를 감지하는 데서 끝나지 않고, 복구 명령 실행과 신규 인스턴스 합류까지 자동화할 수 있는가?
>
> 원문: [auto-recovery-scaleout-verification.md](https://github.com/Kosw6/engineering-notes/blob/main/reports/auto-recovery-scaleout-verification.md)
>
> 라우팅 검증: [Rendezvous Hashing 기반 Failover / Failback](https://github.com/Kosw6/engineering-notes/blob/main/reports/GroupController/poc4-rendezvous-failover-routing.md)
<br>

<nav class="page-quick-nav" aria-label="핵심 섹션 바로가기">
  <strong>빠르게 보기</strong>
  <a href="#summary">결과 요약</a>
  <a href="#app-down-자동-복구">App 재시작</a>
  <a href="#high-cpu-scale-out">ASG 확장</a>
  <a href="#gateway-failover--failback">Gateway 전환</a>
</nav>

<div class="proof-strip">
  <div class="proof-item">
    <span class="proof-item__label">ROUTING</span>
    <strong>Rendezvous Hashing</strong>
    <span>groupId별 안정적 후보 순위와 capacity 예약</span>
  </div>
  <div class="proof-item">
    <span class="proof-item__label">FAILOVER</span>
    <strong>close 1006 -> 전환 약 4초</strong>
    <span>health poller 1 cycle 내 A에서 B로 이동</span>
  </div>
  <div class="proof-item">
    <span class="proof-item__label">FAILBACK</span>
    <strong>신규 연결부터 A로 복귀</strong>
    <span>기존 세션 강제 이동 없음</span>
  </div>
</div>

---

## Summary

| 장애 유형 | 감지 | 자동 조치 | 검증 |
|---|---|---|---|
| App Down | Grafana alert | Lambda -> SSM -> docker compose restart | Prometheus scrape UP 복귀 |
| High CPU | Grafana alert | Lambda -> ASG desired capacity 1 -> 2 | 신규 EC2 생성 및 scrape 합류 |
| Gateway routing | health polling | healthy/draining 상태 기반 라우팅 | 장애 인스턴스 회피 및 failback |

> **검증 범위**: High CPU는 자동화 경로를 확인하기 위해 낮은 테스트 임계치(`process_cpu_usage > 0.002`)를 사용했습니다. 운영 임계치와 cooldown, 반복 재시작 실패 후 인스턴스 교체, scale-in 정책은 별도로 정해야 하며 이 문서의 완료 결과에 포함하지 않습니다.

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

라우팅은 다음 순서로 결정했다.

```text
Eureka instance 목록
-> HEALTHY 후보만 선택
-> Rendezvous Hashing으로 groupId별 후보 순위 계산
-> Redis Lua로 active + reserved < capacity 원자적 확인
-> 예약 실패 시 다음 순위 후보 선택
```

| 검증 시점 | 결과 |
|---|---|
| 초기 | HOLD와 신규 PROBE 모두 hash 1순위 A에 연결 |
| t=19s | A 장애, 기존 세션 close 1006 감지 |
| t=23s | health poller가 A를 제외하고 B로 failover |
| t=45s | 장애 중 신규 연결도 B로 라우팅 |
| t=90s | A 복구 후 신규 연결부터 다시 A로 복귀 |
| t=135s | 신규 연결은 A 유지, 기존 B 세션은 그대로 유지 |

Failover는 health poller 1 cycle 이내인 약 4초에 완료됐다. 인스턴스 추가·제거 시 모든 그룹을 다시 배치하지 않고 영향받는 그룹만 다음 hash 후보로 이동하도록 Rendezvous Hashing을 선택했다.

기존 세션은 강제로 failback하지 않았다. 복구 노드로 기존 WebSocket을 옮기면 reconnect와 room rejoin 비용이 발생하므로, 신규 연결부터 원래 hash 결과로 자연스럽게 복귀시키는 방향을 선택했다.

---

## 개선 과정에서 배운 점

장애를 감지해 알람을 보내는 것만으로 운영 자동화가 끝난다고 생각했다. 검증하면서 알람 이후 실제 대상에서 복구 명령이 실행되고, 서비스 상태가 다시 정상으로 확인돼야 대응이 끝난다는 점을 알게 됐다.

또한 App Down과 High CPU에는 같은 조치를 적용할 수 없었다. 전자는 컨테이너 재시작으로 복구하고, 후자는 용량을 늘린 뒤 신규 인스턴스가 모니터링과 서비스 경로에 합류하는지 확인하도록 완료 기준을 분리했다.
