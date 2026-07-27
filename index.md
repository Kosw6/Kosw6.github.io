---
layout: splash
title: "Sungwon Kim"

excerpt: >
  **성능 병목을 측정하고, 장애 이후에도 복구 가능한 백엔드 플랫폼을 만듭니다.**<br>
  실시간 이벤트 처리부터 데이터 파이프라인, 운영 자동화까지 직접 설계하고 검증했습니다.

header:
  overlay_image: /assets/images/hero-bg-1.png
  overlay_filter: 0.6
  overlay_color: "#000000"
  actions:
    - label: "대표 프로젝트"
      url: "#featured-projects"
      btn_class: "btn--primary"
    - label: "Engineering Reports"
      url: "/reports/"
      btn_class: "btn--inverse"
---

<div class="home-proof-strip" aria-label="대표 검증 결과">
  <div class="home-proof-item">
    <span>TimescaleDB 조회</span>
    <strong>p95 7,247ms → 235ms</strong>
    <small>약 2,600만 행, 300 RPS SLO 달성</small>
  </div>
  <div class="home-proof-item">
    <span>WebSocket 실시간 처리</span>
    <strong>≤200ms 99.97%</strong>
    <small>동시성 제어와 전송 구조 개선</small>
  </div>
  <div class="home-proof-item">
    <span>AWS Python Worker</span>
    <strong>ASG 0 → 1 → 0</strong>
    <small>Kafka lag 기반 자동 확장과 회수</small>
  </div>
</div>

<section class="home-section" id="featured-projects">
  <div class="home-section__header">
    <div>
      <span class="home-section__eyebrow">FEATURED PROJECTS</span>
      <h2>대표 프로젝트</h2>
    </div>
    <p>서비스 구현에 그치지 않고 성능 측정, 장애 복구, 운영 제어까지 연결한 프로젝트입니다.</p>
  </div>

  <div class="home-menu-list home-menu-list--projects">
    <a class="home-menu-item" href="{{ '/projects/trader/' | relative_url }}">
      <div class="home-menu-item__title">
        <span>PROJECT 01</span>
        <h3>Trader 주식투자 복기 플랫폼</h3>
      </div>
      <p class="home-menu-item__description">실시간 협업 캔버스와 투자 일지, 시계열 차트를 제공하고 KIS, BLS, SEC 데이터를 수집·정규화하는 플랫폼입니다.</p>
      <div class="home-menu-item__outcome">
        <strong>WebSocket · TimescaleDB · Kafka ETL</strong>
        <span>프로젝트 보기 <b aria-hidden="true">→</b></span>
      </div>
    </a>

    <a class="home-menu-item" href="{{ '/projects/loadtest-orchestrator/' | relative_url }}">
      <div class="home-menu-item__title">
        <span>PROJECT 02</span>
        <h3>Go 기반 부하·장애 검증 오케스트레이터</h3>
      </div>
      <p class="home-menu-item__description">사용자가 단계, 의존성, 인프라를 정의하면 Terraform, k6, AWS SSM 기반 검증 흐름을 CLI와 UI에서 반복 실행합니다.</p>
      <div class="home-menu-item__outcome">
        <strong>검증 준비부터 복구 확인까지 자동화</strong>
        <span>프로젝트 보기 <b aria-hidden="true">→</b></span>
      </div>
    </a>

    <a class="home-menu-item" href="{{ '/projects/sic-portal/' | relative_url }}">
      <div class="home-menu-item__title">
        <span>PROJECT 03</span>
        <h3>SIC 동아리 포털 및 CI/CD 구축</h3>
      </div>
      <p class="home-menu-item__description">팀 리딩을 맡아 Spring Boot 서비스와 GitHub Actions 배포 파이프라인, AWS 운영 환경을 구축했습니다.</p>
      <div class="home-menu-item__outcome">
        <strong>팀 개발 · 테스트 기준 · 자동 배포</strong>
        <span>프로젝트 보기 <b aria-hidden="true">→</b></span>
      </div>
    </a>
  </div>
</section>

<section class="home-section" id="engineering-cases">
  <div class="home-section__header">
    <div>
      <span class="home-section__eyebrow">ENGINEERING CASES</span>
      <h2>구현 및 문제 해결 사례</h2>
    </div>
    <p>무엇을 잘한다는 설명보다, 실제로 어떤 문제를 다루고 무엇을 검증했는지 보여주는 사례입니다.</p>
  </div>

  <div class="home-menu-list">
    <a class="home-menu-item" href="{{ '/reports/timescaledb-27x/' | relative_url }}">
      <div class="home-menu-item__title">
        <span>DATABASE</span>
        <h3>TimescaleDB 시계열 조회 성능 개선</h3>
      </div>
      <p class="home-menu-item__description">인덱스, 하이퍼테이블, chunk 전략과 Continuous Aggregate를 조회 패턴별로 비교했습니다.</p>
      <div class="home-menu-item__outcome">
        <strong>p95 7,247ms → 235ms</strong>
        <span>보고서 보기 <b aria-hidden="true">→</b></span>
      </div>
    </a>

    <a class="home-menu-item" href="{{ '/reports/websocket-group-canvas/' | relative_url }}">
      <div class="home-menu-item__title">
        <span>REALTIME</span>
        <h3>WebSocket 동시 편집 안정성 개선</h3>
      </div>
      <p class="home-menu-item__description">메시지 순서, 중복 전송, 지연 문제를 분석하고 Dirty Flag와 배치 브로드캐스트를 적용했습니다.</p>
      <div class="home-menu-item__outcome">
        <strong>≤200ms 0.38% → 99.97%</strong>
        <span>보고서 보기 <b aria-hidden="true">→</b></span>
      </div>
    </a>

    <a class="home-menu-item" href="{{ '/reports/realtime-degrade-overview/' | relative_url }}">
      <div class="home-menu-item__title">
        <span>FAILURE RECOVERY</span>
        <h3>Redis·Kafka 장애 대응 및 상태 복구</h3>
      </div>
      <p class="home-menu-item__description">Redis Pub/Sub은 저지연 전파, Kafka는 durable event와 replay를 담당하도록 분리하고 degraded mode를 검증했습니다.</p>
      <div class="home-menu-item__outcome">
        <strong>DB fallback · outbox · replay</strong>
        <span>보고서 보기 <b aria-hidden="true">→</b></span>
      </div>
    </a>

    <a class="home-menu-item" href="{{ '/reports/trader-data-platform/' | relative_url }}">
      <div class="home-menu-item__title">
        <span>DATA PIPELINE</span>
        <h3>KIS·BLS·SEC 데이터 파이프라인 구축</h3>
      </div>
      <p class="home-menu-item__description">raw 원본 보존, DB outbox, Python ETL, lineage와 Consumer commit 상태를 하나의 처리 흐름으로 연결했습니다.</p>
      <div class="home-menu-item__outcome">
        <strong>BLS raw lag 2 → 0</strong>
        <span>보고서 보기 <b aria-hidden="true">→</b></span>
      </div>
    </a>

    <a class="home-menu-item" href="{{ '/reports/trader-data-platform/#aws-worker-scaling' | relative_url }}">
      <div class="home-menu-item__title">
        <span>WORKER CONTROL</span>
        <h3>Kafka lag 기반 AWS Worker 자동 확장</h3>
      </div>
      <p class="home-menu-item__description">Go Controller가 lag, heartbeat, idle 상태를 판단해 Python Worker ASG의 실행과 회수를 제어하도록 구성했습니다.</p>
      <div class="home-menu-item__outcome">
        <strong>desired capacity 0 → 1 → 0</strong>
        <span>검증 보기 <b aria-hidden="true">→</b></span>
      </div>
    </a>

    <a class="home-menu-item" href="{{ '/projects/loadtest-orchestrator/' | relative_url }}">
      <div class="home-menu-item__title">
        <span>TEST AUTOMATION</span>
        <h3>Go 기반 부하·장애 검증 자동화</h3>
      </div>
      <p class="home-menu-item__description">인프라 준비, 부하 실행, 장애 주입, 복구 확인, 결과 정리를 의존성 기반 단계로 실행하도록 만들었습니다.</p>
      <div class="home-menu-item__outcome">
        <strong>Terraform · k6 · AWS SSM</strong>
        <span>프로젝트 보기 <b aria-hidden="true">→</b></span>
      </div>
    </a>
  </div>
</section>

<section class="home-section home-method-section" id="engineering-method">
  <div class="home-section__header">
    <div>
      <span class="home-section__eyebrow">WORKING METHOD</span>
      <h2>문제 해결 방식</h2>
    </div>
    <p>문제를 수치로 고정하고, 처리 책임을 나눈 뒤, 장애 이후의 복구 결과까지 같은 조건에서 확인합니다.</p>
  </div>

  <ol class="home-method-list">
    <li><span>01</span><div><strong>측정</strong><p>p95, lag, GC, received count로 문제를 재현합니다.</p></div></li>
    <li><span>02</span><div><strong>책임 분리</strong><p>hot path, durable log, 원천 데이터의 역할을 구분합니다.</p></div></li>
    <li><span>03</span><div><strong>복구 검증</strong><p>replay, heartbeat, scale command로 운영 상태를 확인합니다.</p></div></li>
  </ol>

  <div class="home-footer-actions">
    <a class="btn btn--primary" href="{{ '/reports/' | relative_url }}">전체 Engineering Reports</a>
    <a class="btn btn--inverse" href="{{ '/resume/' | relative_url }}">Resume</a>
  </div>
</section>
