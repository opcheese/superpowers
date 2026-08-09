---
title: Sync fork to upstream v6.2.0
status: complete
last_verified: 2026-08-09
owner: opcheese
area: maintenance
audience: maintainers
---

# Sync fork to upstream v6.2.0

Second run of the recurring sync described in
[2026-07-15-sync-upstream-v6.1.1.md](2026-07-15-sync-upstream-v6.1.1.md).
That document holds the full rationale for the strategy; this one records what
v6.2.0 specifically required.

**Range:** upstream `v6.1.1` → `v6.2.0` (+ two post-release commits), 52 commits.
**Result:** `agents` at `21df1e9`, `main` at `dfaed66`, both pushed to origin.

## Strategy (unchanged)

Re-apply the fork onto a fresh upstream base; never merge 50+ upstream commits
backwards into the fork. Build an integration branch off `upstream/main`, apply
the fork's intent onto upstream's *new* text, then land it with a tree-exact
merge so both parents are preserved and the resulting tree equals the
integration branch exactly:

```bash
git merge -s ours --no-commit <integration-branch>
git read-tree --reset -u <integration-branch>
git commit
git diff --stat <integration-branch> HEAD   # must be empty
```

## What upstream changed

- **SDD lifecycle restructure** — plan-scoped workspace
  (`.superpowers/sdd/<plan>/`, one per plan, deleted at plan end), a ledger that
  names its own plan file, a resume-based fix loop (rounds 1-3 resume the
  original implementer, 4-5 escalate to a fresh implementer on a better model),
  a five-round breaker with explicit adjudication, and a new scoped
  `re-review-prompt.md`.
- **Skills compression sweep** — "Key Principles" / "The Bottom Line" /
  "Real-World Impact" recap sections folded into their points of use or dropped;
  several guard sections converted to rationalization tables.
- **`testing-anti-patterns.md` → `writing-good-tests.md`** — rebuilt as a
  positive catalog with a falsifiability discipline.
- **`find-polluter.sh` fixes** — accepts `./`-prefixed patterns; `**/` now also
  matches files directly under the base directory; honest zero on no matches.
- **Windows SessionStart hook** dispatched via Git Bash.
- **Gemini CLI support restored** (upstream reverted its own removal) — the
  fork's README had dropped it and now carries it again.
- **"We're Hiring" section removed** from the README.

## Agents branch (unattended)

- **SDD** — the automated verification gate moved into the new *Complete the
  task* step: it runs after the review is clean (or findings are parked at the
  cap) and before the completion line; a red gate re-enters the fix loop as a
  finding, and a still-red gate at the cap is load-bearing by definition
  (`BLOCKED` + escalate). Every human touchpoint the rewrite introduced was
  routed to `superpowers:escalation`: pre-flight plan conflicts, plan-mandated
  findings, the breaker's load-bearing stop, and residual findings after the
  final review. Four rationalization rows added.
- **finishing-a-development-branch** — rewritten on upstream's new base rather
  than carrying the old fork file forward, which picked up the
  `WORKTREE_PATH`-captured-before-`cd` fix and the rationalization table. Still
  always-PR and forge-neutral (`gh`/`glab`), but the base branch is now resolved
  from evidence (tracking branch → remote default) instead of asked for, and
  parked findings/escalations are carried into an **Open items** section of the
  PR description — the workspace ledger is deleted at plan end, so the PR is the
  only place they survive.
- **executing-plans, using-git-worktrees, receiving-code-review,
  systematic-debugging, TDD, writing-plans** — escalation over asking,
  re-applied onto the new text.
- **brainstorming** — kept the fork's headless version, with upstream's
  Key-Principles fold applied to it (YAGNI bullet moved into *Exploring
  approaches*).
- **README** — rebuilt from upstream's current README rather than patched, so
  the restored Gemini CLI section and the removed hiring section both came
  through; banner now reads v6.2.0.

## Main branch (interactive)

Lighter set, same re-apply method:

- **SDD** — end-of-run human sign-off, reconciled with upstream's
  continuous-execution guidance and wired into the new *Finish* step (after the
  workspace is deleted, before finishing the branch), plus a flow node and a
  rationalization row.
- **executing-plans** — Step 3 human verification, renumbering Complete
  Development to Step 4.
- **dispatching-parallel-agents** — step 5 human verification.
- **verification-before-completion** — realistic-environment gate (shared with
  `agents`).
- Fork-only skills: `documentation-hygiene`, `end-of-day-report`,
  `end-of-week-report`, `topic-research`. No `escalation` skill on this branch,
  and nothing on `main` references it.
- pnpm examples.

## Execution notes

- **The pnpm change now has a test.** Upstream added
  `tests/systematic-debugging/test-find-polluter.sh`, which stubs an `npm`
  binary on `PATH`. The fork's `pnpm` switch made it fail. The stub now names
  `pnpm` on both branches. Any future fork change to a script under `skills/`
  should be checked against `tests/` before it is called done.
- **`tests/codex/test-package-codex-plugin.sh` fails in this environment on a
  clean `upstream/main` checkout too** — verified by stashing the fork changes
  and re-running. Pre-existing, not caused by the sync.
- **Shared docs are not fork docs.** `git checkout <fork-branch> -- docs/` also
  reverts upstream's edits to shared files; `docs/porting-to-a-new-harness.md`
  and `docs/windows/polyglot-hooks.md` had to be restored from `upstream/main`.
  Copy fork-only doc paths individually instead.
- **Loss check after the tree-exact merge:**
  `git diff --diff-filter=D --name-only <fork-branch> HEAD` returned only
  `testing-anti-patterns.md` on both branches — upstream's own rename. Nothing
  unintended was dropped this time (contrast with the v6.1.1 sync, which
  silently lost the plan doc).
- Deterministic suites pass on both branches; the LLM-driven suites under
  `tests/claude-code/` and `tests/explicit-skill-requests/` were not run.
