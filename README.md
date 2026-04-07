# GitHub Copilot Agent Configuration

This repository contains my personal configuration for GitHub Copilot Agent in VS Code. \
It can serve as a useful starting point for those new to GitHub Copilot in VS Code. \
These instructions are designed to help achieve effective responses to zero-shot prompting during typical development tasks.

## Environment

- **VS Code**: Version 1.114
- Required MCP commands: `npx`, `docker`, `uvx`
- Tested models: `Claude Sonnet 4.6`

> **Note**: Behavior may vary depending on the model. Each model interprets instructions and context differently.

## Overview

- `.agents`
  - [.agents/skills](.agents/skills): Agent Skills
- `.github`
  - [.github/agents](.github/agents): Custom Agents
  - [.github/instructions](.github/instructions): Custom Instructions
  - [.github/copilot-instructions.md](.github/copilot-instructions.md): Workspace-optimized instructions
- `.vscode`
  - [.vscode/mcp.json](.vscode/mcp.json): Default MCP server configuration
  - [.vscode/settings.json](.vscode/settings.json): Default VS Code settings

## Configuration Files

### Custom Instructions

- [copilot-agent.instructions.md](.github/instructions/copilot-agent.instructions.md): Single Source of Truth (SSOT) for agent behavior — enforces sequential-thinking on every response, skill gate evaluation, and subagent delegation for all I/O and implementation work
- [subagent-templates.instructions.md](.github/instructions/subagent-templates.instructions.md): Context Package contract, sequential vs. parallel decision guide, and prompt templates for dispatching research and implementation subagents

### Agent Skills

| Skill | Source | Description |
| ----- | ------ | ----------- |
| `documentation` | [mcollina/skills](https://github.com/mcollina/skills) | Creates and structures technical docs following the Diátaxis framework |
| `init` | [mcollina/skills](https://github.com/mcollina/skills) | Creates/optimizes AGENTS.md with minimal, high-signal agent instructions |
| `karpathy-guidelines` | [forrestchang/andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills) | Behavioral guidelines to reduce common LLM coding mistakes |
| `linting-neostandard-eslint9` | [mcollina/skills](https://github.com/mcollina/skills) | Configures ESLint v9 flat config and neostandard for JS/TS projects |
| `node` | [mcollina/skills](https://github.com/mcollina/skills) | Node.js best practices with native TypeScript, async patterns, and more |
| `skill-optimizer` | [mcollina/skills](https://github.com/mcollina/skills) | Optimizes AI skill files for activation, clarity, and cross-model reliability |
| `typescript-magician` | [mcollina/skills](https://github.com/mcollina/skills) | Designs complex generic types, removes `any`, creates type guards |

```sh
npx skills add forrestchang/andrej-karpathy-skills
npx skills add mcollina/skills
```

### Custom Agents

- [critical-thinking.agent.md](.github/agents/critical-thinking.agent.md): Question assumptions and guide toward optimal solutions
- [mentor.agent.md](.github/agents/mentor.agent.md): Support engineer growth through critical questioning and mentorship

## Recommended Workflows

[get-shit-done](https://github.com/gsd-build/get-shit-done)

```bash
npx get-shit-done-cc@latest --copilot
```

After installing, verify with `/gsd-help` in Copilot chat.

## VS Code Documentation

- [Custom Instructions](https://code.visualstudio.com/docs/copilot/customization/custom-instructions)
- [Custom Agents](https://code.visualstudio.com/docs/copilot/customization/custom-agents)
- [Prompt Files](https://code.visualstudio.com/docs/copilot/customization/prompt-files)
- [Agent Skills](https://code.visualstudio.com/docs/copilot/customization/agent-skills)
- [Agent Hooks](https://code.visualstudio.com/docs/copilot/customization/hooks)
- [Agent Plugins](https://code.visualstudio.com/docs/copilot/customization/agent-plugins)

## Links

- [Awesome GitHub Copilot](https://github.com/github/awesome-copilot)
- [GitHub Copilot plugins](https://github.com/github/copilot-plugins)
- [GitHub Copilot CLI](https://github.com/github/copilot-cli)
- [GitHub Agentic Workflows](https://github.com/github/gh-aw)
