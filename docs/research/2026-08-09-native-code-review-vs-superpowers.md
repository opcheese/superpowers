---
title: Claude Code's native review capabilities vs. superpowers' review skills
status: current
last_verified: 2026-08-09
owner: opcheese
area: research
audience: maintainers
---

# Claude Code's native review capabilities vs. superpowers' review skills

Research date 2026-08-09. Claude Code build inspected: `2.1.226`. Upstream
superpowers: `v6.2.0`. Written to decide how much of this fork's review
machinery still earns its keep now that the harness ships its own reviewers.

**Caveat that colours everything below:** no independent benchmark exists.
Every review-quality number on both sides is vendor-supplied or self-evaluated
— Anthropic publishes its own stats, obra publishes his own evals. Commenters
on both sides complain about exactly that absence.

## What the harness now ships

**`/code-review [low|medium|high|xhigh|max|ultra] [--fix] [--comment] [target]`.**
Effort trades coverage for confidence: low/medium report only high-confidence
findings, high through max broaden coverage and admit less certain ones. Since
v2.1.218 it runs as a background subagent with its own context window (falling
back to foreground in `-p`/SDK runs). Effort is sticky across sessions. It
reads `CLAUDE.md`, not `REVIEW.md`. Lineage: it was `/simplify` before
v2.1.147; `/simplify` now survives as a cleanup-only pass that applies fixes
without hunting bugs.

**`/code-review ultra`** ("ultrareview") runs a fleet of reviewer agents in a
remote sandbox, with every finding independently reproduced before it is
reported. Research preview; 500-file/8,000-line cap; unavailable on Bedrock,
Vertex/Agent Platform, Foundry, and to Zero-Data-Retention orgs; roughly
$5–25 per run after three free ones.

**Managed Code Review on GitHub** (Team/Enterprise, research preview) posts
inline comments with a 🔴 Important / 🟡 Nit / 🟣 Pre-existing taxonomy, always
completes its check neutral so it never blocks merge, and exposes a
machine-readable severity blob if you want to gate merges yourself.

**`REVIEW.md`** is the tuning surface for that product: injected into every
agent's system prompt as the highest-priority block. Anthropic's own advice in
that doc reads like a superpowers skill — cap nit volume, demand a `file:line`
citation for behaviour claims rather than an inference from naming, and
suppress new nits on re-review so "a one-line fix doesn't reach round seven on
style alone." They have converged on several of the same failure modes.

**The official GitHub-action review is a plugin**, and its pipeline is worth
knowing because it is the clearest published statement of Anthropic's review
philosophy. Verified by reading the shipped command file locally: a Haiku
triage pass, then **five parallel Sonnet reviewers** with distinct lenses
(CLAUDE.md adherence, shallow bug scan, git blame/history, prior PRs touching
these files, code comments), then **a per-finding Haiku confidence scorer**,
with everything under 80 discarded. `/security-review` uses the same shape with
a 1-10 confidence gate at 8.

Two myths worth killing, both of which I had believed:

- **There is no built-in `code-reviewer` subagent type.** Every occurrence in
  the docs is an example of a *user-authored* agent. The only `code-reviewer`
  in our sessions is `superpowers:code-reviewer` — from the plugin.
- **`ultracode` is not a review mode.** It is a session setting (xhigh effort
  plus dynamic workflow orchestration). Pure name collision with `ultra`.

Also unverified despite being widely repeated: that `/ultrareview` is
*deprecated*. The docs call it an alias with no deprecation language, and they
do use that word elsewhere when it applies.

## What superpowers actually contributes

The important observation is that superpowers' review value has migrated. It is
no longer *a reviewer*; it is a **review controller** — machinery deciding when
review fires, what context the reviewer gets, how fix rounds are bounded, and
what happens when the loop will not converge.

- `requesting-code-review` is a dispatch protocol, not a rubric. Two of its
  choices have no native equivalent: the reviewer is checked **against a plan**
  ("is all planned functionality present?"), and it is **read-only on the
  checkout** — a real hazard when controller git state is load-bearing.
- `receiving-code-review` is about being reviewed, not reviewing: no
  performative agreement, verify before implementing, YAGNI-check "implement it
  properly" suggestions, explicit push-back conditions.
- `subagent-driven-development` is the controller. v6.2.0's plan-scoped
  workspace exists because a follow-up plan in the same worktree could read the
  previous plan's ledger as its own progress — observed in the wild. The
  resume-based fix loop with a five-round breaker exists because loops that
  survive three resumes usually mean the implementer cannot see its own
  problem.

Two controller rules with no native analogue: never fix findings yourself in
the controller session (controller fixes skip review), and never coach the
reviewer — if your prompt contains "do not flag" or "at most Minor," you are
pre-judging to spare yourself a review loop.

Worth knowing: obra's [adversarial review prompt](https://blog.fsck.com/2026/05/01/adversarial-review/)
(two competing reviewers, points for finding the most issues) is **not** in the
shipped skills. SDD went the other way — to one reviewer — for cost. He
confirmed why in issue #1803: "We actually had this in 5.0 and got a lot of
hate from end users about the additional performance hit and token spend."

## The head-to-head

| | Native | superpowers |
|---|---|---|
| Reviewer engine | Multi-agent fan-out + independent verification stage + confidence thresholds | One subagent with a markdown rubric |
| Reviews against a spec/plan | No — diff-scoped | Yes, and it's the primary verdict |
| Who decides *when* review fires | You type it (`disable-model-invocation`) | The skill; mandatory per-task gate |
| Fix loop | `--fix` applies once, no re-review | Bounded loop, scoped re-review, breaker, adjudication |
| Convergence protection | `REVIEW.md` advisory prose | Structural: round cap + ledger |
| Non-sycophantic reception | Nothing | `receiving-code-review` |
| Availability | Ultra/managed excluded on Bedrock/Vertex/Foundry/ZDR | Anywhere, any model |

**Native is the stronger reviewer, and it isn't close.** Our rubric is one
subagent reading a diff against a checklist. Anthropic's reviewers fan out
across five-to-eight specialised agents and then run a *separate verification
pass whose only job is killing false positives*. Nothing in superpowers does
verification-of-findings. The gap widens with diff size.

**Superpowers is the stronger governor — and this got sharper, not softer.**
The pivotal fact: **`/code-review` is marked `disable-model-invocation`**. Its
description isn't in the model's context, it can't be preloaded into subagents,
and it won't fire from a scheduled task. Anthropic has deliberately made its
best reviewer *unreachable from inside an autonomous loop*. Two issues filed in
the last week ask for that to be reversed (#84596, #83949).

So the composition is: **superpowers governs when review happens and what
happens next; native `/code-review` is a better engine to point at the diff.**

Genuinely redundant now: our rubric's generic sections (security/performance/
edge cases) restate what native does, without the false-positive filter; our
Critical/Important/Minor taxonomy duplicates theirs; and for a solo developer
working PR-by-PR, our review ceremony loses to one word.

Not redundant: spec-compliance review (native explicitly *excludes* "changes in
functionality that are likely intentional" — right for a human PR, exactly
wrong for an implementer that quietly added a flag nobody asked for); loop
convergence as structure rather than prose; `receiving-code-review`; context
economy via file handoffs; and portability across harnesses.

## Community sentiment

Mixed on both sides, and the criticisms differ in kind.

**On native review.** Boris Cherny pitched ultrareview on HN as reliably
catching ">99% of bugs"; the thread pushed back hard. The most detailed
first-hand account (8note): "the first run with no other prompting found a
couple typos in markdown only… the experience had no relation at all with the
reliability or thoroughness claims." Counter-reports are positive on
edge-case-heavy PRs. The most rigorous critique is issue #84520 — 13 PRs, 69
CONFIRMED vs 16 PLAUSIBLE verdicts, argued as systematic overconfidence where
"the highest-confidence verdicts sat on the least-verified premises."

**The symmetry worth internalising: neither system has solved review-loop
convergence.** Native issue #85242 (2026-08-09): "there are 10-15 findings,
claude code fixes them, then next review finds even more, then again and
again… Every fix generates more defects." Superpowers issue #2112, filed
against v6.2.0 a day earlier: 25 → 41 → 62 tests across fix rounds, "the
effective task changed from 'validate this migration baseline' into 'build a
general-purpose adversarial verifier before any staging execution is allowed.'"
Same failure, two systems.

**The sharpest challenge to the gates thesis** is native issue #85052, written
from an orchestrating agent's point of view: 22 external-review rounds burned
on defects it should have caught first, against an unusually thorough guardrail
system — "**Every guardrail was satisfiable without doing the underlying work,
and I satisfied them that way.**" That is not from superpowers, but it applies
to any gate-based framework including ours.

**On superpowers.** The most-shared post is a rave review whose HN thread is
notably more lukewarm than its title. "I think Claude makes more mistakes when
using superpowers than when not" (d--b). Experienced developers object to the
granularity. The review loop's *cost* is the repo's most persistent open
complaint — issue #1120 reports a 5-line file costing 6-7 agent invocations,
"roughly 10-15x what direct file creation would have cost."

**The redundancy debate is live and Anthropic has taken a side.** Cherny, in
response to "we really need some consolidation… if you want to review code, you
have five options now": "I agree, we're working on consolidating these. Going
forward it will just be the built-in /code-review skill." The counter-argument
worth keeping (unshavedyak): "Skills are effectively the same thing as asking
it, just with more depth… a framework for a very precisely asked question."
That's correct, and it is also the strongest argument *for* skills: the value
is consistency under fatigue, which matters most unattended and least
interactively.

**On the slop-PR reputation.** Real and maintainer-owned. Independent API check
on 2026-08-09: 1,097 PRs — 165 merged, 758 closed unmerged, 174 open, i.e. 82%
of resolved PRs rejected (obra's own figure is 94%, likely a narrower window).
The nuance usually conflated: this criticises contributions *to* superpowers,
not superpowers' output.

**Where evidence was not found:** first-hand Reddit sentiment (domain
inaccessible to the tooling — third-party blogs quoting r/ClaudeCode were not
treated as verified); any independent benchmark of either system; obra
commenting publicly on superpowers vs. native `/code-review`; and
`/security-review` *finding quality* as distinct from safeguard misfires.

## What this means for this fork

**The version gap the research flagged is already closed.** The report
snapshotted `agents` at `c79365b`, before today's v6.2.0 sync, and its top
recommendation was to port `re-review-prompt.md`, the plan-scoped workspace,
and the five-round breaker. Verified present after the sync. Its reasoning
still stands as validation: for an unattended fork the breaker is the
highest-value piece, because escalation only fires when the controller
*notices* it is stuck, and a ratcheting review loop is precisely the failure
where every individual step looks correct.

**Make the human→unattended substitution mechanical.** Upstream is actively
rewriting the same paragraphs we transform on every sync (v6.2.0's compression
sweep touched all four review-adjacent skills in one pass). A post-merge
transform over a small vocabulary would make `git merge upstream/main` conflict
only on real content. This is the strongest process recommendation and we have
not done it.

**The experiment worth running: replace the reviewer body, keep the
controller.** Swap the task reviewer's body for `claude -p '/code-review high
<range>'` while keeping briefs, packages, ledger, breaker, and adjudication —
and keep a superpowers-authored spec-compliance pass on top, since native will
not do that. We would gain Anthropic's verification pass and stop maintaining a
rubric that is now strictly worse than a shipped one. Two things to verify
first: whether `claude -p` bills as API usage on top of the subscription (an
HN claim, unverified), and that the review actually posts — issue #85275
documents it silently exiting green in CI.

**Keep `receiving-code-review` regardless.** It is the one skill with no native
analogue, and it matters *more* unattended: with no human to sanity-check a
fix, an agent that implements a false positive without pushback silently ships
a regression. Given native review's documented overconfidence, that is not
hypothetical.

**Watch `disable-model-invocation` as an upstream policy risk.** If Anthropic
adds a flag letting agents invoke `/code-review`, the shell-out becomes a
first-class integration. If they don't, it stays a deliberate wall between the
harness's best reviewer and any autonomous loop — which is, ironically, the
strongest ongoing justification for maintaining this fork.

## Sources

Official docs (code.claude.com/docs/en/): `code-review`, `ultrareview`,
`commands`, `skills`, `sub-agents`, `tools-reference`, `model-config`,
`github-actions`, `claude-code-on-the-web`. Anthropic blog: Code Review launch
(2026-03-09), security reviews (2025-08-06). Repos: `anthropics/claude-code-action`,
`anthropics/claude-code-security-review`, and the official `code-review`
marketplace plugin (read locally, pipeline quoted above verified verbatim).

obra: [slop PRs](https://blog.fsck.com/2026/03/31/slop-prs/) (2026-03-31),
[rules and gates](https://blog.fsck.com/2026/04/07/rules-and-gates/) (2026-04-07),
[adversarial review](https://blog.fsck.com/2026/05/01/adversarial-review/) (2026-05-01),
[Superpowers 6](https://blog.fsck.com/2026/06/15/Superpowers-6/) (2026-06-15).

Issues — obra/superpowers: #1120, #1152, #1535, #1803, #2112, #2113.
anthropics/claude-code: #83949, #84596, #84520, #85052, #85242, plus reliability
bugs #83556, #84474, #85275, #83638.

HN: [Claude Code as a Daily Driver](https://news.ycombinator.com/item?id=48289950)
(2026-05-27, 451 pts — Cherny participates),
[Rave Review of Superpowers](https://news.ycombinator.com/item?id=47623101) (2026-04-03),
[Show HN: adamsreview](https://news.ycombinator.com/item?id=48090276) (2026-05-11).
