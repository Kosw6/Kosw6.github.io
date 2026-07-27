---
layout: single
title: "Projects"
permalink: /projects/
classes: wide
---

<div class="portfolio-page-intro">
  <span>SELECTED PROJECTS</span>
  <p>기능 구현에서 멈추지 않고 성능 측정, 장애 복구, 운영 자동화까지 이어간 대표 프로젝트입니다. 각 상세 페이지에서 설계 결정과 검증 근거를 확인할 수 있습니다.</p>
</div>

<div class="home-menu-list portfolio-project-list">
  <a class="home-menu-item" href="{{ '/projects/trader/' | relative_url }}">
    <div class="home-menu-item__title">
      <span>PROJECT 01</span>
      <h2>Trader 주식투자 복기 플랫폼</h2>
    </div>
    <p class="home-menu-item__description">실시간 협업 캔버스와 투자 일지, 시계열 차트를 제공하고 KIS, BLS, SEC 데이터를 수집·정규화하는 플랫폼입니다.</p>
    <div class="home-menu-item__outcome">
      <strong>p95 7,247ms → 235ms<br>WebSocket ≤200ms 99.97%</strong>
      <span>상세 보기 <b aria-hidden="true">→</b></span>
    </div>
  </a>

  <a class="home-menu-item" href="{{ '/projects/loadtest-orchestrator/' | relative_url }}">
    <div class="home-menu-item__title">
      <span>PROJECT 02</span>
      <h2>부하·장애 검증 시나리오 오케스트레이터</h2>
    </div>
    <p class="home-menu-item__description">시나리오 작성 UI와 Go 실행 엔진을 통합해 인프라 준비, 부하 실행, 장애 주입, 복구 확인을 반복 가능한 흐름으로 만들었습니다.</p>
    <div class="home-menu-item__outcome">
      <strong>Terraform · k6 · AWS SSM<br>의존성 기반 단계 실행</strong>
      <span>상세 보기 <b aria-hidden="true">→</b></span>
    </div>
  </a>

  <a class="home-menu-item" href="{{ '/projects/sic-portal/' | relative_url }}">
    <div class="home-menu-item__title">
      <span>PROJECT 03</span>
      <h2>SIC 동아리 포털 및 CI/CD 구축</h2>
    </div>
    <p class="home-menu-item__description">동아리 운영 문제를 해결하기 위해 15명 팀을 이끌고 Spring Boot 서비스, 테스트 기준, AWS 배포 파이프라인을 구축했습니다.</p>
    <div class="home-menu-item__outcome">
      <strong>팀 리딩 · GitHub Actions<br>AWS 운영 시간 제어</strong>
      <span>상세 보기 <b aria-hidden="true">→</b></span>
    </div>
  </a>
</div>

<div class="portfolio-page-footer">
  <a class="btn btn--primary" href="{{ '/reports/' | relative_url }}">Engineering Reports</a>
  <a class="btn btn--light-outline" href="{{ '/resume/' | relative_url }}">Resume</a>
</div>
