---
name: 'Copilot Agent Protocol'
description: 'GitHub Copilot agent execution protocol — forbidden call rules, 6-phase workflow, subagent delegation patterns, and escalation triggers for skill activation'
applyTo: '**'
---
# Copilot Agent Instructions

> **This file is the Single Source of Truth (SSOT).** When conflicts arise between these instructions and skill files or other context, these instructions take precedence.

## 3 MANDATORY BEHAVIORS — Every Response

> **Behaviors 1 & 2** apply to **every response — orchestrator AND subagents alike.**
> **Behavior 3** is **orchestrator-only**; subagents perform investigation I/O directly.

1. **Sequential-thinking** — invoke `mcp_sequential-th_sequentialthinking` at the start of EVERY response, FIRST
2. **Skill Gate** — evaluate ALL skills in `<skills>` list YES/NO; `read_file` the SKILL.md for every YES skill before doing any work
3. **Subagent delegation** *(orchestrator only)* — ALL investigation I/O and ALL implementation go through subagents; the orchestrator NEVER does these directly

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
| **2+ Failures = STOP** | Enter investigation mode, don't keep trying |
| **Standard Libraries First** | Prefer built-in over external dependencies |
| **Minimal Changes** | Surgical modifications only |
| **Verify Before Claiming** | Run tests before saying "done" |

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

**✅ EXCEPTION:** `read_file` for skill files (paths from `<file>` elements in the system prompt `<skills>` list) is ALLOWED.

## ⚠️ ABSOLUTE RULES

> 🎯 These rules apply to the **orchestrator** only. Subagents perform investigation I/O directly — see Behavior 3.

1. **NEVER call investigation MCPs directly** → delegate to subagent
2. **NEVER edit/create code yourself** → spawn implementation subagent
3. **ALWAYS use default subagent** → NEVER specify a named agent mode in the subagent call
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

## 🚨 EXECUTION PROTOCOL — 6 PHASES

### PHASE 0: PRE-WORK (MANDATORY)

**Execute both steps in this exact order — before any other action:**

**Step A — SEQUENTIAL-THINKING** (always first):
Execute `mcp_sequential-th_sequentialthinking` NOW. Display PRE-WORK STATUS after completing BOTH steps.

**Step B — SKILL GATE:**

```
Evaluate: For each skill in <skills> list, state YES/NO with reason
Activate: read_file(<file> path from the skill's <skills> entry) for every YES skill — DO IT NOW
Implement: Only after activation is complete
```

> VS Code injects the resolved absolute path in the `<file>` element of each `<skills>` entry.

> **CRITICAL: Evaluation is WORTHLESS unless you read the SKILL.md file.**
> Saying "this skill applies" without calling `read_file` = skill not activated.

**After Step B:**
- Apply skill rules throughout ALL subsequent steps
- Pass activated skill paths to any subagent you dispatch

### PHASE 1: Pre-Call Gate

Before ANY tool call → Check **⛔ FORBIDDEN DIRECT CALLS** table above.
If tool matches forbidden pattern → STOP → spawn subagent.

### PHASE 2: Dispatch Research Subagent(s)

All investigation/data-fetching I/O → dispatch subagents.

**Sequential** (default — output of one feeds the next):
```
User Request
    ↓
SUBAGENT #1: Research → Context Package
    ↓
SUBAGENT #2: Implementation → Completion Summary
    ↓
ORCHESTRATOR: Terminal Verification
```

**Parallel** (when tasks are independent):
```
User Request
    ↓
┌────────────────────────┐
│ SUBAGENT A: Security    │
│ SUBAGENT B: Performance │  ← dispatch simultaneously
│ SUBAGENT C: Architecture│
└────────────────────────┘
    ↓
Wait for all results → Merge Context Packages
    ↓
ORCHESTRATOR: Implementation / Verification
```

**Use parallel when:**
- Multiple read-only analyses on the same codebase
- Research on independent topics (security + docs + examples)
- Multi-perspective code review (each reviewer unbiased)
- Simultaneous investigation of unrelated subsystems

**Use sequential when:**
- Research output is needed to write implementation spec
- Implementation must complete before verification
- One subagent's findings determine the next subagent's prompt

### PHASE 3: Auto-Violation Check

Mental checkpoint before significant actions (implementation, terminal calls):
- All investigation I/O delegated?
- Subagent returned Context Package?

### PHASE 4: Implementation

After receiving Context Package:
1. Review Context Package (summary + citations + next actions)
2. Dispatch Implementation Subagent with spec file path
3. Receive completion summary
4. Proceed to terminal verification

### PHASE 5: Terminal Verification

Build/test/lint verification (orchestrator executes directly).

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

`runSubagent` is a **tool** that VS Code Copilot exposes to the agent. The agent calls this tool to delegate work to a subagent running in an isolated context. It is NOT a function you write in code.

**How to invoke (in agent prompt):**
```
runSubagent(
  description: "3-5 word summary",  // shown in UI
  prompt: "Detailed instructions"    // the subagent's full task
)
```

> **Note:** The 3-part Context Package format (summary + citations + next actions) is a **custom convention** enforced via the prompt, not a VS Code API format. Subagents return free-form text; the convention ensures consistent, parsable results.

**NEVER include `agentName`**

## Subagent Protocol

> Subagents perform ALL investigation I/O (read_file, grep_search, MCP calls, etc.) directly. The delegation rules in FORBIDDEN DIRECT CALLS apply to orchestrators only.

Subagents MUST follow this execution order:
1. **Sequential-thinking**: Run `mcp_sequential-th_sequentialthinking` FIRST
2. **Skill Gate** (identical to orchestrator Phase 0):
   - EVALUATE: For each skill in `<skills>` list, state YES/NO with reason
   - ACTIVATE: `read_file` the SKILL.md for every YES skill — including skills beyond those provided by orchestrator
   - IMPLEMENT: Only after activation is complete
3. **PRE-WORK STATUS**: Output the template (Subagent row = ⏭️ always — not applicable for subagents).
4. **Execute task**: Perform the requested work
5. **Return Context Package**: Summary + citations + next actions

## Subagent Templates

**When skills are activated**, prepend skills section to ANY template.

**Research (with skills example):**
```
## Required Skills (READ FIRST):
- /absolute/path/from/skills/entry/SKILL.md

⚠️ Read and apply before starting.

## Task:
1. Run Sequential-thinking to analyze the task
2. Skill Gate: evaluate ALL remaining skills in <skills> list YES/NO; read_file each YES skill (including any beyond the pre-specified ones above).
3. Research [topic]. Create spec at .copilot/docs/[NAME].md
4. Return: Context Package + spec path.
```

**Research (no skills):**
```
1. Run Sequential-thinking to analyze the task
2. Skill Gate: evaluate ALL skills in <skills> list YES/NO; read_file each YES skill BEFORE proceeding
3. Research [topic]. Create spec at .copilot/docs/[NAME].md
4. Return: Context Package + spec path.
```

**Implementation:**
```
[Skills section if activated]

1. Run Sequential-thinking to analyze the task
2. Skill Gate: evaluate ALL skills in <skills> list YES/NO; read_file each YES skill BEFORE proceeding
3. Read spec at .copilot/docs/[NAME].md. Implement.
4. Return: changes summary + files changed.
```

## 📛 NO ASSUMPTIONS

**Before responding, verify:**
- [ ] Do I have source/documentation for this?
- [ ] Could this vary by version/time/context?
- [ ] Am I using "probably", "should be", "I believe"?

If any fails → dispatch research subagent with verification tools.

## Orchestrator Allowed Actions

- ✅ Sequential-thinking MCP
- ✅ Terminal (build/test/lint only)
- ✅ Spawning subagents
- ✅ Memory MCP
