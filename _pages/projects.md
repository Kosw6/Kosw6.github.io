---
layout: single
title: "Projects"
permalink: /projects/
classes:
  - wide
  - portfolio-index
  - portfolio-index--wide
---

<div class="portfolio-page-intro">
  <span>프로젝트</span>
  <p>서비스 개발, 성능 개선, 장애 복구와 검증 자동화를 직접 진행한 프로젝트입니다.</p>
</div>

<div class="home-menu-list portfolio-project-list">
  <a class="home-menu-item" href="{{ '/projects/trader/' | relative_url }}">
    <div class="home-menu-item__title">
      <span>01</span>
      <h2>Trader 주식투자 복기 서비스</h2>
    </div>
    <p class="home-menu-item__description">투자 판단을 다시 살펴볼 수 있도록 차트, 일지, 실시간 캔버스를 연결하고 주가, 거시경제, 재무 데이터를 수집해 제공했습니다.</p>
    <div class="home-menu-item__outcome">
      <strong>p95 7,247ms → 235ms<br>WebSocket ≤200ms 99.97%</strong>
      <span>상세 보기 <b aria-hidden="true">→</b></span>
    </div>
  </a>

  <a class="home-menu-item" href="{{ '/projects/loadtest-orchestrator/' | relative_url }}">
    <div class="home-menu-item__title">
      <span>02</span>
      <h2>부하 및 장애 검증 시나리오 오케스트레이터</h2>
    </div>
    <p class="home-menu-item__description">반복되는 인프라 준비, 부하 실행, 장애 주입과 복구 확인을 그래프 UI에서 하나의 실행 순서로 만들 수 있게 했습니다.</p>
    <div class="home-menu-item__outcome">
      <strong>단계와 의존성 기반 실행<br>로컬에서 같은 조건으로 반복 검증</strong>
      <span>상세 보기 <b aria-hidden="true">→</b></span>
    </div>
  </a>

  <a class="home-menu-item" href="{{ '/projects/sic-portal/' | relative_url }}">
    <div class="home-menu-item__title">
      <span>03</span>
      <h2>SIC 동아리 포털 개발 리딩 및 CI/CD 구축</h2>
    </div>
    <p class="home-menu-item__description">동아리 자료와 활동을 한곳에서 관리할 수 있도록 팀의 기능 범위와 일정을 조율하고 웹 서비스와 배포 환경을 구축했습니다.</p>
    <div class="home-menu-item__outcome">
      <strong>14명 팀 리딩, 자동 배포<br>AWS 운영 시간 제어</strong>
      <span>상세 보기 <b aria-hidden="true">→</b></span>
    </div>
  </a>
</div>

<div class="portfolio-page-footer">
  <a class="btn btn--primary" href="{{ '/reports/' | relative_url }}">문제 해결 기록</a>
  <a class="btn btn--light-outline" href="{{ '/resume/' | relative_url }}">이력 보기</a>
</div>
