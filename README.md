# GitHub Copilot Agent Configuration

This repository contains my personal configuration for GitHub Copilot Agent in VS Code. \
It can serve as a useful starting point for those new to GitHub Copilot in VS Code. \
These instructions are designed to help achieve effective responses to zero-shot prompting during typical development tasks.

<img width="679" height="437" alt="image" src="https://github.com/user-attachments/assets/beb79374-defa-4b36-a9bf-68ddb9666b78" />

## Environment

- **VS Code**: Version 1.111
- Required MCP commands: `npx`, `docker`, `uvx`
- Tested models: `Claude Sonnet 4.6`

> **Note**: Behavior may vary depending on the model. Each model interprets instructions and context differently.

## Overview

- `.github`
  - [.github/agents](.github/agents): Custom Agents
  - [.github/instructions](.github/instructions): Custom Instructions
  - [.github/skills](.github/skills): Agent Skills
  - [.github/copilot-instructions.md](.github/copilot-instructions.md): Workspace-optimized instructions
- `.vscode`
  - [.vscode/mcp.json](.vscode/mcp.json): Default MCP server configuration
  - [.vscode/settings.json](.vscode/settings.json): Default VS Code settings

## Configuration Files

### Custom Instructions

- [copilot-agent.instructions.md](.github/instructions/copilot-agent.instructions.md): Single Source of Truth — enforces 3 mandatory behaviors (sequential-thinking, skill gate, subagent delegation) on every response, with a 5-phase execution protocol and forbidden call rules
- [subagent-templates.instructions.md](.github/instructions/subagent-templates.instructions.md): Reference for subagents — Context Package contract, Sequential vs. Parallel decision guide, and copy-paste prompt templates for research and implementation subagents

### Agent Skills

- [investigation-mode](.github/skills/investigation-mode/SKILL.md): Stops implementation when the same failure recurs after 2+ fix attempts, switches to evidence-first root-cause analysis, and only resumes once a verified plan is in place
- [minimalist-surgical-development](.github/skills/minimalist-surgical-development/SKILL.md): Keeps changes as small as possible — prefers existing utilities, avoids new abstractions, and never refactors code that wasn't part of the original request
- [root-cause-tracing](.github/skills/root-cause-tracing/SKILL.md): Traces a bug backward through the call stack or data propagation chain to find where it actually originates, rather than patching where it surfaces
- [task-direction-approval](.github/skills/task-direction-approval/SKILL.md): When a change in direction is needed (different library, new dependency, architectural shift, or alternative approach), explains the root cause, offers 2–3 options with trade-offs, and waits for explicit user approval before proceeding
- [uncertainty-verification](.github/skills/uncertainty-verification/SKILL.md): Looks up exact command flags, config keys, API paths, and version-specific behavior in official documentation before stating them — never relies on assumed or remembered specifics
- [verification-before-completion](.github/skills/verification-before-completion/SKILL.md): Requires running the relevant verification commands and confirming their output before claiming that any work is complete or fixed — evidence first, always

### Custom Agents

- [critical-thinking.agent.md](.github/agents/critical-thinking.agent.md): Question assumptions and guide toward optimal solutions
- [mentor.agent.md](.github/agents/mentor.agent.md): Support engineer growth through critical questioning and mentorship

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
