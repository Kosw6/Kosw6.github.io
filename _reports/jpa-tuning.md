---
title: "Canvas Node 조회 성능 개선 (JPA Fetch 전략 비교)"
layout: single
permalink: /reports/jpa-tuning/
toc: true
toc_sticky: true
classes: wide
---

> 🔍 **왜 Projection이 Fetch Join보다 느려졌는지 궁금하다면**  
> GC, 객체 생성 수, Hibernate 동작까지 포함한 전체 분석은 아래 원본 문서에 정리했습니다.  
> → [GitHub 원본 문서 보기](https://github.com/Kosw6/engineering-notes/blob/main/reports/NodeController/JPA%20Fetch%20%EC%A0%84%EB%9E%B5%EB%B3%84%20%EC%A1%B0%ED%9A%8C%20%EC%84%B1%EB%8A%A5%20%ED%85%8C%EC%8A%A4%ED%8A%B8%20(NodeController).md)

---

## 요약

| 항목 | Before | After |
|------|--------|-------|
| 지속 가능한 최대 RPS | ~26 RPS (붕괴) | **120 RPS 안정** |
| 개선 포인트 | Lazy N+1 + 10K payload | Fetch Join + DB 레벨 20자 preview |

- **핵심 발견**: 병목이 DB(N+1) → GC(객체 생성) → Payload(직렬화) 순으로 이동함
- **환경**: 4C/16GB, PostgreSQL 17 + TimescaleDB, k6 120 RPS × 90s, seed=777 고정


---

## 배경

NodeController API의 처리량이 EdgeController 대비 절반 수준(데이터 양은 Node 200만 vs Edge 400만)으로 낮게 나와 원인을 추적했다.
로그 확인 결과 `node_note_link` 테이블을 Lazy 로딩으로 가져오며 N+1 쿼리가 발생 중이었다.

---

## 1차: JPA Fetch 전략 비교 (120 RPS, 웜업 30 RPS 2m)

Lazy / Batch Fetch / Fetch Join 세 가지를 동일 조건(120 RPS, seed=777, 3회)으로 비교했다.

| 전략 | 특징 | P95 결과 |
|------|------|---------|
| Lazy Loading | N+1 쿼리 다수 발생 | 최악 |
| Batch Fetch (`default_batch_fetch_size`) | N+1 일부 완화 | 중간 |
| **Fetch Join** | 왕복 쿼리 최소화 | **최상** |

추가로 `work_mem`을 8MB → 128MB로 늘려봤지만 효과 없었다.
해시/정렬이 병목이 아니라 쿼리 왕복 횟수가 문제였고, 디스크 스필도 확인되지 않았다.

→ **Fetch Join으로 확정**

---

## 2차: JSON Aggregation 시도

UI 요구사항으로 노드 목록에 `noteSubject`(제목)도 함께 반환해야 했다.
행 폭증 문제(Node 10개 × Link 10개 = JOIN 결과 100행)를 해소하려고 `json_agg + GROUP BY`를 시도했다.

- EXPLAIN ANALYZE 결과는 양호, 행 수 감소 확인
- **그러나 k6 부하 결과: p95 증가**

원인: 집계·정렬·JSON 직렬화 CPU 비용이 행 수 절감 효과를 압도했다.

→ **JSON Aggregation 보류**

→ EXPLAIN은 좋아 보였지만 실제 성능은 악화

> 🔍 실제 EXPLAIN vs k6 결과 비교 보기 -> [GitHub에서 보기](https://github.com/Kosw6/engineering-notes/blob/main/reports/NodeController/JPA%20Fetch%20%EC%A0%84%EB%9E%B5%EB%B3%84%20%EC%A1%B0%ED%9A%8C%20%EC%84%B1%EB%8A%A5%20%ED%85%8C%EC%8A%A4%ED%8A%B8%20(NodeController).md)

---

## 3차: 스키마 변경 + 조회 전략 비교

### 스키마 개선

`node_note_link` 테이블에 `note_subject` 컬럼을 직접 추가하고,
`note.subject` 변경 시 자동 동기화되도록 DB 트리거를 적용했다.
(애플리케이션 코드 최소 변경)

### 비교 대상 4종

| 테스트 | 설명 |
|--------|------|
| 500자 / 3테이블 / JSON Aggregation | DB에서 json_agg로 묶음 |
| 500자 / 2테이블 / Projection | DTO로 필요한 필드만 |
| 20자 / 2테이블 / Projection | 프리뷰 버전 |
| 500자 / 2테이블 / Fetch Join | 기존 방식 개선판 |

### 결과

```
JSON Aggregation ≪ 500자 Projection ≪ 20자 Projection ≪ 500자 Fetch Join
```

**Projection이 Fetch Join보다 느린 이유:**
- 두 방식 모두 DB에서 100행을 읽지만
- Fetch Join은 Hibernate 1차 캐시 기반으로 부모(Node) 엔티티를 deduplicate — 자식(Link)만 컬렉션에 추가
- Projection은 DTO 100개를 전부 새로 생성 → 힙 객체 수 약 10배 차이
- GC Pause: Fetch Join **5 ms** vs Projection **6 ms** → p95까지 전파


---

## 4차: Payload 크기 영향 분석 (content 1만자 vs 20자)

동일한 Fetch Join 기반에서 content 처리 위치만 달리해 비교했다.

| 케이스 | 설명 |
|--------|------|
| DB 레벨 프리뷰 | `substring(content, 1, 20)` — 20자만 조회·반환 |
| 원문 그대로 반환 | 1만자(≈30KB)를 그대로 응답 |
| APP 레벨 프리뷰 | 1만자 조회 후 앱에서 20자로 절단 |

### 결과 요약

- **DB 레벨 프리뷰**: 120 RPS 안정
- **원문 반환**: 26 RPS 구간에서 p95 붕괴 시작 (→ GC STW 누적 → 처리율 급락)
- **APP 레벨 프리뷰**: 원문보다 안정적이지만 26 RPS에서 동일 붕괴 발생
  - GC/할당 압력이 주 원인, JSON 직렬화 비용도 tail latency에 유의미한 영향

**동일 p95 구간 기준 유지 가능한 RPS 차이: 약 5배**

---

## 최종 결정

| 항목 | 결정 |
|------|------|
| 조회 전략 | Fetch Join 유지 |
| 목록 반환 | DB 레벨 `substring(content, 1, 20)` preview |
| 상세 조회 | 별도 API — 원문 Lazy Loading |

결국 병목은 DB에서 끝나지 않았다.  
Fetch Join보다 DTO Projection이 더 느렸고, 병목은 **DB → GC → Payload** 순으로 이동했다.

> 📊 GC pause, allocation, 실제 부하 로그까지 포함한 전체 분석 보기  
> → [GitHub 원본 문서 보기](https://github.com/Kosw6/engineering-notes/blob/main/reports/NodeController/JPA%20Fetch%20%EC%A0%84%EB%9E%B5%EB%B3%84%20%EC%A1%B0%ED%9A%8C%20%EC%84%B1%EB%8A%A5%20%ED%85%8C%EC%8A%A4%ED%8A%B8%20(NodeController).md)

---

## 핵심 인사이트

> 성능 병목은 DB에서 끝나지 않는다.
> N+1을 제거하면 GC가, GC를 줄이면 Payload가 다음 병목으로 드러난다.
> 각 단계마다 실측 없이 예상으로 결론 내리면 반대 결과가 나온다 (Projection < Fetch Join).