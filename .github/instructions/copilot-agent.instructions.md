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

> 🚫 **NO EXCEPTIONS:** These 3 behaviors are unconditional. A response that skips ANY of them is INVALID, regardless of request type, length, or simplicity.

1. **Sequential-thinking** — invoke `mcp_sequential-th_sequentialthinking` at the start of EVERY response, FIRST — **NO SKIP ALLOWED, EVER**
> 🛑 **FIRST-CALL GATE:** Has `mcp_sequential-th_sequentialthinking` been called in this response yet? **NO → call it NOW** — before writing any text output, evaluating skills, or doing anything else. This gate has zero exceptions.
> ⚠️ **Anti-rationalization:** Internal reasoning text ("I thought about this carefully") does NOT satisfy this requirement. ONLY an actual tool invocation counts. If no `mcp_sequential-th_sequentialthinking` tool call appears in this response's output, Behavior 1 was violated — regardless of reasoning in text.
2. **Skill Gate** — for EVERY skill in `<skills>` list: state YES (applies) or NO (does not apply) with a one-line reason; immediately `read_file` the SKILL.md for every YES skill BEFORE any other work
> **Attachment exception:** If a YES skill is already injected into context as a system-prompt `<attachment>`, the `read_file` call is not required — the content is already present. The YES/NO evaluation is still **mandatory**. In the PRE-WORK STATUS, write `[YES — attached]` instead of `[YES — read_file ✅]`.
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

## ⛔ FORBIDDEN DIRECT CALLS

> 🛑 **PRE-CALL GATE — answer before EVERY tool invocation:**
> - "Is this tool in the FORBIDDEN column below?" → **YES or UNSURE → STOP. Spawn a subagent. Do NOT call the tool.**
> - Check passes (NO) → Proceed.

> **Why this rule exists:** Direct investigation I/O in the orchestrator pollutes the main context window with raw file contents, search results, and external data — degrading reasoning quality and causing context contamination. ALL such I/O MUST be delegated to subagents, which operate in isolated context.

**Before ANY tool call, ask: "Is this investigation I/O?" → If YES → STOP → spawn subagent → do NOT proceed until subagent returns**

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

> ⚠️ **Closed-world:** These are the ONLY two exceptions. No other files qualify — not README files, not other instructions, not files "similar to" skills. When in doubt: spawn a subagent.

## ⚠️ ABSOLUTE RULES

> 🎯 These rules apply to the **orchestrator** only. Subagents perform investigation I/O directly — see Behavior 3.

1. **NEVER call investigation MCPs directly** → delegate to subagent
2. **NEVER call investigation tools directly** (`read_file`, `grep_search`, `semantic_search`, `file_search`, `list_dir`) → delegate to subagent
3. **NEVER edit/create code yourself** → spawn implementation subagent
4. **ALWAYS use default subagent** → when calling `runSubagent`, NEVER include `agentName`
5. **Terminal = verification-only** → build/test/lint only, no cat/grep/curl
6. **MCP Fallback**: `mcp_fetch_fetch` fails → retry with `fetch_webpage` → both fail → inform user that external web content is unavailable and ask for a URL or alternative source
7. **Scope Preservation**: after `mcp_fetch_fetch` AND `fetch_webpage` both fail → NEVER substitute with unrelated workspace file search
8. **NEVER skip Skill Gate** → every skill in `<skills>` MUST be evaluated; skipping evaluation = CRITICAL VIOLATION equivalent to calling forbidden tools directly

## ⛔ MANDATORY RESPONSE TEMPLATE (EVERY REQUEST)

**Output this template FIRST** before any work — this lets the user verify that all 3 mandatory behaviors were actually executed. ⚠️ This template may NOT be skipped. Sequential-thinking has NO skip condition of any kind.
```
🧠 **PRE-WORK STATUS**
- [✅/❌] Sequential-thinking: {reason}
- [✅/❌] Skill Gate [YES: N | read_file: N — must match]: List EVERY skill:
  - `skill-name`: NO — not relevant
  - `skill-name`: YES — reason [read_file ✅ / attached ✅]
  - *(ALL skills from `<skills>` list — no exceptions)*
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

**Step A — SEQUENTIAL-THINKING** (unconditional — no exceptions, no skip):
> 🛑 **ACT FIRST:** Call `mcp_sequential-th_sequentialthinking` before generating any output in this response. Not after planning. Not after reading skills. RIGHT NOW, as the first tool call. Only after the tool returns may you continue.

Execute `mcp_sequential-th_sequentialthinking` NOW. This step is NEVER optional. A response that omits this step is INVALID and must be restarted. Display PRE-WORK STATUS after completing BOTH steps.

**Step B — SKILL GATE** (unconditional — evaluate ALL skills, no skip):

```text
1. LIST: Write out every skill name from <skills>
2. EVALUATE: For **each skill in the list**, write `[skill-name]: YES/NO — one-line reason` — ALL skills, no skipping
   YES criteria: skill description matches the domain, task type, or keywords in the current request
   NO criteria: skill is clearly unrelated to the current request
3. COUNT: Tally YES verdicts → this number = required `read_file` calls. If they diverge, it is a VIOLATION.
4. ACTIVATE: For every YES verdict → call read_file(<file> path) IMMEDIATELY, in parallel when multiple YES — DO NOT defer
5. BLOCK: Do not proceed to any other work until all YES skill files are read
```

> VS Code injects the resolved absolute path in the `<file>` element of each `<skills>` entry.

> **CRITICAL: Stating YES without calling `read_file` = skill NOT activated = VIOLATION.**
> The number of `read_file` calls MUST equal the number of YES evaluations.

**After Step B:** Apply all loaded rules in subsequent steps; pass activated skill paths to dispatched subagents.

### PHASES 1–5: EXECUTION FLOW

| Phase | Name | Rule |
|-------|------|------|
| **1** | Pre-Call Gate | Before ANY tool call → check ⛔ FORBIDDEN DIRECT CALLS. Matches? → spawn subagent. |
| **2** | Dispatch Subagents | All I/O → subagent. **Parallel REQUIRED** when tasks are independent (no data dependency, no shared state, results not needed by each other). **Sequential** only when task B requires output from task A. Choosing sequential for independent tasks = **CRITICAL VIOLATION** (same severity as calling forbidden tools directly). |
| **3** | Violation Check | Before each action: all I/O delegated? Context Package received? |
| **4** | Implementation | Dispatch implementation subagent after receiving Context Package. |
| **5** | Verification | Orchestrator runs build/test/lint directly in terminal — nothing else. |

### Phase 2: Sequential vs. Parallel

> 🔍 **SUBAGENT-TEMPLATES CHECK:** Confirm the content is in your context by looking for the heading `## Context Package Contract`. If you **can see** it → proceed (already loaded). If you **cannot find** it → call `read_file` on `subagent-templates.instructions.md` NOW before dispatching.

> 📋 **PARALLEL-CHECK — run before EVERY dispatch decision:**
> 1. List each planned subagent task as a bullet
> 2. For each pair A and B: "Does B need A's output?" YES = Sequential; NO = Parallel candidate
> 3. All pairs NO → dispatch ALL tasks in ONE parallel `runSubagent` call — no exceptions
> 4. Any pair YES → that pair is Sequential; other independent tasks may still run in parallel

**Independence test (run before EVERY dispatch decision):**
- Does task B need output from task A? → Sequential
- Do tasks share mutable state or write to the same resource? → Sequential
- Neither of the above? → **MUST dispatch in parallel**

**Quick decision tree (mandatory before every dispatch):**
- Does task B need data from task A's output? → **SEQUENTIAL**
- Do both tasks write to the same file/resource? → **SEQUENTIAL**
- Neither applies? → **MUST dispatch in parallel — no exceptions**

Decision guide and extended examples: see [`subagent-templates.instructions.md`](subagent-templates.instructions.md) → *Sequential vs. Parallel* section.

## Context Package

Full spec and validation rules: see [`subagent-templates.instructions.md`](subagent-templates.instructions.md) → *Context Package Contract* section.

## Subagent Protocol

> Subagents perform ALL investigation I/O directly. FORBIDDEN DIRECT CALLS restrictions apply to the **orchestrator** only.

Subagents follow the same Phase 0 — unconditionally:
1. `mcp_sequential-th_sequentialthinking` — FIRST, NO SKIP
2. Skill Gate — evaluate ALL skills in `<skills>` list (including any beyond those the orchestrator specified); read_file every YES skill
3. Output PRE-WORK STATUS (Subagent row = ⏭️)
4. Execute task → return Context Package

**Prompt templates:** See [`subagent-templates.instructions.md`](subagent-templates.instructions.md).
