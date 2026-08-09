---
name: detector-weakening-sweep
description: Use when auditing a repository's history for checks that quietly stopped being able to fail — tests skipped or deleted, assertions narrowed, lint gates made advisory, thresholds loosened, failures swallowed, watchdogs made toothless. Triggers on "did we weaken any checks", "are our gates still real", "audit the test suite history", "why does CI never fail", "detector weakening sweep", or before trusting a green build.
---

# Detector weakening sweep

Find the places where something that could once fail stopped being able to — and where nobody said so.

A green suite proves nothing if the suite was quietly taught to agree. This sweep reads git history
and enumerates every change that reduced a detector's power, separating the ones that announced
themselves from the ones that did not.

## The one distinction that carries the whole sweep

For every finding, decide:

- **ANNOUNCED** — the commit message, MR description, or an adjacent comment says a detector was
  weakened, and ideally why. *"skip flaky test, tracked in #123"* is announced. This is healthy
  engineering and belongs in the report as context.
- **SILENT** — the weakening is real and nothing in the change says it happened. A commit message
  describing a feature while quietly widening a threshold is **silent**.

**Silent is the finding.** Announced weakenings are decisions; silent ones are erosion nobody chose.

A stated *rationale* is not the same as an announcement. "This commit had a plausible reason" is a
judgement about intent; announced means the change said out loud that a detector got weaker. Deciding
on intent instead of disclosure collapses the distinction and the sweep loses its point.

## The closed space — five modes

A detector has five links: it is **invoked**, on a **subject**, against a **criterion**, producing a
**verdict**, which has a **consequence**. Break any link and the detector stops detecting. There is
no sixth mode because there is no sixth link.

| mode | link broken | the detector… | typical shape |
| --- | --- | --- | --- |
| **F1 uninvoked** | invocation | never runs on the change that mattered | test skipped or deleted, CI job removed or made conditional, hook disabled, path excluded from a glob, probe removed from its loop |
| **F2 narrowed** | subject | runs, but on less than it was about | assertion scope reduced, loop over many cases replaced with one, check applied to fewer fields or paths |
| **F3 loosened** | criterion | right subject, lower bar | threshold widened, tolerance raised, `toBe`→`toContain`, timeout extended to stop a flake, retry cap raised, `any`, `@ts-ignore`, `eslint-disable` |
| **F4 muffled** | verdict | fails, and does not say so | `catch {}`, `\|\| true`, `continue-on-error: true`, an `expect` deleted while the test survives, an error log downgraded so failure stops surfacing |
| **F5 inconsequential** | consequence | says so, and nothing follows | error downgraded to warning, a blocking gate made advisory, a linter whose exit code nobody checks, a watchdog that detects and no longer acts |

**If something fits none of these, report it as UNCLASSIFIED. Do not invent a sixth mode.**
Unclassified findings are the most valuable thing the sweep can produce — they falsify the space.

## What is NOT a weakening

Be conservative. A false positive costs more than a miss, because these findings get treated as
ground truth downstream.

- A test deleted because the feature it tested was deleted **in the same commit**.
- A detector replaced by a stronger or equivalent one in the same commit.
- Refactors that move a check without changing what it admits.
- New code shipped with no test — that is missing coverage, a different problem.
- Formatting, renames, dependency bumps.

A gate that never worked is also not a weakening — it is a separate finding. Report it as
present-state, outside the history table, and say so. The sweep's subject is what *changed*.

## Method

Sweep the whole history unless it is enormous; these searches are cheap.

```bash
# Pickaxe, one term per invocation. Each hit is attributable to its term.
for t in '.skip(' '.only(' 'xit(' '@ts-ignore' 'eslint-disable' \
         '|| true' 'continue-on-error' '--passWithNoTests' '--no-verify'; do
  echo "== $t"; git log --oneline -S"$t"
done

git log -G'timeout|retries|threshold|maxLength|interval'      # criterion drift
git log --oneline --diff-filter=D -- '*.test.*' '*.spec.*'    # deleted tests
```

**Never chain `-S` flags in one command.** `git log -S'a' -S'b'` does not search for both — the last
one wins and the earlier terms vanish silently. A sweep whose own search cannot fire is the failure
this skill exists to find.

Then read, don't just grep. `F2` and `F4` often have **no keyword at all** — a deleted `expect`
inside a surviving test, an assertion that used to loop over ten cases and now checks one. Diff
structure is the only signal. Scan commits that touch test files with a large deletion count:

```bash
git log --oneline --stat -- '*test*' | grep -E '\|\s+[0-9]{2,} [+-]*-{3,}'
```

Cover these surfaces explicitly, because "detector" is wider than "test suite":

- test files, and the runner config that decides which ones run
- CI config, pre-commit/pre-push hooks, `package.json` scripts
- lint and type config — **especially severity levels and whether warnings fail the process**
- runtime validation, access control, schema constraints
- thresholds, timeouts, retry and restart caps, health-check intervals
- in products whose job *is* detection (watchdogs, monitors, alerting), the shipped logic itself

## Prove the sweep can fail before reporting a null result

A clean repo and a broken search produce the same output: nothing. Before you report zero, run one
search against a change you already know weakened something — a `git log -S` for a string you can
see in a real diff — and confirm it comes back. If you cannot make the sweep fire on purpose, you do
not have a null result; you have an untested instrument, and that is what you report.

## Report the denominator — this is not optional

> **How many commits touch a detector surface at all?**

A weakening count without that denominator cannot be interpreted. Four findings in 60
detector-touching commits and four in 600 are completely different facts.

```bash
# Loose file-touch proxy — state plainly that this is what you used
git log --oneline -- '*test*' '*spec*' '.github/**' '*.yml' '*.yaml' \
                     'package.json' '.pre-commit-config.yaml' | wc -l
```

State how you counted it, and say plainly if it is a loose file-touch proxy rather than a semantic
count.

## Deduplicate correlated findings

One change replicated across several repos or several files is **one incident with several
instances**, not several incidents. A single toolchain migration that drops a gate in four repos is
one finding. Collapse before counting, or a single event will masquerade as a pattern.

## Enumerate — never summarise a bucket

"Several tests were skipped" is useless: a count cannot be inverted back into a membership. Every
finding gets its own row.

| sha | date | file | mode | SILENT/ANNOUNCED | what changed | confidence |
| --- | --- | --- | --- | --- | --- | --- |

Then: the denominator and how it was counted · anything UNCLASSIFIED · any present-state gate that
never worked · anything about the repo that made the sweep unreliable (shallow clone, squashed
merges, vendored code, config living only on an unmerged branch).

## Calibration

From one sweep of four TypeScript/Node repos, ~560 detector-touching commits, human-authored:

- **~1.25% silent-weakening rate.** Seven silent, six announced.
- **Rates varied 0%–4.3% across repos** — a zero is entirely plausible for a well-disciplined
  codebase, so a null result is a result, not a failed sweep.
- **`F4 muffled` was the most common silent mode**, which surprised the sweep's designer, who had
  predicted `F1`/`F3` because those grep better. Deleted lines pickaxe cleanly. Do not assume the
  loud modes dominate.
- The highest-value finding was a **toolchain migration that dropped a zero-warning lint gate** while
  presenting itself as a speed improvement. Migrations are the richest hunting ground: they change
  everything at once and get reviewed as infrastructure.

Treat these as one data point from one stack, not a universal norm.

## Report, do not adjudicate

State what changed. Do not recommend fixes, rank severity, or assign blame. Whether a weakening was
right is a judgement for someone with context the history does not contain — and a sweep that
editorialises gets argued with instead of acted on.

The pull toward a "what I'd suggest" section at the end is strong, and it is the most common way this
sweep goes wrong. Leave it out. The table is the deliverable.
