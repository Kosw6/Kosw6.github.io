---
layout: "single"
title: "Contract-driven Refactoring of Node Changes, History, and Events"
permalink: "/en/reports/node-contract-refactoring/"
lang: "en"
locale: "en-US"
translation_url: "/reports/node-contract-refactoring/"
classes: "wide"
description: "One applied-change result, an atomic business commit, and explicit delivery/retry responsibilities."
---


## The contract question

When a Node edit succeeds, which states must agree, and where should external delivery failures be recovered? The risks below were identified in code; they are not all claims of observed production incidents.

## Applied changes and history {#change-contract}

Previously `dirtyFields` selected conflict/history fields while non-null DTO values selected saved fields. A request marked as title-only could also carry a body value, making stored state disagree with history.

The domain's actual applied change now supplies the common fields and version for Node, History, and Outbox.

| Condition | Required behavior |
|---|---|
| Declared fields and supplied values disagree | Reject the contract violation without changing state |
| Valid change | Use the applied fields and version for history and event |
| Validated same-value request | Succeed as a no-op, without a new version, history, or outbox event |
| Linked-note change | Verify ownership and modify only the caller's allowed links |
| Corrupt history | Fail validation rather than silently treating it as an empty change |

## Business success and post-commit work {#transaction-contract}

**Node state, History, and Outbox commit atomically.** Message conversion and propagation are separate responsibilities. `stage()` returning normally means an event obligation was recorded in the active transaction; it does not mean the transaction committed or an external system received it.

Post-commit hints and cache work must not redefine an already committed business result.

## Delivery and recovery {#delivery-contract}

| Boundary | Meaning | Does not guarantee |
|---|---|---|
| Live publish | Invoke TRANSIENT/VOLATILE live path | DB durability or every client's receipt |
| Outbox stage | Record delivery obligation in the active transaction | Commit or Kafka delivery |
| Business commit | Confirm state, history, and outbox together | Completion of external propagation |
| Kafka ACK | Broker acknowledges publication | Consumer processing or WebSocket receipt |
| Outbox SENT | Current claim owner records completion | Exactly-once behavior or full UI convergence |

Retries retain the same event ID and a fixed deadline of **three minutes from the first claim**. A claim token protects ownership; late acknowledgments and stale tokens require explicit handling. This policy supersedes older experiments that recovered after a longer outage.

## Responsibilities and SOLID

- SRP: separate live propagation, durable staging, Kafka publication, and state recording by change reason and failure responsibility.
- ISP: live callers do not depend on durable staging capabilities.
- OCP: execution-mode policies are separate implementations; configuration may still need changes when adding a mode.
- DIP: post-commit listeners depend on `CommittedEventPropagator` rather than a Redis implementation.
- LSP: valid live inputs were aligned, but Local propagates broadcaster exceptions while Multi isolates them; a fully uniform failure contract is not claimed.

## Validation and limits {#validation}

| Test | Contract |
|---|---|
| `NodeServiceContentUpdateTest` | Actual fields, history/events, no-op side effects |
| `NodeContentChangeTest` | Duplicate-link idempotency and read-only collection exposure |
| `NodeServiceTransactionContractTest` | Full rollback on outbox failure, post-commit work, save races |
| `NodeChangeStagingTransactionTest` | Reject calls without a transaction; commit/rollback notification behavior |
| `OutboxRetryWindowTest` | Fixed deadline, claim boundary, late ACKs, stale tokens |

These tests establish the stated domain and transaction contracts. They do not independently prove every Redis/Kafka/WebSocket integration behavior, exactly-once delivery, or complete screen convergence.

## Code evidence {#sources}

The Korean edition links the concrete classes, tests, and detailed examples. The design criterion is that each abstraction exposes what it guarantees and who recovers its failures.


## Detailed evidence

[Read the full Korean report and its linked experiment logs](/reports/node-contract-refactoring/). This English edition presents the design, key measurements, and validation limits.
