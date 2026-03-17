---
name: investigation-mode
description: Use when there are 2+ consecutive failures, repeated errors, or when the same failure recurs after a new fix attempt - pauses implementation, switches to root-cause investigation, gathers evidence, and resumes only after a verified plan and options
---

# Investigation Mode

## Overview

This skill provides a hard stop and a repeatable workflow when progress stalls or errors repeat. It prevents “random walk” fixes and forces evidence-first debugging.

## Use when...

- There are **2+ consecutive failures/errors** in the same feature or approach
- The same error comes back after “fixes”
- There is temptation to “just workaround” or change approach silently
- The problem statement is unclear and implementation would be guesswork

## Symptoms / keywords

- “stuck”, “still failing”, “same error”, “again”, “flaky”, “intermittent”, “can’t reproduce”, “works on my machine”
- “quick workaround”, “let’s just do it manually”, “skip validation”, “good enough”
- CI-only failures, nondeterministic tests, repeating build/compile/lint errors

## On activation

**Announce immediately:** `"🔍 **INVESTIGATION MODE**: Pausing to research root cause"`

When investigation reveals a direction change is needed, use the `task-direction-approval` skill before proceeding.

## How to execute

1. **Freeze changes**: stop making further edits that are not evidence-driven.
2. **Capture evidence**: record the exact error text, stack traces, logs, and minimal repro steps.
3. **Constrain scope**: isolate the smallest failing unit (single test, single endpoint, single build step).
4. **Run root cause analysis**:
   - **Read `root-cause-tracing` skill** (use `read_file` on its path from the `<skills>` list) for systematic isolation techniques.
   - **Read `uncertainty-verification` skill** when the fix depends on exact tool/library behavior.
5. **Propose options**: Use `task-direction-approval` skill when considering an alternative approach.
6. **Test First**: write comprehensive tests that reproduce the root cause before resuming implementation.
7. **Resume** only after tests are green.
8. **Escalate**: If root cause remains unclear after completing steps 1–5, share the captured evidence with the user and ask for guidance — do not resume implementation blindly.

## Common mistakes

- Continuing to code while the failure mode is not understood
- Changing direction silently instead of asking for approval
- “Fixing” by adding retries/timeouts without evidence
