---
layout: "single"
title: "LoadTest Desktop — Orchestrator Runner Stage"
permalink: "/en/projects/loadtest-desktop/"
lang: "en"
locale: "en-US"
translation_url: "/projects/loadtest-desktop/"
classes: "wide"
description: "Sungwon Kim — backend engineering, measured performance, realtime systems, and recovery verification."
---


> Now integrated into the [Load & Fault Test Orchestrator](/en/projects/loadtest-orchestrator/) as its local UI and execution engine. This page records the engine integration and log-streaming work.

## Overview

A Go/Wails v2 and React application loads a scenario ZIP and runs its k6 workflow. Initially the user selected a separate engine executable. I moved the engine into an internal package so the UI and engine ship as one `.exe`.

External programs used by a scenario, including k6, Terraform, and AWS CLI, still need to be installed separately.

## Design decisions

- `engine.Run(cfg, w io.Writer)` decouples the engine from a terminal-specific progress display.
- A mutex-protected Wails writer buffers output and emits complete lines as `run:log` events for the frontend.
- `io.MultiWriter` sends subprocess output to both a log file and the live UI.
- Dependency-ready steps run concurrently in goroutines; each wave completes before the next starts.
- Embedded templates are extracted to a temporary directory at runtime to reuse path-based execution code.

## Supported steps

| Type | Behavior |
|---|---|
| `k6` | Generate and run a script; save `summary.json` |
| `auth` | Log in users from CSV; save authentication context |
| `command` | Run OS-specific shell commands |
| `final_check` | Assert the final HTTP endpoint state |

## Execution

```text
Load ZIP → parse scenario.yml → calculate dependency waves
  → authenticate → run independent load steps concurrently
  → save summary.json and k6.log → generate Markdown/HTML reports
```

[Historical repository](https://github.com/Kosw6/loadtest-desktop)
