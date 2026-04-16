---
title: "PoC 2 — Fallback 환경에서의 상태 동기화 및 충돌 제어"
layout: single
permalink: /reports/websocket-poc2-conflict/
toc: true
toc_sticky: true
classes: wide
---

> 🔍 **Fallback 환경에서 편집 충돌을 어떻게 해결했는지 (Kafka · Redis · 필드 단위 충돌 감지 설계)** 

> **원본 분석 노트**: [GitHub에서 보기](https://github.com/Kosw6/engineering-notes/blob/main/reports/GroupController/poc2-fallback-state-sync-conflict-resolution.md)
<br>

> **시리즈**: [WebSocket 성능 개선](/reports/websocket-group-canvas/) 
 <br>[PoC 1: 샤딩](/reports/websocket-poc1-sharding/)
 <br>**PoC 2: Fallback & 충돌 제어**
 <br>[PoC 3: Failback & Replay](/reports/websocket-poc3-failback/)

---

## 요약

| 시나리오 | 결과 |
|---------|------|
| 다른 필드 수정 후 서버 변경 | **AUTO_MERGE** ✅ |
| 동일 필드 수정 후 서버 변경 | **CONFLICT** 감지 ✅ |
| shard 장애 시 다른 서버로 우회 | **Fallback 라우팅** ✅ |

- **문제**: 샤딩 구조에서 특정 shard 장애 시 → 다른 인스턴스로 fallback → 편집 상태 불일치
- **핵심 설계**: Kafka로 상태를 전파하고, Redis Draft로 필드 단위 변경 이력을 추적하여 fallback 환경에서도 충돌을 정확히 판별

> ⚠️ 왜 fallback 상황에서도 AUTO_MERGE / CONFLICT를 정확히 구분할 수 있었는지 
> **원본 분석 노트**: [GitHub에서 보기](https://github.com/Kosw6/engineering-notes/blob/main/reports/GroupController/poc2-fallback-state-sync-conflict-resolution.md)

---

## 배경

[PoC 1](/reports/websocket-poc1-sharding/)에서 그룹 샤딩으로 부하를 분산했다.
그러나 샤딩 구조에서 특정 서버가 장애를 일으키면,
해당 shard의 사용자가 fallback 서버로 이동하게 된다.

이때 두 가지 문제가 발생한다:

1. **인스턴스 간 메모리 상태 공유 불가** — 기존 서버의 편집 맥락을 fallback 서버가 모름
2. **편집 중 다른 사람이 변경한 경우** — 내가 편집 시작 후 서버에 변경이 생겼을 때 어떻게 판단할 것인가

---

## 핵심 설계

<div style="text-align:center;">
  <img src="{{ '/assets/images/poc2.png' | relative_url }}" alt="아키텍쳐 다이어그램">
</div>

### Draft 상태 구조 (Redis)

사용자가 노드 편집을 시작하면 Redis에 Draft가 생성된다.

```json
{
  "baseVersion": 0,
  "draftPatch": {},
  "dirtyFields": [],
  "serverChangedFieldsAfterEdit": []
}
```

| 필드 | 의미 |
|------|------|
| `baseVersion` | 편집 시작 시점의 서버 버전 |
| `dirtyFields` | 내가 수정한 필드 목록 |
| `serverChangedFieldsAfterEdit` | 내가 편집 중에 서버에서 변경된 필드 목록 |

Draft는 짧은 생명주기(TTL 기반 자동 정리)를 가지므로 Redis를 사용했다.

---

### Kafka → Draft 반영

다른 사용자의 편집이 서버에 저장될 때마다 Kafka 이벤트가 발행된다.
WS 서버는 이를 소비하여 현재 편집 중인 사용자의 Draft에 `serverChangedFieldsAfterEdit`를 추가한다.

```java
@KafkaListener(topics = "canvas-events")
public void consume(CanvasEventEnvelope event) {
    // 해당 노드를 편집 중인 사용자만 찾아서
    Set<String> editingUsers = draftRedisStore.findEditingUsers(event.getGroupId(), event.getEntityId());

    for (String userId : editingUsers) {
        DraftEditState draft = draftRedisStore.find(...);
        if (event.getVersion() > draft.getBaseVersion()) {
            // 내가 편집 시작 이후의 변경분만 기록
            draft.getServerChangedFieldsAfterEdit().addAll(event.getChangedFields());
        }
    }
}
```

fallback 환경에서도 Kafka를 통해 인스턴스 간 상태 동기화가 이루어진다.

---

### 충돌 감지 로직

저장(Validate) 요청 시 세 가지 결과 중 하나를 반환한다.

```java
public String validate(...) {
    // 편집 중 서버 변경이 없으면 안전
    if (draft.getBaseVersion().equals(node.getVersion())) return "SAFE";

    // 내가 수정한 필드와 서버가 수정한 필드의 교집합
    Set<String> conflict = new HashSet<>(draft.getDirtyFields());
    conflict.retainAll(draft.getServerChangedFieldsAfterEdit());

    return conflict.isEmpty() ? "AUTO_MERGE" : "CONFLICT";
}
```

| 결과 | 조건 |
|------|------|
| `SAFE` | 편집 중 서버 변경 없음 |
| `AUTO_MERGE` | 서버 변경 있지만 내가 수정한 필드와 겹치지 않음 |
| `CONFLICT` | 내가 수정한 필드를 서버도 수정함 |

> 🔍 Draft 구조와 필드 단위 충돌 감지 설계 전체 보기  
> **원본 분석 노트**: [GitHub에서 보기](https://github.com/Kosw6/engineering-notes/blob/main/reports/GroupController/poc2-fallback-state-sync-conflict-resolution.md)

---

## E2E 검증 흐름

### Fallback 라우팅

```json
// 정상
{ "primaryShardId": 1, "selectedShardId": 1, "fallbackUsed": false }

// shard 장애 시
{ "primaryShardId": 1, "selectedShardId": 2, "fallbackUsed": true }
```

### AUTO_MERGE 시나리오

```
1. 사용자 → 노드 편집 시작 (baseVersion=0, dirtyFields=[])
2. 사용자 → subject 필드 수정 (dirtyFields=["subject"])
3. 다른 사용자 → x, y 변경 후 저장 → Kafka 이벤트 발행
4. Draft 업데이트 (serverChangedFieldsAfterEdit=["x","y"])
5. Validate → dirtyFields ∩ serverChanged = ∅ → AUTO_MERGE ✅
```

### CONFLICT 시나리오

```
6. 다른 사용자 → subject 변경 후 저장 → Kafka 이벤트 발행
7. Draft 업데이트 (serverChangedFieldsAfterEdit=["subject"])
8. Validate → dirtyFields ∩ serverChanged = {"subject"} → CONFLICT ✅
```

---

## 핵심 인사이트

> 편집 충돌은 "버전이 다르다"는 사실만으로는 판단할 수 없다.
> 버전이 달라도 수정한 필드가 겹치지 않으면 자동 병합이 가능하고,
> 이를 필드 단위로 추적하는 Draft 구조가 fallback 환경에서도 정합성을 유지하게 한다.

## 다음 단계

fallback 환경에서 정합성은 해결했다.

하지만 장애 서버가 복구되면 또 다른 문제가 남는다.

**이벤트를 놓친 서버를 어떻게 다시 정상 상태로 복구할까?  
그리고 서비스를 끊지 않고 어떻게 원래 구조로 되돌릴까?**

>→ [PoC 3 — Failback & Replay](/reports/websocket-poc3-failback/)