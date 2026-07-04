# LastFrame Extractor Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 영상 파일을 드롭하면 마지막(또는 끝부분에서 고른) 프레임을 원본 해상도 PNG로 다운로드하는 단일 HTML 파일 웹앱.

**Architecture:** 외부 의존성 0인 단일 `lastframe.html`. `<video>`로 디코딩, `requestVideoFrameCallback`(rVFC)로 실제 표시 프레임의 `mediaTime`을 추적해 프레임 단위 이동, `<canvas>`에 원본 해상도로 그려 PNG 인코딩.

**Tech Stack:** Vanilla HTML/CSS/JS, rVFC API (Chrome/Edge), Canvas `toBlob`.

## Global Constraints

- 파일 하나(`lastframe.html`), 외부 네트워크 요청 0, 빌드 없음.
- 영상은 로컬에서만 처리 (업로드 없음).
- UI 문구는 한국어.
- 출력: PNG, 파일명 `원본이름_frame.png`.

---

### Task 1: lastframe.html 구현

**Files:**
- Create: `lastframe.html`

**Interfaces:**
- Produces: 브라우저에서 열리는 완성된 웹앱. 핵심 내부 함수 —
  `loadFile(file)`, `sampleFrameRate(): Promise<number>` (frameDuration 초 반환),
  `stepFrame(dir: -1|1)`, `seekTo(t: number)`, `downloadPng()`.

- [ ] **Step 1: 구현** — 아래 동작을 모두 포함한 단일 HTML 작성:
  - 드래그&드롭 + 클릭 파일 선택. `video/*`가 아니고 확장자도 영상이 아니면 안내 메시지.
  - 로드 후: 음소거 재생으로 rVFC `mediaTime` 델타를 ~10개 수집, 중앙값으로 frameDuration 추정(실패 시 1/30). 그 후 `duration`으로 seek → 마지막 프레임 표시.
  - 컨트롤: ◀/▶ 프레임 스텝(현재 mediaTime ± frameDuration로 seek), 키보드 ←/→, 미세 슬라이더(기본: 끝 3초 구간, "전체" 토글), 현재 시각 표시.
  - 다운로드: canvas를 `videoWidth×videoHeight`로 만들어 `drawImage` → `toBlob('image/png')` → `원본이름_frame.png`.
  - 에러: `video.onerror` → "이 형식은 브라우저에서 재생할 수 없습니다" 표시.
  - rVFC 미지원 브라우저 → 안내 후 1/30 스텝으로 동작(폴백).
- [ ] **Step 2: Commit** — `git add lastframe.html && git commit -m "feat: LastFrame extractor single-file web app"`

### Task 2: 브라우저 검증

**Files:**
- Test: Playwright로 `lastframe.html` 열어 실제 동작 확인 (스크래치패드에 샘플 webm 생성)

**Interfaces:**
- Consumes: Task 1의 완성된 `lastframe.html`.

- [ ] **Step 1: 샘플 영상 생성** — Playwright 페이지에서 canvas `captureStream` + `MediaRecorder`로 프레임 번호가 그려진 2초 webm을 만들어 스크래치패드에 저장.
- [ ] **Step 2: 동작 검증** — `lastframe.html`에 샘플 파일 업로드 → 마지막 프레임 자동 표시 확인 → ◀ 스텝 시 시각 감소 확인 → 다운로드 클릭 → PNG 파일 생성·크기 확인.
- [ ] **Step 3: 발견된 버그 수정 후 재검증, 커밋**
