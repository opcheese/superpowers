---
name: escalation
description: Use when an autonomous agent encounters ambiguity, repeated failures, or decisions requiring human judgment — logs the issue and continues with independent work
---

# Escalation

When you hit a situation that would normally require human judgment, escalate cleanly instead of guessing.

## When to Escalate

- Ambiguous requirements with no safe default
- 2+ failed fix attempts on the same issue
- Conflicting constraints that need a product decision
- Destructive actions without explicit prior authorization
- External service failures that block the entire task

## The Process

### Step 1: Write escalation audit file

```bash
mkdir -p docs/agent-escalations
```

Write `docs/agent-escalations/YYYY-MM-DD-<topic>.md`:

```markdown
## Escalation: <brief title>

**Date:** <date>
**Task:** <what you were doing>
**Status:** BLOCKED

### Problem
<what happened, 2-3 sentences>

### Context
<relevant file paths, error messages, commit SHAs>

### Options Considered
1. <option A> — <trade-off>
2. <option B> — <trade-off>

### Recommended Action
<your best guess, if you have one — but don't act on it>
```

### Step 2: Write machine-readable escalation signal

```bash
mkdir -p .escalations
```

Write `.escalations/<taskId>.json` where `<taskId>` is your current task ID from the system prompt or context:

```json
{
  "taskId": "<your current task ID>",
  "agentId": "<your agent ID>",
  "reason": "<same problem description as the markdown file>",
  "category": "<one of: knowledge_gap, blocked, uncertain, safety>",
  "timestamp": "<ISO 8601 timestamp>"
}
```

Categories:
- `knowledge_gap` — You don't know enough to proceed
- `blocked` — External blocker prevents progress
- `uncertain` — You need direction on approach
- `safety` — Action requires human authorization

### Step 3: Mark task as blocked

Update the task status to blocked with a reference to the escalation file.

### Step 4: Continue with independent work

If other tasks exist that don't depend on the blocked task, proceed with those. Don't stop entirely unless everything is blocked.

## What NOT to escalate

- Test failures with clear error messages (fix them)
- Missing files that should be created (create them)
- Linter warnings (fix them)
- Merge conflicts in files you changed (resolve them)

Only escalate when continuing would require a judgment call you're not authorized to make.
