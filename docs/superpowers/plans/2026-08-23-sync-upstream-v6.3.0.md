---
title: Sync fork to upstream v6.3.0
status: complete
last_verified: 2026-08-23
owner: opcheese
area: maintenance
audience: maintainers
---

# Sync fork to upstream v6.3.0

Third run of the recurring sync. See
[2026-07-15-sync-upstream-v6.1.1.md](2026-07-15-sync-upstream-v6.1.1.md)
for the full rationale and
[2026-08-09-sync-upstream-v6.2.0.md](2026-08-09-sync-upstream-v6.2.0.md)
for the previous run.

**Range:** upstream `v6.2.0` → `v6.3.0`, one squashed release commit.
**Result:** `agents` at `a3a66d7`, `main` at `4a506cd`.

## Strategy — a real merge this time

The previous two syncs rebuilt the fork on a fresh upstream base because
upstream had moved 50+ commits and rewritten whole skills. v6.3.0 is one
commit touching 40 files, of which only six overlap the fork's edits on
`agents` and three on `main`. A plain `git merge upstream/main` with
hand-resolved conflicts is correct here and keeps honest history. Reserve
the re-apply strategy for releases that rewrite skills wholesale.

## What upstream changed

- **SDD "Rulings, not stalls"** — the controller now decides conflicts,
  ambiguities, plan defects and cap questions itself, recording each as a
  `Ruling:` ledger line, instead of asking. Only four things stop it: an
  irreversible or destructive operation, a security-sensitive action, a
  side effect outside the worktree, and a plan where every path forward is
  a guess.
- **SDD pre-flight scan is now a table**, not a verdict — one row per task
  pair sharing a file or interface, one row per task for self-consistency.
- **No-subagents contract** for implementers and reviewers, in all three
  prompt templates: a worker-spawned reviewer duplicates a review seat the
  controller already dispatches.
- **Batch small same-shape work** into one dispatch instead of one
  subagent per trivial task; bounded waiting on dispatched children.
- **Brainstorming three-path router** — spike / bounded / architectural,
  with a Red Flags table guarding the classification.
- **Devin CLI and Hermes Agent support**, README table of contents,
  worktree-removal refusal handling, `writing-plans` Spec field.

## The interesting part: upstream converged on this fork's premise

"Rulings, not stalls" is the `agents` branch's thesis, arrived at
independently and tuned better than our version of it. The fork had been
routing every human touchpoint to `superpowers:escalation`; upstream now
routes the same touchpoints to a recorded ruling and keeps going.

A ruling beats an escalation wherever a decision is actually available:
escalation logs the question and moves to other work, leaving *this* task
stalled, which is the outcome the fork exists to avoid. So `agents` adopts
the rulings model wholesale and narrows escalation to the cases upstream
still stops on:

- Of upstream's four stops, two are structurally moot here — this fork
  never merges and never pushes to a shared branch (always-PR), and
  destructive operations are out of scope for a plan task.
- The two that remain — a security-sensitive action, and a plan so broken
  every path forward is a guess — are what `superpowers:escalation` now
  covers, and nothing else. A new rationalization row guards the obvious
  abuse: reaching for escalation to avoid deciding is the stall the
  rulings discipline exists to prevent.
- Upstream's "Rulings I made" final message assumes a reader. Unattended
  there is none, and the workspace is deleted at plan end, so the rulings
  list goes into the PR description's **Open items** section instead —
  `finishing-a-development-branch` already carried parked findings and
  escalations there.

## Agents branch

- **subagent-driven-development** — rulings model adopted; escalation
  narrowed as above; flow nodes updated; rationalization table merged.
- **finishing-a-development-branch** — upstream's worktree-removal refusal
  block adapted: never `--force`, commit what belongs to the branch, and
  if anything remains, leave the worktree standing, list the files under
  Open items and escalate. Open items now also carries rulings.
- **brainstorming** — upstream's three-path router adapted headless. The
  classification is *recorded with the fact that decided it* rather than
  announced for override, because nobody is here to override it; doubt
  therefore resolves upward more readily. Each path keeps a gate: spec
  review via subagent for architectural, TDD plus the task review for
  bounded, nothing for a spike whose output is an answer. Two fork-specific
  Red Flags rows replace the approval-specific ones. Visual companion
  stays deleted.
- **README** — banner to v6.3.0, upstream's new TOC with fork entries
  added and the telemetry entry removed, Devin CLI and Hermes Agent
  sections absorbed.

## Main branch

Only SDD conflicted.

- **subagent-driven-development** — rulings model adopted as written. The
  per-task native-review checkpoint is re-applied *on top* of it and named
  in Continuous execution as an addition to upstream's four stops. The
  distinction that makes it survive the new discipline: a ruling is a
  decision the controller could have made, and running a
  `disable-model-invocation` slash command is not one. The end-of-run
  human sign-off is kept and now also presents the run's rulings.
- Dropped a duplicated rationalization row that predates this sync.

## Execution notes

- **Three test suites fail in this environment on a clean `upstream/main`
  checkout too**, for missing host tools — `tests/version-bump` needs
  `yq`, `tests/writing-skills/test-render-graphs.sh` needs graphviz
  `dot`, `tests/codex/test-package-codex-plugin.sh` needs `zip`. Verified
  by running them in a detached worktree at `upstream/main`. Not
  fork-caused. Installing the three tools would make the sync's
  verification meaningfully stronger next time.
- `tests/brainstorm-server/` on `main` has one failing and one hanging
  suite; those files are untouched by v6.3.0 and by this sync.
- Nothing was dropped by either merge (`git diff --diff-filter=D` against
  each pre-merge tip is empty apart from `visual-companion.md`, which this
  fork deletes on `agents` by design).
- **`main` carries no fork banner in its README** — its only README delta
  was removing the "We're Hiring" section, which upstream then removed
  itself, so `main`'s README is now byte-identical to upstream's. Nothing
  there tells a reader that `main` has fork-only skills or a native-review
  checkpoint. Worth deciding separately.
