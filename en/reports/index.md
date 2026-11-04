---
layout: "single"
title: "Engineering Reports"
permalink: "/en/reports/"
lang: "en"
locale: "en-US"
translation_url: "/reports/"
classes: "wide portfolio-index portfolio-index--wide"
description: "Sungwon Kim — backend engineering, measured performance, realtime systems, and recovery verification."
---

The conditions behind each problem, the reasons for the design choices, and the measured results. English editions cover key decisions, measurements, and limits; full experiment logs remain linked through the Korean originals.

## Domain design and consistency

<div class="report-grid">
<a class="report-card" href="/en/reports/node-contract-refactoring/"><span class="report-card__eyebrow">Domain design and consistency</span><h3>Contract-driven Refactoring of Node Changes, History, and Events</h3><p>One applied-change result, an atomic business commit, and explicit delivery/retry responsibilities.</p><span class="report-card__result">Read report →</span></a>
</div>

## Data processing and operations

<div class="report-grid">
<a class="report-card" href="/en/reports/trader-data-platform/"><span class="report-card__eyebrow">Data processing and operations</span><h3>Trader Data Pipelines — Traceable Processing and Recovery</h3><p>Raw preservation, ETL lineage, database outbox delivery, and Kafka-lag-driven AWS Worker control.</p><span class="report-card__result">Read report →</span></a>
<a class="report-card" href="/en/reports/observability-system/"><span class="report-card__eyebrow">Data processing and operations</span><h3>Observability — Tracing Requests, Domains, and Events</h3><p>Structured logs, trace IDs, Loki/Grafana dashboards, and alerts connect failures to request flows.</p><span class="report-card__result">Read report →</span></a>
<a class="report-card" href="/en/reports/slo-operating-capacity/"><span class="report-card__eyebrow">Data processing and operations</span><h3>SLO-based Capacity Planning — Threads and Connections</h3><p>Selected a workload-specific 2 vCPU/4GB setup with 30 threads and eight Hikari connections.</p><span class="report-card__result">Read report →</span></a>
<a class="report-card" href="/en/reports/auto-recovery-scaleout/"><span class="report-card__eyebrow">Data processing and operations</span><h3>Automatic Recovery — Container Restart, Scale-out, and Routing</h3><p>Connected Grafana alerts to Lambda, SSM, and ASG, and verified failure-aware Gateway routing.</p><span class="report-card__result">Read report →</span></a>
<a class="report-card" href="/en/reports/loadtest-orchestrator-redis-fault-validation/"><span class="report-card__eyebrow">Data processing and operations</span><h3>Automating Redis Fault Injection and Recovery Checks</h3><p>Compared the same WebSocket load with a Redis outage and a no-fault baseline.</p><span class="report-card__result">Read report →</span></a>
<a class="report-card" href="/en/reports/realtime-degrade-overview/"><span class="report-card__eyebrow">Data processing and operations</span><h3>Realtime Degraded Mode — Redis, Kafka, and Pub/Sub Failures</h3><p>Separated immediate propagation, durable recovery, and database state fallback to limit failure impact.</p><span class="report-card__result">Read report →</span></a>
</div>

## Performance

<div class="report-grid">
<a class="report-card" href="/en/reports/timescaledb-27x/"><span class="report-card__eyebrow">Performance</span><h3>TimescaleDB — Meeting the Chart Query p95 SLO</h3><p>Tuning indexed time-series queries over 26 million rows: p95 7,247ms → 235ms at 300 RPS.</p><span class="report-card__result">Read report →</span></a>
<a class="report-card" href="/en/reports/jpa-tuning/"><span class="report-card__eyebrow">Performance</span><h3>JPA — Query Strategies and Payload Costs</h3><p>Compared query shapes, object allocations, and database-side previews under matched k6 load.</p><span class="report-card__result">Read report →</span></a>
<a class="report-card" href="/en/reports/jfr-jmc-hotpath/"><span class="report-card__eyebrow">Performance</span><h3>JFR/JMC — Finding an Allocation Hot Path</h3><p>Removing duplicate JWT validation reduced Old GC total time by about 36% in the matched experiment.</p><span class="report-card__result">Read report →</span></a>
</div>

## Realtime processing and recovery

<div class="report-grid">
<a class="report-card" href="/en/reports/websocket-group-canvas/"><span class="report-card__eyebrow">Realtime processing and recovery</span><h3>WebSocket — Improving Collaborative Canvas Delivery</h3><p>Per-session serialization, latest-value coalescing, and dirty flags raised delivery within 200ms to 99.97%.</p><span class="report-card__result">Read report →</span></a>
<a class="report-card" href="/en/reports/websocket-poc1-sharding/"><span class="report-card__eyebrow">Realtime processing and recovery</span><h3>WebSocket PoC 1 — Group-based Sharding</h3><p>Distributed group-local fan-out across two servers and measured per-JVM allocation and GC.</p><span class="report-card__result">Read report →</span></a>
<a class="report-card" href="/en/reports/websocket-poc2-conflict/"><span class="report-card__eyebrow">Realtime processing and recovery</span><h3>WebSocket PoC 2 — Fallback State and Edit Conflicts</h3><p>Compared draft and server field changes to distinguish safe edits, merge candidates, and conflicts.</p><span class="report-card__result">Read report →</span></a>
<a class="report-card" href="/en/reports/websocket-poc3-failback/"><span class="report-card__eyebrow">Realtime processing and recovery</span><h3>WebSocket PoC 3 — Kafka Catch-up, Drain, and Failback</h3><p>Replayed three events to a target offset before returning a recovered server to service.</p><span class="report-card__result">Read report →</span></a>
</div>

## Suggested reading

Start with the [Trader project](/en/projects/trader/) for the service and architecture, then explore the cases that interest you.
