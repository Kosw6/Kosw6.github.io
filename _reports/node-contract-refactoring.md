---
title: "협업 편집의 변경·이력·이벤트를 일치시키는 계약 중심 리팩터링"
layout: single
permalink: /reports/node-contract-refactoring/
classes: wide
excerpt: "Node 변경 불변식, 업무 트랜잭션과 Outbox 전달 계약을 먼저 정의하고 객체의 책임과 의존성을 분리한 설계 사례"
tags: [java, spring, solid, oop, transaction, outbox, refactoring]
---

> **핵심 질문**: Node 수정이 성공했다면 어떤 상태가 함께 확정되어야 하며, 외부 전파 실패는 어디에서 복구해야 하는가?

<nav class="page-quick-nav" aria-label="핵심 섹션 바로가기">
  <strong>빠르게 보기</strong>
  <a href="#change-contract">변경 불변식</a>
  <a href="#transaction-contract">업무 성공</a>
  <a href="#delivery-contract">전달과 복구</a>
  <a href="#validation">검증 범위</a>
  <a href="#sources">코드 근거</a>
</nav>

<div class="proof-strip">
  <div class="proof-item"><span class="proof-item__label">변경 계약</span><strong>실제 적용 결과를 공통 기준으로</strong><span>Node · History · Outbox의 필드와 버전 일치</span></div>
  <div class="proof-item"><span class="proof-item__label">업무 성공</span><strong>세 DB 상태의 원자적 커밋</strong><span>Redis 힌트와 캐시 처리는 커밋 이후로 분리</span></div>
  <div class="proof-item"><span class="proof-item__label">복구 계약</span><strong>실패 분류와 고정 재처리 기한</strong><span>동일 eventId 재발행 · claimToken 소유권 검사</span></div>
</div>

Trader의 협업 캔버스에서는 사용자가 Node의 제목·본문·연결 노트를 수정하고, 다른 사용자가 그 결과를 실시간으로 확인합니다. 이 기능에는 저장된 내용뿐 아니라 충돌 판단에 쓰는 이력, 업무 버전, 변경 이벤트가 같은 사실을 표현해야 한다는 요구가 있습니다.

기능이 확장되면서 요청 DTO 해석, 상태 변경, 이력 기록, 메시지 변환과 외부 전파의 책임이 한 흐름에 모였습니다. 기존 코드를 분석해 실패 가능한 조건을 먼저 정리하고, 각 단계가 지켜야 할 계약을 정의한 뒤 책임을 분리했습니다. 아래 변경 전 문제는 코드에서 확인한 위험 조건이며, 모두 실제 운영 장애로 발생했다는 의미는 아닙니다.

## 실제 변경과 이력을 일치시키는 계약 {#change-contract}

**발견한 문제**

기존 구조는 `dirtyFields`로 충돌 검사와 이력의 대상을 결정하면서, Node 변경에는 DTO의 null이 아닌 값을 사용했습니다. 제목만 변경한다고 표시한 요청에 본문 값이 함께 들어오면, 실제 저장과 이력이 달라질 수 있었습니다.

```json
{
  "baseVersion": 3,
  "dirtyFields": ["subject"],
  "subject": "새 제목",
  "content": "새 본문"
}
```

이러한 불일치는 이후 충돌 검사에서도 실제 본문 변경을 놓칠 수 있는 원인이 됩니다.

**먼저 정한 계약**

| 조건 | 지켜야 할 동작 |
|---|---|
| 요청 필드와 전달 값이 불일치 | 계약 위반을 명시적으로 거절하고 상태를 변경하지 않음 |
| 유효한 실제 변경 | Node 적용 결과의 필드·버전을 History와 Outbox가 동일하게 사용 |
| 검증을 통과한 동일 값 요청 | 버전 증가·History·Outbox 생성 없이 no-op 성공 |
| 연결 노트 변경 | 서비스가 요청자의 소유권을 확인하고, 요청자가 교체할 수 있는 연결 범위만 변경 |
| 변경 근거인 DB History 손상 | 정상적인 빈 변경으로 바꾸지 않고 검증 실패로 처리 |

Redis 버전 힌트도 같은 적용 결과를 사용하지만, 저장 성공 자체는 업무 성공 조건에 포함하지 않았습니다. 힌트가 없으면 DB History를 기준으로 판단하도록 했습니다.

**책임을 배치한 방법**

생성·내용 수정·위치 수정의 요청 DTO를 분리하고, `NodeChangeRequestValidator`가 요청을 변경 계획으로 해석하게 했습니다. `Node`는 검증된 `NodeContentChange`를 적용하고 `NodeChangeResult`로 실제 적용 필드와 버전을 반환합니다. 서비스는 이 결과를 History와 업무 이벤트에 전달합니다.

```text
요청 DTO
  → NodeChangeRequestValidator: 요청 의도·유효성·실제 차이 확인
  → NodeConflictValidator: 최신 변경 문맥과 충돌 확인
  → Node.applyContentChange(): 상태 변경과 연결 규칙 적용
  → NodeChangeResult: appliedFields + newVersion
       ├─ NodeHistory
       ├─ NodeChange → Outbox
       └─ 커밋 후 Redis 힌트
```

연결 컬렉션은 읽기 전용으로 노출하고, 연결 교체는 `replaceNoteLinks()`를 통해 수행하게 했습니다. 이미 연결된 Note의 중복 연결은 no-op으로 처리합니다. 소유권을 조회하는 책임은 서비스에, 전달받은 교체 가능 범위 안에서 연결을 바꾸는 규칙은 Node에 두었습니다.

**객체지향 관점**

요청 형식의 변경, 도메인 규칙의 변경, 이력 기록 방식의 변경을 구분한 SRP 사례입니다. 특히 연결 컬렉션을 외부에서 직접 수정할 수 없게 하고 행동을 통해 변경하게 한 점은 캡슐화와 불변식 보호를 설명하는 근거입니다. DTO 분리 자체를 곧바로 ISP라고 설명하지는 않습니다.

**검증 근거와 비용**

`NodeServiceContentUpdateTest`에는 실제 변경 필드의 일치와 no-op의 부수효과 부재, 소유권 거절, 빈 노트 목록의 해제 의미를 검증하는 테스트가 있습니다. `NodeContentChangeTest`에는 변경 전 유효성 검사, 중복 연결의 멱등성, 읽기 전용 컬렉션 테스트가 있습니다.

변경 계획과 결과 객체가 늘어나는 비용을 수용했습니다. 그 대신 각 단계가 요청을 제각기 다시 해석하지 않도록 했습니다.

## 업무 성공과 후처리의 경계 {#transaction-contract}

**발견한 문제**

기존 서비스가 `RawCursorMessage`와 `RealtimeEnvelope`를 직접 구성하면 메시지 형식의 변경이 업무 로직까지 영향을 줍니다. 또한 DB 트랜잭션 안에서 Redis 상태나 외부 이벤트를 먼저 변경하면, 뒤의 DB 작업 실패로 롤백되더라도 외부에 남은 정보를 되돌리기 어렵습니다.

**먼저 정한 계약**

팀 Node 내용 수정의 업무 성공은 다음 세 상태의 원자적 커밋입니다.

```text
Node 변경 + NodeHistory 기록 + RELIABLE Outbox 기록
```

Outbox 적재 실패는 업무 실패로 전파하고 전체를 롤백합니다. Redis 힌트, 편집 세션 종료와 캐시 무효화는 커밋 후 실행하며, 각 작업의 실패가 다음 작업을 취소하거나 이미 커밋된 업무 결과를 바꾸지 않도록 했습니다.

**책임과 의존성을 바꾼 방법**

업무 변경 사실을 나타내는 `NodeChange`와 이를 현재 트랜잭션에 기록하는 `NodeChangeEventStager`를 도입했습니다. 서비스는 이 계약에 의존하고, `OutboxNodeChangeEventStager`가 기존 메시지 형식으로의 변환과 Outbox 기록 요청을 담당합니다.

```text
NodeService / TeamNodePositionWriter
  → NodeChangeEventStager.stage(NodeChange)
       ↑ 구현
    OutboxNodeChangeEventStager
      → 전송 DTO·Envelope 변환
      → TransactionalReliableEventService
      → 현재 DB 트랜잭션에 Outbox 기록
```

이 계약은 메서드의 정상 반환 의미까지 명시합니다.

> `stage()`의 정상 반환은 현재 트랜잭션에 변경 이벤트를 기록했다는 뜻입니다. DB 커밋이나 외부 전달 완료를 뜻하지 않습니다.

Outbox 구현에는 `Propagation.MANDATORY`를 적용해 활성 트랜잭션 없는 호출을 거절했습니다. 별도 트랜잭션으로 저장하면 업무 롤백 후에도 이벤트만 남을 수 있기 때문입니다.

**동시성 계약 보강**

필드 충돌 검증 이후에도 저장 전 경쟁이 생길 수 있으므로, 업무 버전과 별도의 JPA `@Version` 필드인 `lockVersion`을 두었습니다. 저장 시 CAS가 실패하면 Node·History·Outbox를 롤백하고 새 트랜잭션에서 최대 한 번 재검증합니다.

현재 내용 수정 정책은 서버의 내용 변경이 있으면 비중첩 필드라도 병합 결과를 검토하도록 `MERGE_REVIEW_REQUIRED`를 반환합니다. 위치만 바뀌어 내용 문맥에 영향이 없는 경우와는 구분합니다. 기술적으로 합칠 수 있는지와 사용자가 합쳐진 내용을 의도했는지를 별도 판단으로 둔 것입니다.

**객체지향 관점**

업무 변경과 전송 표현의 변경 이유를 분리한 SRP, 상위 서비스가 업무 의미의 추상화에 의존하도록 한 DIP 사례입니다. 트랜잭션 원자성과 CAS는 이 설계가 보호해야 할 계약을 구현하는 별도 메커니즘입니다.

**검증 근거와 비용**

`NodeServiceTransactionContractTest`는 Outbox flush 후 실패를 주입하고 별도 조회에서 Node 원복, History·Outbox 부재와 후처리 미실행을 확인합니다. 같은 스냅샷을 읽은 두 트랜잭션의 저장 경쟁 테스트도 있습니다. `NodeChangeStagingTransactionTest`는 트랜잭션 없는 호출 거절과 롤백·커밋에 따른 알림 차이를 검증합니다.

커밋 후 리스너는 동기 실행이므로 응답 시간에 영향을 줄 수 있습니다. 또한 프로세스가 커밋 직후 종료되면 애플리케이션 이벤트 기반 후처리는 실행되지 않을 수 있습니다. RELIABLE 이벤트는 Outbox로 기록하고, 파생 작업에는 DB fallback·캐시 무효화 재시도·TTL 등 작업별 복구 방식을 선택했습니다.

## 이벤트 전달과 복구의 계약 {#delivery-contract}

**발견한 문제**

하나의 Publisher가 실시간 전달과 내구성 기록을 함께 표현하면 호출자는 정상 반환이 무엇을 보장하는지 알기 어렵습니다. Kafka ACK 이후 DB의 SENT 기록이 실패하는 경우도 있으므로, 발행과 완료 기록을 하나의 성공으로 취급할 수 없습니다.

**먼저 정한 계약**

| 경계 | 성공의 의미 | 보장하지 않는 것 |
|---|---|---|
| Live 발행 | TRANSIENT·VOLATILE 실시간 경로 호출 | DB 내구성, 모든 클라이언트 수신 |
| Outbox stage | 활성 트랜잭션에 전달 의무 기록 | 커밋 완료, Kafka 전달 |
| 업무 커밋 | 업무 상태·이력·Outbox가 함께 확정 | 모든 후처리와 외부 전달 완료 |
| Kafka ACK | 브로커 발행 확인 | 소비자 처리·WebSocket 수신 완료 |
| Outbox SENT | 현재 claim 소유자가 완료 상태 기록 | exactly-once 또는 전체 화면 수렴 |

**책임을 배치한 방법**

`LiveRealtimePublisher`에는 `publishTransient()`와 `publishVolatile()`만 두고, RELIABLE은 Outbox 기록 경로로 분리했습니다. 커밋 후 리스너는 `CommittedEventPropagator`를 호출하며, single 구현은 로컬 전파, multi 구현은 로컬 전파와 Redis 원격 전파를 담당합니다. Kafka 전달은 별도 Relay가 처리합니다.

```text
DB 트랜잭션
  Node + History + Outbox
          │ 커밋
          ├─ AFTER_COMMIT → CommittedEventPropagator
          │                    ├─ single: 로컬 전파
          │                    └─ multi: 로컬 전파 + Redis 전파
          └─ Outbox Relay → 선점 → Kafka ACK → SENT 기록
```

Relay는 JSON 해석, Kafka 발행, SENT 기록과 복구 상태 기록의 실패를 구분합니다. DB 기록 실패를 Kafka 장애로 해석하지 않고, 영구적 데이터·발행 오류는 FAILED로 격리합니다. ACK 뒤 SENT 기록이 실패하면 동일한 `eventId`로 재발행할 수 있게 했습니다.

Outbox 선점 시도마다 별도의 `claimToken`을 발급하고, 상태 변경은 `eventId + claimToken`이 일치할 때만 허용합니다. 이벤트의 동일성과 현재 작업자의 갱신 권한을 구분해, lease 만료 후 오래된 작업자가 새 작업자의 상태를 덮어쓰는 것을 방어합니다. 이 토큰이 Kafka 중복 발행 자체를 막는 것은 아닙니다.

**재시도의 종료 조건**

최초 DB 선점 시각부터 3분의 고정 재처리 기한을 두고, 지수 백오프와 지터를 적용했습니다. 최초 시도 전 Kafka DOWN 대기는 기한을 시작하지 않지만, 최초 시도 후의 DOWN 시간은 기한에 포함됩니다. 기한이 지나면 새 선점을 거절하고 실패 상태로 종료하며, 기한 전 선점한 활성 작업의 늦은 ACK 기록은 허용합니다.

따라서 이 구조는 중복을 허용하는 제한된 재시도 계약입니다. 모든 이벤트의 최종 전달이나 exactly-once를 보장한다고 표현하지 않습니다. 수신 측 로컬 중복 캐시도 만료·재시작 한계가 있고, DB 원본 재조회에 의한 화면 수렴은 별도 통합 검증 대상입니다.

**객체지향 관점**

- SRP: 실시간 전파, 내구성 기록, Kafka 전달과 상태 기록의 변경 이유·실패 책임을 분리했습니다.
- ISP: 실시간 발행 호출자는 내구성 기록 능력까지 의존하지 않습니다.
- OCP: 커밋 후 상위 흐름을 유지하면서 실행 모드별 전파 정책을 구현체로 분리했습니다. 새로운 모드를 추가할 때 설정까지 무수정이라는 뜻은 아닙니다.
- DIP: 커밋 후 리스너는 Redis 구현 대신 `CommittedEventPropagator`에 의존합니다.
- LSP: live 구현의 유효 입력 범위를 일치시켰습니다. 다만 Local 구현은 broadcaster 예외를 전파하고 Multi 구현은 격리하므로, 공통 실패 계약까지 완전히 검증했다고 주장하지 않습니다.

**검증 근거와 비용**

`RealtimeOutboxRelayTest`에는 ACK 확인 후 SENT 기록과 발행 실패의 재시도 테스트가 있습니다. 실패 분류 테스트와 `OutboxRetryWindowTest`에는 최초 기한 보존, 정확한 기한의 선점 거절, 늦은 ACK, stale token 거절을 확인하는 코드가 있습니다.

실패 분류와 선점 상태를 저장하는 비용을 수용했고, 자동 복구 기간에 상한을 두었습니다. 이 선택은 운영 정책이며, 모든 서비스에 적합한 보편적 재시도 시간으로 제시하지 않습니다.

## 검증 범위와 남은 과제 {#validation}

2026년 9월 설계·구현 기록에는 단위 테스트와 H2/Spring 트랜잭션 테스트의 완료 결과를 남겼습니다. 아래 테스트는 정상 반환뿐 아니라 실패 이후 DB 상태, 후처리 실행 여부와 재시도 경계를 확인합니다. 2026년 10월 22일 포트폴리오 정리 시 구현과 테스트 코드를 대조했으며, 새 테스트 실행 결과를 추가한 것은 아닙니다.

| 검증 대상 | 근거 테스트 | 확인하는 계약 |
|---|---|---|
| 변경 필드와 no-op | NodeServiceContentUpdateTest | 실제 변경·이력·이벤트의 공통 기준, 동일 값 요청의 부수효과 부재 |
| 연결 불변식 | NodeContentChangeTest | 중복 연결의 멱등성, 컬렉션의 읽기 전용 노출 |
| 업무 트랜잭션 | NodeServiceTransactionContractTest | Outbox 실패 시 전체 롤백, 커밋 후 처리, 저장 경쟁 검출 |
| 기록 경계 | NodeChangeStagingTransactionTest | 트랜잭션 없는 호출 거절, 커밋과 롤백에 따른 알림 차이 |
| 재처리 기한 | OutboxRetryWindowTest | 기한 보존, 경계 시각 선점 거절, 늦은 ACK와 오래된 token 처리 |

실제 PostgreSQL 마이그레이션, Redis·Kafka 장애, 다중 서버 경쟁과 브라우저 수렴은 설계 문서에 별도 통합 검증 과제로 남아 있습니다. 과거 장애 PoC에서 관찰한 ‘중복 0건’을 현재 구조의 exactly-once 보장으로 해석하지 않습니다.

## 설계에서 얻은 기준

요청이 무엇을 바꿀 수 있는지, 정상 반환이 무엇을 뜻하는지, 실패 후 어떤 상태를 허용하는지를 먼저 정했습니다. 이 계약을 기준으로 요청 검증, 도메인 변경, 전달 의무 기록과 외부 전파의 책임을 나누었습니다.

모든 클래스에 인터페이스를 추가하지는 않았습니다. 업무 변경과 전송 표현처럼 변경 이유가 다르거나, single/multi처럼 실제 정책 교체가 필요한 경계에 추상화를 두었습니다. SRP·DIP·ISP·OCP의 적용 근거와 LSP에서 남아 있는 실패 계약 검토 사항을 구분한 이유입니다.

## 계약을 표현한 코드 {#sources}

업무 이벤트 기록 경계는 정상 반환의 의미를 인터페이스에 명시합니다. 아래는 `NodeChangeEventStager`의 계약입니다.

```java
public interface NodeChangeEventStager {
    /**
     * 활성 DB 트랜잭션에 변경 이벤트를 기록한다. 실패는 호출자에게 전파한다.
     * 정상 반환은 커밋 또는 외부 전달 완료를 뜻하지 않는다.
     */
    void stage(NodeChange change);
}
```

`OutboxNodeChangeEventStager`는 이 계약을 구현하고, 활성 트랜잭션이 없으면 호출을 거절합니다. 다음 코드는 트랜잭션 경계를 보여주기 위해 변환 부분을 생략한 발췌입니다.

```java
@Override
@Transactional(propagation = Propagation.MANDATORY)
public void stage(NodeChange change) {
    // NodeChange 검증 → 전송 DTO와 Envelope 변환 → Outbox 기록 요청
}
```

실제 적용 결과는 수정된 필드와 버전을 함께 전달합니다. `NodeChangeResult`는 필드 목록을 복사해 이후 단계가 결과를 임의로 바꾸지 못하게 합니다.

```java
public record NodeChangeResult(
        int previousVersion,
        int newVersion,
        List<String> appliedFields
) {
    public NodeChangeResult {
        appliedFields = List.copyOf(appliedFields);
    }

    public boolean changed() {
        return !appliedFields.isEmpty();
    }
}
```

분석에는 `node_refactor_design.md`, `reliable_event_delivery_before.md`, `reliable_event_delivery_solid_mapping.md`의 설계 결정과 후속 변경 기록을 사용했습니다. 해당 원문과 코드의 공개 URL은 확인되지 않아, 이 글에 핵심 계약·코드 발췌·테스트 범위와 남은 과제를 함께 정리했습니다.

→ [Trader 프로젝트 전체 보기](/projects/trader/) · [전체 문제 해결 기록](/reports/)
