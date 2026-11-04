---
layout: "single"
title: "WebSocket PoC 1 — Group-based Sharding"
permalink: "/en/reports/websocket-poc1-sharding/"
lang: "en"
locale: "en-US"
translation_url: "/reports/websocket-poc1-sharding/"
classes: "wide"
description: "Distributed group-local fan-out across two servers and measured per-JVM allocation and GC."
---


## Problem and setup

Fan-out concentrated on one server increases broadcast cost. This PoC assigned each group to one server, keeping that group's broadcast local.

The test used **100 users in two groups of 50**, with ten senders per group at 10Hz. Total inbound sending was about 200 events/s.

## Results

| Metric | One instance | Sharded instance 1 | Sharded instance 2 |
|---|---:|---:|---:|
| Send attempts | 159,317 | **79,776** | **79,759** |
| GC count | 3 | 1 | 1 |
| `byte[]` allocation | 205 MiB | 93.5 MiB | 111 MiB |
| String allocation | 33.4 MiB | 19 MiB | 16 MiB |

Groups 1 and 32 routed to different instances. Each broadcast internally without inter-server event forwarding in this test. Per-instance send load was roughly halved, and JFR/JMC showed lower GC and allocation pressure per JVM.

This does not mean aggregate allocation across all servers was halved. It demonstrates distribution of work and reduced pressure on each instance for the specified grouping.

## Follow-up

Placement alone does not define state consistency after a shard fails. Subsequent PoCs covered [field-level conflict handling](/en/reports/websocket-poc2-conflict/) and [catch-up before failback](/en/reports/websocket-poc3-failback/).


## Detailed evidence

[Read the full Korean report and its linked experiment logs](/reports/websocket-poc1-sharding/). This English edition presents the design, key measurements, and validation limits.
