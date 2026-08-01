---
layout: single
title: "Resume"
permalink: /resume/
classes:
  - wide
  - portfolio-index
  - portfolio-index--resume
---

## 소개

**김성원, 백엔드 개발자**

실시간 서비스와 데이터 파이프라인을 만들고, 느려지거나 중단되는 지점을 측정해 개선해 왔습니다. 해결책은 같은 조건의 부하 테스트와 장애 복구 시나리오로 다시 확인하고, 처리 상태와 결과를 기록합니다.

- **GitHub**: [github.com/Kosw6](https://github.com/Kosw6)
- **Portfolio**: [kosw6.github.io](https://kosw6.github.io)

---

## 기술 스택

| 분류 | 기술 |
|------|------|
| **Backend** | Spring Boot, Spring MVC, JPA/Hibernate, Spring Security |
| **Language** | Java, **Go** (net/http, embed, goroutine, io.Writer), Python |
| **Database** | PostgreSQL, TimescaleDB (하이퍼테이블, 청크 튜닝, CAGG), 인덱싱 전략 |
| **Cache / MQ** | Redis (TTL 설계, JSON Serializer), Kafka (Consumer Group, offset replay) |
| **Realtime** | WebSocket, Redis Pub/Sub, 서버 분산과 복구 설계 |
| **데이터 파이프라인 / Worker 제어** | DB Outbox, Kafka lag, raw/ETL 분리, lineage, idempotency |
| **Performance** | k6, JFR/JMC, Prometheus/Grafana, Grafana Alert |
| **DevOps** | Docker, Terraform, GitHub Actions, AWS (EC2/S3/CloudFront/SSM/ASG/Lambda) |
| **Frontend** | React, Vite, React Flow, WebGL 차트 |

---

## 주요 경험

### 부하 및 장애 검증 시나리오 오케스트레이터 (개인 프로젝트, 2026.04–2026.06)

수동으로 반복하던 인프라 준비, 부하 실행, 장애 주입, 복구 확인을 하나의 검증 시나리오로 실행하는 Go 기반 도구입니다.

- React/Wails UI에서 단계, 의존성, 인프라 설정과 CLI 명령을 구성하고 YAML/ZIP으로 내보내는 시나리오 작성 흐름 구현
- 분리돼 있던 시나리오 변환기와 실행기를 하나의 Go 엔진과 로컬 UI로 통합해 별도 엔진 바이너리가 없는 데스크톱 앱으로 배포
- 단계별 의존성을 계산하고 서로 독립적인 작업은 goroutine으로 병렬 실행
- 실행 로그를 파일에 저장하면서 UI에도 실시간으로 전달해 진행 상황과 실패 원인을 함께 확인
- Terraform, k6, AWS SSM을 연결해 환경 준비부터 Redis 장애 주입과 복구 확인까지 같은 순서로 반복 검증

---

### Trader (개인 프로젝트, 2025.01–현재)

[소스 코드와 저장소별 역할 보기](/projects/trader/#source-code)

주식투자 학습을 위한 복기 서비스입니다. 약 2,600만 행의 주가 조회 성능 개선에서 시작해 실시간 협업, 장애 복구, KIS, BLS, SEC 데이터 파이프라인과 Worker 제어까지 확장했습니다.

**성능 개선**
- 동일 인덱스 조건의 300 RPS, 90일 차트 조회에서 일반 테이블과 TimescaleDB 하이퍼테이블을 비교하고 청크를 튜닝해 p95를 **7,247ms에서 235ms**로 개선
- 별도의 10 RPS 실험에서 복합 인덱스 적용 효과를 측정해 p95가 **342ms에서 32ms**로 줄어드는 것을 확인
- 네 가지 조회 구조를 동일한 k6 부하에서 비교하고 p95, GC, 객체 할당량을 분석해 Fetch Join을 선택
- 그래프 목록의 1만 자 본문과 DB 20자 프리뷰를 비교해 동일 p95 구간에서 유지 가능한 RPS를 **약 5배 높이고**, 목록은 프리뷰, 원문은 상세 조회로 분리
- JFR/JMC로 객체 할당이 집중되는 경로를 추적해 반복되는 JWT 검증을 제거하고 90초 본부하의 Old GC 총 시간을 **3.47초에서 2.22초로 약 36% 감소**

**실시간 시스템**
- 동시 편집 중 같은 세션으로 전송이 겹치며 발생한 쓰기 충돌을 세션별 전송 직렬화로 제거
- 충돌 제거 후에도 200ms 이내 수신율이 0.38%에 머문 원인을 반복 전송으로 좁히고, 최신 변경만 모아 보내도록 바꿔 **99.97%**로 개선
- 그룹별 연결을 두 인스턴스에 나눠 메시지 전송을 **159K에서 79K + 79K**로 분산하고 GC와 객체 할당 감소를 JFR로 확인
- 장애 중 사용자가 작성한 내용과 서버 변경 내용을 필드별로 비교해 자동 병합 가능 여부와 충돌을 구분
- 복구된 서버가 검증 시나리오의 목표 Kafka offset까지 처리한 뒤 다시 요청을 받도록 전환 순서를 정하고, 기존 서버의 재연결 안내와 drain 흐름을 검증

**아키텍처 및 운영**
- TimescaleDB Continuous Aggregate (CAGG)의 적용 가능성을 검증하고 1W/1M/1Y 조회 전략 설계
- KIS 주가, BLS 거시경제, SEC 재무 데이터를 raw로 보존하고 Python ETL Worker가 정규화 테이블에 적재하는 파이프라인 구현
- 원본, 변환 결과, 이벤트 처리 위치를 기록해 Kafka 발행부터 ETL 완료까지 추적하고 중복 처리 여부를 판단
- Go Controller가 Kafka 대기량과 Worker 상태를 확인해 AWS ASG 인스턴스를 **0 → 1 → 0**으로 조절하는 흐름 검증
- ETL Worker 중지 중 쌓인 BLS 데이터 **2건**이 Worker 재개 후 모두 처리되는 과정 확인
- Prometheus/Grafana로 상태를 관측하고 Grafana Alert, Lambda, SSM/ASG를 연결한 자동 복구 흐름 검증
- Google/Kakao JWT SSO 구현

---

### SIC 동아리 포털 개발 리딩 및 CI/CD 구축 (팀 프로젝트, 2025.09–2025.11)

FE/BE/AI/Design 14명 팀 리딩, 동아리 운영 웹 서비스 구축

- 역할 기준(FE/BE 분리) 스프린트에서 작업 추적 및 API 의존성 문제 발생  
  → 기능 단위로 FE-BE를 묶는 스프린트 구조로 재설계하여 개발 흐름 개선

- GitHub Actions 기반 CI/CD 및 JaCoCo ≥70% 테스트 기준 도입  
  → 병합 전 오류 사전 검출 및 배포 안정성 확보

- 학업 병행으로 스프린트 지연 발생 시 테스트 기준을 일시 완화하고 기능 개발 중심으로 전략 전환  
  → MVP 일정 내 완성 및 개발 속도 회복

- EC2 t3.micro 환경에서 OOM 장애 발생 → free -h 기반 원인 분석  
  → t3.medium + 1GB swap 적용으로 안정화 및 인프라 운영 기준 수립

- AWS EC2/S3/CloudFront + GitHub Actions 기반 인프라 설계 및 배포 자동화<br>
  → EventBridge + Lambda로 운영 시간 제어(09–18시)하여 비용 최적화


---

## 학력

- **세종대학교** (Sejong University)

---

## 링크

| | |
|--|--|
| GitHub | [github.com/Kosw6](https://github.com/Kosw6) |
| Portfolio | [kosw6.github.io](https://kosw6.github.io) |
| Engineering Reports | [kosw6.github.io/reports](https://kosw6.github.io/reports/) |
