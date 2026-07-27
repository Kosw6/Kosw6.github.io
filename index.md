---
layout: splash
title: "Sungwon Kim"
classes: wide
---

<section class="casebook-intro" aria-labelledby="casebook-name">
  <div class="casebook-intro__copy">
    <span class="casebook-kicker">BACKEND PLATFORM ENGINEER</span>
    <h1 id="casebook-name">김성원</h1>
    <p class="casebook-intro__statement">
      문제를 재현하고, 대안을 비교하고, 장애 이후의 복구까지 확인합니다.
    </p>
    <p class="casebook-intro__description">
      실시간 협업 서비스와 금융 데이터 파이프라인을 개발하며 느린 조회, 메시지 누락,
      처리 상태 추적 문제를 해결했습니다. Java, Go, Python으로 구현하고 AWS 환경에서
      성능과 복구 결과를 검증했습니다.
    </p>
    <div class="casebook-intro__links">
      <a href="{{ '/resume/' | relative_url }}">Resume</a>
      <a href="{{ '/projects/' | relative_url }}">Projects</a>
      <a href="{{ '/reports/' | relative_url }}">Engineering Reports</a>
      <a href="https://github.com/Kosw6">GitHub</a>
    </div>
  </div>

  <div class="casebook-method" aria-label="문제 해결 순서">
    <span class="casebook-method__label">문제 해결 방식</span>
    <ol>
      <li><b>01</b><span>사용자에게 발생한 문제를 문장으로 정의</span></li>
      <li><b>02</b><span>같은 조건에서 원인과 대안을 비교</span></li>
      <li><b>03</b><span>p95, lag, 처리 건수로 결과 확인</span></li>
      <li><b>04</b><span>장애와 복구 시나리오까지 반복 검증</span></li>
    </ol>
  </div>
</section>

<section class="casebook-metrics" aria-label="대표 검증 결과">
  <a href="{{ '/reports/timescaledb-27x/' | relative_url }}">
    <span>시계열 차트 조회</span>
    <strong>p95 7,247ms → 235ms</strong>
    <small>약 2,600만 행, 90일 조회, 300 RPS</small>
  </a>
  <a href="{{ '/reports/websocket-group-canvas/' | relative_url }}">
    <span>실시간 변경 수신</span>
    <strong>200ms 이내 99.97%</strong>
    <small>동시 편집 전송 구조 개선</small>
  </a>
  <a href="{{ '/reports/trader-data-platform/#aws-worker-scaling' | relative_url }}">
    <span>데이터 처리 Worker</span>
    <strong>AWS ASG 0 → 1 → 0</strong>
    <small>작업량 기반 실행과 회수</small>
  </a>
</section>

<section class="casebook-section" id="problems" aria-labelledby="problems-title">
  <header class="casebook-section__header">
    <span>한눈에 보기</span>
    <h2 id="problems-title">문제와 해결을 한눈에</h2>
    <p>세부 문서를 열지 않아도 어떤 기능에서 문제가 발생했고, 무엇을 선택해 어떻게 검증했는지 확인할 수 있습니다.</p>
  </header>

  <div class="problem-domains">
    <article>
      <div class="problem-domain__index">
        <span>01</span>
        <b>성능 개선</b>
      </div>
      <div class="problem-domain__content">
        <span>만들던 기능</span>
        <p>약 2,600만 행의 주가 데이터에서 90일 단위로 일별 주가를 탐색하는 투자 복기 차트 API</p>
        <h3>300 RPS에서 일부 요청의 p95가 7,247ms까지 증가해 목표 응답시간 300ms를 지키지 못했습니다.</h3>
        <p><strong>선택</strong> 서버 증설 전에 SQL 실행 계획을 확인하고 인덱스, 하이퍼테이블, chunk 간격, 사전 집계를 같은 부하에서 비교했습니다.</p>
      </div>
      <div class="problem-domain__result">
        <span>검증 결과</span>
        <strong>p95 7,247ms → 235ms</strong>
        <small>90일 조회, 300 RPS 기준</small>
        <a href="#case-performance">비교 과정과 근거 ↓</a>
      </div>
    </article>
    <article>
      <div class="problem-domain__index">
        <span>02</span>
        <b>실시간 처리</b>
      </div>
      <div class="problem-domain__content">
        <span>만들던 기능</span>
        <p>여러 사용자가 같은 React Flow 캔버스를 편집하고 변경 내용을 WebSocket으로 공유하는 기능</p>
        <h3>입력이 몰리면 메시지 순서와 중복 전송이 제어되지 않았고, 200ms 이내 수신 비율이 0.38%에 머물렀습니다.</h3>
        <p><strong>선택</strong> 전송 책임을 한곳으로 모으고 Dirty Flag와 배치 전송을 적용했습니다. 빠른 전파는 Redis, 복구할 변경 이력은 Kafka가 맡도록 나눴습니다.</p>
      </div>
      <div class="problem-domain__result">
        <span>검증 결과</span>
        <strong>200ms 이내 99.97%</strong>
        <small>동시 편집 부하 기준</small>
        <a href="#case-realtime">개선 과정과 복구 설계 ↓</a>
      </div>
    </article>
    <article>
      <div class="problem-domain__index">
        <span>03</span>
        <b>데이터 파이프라인</b>
      </div>
      <div class="problem-domain__content">
        <span>만들던 기능</span>
        <p>KIS 주가, BLS 거시경제, SEC 재무 데이터를 수집하고 공통 테이블로 정규화하는 ETL 파이프라인</p>
        <h3>수집과 정규화를 한 번에 처리하면 Worker 중단 후 원본 보존 여부, 실패 단계와 재처리 범위를 확인하기 어려웠습니다.</h3>
        <p><strong>선택</strong> 원본을 source_object로 먼저 보존하고 DB outbox, 처리 상태, lineage와 Kafka offset을 단계별로 기록했습니다.</p>
      </div>
      <div class="problem-domain__result">
        <span>검증 결과</span>
        <strong>raw lag 2 → 0</strong>
        <small>ASG 0 → 1 → 0</small>
        <a href="#case-pipeline">복구 시나리오와 기록 ↓</a>
      </div>
    </article>
    <article>
      <div class="problem-domain__index">
        <span>04</span>
        <b>운영 복구</b>
      </div>
      <div class="problem-domain__content">
        <span>만들던 기능</span>
        <p>여러 서버와 컨테이너로 구성된 서비스를 AWS에서 운영하는 환경</p>
        <h3>서비스 일부에서 서버가 중단되거나 리소스가 부족해질 때, 장애를 빠르게 발견하고 전체 서비스에 미치는 영향을 줄여야 했습니다.</h3>
        <p><strong>선택</strong> 로그와 자원 지표로 장애 기준과 알람을 구성했습니다. App Down은 컨테이너 재시작으로, 리소스 부족은 인스턴스 확장으로 대응하도록 복구 방식을 분리했습니다.</p>
      </div>
      <div class="problem-domain__result">
        <span>검증 결과</span>
        <strong>감지 → 재시작·확장</strong>
        <small>App scrape UP, ASG 1 → 2</small>
        <a href="#case-recovery">복구 흐름과 검증 과정 ↓</a>
      </div>
    </article>
  </div>
</section>

<section class="casebook-section casebook-cases" id="cases" aria-labelledby="cases-title">
  <header class="casebook-section__header">
    <span>대표 사례</span>
    <h2 id="cases-title">대표 문제 해결 사례</h2>
    <p>문제를 해결한 결과뿐 아니라, 왜 그 방법을 선택했는지 함께 정리했습니다.</p>
  </header>

  <article class="engineering-case" id="case-performance">
    <header class="engineering-case__header">
      <span class="engineering-case__number">CASE 01</span>
      <div>
        <span class="engineering-case__category">TimescaleDB 시계열 조회</span>
        <h3>일별 주가 차트 조회 지연을 어떻게 개선할 것인가</h3>
      </div>
      <strong>목표 응답시간 충족</strong>
    </header>

    <div class="engineering-case__content">
      <div class="engineering-case__context">
        <h4>무엇을 하려고 했나</h4>
        <p>
          90일 단위로 일별 주가 차트를 탐색하는 기능을 만들었습니다.
          약 2,600만 행을 대상으로 조회했을 때 일부 요청의 p95가 7초를 넘어 사용 흐름이 끊겼습니다.
        </p>
      </div>
      <div class="engineering-case__decision">
        <div>
          <h4>판단</h4>
          <p>
            서버 사양을 올리기 전에 SQL 실행 계획을 확인했습니다. 일반 인덱스, 하이퍼테이블,
            chunk 간격, Continuous Aggregate를 조회 기간별로 나누어 같은 부하에서 비교했습니다.
          </p>
        </div>
        <div class="engineering-case__result">
          <h4>검증 결과</h4>
          <p><b>300 RPS에서도 목표 응답시간 300ms 이내를 충족</b></p>
          <p>짧은 조회는 원본, 반복 집계가 많은 구간은 사전 집계를 사용하는 기준도 함께 정리했습니다.</p>
        </div>
      </div>
    </div>

    <footer class="engineering-case__footer">
      <span>PostgreSQL, TimescaleDB, k6</span>
      <a href="{{ '/reports/timescaledb-27x/' | relative_url }}">실행 계획과 측정 결과 보기 →</a>
    </footer>
  </article>

  <article class="engineering-case" id="case-realtime">
    <header class="engineering-case__header">
      <span class="engineering-case__number">CASE 02</span>
      <div>
        <span class="engineering-case__category">WebSocket 실시간 협업</span>
        <h3>빠른 전파와 장애 이후 복구를 어떻게 함께 만족시킬 것인가</h3>
      </div>
      <strong>지연 구간 해소</strong>
    </header>

    <div class="engineering-case__content">
      <div class="engineering-case__context">
        <h4>무엇을 하려고 했나</h4>
        <p>
          여러 사용자가 같은 캔버스를 편집하는 기능을 만들었습니다. 입력이 몰리면 메시지 순서가
          섞이고 중복 전송과 지연이 발생했으며, 연결 장애 중 작성한 내용도 보호해야 했습니다.
        </p>
      </div>
      <div class="engineering-case__decision">
        <div>
          <h4>판단</h4>
          <p>
            전송 책임을 한곳으로 모으고 짧은 시간의 변경을 묶어 보냈습니다. 즉시 보여줄 상태는
            Redis로 전파하고, 복구에 필요한 변경 이력은 Kafka에 남겨 서로 다른 책임을 맡겼습니다.
          </p>
        </div>
        <div class="engineering-case__result">
          <h4>검증 결과</h4>
          <p><b>동시 편집 부하에서도 대부분의 변경을 200ms 이내에 수신</b></p>
          <p>서버 전환 후에도 임시 저장 내용과 변경 이력을 비교해 충돌을 판별하고 다시 서비스에 편입했습니다.</p>
        </div>
      </div>
    </div>

    <footer class="engineering-case__footer">
      <span>Spring Boot, WebSocket, Redis Pub/Sub, Kafka</span>
      <div>
        <a href="{{ '/reports/websocket-group-canvas/' | relative_url }}">성능 개선 보기 →</a>
        <a href="{{ '/reports/realtime-degrade-overview/' | relative_url }}">장애 복구 보기 →</a>
      </div>
    </footer>
  </article>

  <article class="engineering-case" id="case-pipeline">
    <header class="engineering-case__header">
      <span class="engineering-case__number">CASE 03</span>
      <div>
        <span class="engineering-case__category">KIS, BLS, SEC 데이터 파이프라인</span>
        <h3>수집과 변환이 중단돼도 처리 상태를 어떻게 이어 갈 것인가</h3>
      </div>
      <strong>중단 후 처리 재개</strong>
    </header>

    <div class="engineering-case__content">
      <div class="engineering-case__context">
        <h4>무엇을 하려고 했나</h4>
        <p>
          서로 형식이 다른 주가, 거시경제, 재무 데이터를 수집하고 정규화했습니다.
          수집과 적재를 한 번에 처리하면 장애 후 원본과 성공 범위를 확인하기 어려웠습니다.
        </p>
      </div>
      <div class="engineering-case__decision">
        <div>
          <h4>판단</h4>
          <p>
            원본을 먼저 보존한 뒤 수집과 변환을 분리했습니다. 이벤트 발행, 변환 상태,
            원천과 적재 결과의 연결, Kafka 처리 위치를 각각 기록해 실패 지점을 확인하게 했습니다.
          </p>
        </div>
        <div class="engineering-case__result">
          <h4>검증 결과</h4>
          <p><b>Worker 중단 중 쌓인 BLS 원본 2건을 재개 후 모두 처리</b></p>
          <p>Go Controller가 작업이 있을 때 Worker를 실행하고 유휴 상태에서 회수하는 흐름도 확인했습니다.</p>
        </div>
      </div>
    </div>

    <footer class="engineering-case__footer">
      <span>Go Controller, Python Worker, Kafka, PostgreSQL, AWS ASG</span>
      <a href="{{ '/reports/trader-data-platform/' | relative_url }}">파이프라인 설계와 AWS 검증 보기 →</a>
    </footer>
  </article>

  <article class="engineering-case" id="case-recovery">
    <header class="engineering-case__header">
      <span class="engineering-case__number">CASE 04</span>
      <div>
        <span class="engineering-case__category">AWS 컨테이너 운영과 장애 복구</span>
        <h3>서비스 일부의 장애를 어떻게 감지하고 자동으로 복구할 것인가</h3>
      </div>
      <strong>복구 액션 자동화</strong>
    </header>

    <div class="engineering-case__content">
      <div class="engineering-case__context">
        <h4>무엇을 하려고 했나</h4>
        <p>
          여러 서비스가 연결된 AWS 환경에서 일부 서버가 중단되거나 리소스가 부족해져도
          전체 서비스에 미치는 영향을 줄이고 싶었습니다. 장애 판단 기준과 상황별 복구 작업을 정해야 했습니다.
        </p>
      </div>
      <div class="engineering-case__decision">
        <div>
          <h4>판단</h4>
          <p>
            Loki, Prometheus, Grafana로 로그와 자원 지표를 수집하고 장애 판단 기준을 알람으로 연결했습니다.
            App Down은 Lambda와 SSM RunCommand로 컨테이너를 재시작하고, High CPU는 ASG 용량을 늘리도록 분리했습니다.
          </p>
        </div>
        <div class="engineering-case__result">
          <h4>검증 결과</h4>
          <p><b>App Down 감지 후 컨테이너를 재시작하고 Prometheus scrape가 UP으로 복귀</b></p>
          <p>High CPU 알람에서는 ASG가 1대에서 2대로 확장되고 신규 인스턴스가 서비스와 모니터링 경로에 합류했습니다.</p>
        </div>
      </div>
    </div>

    <footer class="engineering-case__footer">
      <span>Loki, Prometheus, Grafana, Lambda, AWS SSM, ASG</span>
      <div>
        <a href="{{ '/reports/observability-system/' | relative_url }}">관측 기준과 알람 보기 →</a>
        <a href="{{ '/reports/auto-recovery-scaleout/' | relative_url }}">자동 복구 검증 보기 →</a>
      </div>
    </footer>
  </article>
</section>

<section class="casebook-section platform-map" id="platform" aria-labelledby="platform-title">
  <header class="casebook-section__header">
    <span>TRADER 플랫폼</span>
    <h2 id="platform-title">하나의 서비스로 연결한 처리 흐름</h2>
    <p>수집, 변환, 조회, 실시간 협업을 분리하되 운영자가 전체 상태를 추적할 수 있도록 연결했습니다.</p>
  </header>

  <div class="platform-flow" aria-label="Trader 데이터 처리 흐름">
    <div><span>01</span><strong>작업 생성</strong><small>Go Controller</small></div>
    <i aria-hidden="true">→</i>
    <div><span>02</span><strong>외부 데이터 수집</strong><small>KIS, BLS, SEC</small></div>
    <i aria-hidden="true">→</i>
    <div><span>03</span><strong>원본 보존</strong><small>source_object</small></div>
    <i aria-hidden="true">→</i>
    <div><span>04</span><strong>변환 요청</strong><small>DB outbox, Kafka</small></div>
    <i aria-hidden="true">→</i>
    <div><span>05</span><strong>정규화</strong><small>Python ETL</small></div>
    <i aria-hidden="true">→</i>
    <div><span>06</span><strong>추적 가능한 저장</strong><small>lineage, processed event</small></div>
  </div>

  <div class="control-flow">
    <span>운영 제어</span>
    <p><b>Kafka lag + Worker 상태</b>를 Go Controller가 확인하고, 필요할 때만 Python Worker를 실행합니다.</p>
    <strong>AWS ASG desired capacity 0 ↔ 1</strong>
  </div>

  <div class="platform-map__links">
    <a href="{{ '/projects/trader/' | relative_url }}">Trader 프로젝트 전체 보기 →</a>
    <a href="https://github.com/Kosw6/trader-backend">Backend source →</a>
    <a href="https://github.com/Kosw6/trader-controller">Controller source →</a>
    <a href="https://github.com/Kosw6/trader-data">Data pipeline source →</a>
  </div>
</section>

<section class="casebook-section" id="project-timeline" aria-labelledby="timeline-title">
  <header class="casebook-section__header">
    <span>프로젝트 이력</span>
    <h2 id="timeline-title">프로젝트</h2>
    <p>기능 구현부터 검증 자동화와 팀 운영까지 경험의 범위를 넓혔습니다.</p>
  </header>

  <div class="project-timeline">
    <article>
      <div class="project-timeline__date">2025.01 - 2026.07</div>
      <div>
        <span>개인 개발, 기여도 100%</span>
        <h3>Trader 주식투자 복기 플랫폼</h3>
        <p>실시간 캔버스, 투자 일지, 차트와 외부 금융 데이터를 연결하고 성능과 장애 복구까지 검증했습니다.</p>
      </div>
      <a href="{{ '/projects/trader/' | relative_url }}">상세 보기 →</a>
    </article>
    <article>
      <div class="project-timeline__date">2026.04 - 2026.06</div>
      <div>
        <span>개인 개발, 기여도 100%</span>
        <h3>부하 및 장애 검증 오케스트레이터</h3>
        <p>인프라 준비, 부하 실행, 장애 주입과 복구 확인의 반복 작업을 단계와 의존성 기반 시나리오로 자동화했습니다.</p>
      </div>
      <a href="{{ '/projects/loadtest-orchestrator/' | relative_url }}">상세 보기 →</a>
    </article>
    <article>
      <div class="project-timeline__date">2025.09 - 2025.11</div>
      <div>
        <span>팀장, 기여도 25%</span>
        <h3>SIC 동아리 포털 및 CI/CD 구축</h3>
        <p>팀의 기획과 일정을 조율하고 Spring Boot 백엔드, 테스트 기준, GitHub Actions 자동 배포와 AWS 환경을 구축했습니다.</p>
      </div>
      <a href="{{ '/projects/sic-portal/' | relative_url }}">상세 보기 →</a>
    </article>
  </div>
</section>

<section class="casebook-section education-section" id="education" aria-labelledby="education-title">
  <header class="casebook-section__header">
    <span>학력</span>
    <h2 id="education-title">학력</h2>
  </header>

  <div class="education-list">
    <div class="education-entry">
      <div class="education-entry__date">2020.03 - 2026.02</div>
      <div>
        <span>졸업</span>
        <h3>세종대학교</h3>
        <p>환경에너지공간융합학과 주전공, 컴퓨터공학과 복수전공</p>
      </div>
    </div>
    <div class="education-entry">
      <div class="education-entry__date">2017.03 - 2020.01</div>
      <div>
        <span>졸업</span>
        <h3>한국디지털미디어고등학교</h3>
      </div>
    </div>
  </div>
</section>

<section class="casebook-section evidence-index" id="evidence" aria-labelledby="evidence-title">
  <header class="casebook-section__header">
    <span>근거 문서</span>
    <h2 id="evidence-title">더 확인할 수 있는 근거</h2>
    <p>관심 있는 문제를 선택하면 측정 환경, 비교 과정과 한계까지 기록한 상세 문서로 이동합니다.</p>
  </header>

  <nav class="evidence-links" aria-label="상세 근거 문서">
    <a href="{{ '/reports/timescaledb-27x/' | relative_url }}"><span>데이터베이스</span><strong>TimescaleDB 조회 성능 개선</strong><small>실행 계획, 인덱스, chunk, 사전 집계 비교</small></a>
    <a href="{{ '/reports/websocket-group-canvas/' | relative_url }}"><span>실시간 처리</span><strong>WebSocket 동시 편집 안정화</strong><small>전송 순서, 중복, 지연 문제 개선</small></a>
    <a href="{{ '/reports/realtime-degrade-overview/' | relative_url }}"><span>장애 복구</span><strong>Redis, Kafka 장애 대응</strong><small>중단 중 저장과 복구 후 재처리</small></a>
    <a href="{{ '/reports/trader-data-platform/' | relative_url }}"><span>데이터 처리</span><strong>ETL 추적과 Worker 제어</strong><small>원본 보존부터 AWS ASG 자동 조절</small></a>
    <a href="{{ '/reports/slo-operating-capacity/' | relative_url }}"><span>운영 사양</span><strong>SLO 기반 운영 사양 산정</strong><small>부하와 자원 사용량의 관계 분석</small></a>
    <a href="{{ '/reports/jfr-jmc-hotpath/' | relative_url }}"><span>프로파일링</span><strong>JFR/JMC Hot Path 분석</strong><small>객체 할당과 GC 병목 확인</small></a>
  </nav>

  <div class="casebook-footer-links">
    <a href="{{ '/reports/' | relative_url }}">전체 Engineering Reports</a>
    <a href="{{ '/resume/' | relative_url }}">Resume</a>
    <a href="https://github.com/Kosw6">GitHub</a>
  </div>
</section>
