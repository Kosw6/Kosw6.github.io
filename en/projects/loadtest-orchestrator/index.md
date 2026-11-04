---
layout: "single"
title: "Load & Fault Test Orchestrator"
permalink: "/en/projects/loadtest-orchestrator/"
lang: "en"
locale: "en-US"
translation_url: "/projects/loadtest-orchestrator/"
classes: "wide"
description: "Sungwon Kim — backend engineering, measured performance, realtime systems, and recovery verification."
---


A Go-based tool for building scenarios in a graph UI and repeatedly running infrastructure preparation, load tests, fault injection, and recovery checks on each user's PC.

## Problem and direction

Trader's Redis/Kafka and WebSocket experiments repeatedly required checking Terraform outputs, launching k6, injecting a fault, and inspecting health and logs. As steps grew, execution order and verification checks became easy to miss.

I separated scenario authoring from load generation. A configuration server exports YAML, templates, and settings as a ZIP; a local Wails application runs it. This reduces contention between users on a shared load-generation server. Results from different PCs are not directly comparable because hardware and networks differ.

## Execution flow

```text
Graph UI → scenario.zip → local application
  → Terraform apply → output substitution → health/auth checks
  → k6 WebSocket load → AWS SSM fault injection
  → final recovery checks → result comparison
```

## Implementation

| Area | Technology |
|---|---|
| Authoring | React graph UI, YAML/ZIP export |
| Local runner | Go, Wails, React |
| Load | k6, WebSocket |
| Infrastructure | Terraform, AWS EC2 |
| Fault injection | AWS SSM |
| Reports | Markdown/HTML result collection |

The local UI and Go execution engine are integrated in one application. The engine schedules independent dependency-ready steps using goroutines. An `io.Writer` interface supports concurrent log streaming to the UI and log files. Templates are embedded in the binary; external tools such as k6, Terraform, and AWS CLI must still be installed.

## Redis fault verification

| Metric | v2: Redis fault | v3: no-fault baseline |
|---|---:|---:|
| Sent | About 26,988 | 26,990 |
| Received | About 34,020 | 89,564 |
| Errors | 0 | 0 |

Connections survived, but Redis Pub/Sub fan-out interruption reduced received messages. The baseline received about 2.6 times as many. Connection survival alone does not prove correct message delivery.

[Fault validation report](/en/reports/loadtest-orchestrator-redis-fault-validation/)

## Repositories

- [loadtest-orchestrator](https://github.com/Kosw6/loadtest-orchestrator) — local UI and execution engine
- [loadtest-converter](https://github.com/Kosw6/loadtest-converter) — graph authoring and ZIP generation

The [Converter](/en/projects/loadtest-converter/) and [Desktop](/en/projects/loadtest-desktop/) pages document earlier implementation stages of this integrated project.
