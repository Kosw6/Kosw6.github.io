---
title: "SLO 기반 운영 사양 산정 - p95 300ms 기준 인프라 결정"
layout: single
permalink: /reports/slo-operating-capacity/
classes: wide
excerpt: "p95 <= 300ms 기준으로 App/DB 2vCPU/4GB, Thread 30, Hikari 8 설정을 도출하고 기본 부하와 별도 Stress 부하의 병목 신호를 구분해 해석한 운영 사양 산정"
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
    <strong>Thread 30 / Hikari 8 @ 150 RPS</strong>
    <span>기본 비교에서 가장 안정적인 구간</span>
  </div>
  <div class="proof-item">
    <span class="proof-item__label">UNDERPROVISIONED</span>
    <strong>Thread 2 / Hikari 2 @ 300 RPS</strong>
    <span>p95 190ms+, nvcswch 약 276</span>
  </div>
  <div class="proof-item">
    <span class="proof-item__label">OVERSIZED</span>
    <strong>Thread 60+ / Hikari 12+ @ 150 RPS</strong>
    <span>처리량 이득 없이 context switch 증가</span>
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

기본 비교와 Stress 검증은 부하 조건이 다르다. 기본 150 RPS에서는 균형 설정과 과다 설정을 비교했고, 별도 300 RPS Stress에서는 작은 Thread/Hikari 설정이 포화되는 구간을 확인했다.

| 구분 | Main 부하 | 비교 설정 | 해석 목적 |
|---|---:|---|---|
| 기본 비교 | 150 RPS / 90s | Thread 16/30/60/120, Hikari 4/8/12/16 | 동일 부하에서 균형·과다 설정 비교 |
| Stress | 300 RPS / 90s | Thread 2/Hikari 2, Thread 4/Hikari 4 | underprovisioned 설정의 saturation 확인 |

아래 p95 그래프의 일반 점은 기본 비교, 빨간색 Stress 점은 별도 Stress 결과다. 두 종류의 점을 한 그래프에 배치했지만 서로 다른 부하의 p95를 직접 비교한 값은 아니다.

<figure class="report-figure">
  <a href="/assets/images/performance/thread-vs-p95.png" target="_blank" rel="noopener">
    <img src="/assets/images/performance/thread-vs-p95.png" alt="Thread와 Hikari 설정 조합별 p95 latency 점도표">
  </a>
  <figcaption>일반 점은 기본 150 RPS, 빨간색 Stress 점은 300 RPS 결과다. 기본 비교에서는 Thread 30 조합이 낮은 latency 구간에 모였고, 별도 Stress의 Thread 2 / Hikari 2에서는 p95 190ms 이상이 관찰됐다. 서로 다른 색상의 절대 p95를 직접 비교하지 않고 각 실험 안에서 설정 구간을 해석했다. <a href="/assets/images/performance/thread-vs-p95.png" target="_blank" rel="noopener">원본 크기로 보기</a></figcaption>
</figure>

<figure class="report-figure">
  <a href="/assets/images/performance/thread-vs-cswch.png" target="_blank" rel="noopener">
    <img src="/assets/images/performance/thread-vs-cswch.png" alt="Thread와 Hikari 설정 조합별 voluntary context switching 점도표">
  </a>
  <figcaption>기본 150 RPS 비교에서 Thread 30 / Hikari 8의 voluntary context switching이 가장 낮았다. DB pending이 없는 상태에서 Hikari 4와 12~16 조합의 cswch가 증가하는 패턴을 scheduling 비용 후보 신호로 해석했다. <a href="/assets/images/performance/thread-vs-cswch.png" target="_blank" rel="noopener">원본 크기로 보기</a></figcaption>
</figure>

| 조건 | 구간 | 관측 결과 | 해석 |
|---|---|---|---|
| Stress 300 RPS | Thread 2 / Hikari 2 | p95 190ms 이상, nvcswch 약 276 | underprovisioned saturation 신호 |
| 기본 150 RPS | Thread 30 / Hikari 8 | nvcswch와 cswch가 가장 안정적 | 현재 2 vCPU workload의 균형점 |
| 기본 150 RPS | Thread 60+ | 처리량 이득 없이 cswch 증가 | scheduling overhead 후보 신호 |
| 기본 150 RPS | Hikari 4 | DB pending은 없지만 cswch 증가 | borrow/return contention 후보 |
| 기본 150 RPS | Hikari 12~16 | active thread와 cswch 증가 | pool 과다에 따른 scheduling 비용 후보 |

당시 active DB connection은 1~2 수준이고 pending이 없어서 DB pool 병목 신호는 약했다. 반면 `pidstat -wt`와 `vmstat`에서는 처리량 이득 없이 context switch가 증가하는 구간이 관찰됐다. 이를 k6 p95와 함께 해석해 CPU scheduling과 애플리케이션 thread 경합을 현재 workload의 우선 병목 후보로 판단했다. 이는 인과관계를 단독으로 증명한 결과가 아니며, 트래픽이나 DB 처리량이 바뀌면 다시 측정해야 한다. 따라서 `30/8`은 절대값이 아니라 해당 workload의 운영 기준이다.

---

## 결정

기본 150 RPS 비교 결과 App/DB를 2 vCPU / 4 GB 구성으로 두고, Thread 30 / Hikari 8 조합을 선택했다. 별도 300 RPS Stress 결과는 이 조합과 p95를 직접 비교하는 대신 underprovisioned 설정의 saturation 경계를 확인하는 근거로 사용했다.

이 조합은 단순히 더 큰 리소스를 투입한 결과가 아니라, p95 <= 300ms 기준에서 CPU, DB pending, context switch 비용을 함께 보고 결정한 운영 기준이다.

---

## 개선 과정에서 배운 점

Thread와 Connection Pool을 늘리면 처리 여유도 함께 커질 것으로 생각했다. 하지만 같은 부하에서 비교해 보니 일정 구간을 넘은 설정은 처리량을 높이지 못하고 context switch와 경합 비용만 늘렸다.

운영 사양은 가장 큰 값이 아니라 목표 p95를 만족하면서 CPU, DB pending, OS 지표가 함께 안정적인 지점을 기준으로 정해야 했다. 서로 다른 부하의 결과를 직접 비교하지 않고, 관측된 신호와 인과관계도 구분해야 한다는 점을 이후 성능 검증의 기준으로 삼았다.
