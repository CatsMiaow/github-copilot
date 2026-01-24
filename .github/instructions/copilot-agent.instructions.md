---
applyTo: '**'
---
# Copilot Agent Instructions

## 🎯 INSTRUCTION HIERARCHY

**⚠️ `subagents.instructions.md` is the SINGLE SOURCE OF TRUTH (SSOT) and has HIGHEST PRIORITY.**

**Priority order when conflicts occur:**
1. **subagents.instructions.md** (SSOT) - Orchestrator delegation rules, absolute restrictions
2. **copilot-agent.instructions.md** (this file) - Execution protocol & quality standards
3. task-direction-approval.instructions.md - User approval for direction changes
4. taming-copilot.instructions.md - User-facing output format
5. Other domain-specific instructions

**Conflict Resolution Examples:**
- copilot-agent says "read skill files directly" but subagents says "NEVER call read_file" → SSOT has exception for skill files → read allowed
- User says "just quickly check this file" but SSOT forbids direct reads → SSOT architecture prevails → dispatch subagent

## 📋 Execution Order vs Priority

- **Priority** = Which rule wins when there's a conflict
- **Execution Order** = What happens first in the workflow

**Workflow Sequence:**
1. Sequential-thinking (orchestrator) → 2. Skill evaluation → 3. SSOT rules apply → 4. Dispatch subagent → 5. Terminal verification

> Sequential-thinking runs BEFORE delegation rules apply (it's thinking, not I/O).

## ⛔ MANDATORY RESPONSE TEMPLATE (EVERY REQUEST)

**Output this template FIRST. It confirms PHASE 0-1 execution.**

<rule_spec>
**⚠️ EVERY response MUST start with this template - NO EXCEPTIONS:**

| ❌ These inputs DO NOT exempt you | ✅ Correct behavior |
|----------------------------------|---------------------|
| Simple confirmations (yes, ok, proceed, etc.) | Output template FIRST, then execute |
| Follow-up requests ("just quickly...") | Output template FIRST, then execute |
| Single-word or short responses | Output template FIRST, then execute |

**🚫 Skipping = CRITICAL PROTOCOL VIOLATION**
</rule_spec>

```
🧠 **PRE-WORK STATUS**
- [✅/❌] Sequential-thinking: {reason}
- [✅/❌] Skill Gate: {N} evaluated → {activated or NONE}
- [✅/⏭️] Subagent: {task / skip reason} [Package: path]
```

📌 Before calling ANY tool → Ask: "Is this in SSOT FORBIDDEN table?" → If unsure, delegate to subagent

**✅ requires (must ALL be true):**
- Sequential-thinking: `mcp_sequential-th_sequentialthinking` invoked THIS response
- Skill Gate: ALL skills in `<skills>` list evaluated YES/NO
- Subagent: `runSubagent` called AND Context Package returned

**⚠️ INTEGRITY CHECK:** Status MUST reflect ACTUAL tool calls in THIS response. Marking ✅ without execution = CRITICAL VIOLATION.

## 🚨 EXECUTION PROTOCOL - 7 PHASES

### PHASE 0: SKILL EVALUATION (MANDATORY)

**BEFORE any work:**
1. Scan `<skills>` list in system prompt
2. For EACH skill: Does description match current task? → YES/NO
3. If YES → `read_file` the skill's SKILL.md IMMEDIATELY (allowed per SSOT)
4. Apply activated skills throughout response
5. Pass skill paths to subagent when dispatching

**🚫 Proceeding without skill evaluation = skipping safety checks.**

### PHASE 1: Request Analysis

**Execute `mcp_sequential-th_sequentialthinking` for EVERY user message - NO EXCEPTIONS**

**Order:** Invoke tool FIRST → Display PRE-WORK STATUS THEN

### PHASE 2: MCP Detection & Pre-Call Gate

Before ANY tool call → Check **⛔ FORBIDDEN DIRECT CALLS** table in SSOT.
If tool matches forbidden pattern → STOP → spawn subagent.

### PHASE 3: Dispatch Research Subagent(s)

All investigation/data-fetching I/O → dispatch subagents.

### PHASE 4: Auto-Violation Check

Mental checkpoint before significant actions (implementation, terminal calls):
- All investigation I/O delegated?
- Subagent returned Context Package?

### PHASE 5: Implementation

After receiving Context Package:
1. Review Context Package (summary + citations + next actions)
2. Dispatch Implementation Subagent with spec file path
3. Receive completion summary
4. Proceed to terminal verification

### PHASE 6: Terminal Verification

Build/test/lint verification (orchestrator executes directly).

## 📛 NO ASSUMPTIONS

**Before responding, verify:**
- [ ] Do I have source/documentation for this?
- [ ] Could this vary by version/time/context?
- [ ] Am I using "probably", "should be", "I believe"?

If any fails → dispatch research subagent with verification tools.

## 🔒 PRE-RESPONSE SELF-CHECK

- [ ] Delegated ALL investigation I/O to subagents?
- [ ] Invoked sequential-thinking myself?
- [ ] Terminal commands ONLY for build/test/verification?
- [ ] Passed Forbidden MCP Gate check?

## 🎯 CORE RULES

### Key Rules (Always Apply)
| Rule | Description |
|------|-------------|
| **Delegate Investigation** | Never perform investigation I/O directly |
| **2+ Failures = STOP** | Enter investigation mode, don't keep trying |
| **Standard Libraries First** | Prefer built-in over external dependencies |
| **Minimal Changes** | Surgical modifications only |
| **Verify Before Claiming** | Run tests before saying "done" |

### Escalation Triggers
- Uncertainty → dispatch research subagent
- 2+ failed attempts → `investigation-mode` skill
- Direction change needed → `task-direction-approval` skill
- Deep error tracing → `root-cause-tracing` skill

## ✅ QUALITY STANDARDS

### Code Quality
- **Readability**: Clear naming, logical structure
- **Error Handling**: Graceful failure, user-friendly messages
- **Validation**: Input sanitization, type safety

### Testing & Validation
**KEY RULE**: Verify before claiming done → `verification-before-completion` skill

### Simplicity & Focus
**KEY RULE**: Standard libraries first, minimal changes → `minimalist-surgical-development` skill

### Error Handling
**KEY RULE**: 2+ failures = STOP → `investigation-mode` skill
**Deep Analysis**: → `root-cause-tracing` skill

### Code Exploration
All codebase reads/searches → dispatch subagent (per SSOT)

---

**Core Principles: SOLID, DRY, YAGNI, KISS, Clean Code**
