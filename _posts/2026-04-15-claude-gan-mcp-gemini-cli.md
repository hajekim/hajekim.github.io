---
title: "Gemini CLI + Claude GAN: MCP로 AI 두 개를 협력시키는 코드 생성 루프"
date: 2026-04-15 00:00:00 +0900
description: "Gemini CLI가 Evaluator로 직접 동작하고 Claude 4.6 Sonnet(Vertex AI)을 MCP 도구로 호출하는 GAN 기반 코드 생성 시스템 설치와 사용법"
categories: ai
tags: [Gemini, "Gemini CLI", Claude, MCP, "Vertex AI", GCP, "코드 생성"]
---

AI 모델 하나에 "코드 짜줘" 하면 한 번에 완성본이 나올 것 같지만, 실제로는 피드백을 주고받으며 반복해야 좋은 결과가 나옵니다. 이 반복을 **사람이 하지 않고 두 번째 AI가 하게** 만들면 어떨까요?

**claude-gan**은 Gemini CLI가 Evaluator(평가자)가 되고, Claude 4.6 Sonnet(Vertex AI)을 Generator(생성자)로 호출하는 GAN 기반 개발 시스템입니다. Gemini가 직접 평가 기준을 작성하고, Claude가 그 기준에 맞는 코드를 생성하고, Gemini가 다시 채점하는 루프가 Gemini CLI 세션 안에서 자동으로 돌아갑니다.

---

## 구조

```
Gemini CLI (Evaluator — 네이티브 추론)
  │
  ├─ 1. Sprint Contract(JSON DoD)를 자체 추론으로 직접 작성
  ├─ 2. claude_generate MCP 도구 호출 ──▶ Claude 4.6 Sonnet (Vertex AI)
  ├─ 3. 반환된 코드를 계약 기준으로 직접 평가
  └─ 4. Grade A 달성까지 구체적 피드백으로 반복
```

Gemini CLI에 내장된 Gemini 모델 자체가 Evaluator입니다. Claude는 서브프로세스가 아닌 **MCP 도구**로 호출됩니다. 코드를 생성하는 역할만 담당하고, 판단과 피드백은 전부 Gemini가 합니다.

---

## 사전 준비

- Google Cloud Project (Vertex AI API 활성화, `global` 리전에서 Claude 4.6 Sonnet 접근 권한)
- `gcloud auth application-default login` 완료
- Gemini CLI 설치 및 인증 완료
- Python 3.10+

---

## 설치

### 1. 리포지토리 클론

```bash
git clone https://github.com/hajekim/gan-cli.git
cd gan-cli
```

### 2. Python 의존성 설치

```bash
pip install -r requirements.txt
```

`requirements.txt`에는 Anthropic Vertex AI SDK, FastMCP, python-dotenv, pytest가 포함되어 있습니다.

### 3. MCP 서버 전역 등록

`~/.gemini/settings.json`의 `mcpServers`에 아래를 추가합니다:

```json
{
  "mcpServers": {
    "claude-generator": {
      "command": "/path/to/python3",
      "args": ["/절대경로/gan-cli/src/mcp_server.py"],
      "env": {
        "GOOGLE_CLOUD_PROJECT": "your-gcp-project-id",
        "GOOGLE_CLOUD_LOCATION": "global",
        "PYTHONPATH": "/절대경로/gan-cli"
      }
    }
  }
}
```

`python3` 경로는 `which python3`로 확인합니다. `/절대경로/gan-cli`는 클론한 실제 경로로 교체합니다.

### 4. 기존 GEMINI.md에 GAN 지시문 추가

`~/.gemini/GEMINI.md` 파일을 **덮어쓰지 말고** 아래 두 항목을 기존 내용에 추가합니다.

**MCP 서버 목록에 추가:**

```
- **claude-generator**: GAN 루프 코드 생성 전용. `claude_generate(task, contract, feedback)`으로
  Claude 4.6 Sonnet (Vertex AI)에게 구현을 위임한다.
  `save_artifact(content, filename)`, `save_progress(sprint_id, status, grade)`로 결과를 저장한다.
```

**실행 워크플로우(ACT 단계 또는 GAN 루프 섹션)에 추가:**

```
코드 생성이 필요한 경우, 직접 작성하지 않고 claude-generator MCP에 위임한다:
1. 작업 요구사항을 분석하여 Sprint Contract(JSON DoD)를 직접 작성한다.
2. claude_generate(task, contract, feedback="")를 호출하여 Claude의 구현을 받는다.
3. 반환된 코드를 Skeptical Judge로서 엄격하게 평가한다 (Gemini = Evaluator).
4. Grade B/C이면 구체적인 피드백으로 claude_generate를 재호출한다. 최대 3회.
5. Grade A 달성 시 save_artifact와 save_progress로 결과를 저장한다.
코드 생성이 아닌 작업(기존 코드 수정, 설정 변경 등)은 직접 처리한다.
```

Gemini CLI를 재시작하고 `mcp list`로 `claude-generator`가 Connected 상태인지 확인합니다.

---

## 실제 사용 예시

Gemini CLI를 재시작한 후 자연어로 요청합니다:

```
Python으로 가위바위보 게임을 만들어줘. 점수 기록 기능도 포함해줘.
```

**계획 단계** — Gemini가 Sprint Contract를 작성하고 승인을 요청합니다:

```
✦ 가위바위보 게임과 점수 기록 기능을 구현하기 위한 개발 계획 PLAN.md를 작성했습니다.
  GAN 루프(Gemini가 평가자, Claude가 생성자)를 사용하여 PEP 8 표준을 준수하고
  기능적으로 완벽한 코드를 작성할 예정입니다.
  해당 계획에 동의하신다면, 상세 작업을 스프린트로 나누는 DEFINE.md를 작성하고
  구현을 시작하겠습니다. 계획을 진행할까요?

  PLAN.md 내용 요약:
   - 목표: 사용자 입력을 받아 컴퓨터와 대결하고 승/무/패 점수를 기록하는 Python 가위바위보 게임 개발.
   - 전략: Claude 4.6 Sonnet(claude_generate)을 통해 코드를 생성하고, Gemini가 이를 검증하는 GAN 루프 적용.
   - 검증 계약: 정확한 승패 로직, 점수 기록/표시, 예외 처리 및 PEP 8 준수 여부 확인.
   - 등급 기준: 완벽한 기능과 스타일(Grade A)을 목표로 함.
```

**구현 단계** — 승인 후 GAN 루프가 실행되어 결과물을 전달합니다:

```
✦ GAN 루프(Claude Generator + Gemini Evaluator)를 통해 고품질의 가위바위보 게임 구현을 완료했습니다.

  구현 결과: rps_game.py
   - 핵심 기능:
       - 사용자 입력(rock, paper, scissors)과 컴퓨터의 무작위 선택 대결.
       - 매 라운드 종료 후 현재 점수(승, 패, 무) 실시간 표시.
       - quit 입력 시 게임이 종료되며 전체 세션의 최종 성적 요약 출력.
       - 잘못된 입력에 대한 예외 처리 및 재입력 요청.

  GAN 루프 검증 결과
   - Generator (Claude 4.6 Sonnet): 명확한 함수 구조와 가독성 높은 코드 생성.
   - Evaluator (Gemini): PEP 8 준수 여부, 점수 기록 로직의 정확성, 예외 처리 완성도를
     검토하여 Grade A로 최종 승인.
```

---

## MCP 도구 목록

전역 등록 후 모든 Gemini CLI 세션에서 사용 가능합니다:

| 도구 | 설명 |
|------|------|
| `claude_generate(task, contract, feedback)` | Claude 4.6 Sonnet (Vertex AI)으로 코드 생성/개선 |
| `save_artifact(content, filename)` | `artifacts/` 디렉토리에 결과물 저장 |
| `load_progress()` | `state/progress.json`에서 현재 스프린트 상태 로드 |
| `save_progress(sprint_id, status, grade)` | 스프린트 상태 저장 |

---

## 프로젝트 구조

```
gan-cli/
├── src/
│   ├── mcp_server.py                # FastMCP 서버 (stdio transport)
│   └── tools/
│       └── claude_tool.py           # Claude Vertex AI 호출 핵심 로직
├── tests/
│   └── test_claude_tool.py          # 단위 테스트
├── state/
│   └── progress.json                # 스프린트 상태 (런타임, gitignored)
├── artifacts/                       # 생성 코드 출력 (런타임, gitignored)
├── GEMINI.md                        # Gemini CLI용 Evaluator 지시문
├── gemini-extension.json.template   # Extension 매니페스트 템플릿
└── requirements.txt
```

---

## GitHub

[hajekim/gan-cli](https://github.com/hajekim/gan-cli)
