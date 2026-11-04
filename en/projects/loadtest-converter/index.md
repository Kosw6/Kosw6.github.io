---
layout: "single"
title: "LoadTest Converter — Early Orchestrator Stage"
permalink: "/en/projects/loadtest-converter/"
lang: "en"
locale: "en-US"
translation_url: "/projects/loadtest-converter/"
classes: "wide"
description: "Sungwon Kim — backend engineering, measured performance, realtime systems, and recovery verification."
---


> Now integrated into the [Load & Fault Test Orchestrator](/en/projects/loadtest-orchestrator/) as its scenario authoring module. This page records the earlier YAML/ZIP builder.

## Overview

A React/Vite UI and Go `net/http` API let users assemble k6 scenarios without manually writing YAML and script templates. The UI supports import, preview, and ZIP export. The API and static UI were deployed separately on Render.

## Design

Embedded templates (`//go:embed`) ship inside the Go binary. `archive/zip` and `text/template` package the scenario without separately deployed template files.

```text
scenario.zip
├── scenario.yml
├── users/{step}.csv
├── params/{step}.json
└── templates/
```

| Endpoint | Responsibility |
|---|---|
| `POST /api/convert/import` | Parse YAML into UI state |
| `POST /api/convert/preview` | Return generated YAML |
| `POST /api/convert/export` | Return the packaged ZIP |

## Scenario model

Step types include `k6`, `auth`, `command`, and `final_check`. Load modes support RPS, duration, and total request count. `depends_on` expresses dependencies for wave-based parallel execution.

The stateless public API used CORS configuration; the frontend received its API endpoint through `VITE_API_URL`. The original deployment used a Render Go Web Service and Static Site.

[Source repository](https://github.com/Kosw6/loadtest-converter)
