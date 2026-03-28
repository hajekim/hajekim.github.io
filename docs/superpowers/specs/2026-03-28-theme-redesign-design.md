# Theme Redesign: Dark Theme + Personal Profile + Blog

**Date:** 2026-03-28
**Status:** Approved by user

---

## 1. 목표

현재 Forever Jekyll 테마를 개선하여 **개인 이력(resume)과 블로그(blog)가 자연스럽게 공존**하는 사이트로 만든다. al-folio의 장점(다크 테마, 타임라인 이력서, 기술 스택 시각화)과 현재 테마의 장점(미니멀한 포스트 목록, 심플한 네비게이션)을 결합한다.

---

## 2. 디자인 결정 사항

| 항목 | 결정 | 근거 |
|---|---|---|
| 전체 색조 | **다크 테마** (`#0d1117` 배경) | 개발자 감성, 현대적 느낌 |
| 포인트 색상 | **오렌지/마호가니** (`#c2410c` 버튼·뱃지, `#fb923c` 링크·아이콘) | 현재 테마 마호가니(`#BD2C00`) 계승 |
| 홈 레이아웃 | **프로필 카드 + 미니멀 포스트 목록** | 방문자에게 즉시 인물 소개 후 콘텐츠 제공 |
| 이력서 레이아웃 | **수직 타임라인** | al-folio 스타일, 커리어 흐름 시각화 |
| 포스트 목록 | **미니멀 리스트** | 현재 구조 계승, 읽기 편함 |

---

## 3. 색상 팔레트

```scss
// 배경
$bg-primary:    #0d1117;   // 페이지 배경
$bg-secondary:  #161b22;   // 카드, 코드블록 배경
$bg-tertiary:   #21262d;   // 구분선, 호버

// 텍스트
$text-primary:  #c9d1d9;   // 본문
$text-muted:    #8b949e;   // 날짜, 부제목
$text-heading:  #ffffff;   // H1~H3

// 포인트 (오렌지/마호가니)
$accent:        #c2410c;   // 뱃지 배경, 타임라인 점, 강조선
$accent-light:  #fb923c;   // 링크, 아이콘, 태그 텍스트
$accent-hover:  #ea580c;   // 호버

// 테두리
$border:        #30363d;   // 카드 테두리, 구분선
$border-accent: #c2410c;   // 현재 회사, 최신 포스트 강조선
```

---

## 4. 컴포넌트별 설계

### 4-1. 헤더 (masthead)

- 배경: `$bg-secondary` (`#161b22`)
- 좌측: 아바타(원형 40px) + 사이트명 + 한 줄 설명
- 우측: 네비게이션 링크 (post · resume · search)
- 하단 구분선: `$border`
- 현재 페이지 링크: `$accent-light` 색상

### 4-2. 홈페이지 — 프로필 카드

헤더 바로 아래, 포스트 목록 위에 삽입되는 프로필 카드.

```
┌─────────────────────────────────────────┐
│  [avatar]  김하제                        │
│            Cloud · Data · SA            │
│            구글코리아 · 10년+             │
│  [GitHub] [LinkedIn] [Resume →]         │
│                                          │
│  GCP  BigQuery  Kubernetes  Airflow     │  ← 기술 스택 뱃지
└─────────────────────────────────────────┘
```

- 배경: `$bg-secondary`, 테두리: `$border`
- 아바타: 원형, 56px
- 기술 스택 뱃지: `border: 1px solid $accent`, `color: $accent-light`, `border-radius: 12px`
- Resume 버튼: `background: $accent`, `color: white`
- GitHub/LinkedIn 버튼: `border: 1px solid $border`, `color: $text-muted`

### 4-3. 홈페이지 — 포스트 목록 (미니멀 리스트)

```
제목 (H2, $text-heading, bold)
날짜 · N min read  ($text-muted, 10px)
요약 텍스트 두 줄   ($text-muted)
[태그1] [태그2]    (뱃지, $accent 계열)
─────────────────── ($border 구분선)
```

- 첫 번째 포스트(가장 최신) 제목만 `$accent-light` 색상, 나머지는 `$text-heading`
- 태그 뱃지: `border: 1px solid $accent`, `color: $accent-light`
- 구분선: `border-bottom: 1px solid $border`
- 페이지네이션 유지 (5개/페이지)

### 4-4. 이력서 페이지 — 수직 타임라인

레이아웃: 단일 컬럼, 최대 너비 740px 유지

```
● 구글코리아                     2022.04~현재
│  Customer Engineer, Data Analytics
│  · 세부 프로젝트 1
│  · 세부 프로젝트 2
│
● GS홈쇼핑                      2021.06~2022.03
│  Data Engineer
│  ...
```

- 타임라인 점: `$accent` (#c2410c), 10px 원형
- 타임라인 세로선: `$border`
- 회사명: `$text-heading`, bold
- 직책: `$accent-light`
- 기간: `$text-muted`, 우측 정렬
- 세부항목: `$text-primary`

페이지 하단 섹션 순서:
1. Work Experience (타임라인)
2. Education (타임라인 동일 스타일)
3. Tech Stack (뱃지 그룹: Cloud / Data / Infra / Middleware)

### 4-5. 코드 블록

현재 Solarized Light → 다크 테마로 전환:
- 배경: `$bg-secondary` (`#161b22`)
- 테두리: `$border`
- 텍스트: 다크 신택스 팔레트 (GitHub Dark 계열)

### 4-6. 푸터

- 배경: `$bg-secondary`
- 아이콘: LinkedIn, GitHub (`$text-muted`, 호버 시 `$accent-light`)
- 저작권 텍스트: `$text-muted`

---

## 5. 구현 범위

### 변경 대상 파일

| 파일 | 변경 내용 |
|---|---|
| `_sass/_variables.scss` | 색상 팔레트 전면 교체 |
| `style.scss` | 배경·텍스트·링크·코드블록 다크 테마 적용 |
| `_sass/_syntax.scss` | 신택스 하이라이트 다크 테마로 교체 |
| `_layouts/default.html` | 헤더/푸터 다크 스타일 클래스 적용 |
| `_layouts/page.html` | 변경 없음 — archive, search 등 다른 페이지에서 계속 사용 |
| `index.html` | 프로필 카드 컴포넌트 삽입 |
| `resume.md` | 레이아웃을 `resume`으로 변경, Tech Stack 내용 채우기 |
| `_layouts/resume.html` | **신규**: 수직 타임라인 전용 레이아웃 |
| `_includes/profile-card.html` | **신규**: 홈 프로필 카드 컴포넌트 |
| `_sass/_resume.scss` | **신규**: 타임라인 스타일 |
| `_config.yml` | `description` 업데이트 |

### 변경하지 않는 것

- 포스트 마크다운 파일 내용 (현재 13개 포스트)
- 플러그인 구성 (Gemfile)
- 네비게이션 구조 (post · resume · search)
- 페이지네이션 (5개/페이지)
- 다크모드 토글 위젯 (darkmode-js, 현재 동작 중)

---

## 6. 구현 순서

1. `_variables.scss` — 색상 팔레트 교체 (기반 작업)
2. `style.scss` — body, a, code, table 등 전역 다크 스타일
3. `_syntax.scss` — 코드 신택스 하이라이트 다크 전환
4. `_includes/profile-card.html` — 프로필 카드 컴포넌트 신규 작성
5. `index.html` — 프로필 카드 삽입
6. `_layouts/resume.html` + `_sass/_resume.scss` — 타임라인 레이아웃
7. `resume.md` — 레이아웃 변경 + Tech Stack 내용 채우기
8. `_layouts/default.html` / `_config.yml` — 마무리 조정

---

## 7. 비고

- `.superpowers/` 를 `.gitignore`에 추가할 것
- 로컬 빌드(`bundle exec jekyll serve`)로 각 단계 검증 후 커밋
