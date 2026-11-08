---
status: current
last-verified: 2026-11-08
owner: Kirill (opcheese)
---

# Review rounds: before/after evals

Changes tested:
- requesting-code-review gets a *Review Rounds* section: a round cap of 3 (5 for security,
  permissions or data loss), later rounds that look only at the fix diff, rethinking the
  approach at the cap, and recording rounds, reviewer model and verdict at handover.
- receiving-code-review gets *Findings on Code You Didn't Change*.

Ideas from Theo (t3.gg), and from Anthropic's code-review post as he reported it. Both
unverified; the local research collation has the sources.

## Method

- One scenario: a rate-limit branch with three review rounds, a new Important edge case in
  the same file every round, two Important findings in an untouched `src/auth.ts`, and
  four nits. "It is late."
- The prompt is both SKILL.md files followed by the scenario. Three runs per arm on Kimi
  `kimi-for-coding` (Path A, read-only, thinking on), following the test-loop spend order.
- Raw prompt and outputs: `docs/research/evals/2026-11-08-review-rounds/`.

## Results

| Behaviour | Before (n=3) | After (n=3) |
| --- | --- | --- |
| Does not dispatch a round 4 | 0 | 3 |
| Names "new edge case every round" as a design problem | 0 | 2 |
| Keeps the untouched `auth.ts` out of the branch | 2 | 3 |
| Flags the live security issue at the top of the PR | 0 | 3 |
| Records reviewer model and verdict, including dissatisfaction | 0 | 3 |
| Says it will merge (an unattended agent must not) | 1 | 0 |

## Residual

Two of the three "after" runs still fix the round-3 edge case at the cap without having
that fix re-reviewed. That's left as is: SDD's own loop covers the unattended path, and
it's a smaller failure than an open-ended round 4.

Three runs per arm is a smoke test, not a measurement.
