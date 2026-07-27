---
title: "SLO 기반 운영 사양 산정 - p95 300ms 기준 인프라 결정"
layout: single
permalink: /reports/slo-operating-capacity/
classes: wide
excerpt: "p95 <= 300ms 기준으로 App/DB 2vCPU/4GB, Thread 30, Hikari 8 설정을 도출하고 context switch와 DB pending으로 병목을 해석한 운영 사양 산정"
tags: [slo, capacity-planning, k6, linux, spring, hikari]
---

> **핵심 질문**: 목표 응답 시간 p95 <= 300ms를 만족하려면 어느 정도의 App/DB 사양과 스레드/커넥션 설정이 필요한가?
>
> 원문: [SLO 기반 운영 사양 산정 실험](https://github.com/Kosw6/engineering-notes/blob/main/reports/SLO%20%EA%B8%B0%EB%B0%98%20%EC%9A%B4%EC%98%81%20%EC%82%AC%EC%96%91%20%EC%82%B0%EC%A0%95%20%EC%8B%A4%ED%97%98.md)
>
> 세부 실험: [Thread / Hikari 설정과 Context Switching 분석](https://github.com/Kosw6/engineering-notes/blob/main/reports/%EC%93%B0%EB%A0%88%EB%93%9C%EC%99%80%20%EC%BB%A4%EB%84%A5%EC%85%98%20%ED%92%80%20%EC%84%A4%EC%A0%95%EC%97%90%20%EB%94%B0%EB%A5%B8%20%EC%BB%A8%ED%85%8D%EC%8A%A4%ED%8A%B8%20%EC%8A%A4%EC%9C%84%EC%B9%AD%20%EB%B6%84%EC%84%9D%20%EB%B0%8F%20%EA%B2%B0%EC%A0%95.md)
<br>

<nav class="page-quick-nav" aria-label="핵심 섹션 바로가기">
  <strong>빠르게 보기</strong>
  <a href="#summary">결과 요약</a>
  <a href="#thread--hikari-비교">설정 비교</a>
  <a href="#결정">최종 결정</a>
  <a href="/reports/auto-recovery-scaleout/">다음: 자동 복구</a>
</nav>

<div class="proof-strip">
  <div class="proof-item">
    <span class="proof-item__label">BALANCED</span>
    <strong>Thread 30 / Hikari 8</strong>
    <span>2 vCPU 환경에서 가장 안정적인 구간</span>
  </div>
  <div class="proof-item">
    <span class="proof-item__label">UNDERPROVISIONED</span>
    <strong>Thread 2: p95 190ms+</strong>
    <span>nvcswch 약 276, saturation 진입</span>
  </div>
  <div class="proof-item">
    <span class="proof-item__label">OVERSIZED</span>
    <strong>Thread 60+ / Hikari 12+</strong>
    <span>처리량 이득 없이 scheduling 비용 증가</span>
  </div>
</div>

---

## Summary

| 항목 | 결정 |
|---|---|
| SLO | p95 <= 300ms |
| App | 2 vCPU / 4 GB |
| DB | 2 vCPU / 4 GB |
| Thread | 30 |
| Hikari pool | 8 |
| 주요 판단 근거 | Linux context switch, DB connection pending, k6 latency 분포 |

---

## 문제

부하 테스트 결과를 단순히 "버틴다 / 못 버틴다"로 판단하면 운영 사양을 설명하기 어렵다.

따라서 목표 SLO를 먼저 정하고, 그 기준을 만족하는 최소 운영 사양과 애플리케이션 설정을 찾는 방식으로 접근했다.

~~~text
SLO 정의
  -> k6 부하 테스트
  -> latency / pending / CPU 지표 확인
  -> Linux context switch 분석
  -> App/DB 사양 및 Thread/Hikari 설정 결정
~~~

---

## 분석 관점

운영 사양 산정에서 본 핵심 지표는 다음과 같다.

| 지표 | 해석 |
|---|---|
| p95 latency | 사용자 관점의 응답 시간 기준 |
| DB connection pending | 커넥션 풀이 병목인지 확인 |
| CPU 사용률 | 인스턴스 사양 부족 여부 확인 |
| Linux context switch | 스레드 수 증가가 실제 처리량 개선이 아니라 스케줄링 비용으로 이어지는지 확인 |
| error rate | SLO를 만족하더라도 실패율이 증가하는지 확인 |

---

## Thread / Hikari 비교

단순히 thread와 connection pool을 크게 잡지 않고, 같은 부하에서 부족·균형·과다 구간을 비교했다.

| 구간 | 관측 결과 | 해석 |
|---|---|---|
| Thread 2 / Hikari 2 | p95 190ms 이상, nvcswch 약 276 | runnable queue 경쟁과 saturation |
| Thread 30 / Hikari 8 | nvcswch와 cswch가 가장 안정적 | 현재 2 vCPU workload의 균형점 |
| Thread 60+ | 처리량 증가 없이 cswch와 latency 증가 | scheduling overhead |
| Hikari 4 | DB pending은 없지만 cswch 증가 | borrow/return contention |
| Hikari 12~16 | active thread와 cswch 증가 | pool 과다로 인한 scheduling 비용 |

당시 active DB connection은 1~2 수준이고 pending이 없었다. 따라서 pool 부족보다 CPU scheduling과 애플리케이션 thread 경합을 현재 병목으로 판단했다. 트래픽이나 DB 처리량이 바뀌면 이 설정도 다시 측정해야 하므로, `30/8`을 절대값이 아니라 해당 workload의 운영 기준으로 기록했다.

---

## 결정

테스트 결과 App/DB를 2 vCPU / 4 GB 구성으로 두고, Thread 30 / Hikari 8 조합을 선택했다.

이 조합은 단순히 더 큰 리소스를 투입한 결과가 아니라, p95 <= 300ms 기준에서 CPU, DB pending, context switch 비용을 함께 보고 결정한 운영 기준이다.

---

## 의미

이 실험의 목적은 최대 성능 수치를 과시하는 것이 아니라, 운영 환경에서 설명 가능한 기준을 만드는 것이었다.

- SLO를 먼저 정의했다.
- 부하 테스트로 지표를 수집했다.
- Linux/DB/App 레벨의 병목 신호를 함께 해석했다.
- 최종 설정을 수치와 근거로 남겼다.

이후 장애 대응과 자동 복구 검증은 이 운영 기준 위에서 진행했다.
