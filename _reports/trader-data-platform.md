---
title: "Trader 데이터 파이프라인"
layout: single
permalink: /reports/trader-data-platform/
classes: wide
excerpt: "KIS, BLS, SEC 데이터를 raw로 보존하고 Kafka outbox, ETL lineage, 장애 복구, AWS ASG worker scaling까지 검증한 데이터 파이프라인"
tags: [go, python, kafka, etl, postgresql, timescaledb, aws, autoscaling]
---

> 외부 데이터 수집이 중단돼도 원본과 처리 위치를 확인하고, 필요한 범위만 다시 처리할 수 있게 만들었습니다.

투자 복기 서비스에 필요한 KIS 시세, BLS 거시경제, SEC 재무 데이터를 수집하고 정규화합니다. Go Controller가 작업과 Worker 실행을 제어하고 Python Worker가 외부 API 수집과 ETL을 담당합니다.

## 검증 결과 한눈에 보기

<div class="proof-strip">
  <div class="proof-item">
    <span class="proof-item__label">BLS 처리 흐름</span>
    <strong>작업 → 원본 → ETL → 연결 기록</strong>
    <span>원본 1건과 정규화 결과 49건 연결</span>
  </div>
  <div class="proof-item">
    <span class="proof-item__label">ETL 복구</span>
    <strong>대기 2건 → 처리 완료</strong>
    <span>Worker 재개 후 처리 상태 전환 확인</span>
  </div>
  <div class="proof-item">
    <span class="proof-item__label">Worker 자동 조절</span>
    <strong>0대 → 1대 → 0대</strong>
    <span>작업 발생 시 실행, 120초 유휴 후 종료</span>
  </div>
</div>

<nav class="page-quick-nav" aria-label="핵심 섹션 바로가기">
  <strong>빠르게 보기</strong>
  <a href="#architecture">구성</a>
  <a href="#failure-recovery">장애 복구</a>
  <a href="#aws-worker-scaling">AWS Worker ASG</a>
  <a href="#구현-범위와-한계">범위와 한계</a>
</nav>

| 영역 | 구현 및 검증 |
|---|---|
| 데이터 소스 | KIS 시세, BLS 매크로, SEC 재무 및 공시 |
| raw 보존 | `source_object`, `storage_key`, `content_hash`로 원본 추적 |
| 이벤트 전달 | DB outbox -> Kafka topic -> Python Worker |
| ETL 추적 | `record_lineage`로 raw와 정규화 레코드 연결 |
| 중복 방지 | `processed_event`, `idempotency_key`, DB commit 이후 Kafka offset commit |
| worker 제어 | Kafka lag와 heartbeat를 기준으로 AWS ASG scale-out/in |

## 문제

외부 API 응답을 바로 정규화 테이블에 넣으면 호출 실패, 잘못된 데이터, ETL 오류가 발생했을 때 원인을 재현하기 어렵습니다. 수집과 적재가 하나의 프로세스에 묶여 있으면 DB 부하를 미루기 어렵고, worker를 항상 켜 두는 구조라면 유휴 시간에도 비용이 발생합니다.

이 문제를 다음 세 가지 질문으로 나눴습니다.

1. 어떤 원본에서 정규화 레코드가 만들어졌는가?
2. Kafka 또는 worker가 중단되면 어느 상태부터 다시 시작해야 하는가?
3. 처리할 데이터가 있을 때만 worker node를 실행할 수 있는가?

## 구성 {#architecture}

<figure class="report-figure">
  <img src="/assets/images/data-platform/trader-data-architecture.svg" alt="Go Controller, Kafka, Python Collector와 ETL Worker, PostgreSQL과 raw storage로 구성한 Trader 데이터 파이프라인 아키텍처">
  <figcaption>Go는 control plane, Python은 data plane을 담당하며 DB가 처리 상태의 source of truth가 됩니다.</figcaption>
</figure>

| 컴포넌트 | 책임 |
|---|---|
| Go Controller | job 생성, outbox relay, raw-ready API, lag monitor, worker scale command |
| Python Collector | KIS/BLS/SEC API 호출, raw 저장, `source_object` 기록 |
| Python ETL Worker | raw 읽기, 정규화 테이블 upsert, lineage 기록 |
| Kafka | job event와 raw-ready event 전달, 미처리 작업량인 lag 제공 |
| PostgreSQL/TimescaleDB | job, outbox, raw 상태, lineage, consumer 처리 결과 저장 |

### raw와 ETL을 분리한 이유

```text
JOB_ITEM_QUEUED
-> Collector
-> raw storage
-> source_object COLLECTED
-> DB outbox RAW_OBJECT_READY
-> Kafka source topic
-> ETL Worker
-> normalized tables + record_lineage
-> source_object PROCESSED
```

- 외부 API 호출과 DB 적재 시간을 분리할 수 있습니다.
- raw를 먼저 보존해 ETL 오류를 같은 입력으로 재현할 수 있습니다.
- source별 Kafka lag를 기준으로 ETL worker만 실행할 수 있습니다.
- KIS, BLS, SEC의 처리량과 실패 영향 범위를 topic 단위로 나눌 수 있습니다.

## 운영 상태 모델

| 저장 위치 | 주요 상태 | 운영자가 판단하는 내용 |
|---|---|---|
| `pipeline_outbox` | PENDING, RETRY_WAIT, PUBLISHED, DEAD | Kafka 발행 요청이 어디까지 진행됐는가 |
| `source_object` | COLLECTED, PROCESSING, PROCESSED, FAILED | raw가 수집만 됐는가, ETL까지 끝났는가 |
| `processed_event` | SUCCESS, FAILED, SKIPPED | consumer가 메시지를 처리했고 중복 방어 기록을 남겼는가 |
| `record_lineage` | raw와 target record 연결 | 정규화 결과가 어떤 원본에서 만들어졌는가 |
| `kafka_consumer_lag_snapshot` | topic, group, total lag | source별 미처리 작업량이 얼마인가 |
| `pipeline_worker_node_heartbeat` | status, idle, idleSince | worker가 살아 있고 실제로 쉬고 있는가 |

이 상태를 관리자 페이지의 Pipeline Jobs, Kafka Pipeline, Raw Data, Worker Control 탭에서 함께 조회하도록 구성했습니다.

## 장애 복구 전략 {#failure-recovery}

<figure class="report-figure">
  <img src="/assets/images/data-platform/etl-recovery-flow.svg" alt="ETL transaction, consumer ledger transaction, Kafka offset commit 사이의 장애 위치에 따른 복구 흐름">
  <figcaption>ETL 결과와 처리 원장을 구간별로 확정하고 Kafka offset은 마지막에 commit해, 중단 위치에 따라 재처리 또는 보강할 수 있게 했습니다.</figcaption>
</figure>

```text
Kafka message 수신
-> processed_event SUCCESS 중복 확인
-> source_object PROCESSING 기록
-> processing state DB commit
-> raw 읽기
-> normalized table upsert + record_lineage insert
-> source_object PROCESSED
-> ETL transaction commit
-> processed_event SUCCESS 기록
-> consumer ledger transaction commit
-> Kafka offset commit
```

| 중단 위치 | 남는 상태 | 재수신 시 처리 |
|---|---|---|
| `PROCESSING` 기록 전 | Kafka offset 미커밋 | 다른 worker가 메시지를 다시 처리 |
| `PROCESSING` commit 후 ETL commit 전 | `PROCESSING`이 남을 수 있고 도메인 적재는 rollback | 재전달된 메시지로 ETL 재처리 |
| ETL commit 후 consumer ledger commit 전 | `PROCESSED`와 lineage 존재 | ETL 생략, `processed_event SUCCESS` 보강 |
| consumer ledger commit 후 Kafka commit 전 | `processed_event SUCCESS` 존재 | 중복 처리를 생략하고 Kafka offset commit |
| ETL 실패 | `FAILED`, 오류 메시지 | 자동 반복 대신 관리자 retry 또는 정책 적용 |

도메인 upsert, `source_object=PROCESSED`, lineage 기록은 하나의 ETL transaction으로 묶었습니다. `processed_event`는 별도의 consumer ledger transaction으로 기록하며, 두 commit 사이의 장애 구간은 `PROCESSED + lineage` 확인으로 보완합니다.

`PROCESSING`은 강한 분산 lock이 아니라 운영 관측 상태로 사용했습니다. 기본 복구는 Kafka consumer group과 미커밋 offset에 맡기고, 오래된 PROCESSING은 관리자 화면에서 확인하도록 범위를 정했습니다.

### 검증 1. Kafka 재시작 후 outbox 재발행

AWS 단일 broker 설정을 수정하고 Kafka를 재시작하는 과정에서 DB에는 `PUBLISHED`가 남았지만 topic에는 메시지가 없고 consumer 처리 기록도 없는 상태가 발생했습니다.

```text
pipeline_job_item 39 = QUEUED
pipeline_outbox 30 = PUBLISHED
processed_event JOB_ITEM_QUEUED:39 = 없음
trader.jobs.events lag = 0
```

DB outbox를 source of truth로 보고 해당 row를 `PENDING`으로 되돌렸습니다. Go outbox relay가 다시 발행했고 Python Worker가 소비한 뒤 BLS raw 수집과 ETL이 완료됐습니다.

<details class="evidence-block" markdown="1">
  <summary>복구 결과 SQL 및 로그 요약</summary>

```text
pipeline_job_item.id=39
status=COLLECTED

pipeline_outbox:
JOB_ITEM_QUEUED -> PUBLISHED, partition=0, offset=0
RAW_OBJECT_READY -> PUBLISHED, partition=0, offset=0

source_object.id=8775
processing_status=PROCESSED
processing_attempt_count=1

macro_series rows=1
macro_observation rows=24
macro_observation_vintage rows=24
record_lineage rows=49
```
</details>

### 검증 2. raw lag 누적 후 ETL 재개

BLS ETL worker를 중지한 상태에서 raw job 2건을 수집했습니다. raw는 `COLLECTED`로 보존되고 `trader.raw.bls.ready` lag가 2까지 증가했습니다. worker를 다시 실행하자 commit offset이 1에서 3으로 이동하고 lag가 0이 됐습니다.

<details class="evidence-block" markdown="1">
  <summary>lag 및 source_object 상태 전이</summary>

```text
Before:
total_lag=2
commitOffsets={"0":1}
latestOffsets={"0":3}
source_object 8777, 8778 = COLLECTED

After:
total_lag=0
commitOffsets={"0":3}
latestOffsets={"0":3}
source_object 8777, 8778 = PROCESSED
processing_attempt_count=1
```
</details>

## AWS Worker ASG 검증 {#aws-worker-scaling}

<figure class="report-figure">
  <img src="/assets/images/data-platform/asg-scaling-timeline.svg" alt="Kafka job lag 발생부터 AWS ASG scale-out, 작업 완료, idle timeout scale-in까지의 타임라인">
  <figcaption>최초 job topic을 기동 신호로 사용하고, 전체 topic lag와 idle 지속 시간을 scale-in 조건으로 사용했습니다.</figcaption>
</figure>

Python worker node가 없는 상태에서 관리자 페이지로 BLS job을 생성했습니다.

```text
trader.jobs.events lag=1
-> worker-scale-planner
-> SCALE_OUT command 14
-> aws-asg-actuator
-> ASG desired 0 -> 1
-> Python Worker가 job 소비
-> 전체 topic lag=0 + idle heartbeat 120초
-> SCALE_IN command 15
-> ASG desired 1 -> 0
```

| 검증 항목 | 결과 |
|---|---|
| Scale-out | `KAFKA_LAG_THRESHOLD_EXCEEDED`, command 14, SUCCEEDED |
| Worker 처리 | BLS job item 42, COLLECTED |
| Scale-in | `IDLE_TIMEOUT`, command 15, SUCCEEDED |
| ASG 상태 | Desired 1 -> 0, instance Terminating |

검증 중 scale planner가 `worker_role=ETL`만 조회해 job topic의 `COMBINED` 정책을 놓치는 문제를 발견했습니다. 조회 조건을 `ETL`, `COMBINED`로 확장했고, heartbeat에는 마지막 수신 시각과 별도로 `idleSince`를 추가해 실제 idle 지속 시간을 판단하도록 보완했습니다.

## 비용과 안정성의 경계

| 컴포넌트 | 운영 선택 | 이유 |
|---|---|---|
| DB | On-Demand | job, outbox, source_object, lineage의 source of truth |
| Kafka | On-Demand | 현재 단일 broker 검증 환경, 이벤트 버퍼 안정성 우선 |
| Go Controller | On-Demand | outbox relay, lag monitor, admin API를 지속 실행 |
| Python Worker | ASG, Spot 후보 | raw와 Kafka 상태를 기준으로 재처리할 수 있는 실행 노드 |

비용 절감은 모든 서버를 Spot으로 바꾸는 방식이 아니라, 중단돼도 다시 처리할 수 있는 worker만 탄력적으로 운영하는 방향으로 제한했습니다. Spot interruption watcher는 아직 구현하지 않았으며, Spot 전환 시 신규 poll 중단과 미완료 offset 미커밋 정책을 추가할 계획입니다.

## 운영자 가시성

| 관리자 화면 | 확인 내용 |
|---|---|
| Pipeline Jobs | job/item 상태, 실패 원인, retry 대상 |
| Kafka Pipeline | outbox 상태, topic별 lag, consumer 처리 결과 |
| Raw Data | 수집, Kafka 발행, ETL, consumer commit, lineage 연결 |
| Worker Control | 정책값, heartbeat, scale command, Local/AWS actuator |

화면은 단순 모니터링보다 복구 판단에 필요한 상태를 한곳에서 연결하는 데 목적을 두었습니다. Raw Data 상세에서는 수집만 된 데이터와 정규화까지 끝난 데이터를 구분하고, Worker Control에서는 왜 node가 실행되거나 종료됐는지 command reason으로 확인할 수 있습니다.

## 구현 범위와 한계

- KIS, BLS, SEC collector와 ETL 구조를 구현했습니다.
- AWS end-to-end와 raw lag 복구 증거는 BLS 단건 및 2건 시나리오로 남겼습니다.
- Kafka는 단일 broker smoke test 구성이며 replica/ISR 검증은 현재 범위가 아닙니다.
- ASG node 단위 scale-out/in은 검증했지만 개별 컨테이너 복구는 Docker restart policy와 node healthcheck 영역으로 분리했습니다.
- DB primary와 Kafka를 Spot으로 운영하는 HA 구성은 구현 범위에서 제외했습니다.

## Implementation Repositories

| Repository | Responsibility |
|---|---|
| [trader-controller](https://github.com/Kosw6/trader-controller) | Go 기반 job, outbox relay, Kafka lag 측정, Worker 정책 및 AWS ASG 제어 |
| [trader-data](https://github.com/Kosw6/trader-data) | Python Collector, raw 저장, ETL 정규화, lineage 및 consumer 처리 |

## Engineering Notes

- [문서 인덱스](https://github.com/Kosw6/engineering-notes/blob/main/reports/Data/00_TRADER_DATA_PLATFORM_INDEX.md)
- [데이터 파이프라인 운영 설계](https://github.com/Kosw6/engineering-notes/blob/main/reports/Data/01_DATA_PIPELINE_ARCHITECTURE.md)
- [장애 대응 및 복구 시나리오](https://github.com/Kosw6/engineering-notes/blob/main/reports/Data/02_FAILURE_RECOVERY_SCENARIOS.md)
- [AWS 배포 및 비용 최적화 설계](https://github.com/Kosw6/engineering-notes/blob/main/reports/Data/03_AWS_DEPLOYMENT_COST_OPTIMIZATION.md)
- [AWS Worker ASG 자동 확장 검증](https://github.com/Kosw6/engineering-notes/blob/main/reports/Data/04_AWS_WORKER_ASG_AUTO_SCALING_VALIDATION.md)

## 개선 과정에서 배운 점

수집과 정규화를 한 번의 작업으로 처리하면 구조가 단순하다고 생각했습니다. 하지만 DB 적재와 Kafka commit 사이에서 장애가 발생하면 원본 보존 여부와 재처리 범위를 확인하기 어려웠습니다.

이후 raw, 변환 상태, lineage, consumer 처리 위치를 각각 기록하고 Worker를 다시 생성할 수 있는 실행 노드로 분리했습니다. 비용 절감도 모든 서버를 줄이는 방식이 아니라, DB와 Kafka의 상태를 기준으로 복구 가능한 Worker에만 적용해야 한다는 기준을 세웠습니다.
