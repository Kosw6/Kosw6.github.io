---
title: "SLO 기반 운영 사양 산정 - p95 300ms 기준 인프라 결정"
layout: single
permalink: /reports/slo-operating-capacity/
toc: true
toc_sticky: true
classes: wide
excerpt: "p95 <= 300ms 기준으로 App/DB 2vCPU/4GB, Thread 30, Hikari 8 설정을 도출하고 context switch와 DB pending으로 병목을 해석한 운영 사양 산정"
tags: [slo, capacity-planning, k6, linux, spring, hikari]
---

> **핵심 질문**: 목표 응답 시간 p95 <= 300ms를 만족하려면 어느 정도의 App/DB 사양과 스레드/커넥션 설정이 필요한가?
>
> 원문: [SLO 기반 운영 사양 산정 실험](https://github.com/Kosw6/engineering-notes/blob/main/reports/SLO%20%EA%B8%B0%EB%B0%98%20%EC%9A%B4%EC%98%81%20%EC%82%AC%EC%96%91%20%EC%82%B0%EC%A0%95%20%EC%8B%A4%ED%97%98.md)

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
