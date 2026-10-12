---
name: brainstorming
description: "You MUST use this before any creative work - creating features, building components, adding functionality, or modifying behavior. Explores requirements and design before implementation."
---

# Brainstorming Ideas Into Designs

Turn ideas into fully formed designs and specs through single-pass analysis with self-review.

Start by classifying how much process the request needs, then work through
your path: understand the context, analyze the requirements, generate a
design, and put it through that path's gate.

<HARD-GATE>
Do NOT invoke any implementation skill, write any code, scaffold any
project, or take any implementation action until you have generated a
design and it has passed its path's gate. This applies to EVERY task on
EVERY path below — the ceremony scales with the task; the gate never does.
</HARD-GATE>

## Three Paths

Before you analyze anything, classify the request and **write the
classification down** with the one fact that decided it — "bounded: the
auth middleware this changes is already in `src/auth/`" — at the top of
whatever record this run produces. Nobody is here to override a
misclassification, so it has to be auditable after the fact:

- **Spike** — a feasibility question ("can we...", "is it possible...",
  "quick and dirty is fine") whose output is an answer, not code you
  keep. Record the question and what you'll try in 2-3 sentences, then
  find out as cheaply as correctness allows. No design doc, no spec file,
  no builder check. Report findings as a recommendation; anything you built
  stays labeled throwaway and does not get merged.
- **Bounded** — a well-scoped change to code that already exists in this
  repo: a new flag, a small endpoint, a one-file fix. Understanding the
  kind of app is not enough — bounded means the flow you are changing is
  already here to read. If there is no existing flow to change, the task
  is not bounded. Analyze the requirements that matter, write a short
  design (a few sentences to a few short paragraphs) into the run's
  record, and implement it through the normal development workflow — TDD
  applies, and the task review is the gate. No spec file, no builder-check
  subagent, no implementation plan document.
- **Architectural** — new projects, new subsystems, changes that
  restructure how components fit together or alter interfaces others
  depend on. Follow the full process: requirements, approaches, sectioned
  design, written spec, builder check via subagent, then the writing-plans
  skill.

When in doubt between two paths, take the heavier one — and unattended,
doubt is cheaper to resolve upward than it looks, because the lighter
path's mistakes surface only after the work is done. The ratchet is
one-way: hidden complexity discovered mid-task upgrades the path — say
so in the record and step up. Nothing downgrades mid-task.

## Anti-Pattern: "Too Simple To Need A Design"

Every path ends with a design that something other than your own
confidence has checked. A todo list, a single-function utility, a config
change — the design may be two sentences, but you MUST generate one and
run it through the path's gate. "Simple" tasks are where unexamined
assumptions cause the most wasted work. What scales with simplicity is
the artifact, never the gate.

## Red Flags

| Thought | Reality |
|---------|---------|
| "This is too simple to need a design" | Simple means a short design, not no design. Two sentences, then the path's gate. |
| "I'll call it bounded and skip the spec" | Reaching for a label to skip work IS the doubt — take the heavier path. |
| "It's bounded, so the task review will catch anything I got wrong" | The task review checks the code against the design. It cannot catch a design nobody wrote down. |
| "I understand this kind of app, so it's bounded" | Bounded measures the repo, not your familiarity. A new project has no existing flow — it is architectural. |
| "The spike works, so I'll keep the code" | A spike's output is an answer. Keeping the code is a new request — classify it. |
| "It grew, but I'm almost done — no need to re-classify" | Hidden complexity upgrades the path mid-task. Record the upgrade and step up. |
| "No human is watching, so the classification is just bookkeeping" | It is the only record of why this task got the process it got. Write it down. |
| "The builder's question is theirs to answer, so I'll escalate it" | Unattended, a question only they could answer gets a recorded ruling and goes to Open items. Escalate only when there is no intent to rule from. |
| "I'll skip the builder check, I know what I meant" | You know what you meant. The check finds what the document doesn't say. |

## Checklist

Classify first, record the path, then create a task for each item on your
path and complete them in order.

**Spike:**
1. **Explore project context** — enough to frame the probe
2. **Record question + probe plan** — 2-3 sentences
3. **Investigate** — as cheaply as correctness allows
4. **Report findings** — a recommendation; label anything built as throwaway

**Bounded:**
1. **Explore project context** — check files, docs, recent commits
2. **Analyze requirements** — purpose, constraints, success criteria
3. **Write a short design into the run's record** — approach, files touched, testing
4. **Implement** — proceed with the normal development workflow (TDD applies); no spec file, no plan document

**Architectural:**
1. **Explore project context** — check files, docs, recent commits
2. **Analyze requirements** — identify purpose, constraints, success criteria from the task description and codebase
3. **Evaluate 2-3 approaches** — with trade-offs, select the best one with reasoning
4. **Name the boundaries** — if the design creates, moves, or removes one, invoke codebase-vocabulary:codebase-design (when installed) and use its vocabulary and deletion test; skip only if no boundary moves
5. **Generate design** — complete design covering architecture, components, data flow, error handling, testing
6. **Record the model** — if the design coins, renames, or redefines a domain term, or you rejected an alternative worth keeping, invoke codebase-vocabulary-human:domain-modeling (when installed); skip only if the vocabulary is unchanged
7. **Write design doc** — save to `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md` and commit
8. **Builder check via subagent** — dispatch a fresh builder with `builder-check-prompt.md` (see below); sort its questions, settle them in the doc, commit
9. **Transition to implementation** — invoke writing-plans skill to create implementation plan

## Process Flow

```dot
digraph brainstorming {
    "Classify: spike / bounded / architectural" [shape=diamond];
    "Record question + probe plan" [shape=box];
    "Investigate; report recommendation" [shape=doublecircle];
    "Write short design into the record" [shape=box];
    "Implement via normal workflow (no spec, no plan doc)" [shape=doublecircle];
    "Explore project context" [shape=box];
    "Analyze requirements" [shape=box];
    "Evaluate approaches" [shape=box];
    "Name the boundaries" [shape=box];
    "Generate complete design" [shape=box];
    "Record the model" [shape=box];
    "Write design doc" [shape=box];
    "Dispatch builder-check subagent" [shape=box];
    "Sort questions: answered / minor / theirs" [shape=box];
    "Settle each in the doc; rule on 'theirs'" [shape=box];
    "Invoke writing-plans skill" [shape=doublecircle];
    "Hidden complexity? Upgrade path" [shape=box];

    "Classify: spike / bounded / architectural" -> "Record question + probe plan" [label="spike"];
    "Classify: spike / bounded / architectural" -> "Write short design into the record" [label="bounded"];
    "Classify: spike / bounded / architectural" -> "Explore project context" [label="architectural"];
    "Record question + probe plan" -> "Investigate; report recommendation";
    "Write short design into the record" -> "Implement via normal workflow (no spec, no plan doc)";
    "Hidden complexity? Upgrade path" -> "Classify: spike / bounded / architectural";

    "Explore project context" -> "Analyze requirements";
    "Analyze requirements" -> "Evaluate approaches";
    "Evaluate approaches" -> "Name the boundaries";
    "Name the boundaries" -> "Generate complete design";
    "Generate complete design" -> "Record the model";
    "Record the model" -> "Write design doc";
    "Write design doc" -> "Dispatch builder-check subagent";
    "Dispatch builder-check subagent" -> "Sort questions: answered / minor / theirs";
    "Sort questions: answered / minor / theirs" -> "Settle each in the doc; rule on 'theirs'";
    "Settle each in the doc; rule on 'theirs'" -> "Invoke writing-plans skill";
}
```

**Terminal states are path-bound.** Architectural: the ONLY skill you
invoke after brainstorming is writing-plans — never frontend-design,
mcp-builder, or any other implementation skill. Bounded: after the short
design is recorded, implementation proceeds directly through the normal
development workflow. Spike: the terminal state is a reported
recommendation.

## The Process

The subsections below serve the bounded and architectural paths (a spike
stops at "record the probe, run it"). Sections from **Exploring
approaches** onward are architectural-path depth — for bounded work,
context plus requirements plus a short recorded design is the whole
process.

**Understanding the task:**

- Check out the current project state first (files, docs, recent commits)
- Assess scope: if the request describes multiple independent subsystems, decompose into sub-projects first. Each sub-project gets its own spec → plan → implementation cycle.
- Identify purpose, constraints, and success criteria from the task description, codebase context, and any referenced docs
- If requirements are ambiguous and no safe default exists, use the escalation skill

**Exploring approaches:**

- Evaluate 2-3 different approaches with trade-offs
- Select the best approach with clear reasoning
- Lead with the recommended option and explain why
- YAGNI ruthlessly - remove unnecessary features from every approach and design

**Generating the design:**

- Generate a complete design in a single pass
- Scale each section to its complexity: a few sentences if straightforward, up to 200-300 words if nuanced
- Cover: architecture, components, data flow, error handling, testing

**Design for isolation and clarity:**

- Break the system into smaller units that each have one clear purpose, communicate through well-defined interfaces, and can be understood and tested independently
- For each unit, you should be able to answer: what does it do, how do you use it, and what does it depend on?
- Can someone understand what a unit does without reading its internals? Can you change the internals without breaking consumers? If not, the boundaries need work.
- Smaller, well-bounded units are also easier for you to work with - you reason better about code you can hold in context at once, and your edits are more reliable when files are focused. When a file grows large, that's often a signal that it's doing too much.

**Working in existing codebases:**

- Explore the current structure before proposing changes. Follow existing patterns.
- Where existing code has problems that affect the work (e.g., a file that's grown too large, unclear boundaries, tangled responsibilities), include targeted improvements as part of the design - the way a good developer improves code they're working in.
- Don't propose unrelated refactoring. Stay focused on what serves the current goal.

## After the Design (architectural path)

**Documentation:**

- Write the validated design (spec) to `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md`
  - (User preferences for spec location override this default)
- Use elements-of-style:writing-clearly-and-concisely skill if available
- Commit the design document to git

**Builder check (via subagent):**
After writing and committing the spec, dispatch a fresh subagent with the
prompt in `builder-check-prompt.md`, passing the spec path and the domain.
It reads the design as the person who has to plan and build from it,
unable to ask the author anything, and lists the questions they would
still have to guess at. This is the autonomous gate that replaces a human
reading the spec. Do not invoke writing-plans until it is back.

Sort every question into one of three piles:

- **Answered already** by the request, the repo, or your analysis: put the
  answer in the spec.
- **Minor:** a sensible builder could settle it without changing what gets
  built. Settle it and mark it in the spec as your call.
- **Theirs:** only the person who asked could answer it, and the answer
  changes what gets built. Unattended, nobody is here to answer, so rule
  on it: pick the option that best serves the stated intent, write it into
  the spec as `Ruling: <decision> — <why> — <what it costs if wrong>`, and
  carry it forward. Rulings reach the human through the PR's **Open
  items** (see superpowers:finishing-a-development-branch). Escalate (see
  superpowers:escalation) only when every option is a guess with no
  intent to rule from.

Choices about how the thing gets made (platform, language, libraries,
medium) follow the same rule: build on what the repo already uses, and
any choice the request didn't make goes in the spec as your call, never as
something the requester agreed to.

Commit the updated spec. The builder check runs once; its job is to
surface what a builder would guess at, not to approve the design.

**Implementation:**

- Invoke the writing-plans skill to create a detailed implementation plan
- Do NOT invoke any other skill. writing-plans is the next step.
