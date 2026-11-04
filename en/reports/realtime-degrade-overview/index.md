---
layout: "single"
title: "Realtime Degraded Mode — Redis, Kafka, and Pub/Sub Failures"
permalink: "/en/reports/realtime-degrade-overview/"
lang: "en"
locale: "en-US"
translation_url: "/reports/realtime-degrade-overview/"
classes: "wide"
description: "Separated immediate propagation, durable recovery, and database state fallback to limit failure impact."
---


> **Historical experiment:** This report records an earlier propagation design. The later contract stages RELIABLE events in an Outbox first, adds server-local duplicate claims, and fixes a three-minute retry window from the first claim. Recovery after the historical 5-minute-38-second outage is not a final-delivery guarantee under the current policy. [Current contract](/en/reports/node-contract-refactoring/).

## Separate the paths

- **Immediate:** WebSocket response and Redis Pub/Sub fan-out, with tested gRPC fallback.
- **Durable:** Kafka event log and database Outbox recovery.
- **State:** PostgreSQL fallback for Redis-backed locks, autosave, and version hints.

Kafka is a recovery/audit path, while Redis handles normal low-latency propagation.

## Reliable event recovery {#reliable-event-recovery}

Redis Pub/Sub cannot replay events missed during a subscription interruption. In the historical design, reliable events also went asynchronously to Kafka. Each server consumed the full reliable log in its own group, such as `reliable-replay-app-1`, and checked a server/event processing key before retransmitting to local WebSocket sessions.

A shared consumer group would distribute events between servers, preventing each server from checking its own misses. Per-server groups and deduplication were therefore essential. When Redis was unavailable, retransmission was allowed with client-side event-ID deduplication, giving at-least-once behavior.

The tests stopped app-2's Pub/Sub listener and verified Kafka-driven lock-state recovery, skipped already propagated events in the normal path, and caught up from prior offsets after app-1 restarted.

## Redis state fallback

| Domain | Normal | Failure path |
|---|---|---|
| Lock | Redis Lua/TTL | PostgreSQL `canvas_lock` UNIQUE |
| Autosave | Redis TTL draft | PostgreSQL `edit_session` upsert |
| Version hint | Redis MGET | PostgreSQL `node_history` query |

In a 50-VU lock-contention test, one request succeeded and 49 were rejected, with **zero HTTP 5xx**. Timeout fallback also covered 500ms Redis latency. Replacing repeated GET calls with MGET improved version-hint p95 by about 44% in that experiment.

## Historical Kafka outage

During an outage of about **5 minutes 38 seconds**, events were preserved as pending Outbox records. After replay, all reached SENT, with final PENDING/FAILED counts of zero and no observed loss or duplicates in that test. The current retry deadline described above limits how this result can be interpreted.

## Relay comparison

| Path, about 500 ops/s | p95 max | Observed missing/dropped rate | Decision |
|---|---:|---:|---|
| gRPC | 14.5ms | About 0.5 ops/s | Redis-outage fallback |
| Redis Pub/Sub | 22.7ms | About 0 ops/s | Default broker fan-out |
| HTTP | 59.2ms | About 6 ops/s | Comparison only |

gRPC used persistent connections and Protobuf. Redis retained the simpler fan-out operating model as nodes increased. The decision balanced normal operation with failure recovery rather than choosing on latency alone.


## Detailed evidence

[Read the full Korean report and its linked experiment logs](/reports/realtime-degrade-overview/). This English edition presents the design, key measurements, and validation limits.
