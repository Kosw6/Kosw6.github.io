---
layout: "single"
title: "WebSocket — Improving Collaborative Canvas Delivery"
permalink: "/en/reports/websocket-group-canvas/"
lang: "en"
locale: "en-US"
translation_url: "/reports/websocket-group-canvas/"
classes: "wide"
description: "Per-session serialization, latest-value coalescing, and dirty flags raised delivery within 200ms to 99.97%."
---


## Conditions

| Item | Setting |
|---|---|
| Connected users | 200 in one room |
| Senders | 20 users at 20Hz |
| Inbound events | 400 messages/s |
| Maximum fan-out | 80,000 sends/s |
| SLO metric | Share received within 200ms |

## 1. Serialize writes per session

Concurrent `sendMessage()` calls caused `TEXT_PARTIAL_WRITING`, disconnects, and room failures. `ConcurrentWebSocketSessionDecorator` provided session-local buffering and send-time/buffer limits. After removing these errors, received messages increased from 6,509 to 2,399,800, but the share within 200ms was still only **0.38%**.

## 2. Match delivery to message semantics

Delivering every cursor or preview update allowed stale state to queue up. I coalesced volatile cursor/highlight state to its latest value and kept committed node changes on an ordered queue.

## 3. Send only changed state

Coalescing alone still resent unchanged values on each flush. Dirty flags and draining prevented duplicate broadcasts and reduced CPU, network, and tail-latency pressure.

| Protocol | Dirty flags | Received | Within 200ms |
|---|---|---:|---:|
| RAW | No | 2,392,680 | 48.55% |
| STOMP | No | 2,396,660 | 48.21% |
| RAW | Yes | 1,958,600 | **99.97%** |
| STOMP | Yes | 2,028,600 | 98.69% |

The no-dirty-flag counts include repeated transmissions. The headline 0.38% baseline is after write serialization and before the transmission improvements.

## Higher-load limit

At 40 senders, RAW reached 89.89% within 200ms and STOMP 78.04%. The 99.97% result therefore applies to the stated 20-sender scenario.

[Next: group sharding](/en/reports/websocket-poc1-sharding/)


## Detailed evidence

[Read the full Korean report and its linked experiment logs](/reports/websocket-group-canvas/). This English edition presents the design, key measurements, and validation limits.
