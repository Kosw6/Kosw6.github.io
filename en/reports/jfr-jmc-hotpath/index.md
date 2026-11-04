---
layout: "single"
title: "JFR/JMC — Finding an Allocation Hot Path"
permalink: "/en/reports/jfr-jmc-hotpath/"
lang: "en"
locale: "en-US"
translation_url: "/reports/jfr-jmc-hotpath/"
classes: "wide"
description: "Removing duplicate JWT validation reduced Old GC total time by about 36% in the matched experiment."
---


## Background

After query improvements, GC remained higher than expected. I inspected JFR recordings in JMC using top allocating classes, stack traces, and GC summaries.

The earlier 10,000-character content path allocated many `byte[]` and `char[]` objects through JDBC decoding, with `PGStream.receiveTupleV3()` prominent in stack traces. A 20-character preview had already reduced allocation by about five times and GC to roughly half that earlier level.

## Duplicate JWT validation

I compared V1 (20-character preview plus Fetch Join) with V2 (the same design plus removal of repeated JWT validation). Stack traces exposed repeated authentication work and buffer allocation in the request path.

| Metric, 60 RPS and 90-second main load | V1 | V2 |
|---|---:|---:|
| Old GC total time | 3.47s | **2.22s** |
| Reduction | — | **About 36%** |

`BaseNCodec.ensureBufferSize` calls and `AbstractQueuedSynchronizer$ConditionNode` allocation also declined. This figure refers to Old GC total time under the stated test, rather than a 36% improvement in every latency or GC measure.

## Follow-up experiment

A two-step query, first loading Nodes and then Links, reduced the share of JDBC receive work but exposed object-graph assembly costs involving `Method`, `ResolvableType`, `Object[]`, and collections. The bottleneck moved again.

The useful practice was to follow allocation stack traces through the whole request rather than assuming that every remaining slowdown was a query problem.


## Detailed evidence

[Read the full Korean report and its linked experiment logs](/reports/jfr-jmc-hotpath/). This English edition presents the design, key measurements, and validation limits.
