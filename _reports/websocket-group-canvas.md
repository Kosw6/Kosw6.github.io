---
title: "WebSocket 실시간 브로드캐스트 성능 개선"
layout: single
permalink: /reports/websocket-group-canvas/
toc: true
toc_sticky: true
classes: wide
---
> ⚠️ 왜 ≤200ms 0.38% → 99.97%가 되었는지 전체 실험 과정  
> **원본 분석 노트**: [GitHub에서 보기](https://github.com/Kosw6/engineering-notes/blob/main/reports/GroupController/WebSocket.md)

---

## 요약

| 항목 | Before | After |
|------|--------|-------|
| ≤200ms 수신 성공률 | **0.38%** | **99.97%** |

- **환경**: 200명 동시 접속, 송신자 20명, 20Hz → 80,000 send/s fan-out
- **문제 1**: 동일 세션에 동시 send → TEXT_PARTIAL_WRITING → room 붕괴
- **문제 2**: Drop 전략 없이 All-delivery → 지연 누적
- **해결**: ConcurrentWebSocketSessionDecorator + 최신값 Coalescing + Dirty Flag

---

## 배경 및 실험 목적

실시간 협업 캔버스에서 마우스 커서, 드래그 미리보기, 하이라이트 등 **휘발성(Ephemeral) 이벤트**를 브로드캐스트한다.
이 데이터는 완전한 전달보다 **최신 상태가 빠르게 도달하는 것**이 중요하다.

| 항목 | 설정 |
|------|------|
| 동시 접속자 | 200명 (동일 룸) |
| 송신자 비율 | 10% (20명), 20Hz |
| 인바운드 이벤트율 | 400 msg/s |
| 아웃바운드 fan-out | 최대 80,000 send/s |
| SLO | **≤200ms 수신 성공률** |

RAW WebSocket과 Spring STOMP를 비교하여 휘발성 이벤트에 적합한 구조를 검증했다.

---

## 문제 1: RAW WebSocket 동시 sendMessage() 충돌

### 증상

```
WARN [exec-144] [RAW] send fail session=b508ac... ex=IllegalStateException: TEXT_PARTIAL_WRITING
WARN [exec-21 ] [RAW] send fail session=c1e70c... ex=IllegalStateException: TEXT_PARTIAL_WRITING
INFO [exec-24 ] [RAW] left session=59c431... roomKey=1:1 size=13 → ... size=0
```

200명 브로드캐스트 중 동일 세션에 멀티스레드 sendMessage()가 동시 호출되어
Tomcat RemoteEndpoint가 예외를 발생시키고, 세션이 room에서 연쇄 탈락했다.

개선 전 수신량: 6,509 events (이상적: 2,400,000)

### 해결: ConcurrentWebSocketSessionDecorator

`synchronized` 대신 Decorator를 선택한 이유:
- `synchronized`는 느린 세션이 다른 스레드를 블록
- Decorator는 세션별 버퍼링 + 전송 시간/버퍼 제한으로 **느린 세션이 전체 성능을 끌어내리는 것을 방지 (백프레셔)**

```java
int sendTimeLimitMs    = 5_000;
int bufferSizeLimit    = 512 * 1024;
WebSocketSession safe  = new ConcurrentWebSocketSessionDecorator(session, sendTimeLimitMs, bufferSizeLimit);
registry.join(roomKey, safe);
```

### 결과

| Type | Received | Recv/s | ≤200ms |
|------|----------|--------|--------|
| 개선 전 RAW | 6,509 | 99.52 | 1.03% |
| **개선 후 RAW** | **2,399,800** | **37,140** | 0.38% |

수신량은 이상적인 수치(2,400,000)에 근접하게 회복되었다.
그러나 ≤200ms 성공률은 여전히 낮아 다음 문제로 이어졌다.

→ 여기까지는 동시성 문제 해결  
→ 이후 "왜 여전히 느린가?" 분석은 아래 참고

> 🔍 전체 분석 과정 보기 → [GitHub에서 보기](https://github.com/Kosw6/engineering-notes/blob/main/reports/GroupController/WebSocket.md)


---

## 문제 2: All-delivery 구조의 지연 누적

수신량 회복 이후에도 ≤200ms 성공률이 0.38%에 불과했다.
세션 직렬화 + 버퍼링으로 인해 모든 이벤트를 순서대로 전송하면 이전 이벤트가 밀려 최신 이벤트가 늦게 도달한다.

### 해결: 메시지 타입별 전략 분리

| 타입 | 전략 |
|------|------|
| CURSOR, HIGHLIGHT 등 (휘발성) | 최신값만 유지 (Coalescing) |
| CONTROL — 노드 이동·수정 확정 | 전량 전달 보장 (LinkedQueue) |

스케줄러(33ms, ≈30Hz)마다 Coalescer에서 최신값 flush.

---

## 문제 3: 더티 플래그 없는 Coalescer의 중복 재전송

초기 구현에서 Coalescer는 전송 후 맵을 비우지 않아,
**이미 보낸 최신값이 매 tick마다 반복 전송**되는 문제가 발생했다.

- 불필요한 브로드캐스트 폭증 → CPU/네트워크 낭비 → tail latency 악화
- ≤200ms 성공률: **48.55%** (수신량 자체는 오염됨)

### 해결: Dirty Flag + Drain

```java
// publish 시 dirty=true 마킹
public void publishLatest(String roomKey, String key, T msg) {
    latestByRoom.computeIfAbsent(roomKey, rk -> new ConcurrentHashMap<>()).put(key, msg);
    markDirty(roomKey);
}

// flush 시 dirty=true인 경우만 스냅샷 후 clear
public Collection<T> drainLatestIfDirty(String roomKey) {
    AtomicBoolean dirty = dirtyByRoom.get(roomKey);
    if (dirty == null || !dirty.compareAndSet(true, false)) return List.of();
    ArrayList<T> out = new ArrayList<>(latestByRoom.get(roomKey).values());
    latestByRoom.get(roomKey).clear();
    return out;
}
```

### 최종 결과 (Sender=20, Room=200, 20Hz)

| Type | Dirty Flag | Received | ≤200ms |
|------|-----------|---------|--------|
| RAW | ❌ 미적용 | 2,392,680 | 48.55% |
| STOMP | ❌ 미적용 | 2,396,660 | 48.21% |
| **RAW** | **✅ 적용** | **1,958,600** | **99.97%** |
| STOMP | ✅ 적용 | 2,028,600 | 98.69% |

---

## RAW vs STOMP 최종 비교 (Sender=40 고부하)

| Sender | Type | ≤200ms |
|--------|------|--------|
| 20 | RAW | 99.97% |
| 20 | STOMP | 98.69% |
| 40 | **RAW** | **89.89%** |
| 40 | STOMP | 78.04% |

고부하(송신자 40명) 구간에서 STOMP의 ≤200ms 성공률이 약 **12%p 하락**했다.
JFR 분석 결과 STOMP의 프레임 파싱·헤더 처리 오버헤드가 추가 allocation을 유발했다.

→ **RAW WebSocket 채택**

---

## 핵심 인사이트

> 휘발성 데이터에서 "전량 전달"과 "최신성 보장"은 반대되는 목표다.
> 전량 전달을 포기하고 Dirty Flag 기반 최신값만 flush하는 전략으로 ≤200ms 성공률을 48% → 99.97%로 회복했다.
> 중복 재전송 루프가 병목이었으며, 수신량 지표만 보면 문제가 보이지 않는다.

> 🔍 과정 상세히 보고 싶다면 → [GitHub에서 보기](https://github.com/Kosw6/engineering-notes/blob/main/reports/GroupController/WebSocket.md)