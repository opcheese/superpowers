---
title: Sync fork to upstream v7.0.0
status: complete
last_verified: 2026-10-12
owner: opcheese
area: maintenance
audience: maintainers
---

# Sync fork to upstream v7.0.0

Fourth run of the recurring sync. Previous run:
[2026-08-23-sync-upstream-v6.3.0.md](2026-08-23-sync-upstream-v6.3.0.md),
which links the full rationale.

**Range:** upstream `v6.3.0` → `v7.0.0`: three squashed release commits
(v6.4.1, v6.4.2, v7.0.0), 106 files.
**Result:** `main` at `522b723` (pushed). `agents` merge at `17819c2` on
`sync/agents-v7.0.0`.

## Strategy

A plain `git merge upstream/main` with hand-resolved conflicts. Three
commits is small enough for honest history. On `main` only
`executing-plans` conflicted; on `agents`, `brainstorming`,
`writing-plans`, `executing-plans` and the visual-companion files
conflicted. For each conflicted skill, upstream's new version is the base
and the fork's intent is re-applied on top of it. The old fork text was not
merged line by line.

## What upstream changed

- **v6.4.1:** `executing-plans` rebuilt as Native inline execution, with
  the same workspace, ledger, rulings and stopping rules as SDD, and one
  whole-branch review at the end. New `diagnosing-superpowers` skill.
  Plans gain a Review Focus section. TDD now means the project's whole
  suite is green. OpenCode 2.0, Muse and Qwen Code support.
- **v6.4.2:** `writing-plans` records decisions instead of transcribing
  code. `plan-document-reviewer-prompt.md` and `CLAUDE.md` removed;
  `AGENTS.md` is now the canonical contributor guide.
- **v7.0.0:** `brainstorming` rebuilt as a conversation: one open question
  per message, a playback of each part as it's understood, sizing the
  work, and a builder check that replaces the spec-document reviewer.
  Native Antigravity manifest. Windows hook hardening.

## Main branch

- **executing-plans:** take upstream's rebuilt skill and re-apply main's
  two human gates, the same way they sit on SDD. The first is the
  per-task native-review checkpoint (`/code-review` is
  `disable-model-invocation`, so only the human can run it). The second is
  the end-of-run human verification. Both appear in the process diagram,
  in Continuous execution and in the rationalization table.
- Brainstorming, writing-plans and SDD merged cleanly. Upstream v7
  brainstorming is interactive, which is what `main` wants.
- New `npm` examples changed to `pnpm`.

## Agents branch

- **brainstorming:** kept the fork's headless three-path skill. Upstream
  v7 is built around a live conversation; a headless port would be a
  different skill under the same name. Adopted the one v7 piece that
  survives without a human, the **builder check**: a fresh subagent reads
  the spec as its builder and lists what it would have to guess. Its three
  piles are handled like this:
  - *answered* goes into the spec;
  - *minor* is settled and marked as the agent's call;
  - *theirs* becomes a `Ruling:` in the spec and reaches the PR's Open
    items.

  Escalation is reserved for questions with no intent to rule from. This
  replaces the spec-document reviewer, whose prompt upstream deleted.
  Visual companion and its tests stay deleted.
- **writing-plans:** upstream's leaner plans taken as written. The
  handoff, which asks the human to review the plan and choose a method, is
  replaced by an automatic choice. It defaults to subagent-driven, because
  unattended, the per-task reviewer is the only second pair of eyes before
  the PR. Native applies without a subagent tool or when the request names
  it. The choice and its reason are recorded.
- **executing-plans:** upstream's rebuild had already converged on this
  fork (continuous run, rulings, ledger, a final review instead of human
  check-ins). Re-applied:
  - the four stops route to escalation, as in SDD;
  - always work on a feature branch;
  - rulings, escalations and deferred minors go to the PR's Open items
    instead of a final message;
  - pnpm in examples.
- README banner: synced version, brainstorming row, new plan-handoff and
  rulings rows.

## Not changed, on purpose

- `diagnosing-superpowers` keeps its approval gates on both branches. A
  human invokes it, and the gates protect privacy before a bundle or a
  GitHub issue leaves the machine.
- `references/claude-code-tools.md` (opt-in nested SDD orchestrator) is
  left as upstream wrote it; opt-in only.

## Verification

- Offline suites pass on both branches: antigravity, executing-plans
  scripts, sdd-workspace, worktree path policy, codex-plugin-sync, codex
  marketplace, devin, diagnosing-superpowers structure, hooks (session
  start and Windows run-hook), kimi, shell-lint, find-polluter.
- Three suites still fail for missing host tools (`yq`, graphviz `dot`,
  `zip`), as on a clean upstream checkout.
- `test-worktree-native-preference.sh` is a live-model eval. Its RED
  baseline no longer reproduces on the current model, which is unrelated
  to this sync.
- The only deletions relative to each pre-merge tip are upstream's own:
  `CLAUDE.md`, `spec-document-reviewer-prompt.md` and
  `plan-document-reviewer-prompt.md`.
- **No RED/GREEN skill evals were run for the re-applied changes.**
  Pending items:
  - On `agents`: the brainstorming builder-check sorting, especially
    "theirs" becoming a ruling and not an escalation.
  - On `agents`: the writing-plans automatic handoff.
  - On `main`: the executing-plans native-review gate under upstream's new
    continuous-execution wording.
