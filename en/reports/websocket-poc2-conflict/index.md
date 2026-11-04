---
layout: "single"
title: "WebSocket PoC 2 — Fallback State and Edit Conflicts"
permalink: "/en/reports/websocket-poc2-conflict/"
lang: "en"
locale: "en-US"
translation_url: "/reports/websocket-poc2-conflict/"
classes: "wide"
description: "Compared draft and server field changes to distinguish safe edits, merge candidates, and conflicts."
---


> **Historical policy:** AUTO_MERGE describes this PoC. Later content-edit rules require `MERGE_REVIEW_REQUIRED` when the server content changed, even for non-overlapping fields. Position-only changes that do not affect content context are distinguished. [Current contract](/en/reports/node-contract-refactoring/).

## Problem

When a shard fails and clients move to a fallback server, their draft may be based on an older server version. The system needs to preserve the draft and detect conflicting server changes.

## Draft model

| Field | Meaning |
|---|---|
| `baseVersion` | Server version when editing began |
| `dirtyFields` | Fields changed by the user |
| `serverChangedFieldsAfterEdit` | Server-side field changes while editing |

Kafka propagated server state changes, while Redis Draft tracked field history for the editing session.

## PoC classification

| Result | Historical rule |
|---|---|
| SAFE | No server change while editing |
| AUTO_MERGE | Server changes did not overlap the user's fields |
| CONFLICT | Both changed at least one of the same fields |

End-to-end scenarios verified routing to another server during shard failure, automatic merge for non-overlapping changes under the then-current rule, and conflict detection for overlapping changes.

## Interpretation

Failover routing alone cannot make an edit safe. The original version, the user's intent, and intervening server changes must be compared. The later review requirement strengthens content-context protection beyond field overlap.

[Next: server failback](/en/reports/websocket-poc3-failback/)


## Detailed evidence

[Read the full Korean report and its linked experiment logs](/reports/websocket-poc2-conflict/). This English edition presents the design, key measurements, and validation limits.
