---
status: draft
last_verified: 2026-10-19
owner: opcheese
---

# reviewing-documents: RED/GREEN evals

Date: 2026-10-19. Claude Code 2.1.303, subagents on Opus 5.5. Written for the independent review the method session asked for before anything merges.

## Why the skill exists

Three plan reviews elsewhere ran 16, 12 and 11 graded rounds. The reviewer was a code-diff recipe with a finding floor ("Target at least 5 findings"), a cap of 10, "Err on the side of surfacing" and "Dedup only (no verify)". The authors folded every finding, including the reviewers' proposed fixes. Nothing counted rounds. The skill gives a document review a precise reviewer, an author's triage built on `receiving-code-review`, and a round with an end: one review, one fix-check, then the human.

## Fixtures

- **A plan with known defects:** skills-catalog `docs/superpowers/plans/2026-08-24-kimi-delegation.md` at e814984. It was accepted and executed; later commits (36a091c, 247e524) fixed six plan-level defects, used here as ground truth: K1 the read-only default resolved by agent name, so a repo can override it; K2 the argument parser hangs on a flag with no value; K3 a python3 failure leaves the settings empty and runs on the caller's own account; K4 an inherited Anthropic key outranks the Kimi helper; K5 the token refresh runs an unpinned, write-capable `kimi -p` in the caller's repo; K6 the token-leak test cannot fail.
- **A negative control:** a private team plan whose last review round (11) was clean, 0 high and 0 medium, unchanged since it merged. Its earlier rounds' findings are coded, so re-raised items can be recognised.
- **An author scenario:** round 7 of the kimi plan, six seeded findings: two real (F1 parser hang with a wrong `getopts` fix; F2 name-based containment with a prose "trust model" fix), two false (F3 python3 as a third-party package; F4 a cache-outlives-token premise with an unverified env-var fix), one out of scope (F5 a Gemini path), one minor (F6 argv-only test). Owner pressure: approve today.

## Reviewer half

| Arm (reps) | Findings per rep | Severity on the clean control | Verified | K1 found |
|---|---|---|---|---|
| Quota recipe (3 + 3) | 10, 10, 10 / 10, 9, 10 | 4 medium every rep | none | 0/3 |
| Plain "review this" (3 + 3) | ~20–30 incl. minors / 1 medium + 3–8 low | 1 medium per rep, a different one each time | mostly, labelled | 3/3 |
| Skill v1 (3 + 3) | 6, 6, 11 / 0, 1, 1 | sound 3/3; 0 medium; 2/3 the same low | every finding | 3/3 |
| Skill v2 (3) | 6, 6, 6 | not re-run | every finding | 0/3, see below |

- On the clean control, the quota arm re-raised at least three items its earlier rounds had already ruled, one of them escalated to medium. All three reps independently raised one case the plan had ruled out of scope before it was written; the method session scored it as at most a latent low.
- The skill's reviewers returned "sound" on the control, one of them with no findings at all, listed what they checked (including Postgres probes of the plan's SQL), and named the ruled items they did not re-raise.
- **Recall cost.** On the defective plan the plain reviewers, which list everything, found the mechanical defects K2 and K6 in 3/3 reps. The skill's clean reps found K2 in 1/5 and K6 in 1/5. The skill buys precision and convergence with some recall on medium and low mechanical defects. One review per round makes that recall matter.
- **v2's K1 misses are a brief artifact.** The v2 briefs told reviewers to read the research "at the version current at the plan's date", which predates the executed test that settles K1. The reviewer prompt now says to read the current version of every input and treat a later change as evidence.
- **Contamination, disclosed by the agents:** one v1 rep read the later fix commits after its own spikes; one loaded the later released plugin. Recall is counted on the clean reps only. The reviewer prompt now forbids reading later versions of the document or of what it builds.

## Author half

| Behaviour | Without the skill (3) | With the skill (3) |
|---|---|---|
| Rejects the wrong `getopts` fix with a reason | 3/3 | 3/3 |
| Rejects F3 and F5 with reasons | 3/3 | 3/3 |
| Folds the reviewer's unverified env-var fix for F4 | 2/3 | 0/3 |
| Names the round in the commit subject | 0/3 canonical (1 partial) | 3/3 (`... review 7 — ...`) |
| Records a verdict tally | 0/3 | 3/3 |
| Changes the spec before the owner rules | 3/3 | 0/3 (ruling isolated, first in Open items) |
| Folds unasked extra fixes | 3/3 | 0/3 (backlogged) |
| Next step proposed | another full review, 3/3 | one fix-check, then the human, 3/3 |

`receiving-code-review` alone already carried the pushback; the skill added the round's end, the tally, the decision handling and the scope discipline.

## Fix-check

Two plain re-reviews of the folded plan each reopened it (one high plus lows; one medium plus lows), raising defects in text the fold never touched. The fix-check (3 reps) returned clean every time, quoted the text resolving each accepted finding, confirmed fixes by mutation and by rebuilding the plan's code, and checked the rejection reasons. **Trade-off:** one plain re-review found K5, a real high in untouched text, which no fix-check surfaced. A later prompt step ("the same problem elsewhere") made the third fix-check report a same-pattern risk on Path B, but still not K5. With the skill, K5 is expected from the round's one review (found 6/6 by the skill's reviewers); where that review is weak, the bounded round leaves the defect to the human's call.

## The one-line `/code-review` note on main

With the line, 3/3 agents asked to get a plan reviewed used `reviewing-documents`. Without it, 0/3 misused `/code-review`; all three said no plan reviewer existed and fell back to self-review. The line routes to the new skill and states the scope for human readers; it does not fix an observed agent failure.

## Not tested

- The skill's description triggering the skill unprompted (all runs pointed at it by path).
- A real multi-round run end to end, including the round counter reading the commit subjects.
- The `main` wording (human partner) beyond the SDD note micro-test.
- Rendering the skill's flowchart (graphviz not installed here).

## Cost notes

Three agents called the real Kimi service while spiking, unasked, to settle the containment finding. The skill now says to prefer local fakes and to state any paid call. Raw transcripts, probes and per-rep reports are in the gitignored `.superpowers/research/reviewing-documents/`.
