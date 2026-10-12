---
date: 2026-10-12
last_verified: 2026-10-12
status: complete
---

# Codex vs Claude Code: the harnesses, October 2026

This compares the two tools as **harnesses**: memory, multi-agent work, review, session-to-session communication, automation and extensibility. Models are left out; for model choice see `2026-10-12-theo-t3dotgg-tooling-digest.md`.

**Versions compared:** Claude Code **2.1.296** (Oct 9 2026, installed here) and Codex CLI **0.162.x** (Oct 8–9 2026) plus the ChatGPT desktop app, which now hosts Codex. Sources are the official docs as fetched on 2026-10-12: [code.claude.com/docs](https://code.claude.com/docs/llms.txt) and [learn.chatgpt.com](https://learn.chatgpt.com/llms.txt). The old developers.openai.com/codex pages redirect to learn.chatgpt.com. Copies of the fetched pages are in the gitignored `.superpowers/research/harness/`. Third-party comparison posts are not used for any fact here, because most still describe GPT-5.x / Opus 4.x setups.

## The one-paragraph answer

**Claude Code is the deeper agent-orchestration and extensibility platform. Codex is the more polished, sandbox-first product that is merging into the ChatGPT app.**

- **Claude Code** has five distinct ways to run agents in parallel, including scripted *dynamic workflows* and experimental agent teams. Only Claude Code has first-class **cross-session messaging**: sessions on one machine, other machines or the cloud can message each other. It also has a far richer hook surface (33 events), plus in-process *mods*. Its auto-memory is **on** by default.
- **Codex** ships an OS-level sandbox that is on by default, with an AI **auto-reviewer** that can approve sandbox escapes. It has a cleaner `/goal` and worktree UX, a strong mobile app, SSH remote hosts, and computer-use/browser built into the desktop app. It reads Claude-format plugin hooks for compatibility and can `/import` a Claude Code setup. Its memory is **off** by default.

For our skills (the superpowers fork, codebase-vocabulary, kimi-delegation), Claude Code is still the primary target. Codex is a capable second harness, and the porting cost is falling.

## Side by side

| Area | Claude Code 2.1.296 | Codex 0.162 / ChatGPT app |
|---|---|---|
| **Surfaces** | Terminal, VS Code/JetBrains, desktop app, claude.ai/code (web, cloud sessions), mobile through Remote Control, Slack ("Claude Tag") | CLI (TUI), IDE extension, ChatGPT desktop app (macOS, Windows, Linux), ChatGPT web ("Work"), iOS app, GitHub Action, SDK |
| **Project instructions** | `CLAUDE.md` (layered user/project/local/managed), plus `.claude/rules/` with path-scoped rules. Reads `AGENTS.md` only when no `CLAUDE.md` exists. | `AGENTS.md`, layered with overrides. `/init` scaffolds one. |
| **Memory** | **Auto memory on by default.** Claude writes typed notes (user/feedback/project/reference) to `~/.claude/projects/<repo>/memory/`, with a `MEMORY.md` index loaded each session. Subagents can have their own persistent memory (`memory: user\|project\|local`). Toggle with `/memory` or `autoMemoryEnabled`. | **Local memories off by default.** Once enabled, background passes summarize idle past chats into `~/.codex/memories/`. They skip active chats, redact secrets, and skip a pass when you are near your rate limit. `/memories` sets per chat whether it reads or feeds memory. The docs call it a "recall layer", not a place for rules. macOS-only *Computer History* turns app activity into memories. |
| **Subagents** | Custom agents in `.claude/agents/*.md`, each with its own tools, model, `effort`, `isolation: worktree`, `background`, `memory`, preloaded skills and `autoCompactWindow`. Forked subagents inherit full context (`/subtask`). Claude delegates on its own. | On by default. Built-in `default`, `worker` and `explorer`. Custom agents are TOML in `.codex/agents/` (`name`, `description`, `developer_instructions`, plus any config override such as model, effort or sandbox). The concurrency cap is `agents.max_concurrent_threads_per_session`. **Local Codex only delegates when asked**, or when AGENTS.md or a skill says to. `/agent` switches between threads. |
| **Bigger multi-agent modes** | **Agent view** (`claude agents`): a dashboard of background sessions, each moved into its own worktree. **Agent teams** (experimental, off by default): a lead plus teammates with a shared task list and direct messages between them. **Dynamic workflows**: a JS script Claude writes that fans out many subagents and cross-checks their results, for audits, 500-file migrations and multi-source research. `/batch` splits one change across 5–30 worktree subagents. | Parallel chats or tasks, each with a managed worktree (new in 0.162: worktree tools and pinning in the Command Center). Cloud tasks run in parallel in OpenAI containers. On ChatGPT web, the "Ultra" intelligence level delegates proactively. There is nothing like scripted workflows or agent teams. |
| **Long-running work** | `/goal`: after each turn a model checks the completion condition and keeps going until it holds. `/loop` for recurring prompts. Background sessions survive restarts and can be resumed by id. | `/goal` (Goal mode) sets outcome, constraints and verification, with pause, resume and edit controls. `/plan` comes first if the goal is fuzzy. |
| **Talking to other sessions** | **Cross-session messaging**: `SendMessage`/`ListAgents` reach your other sessions on this machine, on other machines, or in the cloud. `@session-name` mentions name a target, `notify_when_idle` asks for a notice when another session finishes, and inbound messages can be delivered, held or refused. **Channels** push MCP events (CI, chat, webhooks) into a running session. | **No message passing between sessions.** Subagents report only to their parent. You can **hand off** a named chat to another connected host (SSH or remote). `@`-mentioning another task in the composer (iOS, app) pulls it in as context. The app-server JSON-RPC API lets your own tooling drive threads. |
| **Remote and mobile** | Remote Control drives a local session from a phone or browser, with subagent and workflow progress kept in sync. Cloud sessions on claude.ai/code. *Projects* (beta) is one long-running conversation that starts parallel threads in the cloud or on your machine. | Remote connections: a phone controls a desktop host, Codex connects to SSH hosts, chats hand off between hosts. A strong iOS app with widgets, live approvals and Live Activities for worktree setup. Codex Cloud runs tasks in hosted environments. |
| **Code review** | Local `/code-review` (effort levels, `--fix`, `--comment`, `--max-findings`). **`/code-review ultra`** runs a fleet of reviewer agents in a cloud sandbox and reproduces every finding before reporting it. **Managed Code Review** for GitHub PRs (Team/Enterprise, research preview) posts severity-tagged inline comments and is triggered by `@claude review`, tunable with `REVIEW.md`. GitHub Actions and GitLab CI for self-hosted runs. | `/review` reviews the working tree or a base branch (choose the model with `review_model`). The desktop **Code Review** pane shows PRs, diffs, checks and inline comments, with staging and reverting. **Automatic cloud reviews** on GitHub PRs and `@codex review`, with GitLab MRs in preview. |
| **Safety and permissions** | Permission modes include **auto mode**, where a classifier approves actions. The OS sandbox is **off by default** (`/sandbox` turns it on). Settings hooks can block anything. | Sandbox is **on by default** (`read-only` / `workspace-write` / full access). **Auto-review** sends sandbox-escape approval requests to a separate reviewer agent instead of a human, and denials can be retried with `/approve`. Hooks must be explicitly *trusted* before they run. |
| **Hooks** | 33 settings-hook events, including `PreToolUse`, `PermissionRequest`, `SubagentStart/Stop`, `TaskCreated/Completed`, `TeammateIdle`, `WorktreeCreate`, `FileChanged`, `Pre/PostModelSwitch` and `Elicitation`. Hooks can be command, HTTP, prompt or agent hooks; `onFailure: "block"` was added in 2.1.295. | 13 events: `SessionStart`/`SessionEnd`, `SubagentStart`/`SubagentStop`, `Pre`/`PostToolUse`, `PermissionRequest`, `Pre`/`PostCompact`, `UserPromptSubmit`, `Stop`, `Interrupt`, plus MCP-tool hooks. Hooks can run in the background. Managed hooks come from `requirements.toml`. |
| **Extensibility** | Plugins (skills, agents, hooks, MCP, commands) from git marketplaces. **Mods** are TS/JS handlers that run inside Claude Code and can add panes, commands, tool rules or a side agent ("You should know"). `claude plugin eval` and `validate`. The Agent SDK is for building your own agents. | Plugins from marketplaces, with a universal plugin directory and workspace-synced GitHub marketplaces. **Skills follow the agentskills.io standard (`SKILL.md`), the same format as Claude Code.** Plugin hooks get `CLAUDE_PLUGIN_ROOT`/`CLAUDE_PLUGIN_DATA` for compatibility. `/import` brings in a Claude Code or Cursor setup plus up to 50 recent chats. Codex SDK (TS/Python) and the app server. The old `codex mcp-server` was **removed Sep 5**. |
| **Scheduled and triggered work** | **Routines**: cloud-run sessions on a schedule, on an API call or on GitHub events. Desktop scheduled tasks. Channels for live events. | Scheduled tasks in the app. **Event triggers** (Gmail, Slack, GitHub PR activity) run on ChatGPT web and mobile only, not in the CLI. |
| **Speed tiers** | Fast mode (`/fast`). | `/fast` uses subscription limits at 2.5× the normal rate. **Ultrafast** (Pro $500, Enterprise/Edu) uses them at 8×. |
| **Beyond code** | Artifacts (published pages), Claude Tag in Slack, desktop computer use. | Built-in browser and Chrome extension, computer use, Appshots, image generation, Sites, "dots" (a persistent personal agent), Pets. The app is becoming a general work agent. |

## What actually differs in practice

**Memory, philosophically opposite defaults.** Claude Code learns by default and curates a typed, indexed memory per repo. Codex leaves memory off and treats it, when on, as lossy background recall. Theo's "Turn off Claude Code's Memory" (Aug 25) argues Codex's default is right for coding: he found most auto-memories on his machine unread or stale. Our fork's memory files are curated by hand, which avoids the problem he describes. Teams that don't curate should either audit `~/.claude/projects/*/memory/` or set `autoMemoryEnabled: false` per repo.

**Multi-agent depth is Claude Code's clearest lead.** Codex has competent subagents and parallel worktree tasks. Claude Code has three more layers on top of that:

- scripted workflows that cross-check results, which is what you want for audits;
- teams that message each other directly;
- cross-session messaging, so independent sessions, including ones on other machines, can coordinate.

Codex's answer to "make two sessions talk" is handoff or your own tooling on the app server.

**Review is close, but shaped differently.** Both have a local review command and GitHub PR bots. Claude's `ultra` mode verifies each finding by reproducing it, so it reports fewer false positives. Codex puts review in the desktop app (a PR inbox, diffs, staging) and adds *auto-review* for safety approvals, which is a different thing from code review. Theo's advice holds whichever harness you use: **review with a different model from the one that wrote the code.**

**Safety posture.** Codex sandboxes by default and can let an AI reviewer approve escapes, which suits unattended runs. Claude Code relies on permission modes and hooks, with the sandbox opt-in. For our unattended `agents` spine, Claude Code users should turn on `/sandbox` or keep tight hooks.

**Porting skills is getting cheaper.** Both read `SKILL.md`. Codex runs Claude-style plugin hooks through the `CLAUDE_PLUGIN_ROOT` aliases. The remaining gaps for superpowers on Codex are the bootstrap injection (our existing Codex reference, `skills/using-superpowers/references/codex-tools.md`) and features with no Codex equivalent: `SendMessage`, workflows, agent teams, mods and most of the 33 hook events.

## Recommendation for the team

- **Primary harness: Claude Code**, for the orchestration features, the ecosystem our plugins target, and Opus 5.5.
- **Keep Codex on hand for a second opinion and specific jobs:** GPT-6.1 Sol reviews and audits (cheap per task), computer-use and browser work in the ChatGPT app, and mobile-first task management. If you use it, turn on memories only deliberately and turn off "improve the model for everyone".
- **Cursor users:** OpenAI models leave Cursor on **Nov 12 2026**. Move to direct subscriptions before then.

## Open questions

- Whether Codex's `max_concurrent_threads_per_session` default changed in 0.162 is undocumented; one third-party post claims 8 parallel cloud sandboxes, which we have not verified.
- We haven't tested the superpowers bootstrap on Codex 0.162. It needs the CLAUDE.md acceptance test ("Let's make a react todo list" → brainstorming triggers) before we claim support.
