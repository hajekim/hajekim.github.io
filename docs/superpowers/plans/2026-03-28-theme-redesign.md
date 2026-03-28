# Theme Redesign Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Forever Jekyll 테마를 다크 테마로 교체하고, 홈에 프로필 카드를 추가하며, 이력서 페이지를 수직 타임라인 레이아웃으로 개선한다.

**Architecture:** `_sass/_variables.scss`의 색상 팔레트 교체를 기반으로 전역 스타일을 다크로 전환한다. 프로필 카드는 `_includes/profile-card.html` 컴포넌트로 분리하고, 이력서는 `_layouts/resume.html` + `_sass/_resume.scss` 신규 파일로 전용 레이아웃을 제공한다.

**Tech Stack:** Jekyll, SCSS, Liquid, GitHub Pages

---

## 파일 맵

| 파일 | 작업 |
|---|---|
| `.gitignore` | `.superpowers/` 추가 |
| `_sass/_variables.scss` | 색상 팔레트 전면 교체 |
| `style.scss` | body/a/code/table/header/footer 다크 스타일 |
| `_sass/_syntax.scss` | Solarized Light → GitHub Dark |
| `_includes/profile-card.html` | **신규**: 홈 프로필 카드 컴포넌트 |
| `index.html` | 프로필 카드 삽입 + 태그 뱃지 추가 |
| `_layouts/resume.html` | **신규**: 수직 타임라인 전용 레이아웃 |
| `_sass/_resume.scss` | **신규**: 타임라인 스타일 |
| `resume.md` | `layout: resume` 변경 + Tech Stack 내용 채우기 |

---

## Task 1: .gitignore + 색상 팔레트 교체

**Files:**
- Modify: `.gitignore`
- Modify: `_sass/_variables.scss`

- [ ] **Step 1: .gitignore에 .superpowers/ 추가**

`.gitignore` 파일에 아래 줄을 추가 (파일이 없으면 생성):

```
.superpowers/
```

- [ ] **Step 2: _variables.scss 색상 팔레트 전면 교체**

`_sass/_variables.scss` 전체를 아래로 교체:

```scss
@charset "utf-8";

//
// VARIABLES — Dark Theme (GitHub Dark + Orange/Mahogany)
//

// Backgrounds
$bg-primary:    #0d1117;
$bg-secondary:  #161b22;
$bg-tertiary:   #21262d;

// Text
$text-primary:  #c9d1d9;
$text-muted:    #8b949e;
$text-heading:  #ffffff;

// Accent (Orange / Mahogany)
$accent:        #c2410c;
$accent-light:  #fb923c;
$accent-hover:  #ea580c;

// Borders
$border:        #30363d;

// Legacy aliases (기존 코드 호환)
$white:         #ffffff;
$black:         #000000;
$gray:          $text-muted;
$lightGray:     $bg-tertiary;
$paleGray:      $border;
$darkGray:      $text-primary;
$darkerGray:    $text-heading;
$blue:          #58a6ff;
$purple:        #a371f7;
$mahogany:      $accent;

// Pagination colors
$pagination-color:       $text-primary;
$pagination-hover-color: $accent-light;
$pagination-active-bg:   $accent;
$pagination-active-color: $white;
$pagination-separator-color: $border;

// Font stacks
$sans-font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", "Noto Sans", Roboto, "Droid Sans", Arimo, "Helvetica Neue", Helvetica, "Liberation Sans", Arial, Ubuntu, "DejaVu Sans", "Segoe UI Symbol", "Segoe UI Emoji", "Apple Color Emoji", sans-serif;
$code-font-family: ui-monospace, "Cascadia Mono", "Segoe UI Mono", Consolas, Menlo, Monaco, "Noto Sans Mono", "Noto Mono", "Roboto Mono", "Droid Sans Mono", Cousine, Inconsolata, "Liberation Mono", "Ubuntu Mono", "DejaVu Sans Mono", "Courier New", monospace;

// Mobile breakpoints
@mixin mobile {
  @media screen and (max-width: 640px) {
    @content;
  }
}

// Transitions
$time: 250ms;
```

- [ ] **Step 3: 빌드 확인**

```bash
cd /Users/haje/sandbox/hajekim.github.io && bundle exec jekyll build 2>&1 | tail -5
```

기대 출력: `done in X seconds` (에러 없음)

- [ ] **Step 4: 커밋**

```bash
cd /Users/haje/sandbox/hajekim.github.io
git add .gitignore _sass/_variables.scss
git commit -m "refactor: replace color palette with dark theme variables

- Switch from light theme colors to GitHub Dark base
- Accent: orange/mahogany (#c2410c / #fb923c)
- Add legacy aliases for backward compatibility

TEST=bundle exec jekyll build (no errors)

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
```

---

## Task 2: 전역 다크 스타일 (style.scss)

**Files:**
- Modify: `style.scss`

- [ ] **Step 1: body 배경·텍스트 다크로 변경**

`style.scss`의 `body { ... }` 블록에서 아래 두 줄을 변경:

변경 전:
```scss
body {
  background: $white;
  ...
  color: $darkGray;
```

변경 후:
```scss
body {
  background: $bg-primary;
  ...
  color: $text-primary;
```

- [ ] **Step 2: h1~h6 색상 변경**

변경 전:
```scss
h1, h2, h3, h4, h5, h6 {
  margin: 01em 0 15px;
  color: $darkerGray;
```

변경 후:
```scss
h1, h2, h3, h4, h5, h6 {
  margin: 01em 0 15px;
  color: $text-heading;
```

- [ ] **Step 3: 링크 색상 변경**

변경 전:
```scss
a {
  color: $blue;
  text-decoration: none;

  &:visited {
    color: $purple;
  }
```

변경 후:
```scss
a {
  color: $accent-light;
  text-decoration: none;

  &:visited {
    color: $accent-light;
  }
```

- [ ] **Step 4: blockquote 다크 스타일**

변경 전:
```scss
blockquote {
  background-color: $white;
  color: $gray;
  border-left: 04px solid #eeeeee;
```

변경 후:
```scss
blockquote {
  background-color: $bg-secondary;
  color: $text-muted;
  border-left: 04px solid $accent;
```

- [ ] **Step 5: code/pre 다크 스타일**

변경 전:
```scss
pre,
code {
  font-family: $code-font-family !important;
  font-size: 15px !important;
  color: #586e75;
  border: 01px solid #dddddd;
  border-radius: 03px;
  background-color: #fdf6e3;
}
```

변경 후:
```scss
pre,
code {
  font-family: $code-font-family !important;
  font-size: 15px !important;
  color: #e6edf3;
  border: 01px solid $border;
  border-radius: 03px;
  background-color: $bg-secondary;
}
```

- [ ] **Step 6: .highlight 다크 스타일**

변경 전:
```scss
.highlight {
  isolation: isolate;
  border-radius: 03px;
  background: #fdf6e3;
  @extend %vertical-rhythm;

  .highlighter-rouge & {
    background: #fdf6e3;
  }
}
```

변경 후:
```scss
.highlight {
  isolation: isolate;
  border-radius: 03px;
  background: $bg-secondary;
  @extend %vertical-rhythm;

  .highlighter-rouge & {
    background: $bg-secondary;
  }
}
```

- [ ] **Step 7: table 다크 스타일**

변경 전:
```scss
table {
  ...
  border: 01px solid #dddddd;
  ...
  tr {
    &:nth-child(even) {
      background-color: #efefef;
    }
  }

  th, td {
    padding: 10px 15px;
  }

  th {
    background-color: #efefef;
    border: 01px solid #dddddd;
  }

  td {
    border: 01px solid #dddddd;
  }
```

변경 후:
```scss
table {
  ...
  border: 01px solid $border;
  ...
  tr {
    &:nth-child(even) {
      background-color: $bg-tertiary;
    }
  }

  th, td {
    padding: 10px 15px;
  }

  th {
    background-color: $bg-tertiary;
    border: 01px solid $border;
  }

  td {
    border: 01px solid $border;
  }
```

- [ ] **Step 8: masthead 다크 스타일**

변경 전:
```scss
.masthead {
  display: flex;
  align-items: center;
  padding: 20px 0;
  border-bottom: 01px solid $lightGray;
```

변경 후:
```scss
.masthead {
  display: flex;
  align-items: center;
  padding: 20px 0;
  border-bottom: 01px solid $border;
  background-color: $bg-secondary;
```

- [ ] **Step 9: wrapper-masthead 배경 추가**

변경 전:
```scss
.wrapper-masthead {
  margin-bottom: auto;
}
```

변경 후:
```scss
.wrapper-masthead {
  margin-bottom: auto;
  background-color: $bg-secondary;
}
```

- [ ] **Step 10: site-name, site-description 색상**

변경 전:
```scss
.site-name {
  margin: 0;
  color: $darkGray;
```

변경 후:
```scss
.site-name {
  margin: 0;
  color: $text-heading;
```

변경 전:
```scss
.site-description {
  margin: -05px 0 0 0;
  color: $gray;
```

변경 후:
```scss
.site-description {
  margin: -05px 0 0 0;
  color: $text-muted;
```

- [ ] **Step 11: nav 링크 색상**

변경 전:
```scss
nav {
  ...
  a {
    margin-left: 20px;
    color: $darkGray;
```

변경 후:
```scss
nav {
  ...
  a {
    margin-left: 20px;
    color: $text-primary;
```

- [ ] **Step 12: footer 다크 스타일**

변경 전:
```scss
.wrapper-footer {
  margin-top: 30px;
  border-top: 01px solid #ddd;
  border-bottom: 01px solid #ddd;
  background-color: $lightGray;
}
```

변경 후:
```scss
.wrapper-footer {
  margin-top: 30px;
  border-top: 01px solid $border;
  border-bottom: 01px solid $border;
  background-color: $bg-secondary;
}
```

- [ ] **Step 13: hr 색상**

변경 전:
```scss
hr {
  position: relative;
  margin-top: 35px;
  margin-bottom: 35px;
  border: 01px solid $lightGray;
}
```

변경 후:
```scss
hr {
  position: relative;
  margin-top: 35px;
  margin-bottom: 35px;
  border: 01px solid $border;
}
```

- [ ] **Step 14: 빌드 확인**

```bash
cd /Users/haje/sandbox/hajekim.github.io && bundle exec jekyll build 2>&1 | tail -5
```

기대 출력: `done in X seconds` (에러 없음)

- [ ] **Step 15: 커밋**

```bash
cd /Users/haje/sandbox/hajekim.github.io
git add style.scss
git commit -m "style: apply dark theme to global styles

- body background #0d1117, text #c9d1d9
- links accent-light (#fb923c)
- blockquote, code, pre, table all dark
- masthead/footer bg-secondary (#161b22)

TEST=bundle exec jekyll build (no errors)

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
```

---

## Task 3: 신택스 하이라이트 다크 전환

**Files:**
- Modify: `_sass/_syntax.scss`

- [ ] **Step 1: _syntax.scss 전체를 GitHub Dark 팔레트로 교체**

`_sass/_syntax.scss` 전체 내용을 아래로 교체:

```scss
/* GitHub Dark syntax highlighting */

.highlight { background-color: #161b22; color: #e6edf3 }
.highlight .c  { color: #8b949e; font-style: italic } /* Comment */
.highlight .err { color: #f85149 } /* Error */
.highlight .k  { color: #ff7b72 } /* Keyword */
.highlight .l  { color: #79c0ff } /* Literal */
.highlight .n  { color: #e6edf3 } /* Name */
.highlight .o  { color: #ff7b72 } /* Operator */
.highlight .p  { color: #e6edf3 } /* Punctuation */
.highlight .cm { color: #8b949e; font-style: italic } /* Comment.Multiline */
.highlight .cp { color: #ff7b72 } /* Comment.Preproc */
.highlight .c1 { color: #8b949e; font-style: italic } /* Comment.Single */
.highlight .cs { color: #8b949e; font-style: italic } /* Comment.Special */
.highlight .gd { color: #ffa198; background-color: #490202 } /* Generic.Deleted */
.highlight .ge { font-style: italic } /* Generic.Emph */
.highlight .gh { color: #79c0ff; font-weight: bold } /* Generic.Heading */
.highlight .gi { color: #56d364; background-color: #0f5323 } /* Generic.Inserted */
.highlight .gp { color: #8b949e } /* Generic.Prompt */
.highlight .gs { font-weight: bold } /* Generic.Strong */
.highlight .gu { color: #79c0ff } /* Generic.Subheading */
.highlight .kc { color: #79c0ff } /* Keyword.Constant */
.highlight .kd { color: #ff7b72 } /* Keyword.Declaration */
.highlight .kn { color: #ff7b72 } /* Keyword.Namespace */
.highlight .kp { color: #79c0ff } /* Keyword.Pseudo */
.highlight .kr { color: #ff7b72 } /* Keyword.Reserved */
.highlight .kt { color: #ff7b72 } /* Keyword.Type */
.highlight .m  { color: #79c0ff } /* Literal.Number */
.highlight .s  { color: #a5d6ff } /* Literal.String */
.highlight .na { color: #79c0ff } /* Name.Attribute */
.highlight .nb { color: #e6edf3 } /* Name.Builtin */
.highlight .nc { color: #f0883e } /* Name.Class */
.highlight .no { color: #79c0ff } /* Name.Constant */
.highlight .nd { color: #d2a8ff } /* Name.Decorator */
.highlight .ni { color: #e6edf3 } /* Name.Entity */
.highlight .ne { color: #f0883e } /* Name.Exception */
.highlight .nf { color: #d2a8ff } /* Name.Function */
.highlight .nl { color: #79c0ff } /* Name.Label */
.highlight .nn { color: #ff7b72 } /* Name.Namespace */
.highlight .nt { color: #7ee787 } /* Name.Tag */
.highlight .nv { color: #79c0ff } /* Name.Variable */
.highlight .ow { color: #ff7b72 } /* Operator.Word */
.highlight .w  { color: #e6edf3 } /* Text.Whitespace */
.highlight .mf { color: #79c0ff } /* Literal.Number.Float */
.highlight .mh { color: #79c0ff } /* Literal.Number.Hex */
.highlight .mi { color: #79c0ff } /* Literal.Number.Integer */
.highlight .mo { color: #79c0ff } /* Literal.Number.Oct */
.highlight .sb { color: #a5d6ff } /* Literal.String.Backtick */
.highlight .sc { color: #a5d6ff } /* Literal.String.Char */
.highlight .sd { color: #8b949e } /* Literal.String.Doc */
.highlight .s2 { color: #a5d6ff } /* Literal.String.Double */
.highlight .se { color: #79c0ff } /* Literal.String.Escape */
.highlight .si { color: #a5d6ff } /* Literal.String.Interpol */
.highlight .sx { color: #a5d6ff } /* Literal.String.Other */
.highlight .sr { color: #a5d6ff } /* Literal.String.Regex */
.highlight .s1 { color: #a5d6ff } /* Literal.String.Single */
.highlight .ss { color: #a5d6ff } /* Literal.String.Symbol */
.highlight .bp { color: #e6edf3 } /* Name.Builtin.Pseudo */
.highlight .vc { color: #79c0ff } /* Name.Variable.Class */
.highlight .vg { color: #79c0ff } /* Name.Variable.Global */
.highlight .vi { color: #79c0ff } /* Name.Variable.Instance */
.highlight .il { color: #79c0ff } /* Literal.Number.Integer.Long */
```

- [ ] **Step 2: 빌드 확인**

```bash
cd /Users/haje/sandbox/hajekim.github.io && bundle exec jekyll build 2>&1 | tail -5
```

기대 출력: `done in X seconds` (에러 없음)

- [ ] **Step 3: 커밋**

```bash
cd /Users/haje/sandbox/hajekim.github.io
git add _sass/_syntax.scss
git commit -m "style: replace Solarized Light with GitHub Dark syntax highlighting

TEST=bundle exec jekyll build (no errors)

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
```

---

## Task 4: 프로필 카드 컴포넌트

**Files:**
- Create: `_includes/profile-card.html`
- Modify: `_config.yml` (bio 필드 추가)

- [ ] **Step 1: _config.yml에 bio 필드 추가**

`_config.yml`의 `description: Solutions Architect` 바로 아래에 추가:

```yaml
bio: "10년+ 경력의 Cloud & Data Engineer. GCP, BigQuery, Kubernetes를 주로 다룹니다."
```

- [ ] **Step 2: profile-card.html 생성**

`_includes/profile-card.html` 파일을 아래 내용으로 생성:

```html
<div class="profile-card">
  <div class="profile-card-inner">
    <div class="profile-avatar">
      <a href="{{ site.baseurl }}/"><img src="{{ site.avatar }}" alt="{{ site.title }}" /></a>
    </div>
    <div class="profile-info">
      <h2 class="profile-name">{{ site.title }}</h2>
      <p class="profile-description">{{ site.description }}</p>
      {% if site.bio %}
      <p class="profile-bio">{{ site.bio }}</p>
      {% endif %}
      <div class="profile-links">
        {% for link in site.footer_links %}
        {% if link.url contains "://" %}
        {% assign url = link.url %}
        {% else %}
        {% assign url = link.url | relative_url %}
        {% endif %}
        <a class="profile-link-btn" href="{{ url }}" title="{{ link.title }}">
          <i class="{{ link.icon | default: 'fa fa-link' }}"></i>&nbsp;{{ link.title }}
        </a>
        {% endfor %}
        <a class="profile-link-btn profile-link-btn--accent" href="{{ site.baseurl }}/resume">Resume →</a>
      </div>
      <div class="profile-stack">
        <span class="stack-badge">GCP</span>
        <span class="stack-badge">BigQuery</span>
        <span class="stack-badge">Kubernetes</span>
        <span class="stack-badge">Airflow</span>
        <span class="stack-badge">PySpark</span>
        <span class="stack-badge">Looker</span>
      </div>
    </div>
  </div>
</div>
```

- [ ] **Step 3: profile-card 스타일을 style.scss에 추가**

`style.scss`의 `@import "message";` 줄 바로 위에 추가:

```scss
// Profile Card (homepage)
.profile-card {
  background-color: $bg-secondary;
  border: 01px solid $border;
  border-radius: 06px;
  padding: 20px;
  margin-bottom: 30px;
}

.profile-card-inner {
  display: flex;
  gap: 20px;
  align-items: flex-start;

  @include mobile {
    flex-direction: column;
    align-items: center;
    text-align: center;
  }
}

.profile-avatar {
  flex-shrink: 0;

  img {
    width: 72px;
    height: 72px;
    border-radius: 50%;
    border: 02px solid $border;
  }
}

.profile-info {
  flex: 1;
}

.profile-name {
  margin: 0 0 02px !important;
  font-size: 20px !important;
  color: $text-heading;
}

.profile-description {
  margin: 0 0 04px !important;
  color: $accent-light;
  font-size: 14px !important;
  font-weight: 600;
}

.profile-bio {
  margin: 0 0 12px !important;
  color: $text-muted;
  font-size: 14px !important;
  line-height: 1.5;
}

.profile-links {
  display: flex;
  flex-wrap: wrap;
  gap: 06px;
  margin-bottom: 12px;

  @include mobile {
    justify-content: center;
  }
}

.profile-link-btn {
  display: inline-flex;
  align-items: center;
  border: 01px solid $border;
  border-radius: 04px;
  padding: 04px 10px;
  font-size: 13px !important;
  color: $text-muted;
  text-decoration: none;
  transition: border-color $time, color $time;

  &:hover {
    border-color: $accent-light;
    color: $accent-light;
    text-decoration: none;
  }

  &:visited {
    color: $text-muted;
  }
}

.profile-link-btn--accent {
  background-color: $accent;
  border-color: $accent;
  color: $white !important;

  &:hover {
    background-color: $accent-hover;
    border-color: $accent-hover;
    color: $white !important;
  }

  &:visited {
    color: $white !important;
  }
}

.profile-stack {
  display: flex;
  flex-wrap: wrap;
  gap: 06px;

  @include mobile {
    justify-content: center;
  }
}

.stack-badge {
  display: inline-block;
  border: 01px solid $accent;
  color: $accent-light;
  border-radius: 12px;
  padding: 02px 10px;
  font-size: 12px !important;
}
```

- [ ] **Step 4: 빌드 확인**

```bash
cd /Users/haje/sandbox/hajekim.github.io && bundle exec jekyll build 2>&1 | tail -5
```

기대 출력: `done in X seconds` (에러 없음)

- [ ] **Step 5: 커밋**

```bash
cd /Users/haje/sandbox/hajekim.github.io
git add _includes/profile-card.html style.scss _config.yml
git commit -m "feat: add profile card component

- New _includes/profile-card.html with avatar, bio, links, stack badges
- Profile card styles added to style.scss
- bio field added to _config.yml

TEST=bundle exec jekyll build (no errors)

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
```

---

## Task 5: 홈페이지 — 프로필 카드 삽입 + 태그 뱃지

**Files:**
- Modify: `index.html`

- [ ] **Step 1: index.html 전체 교체**

`index.html`을 아래로 교체:

```html
---
layout: default
---

{% include profile-card.html %}

<div class="posts">
  {% for post in paginator.posts %}
  <article class="post">

    <h1 class="{% if forloop.first %}post-title--latest{% endif %}">
      <a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a>
    </h1>

    <div class="date">
      {{ post.date | date: "%Y.%m.%d" }}
      &nbsp;·&nbsp;
      {% include read-time.html content=post.content %}
    </div>

    <div class="entry">
      {% if post.content contains '<!--more-->' %}
      {{ post.excerpt }}
      {% else %}
      {{ post.excerpt | markdownify | strip_html | truncate: 160 }}
      {% endif %}
    </div>

    {% if post.categories.size > 0 %}
    <div class="post-tags">
      {% for category in post.categories %}
      <a class="tag-badge" href="{{ site.baseurl }}/categories/#{{ category | slugize }}">{{ category }}</a>
      {% endfor %}
    </div>
    {% endif %}

  </article>
  {% endfor %}

  {% include pagination.html %}
</div>
```

- [ ] **Step 2: 포스트 목록 다크 스타일을 style.scss에 추가**

`style.scss`에서 `.profile-card` 블록 바로 아래에 추가:

```scss
// Post list
.post-title--latest a {
  color: $accent-light !important;
}

.post-tags {
  margin-bottom: 12px;
  display: flex;
  flex-wrap: wrap;
  gap: 06px;
}

.tag-badge {
  display: inline-block;
  border: 01px solid $accent;
  color: $accent-light !important;
  border-radius: 12px;
  padding: 01px 08px;
  font-size: 12px !important;
  text-decoration: none;

  &:hover {
    background-color: $accent;
    color: $white !important;
    text-decoration: none;
  }

  &:visited {
    color: $accent-light !important;
  }
}
```

- [ ] **Step 3: 빌드 확인**

```bash
cd /Users/haje/sandbox/hajekim.github.io && bundle exec jekyll build 2>&1 | tail -5
```

기대 출력: `done in X seconds` (에러 없음)

- [ ] **Step 4: 커밋**

```bash
cd /Users/haje/sandbox/hajekim.github.io
git add index.html style.scss
git commit -m "feat: add profile card to homepage and tag badges to post list

- Profile card inserted above post list
- First post title highlighted with accent color
- Tag badges with orange/mahogany border

TEST=bundle exec jekyll build (no errors)

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
```

---

## Task 6: 이력서 — 수직 타임라인 레이아웃

**Files:**
- Create: `_layouts/resume.html`
- Create: `_sass/_resume.scss`
- Modify: `style.scss` (import 추가)

- [ ] **Step 1: _layouts/resume.html 생성**

```html
---
layout: default
---

<article class="resume">
  {{ content }}
</article>
```

- [ ] **Step 2: _sass/_resume.scss 생성**

```scss
// Resume page — vertical timeline layout

.resume {
  h1 {
    font-size: 28px !important;
    color: $text-heading;
    margin-bottom: 04px !important;
  }

  // 섹션 구분 (Work Experience, Education 등)
  h1 + p,
  .resume-intro {
    color: $text-muted;
    font-size: 15px !important;
    margin-bottom: 24px !important;
  }

  // 섹션 헤더 (# Work Experience)
  > h1:not(:first-child) {
    margin-top: 40px !important;
    padding-top: 24px;
    border-top: 01px solid $border;
    font-size: 22px !important;
    color: $text-heading;
  }

  // 타임라인 아이템 (## 회사명)
  h2 {
    position: relative;
    padding-left: 24px;
    font-size: 17px !important;
    color: $text-heading;
    margin-top: 24px !important;
    margin-bottom: 02px !important;

    &::before {
      content: "";
      position: absolute;
      left: 0;
      top: 8px;
      width: 10px;
      height: 10px;
      border-radius: 50%;
      background-color: $accent;
    }

    // 타임라인 세로선
    &::after {
      content: "";
      position: absolute;
      left: 04px;
      top: 20px;
      width: 02px;
      bottom: -24px;
      background-color: $border;
    }
  }

  // 직책 + 기간 줄 (br/ 포함된 h2 내 텍스트)
  h2 br {
    display: none;
  }

  // 프로젝트 헤더 (### 프로젝트명)
  h3 {
    padding-left: 24px;
    font-size: 14px !important;
    color: $accent-light;
    font-weight: 600;
    margin-top: 16px !important;
    margin-bottom: 06px !important;
  }

  // 본문 텍스트
  p {
    padding-left: 24px;
    color: $text-muted;
    font-size: 14px !important;
  }

  // 리스트
  ul, ol {
    padding-left: 44px;
    color: $text-muted;
    font-size: 14px !important;
  }

  li {
    margin-bottom: 04px;
  }

  // 링크
  a {
    color: $accent-light;

    &:visited {
      color: $accent-light;
    }
  }

  // Tech Stack 뱃지 그룹
  .tech-stack {
    display: flex;
    flex-wrap: wrap;
    gap: 08px;
    padding-left: 24px;
    margin-top: 16px;
  }

  .tech-badge {
    display: inline-block;
    border: 01px solid $accent;
    color: $accent-light;
    border-radius: 12px;
    padding: 03px 12px;
    font-size: 13px !important;
  }

  // hr 섹션 구분선
  hr {
    border-color: $border;
    margin: 32px 0;
  }
}
```

- [ ] **Step 3: style.scss에 _resume import 추가**

`style.scss` 맨 마지막 `@import "mermaid";` 줄 아래에 추가:

```scss
@import "resume";
```

- [ ] **Step 4: 빌드 확인**

```bash
cd /Users/haje/sandbox/hajekim.github.io && bundle exec jekyll build 2>&1 | tail -5
```

기대 출력: `done in X seconds` (에러 없음)

- [ ] **Step 5: 커밋**

```bash
cd /Users/haje/sandbox/hajekim.github.io
git add _layouts/resume.html _sass/_resume.scss style.scss
git commit -m "feat: add resume layout with vertical timeline

- New _layouts/resume.html
- New _sass/_resume.scss: h2 with orange dot, vertical line, indented content
- Import _resume in style.scss

TEST=bundle exec jekyll build (no errors)

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
```

---

## Task 7: resume.md 업데이트

**Files:**
- Modify: `resume.md`

- [ ] **Step 1: resume.md front matter 변경**

변경 전:
```yaml
---
layout: page
title: resume
permalink: /resume
---
```

변경 후:
```yaml
---
layout: resume
title: Resume
permalink: /resume
---
```

- [ ] **Step 2: Tech Stack 섹션 내용 채우기**

`resume.md`에서 아래 부분을 찾아:

```markdown
# Tech Stack 🥞



---
```

아래로 교체:

```markdown
# Tech Stack 🥞

<div class="tech-stack">
  <span class="tech-badge">GCP</span>
  <span class="tech-badge">BigQuery</span>
  <span class="tech-badge">Looker</span>
  <span class="tech-badge">Vertex AI</span>
  <span class="tech-badge">Kubernetes</span>
  <span class="tech-badge">Airflow</span>
  <span class="tech-badge">PySpark</span>
  <span class="tech-badge">dbt</span>
  <span class="tech-badge">AWS</span>
  <span class="tech-badge">OCI</span>
  <span class="tech-badge">Docker</span>
  <span class="tech-badge">Python</span>
  <span class="tech-badge">SQL</span>
  <span class="tech-badge">Oracle WebLogic</span>
</div>

---
```

- [ ] **Step 3: 빌드 확인**

```bash
cd /Users/haje/sandbox/hajekim.github.io && bundle exec jekyll build 2>&1 | tail -5
```

기대 출력: `done in X seconds` (에러 없음)

- [ ] **Step 4: 커밋**

```bash
cd /Users/haje/sandbox/hajekim.github.io
git add resume.md
git commit -m "feat: switch resume to timeline layout and fill Tech Stack

- layout: page -> layout: resume
- Tech Stack section now has badge-styled items

TEST=bundle exec jekyll build (no errors)

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
```

---

## Task 8: 최종 검증 및 마무리

**Files:**
- 없음 (빌드·브라우저 검증 전용)

- [ ] **Step 1: 로컬 서버 기동**

```bash
cd /Users/haje/sandbox/hajekim.github.io && bundle exec jekyll serve --livereload
```

브라우저에서 `http://localhost:4000` 열기

- [ ] **Step 2: 홈페이지 체크리스트**

- [ ] 다크 배경(`#0d1117`) 적용 확인
- [ ] 헤더 배경 `#161b22` 확인
- [ ] 프로필 카드 표시 확인 (아바타, bio, 링크 버튼, 스택 뱃지)
- [ ] 첫 번째 포스트 제목이 오렌지(`#fb923c`) 색상인지 확인
- [ ] 포스트 태그 뱃지 표시 확인

- [ ] **Step 3: 이력서 페이지 체크리스트**

브라우저에서 `http://localhost:4000/resume` 열기

- [ ] 타임라인 오렌지 점(●) 표시 확인
- [ ] 타임라인 세로선 표시 확인
- [ ] 직책 오렌지 색상 확인 (h3)
- [ ] Tech Stack 뱃지 표시 확인
- [ ] 섹션 구분(hr) 확인

- [ ] **Step 4: 포스트 페이지 체크리스트**

포스트 하나 클릭 후 확인:

- [ ] 코드 블록 다크 배경 + GitHub Dark 신택스 확인
- [ ] 본문 텍스트 `#c9d1d9` 색상 확인

- [ ] **Step 5: 모바일 반응형 체크**

브라우저 개발자도구에서 모바일 뷰(375px) 전환 후:

- [ ] 프로필 카드 세로 정렬 확인
- [ ] 네비게이션 표시 확인

- [ ] **Step 6: 마무리 커밋 (변경 사항 있을 경우만)**

```bash
cd /Users/haje/sandbox/hajekim.github.io
git add -p   # 변경된 파일만 선택적으로 추가
git commit -m "fix: post-verification adjustments

TEST=visual inspection on localhost:4000

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
```
