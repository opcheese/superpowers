---
name: finishing-a-development-branch
description: Use when implementation is complete, all tests pass, and you need to decide how to integrate the work - guides completion of development work by presenting structured options for merge, PR, or cleanup
---

# Finishing a Development Branch

## Overview

Complete development work by verifying tests and creating a PR for review.

**Core principle:** Verify tests → Detect environment → Create PR → Clean up.

**Announce at start:** "I'm using the finishing-a-development-branch skill to complete this work."

## The Process

### Step 1: Verify Tests

**Before pushing, verify tests pass:**

```bash
# Run project's test suite
pnpm test / cargo test / pytest / go test ./...
```

**If tests fail:**
```
Tests failing (<N> failures). Must fix before completing:

[Show failures]

Cannot proceed with PR until tests pass.
```

Stop. Don't proceed to Step 2.

**If tests pass:** Continue to Step 2.

### Step 2: Detect Environment

**Determine workspace state before pushing:**

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
```

| State | Cleanup |
|-------|---------|
| `GIT_DIR == GIT_COMMON` (normal repo) | No worktree to clean up |
| `GIT_DIR != GIT_COMMON`, named branch | Provenance-based (see Step 4) |
| `GIT_DIR != GIT_COMMON`, detached HEAD | No cleanup (externally managed); push as new branch |

### Step 3: Push and Create PR

For autonomous agents, always create a PR (never merge directly — a human can review later).

If on detached HEAD, first create a named branch:

```bash
git checkout -b <feature-branch>
```

Then push and open the PR:

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

**Do NOT clean up worktree** — it stays alive so a follow-up session can iterate on PR feedback.

### Step 4: Cleanup Workspace (only if PR creation failed or work is being discarded)

**For successful PR creation, skip this step** — preserve the worktree.

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
WORKTREE_PATH=$(git rev-parse --show-toplevel)
```

**If `GIT_DIR == GIT_COMMON`:** Normal repo, no worktree to clean up. Done.

**If worktree path is under `.worktrees/`, `worktrees/`, or `~/.config/superpowers/worktrees/`:** Superpowers created this worktree — we own cleanup.

```bash
MAIN_ROOT=$(git -C "$(git rev-parse --git-common-dir)/.." rev-parse --show-toplevel)
cd "$MAIN_ROOT"
git worktree remove "$WORKTREE_PATH"
git worktree prune  # Self-healing: clean up any stale registrations
```

**Otherwise:** The host environment (harness) owns this workspace. Do NOT remove it. If your platform provides a workspace-exit tool, use it. Otherwise, leave the workspace in place.

## Common Mistakes

**Skipping test verification**
- **Problem:** Create failing PR
- **Fix:** Always verify tests before creating PR

**Merging directly**
- **Problem:** No review opportunity for unattended work
- **Fix:** Always create PR — let a human review later

**Cleaning up worktree after PR**
- **Problem:** Remove worktree needed for PR iteration
- **Fix:** Preserve worktree once PR is open

**Running git worktree remove from inside the worktree**
- **Problem:** Command fails silently when CWD is inside the worktree being removed
- **Fix:** Always `cd` to main repo root before `git worktree remove`

**Cleaning up harness-owned worktrees**
- **Problem:** Removing a worktree the harness created causes phantom state
- **Fix:** Only clean up worktrees under `.worktrees/`, `worktrees/`, or `~/.config/superpowers/worktrees/`

## Red Flags

**Never:**
- Proceed with failing tests
- Merge directly without a PR (in autonomous mode)
- Force-push without prior authorization
- Clean up worktrees you didn't create (provenance check)
- Run `git worktree remove` from inside the worktree

**Always:**
- Verify tests before pushing
- Detect environment before pushing
- Create a PR (never merge locally)
- `cd` to main repo root before worktree removal
- Run `git worktree prune` after removal

## Integration

**Called by:**
- **subagent-driven-development** (Step 7) - After all tasks complete
- **executing-plans** (Step 5) - After all batches complete

**Pairs with:**
- **using-git-worktrees** - Cleans up worktree created by that skill
