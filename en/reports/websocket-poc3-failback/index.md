---
layout: "single"
title: "WebSocket PoC 3 — Kafka Catch-up, Drain, and Failback"
permalink: "/en/reports/websocket-poc3-failback/"
lang: "en"
locale: "en-US"
translation_url: "/reports/websocket-poc3-failback/"
classes: "wide"
description: "Replayed three events to a target offset before returning a recovered server to service."
---


## Problem

A recovered server being alive does not mean it has processed changes made during its outage. The PoC gated failback on reaching a target Kafka offset, then guided clients through reconnection and draining.

## Lifecycle

| State | Meaning |
|---|---|
| UP | Alive, but not ready to broadcast |
| HEALTHY | Alive and ready |
| DRAINING | Reject new connections while handling existing sessions |
| DOWN | Unresponsive |

Broadcast and Catch-up consumers use separate group IDs so replay does not interfere with live consumption. The Gateway coordinates lifecycle transitions rather than letting servers make conflicting independent decisions.

## Verified sequence

1. ws-1 fails; ws-2 takes over and continues processing.
2. ws-1 restarts and replays retained events up to the recovery target offset.
3. The test records **`replayCount=3`** and confirms the target offset is reached.
4. ws-2 sends reconnection guidance and drains sessions.
5. Clients reconnect through the Gateway to the recovered server.

## Scope and subsequent design

This verifies the stated recovery sequence and target offset in a PoC. It does not establish an unconditional guarantee for arbitrary ongoing traffic or all client screen states.

Subsequent propagation separates Redis Pub/Sub for normal fan-out and Kafka for reliable-event durability and missed-event recovery. Each server independently consumes reliable events through `reliable-replay-${instanceId}` and deduplicates by server/event identity. Retry policy and current contract limits are documented separately.

[Reliable-event recovery](/en/reports/realtime-degrade-overview/#reliable-event-recovery) · [Current contracts](/en/reports/node-contract-refactoring/)


## Detailed evidence

[Read the full Korean report and its linked experiment logs](/reports/websocket-poc3-failback/). This English edition presents the design, key measurements, and validation limits.
