---
name: finishing-a-development-branch
description: Use when implementation is complete, all tests pass, and you need to integrate the work — autonomous operation always opens a pull request for later human review
---

# Finishing a Development Branch

## Overview

**Core principle:** Verify tests → Detect environment → Open a PR → Clean up.

This fork runs unattended. There is no integration menu and no one to answer
it: the safe default for autonomous work is always a pull request, never a
local merge. A human reviews and merges later.

**Announce at start:** "I'm using the finishing-a-development-branch skill to complete this work."

## Step 1: Verify Tests

Run the project's full test suite (`pnpm test` / `cargo test` / `pytest` / `go test ./...`).

**If tests fail:** the work is not complete. Attempt a fix once; if the suite
is still red, do not open a PR — invoke **superpowers:escalation** to log the
failing tests with their output, then continue with other independent work.

```
Tests failing (<N> failures) at branch completion:

[Show failures]

Cannot open a PR until tests pass — escalated.
```

**If tests pass:** continue to Step 2.

## Step 2: Detect Environment

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
# Capture now, while still inside the workspace — later steps may change
# directory before cleanup needs this value
WORKTREE_PATH=$(git rev-parse --show-toplevel)
```

| State | Push | Cleanup |
|-------|------|---------|
| `GIT_DIR == GIT_COMMON` (normal repo) | Push the current branch | No worktree to clean up |
| `GIT_DIR != GIT_COMMON`, named branch | Push the current branch | Preserved for PR iteration |
| `GIT_DIR != GIT_COMMON`, detached HEAD | Name a branch first, then push | Externally managed — leave in place |

## Step 3: Determine Base Branch

The base branch is whatever this work forked from — usually named in the
plan, the conversation, or the branch's upstream. Resolve it from evidence,
in this order, rather than asking:

```bash
git rev-parse --abbrev-ref --symbolic-full-name @{u} 2>/dev/null   # tracking branch
git symbolic-ref --short refs/remotes/origin/HEAD 2>/dev/null      # remote default branch
```

Opening a PR against the wrong base is cheap to correct and visible in the
PR itself — unlike a wrong merge. If the evidence genuinely conflicts, pick
the remote's default branch, say which base you chose and why in the PR
description, and escalate the ambiguity (see superpowers:escalation).

## Step 4: Push and Open the PR

Always open a PR. Never merge locally in unattended operation — the PR *is*
the review gate that a human would otherwise have provided.

If on a detached HEAD, name a branch first:

```bash
git checkout -b <feature-branch>
```

Then push:

```bash
git push -u origin <feature-branch>
# From a detached HEAD you did not name locally:
# git push origin HEAD:refs/heads/<new-branch>
```

Open the pull/merge request against the base branch with the forge's own
tooling — do **not** assume GitHub. Detect the forge from the remote:

- **GitHub** (`gh` available): `gh pr create --base <base> --title "<title>" --body "<body>"`
- **GitLab** (`glab` available): `glab mr create --target-branch <base> --title "<title>" --description "<body>"`
- **Otherwise:** the push output prints a compare/MR creation URL — report
  that URL so a human can open it in the web UI.

Follow the repo's PR template and conventions if present. Otherwise use this
body, adapting field names to the forge:

```
## Summary
<2-3 bullets of what changed>

## Test Plan
- [ ] <verification steps>

## Automated Verification
- Tests: <pass/fail with count>
- Linter: <pass/fail>

## Open items
<parked findings, escalations, and residual review findings — or "none">
```

Carry any parked findings, escalations, and residual final-review findings
into that last section. An unattended run's PR is the only place a human
learns what the loop could not resolve; a finding that lives only in a
deleted workspace ledger is a silent discard.

Report the PR/MR URL.

**Keep the worktree** — follow-up sessions iterate on PR feedback there.

## Step 5: Cleanup Workspace

Cleanup runs only when the PR could not be opened and the branch is being
abandoned. A successful PR always preserves the worktree.

Run from the main repo root — worktree removal fails from inside the
worktree — using the values captured in Step 2:

```bash
MAIN_ROOT=$(git -C "$(git rev-parse --git-common-dir)/.." rev-parse --show-toplevel)
cd "$MAIN_ROOT"
```

**If `GIT_DIR == GIT_COMMON`:** Normal repo, no worktree to clean up. Done.

**If `WORKTREE_PATH` is under `.worktrees/` or `worktrees/`:** Superpowers
created this worktree — we own cleanup:

```bash
git worktree remove "$WORKTREE_PATH"
git worktree prune  # Self-healing: clean up any stale registrations
```

**Otherwise:** The host environment owns this workspace — leave it in place.
If your platform provides a workspace-exit tool, use it.

## Quick Reference

| Situation | Merge | Push | Open PR | Keep Worktree |
|-----------|-------|------|---------|---------------|
| Tests green | - | yes | yes | yes |
| Tests red after one fix attempt | - | - | - | yes (escalate) |
| PR creation failed | - | yes | report URL | yes |

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "Tests passed earlier this session" | Run the suite on the tree you are about to push. A green run only proves the tree it ran on. |
| "Nobody is watching, merging is faster" | The PR is the review gate that replaces the human sign-off. Unattended work never merges itself. |
| "The change is trivial, it can go straight to the base branch" | Triviality is a claim no reviewer got to check. Open the PR. |
| "The PR is up, so the worktree is clutter now" | PR feedback gets fixed in that worktree. It stays until the work lands. |
| "This other worktree looks stale — I'll clean it too" | Clean up only worktrees under `.worktrees/` or `worktrees/`. Everything else belongs to the host. |
| "The base branch is obviously main" | Resolve it from the tracking branch or the remote's default, and say which you chose in the PR. |
| "The push was rejected — force-push will fix it" | A rejected push means the remote moved. Investigate and rebase; never force-push unattended. |
| "The parked findings are in the ledger, that's enough" | The workspace is deleted at the end of the plan. Findings that matter go in the PR description. |

## Integration

**Called by:**
- **subagent-driven-development** — after the final whole-branch review is clean
- **executing-plans** — after all tasks complete and verification passes

**Pairs with:**
- **using-git-worktrees** — cleans up the worktree created by that skill
