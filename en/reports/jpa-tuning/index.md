---
layout: "single"
title: "JPA — Query Strategies and Payload Costs"
permalink: "/en/reports/jpa-tuning/"
lang: "en"
locale: "en-US"
translation_url: "/reports/jpa-tuning/"
classes: "wide"
description: "Compared query shapes, object allocations, and database-side previews under matched k6 load."
---


## Problem and method

The canvas node-list API needed a stable query strategy and smaller list payloads. I used k6, p95, GC, allocation profiles, and Hibernate behavior to compare strategies on a 4-core/16GB environment with PostgreSQL 17 and TimescaleDB.

The strategy comparison used a 30 RPS two-minute warmup and a 120 RPS 90-second main load. Payload experiments increased RPS separately.

## Query strategies

Lazy loading caused N+1 queries. Batch fetching partly reduced them. Fetch Join minimized round trips and was the most stable tested option: list p95 improved from **2,551ms to 413ms** at 120 RPS.

Database JSON aggregation reduced row counts and looked reasonable in `EXPLAIN ANALYZE`, but worsened end-to-end p95. A favorable query plan alone did not establish an API improvement.

The later comparison covered JSON aggregation over three tables, Projection with 500-character or 20-character content over two tables, and Fetch Join over two tables. In this fixture, Fetch Join reused parent Node entities through Hibernate's first-level cache, whereas Projection created a DTO for every row, increasing allocation pressure. This is a workload-specific result, not a universal rule about projections.

## Payload experiment

| Approach | Behavior |
|---|---|
| Full body | Retrieve and return 10,000 characters |
| Application preview | Retrieve full body, then truncate to 20 characters |
| Database preview | Retrieve only `substring(content, 1, 20)` |

The full-body and application-truncation variants degraded around **26 RPS**. Database previews remained stable at **120 RPS**, sustaining about five times the RPS in the same p95 range.

## Decision

Keep the measured Fetch Join approach, return database-side 20-character previews for lists, and load full bodies through a separate detail API. The bottleneck shifted from database round trips to allocations and serialization, so each change needed end-to-end measurement.

[Runtime profiling](/en/reports/jfr-jmc-hotpath/)


## Detailed evidence

[Read the full Korean report and its linked experiment logs](/reports/jpa-tuning/). This English edition presents the design, key measurements, and validation limits.
