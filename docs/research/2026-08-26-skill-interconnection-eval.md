---
date: 2026-08-26
last_verified: 2026-08-26
status: complete
---

# Wiring the vocabulary skills into the Superpowers spine

Superpowers works partly because its skills hand off to each other: brainstorming
ends by naming writing-plans, writing-plans ends by naming
subagent-driven-development, and `superpowers:escalation` is referenced 23 times
because every skill reaches moments that are not the agent's call.

The vocabulary plugins we adopted (`codebase-vocabulary`,
`codebase-vocabulary-human`) had **zero** outbound references, in any of their four
skills. They were islands: they fired only when someone said their name.

This is the RED/GREEN record for connecting them. Every claim below is a measured
session, not a prediction.

## Method

In-session subagents were unusable: this machine's running session still pointed at
a deleted `superpowers-dev/5.0.6` cache path, so a subagent could not load the spine
at all and would have measured the wrong failure. Instead each scenario ran as a real
headless session (`claude -p --output-format stream-json`) against a throwaway git
fixture, with skills read from the actual plugin cache. Skill invocations were counted
from `Skill` tool-use entries in the transcript, not from what the agent said it did.

Scenarios, all phrased as ordinary work with no mention of skills:

- **s1** — a Stripe-only billing service must also support net-30 invoicing. Design only.
- **s2** — add a retry policy to `chargeCustomer`, test-first.
- **s3** — rebase a diverged branch whose conflict genuinely requires a judgment call.
- **s4** — invoke `codebase-design` explicitly, then "take it from there."

## RED — the baseline

| Scenario | Skills invoked | What it did instead |
|---|---|---|
| s1 | `brainstorming` | Called `recordPayment` "the load-bearing **seam**" — reached for the word, never the skill. Coined `Invoice`, `collectionMethod`, `amountSettledCents` and three more terms; wrote no glossary and no ADR. |
| s2 | `test-driven-development` | Injected a dependency, added an options bag and made the Stripe client lazy — a seam decision — then apologised for it as scope creep. Never named it. |
| s3 | `resolving-merge-conflicts` | Fired unprompted and resolved well, spotting that the branch's absolute prices would silently undo main's price rise. |
| s4 | `codebase-design` | Designed a good seam, then implemented it *and wrote its tests together*. Never handed off to TDD. |

s3 is the useful negative result: that join needs no pointer, because
`resolving-merge-conflicts` has a well-targeted description and triggers on its own.
One of the five planned pointers was deleted on this evidence.

## The finding that mattered

Two rounds of pointers written as **prose in the skill body** failed, including a
round using the strongest phrasing in the codebase (`REQUIRED SUB-SKILL`).

They were not ignored. They leaked: mentions of "seam" in s2 went from 0 to 4, and s1
picked up "deletion test". The agent read the pointer, borrowed its vocabulary, and
treated the one-line summary as a sufficient substitute for loading the skill — the
same shortcut `writing-skills` warns about for description fields, appearing here in
the body.

The third round moved the pointers to **structural positions** and they fired:

- a numbered **checklist item**, which brainstorming turns into a tracked task
- a **terminal handoff** section at the end of a skill

That matches the grain of the library. Every working interconnection in Superpowers
is an entry gate or a terminal handoff. Nothing anywhere says "pause mid-skill and
load another," and prose that tries to is absorbed rather than obeyed.

## GREEN — what ships

| Join | Mechanism | Result |
|---|---|---|
| `brainstorming` → `codebase-design` | checklist step, "Name the boundaries" | **Fires.** Control run without it: brainstorming alone. |
| `brainstorming` → `domain-modeling` | checklist step, "Record the model" | **Fires** on a repo with a `CONTEXT.md`/ADR convention — wrote ADR 0002 and updated the glossary. Control on the same fixture: neither. |
| `codebase-design` → `test-driven-development` | terminal "After the design" section | **Fires.** Baseline wrote code and tests together; now it hands off. |
| `test-driven-development` → `codebase-design` | entry gate, "Before The First Test" | **Partial.** Changes behaviour without invoking. |

`domain-modeling` correctly *declined* on the fixture with no `CONTEXT.md` and no ADR
directory — its own description scopes it to repos that keep those. That is the skill
judging its own applicability, not the pointer failing, which is why the hypothesis was
retested against a fixture that has both.

## The partial, stated plainly

The TDD gate does not produce an invocation. It does change what the agent does: where
the baseline reshaped `chargeCustomer` and then apologised, the same scenario now
reasons explicitly that "the seam had to move before any of this was testable" and
settles it before writing the first test. Better outcome, wrong mechanism. It ships
labelled as such rather than as a working link.

## Not tested

- `writing-plans` → `codebase-design`. Never written. Likely redundant, since the
  vocabulary now arrives through the spec that brainstorming produces.
- `domain-modeling` and `resolving-merge-conflicts` outbound. Never written. The
  conflict fixture has no test suite, so there was nothing for a
  `verification-before-completion` pointer to verify.
