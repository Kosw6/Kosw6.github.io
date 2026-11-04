---
layout: "single"
title: "Resume"
permalink: "/en/resume/"
lang: "en"
locale: "en-US"
translation_url: "/resume/"
classes: "wide portfolio-index portfolio-index--resume"
description: "Sungwon Kim — backend engineering, measured performance, realtime systems, and recovery verification."
---


## Profile

**Sungwon Kim · Backend Engineer**

I build realtime services and data pipelines, measure where they slow down or stop, and improve them. I verify solutions with repeatable load tests and failure recovery scenarios, recording processing state and results.

- **GitHub**: [github.com/Kosw6](https://github.com/Kosw6)
- **Portfolio**: [kosw6.github.io/en](https://kosw6.github.io/en/)

## Technical skills

| Area | Technologies |
|---|---|
| Backend | Spring Boot, Spring MVC, JPA/Hibernate, Spring Security |
| Languages | Java, Go (net/http, embed, goroutines, io.Writer), Python |
| Database | PostgreSQL, TimescaleDB (hypertables, chunk tuning, CAGG), indexing |
| Cache / Messaging | Redis (TTL design, JSON serialization), Kafka (consumer groups, offset replay) |
| Realtime | WebSocket, Redis Pub/Sub, server distribution and recovery |
| Data pipelines / Workers | DB Outbox, Kafka lag, raw/ETL separation, lineage, idempotency |
| Performance | k6, JFR/JMC, Prometheus/Grafana, Grafana Alert |
| DevOps | Docker, Terraform, GitHub Actions, AWS (EC2/S3/CloudFront/SSM/ASG/Lambda) |
| Frontend | React, Vite, React Flow, WebGL charts |

## Selected experience

### Load & Fault Test Orchestrator · Solo project · Apr – Jun 2026

A Go tool that connects infrastructure preparation, load generation, fault injection, and recovery checks into one repeatable scenario.

- Built a React/Wails workflow for defining steps, dependencies, infrastructure settings, and CLI commands, with YAML/ZIP export.
- Integrated the scenario runner and local UI into one desktop application without a separate engine binary.
- Scheduled independent steps concurrently with goroutines based on their dependencies.
- Saved execution logs to files while streaming them to the UI to expose progress and failure causes.
- Connected Terraform, k6, and AWS SSM to repeat Redis failure injection and recovery checks in a consistent order.

### Trader · Solo project · Jan 2025 – Present

[Source repositories and responsibilities](/en/projects/trader/#source-code)

An investment review service that grew from optimizing queries over roughly 26 million stock-price rows to realtime collaboration, recovery, KIS/BLS/SEC data pipelines, and Worker control.

**Performance**

- Compared ordinary tables and TimescaleDB hypertables with identical indexes for 90-day queries at 300 RPS. Chunk tuning reduced p95 from **7,247ms to 235ms**.
- Measured indexing separately at 10 RPS: a composite index reduced p95 from **342ms to 32ms**.
- Compared four query structures under the same k6 load and selected Fetch Join based on p95, GC, and allocation measurements.
- Compared full 10,000-character bodies with 20-character database previews. The preview sustained **about five times the RPS** in the same p95 range; full text moved to a detail endpoint.
- Traced allocation hot paths with JFR/JMC and removed repeated JWT validation. Old GC total time during a 90-second main load fell from **3.47s to 2.22s, about 36%**.

**Realtime systems**

- Eliminated same-session write collisions by serializing WebSocket sends per session.
- Reduced repeated transmissions and coalesced latest state, raising delivery within 200ms from **0.38% to 99.97%** after the write-collision fix.
- Distributed group connections across two instances, splitting send attempts from **159K to 79K + 79K**, and confirmed lower per-JVM GC and allocation pressure with JFR.
- Compared user edits with server changes by field to identify merge candidates and conflicts during failure scenarios.
- Verified a failback sequence in which a recovered server reaches a target Kafka offset before accepting traffic, followed by reconnection guidance and draining on the fallback server.

**Architecture and operations**

- Evaluated TimescaleDB Continuous Aggregates and designed 1W/1M/1Y query strategies; the connected user feature currently uses 1D data.
- Preserved raw KIS/BLS/SEC responses and built Python ETL Workers to populate normalized tables.
- Linked original inputs, transformation results, and consumer progress to trace delivery through ETL and identify duplicate processing.
- Verified Go Controller-driven AWS ASG Worker capacity changes of **0 → 1 → 0** using Kafka lag and heartbeat state.
- Confirmed both BLS objects accumulated during an ETL interruption were processed after restart.
- Built observability with Prometheus/Grafana and verified recovery through Grafana Alert, Lambda, SSM, and ASG.
- Implemented Google/Kakao JWT SSO.

### SIC club portal · Team lead · Sep – Nov 2025

Led a 14-person FE/BE/AI/Design team building a club operations website.

- Reorganized separate FE/BE sprints into feature-based work to make API dependencies and progress visible.
- Introduced GitHub Actions CI/CD and a JaCoCo coverage target of at least 70%.
- Temporarily relaxed the testing target when coursework delayed sprints, focusing on completing the MVP while investigating logs and clarifying API contracts.
- Diagnosed an EC2 t3.micro OOM issue with instance logs and `free -h`; moved to t3.medium and added 1GB swap.
- Built EC2/S3/CloudFront deployment automation and scheduled operating hours of 09:00–18:00 with EventBridge and Lambda to control costs.

## Education

- **Sejong University**

## Links

- [GitHub](https://github.com/Kosw6)
- [Portfolio](/en/)
- [Engineering reports](/en/reports/)
