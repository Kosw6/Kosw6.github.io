---
title: "JFR / JMC 기반 Allocation Hotspot 분석 및 JWT 핫패스 개선"
layout: single
permalink: /reports/jfr-jmc-hotpath/
toc: true
toc_sticky: true
classes: wide
---

> 🔍 **쿼리 문제가 아닌데 왜 느렸는지, JFR/JMC로 추적한 전체 과정 보기**  
> (Stack Trace · Allocation 분석 · JWT 핫패스 발견 과정 포함)  
> → [GitHub 원본 문서 보기](https://github.com/Kosw6/engineering-notes/blob/main/reports/NodeController/JFR/JMC%EB%A5%BC%20%ED%99%9C%EC%9A%A9%ED%95%9C%20Allocation%20%EA%B8%B0%EB%B0%98%20%EC%84%B1%EB%8A%A5%20%EB%B3%91%EB%AA%A9%20%EB%B6%84%EC%84%9D.md)

---

## 요약

| 항목 | V1 (개선 전) | V2 (개선 후) |
|------|-------------|-------------|
| Old GC Total Time (90s 본부하) | 3.47 s | **2.22 s** |
| 개선율 | — | **약 36% 감소** |

> ⚠️ 왜 GC 시간이 줄어들었는지 (JWT 중복 검증 발견 과정)  
> → [원본 문서 보기](https://github.com/Kosw6/engineering-notes/blob/main/reports/NodeController/JFR/JMC%EB%A5%BC%20%ED%99%9C%EC%9A%A9%ED%95%9C%20Allocation%20%EA%B8%B0%EB%B0%98%20%EC%84%B1%EB%8A%A5%20%EB%B3%91%EB%AA%A9%20%EB%B6%84%EC%84%9D.md)

- **발견 방법**: JFR + JMC Stack Trace → JWT 검증 중복 실행 확인
- **핵심**: 쿼리 문제가 아닌 상황에서, JFR/JMC Stack Trace 분석으로 JWT 인증 경로의 숨은 핫패스를 발견

---

## 배경

JPA Fetch 전략 최적화 이후에도 RPS 증가 시 p95가 상승하는 문제가 남아 있었다.
쿼리 레벨에서 개선 방향이 불명확해진 시점에, **JFR + JMC를 이용한 런타임 분석**으로 전환했다.

분석 대상:
- Top Allocating Classes
- Top Stack Trace
- GC Summary (Young / Old / All / Pause)

비교 구조:
- **V1**: 기존 구조 (20자 preview + Fetch Join)
- **V2**: V1 + hotpath 개선 (JWT 중복 제거)
- **2step**: 조회 분리 (Node만 먼저 조회 → Link 별도 조회)

---

## 사전 분석: 10K content의 JMC 프로파일

10K 콘텐츠(20 RPS 부하) JMC 분석 결과:

- **Top Allocation**: `byte[]`, `char[]` — DB → JDBC → String 디코딩 과정에서 폭발적 생성
- **Top Stack Trace**: `PGStream.receiveTupleV3()` 높은 비중
- GC Summary: 총 GC 시간 9.59s, 총 STW 5.06s (90s 본부하 중)

20자 preview로 전환 후 동일 분석 (60 RPS 본부하):
- Memory Allocation 약 **5배 감소**
- GC 약 **절반 수준**

→ 대용량 컨텐츠 목록 반환이 JVM에 미치는 영향을 정량적으로 확인

---

## V1 → V2: JWT 핫패스 발견 및 개선

### JMC Stack Trace 분석 결과

V1의 Top Stack Trace에서 `BaseNCodec.ensureBufferSize()`가 높은 빈도로 등장했다.

라이브러리 추적 결과:
```
BaseNCodec.decode()
  └─ Base64.decodeBase64()
       └─ JWTDecoder.Base64.decodeBase64()
            └─ JWT.require().verify(token)
```

### 근본 원인

코드 분석 결과, `JwtFilter`와 `JwtTokenProvider` 두 곳에서 **JWT 검증이 중복 실행**되고 있었다.

```java
// 기존: 필터에서 validateToken() 호출 → 내부에서 또 getTokenInfo() 호출 → validateToken() 재호출
Authentication auth = jwtTokenProvider.getAuthentication(token, userDetailService);
// → validateToken이 2회 실행 → Base64 decode 2배 호출
```

```java
// 개선: 검증을 1회로 통합 — DecodedJWT를 직접 전달
DecodedJWT jwt = jwtTokenProvider.validateTokenOrThrow(token); // 1번만 검증
Authentication auth = jwtTokenProvider.getAuthentication(jwt, userDetailService);
```

### 개선 효과 (60 RPS, 동일 환경)

| 항목 | V1 | V2 |
|------|----|----|
| Old GC Total Time | 3.47 s | **2.22 s** |
| 개선율 | — | **약 36% 감소** |

- `BaseNCodec.ensureBufferSize` 호출 수 감소
- `AbstractQueuedSynchronizer$ConditionNode` 할당 감소
- 핫패스 개선만으로 메모리 안정성과 성능이 동시에 개선됨

> 🔍 JFR Stack Trace에서 어떻게 이 경로를 추적했는지 전체 분석 보기  
> → [GitHub](https://github.com/Kosw6/engineering-notes/blob/main/reports/NodeController/JFR/JMC%EB%A5%BC%20%ED%99%9C%EC%9A%A9%ED%95%9C%20Allocation%20%EA%B8%B0%EB%B0%98%20%EC%84%B1%EB%8A%A5%20%EB%B3%91%EB%AA%A9%20%EB%B6%84%EC%84%9D.md)

---

## V2 → 2step: 병목 이동 확인

2step 구조(Node만 먼저 조회 후 Link 별도 조회)에서 JMC 재분석:

- `PGStream.receiveTupleV3` 비중 감소 → DB/JDBC 수신 병목 완화
- 그러나 새로운 병목 등장: `Method`, `ResolvableType`, `Object[]`, `ArrayList` 등 **객체 그래프 조립 비용**

→ 병목은 제거되는 게 아니라 이동한다. 구조 변경 후 반드시 재분석 필요.

---

## 핵심 인사이트

> 쿼리 튜닝 이후 보이지 않던 병목은 런타임 레이어에 숨어 있다.
> JMC Stack Trace는 코드 리뷰만으로 발견하기 어려운 공통 핫패스(인증 경로)를 드러냈다.
> 인증 로직은 모든 요청에 실행되므로, 고정 비용이 작아도 부하가 높아지면 tail latency에 누적된다.

---

## 연결되는 문제

 WebSocket 환경에서는 단순 쿼리 문제가 아닌  
**런타임 처리 비용과 fanout 구조 자체가 병목이 된다**

>→ [WebSocket 성능 개선 시리즈 보기](/reports/websocket-group-canvas/)