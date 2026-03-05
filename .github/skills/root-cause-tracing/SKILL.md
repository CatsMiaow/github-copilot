---
name: root-cause-tracing
description: Use when errors occur deep in execution and you need to trace back to find the original trigger, or when invalid data propagates through multiple layers to its origin - systematically traces bugs backward through call stack, adding instrumentation when needed, to identify source of incorrect behavior
---

# Root Cause Tracing

## Overview

Bugs often manifest deep in the call stack (git init in wrong directory, file created in wrong location, database opened with wrong path). Your instinct is to fix where the error appears, but that's treating a symptom.

**Core principle:** Trace backward through the call chain until you find the original trigger, then fix at the source.

## When to Use

**Use when:**
- Error happens deep in execution (not at entry point)
- Stack trace shows long call chain
- Unclear where invalid data originated
- Need to find which test/code triggers the problem

> Usually invoked from `investigation-mode`. After identifying root cause, if the fix requires changing approach or tools, use `task-direction-approval`.

## Symptoms / keywords

- "wrong directory", "wrong path", "unexpected value", "shouldn't be empty/null"
- "`X` was called with `Y`" — trace where `Y` came from
- Error thrown inside a library or framework, not in your own code
- "It works in isolation but fails in the full test suite"
- Stack trace passes through 3+ call frames before reaching your code

## The Tracing Process

### 1. Observe the Symptom
```
Error: git init failed in /Users/jesse/project/packages/core
```

### 2. Find Immediate Cause
**What code directly causes this?**
Locate the line that executes the failing operation. What arguments did it receive?

### 3. Ask: What Called This?
Trace one level up: what function/method invoked the immediate cause, and with what value?

```
# example — class names will differ per project
WorktreeManager.createWorktree(projectDir, sessionId)
  → called by Session.initializeWorkspace()
  → called by Session.create()
  → called by test setup
```

(Use IDE "Find Usages", stack trace, or grep to follow the chain.)

### 4. Keep Tracing Up
**What value was passed?**
- `projectDir = ''` (empty string!)
- Empty string as `cwd` resolved to the current working directory
- That's the source code directory!

### 5. Find Original Trigger
**Where did empty string come from?**
Search where the variable is assigned. Keep tracing until you reach the assignment that introduced the bad value.

## Adding Stack Traces

When you can't trace manually, add instrumentation:

Add a log statement **immediately before** the problematic operation. Capture:
- The argument being passed (the suspicious value)
- Current working directory
- Relevant environment variable
- Full stack trace

Write to **stderr** so it isn't swallowed by test loggers.
- JS/TS: `console.error()` | Python: `print(..., file=sys.stderr)` | Rust: `eprintln!()` | Go: `fmt.Fprintln(os.Stderr, ...)` | Java: `System.err.println()`

**Run and capture (via subagent):**
```bash
# (subagent) <run-tests> 2>&1 | grep '<symptom-keyword>'
# e.g. npm test / pytest / cargo test / go test ./...
```

**Analyze stack traces:**
- Look for test file names
- Find the line number triggering the call
- Identify the pattern (same test? same parameter?)

## Finding Which Test Causes Pollution

If something appears during tests but you don't know which test, use bisection:

1. **Binary search over test files**: Run half the test suite at a time until the polluter is isolated
2. **Dispatch a subagent** to run individual test files and identify the first one that causes the artifact
3. Return a cited Context Package with the polluter file and line

```bash
# (subagent) run tests one-by-one and stop at first polluter
# e.g. npm test -- --testPathPattern="<file>" / pytest <file> / cargo test <name>
<run-tests> --filter="<single-file-or-test>" 2>&1
```

## Key Principle

**NEVER fix just where the error appears.** Always trace backward until you find the true origin of the invalid value.

```
SYMPTOM → Immediate Cause → Caller → Caller → ... → Root Cause → FIX HERE
```

Add validation at each layer so the bug becomes impossible to reintroduce.

## Stack Trace Tips

**In tests:** Write to stderr, not a logger that may be suppressed
**Before operation:** Log before the dangerous operation, not after it fails
**Include context:** Directory, cwd, environment variables, timestamps
**Capture stack:** Use your language's native stack trace facility
- JS/TS: `new Error().stack` | Python: `traceback.format_exc()` | Rust: `std::backtrace::Backtrace` | Go: `runtime/debug.Stack()`

## Real-World Impact

From a real debugging session:
- Found root cause through 5-level trace
- Fixed at source (validation at origin)
- Added 4 layers of defense
- All tests passed, zero pollution
