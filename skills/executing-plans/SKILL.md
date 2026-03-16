---
name: executing-plans
description: Use when you have a written implementation plan to execute in a separate session with review checkpoints
---

# Executing Plans

## Overview

Load plan, review critically, execute all tasks, run automated verification before completing.

**Core principle:** Execute tasks, then run automated verification before considering work done.

**Announce at start:** "I'm using the executing-plans skill to implement this plan."

**Note:** If subagents are available, use superpowers:subagent-driven-development instead of this skill for higher quality output.

## The Process

### Step 1: Load and Review Plan
1. Read plan file
2. Review critically - identify any questions or concerns about the plan
3. If concerns that block execution: Escalate (see escalation skill)
4. If no concerns: Create TodoWrite and proceed

### Step 2: Execute Tasks

For each task:
1. Mark as in_progress
2. Follow each step exactly (plan has bite-sized steps)
3. Run verifications as specified
4. Mark as completed

### Step 3: Automated Verification
When all tasks complete:
- Run the full test suite
- Run linter if configured
- Verify all plan requirements are met (line-by-line checklist)
- If all pass, proceed to completion
- If any fail, attempt fix once. If still failing, escalate (see escalation skill)

### Step 4: Complete Development

After all tasks complete and verified:
- Announce: "I'm using the finishing-a-development-branch skill to complete this work."
- **REQUIRED SUB-SKILL:** Use superpowers:finishing-a-development-branch
- Follow that skill to verify tests, present options, execute choice

## When to Escalate

**Escalate (see escalation skill) when:**
- Hit a blocker (missing dependency, test fails, instruction unclear)
- Plan has critical gaps preventing starting
- You don't understand an instruction
- Verification fails after one retry

Continue with other independent tasks if any remain.

## Remember
- Review plan critically first
- Follow plan steps exactly
- Don't skip verifications
- Reference skills when plan says to
- Run automated verification before completing — never finalize without test evidence
- When blocked, escalate (see escalation skill) and continue with independent tasks
- Never start implementation on main/master branch — always use a feature branch

## Integration

**Required workflow skills:**
- **superpowers:using-git-worktrees** - REQUIRED: Set up isolated workspace before starting
- **superpowers:writing-plans** - Creates the plan this skill executes
- **superpowers:finishing-a-development-branch** - Complete development after all tasks
