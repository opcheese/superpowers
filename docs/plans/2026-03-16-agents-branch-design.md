# Superpowers for Autonomous Agents — Design

**For agentic workers** | 2026-03-16

## Goal

Create an `agents` branch of superpowers skills adapted for unattended Claude Code sessions (`claude -p` in CI/pipelines/cron). Same tools available, no human watching.

## Principles

1. **Human gates → automated verification gates** — run tests + linter instead of waiting for human approval
2. **Interactive prompts → sensible defaults** — pick the right option automatically
3. **Visual/browser features → removed** — no browser available in headless sessions
4. **Ambiguity → escalation** — log issue, stop task, continue with independent work
5. **Always PR, never merge** — safest default for unattended work

## Changes

### Deleted files
- `brainstorming/scripts/server.js` — WebSocket server for visual companion
- `brainstorming/scripts/helper.js` — client-side click handling
- `brainstorming/scripts/frame-template.html` — HTML frame template
- `brainstorming/visual-companion.md` — visual companion guide

### New file
- `escalation/SKILL.md` — shared escalation pattern for when agents hit ambiguity or repeated failures

### Modified skills (12)

| Skill | Change |
|-------|--------|
| brainstorming | Single-pass design generation + self-review via subagent. Remove interactive Q&A, visual companion, user approval gates. |
| subagent-driven-development | Replace human verification between tasks with automated test/lint gate. Retry once on failure, then escalate. |
| executing-plans | Replace "WAIT for human approval" with automated verification suite. |
| dispatching-parallel-agents | Replace "demand human verification" with automated verification. |
| finishing-a-development-branch | Always create PR (remove 4-option menu). Run tests before PR. |
| using-git-worktrees | Auto-select worktree location (parent of cwd). Remove user prompt. |
| writing-plans | Auto-proceed to execution after plan written. Remove "ready to execute?" prompt. |
| systematic-debugging | Replace "discuss with human after 3 failures" with escalate-and-stop. |
| end-of-day-report | Default to full report format. Remove format choice prompt. |
| end-of-week-report | Default to full report format. Remove format choice prompt. |
| verification-before-completion | Run automated checks. Remove "show to human" presentation. |
| receiving-code-review | Auto-assess review clarity. Escalate only on true ambiguity. |

## Escalation Pattern

When an agent hits a situation that would normally require human judgment:

1. Write `docs/agent-escalations/YYYY-MM-DD-<topic>.md` with context and options
2. Mark current task as blocked
3. Continue with other independent tasks if any remain
4. Never guess on ambiguous requirements

## Automated Verification Gate

Replaces human verification everywhere:

```
1. Run test suite (language-appropriate: npm test, pytest, go test, etc.)
2. Run linter if configured
3. If both pass → proceed
4. If fail → dispatch fix subagent once
5. If still fails → escalate
```
