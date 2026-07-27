---
layout: splash
title: "Sungwon Kim"
classes: wide
---

<section class="resume-hero" aria-labelledby="resume-title">
  <div class="resume-hero__intro">
    <span class="resume-kicker">BACKEND DEVELOPER</span>
    <h1 id="resume-title">김성원</h1>
    <p class="resume-hero__lead">
      느린 조회, 끊기는 실시간 협업, 추적하기 어려운 데이터 처리 문제를 직접 재현하고 개선했습니다.
    </p>
    <p class="resume-hero__summary">
      문제를 수치로 확인하고 여러 대안을 비교한 뒤, 장애 이후에도 복구 가능한 구조로 마무리하는 개발을 지향합니다.
    </p>
    <div class="resume-hero__actions">
      <a class="btn btn--primary" href="{{ '/resume/' | relative_url }}">이력서 보기</a>
      <a class="resume-text-link" href="https://github.com/Kosw6">GitHub</a>
    </div>
  </div>

  <aside class="resume-profile" aria-label="핵심 정보">
    <dl>
      <div>
        <dt>관심 분야</dt>
        <dd>성능 개선, 실시간 처리, 데이터 파이프라인</dd>
      </div>
      <div>
        <dt>주요 언어</dt>
        <dd>Java, Go, Python</dd>
      </div>
      <div>
        <dt>검증 방식</dt>
        <dd>부하 테스트, 장애 주입, 복구 확인</dd>
      </div>
    </dl>
  </aside>
</section>

<section class="resume-section" aria-labelledby="strengths-title">
  <div class="resume-section__heading">
    <span>CORE STRENGTHS</span>
    <h2 id="strengths-title">주로 해결해 온 문제</h2>
  </div>

  <div class="strength-list">
    <div class="strength-item">
      <strong>느린 요청의 원인 찾기</strong>
      <p>응답 시간과 런타임 지표를 측정하고 병목 구간을 좁혀 데이터 구조와 처리 방식을 개선합니다.</p>
    </div>
    <div class="strength-item">
      <strong>실시간 서비스 안정화</strong>
      <p>동시성 문제와 장애 상황을 재현하고, 사용자의 작업이 끊기거나 사라지지 않는 복구 흐름을 설계합니다.</p>
    </div>
    <div class="strength-item">
      <strong>데이터 처리 과정 추적</strong>
      <p>외부 데이터 수집부터 변환과 저장까지 처리 상태를 남겨 실패 지점과 재처리 범위를 확인할 수 있게 합니다.</p>
    </div>
  </div>
</section>

<section class="resume-section" id="problem-solving" aria-labelledby="problems-title">
  <div class="resume-section__heading resume-section__heading--split">
    <div>
      <span>PROBLEM SOLVING</span>
      <h2 id="problems-title">대표 문제 해결 경험</h2>
    </div>
    <p>문제를 재현하고 대안을 비교한 뒤, 같은 조건에서 개선 결과를 확인했습니다.</p>
  </div>

  <div class="case-list">
    <a class="case-row" href="{{ '/reports/timescaledb-27x/' | relative_url }}">
      <div class="case-row__number">01</div>
      <div class="case-row__body">
        <span class="case-row__label">주가 차트 조회</span>
        <h3>2,600만 건의 데이터에서 차트가 열리는 데 7초가 걸렸습니다</h3>
        <p><b>판단</b> 조회 범위와 정렬 방식에 맞춰 인덱스, 데이터 분할, 사전 집계 방식을 단계별로 비교했습니다.</p>
      </div>
      <div class="case-row__result">
        <span>결과</span>
        <strong>7,247ms → 235ms</strong>
        <small>과정 보기 →</small>
      </div>
    </a>

    <a class="case-row" href="{{ '/reports/websocket-group-canvas/' | relative_url }}">
      <div class="case-row__number">02</div>
      <div class="case-row__body">
        <span class="case-row__label">실시간 동시 편집</span>
        <h3>여러 사용자가 동시에 편집하면 메시지가 겹치고 연결이 끊겼습니다</h3>
        <p><b>판단</b> 전송 책임을 한곳으로 모으고, 짧은 시간의 변경 내용을 묶어 보내도록 구조를 바꿨습니다.</p>
      </div>
      <div class="case-row__result">
        <span>결과</span>
        <strong>200ms 이내 99.97%</strong>
        <small>과정 보기 →</small>
      </div>
    </a>

    <a class="case-row" href="{{ '/reports/trader-data-platform/' | relative_url }}">
      <div class="case-row__number">03</div>
      <div class="case-row__body">
        <span class="case-row__label">외부 데이터 수집</span>
        <h3>수집이 중단되면 어디까지 처리됐는지 확인하기 어려웠습니다</h3>
        <p><b>판단</b> 원본을 먼저 보존하고 수집과 변환을 분리해, 실패 지점과 다시 처리할 범위를 기록했습니다.</p>
      </div>
      <div class="case-row__result">
        <span>결과</span>
        <strong>대기 2 → 0</strong>
        <small>과정 보기 →</small>
      </div>
    </a>
  </div>
</section>

<section class="resume-section" id="featured-projects" aria-labelledby="projects-title">
  <div class="resume-section__heading resume-section__heading--split">
    <div>
      <span>PROJECTS</span>
      <h2 id="projects-title">프로젝트</h2>
    </div>
    <p>기술보다 먼저, 어떤 사용자를 위해 무엇을 만들었는지 설명했습니다.</p>
  </div>

  <div class="project-list">
    <a class="project-row" href="{{ '/projects/trader/' | relative_url }}">
      <div class="project-row__meta">
        <span>개인 프로젝트</span>
        <small>2025.01 - 2026.07</small>
      </div>
      <div class="project-row__body">
        <h3>Trader 주식투자 복기 플랫폼</h3>
        <p>투자 판단 근거를 캔버스와 일지로 기록하고, 나중에 매매 과정의 인과관계를 복기할 수 있도록 만든 서비스입니다.</p>
      </div>
      <div class="project-row__link">프로젝트 보기 →</div>
    </a>

    <a class="project-row" href="{{ '/projects/loadtest-orchestrator/' | relative_url }}">
      <div class="project-row__meta">
        <span>개인 프로젝트</span>
        <small>2026.04 - 2026.06</small>
      </div>
      <div class="project-row__body">
        <h3>부하 및 장애 검증 오케스트레이터</h3>
        <p>반복되는 인프라 준비, 부하 실행, 장애 주입, 복구 확인을 하나의 시나리오로 작성하고 실행하는 도구입니다.</p>
      </div>
      <div class="project-row__link">프로젝트 보기 →</div>
    </a>

    <a class="project-row" href="{{ '/projects/sic-portal/' | relative_url }}">
      <div class="project-row__meta">
        <span>팀 프로젝트</span>
        <small>2025.09 - 2025.11</small>
      </div>
      <div class="project-row__body">
        <h3>SIC 동아리 포털</h3>
        <p>흩어진 동아리 자료와 활동을 한곳에서 관리하고 금융 데이터를 활용한 서비스를 제공하기 위해 팀을 이끌며 개발했습니다.</p>
      </div>
      <div class="project-row__link">프로젝트 보기 →</div>
    </a>
  </div>
</section>

<section class="resume-section resume-more" aria-labelledby="more-title">
  <div class="resume-section__heading">
    <span>MORE</span>
    <h2 id="more-title">더 살펴보기</h2>
  </div>

  <nav class="more-links" aria-label="추가 포트폴리오 링크">
    <a href="{{ '/projects/' | relative_url }}">
      <strong>전체 프로젝트</strong>
      <span>서비스의 목적과 구현 범위</span>
    </a>
    <a href="{{ '/reports/' | relative_url }}">
      <strong>문제 해결 보고서</strong>
      <span>성능 측정과 장애 검증 과정</span>
    </a>
    <a href="{{ '/reports/realtime-degrade-overview/' | relative_url }}">
      <strong>장애 대응과 복구</strong>
      <span>서비스가 멈췄을 때의 판단과 대응</span>
    </a>
    <a href="{{ '/resume/' | relative_url }}">
      <strong>전체 이력서</strong>
      <span>경험과 기술을 한 페이지에서 확인</span>
    </a>
  </nav>
</section>
