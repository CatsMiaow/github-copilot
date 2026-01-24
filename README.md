# GitHub Copilot Agent Configuration

This repository contains my personal configuration for GitHub Copilot Agent in VS Code. \
It can serve as a useful starting point for those new to GitHub Copilot in VS Code. \
These instructions are designed to help achieve effective responses to zero-shot prompting during typical development tasks.

<img width="1378" height="1054" alt="image" src="https://github.com/user-attachments/assets/822a81e6-fb3d-42b0-9a77-2635fa739acf" />

## Overview

- `.github`
  - [.github/agents](.github/agents): Custom Agents
  - [.github/instructions](.github/instructions): Custom Instructions
  - [.github/skills](.github/skills): Agent Skills
  - [.github/copilot-instructions.md](.github/copilot-instructions.md): Workspace-optimized instructions
- `.vscode`
  - [.vscode/mcp.json](.vscode/mcp.json): Default MCP server configuration
  - [.vscode/settings.json](.vscode/settings.json): Default VS Code settings

## Environment

- **VS Code**: Version 1.108.x
- Required MCP commands: `npx`, `docker`, `uvx`
- Tested models: `Claude Opus 4.5`, `Claude Sonnet 4.5`, `GPT-5.2`

> **Note**: Behavior may vary depending on the model. Each model interprets instructions and context differently.

## Configuration Files

### Custom Instructions

- [copilot-agent.instructions.md](.github/instructions/copilot-agent.instructions.md): 6-phase execution protocol with SSOT hierarchy, quality standards, and no-assumption rules
- [subagents.instructions.md](.github/instructions/subagents.instructions.md): SSOT defining forbidden MCP calls, delegation rules, and Context Package contract
- [taming-copilot.instructions.md](.github/instructions/taming-copilot.instructions.md): User directive priority, code-on-request-only, and direct/concise response philosophy
- [task-direction-approval.instructions.md](.github/instructions/task-direction-approval.instructions.md): Mandatory user consent before tech/architecture changes with failure-to-options protocol

### Agent Skills

- [investigation-mode](.github/skills/investigation-mode/SKILL.md): Pause implementation after 2+ consecutive failures, switch to root-cause analysis before resuming
- [minimalist-surgical-development](.github/skills/minimalist-surgical-development/SKILL.md): Smallest diff first, standard libraries preferred, no unsolicited refactoring
- [root-cause-tracing](.github/skills/root-cause-tracing/SKILL.md): Trace bugs backward through call stack to find the original trigger, not the symptom
- [task-direction-approval](.github/skills/task-direction-approval/SKILL.md): Explain failure cause, present 2-3 options with trade-offs, wait for explicit user choice
- [uncertainty-verification](.github/skills/uncertainty-verification/SKILL.md): Verify exact commands/configs/APIs via official docs before stating - no assumption-based specifics
- [verification-before-completion](.github/skills/verification-before-completion/SKILL.md): Run verification commands and confirm output before any completion claims - evidence before assertions

### Custom Agents

- [critical-thinking.agent.md](.github/agents/critical-thinking.agent.md): Question assumptions and guide toward optimal solutions
- [mentor.agent.md](.github/agents/mentor.agent.md): Support engineer growth through critical questioning and mentorship

## VS Code Documentation

- [Custom Instructions](https://code.visualstudio.com/docs/copilot/customization/custom-instructions)
- [Custom Agents](https://code.visualstudio.com/docs/copilot/customization/custom-agents)
- [Prompt Files](https://code.visualstudio.com/docs/copilot/customization/prompt-files)
- [Agent Skills](https://code.visualstudio.com/docs/copilot/customization/agent-skills)

## Links

- [Awesome GitHub Copilot](https://github.com/github/awesome-copilot)
- [Spec-Driven Development](https://github.com/github/spec-kit)
- [Awesome Claude Skills](https://github.com/ComposioHQ/awesome-claude-skills)
- [GitHub Copilot CLI](https://github.com/github/copilot-cli)
- [GitHub Agentic Workflows](https://github.com/githubnext/gh-aw)
