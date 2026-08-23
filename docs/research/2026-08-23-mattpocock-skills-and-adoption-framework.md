---
title: mattpocock/skills — what to take, and a framework for deciding
status: complete
last_verified: 2026-08-23
owner: opcheese
area: research
audience: maintainers
---

# mattpocock/skills — what to take, and a framework for deciding

Two questions: what do we do with [mattpocock/skills](https://github.com/mattpocock/skills),
and how do we decide this class of question without re-arguing it every time
a popular skill pack appears.

The second question is the more valuable one, so the framework comes first
and the verdict is an application of it.

## What was examined

Cloned at commit dated 2026-08-21 (454 commits, MIT, ~233.6k stars — by a
wide margin the most-installed skills pack in the ecosystem). 36 `SKILL.md`
files across `engineering/`, `productivity/`, `misc/`, `in-progress/` and
`deprecated/`. Every skill's frontmatter and the full text of `tdd`,
`code-review`, `grilling`, `domain-modeling`, `implement` and `wayfinder`
were read directly; the rest were classified from frontmatter.

Its stated problem is the inverse of ours: agents accelerate software
entropy, so the fix is caring about code *design* (Ousterhout's deep
modules). Superpowers' stated problem is agents skipping *process*. Those
are different diseases, which is the first hint that the two packs are more
complementary than competitive — except along one seam, described below.

## The framework

Five filters, applied in order. The first that fires decides.

### Filter 0 — Reachability (mechanical, branch-specific)

Does the skill require a human turn to run at all? Two ways it can:

- `disable-model-invocation: true` in its frontmatter — an agent cannot
  invoke it, it cannot be preloaded into a subagent, and it will not fire
  from a scheduled task.
- It runs an interview and *waits* for answers mid-flow, flag or no flag.

On `agents`, either answer is disqualifying: the skill is inert, and a skill
that cannot fire is worse than absent because it still costs description
tokens in every session's skill listing. On `main`, neither is disqualifying
— `main`'s per-task native-review checkpoint exists precisely because a
`disable-model-invocation` reviewer was worth asking a human to type.

**This filter alone settles 21 of the 36 skills for the `agents` branch.**

### Filter 1 — Layer

Three layers, with different adoption economics:

| Layer | What it supplies | How many can coexist |
|-------|------------------|----------------------|
| **Vocabulary / reference** | Nouns and distinctions — *seam*, *deep module*, *ADR*, *glossary*, *frontier* | Many. This is the free lane. |
| **Discipline** | A rule with a gate — red-before-green, no silent discard, evidence before assertions | **One per topic.** Two disciplines on one topic is worse than either alone. |
| **Orchestration** | Drives a multi-step flow and dispatches subagents | **One spine per repo.** Ours is superpowers. |

Reference-layer skills compose with anything and are cheap to adopt.
Discipline and orchestration are exclusive goods.

### Filter 2 — Spine collision

Does it re-implement a step of spec → plan → implement → review → finish?
If yes, it is a **replacement candidate**, not an addition. Decide it
head-to-head on merits and install exactly one. Installing both does not
give you two opinions; it gives you a coin flip.

### Filter 3 — Trigger overlap

Would its `description` fire on the same user message as a skill we already
have? Skill selection happens before either body is read, so two skills with
overlapping triggers are resolved by whichever the model reaches for that
day. Non-determinism at the trigger layer is the failure mode that is
hardest to notice and hardest to debug, because both outcomes look
reasonable in isolation.

Name collisions are a special case worth checking by hand: this pack ships a
skill literally named `code-review`, which is also the name of Claude Code's
own built-in `/code-review`. On `main`, whose whole per-task checkpoint rests
on the human typing that command and getting the native multi-agent
reviewer, that ambiguity is not cosmetic.

### Filter 4 — Setup coupling and carrying cost, and *where* you vendor

Two costs, both recurring:

- **Per-repo setup.** Does it need scaffolding — a `CONTEXT.md`, an
  `docs/agents/issue-tracker.md`, an ADR directory — before it works? This
  pack's engineering flow does; several skills fail closed and tell the user
  to run `/setup-matt-pocock-skills`.
- **Per-sync carrying cost — and this splits three ways, not two.**
  Vendoring into *this fork's* `skills/` means re-applying across every
  upstream sync forever: expensive. Installing alongside as a separate
  plugin costs nothing at sync time but pays the Filter 3 tax at runtime.
  The third option is the cheap one and is easy to miss: **vendor into a
  sibling plugin in a marketplace repo you own.** The synced fork never
  sees those files, so the sync cost is zero, and because you control the
  plugin boundary you can carry a curated subset instead of someone else's
  whole pack — which is the only way to get past Filter 3, since a git
  plugin source is all-or-nothing.

  A marketplace can list plugins that live in *other* repositories, each
  pinned independently by `ref` or exact `sha`. So "one pack for the team"
  is a packaging question with a clean answer, and it does not force a
  vendor-everything decision.

### The four outcomes

- **Adopt-vendored** — copy into `skills/`, in this fork's idiom, carried
  across syncs. Reserve for things that must be on both branches and that
  upstream will never supply.
- **Install-alongside** — leave it as its own plugin. Default for
  reference-layer skills with no trigger overlap.
- **Borrow-the-idea** — take the distinction, rewrite it in our idiom inside
  an existing skill. Default when a discipline collides: one gate, enriched.
- **Decline** — an orchestration spine that is not ours, or a skill Filter 0
  makes inert on the branch in question.

## Distribution is a separate question from adoption

Worth stating because conflating the two produced the wrong answer the
first time through. "I want to hand my devs one thing" is a *distribution*
requirement, and it is satisfied by a marketplace repo, not by vendoring.
Adoption decides *what* is in the set; distribution decides how it arrives.
Filters 1-3 answer the first. Filter 4 answers the second.

One consequence worth keeping: a marketplace entry pins a `ref`, so a fork
with two branch philosophies becomes two named plugins in one catalog —
`superpowers-human` at `main`, `superpowers-agents` at `agents`. The
human/unattended split stops living in a branch name and becomes something
a developer picks at install time.

## Applying it

### Declined by Filter 0 on `agents` (21 skills)

`ask-matt`, `grill-with-docs`, `grill-me`, `implement`, `to-spec`,
`to-tickets`, `triage`, `wayfinder`, `improve-codebase-architecture`,
`handoff`, `teach`, `to-questionnaire`, `wait-what`,
`setup-matt-pocock-skills`, and the whole `in-progress/` tree.

Note what that list *is*: every orchestrator in the pack. The entire
user-invoked flow — spec, tickets, implement, review, wayfind — is
structurally unavailable to an unattended run. This is the same wall the
native `/code-review` research hit in the previous cycle, and it is now
worth stating as a general property rather than a one-off finding:
**skill packs designed for an attended human are load-bearing on their
orchestrators, and orchestrators are exactly what gets flagged
`disable-model-invocation`.**

For `main` these stay eligible, but they then meet Filter 2 as a group:
`to-spec` → `to-tickets` → `implement` → `code-review` is a complete second
spine parallel to brainstorming → writing-plans → SDD → requesting-code-review.
Running both is not an option; replacing ours with theirs is a much larger
decision than this research was scoped to make, and `main`'s spine is
additionally tuned around the native reviewer. **Declined for now, on spine
grounds, not on quality.**

### Discipline collisions — borrow, don't install

| Their skill | Our skill | Verdict |
|---|---|---|
| `tdd` | `superpowers:test-driven-development` | Borrow. Theirs is the better *reference* (seams, tautological-test detection, vertical-slicing argument); ours is the better *gate* (Iron Law, rationalization table). The two ideas worth stealing outright: **pre-agreed seams confirmed before any test is written**, and **tautological tests** as a named anti-pattern — an assertion that recomputes the expected value the way the code does can never disagree with the code. That second one is a `detector-weakening-sweep` finding in test-authoring clothes; it belongs in both skills. |
| `diagnosing-bugs` | `superpowers:systematic-debugging` | Borrow at most. Same topic, same gate, no room for two. |
| `research` | fork-only `topic-research` | Decline. Ours is 108 lines and tuned; theirs is 131 words. |
| `writing-for-agents` | `superpowers:writing-skills` | Decline. Ours carries the RED/GREEN discipline that is the whole point. |
| `code-review` | `requesting-code-review` + native `/code-review` | Decline, and note the name collision above. Their two-axis split (standards vs spec, in parallel subagents) is the same architecture native already runs with more axes and a false-positive verification stage. |
| `prototype` | brainstorming's new **spike** path | Decline — and note the timing. Upstream v6.3.0 landed the spike path in this same sync. The gap this skill would have filled closed on its own. |

### Filter 0 has a second half, and it bites after adoption

The `disable-model-invocation` flag is the loud half. The quiet half — *does
it wait for a person mid-flow?* — has to be read out of the skill body, and
two of the four skills below fail it despite having no flag:

- **`domain-modeling`** works by interrogating the user ("your glossary
  defines 'cancellation' as X, but you seem to mean Y, which is it?"). No
  answer ever arrives in an unattended run.
- **`git-guardrails-claude-code`** is worse than inert, it is actively
  breaking: its hook blocks `git push` with no branch distinction, and the
  `agents` spine is always-PR, so it must push a feature branch. Installing
  it unattended fails every run at its finish step.

The lesson for the framework: **run Filter 0 per skill, not per pack, and
run it against the skill body rather than the frontmatter.** A pack can be
uniformly model-invokable and still be half unusable. Where a pack splits
this way, split it at the *plugin* boundary rather than forking each skill
into human and agent variants — variants double the maintenance and
recreate exactly the trigger competition Filter 3 exists to prevent.

### The actual prize — reference layer, no collision (4 skills)

These occupy a layer superpowers has nothing in. Superpowers tells an agent
*what process to follow*; it says almost nothing about *what good structure
looks like*. That is a real hole, and this is the pack that fills it.

- **`codebase-design`** — the shared vocabulary of module, interface, depth,
  seam, adapter, leverage, locality. Model-invokable, pure reference,
  explicitly designed to be consulted rather than run.
- **`domain-modeling`** — glossary and ADR discipline: challenge terms
  against `CONTEXT.md`, sharpen fuzzy language, and the three-part ADR test
  (hard to reverse, surprising without context, a real trade-off). The ADR
  test is good enough to be worth having regardless of the rest.
- **`resolving-merge-conflicts`** — 133 words, no analogue anywhere in
  superpowers, and merge conflicts are exactly the situation where an
  unattended agent does something regrettable.
- **`git-guardrails-claude-code`** — hooks that block `push`, `reset --hard`,
  `clean`, branch deletion. This is worth a hard look for `agents`
  specifically: upstream's own v6.3.0 stop-list names "irreversible or
  destructive operation" as one of the four things that must stop a run, and
  our branch answers that structurally rather than mechanically. A hook is a
  mechanical answer.

**Recommendation: vendor the four into a curated plugin in our own
marketplace repo**, split by the Filter 0 result above —
`codebase-vocabulary` (codebase-design, resolving-merge-conflicts) for both
spines, `codebase-vocabulary-human` (domain-modeling, git-guardrails) for
interactive use only.

This supersedes an earlier draft of this document, which said
"install-alongside, not vendored." That was right about the cost of
vendoring *into the fork* and wrong about the alternatives: it assumed the
only two options were "inside `skills/`" or "a whole third-party plugin,"
and a curated sibling plugin is both cheaper than the first and more precise
than the second. Since a git plugin source is all-or-nothing, carrying a
four-skill subset is only possible by copying — MIT, with attribution.

`domain-modeling` carries a `CONTEXT.md` setup cost; adopt it only in repos
that will actually keep a glossary.

## Open items

- The two `tdd` borrowings (pre-agreed seams, tautological tests) are skill
  edits, so under this repo's Iron Law they need RED/GREEN evidence before
  they land. Not done here.
- Whether `main` should eventually swap its spine for the mattpocock flow is
  a real question this doc deliberately does not answer.
- ~~`git-guardrails-claude-code` needs its blocked list narrowed~~ — **done.**
  Our copy in the catalog is branch-aware: it blocks pushes resolving to a
  protected branch and every force/mirror/all/delete push, and allows the
  feature-branch push a PR requires. It is the one file in the catalog
  modified from upstream, recorded in that plugin's `NOTICE.md`, and it
  ships with a 34-case suite.

  Worth recording *why* the suite exists: the first version of that script
  silently allowed every push. It read command segments from a `printf`
  with no trailing newline, so `read` hit EOF and the loop body never
  executed once. The script looked right, exited 0 on everything, and
  blocked nothing — a gate that could not fail, which is precisely what
  `detector-weakening-sweep` is written to find. It was caught only because
  the tests asserted the *blocking* direction rather than just checking
  that safe commands still passed. A guard whose tests only prove it does
  not fire is not tested at all.
