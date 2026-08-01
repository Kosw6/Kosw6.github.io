---
layout: splash
title: "Sungwon Kim"
classes: wide
---

<div class="portfolio-home">
  <section class="home-profile" aria-labelledby="home-name">
    <span class="home-profile__role">백엔드 개발자</span>
    <h1 id="home-name">김성원</h1>
    <p class="home-profile__lead">
      실시간 서비스와 데이터 파이프라인을 만들고, 느려지거나 중단되는 지점을 측정해 개선합니다.
    </p>
    <p class="home-profile__summary home-sentence-lines">
      <span>Spring Boot 기반 실시간 서비스와 Go, Python 기반 데이터 파이프라인을 개발했습니다.</span>
      <span>성능과 장애 대응은 같은 조건의 부하 테스트와 복구 시나리오로 확인했습니다.</span>
    </p>
    <nav class="home-profile__links" aria-label="주요 링크">
      <a href="{{ '/resume/' | relative_url }}">Resume</a>
      <a href="{{ '/projects/' | relative_url }}">Projects</a>
      <a href="https://github.com/Kosw6">GitHub</a>
    </nav>
  </section>

  <section class="home-primary-project" aria-labelledby="primary-project-title">
    <header class="home-section-heading">
      <span>프로젝트 개요</span>
      <div>
        <h2 id="primary-project-title">Trader, 주식투자 복기 서비스</h2>
        <p>2025.01 - 현재, 개인 개발, 기여도 100%</p>
      </div>
    </header>

    <p class="home-primary-project__description home-sentence-lines">
      <span>
        투자 판단과 결과를 다시 살펴볼 수 있도록 <strong>차트, 투자 일지, React Flow 캔버스</strong>를
        하나의 서비스로 연결했습니다.
      </span>
      <span>
        KIS 주가, BLS 거시경제, SEC 재무 데이터를 수집하고 정규화해 차트와 조회 기능에 제공하며,
        실시간 협업 그래프 기능 개발과 데이터 처리 상태를 운영자가 확인할 수 있게 만들었습니다.
      </span>
    </p>

    <div class="home-project-scope" aria-label="Trader 구현 범위">
      <div>
        <span>사용자 기능</span>
        <strong>차트, 일지, 실시간 캔버스</strong>
        <p>Spring Boot, React, WebSocket</p>
      </div>
      <div>
        <span>데이터 처리</span>
        <strong>외부 데이터 수집과 ETL</strong>
        <p>Go Controller, Python Worker, Kafka</p>
      </div>
      <div>
        <span>운영 검증</span>
        <strong>성능 측정과 장애 복구</strong>
        <p>TimescaleDB, k6, AWS, Terraform</p>
      </div>
    </div>

    <div class="home-primary-project__links">
      <a href="{{ '/projects/trader/' | relative_url }}">프로젝트 전체 보기 →</a>
      <a href="https://github.com/Kosw6/trader-backend">Backend</a>
      <a href="https://github.com/Kosw6/trader-controller">Controller</a>
      <a href="https://github.com/Kosw6/trader-data">Data Pipeline</a>
    </div>
  </section>

  <section class="home-case-section" aria-labelledby="case-section-title">
    <header class="home-section-heading home-section-heading--intro">
      <span>문제 해결</span>
      <div>
        <h2 id="case-section-title">대표 문제 해결 사례</h2>
        <p>같은 조건에서 대안을 비교하고 결과를 확인한 네 가지 사례입니다.</p>
      </div>
    </header>

    <div class="home-case-list">
      <article class="home-case">
        <header class="home-case__label">
          <span>01</span>
          <strong>데이터 파이프라인</strong>
        </header>
        <div class="home-case__content">
          <h3>데이터 수집 파이프라인: 중단 이후 처리 추적과 재처리 구조 설계</h3>
          <p>
            <b>문제</b>
            <span class="home-sentence-lines">
              <span>투자 복기 화면에 주가, 거시경제, 재무 데이터를 지속해서 제공하려면 외부 API 수집과 데이터 변환 중 어느 단계에서 멈췄는지 확인할 수 있어야 했습니다.</span>
              <span>KIS, BLS, SEC 응답을 바로 정규화 테이블에 넣으면 Worker 중단 후 원본 보존 여부와 성공한 단계를 판단하기 어려웠습니다.</span>
            </span>
          </p>
          <p>
            <b>판단과 선택</b>
            <span class="home-case__sentences">
              <span>핵심은 Kafka가 아니라 DB 기록만으로도 처리 완료 여부를 판단할 수 있어야 한다는 점이었습니다.</span>
              <span>외부 응답을 바로 적재하면 실패 지점과 원본이 남지 않고, Kafka만 기준으로 두면 DB 반영 상태를 확인하기 어렵습니다.</span>
              <span>그래서 원본, 발행 요청과 변환 결과의 연결을 DB에 먼저 기록하고 Kafka는 비동기 전달과 대기량 측정에 사용했습니다.</span>
            </span>
          </p>
          <p class="home-case__result">
            <b>결과</b>
            <span class="home-sentence-lines">
              <span>ETL 재개 검증에서는 중단 중 쌓인 <strong>BLS raw 2건을 모두 처리</strong>했습니다.</span>
              <span>별도의 ASG 검증에서는 job lag를 감지한 AWS Worker가 <strong>0 → 1 → 0</strong>으로 동작했습니다.</span>
            </span>
          </p>
          <footer>
            <span>Go, Python, Kafka, PostgreSQL, AWS ASG</span>
            <a href="{{ '/reports/trader-data-platform/' | relative_url }}">처리 기록과 복구 검증 보기 →</a>
          </footer>
        </div>
      </article>

      <article class="home-case">
        <header class="home-case__label">
          <span>02</span>
          <strong>실시간 처리</strong>
        </header>
        <div class="home-case__content">
          <h3>WebSocket 동시 편집 전송 개선: 200ms 이내 수신율 0.38% → 99.97%</h3>
          <p>
            <b>문제</b>
            <span class="home-sentence-lines">
              <span>실시간 협업 캔버스는 한 사용자의 편집 결과를 다른 사용자에게 빠르게 보여줘야 했습니다.</span>
              <span>동시 편집 부하 테스트 초기에 여러 작업이 같은 세션으로 동시에 전송하면서 쓰기 충돌과 세션 이탈이 발생했습니다.</span>
              <span>세션별 전송을 직렬화해 오류를 제거한 뒤에도 반복 전송이 쌓여 200ms 이내 수신 비율은 0.38%에 머물렀습니다.</span>
            </span>
          </p>
          <p>
            <b>판단과 선택</b>
            <span class="home-case__sentences">
              <span>핵심은 모든 편집 이벤트를 빠짐없이 보내는 것이 아니라 최신 화면 상태를 제때 전달하는 것이었습니다.</span>
              <span>먼저 세션별 쓰기를 직렬화해 충돌을 막고, 커서와 드래그 미리보기는 최신값만 모아 보냈습니다.</span>
              <span>실제 편집 결과처럼 유실되면 안 되는 이벤트는 순서대로 전달했습니다.</span>
              <span>변경이 없을 때는 다시 보내지 않도록 중복 전송도 제거했습니다.</span>
            </span>
          </p>
          <p class="home-case__result">
            <b>결과</b>
            <span class="home-sentence-lines">
              <span>동시 접속 200명, 송신자 20명의 같은 부하에서 <strong>200ms 이내 수신 비율을 99.97%</strong>로 높였습니다.</span>
              <span>별도 장애 전환 PoC에서는 서버가 변경 이력을 따라잡은 뒤 다시 서비스에 편입되는 흐름을 검증했습니다.</span>
            </span>
          </p>
          <footer>
            <span>Spring Boot, WebSocket, Redis Pub/Sub, Kafka</span>
            <div>
              <a href="{{ '/reports/websocket-group-canvas/' | relative_url }}">성능 분석 →</a>
              <a href="{{ '/reports/realtime-degrade-overview/' | relative_url }}">장애 복구 →</a>
            </div>
          </footer>
        </div>
      </article>

      <article class="home-case">
        <header class="home-case__label">
          <span>03</span>
          <strong>조회 성능</strong>
        </header>
        <div class="home-case__content">
          <h3>주가 차트 조회 병목 개선: p95 7,247ms → 235ms</h3>
          <p>
            <b>문제</b>
            <span class="home-sentence-lines">
              <span>사용자는 차트를 좌우로 이동하며 90일 단위로 과거 주가를 반복 조회합니다.</span>
              <span>데이터가 약 2,600만 행까지 쌓이자 이 API는 300 RPS에서 목표 응답시간 300ms를 지키지 못했고 p95가 7,247ms까지 늘어났습니다.</span>
            </span>
          </p>
          <p>
            <b>판단과 선택</b>
            <span class="home-case__sentences">
              <span>핵심은 약 2,600만 행의 시계열 데이터를 더 효율적으로 조회하는 방법을 찾는 것이었습니다.</span>
              <span>기존 PostgreSQL 기반 코드와 운영 방식을 유지하면서 시계열 데이터 분할을 적용할 수 있는 TimescaleDB 확장을 선택했습니다.</span>
              <span>이후 하루 한 번 약 1만 건을 배치로 적재하고 조회가 대부분인 특성을 고려해 쓰기 병렬성보다 읽기 성능을 우선하고, 90일 시간 분할과 공간 분할 4개를 적용했습니다.</span>
            </span>
          </p>
          <p class="home-case__result">
            <b>결과</b>
            <span>
              동일한 90일 조회와 300 RPS 조건에서 p95를
              <strong>7,247ms에서 235ms</strong>로 줄여 목표 응답시간을 충족했습니다.
            </span>
          </p>
          <footer>
            <span>PostgreSQL, TimescaleDB, k6</span>
            <a href="{{ '/reports/timescaledb-27x/' | relative_url }}">실행 계획과 측정 결과 보기 →</a>
          </footer>
        </div>
      </article>

      <article class="home-case">
        <header class="home-case__label">
          <span>04</span>
          <strong>장애 복구</strong>
        </header>
        <div class="home-case__content">
          <h3>장애 유형별 자동 복구 설계: 컨테이너 재시작과 ASG 확장 검증</h3>
          <p>
            <b>문제</b>
            <span class="home-sentence-lines">
              <span>서비스 일부의 장애가 전체 기능 중단으로 이어지지 않게 하려면 장애 유형을 구분해 필요한 조치를 빠르게 실행해야 했습니다.</span>
              <span>여러 컨테이너가 연결된 AWS 환경에서는 애플리케이션 중단과 CPU 부족에 서로 다른 복구 작업이 필요했습니다.</span>
            </span>
          </p>
          <p>
            <b>판단과 선택</b>
            <span class="home-case__sentences">
              <span>핵심은 장애마다 필요한 복구 단위가 다르다는 점이었습니다.</span>
              <span>애플리케이션 중단에 즉시 EC2를 교체하면 복구가 느리고 로그와 WebSocket 연결에 미치는 영향이 커지므로 먼저 컨테이너만 재시작했습니다.</span>
              <span>반면 지속적인 CPU 부족은 재시작으로 처리 용량이 늘지 않으므로 ASG 확장을 선택했습니다.</span>
            </span>
          </p>
          <p class="home-case__result">
            <b>결과</b>
            <span class="home-sentence-lines">
              <span>애플리케이션 중단 이후 컨테이너가 재시작되고 모니터링 상태가 <strong>정상으로 복귀</strong>했습니다.</span>
              <span>CPU 임계치 초과 상황에서는 ASG가 <strong>1대에서 2대</strong>로 확장됐습니다.</span>
            </span>
          </p>
          <footer>
            <span>Prometheus, Loki, Lambda, AWS SSM, ASG</span>
            <div>
              <a href="{{ '/reports/observability-system/' | relative_url }}">감지 기준 →</a>
              <a href="{{ '/reports/auto-recovery-scaleout/' | relative_url }}">복구 검증 →</a>
            </div>
          </footer>
        </div>
      </article>
    </div>
  </section>

  <section class="home-other-projects" aria-labelledby="other-projects-title">
    <header class="home-section-heading">
      <span>다른 경험</span>
      <div>
        <h2 id="other-projects-title">다른 프로젝트</h2>
      </div>
    </header>

    <div class="home-project-list">
      <article>
        <div class="home-project-list__meta">2026.04 - 2026.06, 개인 개발</div>
        <div>
          <h3>부하 및 장애 검증 시나리오 오케스트레이터</h3>
          <p class="home-sentence-lines">
            <span>반복되는 부하 테스트, 장애 주입, 복구 확인을 단계와 의존성으로 연결했습니다.</span>
            <span>그래프 UI에서 시나리오를 만들고 ZIP으로 내려받아 각 사용자의 로컬 환경에서 실행하도록 구성했습니다.</span>
          </p>
        </div>
        <a href="{{ '/projects/loadtest-orchestrator/' | relative_url }}">상세 보기 →</a>
      </article>

      <article>
        <div class="home-project-list__meta">2025.09 - 2025.11, 팀장</div>
        <div>
          <h3>SIC 동아리 포털 개발 리딩 및 CI/CD 구축</h3>
          <p>
            팀의 기능 범위와 일정을 조율하고 Spring Boot 백엔드, 테스트 기준,
            GitHub Actions 자동 배포와 AWS 배포 환경을 구축했습니다.
          </p>
        </div>
        <a href="{{ '/projects/sic-portal/' | relative_url }}">상세 보기 →</a>
      </article>
    </div>
  </section>

  <section class="home-background" aria-labelledby="background-title">
    <header class="home-section-heading">
      <span>이력</span>
      <div>
        <h2 id="background-title">기술과 학력</h2>
      </div>
    </header>

    <div class="home-background__grid">
      <div>
        <h3>주요 기술</h3>
        <dl>
          <div><dt>Backend</dt><dd>Java, Spring Boot, Go, Python</dd></div>
          <div><dt>Data</dt><dd>PostgreSQL, TimescaleDB, Redis, Kafka</dd></div>
          <div><dt>Infra</dt><dd>AWS, Docker, Terraform, Prometheus, Grafana</dd></div>
          <div><dt>Test</dt><dd>k6, JFR, JMC, Testcontainers</dd></div>
        </dl>
      </div>
      <div>
        <h3>학력</h3>
        <div class="home-education">
          <p><span>2020.03 - 2026.02</span><strong>세종대학교 졸업</strong></p>
          <p>환경에너지공간융합학과 주전공, 컴퓨터공학과 복수전공</p>
        </div>
        <div class="home-education">
          <p><span>2017.03 - 2020.01</span><strong>한국디지털미디어고등학교 졸업</strong></p>
        </div>
      </div>
    </div>
  </section>

  <section class="home-more" aria-labelledby="more-title">
    <header class="home-section-heading">
      <span>근거 문서</span>
      <div>
        <h2 id="more-title">추가 문제 해결 기록</h2>
        <p>대표 사례 외의 실험과 판단 근거는 짧은 리포트로 정리했습니다.</p>
      </div>
    </header>

    <nav class="home-more__links" aria-label="추가 엔지니어링 리포트">
      <a href="{{ '/reports/realtime-degrade-overview/#reliable-event-recovery' | relative_url }}"><strong>Redis Pub/Sub 누락 보정</strong><span>Redis는 즉시 전파, Kafka는 Reliable 이벤트 누락 복구</span></a>
      <a href="{{ '/reports/websocket-poc3-failback/' | relative_url }}"><strong>WebSocket 서버 복귀 검증</strong><span>목표 Kafka offset까지 처리한 뒤 복구 서버를 운영 경로에 재편입</span></a>
      <a href="{{ '/reports/loadtest-orchestrator-redis-fault-validation/' | relative_url }}"><strong>부하, 장애 검증 자동화</strong><span>장애 주입부터 복구 확인까지 같은 순서로 반복 실행</span></a>
      <a href="{{ '/reports/jpa-tuning/' | relative_url }}"><strong>JPA 조회 전략 비교</strong><span>N+1, Fetch Join, Projection 비교</span></a>
      <a href="{{ '/reports/jfr-jmc-hotpath/' | relative_url }}"><strong>JFR/JMC Hot Path 분석</strong><span>JWT 중복 검증과 객체 할당 병목</span></a>
      <a href="{{ '/reports/slo-operating-capacity/' | relative_url }}"><strong>SLO 기반 인프라 사양 검증</strong><span>부하와 자원 사용량의 관계 확인</span></a>
      <a href="{{ '/reports/' | relative_url }}"><strong>전체 Engineering Reports</strong><span>측정 조건, 비교 과정과 한계</span></a>
    </nav>

    <div class="home-closing-links">
      <a href="{{ '/resume/' | relative_url }}">Resume</a>
      <a href="{{ '/projects/' | relative_url }}">All Projects</a>
      <a href="https://github.com/Kosw6">GitHub</a>
    </div>
  </section>
</div>
