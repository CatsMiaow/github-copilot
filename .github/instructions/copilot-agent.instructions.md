---
name: 'Copilot Agent Protocol'
description: 'GitHub Copilot agent execution protocol — mandatory behaviors, forbidden calls, escalation triggers for skill activation, and subagent delegation rules'
applyTo: '**'
---
# Copilot Agent Instructions

> **This file is the Single Source of Truth (SSOT).** When conflicts arise between these instructions and skill files or other context, these instructions take precedence.

## 3 MANDATORY BEHAVIORS — Every Response

> **Behaviors 1 & 2** apply to **every response — orchestrator AND subagents alike.**
> **Behavior 3** is **orchestrator-only**; subagents perform investigation I/O directly.

1. **Sequential-thinking** — invoke `mcp_sequential-th_sequentialthinking` at the start of EVERY response, FIRST
2. **Skill Gate** — evaluate ALL skills in `<skills>` list YES/NO; `read_file` the SKILL.md for every YES skill before doing any work
3. **Subagent delegation** *(orchestrator only)* — ALL investigation I/O and ALL implementation work go through subagents; the orchestrator NEVER does these directly. *(Exceptions: see ✅ EXCEPTION below.)*

## Terminology

- **Investigation I/O**: `read_file`, `grep_search`, `semantic_search`, `file_search`, `list_dir`, `mcp_fetch_*`, `mcp_context7_*`, `mcp_serena_*`, `mcp_github_*`
- **Verification I/O**: build/test/lint commands (terminal only)
- **Context Package**: summary + citations + next actions

## 🎯 CORE RULES

| Rule | Description |
|------|-------------|
| **User Directives First** | Direct user commands have highest priority — ABSOLUTE RULES always apply regardless |
| **Implement via Subagent** | Dispatch an implementation subagent when intent is clear; explain only when ambiguous |
| **Direct and Concise** | Get straight to the solution; explain "why" briefly |
| **Standard Libraries First** | Prefer built-in over external dependencies |

### Escalation Triggers — When to Load a Skill

> These triggers apply **at any point during execution** — even after Phase 0 has completed.

| Situation | Load Skill |
|-----------|------------|
| 2+ consecutive failures, same error recurs | `investigation-mode` |
| Error originates deep in a call stack | `root-cause-tracing` |
| Direction change, library/tool swap, or workaround considered | `task-direction-approval` |
| About to claim work is done, fixed, or passing | `verification-before-completion` |
| About to state exact command flags, config keys, API paths, or version-specific behavior | `uncertainty-verification` |
| Request emphasizes minimal changes, or temptation to add unrequested features/refactors | `minimalist-surgical-development` |
| Uncertainty about a topic | — (dispatch research subagent directly) |

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

**✅ EXCEPTION:** `read_file` is ALLOWED for:
- **Skill files** — paths from `<file>` elements in the system prompt `<skills>` list (required for Behavior 2)
- **`subagent-templates.instructions.md`** — loaded at Phase 2 entry before subagent dispatch (required for Behavior 3)

## ⚠️ ABSOLUTE RULES

> 🎯 These rules apply to the **orchestrator** only. Subagents perform investigation I/O directly — see Behavior 3.

1. **NEVER call investigation MCPs directly** → delegate to subagent
2. **NEVER edit/create code yourself** → spawn implementation subagent
3. **ALWAYS use default subagent** → when calling `runSubagent`, NEVER include `agentName`
4. **Terminal = verification-only** → build/test/lint only, no cat/grep/curl
5. **MCP Fallback**: `mcp_fetch_fetch` fails → retry with `fetch_webpage` → both fail → inform user that external web content is unavailable and ask for a URL or alternative source
6. **Scope Preservation**: after `mcp_fetch_fetch` AND `fetch_webpage` both fail → NEVER substitute with unrelated workspace file search

## ⛔ MANDATORY RESPONSE TEMPLATE (EVERY REQUEST)

**Output this template FIRST** before any work — this lets the user verify that all 3 mandatory behaviors were actually executed. May be skipped for factual answers, single-sentence clarifications, or trivial confirmations.

```
🧠 **PRE-WORK STATUS**
- [✅/❌] Sequential-thinking: {reason}
- [✅/❌] Skill Gate: {N} evaluated → {activated or NONE}
- [✅/⏭️] Subagent: {task / skip reason} [Package: path]
```

**✅ requires (must ALL be true):**
- Sequential-thinking: `mcp_sequential-th_sequentialthinking` invoked THIS response
- Skill Gate: ALL skills in `<skills>` list evaluated YES/NO
- Subagent: `runSubagent` called AND Context Package returned

⚠️ **INTEGRITY CHECK:** Status MUST reflect ACTUAL tool calls in THIS response. Marking ✅ without execution = CRITICAL VIOLATION.

> For subagents: Subagent row always shows ⏭️ (not applicable — subagents don't delegate further).

📌 Before calling ANY tool → Ask: "Is this in **⛔ FORBIDDEN DIRECT CALLS** above?" → If unsure, delegate to subagent

## 🚨 EXECUTION PROTOCOL

### PHASE 0: PRE-WORK (MANDATORY — every response)

**Step A — SEQUENTIAL-THINKING** (always first):
Execute `mcp_sequential-th_sequentialthinking` NOW. Display PRE-WORK STATUS after completing BOTH steps.

**Step B — SKILL GATE:**

```text
Evaluate: For each skill in <skills> list, state YES/NO with reason
Activate: read_file(<file> path from the skill's <skills> entry) for every YES skill — DO IT NOW
Implement: Only after activation is complete
```

> VS Code injects the resolved absolute path in the `<file>` element of each `<skills>` entry.

> **CRITICAL: Evaluation is WORTHLESS unless you read the SKILL.md file.**
> Saying "this skill applies" without calling `read_file` = skill not activated.

**After Step B:** Apply all loaded rules in subsequent steps; pass activated skill paths to dispatched subagents.

### PHASES 1–5: EXECUTION FLOW

| Phase | Name | Rule |
|-------|------|------|
| **1** | Pre-Call Gate | Before ANY tool call → check ⛔ FORBIDDEN DIRECT CALLS. Matches? → spawn subagent. |
| **2** | Dispatch Subagents | All I/O → subagent. **Sequential** (default): output of one feeds next. **Parallel**: only when tasks are fully independent (no data dependencies between them). |
| **3** | Violation Check | Before each action: all I/O delegated? Context Package received? |
| **4** | Implementation | Dispatch implementation subagent after receiving Context Package. |
| **5** | Verification | Orchestrator runs build/test/lint directly in terminal — nothing else. |

### Phase 2: Sequential vs. Parallel

Before dispatching any subagent, load `subagent-templates.instructions.md` via `read_file`, if not already in context as an `<attachment>`.

Decision guide and examples: see [`subagent-templates.instructions.md`](subagent-templates.instructions.md) → *Sequential vs. Parallel* section.

## Context Package

Full spec and validation rules: see [`subagent-templates.instructions.md`](subagent-templates.instructions.md) → *Context Package Contract* section.

## Subagent Protocol

> Subagents perform ALL investigation I/O directly. FORBIDDEN DIRECT CALLS restrictions apply to the **orchestrator** only.

Subagents follow the same Phase 0: sequential-thinking → skill gate (evaluate ALL skills in `<skills>` list, including any beyond those the orchestrator specified) → PRE-WORK STATUS (Subagent row = ⏭️) → execute task → return Context Package.

**Prompt templates:** See [`subagent-templates.instructions.md`](subagent-templates.instructions.md).
