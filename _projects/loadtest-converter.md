---
title: "LoadTest Converter"
layout: single
sidebar:
  nav: "main"
toc: true
toc_sticky: true
classes: wide
excerpt: "k6 부하테스트 시나리오 YAML 빌더 — Go API + React UI, Render 배포"
tags: [go, react, k6, loadtest, saas]
---

> **"시나리오 YAML을 직접 작성하지 않고, UI에서 조립하면 자동으로 생성되게 만들었습니다."**

---

## 프로젝트 개요

k6 부하테스트 시나리오를 GUI로 작성하고 YAML + 템플릿 ZIP으로 내보내는 웹 도구.

부하테스트를 실행하려면 시나리오 YAML 파일과 k6 스크립트 템플릿이 함께 필요한데,
이를 매번 수작업으로 작성하는 번거로움을 해소하기 위해 설계했습니다.

| 항목 | 내용 |
|------|------|
| **Stack** | Go (net/http), React (Vite), YAML |
| **배포** | Render — Go Web Service + Static Site |
| **핵심 기능** | 시나리오 빌더 UI → YAML 생성 → ZIP 내보내기 / 가져오기 |

---

## 시스템 구성

```
[React UI]  ──── POST /api/convert/preview ────▶ [Go API]
                 POST /api/convert/export           │
                 POST /api/convert/import           │
                                             YAML 생성 + ZIP 패키징
                                             (//go:embed templates)
```

- **UI (Static Site)**: React + Vite, `VITE_API_URL` 환경변수로 API 주소 주입
- **API (Web Service)**: Go 표준 라이브러리만 사용, `archive/zip` + `text/template`
- **CORS**: `github.com/rs/cors`, `AllowedOrigins: ["*"]` (stateless 공개 API)

---

## 핵심 설계 포인트

### 1. k6 템플릿 내장 (embed)

```go
//go:embed embed/templates
var embeddedTemplates embed.FS
```

빌드 결과물 단일 바이너리에 템플릿 파일 포함.
Render 배포 시 별도 파일 배포 없이 실행 가능.

### 2. ZIP 내보내기 구조

내보내기 시 생성되는 ZIP 구조:

```
scenario.zip
├── scenario.yml          ← UI에서 조립한 시나리오
├── users/{step}.csv      ← auth/k6 step용 유저 데이터
├── params/{step}.json    ← k6 step용 파라미터 데이터
└── templates/            ← k6 스크립트 템플릿 (embed에서 복사)
    ├── k6_base.js.tmpl
    └── k6_http_actions_cookie_auth.js.tmpl
```

### 3. Import / Preview / Export 분리

| 엔드포인트 | 역할 |
|-----------|------|
| `POST /api/convert/import` | YAML → UI 상태 역파싱 |
| `POST /api/convert/preview` | UI 상태 → YAML 문자열 반환 |
| `POST /api/convert/export` | UI 상태 → ZIP 바이너리 반환 |

기존 시나리오 파일을 UI로 불러와 수정 후 재내보내기 가능.

---

## 지원하는 시나리오 구조

```yaml
meta:
  name: wallet-load-test

steps:
  - id: reset
    type: k6
    load_mode: total_requests   # RPS / Duration / TotalRequests 3가지 모드
    vus: 1
    total_requests: 1
    actions:
      - method: DELETE
        path: /wallet/reset

  - id: deposit
    type: k6
    depends_on: [reset]         # wave 기반 의존성 실행
    load_mode: rps
    rps: 10
    duration: 30s
    users:
      file: users/deposit.csv
      assign: round_robin
```

- **Step 타입**: `k6`, `auth`, `command`, `final_check`
- **실행 모드**: `rps` (constant-arrival-rate) / `duration` / `total_requests` (shared-iterations)
- **의존성**: `depends_on` 기반 wave 병렬 실행

---

## 배포

| 구성 요소 | 플랫폼 | 비고 |
|-----------|--------|------|
| Go API | Render Web Service (Free) | cold start ~1–2s |
| React UI | Render Static Site | cold start 없음 |

```yaml
# render.yaml
services:
  - type: web
    name: loadtest-converter-api
    runtime: go
    rootDir: server
    buildCommand: go build -o app .

  - type: static
    name: loadtest-converter-ui
    rootDir: ui
    buildCommand: npm ci && npm run build
    envVars:
      - key: VITE_API_URL
        sync: false
```

---

## GitHub

- [loadtest-converter](https://github.com/Kosw6/loadtest-converter)
