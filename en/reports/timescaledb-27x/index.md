---
layout: "single"
title: "TimescaleDB — Meeting the Chart Query p95 SLO"
permalink: "/en/reports/timescaledb-27x/"
lang: "en"
locale: "en-US"
translation_url: "/reports/timescaledb-27x/"
classes: "wide"
description: "Tuning indexed time-series queries over 26 million rows: p95 7,247ms → 235ms at 300 RPS."
---


## Problem and target

Trader renders 90-day stock-price windows from about 26 million OHLCV rows on PostgreSQL 17 with TimescaleDB. The workload is read-heavy, with a daily batch of roughly 10,000 writes. The target was **p95 < 300ms at 300 RPS**.

## Separate the index effect from partitioning

A missing `(symb, timestamp)` index initially caused full scans. A separate 10 RPS experiment reduced p95 from **342.14ms to 32.06ms** after adding it. That result is distinct from the following 300 RPS comparison, where indexes were identical.

The dump/restore process had also lost hypertable metadata. I checked `timescaledb_information.hypertables` and restored time-series partitioning.

## Chunk comparison

| Configuration | Average cold planning | Average execution |
|---|---:|---:|
| 30-day interval, no space partition | 49ms | 3ms |
| 90-day interval, eight space partitions | 31ms | 3ms |
| 90-day interval, four space partitions | 23ms | 3ms |

Matching the interval to the query reduced scanned chunks and cold planning overhead. Warm planning converged around 3ms, so the distinction mattered mainly for cold planning.

## Load-test results, identical indexes

| Structure | Target RPS | p95 | Achieved throughput | Failure rate |
|---|---:|---:|---:|---:|
| Ordinary PostgreSQL table | 300 | 7,247.28ms | 213.40 req/s | 0.00% |
| Hypertable: 90 days, eight partitions | 300 | 331.99ms | 300.01 req/s | 0.00% |
| Hypertable: 90 days, four partitions | 300 | **235.32ms** | 300.01 req/s | 0.00% |

I selected **90-day chunks and four space partitions**, prioritizing read performance and planner overhead for this workload.

## Query-frame scope

The user-facing 1D chart reads the hypertable directly. Weekly/monthly/yearly Continuous Aggregate queries, `(symb, bucket DESC)` indexing, and refresh after the daily batch were designed and evaluated, but are not yet connected user features.

The experiment showed why indexing and chunk pruning must be assessed separately under the actual query range and load.


## Detailed evidence

[Read the full Korean report and its linked experiment logs](/reports/timescaledb-27x/). This English edition presents the design, key measurements, and validation limits.
