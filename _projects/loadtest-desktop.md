---
title: "LoadTest Desktop - Orchestrator 실행기 단계"
layout: single
toc: true
toc_sticky: true
classes: wide
excerpt: "k6 부하테스트 데스크탑 실행기 — Go + Wails, 엔진 내장, 실시간 로그 스트리밍"
tags: [go, wails, k6, loadtest, desktop, concurrency]
---

> 현재는 [부하·장애 검증 시나리오 오케스트레이터](/projects/loadtest-orchestrator/)의 로컬 UI와 실행 엔진으로 통합했습니다. 이 페이지는 별도 엔진 바이너리를 제거하고 실시간 로그 처리를 구현한 기록입니다.

> **"별도 Go 엔진 바이너리 없이, UI와 실행 엔진을 하나의 데스크톱 앱으로 제공합니다."**

---

## 프로젝트 개요

LoadTest Converter에서 내보낸 ZIP 파일을 로드해 k6 부하테스트를 실행하는 데스크탑 앱.

초기에는 오케스트레이터 바이너리를 별도로 선택해야 했는데,
이를 단일 실행파일로 통합하기 위해 엔진 패키지를 앱 내부에 흡수했습니다.

| 항목 | 내용 |
|------|------|
| **Stack** | Go (Wails v2), React (Vite), k6 |
| **배포 형태** | UI와 Go 실행 엔진을 통합한 단일 `.exe` |
| **핵심 구조** | `engine/` 패키지 내장 + `//go:embed templates` |

시나리오에서 사용하는 k6, Terraform, AWS CLI 등 외부 도구는 사용자 환경에 별도로 설치되어 있어야 합니다.

---

## 아키텍처 — 엔진 통합

```
loadtest-desktop/
├── main.go          ← Wails 엔트리
├── app.go           ← Wails 바인딩 (RunTest, LoadProject 등)
└── engine/
    ├── engine.go    ← Run(cfg, w io.Writer) 공개 API
    ├── executor.go  ← wave 기반 병렬 스텝 실행
    ├── k6.go        ← k6 실행 + 실시간 스트리밍
    ├── auth.go      ← auth step (users.csv → 로그인)
    ├── scenario.go  ← YAML 로드 + 정규화
    └── templates/   ← //go:embed로 바이너리에 내장
```

### 통합 전 vs 후

| 항목 | 통합 전 | 통합 후 |
|------|---------|---------|
| 실행 파일 수 | 앱 + 엔진 2개 | **1개** |
| 엔진 경로 선택 | 사용자가 직접 선택 | 불필요 |
| 템플릿 관리 | 엔진 바이너리에 번들 | `//go:embed` 내장 |
| mpb 터미널 프로그레스 | 터미널 전용 | **io.Writer로 교체** |

---

## 핵심 설계 포인트

### 1. io.Writer 기반 실시간 스트리밍

엔진의 모든 출력이 `io.Writer` 하나로 추상화되어 있습니다.

```go
// engine.Run() 시그니처
func Run(cfg Config, w io.Writer) error

// app.go — wailsWriter가 io.Writer를 구현
type wailsWriter struct {
    mu      sync.Mutex
    ctx     context.Context
    buf     strings.Builder   // 전체 출력 버퍼
    lineBuf strings.Builder   // 줄 단위 이벤트 발행용
}

func (w *wailsWriter) Write(p []byte) (n int, err error) {
    w.mu.Lock()
    defer w.mu.Unlock()
    w.buf.Write(p)
    // 줄 단위로 분리해 Wails 이벤트로 프론트엔드에 전달
    for _, b := range p {
        if b == '\n' {
            runtime.EventsEmit(w.ctx, "run:log", w.lineBuf.String())
            w.lineBuf.Reset()
        } else {
            w.lineBuf.WriteByte(b)
        }
    }
    return len(p), nil
}
```

- 병렬 스텝 실행 시 여러 고루틴이 동시에 Write() → `sync.Mutex`로 스레드 안전 보장
- 프론트엔드는 `"run:log"` 이벤트를 구독해 터미널 UI에 실시간 출력

### 2. k6 출력 동시 기록 (io.MultiWriter)

```go
// k6 stdout/stderr → 로그 파일 저장 + UI 실시간 스트리밍 동시 처리
combined := io.MultiWriter(logFile, w)
cmd.Stdout = combined
cmd.Stderr = combined
```

### 3. wave 기반 병렬 스텝 실행

```go
// depends_on이 없는 스텝들을 같은 wave로 묶어 goroutine 병렬 실행
// wave 완료 후 다음 wave 진행 (WaitGroup + channel 에러 수집)

for _, step := range readySteps {
    wg.Add(1)
    go func(step StepConfig) {
        defer wg.Done()
        err := executeStep(step, ..., w)
        if err != nil { errCh <- err }
    }(step)
}
wg.Wait()
```

### 4. 템플릿 실행 시 임시 추출

```go
//go:embed templates
var embeddedTemplates embed.FS

func Run(cfg Config, w io.Writer) error {
    // 실행 시 임시 디렉토리에 추출 → 기존 파일 경로 기반 코드 변경 없이 재사용
    tmpDir, _ := os.MkdirTemp("", "loadtest-engine-*")
    defer os.RemoveAll(tmpDir)
    extractEmbeddedTemplates(tmpDir)
    ...
}
```

---

## 지원 Step 타입

| 타입 | 동작 |
|------|------|
| `k6` | k6 스크립트 생성 후 실행, summary.json 저장 |
| `auth` | users.csv 읽어 각 유저 로그인 → auth_context.json 저장 |
| `command` | shell 명령 실행 (OS별 분기) |
| `final_check` | HTTP 엔드포인트 상태 검증 (assert) |

---

## 실행 흐름

```
ZIP 로드 → scenario.yml 파싱
    │
    └─ wave 계산 (depends_on 위상 정렬)
         │
         ├─ wave 1: [auth step] ──────────────────────── (동기)
         └─ wave 2: [k6 step A] ┐
                    [k6 step B] ┤ goroutine 병렬 실행
                    [k6 step C] ┘
                         │
                    실행 완료 → summary.json + k6.log 저장
                         │
                    report.md + report.html 자동 생성
```

---

## GitHub

- [loadtest-desktop](https://github.com/Kosw6/loadtest-desktop)
