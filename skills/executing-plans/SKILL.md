---
name: executing-plans
description: Use when you have a written implementation plan to execute in a separate session with review checkpoints
---

# Executing Plans

## Overview

Load plan, review critically, execute all tasks, demand human verification, report when complete.

**Core principle:** Execute tasks, then present the work to your human partner and get explicit verification before considering it done.

**Announce at start:** "I'm using the executing-plans skill to implement this plan."

**Note:** Tell your human partner that Superpowers works much better with access to subagents (Claude Code, Codex CLI, Codex App, Copilot CLI, and Gemini CLI all qualify; see the per-platform tool refs in `../using-superpowers/references/`). If subagents are available, use superpowers:subagent-driven-development instead of this skill.

## The Process

### Step 1: Load and Review Plan
1. Ensure an isolated workspace: use superpowers:using-git-worktrees to create one or verify the existing one
2. Read plan file
3. Review critically - identify any questions or concerns about the plan
4. If concerns: Raise them with your human partner before starting
5. If no concerns: Create todos for the plan items and proceed

### Step 2: Execute Tasks

For each task:
1. Mark as in_progress
2. Follow each step exactly (plan has bite-sized steps)
3. Run verifications as specified
4. Commit, then run the native-review checkpoint below
5. Mark as completed

**Native-review checkpoint (every task, before marking it completed).** Ask
your human partner to run the native reviewer over what this task changed:

```
Task <N> (<one-line description>) is done and verified: <base7>..<head7>.
Please run:  /code-review high <base7>..<head7>
Paste anything it finds and I'll fix it before moving on. Reply "skip" to
waive review for this task, or "waive run" to waive it for the rest of
this run.
```

Then wait. This is a gate, not a notification, and it is the one place you
stop between tasks.

**Why this and nothing else interrupts them:** `/code-review` is marked
`disable-model-invocation` — you cannot invoke it, and it cannot be preloaded
into a subagent. Your human partner typing it is the only path to the
strongest reviewer available. That is a request for work no one else can do,
which is what separates it from a "should I continue?" ping.

Findings go back into the task before you mark it completed. A waiver is
explicit — "skip" or "waive run" — never inferred from silence, and it is
noted in your report at the end.

### Step 3: Demand Human Verification

After all tasks complete:
- Present the completed work to your human partner (what changed, test results)
- Wait for their explicit verification before proceeding — do not assume approval
- If they request changes, make them and re-verify before continuing

### Step 4: Complete Development

After all tasks complete and the human has verified the work:
- Announce: "I'm using the finishing-a-development-branch skill to complete this work."
- **REQUIRED SUB-SKILL:** Use superpowers:finishing-a-development-branch
- Follow that skill to verify tests, present options, execute choice

## When to Stop and Ask for Help

**STOP executing immediately when:**
- Hit a blocker (missing dependency, test fails, instruction unclear)
- Plan has critical gaps preventing starting
- You don't understand an instruction
- Verification fails repeatedly

**Ask for clarification rather than guessing.**

## When to Revisit Earlier Steps

**Return to Review (Step 1) when:**
- Partner updates the plan based on your feedback
- Fundamental approach needs rethinking

**Don't force through blockers** - stop and ask.

## Remember
- Review plan critically first
- Follow plan steps exactly
- Don't skip verifications
- Reference skills when plan says to
- Stop when blocked, don't guess
- Never start implementation on main/master branch without explicit user consent
