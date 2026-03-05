---
name: task-direction-approval
description: Use when considering switching libraries/tools, changing architecture, replacing automation with manual workarounds, adding new dependencies, or changing approach while keeping the same library - explains root cause, offers 2-3 options with trade-offs, and requests explicit user choice
---

# Task Direction Approval

## Overview

This skill prevents unauthorized direction changes by forcing explicit user consent before deviating from the original requirements.

## Use when...

- Switching from the original tech/library to alternatives
- Replacing an automated approach with a manual workaround
- Changing architecture or design patterns
- Delivering different results than requested

## Symptoms / keywords

- "direction change", "can't use X", "let's switch to Y", "workaround", "manual", "different approach", "alternative library", "rewrite", "change architecture"

## Communication protocol (template)

**❌ Wrong Response:**

"Code generator X failed, so I'll define types manually instead."

**✅ Correct Response:**

"Code generator X failed due to authentication error. Options available:
1. Add authentication headers
2. Try different endpoint
3. Download schema/spec file directly
Which approach would you prefer?"

## Minimal workflow

1. Explain the failure root cause clearly (what failed, why it failed, evidence).
2. Present 2-3 viable options with trade-offs (speed, risk, maintenance, correctness).
3. Ask for explicit user choice.
4. Only proceed after approval.
5. **If user rejects all options**: Present the constraints clearly and ask the user to define the new direction — do not proceed unilaterally.

## Exceptions (no approval needed)

- Fix obvious typos
- Correct configuration mistakes
- User says "any method is fine"

## Notes

- Use `investigation-mode` skill when there are repeated failures or the situation is unclear.
