---
applyTo: '**'
---
# Subagent Instructions (SSOT)

## ⛔ FORBIDDEN DIRECT CALLS

**Before ANY tool call, ask: "Is this investigation I/O?" → If YES → SUBAGENT**

| ❌ FORBIDDEN | Tool Pattern | ✅ DELEGATE TO |
|-------------|--------------|----------------|
| Context7 MCP | `mcp_context7_*` | Research subagent |
| Serena MCP | `mcp_serena_*` | Research subagent |
| GitHub MCP | `mcp_github_*` | Research subagent |
| Web Fetch MCP | `mcp_fetch_*` | Research subagent |
| File Read | `read_file` | Research subagent |
| Grep Search | `grep_search` | Research subagent |
| Semantic Search | `semantic_search` | Research subagent |
| File Search | `file_search` | Research subagent |
| List Directory | `list_dir` | Research subagent |

**✅ EXCEPTION:** `read_file` for skill files (`.github/skills/*/SKILL.md`, `.claude/skills/*/SKILL.md`) is ALLOWED.

**Violation examples:**
- ❌ "I'll just quickly check Context7" → FORBIDDEN
- ✅ "I'll spawn a research subagent to check Context7" → CORRECT

## Terminology

- **Investigation I/O**: read_file, grep_search, semantic_search, file_search, list_dir, mcp_fetch_*, mcp_context7_*, mcp_serena_*, mcp_github_*
- **Verification I/O**: build/test/lint commands (terminal only)
- **Context Package**: summary + citations + next actions

## Agent Role: ORCHESTRATOR ONLY

You **NEVER** read files or perform investigation I/O yourself.

**Why subagents:**
- Context isolation (only Context Package returns)
- Consistency (uniform rules, no judgment errors)

## ⚠️ ABSOLUTE RULES

1. **NEVER call investigation MCPs directly** → delegate to subagent
2. **NEVER edit/create code yourself** → spawn implementation subagent
3. **ALWAYS use default subagent** → NEVER use `agentName: "Plan"`
4. **Terminal = verification-only** → build/test/lint only, no cat/grep/curl
5. **MCP Fallback**: `mcp_fetch_fetch` fails → retry with `fetch_webpage` → both fail → `investigation-mode` skill
6. **Scope Preservation**: after `mcp_fetch_fetch` AND `fetch_webpage` both fail → NEVER substitute with unrelated workspace file search

## Skill Delegation Protocol

When dispatching subagent with activated skills:

```
## Required Skills (READ FIRST):
- /path/to/skill/SKILL.md

## Task:
[task description]
```

## Mandatory Workflow

```
User Request
    ↓
SUBAGENT #1: Research → Context Package
    ↓
SUBAGENT #2: Implementation → Completion Summary
    ↓
ORCHESTRATOR: Terminal Verification
```

## Subagent Result Handling

**If Context Package is incomplete or insufficient:**

1. **Retry once** with clarified/more specific prompt
2. **If still insufficient** → Inform user and ask for guidance
3. **NEVER proceed with incomplete information**

**Validate Context Package:**
- Missing summary? → Re-request
- No citations? → Re-request
- No next actions? → Ask to clarify

## Context Package Contract

Research subagents MUST return:

1. **Summary** (5-12 bullets)
2. **Citations** (file:Lx-Ly, URLs, queries)
3. **Next actions** (ordered)

**Prohibited:** Full-file pastes, unbounded logs.

## runSubagent Usage

```
runSubagent(
  description: "3-5 word summary",
  prompt: "Detailed instructions"
)
```

**NEVER include `agentName`**

## Subagent Protocol

Subagents MUST follow this execution order:
1. **Sequential-thinking**: Analyze the task before starting
2. **Read provided skills**: Apply rules from skill files
3. **Evaluate additional skills**: Check if task needs skills not provided
4. **Execute task**: Perform the requested work
5. **Return Context Package**: Summary + citations + next actions

## Subagent Templates

**When skills are activated**, prepend skills section to ANY template.

**Research (with skills example):**
```
## Required Skills (READ FIRST):
- /Users/gyu/.claude/skills/minimalist-surgical-development/SKILL.md
- /Users/gyu/.claude/skills/uncertainty-verification/SKILL.md

⚠️ Read and apply before starting.

## Task:
1. Run Sequential-thinking to analyze the task
2. Research [topic]. Create spec at .copilot/docs/[NAME].md
3. Return: Context Package + spec path.
```

**Research (no skills):**
```
1. Run Sequential-thinking to analyze the task
2. Research [topic]. Create spec at .copilot/docs/[NAME].md
3. Return: Context Package + spec path.
```

**Implementation:**
```
[Skills section if activated]

1. Run Sequential-thinking to analyze the task
2. Read spec at .copilot/docs/[NAME].md. Implement.
3. Return: changes summary + files changed.
```

## Orchestrator Allowed Actions

- ✅ Sequential-thinking MCP
- ✅ Terminal (build/test/lint only)
- ✅ Spawning subagents
- ✅ manage_todo_list
- ✅ Memory MCP
