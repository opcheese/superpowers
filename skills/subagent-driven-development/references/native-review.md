# Native Review at Each Task Boundary

Part of superpowers:subagent-driven-development. The task review came back
clean, or every open finding is parked with a ruling at the cap. Read this
at every task boundary, before you write the completion line.

## Why this stop exists

**This stop is not a progress ping.**
At each task boundary you stop for a specific reason:
the strongest reviewer available is one you cannot run. `/code-review` is
marked `disable-model-invocation` — you cannot invoke it, it cannot be
preloaded into a subagent, and it will not fire from a scheduled task. Your
human partner typing it is the *only* path to that reviewer. Asking them to
run it is a request for work no one else in this loop can do, which is what
separates it both from "should I continue?" and from the adjudication
questions the rulings discipline forbids — a ruling is a decision you could
have made, and this is not one.

## The checkpoint

When the review comes back clean — or every open finding is parked with a
ruling at the cap — stop and ask your human partner to run the native
reviewer over this task's range before you write the completion line:

```
Task <N> (<one-line description>) is reviewed clean: <base7>..<head7>.
Please run:  /code-review high <base7>..<head7>
Paste anything it finds and I'll route it into the fix loop. Reply "skip"
to waive review for this task, or "waive run" to waive it for the rest of
this run.
```

**Prefer the native reviewer over your own,** but check what it returns. It
reviews from several specialised angles, and some of its variants verify each
finding. Not all do: in Claude Code 2.1.303 the inline variants it runs at
medium, high and xhigh effort set a findings target and deduplicate without a
verification pass. Check each finding before it enters the fix loop. Your reviewer
is the spec-compliance and task-quality gate — the thing native will not do,
because it reviews a diff and has no idea what the plan required. The two are
complementary, and this checkpoint is where the second one gets run.

`/code-review` is for code diffs, not plans or other documents: review those with superpowers:reviewing-documents.

A `/code-review` run in the background or in a forked session can move the checkout it runs in onto the head it reviews, and your next commit then lands on a detached HEAD. Run it from a worktree of its own (`git worktree add --detach <tmp-dir> <head7>`, removed afterwards), not from the checkout you are working in.

Ask through your harness's question tool when it has one (AskUserQuestion in
Claude Code), with "ran it, pasting findings", "skip" and "waive run" as the
options. A plain-text request at the end of a turn reads like a finished
thread and gets missed.

Then handle the answer:

- **Findings returned:** they enter the fix loop exactly like reviewer
  findings, at the current round number. Re-run this checkpoint after the
  fix round closes.
- **Clean:** note it and continue.
- **"skip" / "waive run":** record it —
  `Task <N>: native review waived (<scope>)` — and continue. A waiver is your
  human partner's to give; never take it on their behalf, and never infer one
  from silence.
- **No answer yet:** wait. This is a gate, not a notification.

Record the outcome on the completion line (SKILL.md, step 5) as `native clean`
or `native waived`, whichever applies.

## Rationalizations

| Excuse | Reality |
|--------|---------|
| "My task reviewer already approved it, native would be redundant" | Your reviewer has no false-positive verification stage and native has no idea what the plan required. Neither substitutes for the other. |
| "My task reviewer already approved it, native would be redundant" | Your reviewer has no false-positive verification stage and native has no idea what the plan required. Neither substitutes for the other. |
| "Continuous execution says don't interrupt them" | It carves out this checkpoint by name. Continuous execution bans status pings, not gates. |
| "A per-task checkpoint is a fake checkpoint that trains rubber-stamping" | A fake checkpoint asks for a decision already made. This asks them to run a reviewer you are structurally unable to run. |
| "They're busy — I'll batch the review requests at the end" | Batched findings land on code three tasks deep, and the fix touches work already built on. Per-task is why the range is small. |
| "They didn't answer, so they're fine with it" | Silence is not a waiver. Waivers are explicit and go in the ledger. |
