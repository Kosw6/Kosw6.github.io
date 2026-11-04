---
layout: "single"
title: "Automating Redis Fault Injection and Recovery Checks"
permalink: "/en/reports/loadtest-orchestrator-redis-fault-validation/"
lang: "en"
locale: "en-US"
translation_url: "/reports/loadtest-orchestrator-redis-fault-validation/"
classes: "wide"
description: "Compared the same WebSocket load with a Redis outage and a no-fault baseline."
---


## Scenario

The orchestrator runs Terraform, substitutes output variables, checks health/authentication, starts a **50-VU WebSocket load for about 90 seconds**, injects Redis stop/start through AWS SSM, polls each command ID to Success, and checks recovery.

## Results

| Metric | v2: Redis fault | v3: no-fault baseline |
|---|---:|---:|
| Sent | About 26,988 | 26,990 |
| Received | About 34,020 | 89,564 |
| WebSocket errors | 0 | 0 |

Redis stop and start commands both reached SSM Success. The recovery health endpoint changed from **503 to 200** after Redis returned.

## Interpretation

Nearly equal sending load produced very different receiving counts: the baseline received about **2.6 times** as many messages. WebSocket connections stayed alive during the Redis outage, while Pub/Sub fan-out interruption reduced delivery.

Zero connection errors therefore did not imply that all collaboration messages arrived. Fault verification needed message counts, actual command results, and final health alongside connection metrics.

The automation replaces repeated manual container operations with one ordered scenario while leaving k6 responsible for load generation and measurement.

[Orchestrator project](/en/projects/loadtest-orchestrator/)


## Detailed evidence

[Read the full Korean report and its linked experiment logs](/reports/loadtest-orchestrator-redis-fault-validation/). This English edition presents the design, key measurements, and validation limits.
