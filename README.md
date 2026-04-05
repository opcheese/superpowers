# Superpowers — Agents Branch

> **This is the `agents` branch** — adapted for unattended Claude Code sessions (`claude -p` in CI/pipelines/cron). No human-in-the-loop required. For the interactive version with human oversight, see the `main` branch.

Superpowers is a complete software development workflow for coding agents, built on composable "skills" that trigger automatically.

## What's different on this branch

This branch removes all human oversight gates and interactive prompts, replacing them with **automated verification** and **escalation patterns** suitable for autonomous operation.

### Key changes from `main`

| Area | `main` (interactive) | `agents` (autonomous) |
|------|---------------------|----------------------|
| **Quality gates** | "Demand human verification" | Automated: run tests + linter, retry once, escalate on failure |
| **Brainstorming** | Interactive Q&A, one question at a time, visual companion | Single-pass design generation with self-review via subagent |
| **Finishing work** | Present 4 options (merge/PR/keep/discard) | Always create PR (safest default for unattended work) |
| **Ambiguity** | Ask the user | Escalate to `docs/agent-escalations/` and continue with independent tasks |
| **Reports** | Ask user for format preference | Default to full report format |
| **Worktrees** | Ask user for directory preference | Auto-select `.worktrees/` |
| **Debugging** | "Discuss with human after 3 failures" | Escalate and stop |
| **Visual features** | Browser-based visual companion for brainstorming | Removed (server.js, WebSocket, HTML frames) |

### New skill: `escalation`

When an agent hits ambiguity, repeated failures, or decisions requiring human judgment:
1. Write `docs/agent-escalations/YYYY-MM-DD-<topic>.md` with context and options
2. Mark current task as blocked
3. Continue with other independent tasks

### Modified skills (12)

- **brainstorming** — single-pass design, no interactive Q&A, no visual companion
- **subagent-driven-development** — automated verification gate replaces human review between tasks
- **executing-plans** — automated verification replaces human approval
- **dispatching-parallel-agents** — automated verification replaces human demand
- **finishing-a-development-branch** — always create PR, no option menu
- **using-git-worktrees** — auto-select directory
- **writing-plans** — auto-proceed to execution
- **systematic-debugging** — escalate-and-stop replaces "discuss with human"
- **end-of-day-report** — default full report format
- **end-of-week-report** — default full report format
- **verification-before-completion** — automated checks only
- **receiving-code-review** — auto-assess clarity, escalate true ambiguity

### Deleted files

- `skills/brainstorming/scripts/server.js` — WebSocket server
- `skills/brainstorming/scripts/helper.js` — client-side JS
- `skills/brainstorming/scripts/frame-template.html` — HTML frame
- `skills/brainstorming/visual-companion.md` — visual companion guide

## How it works

The agent analyzes the task, generates a design spec, self-reviews it via subagent, creates an implementation plan, then executes it with fresh subagents per task — each going through spec compliance review, code quality review, and automated test verification before proceeding.

Escalation files in `docs/agent-escalations/` collect anything the agent couldn't resolve autonomously for later human review.

## Sponsorship

If Superpowers has helped you do stuff that makes money and you are so inclined, I'd greatly appreciate it if you'd consider [sponsoring my opensource work](https://github.com/sponsors/obra).

Thanks!

- Jesse


## Installation

**Note:** Installation differs by platform. Claude Code or Cursor have built-in plugin marketplaces. Codex and OpenCode require manual setup.

### Claude Code Official Marketplace

Superpowers is available via the [official Claude plugin marketplace](https://claude.com/plugins/superpowers)

Install the plugin from Claude marketplace:

```bash
/plugin install superpowers@claude-plugins-official
```

### Claude Code (via Plugin Marketplace)

In Claude Code, register the marketplace first:

```bash
/plugin marketplace add obra/superpowers-marketplace
```

Then install the plugin from this marketplace:

```bash
/plugin install superpowers@superpowers-marketplace
```

### Cursor (via Plugin Marketplace)

In Cursor Agent chat, install from marketplace:

```text
/add-plugin superpowers
```

or search for "superpowers" in the plugin marketplace.

### Codex

Tell Codex:

```
Fetch and follow instructions from https://raw.githubusercontent.com/obra/superpowers/refs/heads/main/.codex/INSTALL.md
```

**Detailed docs:** [docs/README.codex.md](docs/README.codex.md)

### OpenCode

Tell OpenCode:

```
Fetch and follow instructions from https://raw.githubusercontent.com/obra/superpowers/refs/heads/main/.opencode/INSTALL.md
```

**Detailed docs:** [docs/README.opencode.md](docs/README.opencode.md)

### GitHub Copilot CLI

```bash
copilot plugin marketplace add obra/superpowers-marketplace
copilot plugin install superpowers@superpowers-marketplace
```

### Gemini CLI

```bash
gemini extensions install https://github.com/obra/superpowers
```

To update:

```bash
gemini extensions update superpowers
```

### Verify Installation

Start a new session in your chosen platform and ask for something that should trigger a skill (for example, "help me plan this feature" or "let's debug this issue"). The agent should automatically invoke the relevant superpowers skill.

## The Basic Workflow

1. **brainstorming** - Activates before writing code. Refines rough ideas through questions, explores alternatives, presents design in sections for validation. Saves design document.

2. **using-git-worktrees** - Activates after design approval. Creates isolated workspace on new branch, runs project setup, verifies clean test baseline.

3. **writing-plans** - Activates with approved design. Breaks work into bite-sized tasks (2-5 minutes each). Every task has exact file paths, complete code, verification steps.

4. **subagent-driven-development** or **executing-plans** - Activates with plan. Dispatches fresh subagent per task with two-stage review (spec compliance, then code quality), or executes in batches with human checkpoints.

5. **test-driven-development** - Activates during implementation. Enforces RED-GREEN-REFACTOR: write failing test, watch it fail, write minimal code, watch it pass, commit. Deletes code written before tests.

6. **requesting-code-review** - Activates between tasks. Reviews against plan, reports issues by severity. Critical issues block progress.

7. **finishing-a-development-branch** - Activates when tasks complete. Verifies tests, presents options (merge/PR/keep/discard), cleans up worktree.

**The agent checks for relevant skills before any task.** Mandatory workflows, not suggestions.

## What's Inside

### Skills Library

**Testing**
- **test-driven-development** - RED-GREEN-REFACTOR cycle (includes testing anti-patterns reference)

**Debugging**
- **systematic-debugging** - 4-phase root cause process (includes root-cause-tracing, defense-in-depth, condition-based-waiting techniques)
- **verification-before-completion** - Ensure it's actually fixed

**Collaboration** 
- **brainstorming** - Socratic design refinement
- **writing-plans** - Detailed implementation plans
- **executing-plans** - Batch execution with checkpoints
- **dispatching-parallel-agents** - Concurrent subagent workflows
- **requesting-code-review** - Pre-review checklist
- **receiving-code-review** - Responding to feedback
- **using-git-worktrees** - Parallel development branches
- **finishing-a-development-branch** - Merge/PR decision workflow
- **subagent-driven-development** - Fast iteration with two-stage review (spec compliance, then code quality)

**Meta**
- **writing-skills** - Create new skills following best practices (includes testing methodology)
- **using-superpowers** - Introduction to the skills system

## Philosophy

- **Test-Driven Development** - Write tests first, always
- **Systematic over ad-hoc** - Process over guessing
- **Complexity reduction** - Simplicity as primary goal
- **Evidence over claims** - Verify before declaring success

Read more: [Superpowers for Claude Code](https://blog.fsck.com/2025/10/09/superpowers/)

## Contributing

Skills live directly in this repository. To contribute:

1. Fork the repository
2. Create a branch for your skill
3. Follow the `writing-skills` skill for creating and testing new skills
4. Submit a PR

See `skills/writing-skills/SKILL.md` for the complete guide.

## Updating

Skills update automatically when you update the plugin:

```bash
/plugin update superpowers
```

## License

MIT License - see LICENSE file for details

## Community

Superpowers is built by [Jesse Vincent](https://blog.fsck.com) and the rest of the folks at [Prime Radiant](https://primeradiant.com).

- **Discord**: [Join us](https://discord.gg/Jd8Vphy9jq) for community support, questions, and sharing what you're building with Superpowers
- **Issues**: https://github.com/obra/superpowers/issues
- **Release announcements**: [Sign up](https://primeradiant.com/superpowers/) to get notified about new versions
