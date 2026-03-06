# GitHub Copilot Agent 설정

이 저장소는 제가 VS Code에서 GitHub Copilot Agent를 사용할 때 설정하는 구성입니다. \
VS Code에서 GitHub Copilot을 처음 사용하시는 분들께는 유용한 시작점이 될 수 있습니다. \
이 구성은 일반적인 개발 작업에서 사용자 입장에서의 제로샷 프롬프팅에 대한 효과적인 응답을 위한 지침입니다.

## 환경

- **VS Code**: 버전 1.110.0
- 필요 MCP 실행 커맨드: `npx`, `docker`, `uvx`
- 테스트한 모델: `Claude Sonnet 4.6`

> **참고**: 사용하는 모델에 따라 작동 방식이 다를 수 있습니다. 각 모델은 지침과 컨텍스트를 다르게 해석할 수 있습니다.

## 개요

- `.github`
  - [.github/agents](.github/agents): 커스텀 에이전트
  - [.github/instructions](.github/instructions): 커스텀 지침
  - [.github/skills](.github/skills): 에이전트 스킬
  - [.github/copilot-instructions.md](.github/copilot-instructions.md): 워크스페이스에 특화된 지침 작성
- `.vscode`
  - [.vscode/mcp.json](.vscode/mcp.json): 기본 MCP 서버 구성
  - [.vscode/settings.json](.vscode/settings.json): 기본 VS Code 설정

## 설정 파일

### 커스텀 지침

- [copilot-agent.instructions.md](.github/instructions/copilot-agent.instructions.md): 단일 진실 공급원(SSOT) — 매 응답마다 3가지 필수 동작(순차적 사고, 스킬 게이트, 서브에이전트 위임)을 강제하며 6단계 실행 프로토콜과 금지 호출 규칙을 포함

### 에이전트 스킬

- [investigation-mode](.github/skills/investigation-mode/SKILL.md): 같은 실패가 2회 이상 반복되면 구현을 중단하고 증거 기반의 근본 원인 분석으로 전환하며, 검증된 계획이 수립된 후에만 재개합니다
- [minimalist-surgical-development](.github/skills/minimalist-surgical-development/SKILL.md): 변경 범위를 최대한 작게 유지하고, 기존 유틸리티를 우선 활용하며, 요청 범위를 벗어난 새로운 추상화나 리팩토링을 추가하지 않습니다
- [root-cause-tracing](.github/skills/root-cause-tracing/SKILL.md): 버그가 나타난 지점을 패치하는 대신, 콜스택이나 데이터 전파 경로를 역추적하여 실제 발생 원인을 찾아냅니다
- [task-direction-approval](.github/skills/task-direction-approval/SKILL.md): 라이브러리 교체, 새 의존성 추가, 아키텍처 변경, 접근 방식 전환 등 방향 변경이 필요한 경우 근본 원인을 설명하고 2–3가지 옵션과 트레이드오프를 제시한 후 사용자의 명시적 선택을 기다립니다
- [uncertainty-verification](.github/skills/uncertainty-verification/SKILL.md): 정확한 명령어 플래그, 설정 키, API 경로, 버전별 동작 방식은 공식 문서를 통해 직접 확인한 후 제시하며, 기억이나 추정에 의존하지 않습니다
- [verification-before-completion](.github/skills/verification-before-completion/SKILL.md): 작업 완료나 수정 완료를 주장하기 전에 항상 관련 검증 명령어를 실행하고 결과를 확인합니다 — 커밋이나 PR에 국한하지 않고 모든 완료 주장에 적용됩니다

### 커스텀 에이전트

- [critical-thinking.agent.md](.github/agents/critical-thinking.agent.md): 전제를 질문하고 최적의 해결책 도출 지원
- [mentor.agent.md](.github/agents/mentor.agent.md): 비판적 질문으로 엔지니어 성장 지원 및 멘토링 제공

## VS Code 문서

- [커스텀 지침](https://code.visualstudio.com/docs/copilot/customization/custom-instructions)
- [커스텀 에이전트](https://code.visualstudio.com/docs/copilot/customization/custom-agents)
- [프롬프트 파일](https://code.visualstudio.com/docs/copilot/customization/prompt-files)
- [에이전트 스킬](https://code.visualstudio.com/docs/copilot/customization/agent-skills)
- [에이전트 훅](https://code.visualstudio.com/docs/copilot/customization/hooks)
- [에이전트 플러그인](https://code.visualstudio.com/docs/copilot/customization/agent-plugins)
