---
title: "Gemini CLI Extension으로 AI 에이전트 디자인 패턴 28개를 한 번에 — Agentic Design Patterns"
date: 2026-03-28 00:00:00 +0900
description: "28개의 AI 에이전트 디자인 패턴을 Gemini CLI Extension 하나로 설치하고, 슬래시 커맨드와 서브에이전트로 즉시 활용하는 방법"
categories: AI
tags: [Gemini, "Gemini CLI", Agent, "AI Agent", GCP, ADK]
---

AI 에이전트를 설계하다 보면 반복적으로 등장하는 구조적 문제들이 있습니다. "여러 단계를 순서대로 실행해야 하는데", "에러가 나면 어떻게 복구하지?", "외부 도구를 연결하는 방법은?", "멀티에이전트는 어떻게 구성하지?" 등등.

이런 문제들에 대한 검증된 해법이 **에이전트 디자인 패턴(Agentic Design Patterns)** 입니다. Antonio Gulli의 동명 저서(424페이지, 21챕터)를 기반으로 28개의 패턴을 Gemini CLI Extension으로 패키징했습니다.

---

## 무엇을 제공하는가

**28개의 에이전트 스킬**을 단일 Extension으로 설치하면, Gemini CLI가 사용자의 의도를 파악해 자동으로 관련 패턴 가이드를 불러옵니다. 코드 스켈레톤 생성, 패턴 추천, 코드 리뷰까지 하나의 도구에서 처리합니다.

| 카테고리 | 패턴 수 | 포함 패턴 |
|---------|--------|---------|
| **Core** | 7 | Prompt Chaining, Routing, Parallelization, Reflection, Tool Use, Planning, Multi-Agent Collaboration |
| **State Management** | 4 | Memory Management, Learning & Adaptation, MCP, Goal Setting |
| **Reliability** | 3 | Exception Handling, Human-in-the-Loop, RAG |
| **Advanced** | 7 | A2A, Resource-Aware, Reasoning, Guardrails, Evaluation, Prioritization, Exploration |
| **Appendix** | 7 | Prompt Engineering, GUI Agents, Agentic Frameworks, AgentSpace, AI CLI, Coding Agents, Reasoning Engines |

---

## 설치

```bash
gemini extensions install https://github.com/hajekim/agentic-design-patterns-extension
```

설치 후 Gemini CLI를 재시작하면 28개의 스킬이 자동으로 활성화됩니다. 추가 설정은 필요하지 않습니다.

설치된 Extension 확인:

```bash
gemini extensions list
```

---

## 스킬 자동 활성화 — 어떻게 동작하는가

Gemini CLI는 세션 시작 시 모든 스킬의 `name`과 `description`을 색인합니다. 사용자의 요청이 스킬 설명과 의미적으로 매칭되면, 모델이 자율적으로 해당 스킬의 전체 내용을 로드합니다. 별도의 명령어 없이 **자연어 요청만으로 패턴 가이드가 활성화**됩니다.

**한국어 요청 예시:**

| 요청 | 활성화되는 패턴 |
|------|--------------|
| "프롬프트 체이닝으로 파이프라인 만들어줘" | Prompt Chaining |
| "MCP 구성을 해줘" | MCP |
| "메모리뱅크 만들어줘" | Memory Management |
| "멀티에이전트 시스템 설계해줘" | Multi-Agent Collaboration |
| "에러 처리 어떻게 해?" | Exception Handling |
| "추론 모델 언제 써야 해?" | Reasoning Engines |
| "RAG 파이프라인 구축해줘" | RAG |

4개 언어(영어·한국어·일본어·중국어) 총 **1,376개의 트리거 문구**가 등록되어 있습니다.

---

## 슬래시 커맨드

### `/pattern-summary` — 패턴 목록 탐색

28개 패턴 전체를 카테고리별로 한눈에 봅니다.

```bash
# 전체 28개 패턴 목록 (카테고리별 테이블)
/pattern-summary

# 특정 카테고리만 보기
/pattern-summary core
/pattern-summary reliability
/pattern-summary advanced

# 특정 패턴 상세 보기 (언제 쓰는지 + 관련 패턴 추천)
/pattern-summary planning
/pattern-summary rag
```

### `/gen-skeleton` — 코드 스켈레톤 생성

패턴 이름을 인자로 넘기면 즉시 실행 가능한 Python 코드 스켈레톤을 생성합니다.

```bash
/gen-skeleton planning
/gen-skeleton rag
/gen-skeleton multi-agent-collaboration
/gen-skeleton exception-handling
```

생성되는 코드는 다음 규칙을 강제합니다:
- **Gemini SDK**: `google-genai` (`import google.genai as genai`)
- **ADK**: `LlmAgent` + `Runner` + `InMemorySessionService`
- **기본 모델**: `gemini-2.5-flash`

스켈레톤은 Imports → Configuration → Core 패턴 구현 → Main 실행 블록 순서로 구성됩니다.

---

## 서브에이전트

### `@architect` — 패턴 조합 추천

어떤 패턴을 써야 할지 모를 때, 문제를 설명하면 최적의 패턴 조합을 추천합니다.

```bash
@architect "피드백을 학습하는 고객 지원 봇을 만들고 싶어"
@architect "실시간 데이터를 분석해서 이상 탐지를 하는 에이전트 설계해줘"
@architect "여러 전문 에이전트가 협력해서 리포트를 작성하는 시스템 필요해"
```

출력 형식:
```
## Pattern Recommendation

Primary Pattern: `planning`
Supporting Patterns: `reflection`, `memory-management`

### Why This Combination
...

### Implementation Sequence
1. Start with `planning` — ...
2. Add `reflection` — ...

### Watch Out For
- ...

### Next Step
Run `/gen-skeleton planning` to generate the code skeleton.
```

### `@reviewer` — 코드 패턴 컴플라이언스 검토

작성한 에이전트 코드를 붙여넣으면 패턴 구현 적합성과 SDK 사용 오류를 검토합니다.

```bash
@reviewer <코드 붙여넣기>
```

검토 항목:

| 항목 | 올바른 사용 | 잘못된 사용 |
|------|-----------|-----------|
| Gemini SDK import | `import google.genai as genai` | `import google.generativeai` |
| ADK Agent 클래스 | `LlmAgent` | `Agent` |
| ADK Runner | `Runner` + `InMemorySessionService` | `InMemoryRunner` |
| LangChain vector store | `langchain-chroma` | `langchain_community.vectorstores.Chroma` |
| LangChain conversation | `RunnableWithMessageHistory` | `ConversationChain` |

출력 형식:
```
## Pattern Review: `rag`

### SDK Compliance
✅ Gemini SDK import correct
❌ ADK Runner: `InMemoryRunner` → use `Runner` + `InMemorySessionService`

### Pattern Implementation
✅ Retrieval step present before generation
❌ Source attribution missing

### Overall: NEEDS FIXES
```

---

## MCP 서버 (스킬 검색)

Extension이 설치되면 `mcp_server.py`가 자동으로 MCP 서버로 등록됩니다. 모델이 직접 패턴을 검색하고 스킬 내용을 가져올 수 있으며, 대화 중 자연스럽게 호출됩니다.

| 도구 | 설명 |
|------|------|
| `list_patterns([category])` | 28개 패턴 목록 반환. 카테고리 필터 가능 (`core` / `state` / `reliability` / `advanced` / `appendix`) |
| `get_skill(pattern_name)` | 패턴 이름으로 `SKILL.md` 전체 내용 조회. 오타 시 유사 패턴 제안 |
| `search_skills(query)` | 키워드로 전체 28개 스킬을 검색, 매칭된 라인과 패턴 반환 |

### `list_patterns` — 패턴 목록 조회

카테고리 없이 호출하면 28개 전체 목록을, 카테고리를 지정하면 해당 카테고리만 반환합니다.

```
# 전체 28개 목록 (카테고리별 그룹)
list_patterns()

# 특정 카테고리만
list_patterns("core")       → Core 7개 패턴
list_patterns("state")      → State Management 4개 패턴
list_patterns("reliability") → Reliability 3개 패턴
list_patterns("advanced")   → Advanced 7개 패턴
list_patterns("appendix")   → Appendix 7개 패턴
```

잘못된 카테고리를 입력하면 유효한 카테고리 목록을 안내합니다:
```
Unknown category 'xxx'. Valid categories: `core`, `state`, `reliability`, `advanced`, `appendix`.
```

### `get_skill` — 스킬 전체 내용 조회

패턴 이름으로 해당 `SKILL.md` 전체를 반환합니다. 패턴의 정의, 트리거 문구, 구현 예제 코드를 모두 포함합니다.

```
get_skill("planning")
get_skill("rag")
get_skill("mcp-setup")
get_skill("multi-agent-collaboration")
```

오타나 불완전한 이름 입력 시 유사 패턴을 제안합니다:
```
# 입력: get_skill("plan")
Pattern 'plan' not found. Did you mean: `planning`?

# 입력: get_skill("memory")
Pattern 'memory' not found. Did you mean: `memory-management`?
```

패턴 이름을 전혀 모를 경우에는 `list_patterns()`로 전체 목록을 먼저 확인합니다.

### `search_skills` — 키워드 검색

키워드가 포함된 스킬을 28개 전체에서 검색합니다. 매칭된 스킬 이름, 매칭 횟수, 첫 번째 매칭 줄 미리보기를 반환합니다.

```
search_skills("LangGraph")
search_skills("Thinking Budget")
search_skills("fallback")
search_skills("human approval")
```

출력 예시:
```
Found 'Thinking Budget' in 3 pattern(s):

`reasoning` (5 matches)
  → Thinking Budget controls how deeply the model reasons before responding...

`resource-aware` (3 matches)
  → Set Thinking Budget high when accuracy matters more than latency...

`appendix-reasoning-engines` (8 matches)
  → Thinking Budget is configured via thinking_config, not a separate model...
```

특정 기술(예: `LangGraph`, `Chroma`, `Redis`)이나 개념(예: `retry`, `timeout`, `fallback`)으로 검색해 관련 패턴을 빠르게 찾을 때 유용합니다.

---

## 권장 워크플로우

```
1. @architect로 패턴 추천받기
       ↓
2. /pattern-summary <패턴명>으로 상세 확인
       ↓
3. /gen-skeleton <패턴명>으로 코드 스켈레톤 생성
       ↓
4. 코드 작성
       ↓
5. @reviewer로 패턴 컴플라이언스 검토
```

---

## 모델 선택 가이드

| 작업 유형 | 권장 모델 | Thinking Budget |
|---------|---------|----------------|
| 단순 파이프라인 (prompt-chaining, routing) | `gemini-2.5-flash-lite` | 미지원 |
| 중간 복잡도 (tool-use, RAG, parallelization) | `gemini-2.5-flash` | Dynamic (기본값) |
| 복잡한 추론 (planning, reasoning, evaluation) | `gemini-2.5-flash` 또는 `gemini-2.5-pro` | 높게 설정 |
| 대규모 조율 (multi-agent, a2a) | `gemini-2.5-pro` | 높게 설정 |

Thinking Budget은 Flash와 Pro 모델에서 조절 가능한 추론 깊이 파라미터입니다. 대부분의 작업에서는 기본값(Dynamic)을 유지하면 모델이 자동으로 결정합니다.

---

## 플랫폼 호환성

| 플랫폼 | 설치 방법 | 활성화 방식 |
|--------|---------|-----------|
| **Gemini CLI** | `gemini extensions install <url>` | 시맨틱 — 모델이 설명 기반으로 자율 판단 |
| **Antigravity** | `skills/`를 `.agents/skills/`에 복사 | 키워드 패턴 매칭 |
| **Claude Code** | `skills/`를 `.claude/skills/`에 심볼릭 링크 | 시맨틱 판단 + 슬래시 커맨드 |

Antigravity와 Claude Code에는 [Skills-only 버전](https://github.com/hajekim/agentic-design-patterns-skills)을 사용하세요.

---

## Extension 관리

```bash
# 최신 버전으로 업데이트
gemini extensions update agentic-design-patterns

# 임시 비활성화 (제거 없이)
gemini extensions disable agentic-design-patterns

# 제거
gemini extensions uninstall agentic-design-patterns
```

---

## Extension 구조

```
agentic-design-patterns/
├── gemini-extension.json      ← Extension 매니페스트
├── GEMINI.md                  ← 전역 컨텍스트 (28개 패턴 퀵레퍼런스, 모델 가이드)
├── mcp_server.py              ← 스킬 검색 MCP 서버
├── commands/
│   ├── gen-skeleton.toml      ← /gen-skeleton <패턴명>
│   └── pattern-summary.toml  ← /pattern-summary [필터]
├── agents/
│   ├── architect.md           ← @architect 서브에이전트
│   └── reviewer.md            ← @reviewer 서브에이전트
└── skills/                    ← 28개 스킬 정의
    ├── planning/SKILL.md
    ├── rag/SKILL.md
    └── ...
```

각 스킬은 **DEFINE → PLAN → ACTION** 워크플로우를 따르며, Google ADK, LangChain, LangGraph 구현 예제를 포함합니다.

---

## 설치 링크

- **Extension (Gemini CLI용):** [hajekim/agentic-design-patterns-extension](https://github.com/hajekim/agentic-design-patterns-extension)
- **Skills-only (Antigravity / Claude Code용):** [hajekim/agentic-design-patterns-skills](https://github.com/hajekim/agentic-design-patterns-skills)

현재 버전: **v2.2.3**
