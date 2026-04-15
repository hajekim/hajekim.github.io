# hajekim.github.io — 블로그 컨텍스트

로컬 전용 파일. `.gitignore`에 등록되어 있습니다.

---

## 블로그 개요

- **URL:** https://hajekim.github.io
- **프레임워크:** Jekyll (Forever Jekyll 기반 커스텀 다크 테마)
- **호스팅:** GitHub Pages
- **저장소:** https://github.com/hajekim/hajekim.github.io
- **기본 브랜치:** `master`

---

## 테마 / 디자인

### 색상 (다크 테마)

| 변수 | 값 | 용도 |
|------|-----|------|
| `$bg-primary` | `#0d1117` | 배경 (GitHub Dark) |
| `$bg-secondary` | `#161b22` | 사이드바, 코드블록, 헤더 |
| `$bg-tertiary` | `#21262d` | 테이블 홀수행 등 |
| `$text-primary` | `#c9d1d9` | 본문 텍스트 |
| `$text-muted` | `#8b949e` | 설명, 날짜 등 보조 텍스트 |
| `$text-heading` | `#ffffff` | 제목 |
| `$accent` | `#c2410c` | 포인트 (주황/마호가니) |
| `$accent-light` | `#fb923c` | 링크, 태그 |
| `$border` | `#30363d` | 구분선 |

### 레이아웃 구조

```
[slim masthead — 상단 네비게이션 바]
  site-name (haje.log) + nav (post / resume / search)

[layout-sidebar — max-width: 1100px]
  [aside.sidebar — 240px, border-right]
    avatar (80px, 원형)
    sidebar-name / sidebar-description
    profile-bio
    profile-links (LinkedIn, GitHub)
    stack-badge 목록

  [main.main-content — flex: 1]
    {{ content }}

[footer]
  소셜 아이콘 + copyright
```

### 주요 파일

| 파일 | 역할 |
|------|------|
| `_layouts/default.html` | 전체 레이아웃 (masthead + sidebar + main) |
| `style.scss` | 메인 스타일시트 |
| `_sass/_variables.scss` | 색상·폰트·브레이크포인트 변수 |
| `_sass/_resume.scss` | Resume 페이지 전용 스타일 |
| `_config.yml` | 사이트 설정 (title, bio, nav, plugins 등) |

### _config.yml 주요 설정

```yaml
title: haje.log
description: Solutions Architect
bio: "13+년 경력의 Cloud & Data Engineer. Gemini와 Google Cloud를 주로 다룹니다."
avatar: /assets/image/avatar.png
permalink: pretty   # /YYYY/MM/DD/slug/
paginate: 5
```

### 사이드바 스택 배지 (default.html 하드코딩)

Google Cloud · Gemini · BigQuery · Vertex AI

---

## 작업 이력

### 2026-03-28 (이번 세션 포함)
- Forever Jekyll 기반 커스텀 다크 테마 적용
- 전체 페이지에 sidebar 레이아웃 적용 (max-width: 1100px, 240px 사이드바)
- 상단 slim masthead + 좌측 프로필 사이드바 구조로 재설계
- `_config.yml` defaults로 포스트 layout 자동 적용 (`layout: post`)
- jekyll-compose 설치 (포스팅 자동 생성)
- `.gitignore`에 `_site/`, `.jekyll-cache/`, `.bundle/`, `vendor/` 추가
- darkmode-js 제거 (영구 다크 테마와 충돌)
- README.md 정비
- 사이드바 스택 배지 변경: GCP/BigQuery/Kubernetes/Airflow/PySpark/Looker → Google Cloud/Gemini/BigQuery/Vertex AI
- `/search` 페이지에 카테고리 태그 필터 추가: 배지 클릭 시 SimpleJekyllSearch로 필터링, 재클릭 시 해제(토글), 활성 배지 주황색 강조
- 포스트 categories 한국어 → 영어 일괄 변환 (데이터 엔지니어링→data-engineering, 머신러닝→ml, 개발론→dev, 컨테이너→container, Javascript NodeJS→javascript)
- 포스트 카테고리 재분류 및 정리 (변경 이력):
  - 데이터 엔지니어링→data-engineering→data, 머신러닝→ml→ai, 컨테이너→container, 개발론→dev, Javascript NodeJS→dev
  - Vertex AI AutoML 1·2편: data→ai, Kubeflow/Vagrant: data→container
- 카테고리 정책: **포스트당 1개** (permalink에 포함되어 URL 구조에 영향), 세부 분류는 tags 활용
- 현재 카테고리 목록 (최종): `ai` · `data` · `container` · `dev`
- resume.md 포스트 링크 수정: `/2021/07/26/post/` → `/data-engineering/2021/07/26/post/`
- 모바일 반응형 개선:
  - `.main-content` 모바일 패딩 축소 (32px 40px → 20px 16px)
  - `width: 100%` + `overflow-x: auto` 적용 — 표/코드블록 넓을 때 콘텐츠 영역 내부에서만 가로 스크롤
  - 모바일 breakpoint: ≤ 640px (`$mobile` mixin)

### resume.md 업데이트 이력
- 구글클라우드코리아 섹션에 CE 업무 항목 추가 (최신순 정렬):
  - Agentic Design Patterns Gemini CLI Extension (2026.03) + geminicli.com 갤러리 등재
  - AmCham Korea "AI-augmented Workforce" 발표 (2026.01)
  - KOBETA 클라우드 아키텍처 설계 강의 (2023.06)
  - 디지털 네이티브/게임 고객사 기술 지원 (BigQuery, Vertex AI, Gemini 에이전트)
- 소개 문구: `10+년` → `13+년`, 현재 역할(Customer Engineer) 명시

### _config.yml / README.md 업데이트 이력
- `bio`: `"10년+ 경력..."` → `"13+년 경력의 Cloud & Data Engineer. Gemini와 Google Cloud를 주로 다룹니다."`
- README About 문구: Google Cloud Korea Customer Engineer 역할 명시

### 포스팅 작성 이력
- `2026-03-28-agentic-design-patterns-extension.md` — Gemini CLI Extension 소개
- `2026-02-12-gemini-cli-antigravity-guide.md` — Gemini CLI + Antigravity 가이드
- (Agent Harness / Ralph Loop 포스트도 작성됨)

---

## 포스팅 방법

```bash
bundle exec jekyll post "포스트 제목"
# → _posts/YYYY-MM-DD-title.md 생성
```

permalink 형식: `/YYYY/MM/DD/slug/`

---

## 로컬 개발

```bash
bundle install
bundle exec jekyll serve
```
