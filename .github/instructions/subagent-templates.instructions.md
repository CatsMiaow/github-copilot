---
name: 'Subagent Prompt Templates'
description: 'Context Package contract, Sequential vs. Parallel decision guide, and copy-paste prompt templates for dispatching research and implementation subagents'
---
# Subagent Prompt Templates

## Context Package Contract

Research subagents MUST return:

1. **Summary** (5–12 bullets)
2. **Citations** (file:Lx-Ly, URLs, queries)
3. **Next actions** (ordered)

**Prohibited:** Full-file pastes, unbounded logs.

**If incomplete or insufficient:** Orchestrator retries once with a clearer prompt. If still missing → inform user, stop. NEVER proceed with an incomplete Context Package.

## Sequential vs. Parallel

**Sequential is the default.** Use parallel only when subagent tasks have no data dependency between them.

| Situation | Mode | Example |
| --------- | ---- | ------- |
| Output of A is input to B | Sequential | Research → Implementation → Verification |
| One finding shapes the next prompt | Sequential | Root cause research → Fix design |
| Must complete before claiming done | Sequential | Implementation → Verification |
| Independent read-only analyses | Parallel | Security audit + Performance audit |
| Same data, different perspectives | Parallel | Multi-reviewer code review |
| Unrelated subsystems | Parallel | Frontend bug + Backend schema investigation |

## runSubagent Tool

`runSubagent` is a VS Code Copilot **tool** — call it directly, do NOT write it as code.

```text
runSubagent(
  description: "3-5 word summary",  // shown in UI
  prompt: "Detailed instructions"    // the subagent's full task
)
```

> The 3-part Context Package format (summary + citations + next actions) is a custom convention enforced via the prompt, not a VS Code API format.

## Research subagent (no skills)

```text
1. Run Sequential-thinking to analyze the task.
2. Skill Gate: evaluate ALL skills in <skills> list YES/NO; read_file each YES skill BEFORE proceeding.
3. Research [topic]. Create spec at .copilot/docs/[NAME].md.
4. Return: Context Package + spec path.
```

## Research subagent (with pre-activated skills)

```text
## Required Skills (READ FIRST):
- /absolute/path/from/skills/entry/SKILL.md

⚠️ Read and apply before starting.

## Task:
1. Run Sequential-thinking to analyze the task.
2. Skill Gate: evaluate ALL remaining skills in <skills> list YES/NO; read_file each YES skill (including those beyond the pre-specified list above).
3. Research [topic]. Create spec at .copilot/docs/[NAME].md.
4. Return: Context Package + spec path.
```

## Implementation subagent

```text
[Prepend skills section if any were activated by orchestrator]

1. Run Sequential-thinking to analyze the task.
2. Skill Gate: evaluate ALL skills in <skills> list YES/NO; read_file each YES skill BEFORE proceeding.
3. Read spec at .copilot/docs/[NAME].md. Implement.
4. Return: changes summary + files changed.
```
