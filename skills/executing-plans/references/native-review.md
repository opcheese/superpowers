# Native Review at Each Task Boundary

Part of superpowers:executing-plans. The task is done and its tests pass.
Read this at every task boundary, before you mark the todo complete.

## Why this stop exists

**This stop is not a progress ping.** At each
task boundary you ask your human partner to run `/code-review` over the
task's range. It is marked `disable-model-invocation` — you cannot invoke
it and it cannot be preloaded into a subagent — so your human partner
typing it is the only path to the strongest reviewer available. That is a
request for work no one else in this loop can do, which separates it from
"should I continue?".

## The checkpoint

Before marking the task's todo complete, ask your human partner:

```
Task <N> (<one-line description>) is done and verified: <base7>..<head7>.
Please run:  /code-review high <base7>..<head7>
Paste anything it finds and I'll fix it before moving on. Reply "skip" to
waive review for this task, or "waive run" to waive it for the rest of
this run.
```

Ask through your harness's question tool when it has one (AskUserQuestion in
Claude Code): a plain-text request at the end of a turn reads like a finished
thread and gets missed.

Then wait. This is a gate, not a notification, and it is the one place you
stop between tasks.

Findings go back into the task before you take the next one: fix each
under TDD (a test that reproduces it, RED→GREEN, suite green) and ledger
it as `Task <N>: native: fixed <finding> — <test name> RED→GREEN`. A
waiver is explicit — "skip" or "waive run" — never inferred from silence;
ledger it as `Task <N>: native review waived (<skip|run>)` and name it in
your final message.

## Rationalizations

| Excuse | Reality |
|--------|---------|
| "The native review is just another check-in, I'll skip it" | It is the only path to a reviewer you cannot run. Ask for it at every task boundary; only your human partner waives it. |
