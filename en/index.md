---
layout: "splash"
title: "Sungwon Kim"
permalink: "/en/"
lang: "en"
locale: "en-US"
translation_url: "/"
classes: "wide"
description: "Sungwon Kim — backend engineering, measured performance, realtime systems, and recovery verification."
---


<div class="portfolio-home">
  <section class="home-profile" aria-labelledby="home-name">
    <span class="home-profile__role">Backend Engineer</span>
    <h1 id="home-name">Sungwon Kim</h1>
    <p class="home-profile__lead">I build realtime services and data pipelines, measure where they slow down or stop, and improve them.</p>
    <p class="home-profile__summary home-sentence-lines">
      <span>I develop realtime services with Spring Boot and data pipelines with Go and Python.</span>
      <span>I verify performance and recovery through repeatable load tests and failure scenarios.</span>
    </p>
    <nav class="home-profile__links" aria-label="Main links">
      <a href="/en/resume/">Resume</a><a href="/en/projects/">Projects</a><a href="https://github.com/Kosw6">GitHub</a>
    </nav>
  </section>
  <section class="home-primary-project" aria-labelledby="primary-project-title">
    <header class="home-section-heading"><span>Featured project</span><div>
      <h2 id="primary-project-title">Trader, an investment review service</h2>
      <p>Jan 2025 – Present · Solo project · 100% contribution</p>
    </div></header>
    <p class="home-primary-project__description home-sentence-lines">
      <span>I connected <strong>charts, investment journals, and a React Flow canvas</strong> so users can revisit their investment decisions and outcomes.</span>
      <span>The service collects and normalizes KIS market prices, BLS macroeconomic data, and SEC financial data. It also supports realtime collaborative graphs and gives operators visibility into data processing.</span>
    </p>
    <div class="home-project-scope" aria-label="Trader implementation scope">
      <div><span>User features</span><strong>Charts, journals, realtime canvas</strong><p>Spring Boot, React, WebSocket</p></div>
      <div><span>Data processing</span><strong>External data collection and ETL</strong><p>Go Controller, Python Worker, Kafka</p></div>
      <div><span>Operational validation</span><strong>Performance and failure recovery</strong><p>TimescaleDB, k6, AWS, Terraform</p></div>
    </div>
    <div class="home-primary-project__links">
      <a href="/en/projects/trader/">Explore the project →</a>
      <a href="https://github.com/Kosw6/trader-backend">Backend</a>
      <a href="https://github.com/Kosw6/trader-controller">Controller</a>
      <a href="https://github.com/Kosw6/trader-data">Data Pipeline</a>
    </div>
  </section>
  <section class="home-case-section" aria-labelledby="case-section-title">
    <header class="home-section-heading home-section-heading--intro"><span>Problem solving</span><div>
      <h2 id="case-section-title">Selected engineering cases</h2>
      <p>Five cases designed and verified against measured performance, behavioral contracts, and failure conditions.</p>
    </div></header>
    <div class="home-case-list">
      <article class="home-case">
<header class="home-case__label"><span>01</span><strong>Data pipelines</strong></header>
<div class="home-case__content"><h3>Tracking and resuming interrupted data processing</h3>
<p><b>Problem</b><span>KIS, BLS, and SEC collection and transformation can stop at different stages. Writing responses directly into normalized tables makes it difficult to tell whether the original input was preserved or which steps succeeded.</span></p>
<p><b>Decision</b><span>I made database records the basis for judging completion. Raw inputs, publication requests, and transformation results are linked before Kafka is used for asynchronous delivery and backlog measurement.</span></p>
<p class="home-case__result"><b>Result</b><span>The ETL restart test processed both pending BLS raw objects. In a separate ASG test, job lag triggered a Worker capacity change of <strong>0 → 1 → 0</strong>.</span></p>
<footer><span>Go, Python, Kafka, PostgreSQL, AWS ASG</span><a href="/en/reports/trader-data-platform/">Processing records and recovery →</a></footer></div></article>
<article class="home-case">
<header class="home-case__label"><span>02</span><strong>Realtime systems</strong></header>
<div class="home-case__content"><h3>WebSocket delivery within 200ms: 0.38% → 99.97%</h3>
<p><b>Problem</b><span>Concurrent sends to the same session caused write collisions and disconnects. Serializing writes fixed those errors, but repeated transmissions still left only 0.38% of messages arriving within 200ms.</span></p>
<p><b>Decision</b><span>I serialized writes per session, coalesced cursor and drag previews to their latest values, and kept committed edits on an ordered delivery path. Dirty flags stopped unchanged state from being sent repeatedly.</span></p>
<p class="home-case__result"><b>Result</b><span>Under the same load of 200 connected users and 20 senders, the share received within 200ms reached <strong>99.97%</strong>. A separate failback PoC verified catch-up before a recovered server rejoined service.</span></p>
<footer><span>Spring Boot, WebSocket, Redis Pub/Sub, Kafka</span><a href="/en/reports/websocket-group-canvas/">Performance analysis →</a></footer></div></article>
<article class="home-case">
<header class="home-case__label"><span>03</span><strong>Domain design</strong></header>
<div class="home-case__content"><h3>Aligning edits, history, and events through explicit contracts</h3>
<p><b>Problem</b><span>Conflict detection and history could use a different field set from the actual database update. Persistence and external propagation also shared responsibilities, obscuring what a successful request guaranteed.</span></p>
<p><b>Decision</b><span>I made the domain’s applied change the common source for history and events. Business success means an atomic Node, History, and Outbox commit; message conversion and post-commit propagation have separate responsibilities.</span></p>
<p class="home-case__result"><b>Result</b><span>Contract tests verify <strong>matching changed fields, no-op behavior, full rollback, and isolation of post-commit failures</strong>. External-system integration limits are documented separately.</span></p>
<footer><span>Java, Spring, JPA, Transactional Outbox, SOLID</span><a href="/en/reports/node-contract-refactoring/">Contracts and validation →</a></footer></div></article>
<article class="home-case">
<header class="home-case__label"><span>04</span><strong>Query performance</strong></header>
<div class="home-case__content"><h3>Stock chart query p95: 7,247ms → 235ms</h3>
<p><b>Problem</b><span>Users repeatedly browse 90-day price windows. At roughly 26 million rows, this API reached a p95 of 7,247ms at 300 RPS, missing its 300ms target.</span></p>
<p><b>Decision</b><span>I chose TimescaleDB to retain the PostgreSQL code and operating model while adding time-series partitioning. For a read-heavy workload with one daily batch of about 10,000 writes, I used a 90-day interval and four space partitions.</span></p>
<p class="home-case__result"><b>Result</b><span>At the same 90-day query range and 300 RPS, p95 fell from <strong>7,247ms to 235ms</strong>, meeting the response-time target.</span></p>
<footer><span>PostgreSQL, TimescaleDB, k6</span><a href="/en/reports/timescaledb-27x/">Query plans and measurements →</a></footer></div></article>
<article class="home-case">
<header class="home-case__label"><span>05</span><strong>Failure recovery</strong></header>
<div class="home-case__content"><h3>Matching recovery actions to failure types</h3>
<p><b>Problem</b><span>An application outage and sustained CPU pressure require different recovery actions. A failure in one container should not unnecessarily interrupt the entire service.</span></p>
<p><b>Decision</b><span>I restarted the affected container first for application failure, preserving the EC2 instance and reducing disruption to logs and WebSocket connections. Sustained CPU pressure instead triggered ASG scale-out.</span></p>
<p class="home-case__result"><b>Result</b><span>After the application stopped, the container restarted and monitoring returned to <strong>healthy</strong>. In the CPU test, ASG capacity increased from <strong>one to two instances</strong>.</span></p>
<footer><span>Prometheus, Loki, Lambda, AWS SSM, ASG</span><a href="/en/reports/auto-recovery-scaleout/">Recovery verification →</a></footer></div></article>
    </div>
  </section>
  <section class="home-other-projects" aria-labelledby="other-projects-title">
    <header class="home-section-heading"><span>Other experience</span><div><h2 id="other-projects-title">Other projects</h2></div></header>
    <div class="home-project-list">
      <article><div class="home-project-list__meta">Apr – Jun 2026 · Solo project</div><div><h3>Load &amp; Fault Test Orchestrator</h3><p>Connects repeated load tests, fault injection, and recovery checks through steps and dependencies. Users build scenarios in a graph UI, export a ZIP, and run it locally.</p></div><a href="/en/projects/loadtest-orchestrator/">Details →</a></article>
      <article><div class="home-project-list__meta">Sep – Nov 2025 · Team lead</div><div><h3>SIC club portal: team leadership and CI/CD</h3><p>Coordinated scope and schedules, developed the Spring Boot backend, established testing criteria, and built automated GitHub Actions deployment on AWS.</p></div><a href="/en/projects/sic-portal/">Details →</a></article>
    </div>
  </section>
  <section class="home-background" aria-labelledby="background-title">
    <header class="home-section-heading"><span>Background</span><div><h2 id="background-title">Skills and education</h2></div></header>
    <div class="home-background__grid"><div><h3>Core technologies</h3><dl>
      <div><dt>Backend</dt><dd>Java, Spring Boot, Go, Python</dd></div>
      <div><dt>Data</dt><dd>PostgreSQL, TimescaleDB, Redis, Kafka</dd></div>
      <div><dt>Infra</dt><dd>AWS, Docker, Terraform, Prometheus, Grafana</dd></div>
      <div><dt>Test</dt><dd>k6, JFR, JMC, Testcontainers</dd></div>
    </dl></div><div><h3>Education</h3>
      <div class="home-education"><p><span>Mar 2020 – Feb 2026</span><strong>Sejong University · Graduated</strong></p><p>Major in Environment, Energy and Geoinformatics; double major in Computer Engineering.</p></div>
      <div class="home-education"><p><span>Mar 2017 – Jan 2020</span><strong>Korea Digital Media High School · Graduated</strong></p></div>
    </div></div>
  </section>
  <section class="home-more" aria-labelledby="more-title">
    <header class="home-section-heading"><span>Supporting evidence</span><div><h2 id="more-title">More engineering reports</h2><p>Experiment conditions, alternatives, measurements, and limits.</p></div></header>
    <nav class="home-more__links" aria-label="Additional engineering reports">
      <a href="/en/reports/realtime-degrade-overview/#reliable-event-recovery"><strong>Recovering missed Pub/Sub events</strong><span>Redis for immediate propagation; Kafka for reliable event recovery</span></a>
      <a href="/en/reports/websocket-poc3-failback/"><strong>WebSocket server failback</strong><span>Catch up to a target Kafka offset before rejoining service</span></a>
      <a href="/en/reports/loadtest-orchestrator-redis-fault-validation/"><strong>Automated fault verification</strong><span>Repeat fault injection and recovery checks in the same order</span></a>
      <a href="/en/reports/jpa-tuning/"><strong>JPA query strategies</strong><span>N+1, Fetch Join, and Projection</span></a>
      <a href="/en/reports/jfr-jmc-hotpath/"><strong>JFR/JMC hot path analysis</strong><span>Duplicate JWT validation and allocation bottlenecks</span></a>
      <a href="/en/reports/slo-operating-capacity/"><strong>SLO-based capacity planning</strong><span>Compare load and resource usage</span></a>
      <a href="/en/reports/"><strong>All engineering reports</strong><span>Conditions, comparisons, and limitations</span></a>
    </nav>
    <div class="home-closing-links"><a href="/en/resume/">Resume</a><a href="/en/projects/">All Projects</a><a href="https://github.com/Kosw6">GitHub</a></div>
  </section>
</div>
