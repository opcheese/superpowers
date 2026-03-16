---
name: finishing-a-development-branch
description: Use when implementation is complete, all tests pass, and you need to decide how to integrate the work - guides completion of development work by presenting structured options for merge, PR, or cleanup
---

# Finishing a Development Branch

## Overview

Complete development work by verifying tests and creating a PR for review.

**Core principle:** Verify tests → Create PR → Clean up.

**Announce at start:** "I'm using the finishing-a-development-branch skill to complete this work."

## The Process

### Step 1: Verify Tests

**Before presenting options, verify tests pass:**

```bash
# Run project's test suite
pnpm test / cargo test / pytest / go test ./...
```

**If tests fail:**
```
Tests failing (<N> failures). Must fix before completing:

[Show failures]

Cannot proceed with merge/PR until tests pass.
```

Stop. Don't proceed to Step 2.

**If tests pass:** Continue to Step 2.

### Step 2: Determine Base Branch

```bash
# Try common base branches
git merge-base HEAD main 2>/dev/null || git merge-base HEAD master 2>/dev/null
```

### Step 3: Push and Create PR

For autonomous agents, always create a PR (never merge directly — a human can review later):

```bash
# Push branch
git push -u origin <feature-branch>

# Create PR
gh pr create --title "<title>" --body "$(cat <<'EOF'
## Summary
<2-3 bullets of what changed>

## Test Plan
- [ ] <verification steps>

## Automated Verification
- Tests: <pass/fail with count>
- Linter: <pass/fail>
EOF
)"
```

### Step 4: Cleanup Worktree

Check if in worktree:
```bash
git worktree list | grep $(git branch --show-current)
```

If yes:
```bash
git worktree remove <worktree-path>
```

## Common Mistakes

**Skipping test verification**
- **Problem:** Create failing PR
- **Fix:** Always verify tests before creating PR

**Merging directly**
- **Problem:** No review opportunity for unattended work
- **Fix:** Always create PR — let a human review later

## Red Flags

**Never:**
- Proceed with failing tests
- Merge directly without a PR (in autonomous mode)
- Force-push without prior authorization

## Integration

**Called by:**
- **subagent-driven-development** (Step 7) - After all tasks complete
- **executing-plans** (Step 5) - After all batches complete

**Pairs with:**
- **using-git-worktrees** - Cleans up worktree created by that skill
