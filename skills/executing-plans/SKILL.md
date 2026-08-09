---
name: executing-plans
description: Use when you have a written implementation plan to execute in a separate session with review checkpoints
---

# Executing Plans

## Overview

Load plan, review critically, execute all tasks, run automated verification before completing.

**Core principle:** Execute tasks, then run automated verification before considering work done.

**Announce at start:** "I'm using the executing-plans skill to implement this plan."

**Note:** Superpowers works much better with access to subagents (Claude Code, Codex CLI, Codex App, Copilot CLI, and Gemini CLI all qualify; see the per-platform tool refs in `../using-superpowers/references/`). If subagents are available, use superpowers:subagent-driven-development instead of this skill.

## The Process

### Step 1: Load and Review Plan
1. Ensure an isolated workspace: use superpowers:using-git-worktrees to create one or verify the existing one
2. Read plan file
3. Review critically - identify any questions or concerns about the plan
4. If concerns that block execution: Escalate (see superpowers:escalation) and continue with what is not blocked
5. If no concerns: Create todos for the plan items and proceed

### Step 2: Execute Tasks

For each task:
1. Mark as in_progress
2. Follow each step exactly (plan has bite-sized steps)
3. Run verifications as specified
4. Mark as completed

### Step 3: Automated Verification

When all tasks are complete:
- Run the full test suite
- Run the linter/type-check if configured
- Verify all plan requirements are met (line-by-line checklist)
- If all pass, proceed to completion
- If any fail, attempt a fix once. If still failing, escalate (see superpowers:escalation)

In unattended operation this gate stands where the interactive flow would ask a
human to sign off. Never finalize on a report of passing tests — run them.

### Step 4: Complete Development

After all tasks complete and verified:
- Announce: "I'm using the finishing-a-development-branch skill to complete this work."
- **REQUIRED SUB-SKILL:** Use superpowers:finishing-a-development-branch
- Follow that skill to verify tests and create a PR for review

## When to Escalate

**Escalate (see superpowers:escalation) when:**
- Hit a blocker (missing dependency, test fails, instruction unclear)
- Plan has critical gaps preventing starting
- You don't understand an instruction
- Verification fails after one retry

Log it and continue with other independent tasks if any remain. Do not guess
past a genuine blocker, and do not wait on an answer that is not coming.

## Remember
- Review plan critically first
- Follow plan steps exactly
- Don't skip verifications
- Reference skills when plan says to
- Run automated verification before completing — never finalize without test evidence
- When blocked, escalate and continue with independent tasks
- Never start implementation on main/master branch — always use a feature branch
