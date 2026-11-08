---
status: current
last-verified: 2026-11-08
owner: Kirill (opcheese)
---

# Skill-set consistency audit, 2026-11-08

## Method

Kimi `k3.1-pro`, read-only, ran once per branch over every file under `skills/`. It was asked
to report only contradictions, duplicates, drift and dead references, each with a verbatim
quote. A local script checked links mechanically. Every finding was then classified by
hand, against the files and against upstream v7.0.2. Raw outputs are in
`docs/research/evals/2026-11-08-consistency-audit/`.

Kimi reported 14 findings on main and 15 on agents. Dead links: none real; the only hits
were placeholders in example text in `anthropic-best-practices.md`.

## Fixed (real, and either ours or a one-line upstream bug)

- **F1, both branches. requesting-code-review's round cap** ("record what's open and finish")
  contradicted its own red flag ("never proceed with unfixed Important"). Now an Important
  finding still open at the cap needs a written ruling. The cap also defers to SDD and
  executing-plans, which have their own caps. Introduced today.
- **F2, both. `BASE_SHA=HEAD~1`** in requesting-code-review drops every commit but the last;
  SDD forbids it. Changed to the merge base. Upstream line.
- **F3, both. end-of-day and end-of-week reports** looked only in `docs/plans/`; writing-plans
  saves to `docs/superpowers/plans/`. They now look in both.
- **F4, main. SDD deleted the workspace before human verification.** A requested change would
  then have no ledger. Verification now comes first. Introduced in today's sync.
- **F5, agents. Escalation fired at 2 or more failed fixes;** systematic-debugging allows 3
  before questioning the architecture. Aligned to 3.
- **F6, agents. receiving-code-review said "Clarify anything unclear FIRST"** with nobody to
  ask. Now: rule on it, or escalate. Upstream line the fork missed.

## Not fixed: real, but cost more than they're worth or need a decision

- **The rulings-to-PR hand-off is written in three places on agents** (executing-plans, SDD
  final-review, finishing). The copies are consistent today. If they drift, make finishing
  the owner and have the others point to it.
- **On main, executing-plans writes the completion line before native review** (the
  `task-done` script writes it), while SDD writes it after. Minor; the script owns the order.
- **diagnosing-superpowers on agents** stops to wait for a human. It is human-invoked by
  nature, and stopping is a safe outcome.

## Upstream text, left alone (changing it adds sync cost; we never PR upstream)

- writing-skills' description rule ("when, not what") versus Anthropic's best-practices doc
  ("what and when"). This difference is deliberate in this project.
- writing-skills' "test every edit" versus "don't test pure reference skills".
- TDD's "every function has a test" versus writing-good-tests' exemption for trivial functions.
- executing-plans' task-done runs the brief's test command, not the full suite.
- The executing-plans preflight is narrower than SDD's.
- The implementer prompt says "TDD if the task says to".
- codex-tools cites finishing "Step 1" for environment detection; it is Step 2.
- final-review.md names `references/fix-loop.md` as prose from inside `references/`.
- "merged" versus "committed" wording in the two finish sections.

## False positives

- The revert-to-prove-a-regression-test step versus TDD: these cover different moments, so
  they don't conflict.
- The reviewer no-spawn rule repeated in three prompt templates: each template is meant to be
  self-contained.
- The writing-plans header versus documentation-hygiene frontmatter: both fit in one file.
- The finishing fallback that prints a PR URL: a safe path when no CLI is available, not a
  contradiction.

## Takeaway

Of 29 findings, 6 needed a fix and 4 of those came from the fork, 2 of them from today's
changes. The fork's additions are where new inconsistencies come from. Run this audit after
every sync, and after any session that edits more than one skill.
