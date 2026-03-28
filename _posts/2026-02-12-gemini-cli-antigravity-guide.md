---
title: "Gemini CLI & Antigravity IDE 설정 완전 가이드"
date: 2026-02-12 00:00:00 +0900
description: "Gemini CLI와 Google Antigravity IDE의 설정 체계, 인증, MCP, Agent Skills, 세션 관리 등 상세 구조 분석 및 통합 활용 가이드"
categories: ai
tags: [Gemini, AI, CLI, IDE]
---

> **작성일:** 2026-02-12
> **대상:** Gemini CLI (v0.3.x+), Google Antigravity IDE
> **목적:** 양 플랫폼의 설정 체계·인증·MCP·Agent Skills·세션 관리 등의 상세 구조 분석 및 통합 활용 가이드

---

## 목차

1. [개요](#1-개요)
2. [GEMINI.md - 컨텍스트 및 메모리 파일](#2-geminimd---컨텍스트-및-메모리-파일)
3. [settings.json - 동작 설정 및 에이전트 모드](#3-settingsjson---동작-설정-및-에이전트-모드)
4. [Auth 설정 - 인증 체계](#4-auth-설정---인증-체계)
5. [.env 설정 - 환경변수 관리](#5-env-설정---환경변수-관리)
6. [Rules - 항상 적용되는 가이드라인](#6-rules---항상-적용되는-가이드라인)
7. [Workflows - 사용자 트리거 매크로](#7-workflows---사용자-트리거-매크로)
8. [Agent Skills - 핵심 상호호환 계층](#8-agent-skills---핵심-상호호환-계층)
9. [MCP - 외부 통합](#9-mcp---외부-통합)
10. [Agent 아키텍처 및 특수 에이전트](#10-agent-아키텍처-및-특수-에이전트)
11. [세션 및 작업 관리](#11-세션-및-작업-관리)
12. [상호호환성 종합 매핑](#12-상호호환성-종합-매핑)
13. [통합 실전 구성 예시 및 팁](#13-통합-실전-구성-예시-및-팁)
14. [참고 자료](#14-참고-자료)

---

## 1. 개요

Gemini CLI는 터미널 기반 AI 에이전트이고, Antigravity는 VSCode 포크 기반의 에이전트 중심 IDE입니다. 두 도구는 동일한 **Gemini 에이전트 코어** 엔진을 공유하며, 핵심적인 도구 시스템과 설정 로직은 호환되지만 사용자의 작업 환경에 따라 제공되는 인터페이스와 관리 방식에서 차이가 있습니다. 핵심은 **Agent Skills라는 오픈 표준([agentskills.io](agentskills.io))을 통해 양 플랫폼 간 상호호환이 가능하다는 점입니다.**

| 항목 | Gemini CLI | Antigravity |
|------|-----------|-------------|
| **형태** | 터미널 CLI 에이전트입니다. | VSCode 포크 IDE입니다. |
| **인터페이스** | 커맨드라인 (터미널 상호작용 최적화) | GUI (에디터 + Agent Panel) |
| **에이전트** | 단일 순차 에이전트 (셸 명령, 파일 직접 제어) | 멀티 에이전트 병렬 실행 (시각적 워크플로우) |
| **모델** | Gemini 2.5 Pro/Flash (Free), API키로 상위 모델 | Gemini 3 모델 (역할별 특화) |
| **설정 방식** | JSON 파일 + .env + CLI 인수 | IDE UI + JSON 파일 |

---

## 2. GEMINI.md - 컨텍스트 및 메모리 파일

GEMINI.md는 양 플랫폼에서 **항상 로드되는 지속적 컨텍스트 파일**로, 프로젝트별 지시사항·코딩 스타일·아키텍처 규칙을 정의합니다. 에이전트는 대화 시작 시 여러 위치의 파일을 찾아 하나의 시스템 지침으로 결합합니다.

### 2.1 계층 구조 비교

| 계층 | Gemini CLI | Antigravity |
|------|-----------|-------------|
| **전역** | `~/.gemini/GEMINI.md` | `~/.gemini/GEMINI.md` |
| **프로젝트** | 프로젝트 루트 + 상위 디렉토리 (`.git`까지) | 전역 규칙으로 적용됩니다. |
| **서브디렉토리** | 하위 디렉토리 GEMINI.md 자동 스캔 (`.gitignore` 존중) | 지원하지 않습니다. |
| **커스텀 이름** | `context.fileName`으로 변경 가능합니다. (예: `["AGENTS.md", "CONTEXT.md", "GEMINI.md"]`) | GEMINI.md로 고정되어 있습니다. |

> **💡 프로젝트 내 GEMINI.md 배치 팁**
> *   **표준 위치 (권장)**: 프로젝트 루트 디렉토리에 배치합니다. 에이전트가 별도 설정 없이 즉시 파일을 인식하여 적용합니다.
> *   **통합 관리 위치**: 설정을 한곳에 모으기 위해 `.gemini/GEMINI.md` 경로를 사용할 수도 있습니다. 이 경우 `settings.json` 내의 `context.fileName`에 해당 경로를 명시해야 에이전트가 올바르게 로드할 수 있습니다. (13.1 통합 디렉토리 구조 참조)

*   **계층적 로드**: 전역 → 프로젝트 루트 → 현재 디렉토리 순으로 로드되며, 하위 폴더의 지침이 상위의 지침을 구체화하거나 덮어쓸 수 있습니다.
*   **모듈화**: `@filename.md` 구문을 사용하여 공통 컨벤션이나 아키텍처 문서를 여러 프로젝트에서 가져와 사용할 수 있습니다.

### 2.2 Gemini CLI의 GEMINI.md 관리 명령어

| 명령어 | 설명 |
|--------|------|
| `/memory show` | 현재 로드된 전체 컨텍스트를 표시합니다. |
| `/memory refresh` | 모든 GEMINI.md를 재스캔하고 즉시 반영합니다. |
| `/memory add <text>` | 전역 GEMINI.md에 내용을 추가합니다. |
| `/init` | 프로젝트용 GEMINI.md를 초기에 생성합니다. |

### 2.3 실전 GEMINI.md 작성 샘플
범용적인 개발 환경에서 사용할 수 있는 표준적인 `GEMINI.md` 예시입니다.

```markdown
# 프로젝트 컨텍스트 (GEMINI.md)

## 0. 핵심 사고 지침 (Core Thinking Rule) - [CRITICAL]
*   **Mandatory Thinking Process**: For EVERY user request, without exception, you MUST initiate your process by calling the `sequentialthinking` tool. You are prohibited from taking any other actions (reading files, executing commands, etc.) until you have systematically analyzed the request and planned your approach within a `sequentialthinking` block.

## 1. 표준 워크플로우 (Plan-Define-Act)
모든 복합적인 작업 시 다음 절차를 엄격히 준수합니다.
1. **PLAN (계획)**: 요청을 분석하여 단계별 실행 계획을 수립하고 사용자 승인을 받습니다.
2. **DEFINE (정의)**: 승인된 계획을 상세 TODO 리스트로 분해하여 DEFINE.md에 기록합니다.
3. **ACT (실행)**: 지침에 따라 코드를 작성하고, 테스트 및 린트를 통해 최종 검증합니다.

## 2. 코딩 원칙 및 스타일 가이드
*   **Consistency**: 기존 코드의 네이밍 컨벤션과 아키텍처 패턴을 분석하여 일관성을 유지합니다.
*   **Naming**: 변수/함수는 명확한 의미를 담아야 하며, Boolean 타입은 `is`, `has` 등의 접두사를 사용합니다.
*   **Documentation**: 복잡한 로직은 '무엇'보다 '왜' 작성했는지를 설명하는 주석(Why-comment)을 추가합니다.

## 3. 검증 및 보안 (Verification & Security)
*   **Test-First**: 코드 변경 후에는 반드시 `npm test` 또는 `pytest`를 실행하여 안정성을 확인합니다.
*   **Security**: API Key나 Token 등 민감 정보는 절대 코드나 로그에 노출하지 않으며 `.env`로 관리합니다.
```

### 2.4 [Tip] 작성 언어 선택 가이드 (Korean vs English)
`GEMINI.md` 작성 시 언어 선택은 모델의 성능과 유지보수 효율성에 영향을 미칩니다.

*   **영문 작성의 장점**: 기술적 정밀도가 높고 모델이 지시 사항을 더 강력하게 수용하며, 토큰 효율성이 좋습니다. (핵심 제약 조건 작성에 권장합니다.)
*   **국문 작성의 장점**: 팀 내 소통이 원활하고 비즈니스 로직 및 도메인 지식을 상세히 설명하기에 유리합니다. (맥락 공유에 권장합니다.)
*   **권장하는 하이브리드 방식**:
    - 기술적 용어, 핵심 명령어, 제약 조건은 **영문**으로 작성합니다.
    - 이에 대한 구체적인 배경 설명과 맥락은 **국문**으로 작성합니다.

---

## 3. settings.json - 동작 설정 및 에이전트 모드

settings.json은 **Gemini CLI 전용** 설정 체계입니다. Antigravity는 프로젝트 루트의 **`.agent/`** 디렉토리를 초기 설정 공간으로 인식하며 IDE 내장 설정 UI를 주로 사용하지만, 핵심 로직은 이 설정을 기반으로 동작합니다.

### 3.1 설정 파일 우선순위 (높을수록 우선)

```
1. 시스템 기본값    /etc/gemini-cli/system-defaults.json
2. 사용자 설정      ~/.gemini/settings.json
3. 프로젝트 설정    .gemini/settings.json
4. 시스템 오버라이드  /etc/gemini-cli/settings.json
5. 환경 변수        .env 파일 및 OS 환경변수
6. CLI 인수         최우선 (예: --model, -y)
```

> **Tip:** JSON 에디터에서 스키마 자동완성을 사용하려면 아래 URL을 참조하십시오:
> `https://raw.githubusercontent.com/google-gemini/gemini-cli/main/schemas/settings.schema.json`

### 3.2 전체 설정 카테고리 레퍼런스

#### `general` — 일반 설정

```json
{
  "general": {
    "previewFeatures": false,
    "preferredEditor": "code",
    "vimMode": false,
    "disableAutoUpdate": false,
    "disableUpdateNag": false,
    "checkpointing": { "enabled": false },
    "enablePromptCompletion": false,
    "retryFetchErrors": false,
    "sessionRetention": {
      "enabled": false,
      "maxAge": "30d",
      "maxCount": 100,
      "minRetention": "1d"
    }
  }
}
```

#### `model` — 모델 설정

```json
{
  "model": {
    "name": "gemini-2.5-pro",
    "maxSessionTurns": -1,
    "compressionThreshold": 0.5,
    "skipNextSpeakerCheck": true,
    "summarizeToolOutput": {
      "run_shell_command": { "tokenBudget": 2000 }
    }
  }
}
```

#### `modelConfigs` — 모델 프리셋 및 별칭

20개 이상의 사전 정의 모델 프리셋이 포함되어 있으며, `customAliases`로 확장 가능합니다.

```json
{
  "modelConfigs": {
    "customAliases": {
      "my-fast": {
        "extends": "chat-base-3",
        "modelConfig": {
          "model": "gemini-3-flash-preview",
          "generateContentConfig": {
            "thinkingConfig": { "thinkingLevel": "LOW" }
          }
        }
      }
    }
  }
}
```

주요 내장 별칭: `gemini-3-pro-preview`, `gemini-3-flash-preview`, `gemini-2.5-pro`, `gemini-2.5-flash`, `gemini-2.5-flash-lite`, `classifier`, `summarizer-default`, `web-search`, `web-fetch`, `loop-detection`, `chat-compression-*` 등입니다.

#### `context` — 컨텍스트 파일 설정

```json
{
  "context": {
    "fileName": ["GEMINI.md"],
    "importFormat": null,
    "discoveryMaxDirs": 200,
    "includeDirectories": [],
    "loadMemoryFromIncludeDirectories": false,
    "fileFiltering": {
      "respectGitIgnore": true,
      "respectGeminiIgnore": true,
      "enableRecursiveFileSearch": true,
      "disableFuzzySearch": false
    }
  }
}
```

#### `tools` — 도구 설정

```json
{
  "tools": {
    "sandbox": "docker",
    "shell": {
      "enableInteractiveShell": true,
      "pager": "cat",
      "showColor": false,
      "inactivityTimeout": 300,
      "enableShellOutputEfficiency": true
    },
    "autoAccept": false,
    "core": null,
    "allowed": ["run_shell_command(git)", "run_shell_command(npm test)"],
    "exclude": [],
    "useRipgrep": true,
    "enableToolOutputTruncation": true,
    "truncateToolOutputThreshold": 4000000,
    "truncateToolOutputLines": 1000,
    "enableHooks": true
  }
}
```

#### `security` — 보안 및 인증 설정

```json
{
  "security": {
    "disableYoloMode": false,
    "enablePermanentToolApproval": false,
    "blockGitExtensions": false,
    "folderTrust": { "enabled": false },
    "environmentVariableRedaction": {
      "enabled": false,
      "allowed": [],
      "blocked": []
    },
    "auth": {
      "selectedType": null,
      "enforcedType": null,
      "useExternal": null
    }
  }
}
```

> `security.auth.selectedType`: 현재 선택된 인증 유형입니다 (OAuth, API 키 등).
> `security.auth.enforcedType`: 강제할 인증 유형입니다 (불일치 시 재인증 요구).
> `security.auth.useExternal`: 외부 인증 플로우 사용 여부입니다.

#### `mcp` — MCP 전역 설정

```json
{
  "mcp": {
    "serverCommand": null,
    "allowed": ["my-trusted-server"],
    "excluded": ["experimental-server"]
  }
}
```

#### `mcpServers` — MCP 서버별 설정

```json
{
  "mcpServers": {
    "serverName": {
      "command": "path/to/server",
      "args": ["--arg1", "value1"],
      "env": { "API_KEY": "$MY_API_TOKEN" },
      "cwd": "./server-directory",
      "url": null,
      "httpUrl": null,
      "headers": {},
      "timeout": 30000,
      "trust": false,
      "description": "서버 설명",
      "includeTools": [],
      "excludeTools": []
    }
  }
}
```

#### `experimental` — 실험적 기능

```json
{
  "experimental": {
    "skills": true,
    "enableAgents": false,
    "jitContext": false,
    "extensionManagement": true,
    "extensionReloading": false,
    "codebaseInvestigatorSettings": {
      "enabled": true,
      "maxNumTurns": 10,
      "maxTimeMinutes": 3,
      "thinkingBudget": 8192,
      "model": "auto"
    },
    "cliHelpAgentSettings": { "enabled": true }
  }
}
```

#### `skills` — Agent Skills 관리

```json
{
  "skills": {
    "disabled": ["skill-to-disable"]
  }
}
```

#### `hooks` — 훅 시스템 (Gemini CLI 전용)

에이전트 루프의 특정 이벤트 시점에 스크립트를 실행하여 동작을 사용자 정의합니다.

```json
{
  "hooks": {
    "enabled": false,
    "disabled": [],
    "notifications": true,
    "BeforeTool": [],
    "AfterTool": [],
    "BeforeAgent": [],
    "AfterAgent": [],
    "BeforeModel": [],
    "AfterModel": [],
    "BeforeToolSelection": [],
    "SessionStart": [],
    "SessionEnd": [],
    "PreCompress": [],
    "Notification": []
  }
}
```

- **BeforeTool**: 도구 실행 전 보안 취약점 검사 등을 수행합니다.
- **SessionStart**: 사용자가 대화를 시작할 때 현재 시스템의 로드 상황이나 특정 라이브러리 버전을 프롬프트에 주입합니다.

#### `ui` — UI 설정

```json
{
  "ui": {
    "theme": "GitHub",
    "hideBanner": false,
    "hideTips": false,
    "hideContextSummary": false,
    "showLineNumbers": true,
    "showCitations": false,
    "showModelInfoInChat": false,
    "useFullWidth": true,
    "useAlternateBuffer": false,
    "footer": {
      "hideCWD": false,
      "hideSandboxStatus": false,
      "hideModelInfo": false,
      "hideContextPercentage": true
    },
    "customWittyPhrases": [],
    "accessibility": {
      "disableLoadingPhrases": false,
      "screenReader": false
    }
  }
}
```

#### 기타 카테고리

```json
{
  "output": { "format": "text" },
  "ide": { "enabled": false },
  "privacy": { "usageStatisticsEnabled": true },
  "useWriteTodos": true,
  "admin": {
    "secureModeEnabled": false,
    "extensions": { "enabled": true },
    "mcp": { "enabled": true }
  },
  "telemetry": {
    "enabled": true,
    "target": "local",
    "otlpEndpoint": "http://localhost:4317",
    "otlpProtocol": "grpc",
    "logPrompts": false
  },
  "advanced": {
    "autoConfigureMemory": false,
    "excludedEnvVars": ["DEBUG", "DEBUG_MODE"]
  }
}
```

### 3.3 실전 settings.json 작성 샘플

자주 사용되는 주요 옵션들을 결합한 종합 설정 샘플입니다.

```json
{
  "general": {
    "previewFeatures": true,
    "vimMode": false,
    "checkpointing": { "enabled": false }
  },
  "model": {
    "name": "gemini-2.5-pro",
    "maxSessionTurns": -1,
    "compressionThreshold": 0.5
  },
  "context": {
    "fileName": ["GEMINI.md"],
    "includeDirectories": [],
    "fileFiltering": { "respectGitIgnore": true }
  },
  "tools": {
    "sandbox": "docker",
    "autoAccept": false,
    "allowed": ["run_shell_command(git)"],
    "useRipgrep": true
  },
  "experimental": {
    "skills": true,
    "enableAgents": false,
    "jitContext": false
  },
  "skills": {
    "disabled": []
  },
  "hooks": {
    "enabled": false,
    "BeforeTool": [],
    "AfterTool": [],
    "SessionStart": []
  },
  "mcpServers": {
    "myServer": {
      "command": "node",
      "args": ["server.js"]
    }
  }
}
```

#### **[주요 설정 포인트 설명]**
*   **일반 및 모델 (`general`, `model`)**: 최신 기능을 미리 사용해 볼 수 있도록 `previewFeatures`를 활성화하고, 최적의 성능을 위해 `gemini-2.5-pro` 모델을 세션 제한 없이 사용하도록 구성합니다.
*   **컨텍스트 및 도구 (`context`, `tools`)**: 프로젝트의 핵심 지침인 `GEMINI.md`를 우선 로드하며, 보안을 위해 **Docker 샌드박스** 내에서 `git` 관련 도구만 명시적으로 허용하도록 설정하여 안전한 자동화를 도모합니다.
*   **실험적 기능 및 스킬 (`experimental`, `skills`)**: 차세대 핵심 기능인 **Agent Skills**를 활성화하여 에이전트의 전문성을 높입니다. 단, 서브에이전트 기능은 예기치 못한 도구 실행을 방지하기 위해 비활성화 상태를 유지합니다.
*   **확장성 (`hooks`, `mcpServers`)**: 세션 시작 시 환경을 점검하거나 도구 실행 전후에 개입할 수 있는 `hooks` 시스템과, 외부 서비스를 연동할 수 있는 MCP 서버의 기본 구조를 포함하고 있습니다.

### 3.4 환경변수 참조 문법

settings.json 내 문자열 값에서 `$VAR_NAME` 또는 `${VAR_NAME}` 구문으로 환경변수를 참조할 수 있습니다.

```json
{
  "mcpServers": {
    "github": {
      "env": { "GITHUB_TOKEN": "$GITHUB_TOKEN" }
    }
  }
}
```

### 3.5 Antigravity의 대응 및 에이전트 모드

Antigravity는 별도의 settings.json 없이 **IDE 내 커스터마이징 UI**에서 규칙/워크플로우/스킬을 관리합니다. MCP 서버 설정, 모델 선택 등은 IDE 설정 패널에서 처리합니다. 특히 Antigravity에서는 에이전트의 페르소나를 결정하는 **에이전트 모드** 설정이 핵심이며, IDE 하단 상태 표시줄의 모드 선택기 또는 `settings.json` 내 `agent.mode` 필드를 통해 전환할 수 있습니다.

*   **Code 모드**: 테스트 주도 개발(TDD), 버그 수정, 신규 기능 구현 등 실제 코드 작성에 최적화된 모드입니다. 에이전트가 파일 수정 도구를 적극적으로 사용합니다.
*   **Architect 모드**: 대규모 리팩토링 계획 수립, 시스템 설계 분석, 의존성 맵 작성 등에 사용됩니다. 코드 수정보다는 구조적 제안에 집중합니다.
*   **Ask 모드**: 코드에 영향을 주지 않고 질문 답변, 코드 리뷰, 기술 문서 설명 등 지식 전달 위주의 작업을 수행합니다.
*   **설정 예시 (`settings.json`)**:
    ```json
    {
      "agent": {
        "mode": "code"
      }
    }
    ```

---

## 4. Auth 설정 - 인증 체계

### 4.1 Gemini CLI 인증 방식 개요
Gemini CLI는 크게 세 가지 계열의 인증을 지원합니다.

| 인증 방식 | 환경변수 | 용도 |
|-----------|---------|------|
| **Gemini API 키** | `GEMINI_API_KEY` | 개인 개발/테스트 (가장 단순합니다.) |
| **Google AI / Vertex AI API 키** | `GOOGLE_API_KEY` + `GOOGLE_GENAI_USE_VERTEXAI=true` | Vertex AI Express 사용 시 필요합니다. |
| **서비스 계정 / OAuth** | `GOOGLE_APPLICATION_CREDENTIALS` | 엔터프라이즈/프로덕션용입니다. |

**공통 보조 변수:**
*   `GOOGLE_CLOUD_PROJECT`: GCP 프로젝트 ID (필수인 경우 있습니다.)
*   `GOOGLE_CLOUD_LOCATION`: GCP 리전 (예: `us-central1`)
*   `GOOGLE_GENAI_USE_VERTEXAI`: `true`로 설정 시 Vertex AI 엔드포인트를 사용합니다.

### 4.2 Vertex AI 및 ADC 인증
Google Cloud Vertex AI(엔터프라이즈 환경)를 사용하기 위해 ADC(Application Default Credentials) 메커니즘을 통한 인증을 구성합니다.

#### **4.2.1 ADC 인증 방식 선택**
환경에 따라 에이전트가 인증 정보를 획득하는 방식을 결정합니다.

1.  **로컬 사용자 ADC 인증 (로컬 개발 권장)**
    개인 개발 환경에서 자신의 Google 계정 권한을 에이전트에게 부여합니다.
    *   **명령어**: `gcloud auth application-default login` 을 실행하여 브라우저 로그인을 수행합니다.
    *   **장점**: 별도의 키 파일 관리 없이 안전하게 인증 정보를 유지할 수 있습니다.

2.  **서비스 계정 키 인증 (서버/자동화용)**
    CI/CD 파이프라인이나 독립된 서버 환경에서 특정 권한을 가진 키 파일을 사용합니다.
    *   **설정**: 환경변수 `GOOGLE_APPLICATION_CREDENTIALS`에 발급받은 JSON 키 파일의 **절대 경로**를 지정합니다.
    ```bash
    export GOOGLE_APPLICATION_CREDENTIALS="/path/to/key.json"
    ```

#### **4.2.2 Vertex AI 환경변수 샘플**
GCP 프로젝트 정보와 Vertex AI 사용 여부를 정의합니다.
```bash
export GOOGLE_GENAI_USE_VERTEXAI=True
export GOOGLE_CLOUD_PROJECT="your-project-id"
export GOOGLE_CLOUD_LOCATION="global"
```

### 4.3 settings.json을 통한 정밀 제어 및 Fallback 설정
인증 유형을 고정하거나 환경변수가 누락되었을 때의 동작을 제어할 수 있습니다.

```json
{
  "security": {
    "auth": {
      "selectedType": "vertex-ai",
      "enforcedType": null,
      "useExternal": false
    }
  }
}
```
*   **selectedType**: 현재 선택된 인증 유형입니다. `/auth` 명령으로 변경 시 자동 저장됩니다.
*   **enforcedType**: 강제할 인증 유형입니다. 불일치 시 재인증을 요구하여 보안 정책을 준수하게 합니다.
*   **useExternal**: 외부 인증 플로우 사용 여부입니다.

#### **GCP 프로젝트/리전 Fallback 로직**
시스템 환경변수가 설정되지 않은 경우 에이전트는 `settings.json` 내의 값을 Fallback으로 참조합니다.
*   환경변수 `GOOGLE_CLOUD_PROJECT` 부재 시 → `settings.json`의 해당 값 참조합니다.
*   환경변수 `GOOGLE_CLOUD_LOCATION` 부재 시 → `settings.json`의 해당 값 참조합니다.

> **💡 인증 방식 리셋 팁:** 인증 방식이 꼬였을 때 `~/.gemini/settings.json`의 `security.auth.selectedType` 값을 지우거나 수정하여 초기화할 수 있습니다.

### 4.4 실전 인증 구성 전략

| 설정 대상 | 권장 위치 | 이유 |
|-----------|----------|------|
| API 키 / 서비스 계정 경로 | `.env` / OS 환경변수 | 보안상 코드 포함 방지 및 최우선 공식 방식입니다. |
| 프로젝트·리전 기본값 | `.gemini/settings.json` | 환경변수 누락 시를 대비한 안전한 Fallback 수단입니다. |
| 인증 유형 고정 정책 | `security.auth.enforcedType` | 관리자가 특정 인증 방식을 강제해야 할 때 사용합니다. |

### 4.5 환경변수 기반 설정 (일시용 예시)
터미널에서 즉시 인증을 설정할 때 사용하는 명령어 세트입니다.

```bash
# 1) 기존 값 제거 (충돌 방지)
unset GOOGLE_API_KEY GEMINI_API_KEY

# 2) Gemini API 키 설정 (ai.google.dev에서 발급받은 키)
export GEMINI_API_KEY="YOUR_GEMINI_API_KEY"

# 3) Vertex AI(Express) API 키를 사용할 경우
export GOOGLE_API_KEY="YOUR_GOOGLE_API_KEY"
export GOOGLE_GENAI_USE_VERTEXAI=true

# 4) Google Cloud 서비스 계정 키를 사용할 경우
export GOOGLE_APPLICATION_CREDENTIALS="/absolute/path/to/key.json"
export GOOGLE_CLOUD_PROJECT="your-gcp-project-id"
export GOOGLE_CLOUD_LOCATION="us-central1"
```

위와 같이 환경변수를 설정한 뒤, `gemini` 명령을 실행하여 대화창 내의 `/auth` 메뉴에서 실제 사용할 인증 방식을 최종 선택하십시오.

### 4.6 Antigravity 인증 흐름
Antigravity는 VSCode 기반 UI에서 초기 실행 시 다음과 같은 순서로 안내합니다.

1.  **모드 선택**: Secure / Review-driven / Agent-driven / Custom 중 선택합니다.
2.  **Model & Auth 설정**:
    - Google 계정 로그인 (gcloud / OAuth 플로우) 수행합니다.
    - Gemini API / Vertex AI API 키를 입력합니다.
3.  선택한 방법에 따라 IDE 내부에서 환경 설정을 관리합니다.

Antigravity는 별도 `.env` 파일을 직접 읽지는 않고, **VSCode 설정 및 OS 환경변수**를 활용합니다. 이미 `GEMINI_API_KEY` 등이 셸이나 OS에 설정되어 있다면 그대로 재사용됩니다.

> **참고:** Antigravity는 Google 계정 로그인이 필수이며 이를 우회할 방법은 없습니다.

---

## 5. .env 설정 - 환경변수 관리

### 5.1 Gemini CLI의 .env 로딩 순서
CLI는 다음 순서로 `.env` 파일을 탐색하며, 첫 번째 발견된 파일만 로드합니다 (병합하지 않습니다):

1. 현재 작업 디렉토리의 `.env`
2. 상위 디렉토리로 올라가며 탐색 (`.git` 또는 프로젝트 루트까지)
3. `~/.gemini/.env`
4. `~/.env`

> **주의:** `.gemini/.env`를 사용하는 것이 권장됩니다. 일반 `.env`는 다른 개발 도구와 설정값이 충돌할 위험이 있습니다.

### 5.2 전역 .env 설정 예시
어떤 디렉토리에서 `gemini`를 실행해도 기본값으로 사용될 전역 설정입니다.

```bash
mkdir -p ~/.gemini

cat >> ~/.gemini/.env << 'EOF'
GEMINI_API_KEY="your-gemini-api-key"
GOOGLE_CLOUD_PROJECT="your-project-id"
GOOGLE_CLOUD_LOCATION="us-central1"
EOF
```

### 5.3 프로젝트 전용 .env 설정 예시
프로젝트 루트에서 실행 시 전역 설정보다 우선하여 적용됩니다.

```bash
mkdir -p .gemini

cat >> .gemini/.env << 'EOF'
GEMINI_API_KEY="project-specific-api-key"
GOOGLE_CLOUD_PROJECT="project-specific-id"
GOOGLE_CLOUD_LOCATION="asia-northeast3"
EOF
```

### 5.4 전체 우선순위 종합 정리
설정값이 충돌할 경우 아래의 우선순위에 따라 최종 결정됩니다 (오른쪽으로 갈수록 높은 우선순위).

```
낮음 ←――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――→ 높음

시스템 기본값 → 사용자 settings.json → 프로젝트 settings.json
    → 전역 .env (~/.gemini/.env) → 프로젝트 .env (.gemini/.env)
        → 셸 환경변수 (export) → CLI 인수 플래그 (--model 등)
```

### 5.5 실전 활용 팁

| 용도 | 권장 위치 | 이유 |
|-----------|----------|------|
| 개인 계정의 기본 API 키 | `~/.gemini/.env` | 모든 프로젝트에서 공통 사용이 가능합니다. |
| 조직/프로젝트별 별도 키 | `.gemini/.env` | 프로젝트별 과금이나 권한을 분리할 때 사용합니다. |
| 임시 실험 및 테스트 | 셸 직접 `export` | 설정 파일 수정 없이 즉시 변경이 가능합니다. |
| 엔터프라이즈 강제 정책 | `/etc/gemini-cli/settings.json` | 시스템 수준에서 설정을 고정할 때 사용합니다. |

---

## 6. Rules - 항상 적용되는 가이드라인

Rules는 예외 없이 모든 대화에 적용되는 지침(예: 코딩 컨벤션)입니다. 에이전트의 '헌법'과 같은 역할을 수행합니다.

### 6.1 플랫폼별 규칙 적용 비교

| 항목 | Gemini CLI | Antigravity |
|------|-----------|-------------|
| **메커니즘** | `GEMINI.md` 계층 구조가 규칙 역할을 수행합니다. | 전용 규칙 시스템을 별도로 운영합니다. |
| **저장 위치** | **전역:** `~/.gemini/GEMINI.md` / **프로젝트:** `.gemini/GEMINI.md` | **전역:** `~/.gemini/GEMINI.md` / **프로젝트:** `.agent/rules/*.md` |
| **활성화 모드** | 항상 (자동) 로드됩니다. | **Always on** / **Manual** 중 선택 가능합니다. |
| **설정 방법** | 파일을 직접 편집합니다. | UI(Agent Brain 패널) 또는 파일을 직접 편집합니다. |

### 6.2 Antigravity 규칙 상세
Antigravity는 규칙의 활성화 레벨을 세밀하게 제어할 수 있습니다.

*   **System Rules**: Google DeepMind가 정의한 불변 규칙입니다. (수정 불가)
*   **Global Rules**: 모든 프로젝트에 공통으로 적용되는 개인 설정입니다.
*   **프로젝트 규칙**: 특정 프로젝트에만 적용되며 `.agent/rules/` 폴더 내에 마크다운 파일로 저장합니다. 에이전트는 이를 프로젝트 고유의 '원칙'으로 인식합니다.
*   **시각적 관리**: IDE의 '에이전트 브레인' 패널에서 로드된 규칙을 실시간으로 확인하고 새로고침하거나 비활성화할 수 있습니다.

### 6.3 실전 Rules 작성 샘플
프로젝트의 코딩 품질과 일관성을 유지하기 위한 `coding-standards.md` 예시입니다.

```markdown
# 프로젝트 코딩 표준 규칙

## 1. 기술 스택 및 언어
- **TypeScript Only**: 모든 신규 코드는 반드시 TypeScript를 사용해야 하며, `any` 타입 사용을 엄격히 금지합니다.
- **Functional Programming**: 가급적 순수 함수와 불변성을 지향하는 함수형 프로그래밍 스타일을 따릅니다.

## 2. 네이밍 및 구조
- **Components**: UI 컴포넌트는 PascalCase를 사용하며, 폴더명과 파일명을 일치시킵니다.
- **Hooks**: 모든 커스텀 훅은 `use` 접두사를 사용하고 `src/hooks` 경로에 배치합니다.

## 3. 문서화 및 주석
- **JSDoc**: 모든 익스포트되는 함수와 클래스에는 JSDoc 형식의 설명을 반드시 추가해야 합니다.
- **Why-Comments**: 코드의 작동 방식보다는 '왜' 이 로직이 필요한지를 설명하는 주석을 우선시합니다.

## 4. 제약 사항
- 외부 라이브러리 추가 전 반드시 `package.json`을 검토하여 중복 기능이 있는지 확인합니다.
- 모든 API 호출부는 반드시 에러 핸들링(`try-catch`)을 포함해야 합니다.
```

### 6.4. 규칙이 설정된 Antigravity
![규칙이 설정된 Antigravity](/assets/image/posts/gemini-antigravity-rules.png)


---

## 7. Workflows - 사용자 트리거 매크로

워크플로우는 사용자가 `/명령어`로 호출하거나 자연어로 요청하는 다단계 레시피입니다.

### 7.1 Antigravity Workflows (전용)
**저장 위치:**
*   **프로젝트:** `.agent/workflows/`
*   **전역:** `~/.gemini/antigravity/global_workflows/`

**핵심 기능:**

| 기능 | 설명 |
|------|------|
| **Smart Detection** | "컴포넌트 만들어줘" 같은 자연어로도 관련 워크플로우를 자동 탐지합니다. |
| **Slash Command** | `/deploy`처럼 직접 호출 시 `.agent/workflows/deploy.md`를 실행합니다. |
| **Turbo Mode** | `// turbo`(개별 단계) 또는 `// turbo-all`(전체)로 명령을 자동 승인합니다. |

#### 7.1.1 워크플로우 생성 방법 및 파일 형식
워크플로우는 **YAML 프론트매터 + 마크다운 단계** 구조로 작성합니다.

```markdown
---
description: Create a new React component with standard structure
---
# React 컴포넌트 생성

1. 사용자에게 생성할 컴포넌트의 이름을 묻습니다.
2. `src/components/[Name]` 디렉토리를 생성합니다.
3. `index.jsx` 파일을 보일러플레이트로 작성합니다. // turbo
4. `styles.css` 파일을 기본 스타일과 함께 생성합니다.
```

#### 7.1.2 실전 활용 예시

**예시 1: React 컴포넌트 스캐폴딩 (`create-component.md`)**
```markdown
---
description: 표준 구조의 React 컴포넌트 세트를 생성합니다.
---
# React 컴포넌트 생성 워크플로우

1. **정보 수집**: 사용자에게 생성할 컴포넌트의 이름을 확인합니다.
2. **디렉토리 생성**: `src/components/[Name]/` 경로에 새로운 디렉토리를 만듭니다.
3. **파일 작성**: // turbo
   - `index.tsx`: 컴포넌트 엔트리 파일을 작성합니다.
   - `[Name].tsx`: 기본 함수형 컴포넌트 구조를 작성합니다.
   - `styles.css`: 기본 스타일 시트를 생성합니다.
4. **등록**: 생성된 컴포넌트를 `src/components/index.ts`에 익스포트하여 등록합니다.
```

**예시 2: API 엔드포인트 및 테스트 생성 (`add-api-route.md`)**
```markdown
---
description: 새로운 API 라우트를 추가하고 관련 단위 테스트를 작성합니다.
---
# API 개발 및 검증 워크플로우

1. **라우트 생성**: `src/pages/api/` 폴더 내에 사용자가 요청한 엔드포인트 파일을 생성합니다.
2. **로직 구현**: 해당 API의 비즈니스 로직을 표준 핸들러 형식으로 구현합니다.
3. **테스트 작성**: `tests/api/` 경로에 해당 라우트를 검증하는 단위 테스트를 작성합니다.
4. **자율 검증**: // turbo
   - `npm test` 명령어를 실행하여 테스트 성공 여부를 확인합니다.
   - 테스트 실패 시 로그를 분석하여 코드를 수정한 후 재시도합니다.
```

### 7.2 Gemini CLI의 대응
동일한 워크플로우 시스템은 없으나, **커스텀 커맨드** (`.gemini/commands/*.toml`) 및 **훅** 시스템을 통해 유사한 다단계 자동화를 구현합니다.

### 7.3. 워크플로우가 설정된 Antigravity
![워크플로우가 설정된 Antigravity](/assets/image/posts/gemini-antigravity-workflows.png)


### 7.4 규칙과 워크플로우의 차이 비교

| 구분 | 규칙 (Rules) | 워크플로우 (Workflows) |
|------|------------|---------------------|
| **핵심 개념** | 에이전트의 **'원칙'과 '성격'** 을 정의합니다. | 에이전트가 수행할 **'절차적 레시피'** 를 정의합니다. |
| **작동 방식** | 모든 대화에 상시 적용됩니다 (시스템 프롬프트). | 사용자 호출(`/명령어`) 또는 특정 조건 시 트리거됩니다. |
| **실행 구조** | **정적**: 항상 준수해야 할 지침 목록입니다. | **동적**: 계획-수행-검증의 단계적 흐름입니다. |
| **주요 용도** | 코딩 컨벤션, 필수 라이브러리 제약, 언어 설정 등입니다. | 신규 컴포넌트 생성, 대규모 리팩토링, 배포 자동화 등입니다. |

### 7.5 규칙과 워크플로우의 상호 보완적 활용

*   **워크플로우 (The What - 절차)**: 에이전트가 수행해야 할 **작업의 순서와 단계**를 정의합니다.
*   **규칙 (The How - 방법과 품질)**: 에이전트가 작업을 수행하는 과정에서 준수해야 할 **품질 기준과 제약 조건**을 정의합니다.

#### 활용 시나리오 1: "보안 강화 API 개발 파이프라인"

**워크플로우 정의 (`.agent/workflows/secure-api.md`)**
```markdown
# 보안 API 개발 워크플로우
1. 사용자가 요청한 엔드포인트를 생성합니다.
2. 비즈니스 로직을 구현합니다. // turbo
3. 보안 스캔 도구를 실행하여 취약점을 점검합니다.
4. 결과가 통과되면 코드를 메인 브랜치에 병합합니다.
```

**규칙 정의 (`.agent/rules/security-standards.md`)**
```markdown
# API 보안 표준 규칙
- 모든 API 엔드포인트는 반드시 JWT 인증 미들웨어를 포함해야 합니다.
- 데이터베이스 쿼리 시 반드시 ORM의 Parameterized Query를 사용하여 SQL Injection을 방지합니다.
- 에러 응답 시 스택 트레이스 등 시스템 내부 정보를 노출하지 않습니다.
```

#### 활용 시나리오 2: "디자인 시스템 기반 UI 컴포넌트 개발 (TDD)"

**워크플로우 정의 (`.agent/workflows/create-ui-component.md`)**
```markdown
# UI 컴포넌트 TDD 워크플로우
1. 컴포넌트의 요구사항과 Props 명세를 확인합니다.
2. `src/components/` 경로에 테스트 파일(`.test.tsx`)을 먼저 생성합니다.
3. 테스트가 실패하는 것을 확인한 후, 컴포넌트 본체 파일(`.tsx`)을 생성합니다. // turbo
4. 스타일과 로직을 구현하여 모든 테스트를 통과시킵니다.
5. 디자인 시스템 가이드를 준수했는지 최종 검토합니다.
```

**규칙 정의 (`.agent/rules/design-system-standards.md`)**
```markdown
# 디자인 시스템 및 품질 규칙
- **Color**: 하드코딩된 HEX 코드를 금지하며, 반드시 `theme.color.*` 토큰을 사용해야 합니다.
- **Accessibility**: 모든 대화형 요소(Button, Input)는 반드시 적절한 `aria-label`을 포함해야 합니다.
- **Testing**: 모든 UI 컴포넌트는 최소한 '렌더링 여부'와 '클릭 이벤트 발생 여부'에 대한 테스트를 포함해야 합니다.
```

#### 활용 시나리오 3: "자동화된 릴리즈 및 변경 이력 관리"

**워크플로우 정의 (`.agent/workflows/release.md`)**
```markdown
# 릴리즈 자동화 워크플로우
1. 마지막 태그 이후의 모든 커밋 내역을 분석합니다.
2. 분석된 내역을 바탕으로 `CHANGELOG.md` 파일을 업데이트합니다. // turbo
3. 프로젝트 버전을 한 단계 올립니다 (Patch/Minor/Major 결정).
4. 새로운 버전으로 Git 태그를 생성하고 원격에 푸시합니다.
```

**규칙 정의 (`.agent/rules/release-standards.md`)**
```markdown
# 릴리즈 및 버저닝 규칙
- **Versioning**: 반드시 SemVer 형식을 따라야 합니다.
- **Changelog Style**: 단순 커밋 나열이 아닌, 'Feature', 'Fix', 'Breaking Changes'로 카테고리를 나누어 요약해야 합니다.
- **Exclude**: 'chore', 'refactor' 성격의 커밋은 릴리즈 노트에서 제외합니다.
```

---

## 8. Agent Skills - 핵심 상호호환 계층

**Agent Skills는 Anthropic이 도입하고 오픈 표준(agentskills.io)으로 공개된 포맷**으로, Gemini CLI와 Antigravity에서 동일한 `SKILL.md` 파일로 작동합니다.

### 8.1 Skills vs Rules vs Workflows 핵심 차이

| 구분 | 로드 시점 | 트리거 방식 | 용도 |
|------|----------|-----------|------|
| **Rules** | 항상 (시스템 프롬프트) | 자동 적용됩니다. | 코딩 스타일, 필수 컨벤션 설정용입니다. |
| **Workflows** | 사용자 호출 시 | `/명령어` 또는 자연어 | 멀티스텝 매크로 수행용입니다. |
| **Skills** | 에이전트 판단 시 | 자동 (의미 매칭) | 주문형 전문 지식 제공용입니다. |

### 8.2 SKILL.md 구조 (공통 표준)

```yaml
---
name: deploy-staging
description: Deploys current branch to staging environment.
  Use when user asks to "deploy", "push to staging",
  or "test on staging server".
---
# Deploy to Staging

## Prerequisites
1. Verify `git status` is clean
2. Run `npm run test`

## Deployment Steps
1. Execute `./scripts/deploy.sh staging`
2. Wait for health check HTTP 200
3. Report staging URL to user
```

> **중요:** `description` 필드가 가장 중요합니다. 에이전트가 어떤 상황에서 이 스킬을 꺼내 쓸지 결정하는 기준이 되기 때문입니다.

### 8.3 권장 스킬 폴더 구조

```text
my-skill/
├── SKILL.md       (필수) 지침 및 메타데이터 파일입니다.
├── scripts/       (선택) 실행 가능한 스크립트들을 포함합니다.
├── references/    (선택) 정적 문서 자료들을 포함합니다.
└── assets/        (선택) 템플릿 및 기타 자원들을 포함합니다.
```

### 8.4 저장 위치 및 로딩 경로
*   **Gemini CLI**: 전역 `~/.gemini/skills/` / 프로젝트 `.gemini/skills/`
*   **Antigravity**: 전역 `~/.gemini/antigravity/skills/` / 프로젝트 `.agent/skills/`

### 8.5 스킬 관리 명령어 (Gemini CLI)

```bash
/skills list          # 전체 스킬 목록을 표시합니다.
/skills link <path>   # 로컬 디렉토리 심볼릭 링크를 생성합니다.
/skills disable <name> # 특정 스킬을 비활성화합니다.
/skills enable <name>  # 특정 스킬을 재활성화합니다.
/skills reload         # 모든 스킬을 다시 스캔합니다.
```

### 8.6 실전 스킬 샘플

#### **예시 1: `git-helper` (버전 관리 도우미)**

```markdown
---
name: git-helper
description: Git 커밋 메시지 작성, PR 초안 작성을 돕습니다. "커밋해줘" 요청 시 사용합니다.
---
# Git Helper Instructions
당신은 버전 관리 전문가입니다.
1. `git diff --staged`를 분석하여 시맨틱 커밋 메시지(feat, fix 등)를 제안합니다.
2. 사용자에게 메시지 승인을 받은 후 `git commit` 명령을 수행합니다.
3. **보안 철칙**: 비밀번호나 API 키가 포함된 경우 절대 커밋하지 않고 즉시 경고합니다.
```

#### **예시 2: `python-quality` (파이썬 품질 전문가)**

```markdown
---
name: python-quality
description: 파이썬 코드 품질을 검사하고 수정합니다. "코드 봐줘" 요청 시 사용합니다.
---
# Python Quality Expert
1. `pyproject.toml` 설정을 확인하고 `ruff` 또는 `black` 도구를 실행합니다.
2. 도구가 없는 경우 PEP 8 표준에 따라 수동 리뷰를 수행하고 보고합니다.
3. 이슈 발견 시 한국어로 설명하고, 개선된 코드를 직접 제안하거나 적용합니다.
```

---

## 9. MCP - 외부 통합

에이전트가 외부 시스템(파일, DB, SaaS 등)과 통신하기 위한 표준 프로토콜입니다.

### 9.1 Gemini CLI MCP 설정

```json
{
  "mcpServers": {
    "sequential-thinking": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"]
    },
    "github": {
      "command": "node",
      "args": ["./mcp/github-server.js"],
      "env": { "GITHUB_TOKEN": "$GITHUB_TOKEN" },
      "timeout": 60000,
      "trust": false
    }
  }
}
```

#### **MCP 서버 속성 상세**

| 속성 | 설명 |
|------|------|
| `command` | MCP 서버 실행 프로그램 (node, python, uv 등) 입니다. |
| `args` | 서버 스크립트 및 인자입니다. |
| `env` | 서버에 전달할 환경 변수입니다 (`$`로 OS 변수 참조 가능합니다). |
| `timeout` | 응답 타임아웃 (ms) 입니다. |
| `trust` | `true` 설정 시 모든 도구를 자동 승인합니다. |
| `includeTools` | 허용할 도구 화이트리스트입니다. |
| `excludeTools` | 제외할 도구 블랙리스트입니다. |

#### **지원 트랜스포트 구분**

| 트랜스포트 | 설정 키 | 용도 |
|-----------|---------|------|
| **Stdio** | `command` + `args` | 로컬 프로세스 간 통신 시 사용합니다. |
| **SSE** | `url` | Server-Sent Events 방식입니다. |
| **Streamable HTTP** | `httpUrl` | HTTP 기반 스트리밍 방식입니다. |

#### **gemini mcp CLI 명령어**

```bash
gemini mcp add my-server python server.py
gemini mcp add --transport http http-server https://api.example.com/mcp/
gemini mcp list
gemini mcp remove my-server
/mcp
```

### 9.2 MCP 설정 비교 요약

| 항목 | Gemini CLI | Antigravity |
|------|-----------|-------------|
| **설정 파일** | `settings.json` 내 `mcpServers` | `mcp_config.json` |
| **설정 위치** | `.gemini/settings.json` | `~/.gemini/antigravity/mcp_config.json` |
| **트랜스포트** | stdio, SSE, Streamable HTTP | stdio, SSE |
| **UI 관리** | 없음 (CLI 전용) | MCP Store + 설정 패널 제공합니다. |

---

## 10. Agent 아키텍처 및 특수 에이전트

| 항목 | Gemini CLI | Antigravity |
|------|-----------|-------------|
| **에이전트 유형** | 단일 대화형 에이전트 (순차 실행) | 멀티 에이전트 병렬 실행 시스템입니다. |
| **서브에이전트** | 실험적 기능으로 제공됩니다. | 매니저 표면을 통해 기본 지원합니다. |
| **컨텍스트 윈도우** | 1M (Free) ~ 2M (Enterprise) 토큰입니다. | 1M+ 토큰의 대용량 윈도우를 제공합니다. |
| **샌드박스** | Docker/Podman/macOS 지원합니다. | IDE 내장 샌드박스를 사용합니다. |

### 10.1 Gemini CLI 실험적 에이전트 설정

```json
{
  "experimental": {
    "enableAgents": true,
    "codebaseInvestigatorSettings": {
      "enabled": true,
      "maxNumTurns": 10,
      "maxTimeMinutes": 3,
      "thinkingBudget": 8192,
      "model": "auto"
    }
  }
}
```

### 10.2 주요 Subagents
*   `codebase_investigator`: 프로젝트 의존성 구조 파악 및 심볼 추적을 수행합니다.
*   `cli_help`: CLI 사용법 및 환경 설정 질문에 특화되어 있습니다.

### 10.3 Browser Subagent (Antigravity 전용)
실시간 웹 상호작용 및 문서 탐색을 수행하는 서브 에이전트로, 전용 프로필을 사용하여 개인정보를 보호합니다.

---

## 11. 세션 및 작업 관리

### 11.1 세션 보존 및 복구 (Gemini CLI)
*   **체크포인트**: 코드 수정 직전 스냅샷과 대화 상태를 자동 저장합니다.
*   **복구**: `/restore` 명령으로 시점을 선택하고 롤백합니다.
*   **재개**: `gemini --resume` 명령으로 과거 세션을 검색하고 재개합니다.

### 11.2 작업 그룹 (Antigravity)
*   **할 일 연동**: 에이전트가 그룹 내 할 일을 완수하며 진행률을 시각적으로 보고합니다.
*   **이정표(Milestone)**: 검증 성공 시 이정표를 기록하고 다음 단계 계획을 수립합니다.

---

## 12. 상호호환성 종합 매핑

| 기능 | Gemini CLI | Antigravity | 호환성 |
|------|-----------|-------------|--------|
| **전역 컨텍스트** | `~/.gemini/GEMINI.md` | `~/.gemini/GEMINI.md` | ✅ 동일 파일 공유 |
| **프로젝트 컨텍스트** | `.gemini/GEMINI.md` | `.agent/rules/*.md` | ⚠️ 경로·형식 상이 |
| **Agent Skills** | `.gemini/skills/` | `.agent/skills/` | ✅ 오픈 표준 호환 |
| **워크플로우** | Custom Commands | `.agent/workflows/` | ❌ Antigravity 전용 |
| **동작 설정** | `settings.json` (JSON) | IDE 설정 UI | ❌ 형식 불호환 |
| **MCP 서버** | `mcpServers` | `mcp_config.json` | ⚠️ 기능 동일, 파일 형식 상이 |
| **훅** | `hooks.*` | 없음 | ❌ Gemini CLI 전용 |
| **인증** | `.env` + `security.auth` | IDE UI + OS 환경변수 | ⚠️ 환경변수 공유 가능 |

---

## 13. 통합 실전 구성 예시 및 팁

### 13.1 권장 디렉토리 구조

**사용자 홈 디렉토리 (전역):**
```
~/.gemini/
├── .env                    # 전역 인증 (API 키, GCP 프로젝트 등)
├── settings.json           # Gemini CLI 전역 설정
├── GEMINI.md               # 전역 공통 컨텍스트 (양쪽 공유)
├── skills/                 # 전역 Agent Skills (CLI용)
└── antigravity/
    ├── mcp_config.json     # Antigravity 전용 MCP 설정
    ├── skills/             # 전역 Agent Skills (IDE용)
    └── global_workflows/   # Antigravity 전용 전역 워크플로우
```

**프로젝트 디렉토리:**
```
my-project/
├── .gemini/                    # Gemini CLI 전용 영역
│   ├── GEMINI.md
│   ├── settings.json
│   └── skills/
│       └── deploy-staging/
│           └── SKILL.md
├── .agent/                     # Antigravity 전용 영역
│   ├── rules/
│   ├── workflows/
│   └── skills/
│       └── deploy-staging -> ../../.gemini/skills/deploy-staging  # 심볼릭 링크
└── ...
```

> **💡 심볼릭 링크 활용 팁:** `ln -s ../../.gemini/skills/deploy-staging .agent/skills/deploy-staging` 으로 한 곳에서 수정하면 양 플랫폼에 즉시 반영됩니다.

### 13.2 실전 활용 팁
1.  **환경 전환**: 터미널 작업 중 시각적 워크플로우가 필요하면 Antigravity로 전환하십시오. 설정이 공유되어 맥락이 유지됩니다.
2.  **보안 우선**: `settings.json`에서 **샌드박스**를 활성화하여 신뢰할 수 없는 코드 실행을 격리하십시오.
3.  **지침 최적화**: `GEMINI.md` 수정 후 `/memory refresh`를 실행하여 즉시 반영하십시오.
4.  **CI/CD 통합**: 파이프라인 자동화는 Gemini CLI와 GitHub Actions 조합이 가장 유리합니다.

---

## 14. 참고 자료

*   [Gemini CLI Configuration (GitHub)](https://github.com/google-gemini/gemini-cli/blob/main/docs/get-started/configuration.md)
*   [Gemini CLI Authentication (GitHub Pages)](https://google-gemini.github.io/gemini-cli/docs/get-started/authentication.html)
*   [Gemini CLI MCP Server (GitHub Pages)](https://google-gemini.github.io/gemini-cli/docs/tools/mcp-server.html)
*   [Gemini CLI Agent Skills](https://geminicli.com/docs/cli/skills/)
*   [Antigravity Rules/Workflows](https://antigravity.codes/blog/user-rules)
*   [Antigravity Skills Codelab](https://codelabs.developers.google.com/getting-started-with-antigravity-skills)
*   [Google Cloud Blog: Choosing Antigravity or Gemini CLI](https://cloud.google.com/blog/topics/developers-practitioners/choosing-antigravity-or-gemini-cli)
