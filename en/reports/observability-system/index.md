---
layout: "single"
title: "Observability — Tracing Requests, Domains, and Events"
permalink: "/en/reports/observability-system/"
lang: "en"
locale: "en-US"
translation_url: "/reports/observability-system/"
classes: "wide"
description: "Structured logs, trace IDs, Loki/Grafana dashboards, and alerts connect failures to request flows."
---


## Problem

Plain log strings made it difficult to identify the request, domain, instance, or event behind an error. The operational questions were which API failed, whether Redis/Kafka/DB was involved, and whether a reproduced failure could be followed with one trace ID.

## Design

`TraceIdFilter`, MDC, and AOP produce structured logs. Promtail sends logs to Loki; Grafana connects them with Prometheus resource metrics, latency dashboards, error counts/rates, and alerts.

| Field | Purpose |
|---|---|
| `traceId` | Follow one request |
| `domain` | Identify the business area |
| `instance` | Identify the server |
| `event` | Name the observed operation |
| `level`, `errorCode` | Filter severity and failure type |
| `elapsedMs`, `uri` | Connect timing to the request path |

## Why Loki

I compared ELK and Loki at idle, 150 RPS, and 500 RPS. ELK consumed more baseline CPU/memory and showed greater resource growth under this workload. Loki remained comparatively stable and integrated naturally with the existing Grafana workflow.

The tradeoff is search capability: ELK supports richer full-text search and analysis, while Loki relies more on labels and filters. Loki's disk writes grow with log volume, so lower indexing overhead does not remove storage cost.

## Result

The system exposes per-instance log collection, domain-level ERROR counts, rate-based alerts, trace-ID search, API latency and log correlation, and resource metrics alongside events. This supplies the evidence for deciding when and how to recover.

[Capacity planning](/en/reports/slo-operating-capacity/) · [Automatic recovery](/en/reports/auto-recovery-scaleout/)


## Detailed evidence

[Read the full Korean report and its linked experiment logs](/reports/observability-system/). This English edition presents the design, key measurements, and validation limits.
