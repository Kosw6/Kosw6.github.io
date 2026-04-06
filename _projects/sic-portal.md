---
title: "SIC Club Portal"
layout: single
sidebar:
  nav: "main"
toc: true
toc_sticky: true
classes: wide
excerpt: "FE/BE/AI/Design 14–15명 팀 리딩 · CI/CD · 출결/게시판/백테스팅/AI퀀트봇/주식게임 모듈"
tags: [react, spring, postgresql, ci-cd, leadership, aws]
---

## 프로젝트 개요

대학 동아리 운영을 위한 클럽 포털 웹 서비스.
**14–15명(FE/BE/AI/Design) 팀을 리딩**하며 서비스 기획부터 인프라 구축까지 전 과정을 주도했습니다.

아래 서술할 내용은 제가 담당한 기간 동안 진행한 사항만 작성하였습니다.

| 항목 | 내용 |
|------|------|
| **Stack** | React · Vite PWA · Spring Boot · PostgreSQL · Redis · Docker |
| **DevOps** | GitHub Actions · JaCoCo · Grafana ·  Jira |
| **Cloud** | AWS (EC2 · S3 · CloudFront · RDS · IAM · EventBridge · Lambda) |
| **팀 규모** | 14–15명 (FE / BE / AI / Design) |
| **담당 기간** | 2025.09(개발 시작) ~ 2025.11(후임 인계) |

---

## 팀 리딩

### 워크플로우 구축

- CODEOWNERS, PR 규칙, 브랜치 전략 수립으로 **코드 레벨 책임 구조** 정착
- Jira / Slack / Notion 
- 일정 조율 및 마일스톤 관리 — 기획 · 디자인 · 개발 · 배포 각 단계 조율

### 품질 기준 수립

- GitHub Actions CI 파이프라인: PR 머지 전 빌드 · 테스트 자동 실행
- JaCoCo **≥70% 테스트 커버리지** 기준을 적용

---

## 기술 구현

### 핵심 모듈

| 모듈 | 내용 |
|------|------|
| **출결 관리** | 출결 기록 · 통계 · 예외 처리 |
| **게시판 관리** | 팀별 게시판 · 파일 업로드 · 다운 |
| **백테스팅** | 사용자가 설정한 조건으로 기간내 예상 수익치 및 수익률 곡선 |
| **AI퀀트 봇 대시보드** | 봇의 매매기록 · 총 수익률 · 기간 · 차트|
| **미니 주식 게임** | 일/주당 제시되는 주식의 상승 · 하락을 맞춰 포인트 제공 |

### 인프라


<div style="text-align:center;">
  <img src="{{ '/assets/images/sisc-architecture.png' | relative_url }}" alt="아키텍쳐 다이어그램">
</div>

- **배포**: EC2 + GitHub Actions 자동 배포
- **정적 자산**: S3 + CloudFront CDN
- **DB**: NeonDB PostgreSQL, Redis 캐싱





---

## 역할 정리

> 코드 레벨 구현보다 **팀이 효율적으로 개발할 수 있는 환경**을 만드는 데 집중했습니다.
> 인원 관리, 일정 조율, 기획/디자인/UX 방향 결정, 인프라 구성을 담당하며 서비스가 실제로 배포되고 운영되는 전 과정을 리딩했습니다.

---

## GitHub

- [SISC-IT / sisc-web](https://github.com/SISC-IT/sisc-web)
