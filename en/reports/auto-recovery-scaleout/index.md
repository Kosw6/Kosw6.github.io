---
layout: "single"
title: "Automatic Recovery — Container Restart, Scale-out, and Routing"
permalink: "/en/reports/auto-recovery-scaleout/"
lang: "en"
locale: "en-US"
translation_url: "/reports/auto-recovery-scaleout/"
classes: "wide"
description: "Connected Grafana alerts to Lambda, SSM, and ASG, and verified failure-aware Gateway routing."
---


## Match the action to the failure

| Failure | Detection | Action | Verification |
|---|---|---|---|
| Application down | Grafana alert | Lambda → SSM → Docker Compose restart | Prometheus scrape returns UP |
| High CPU | Grafana alert | Lambda → ASG desired capacity 1 → 2 | New EC2 instance joins scraping |
| Gateway routing | Health polling | Route by healthy/draining state | Failover and failback |

Restarting only the affected container avoids replacing EC2 immediately, reducing recovery time and disruption to logs and connections. Sustained CPU pressure needs more capacity rather than a restart.

## Validation boundary

The CPU automation test deliberately used a low threshold of **`process_cpu_usage > 0.002`** to exercise the path. It did not establish a production CPU threshold. Production cooldowns, instance replacement after repeated restart failure, and scale-in policy remain separate decisions.

## Gateway failover and failback

The routing model distinguishes HEALTHY, DRAINING, and DOWN. Draining rejects new connections while existing sessions are handled separately.

| Time in test | Observation |
|---|---|
| Initial | Existing HOLD and new PROBE sessions use preferred instance A |
| 19s | A fails; existing session sees close code 1006 |
| 23s | Health poller excludes A; new traffic fails over to B |
| 45s | Connections opened during the outage still use B |
| 90s | A recovers; new connections return to A |
| 135s | New connections remain on A; existing B sessions stay on B |

The observed detection-to-routing change was about four seconds. New-connection failback and migration of existing sessions are distinct operations.

The central lesson was to verify not just an alert, but successful command execution, recovered health, and the new instance's entry into service.


## Detailed evidence

[Read the full Korean report and its linked experiment logs](/reports/auto-recovery-scaleout/). This English edition presents the design, key measurements, and validation limits.
