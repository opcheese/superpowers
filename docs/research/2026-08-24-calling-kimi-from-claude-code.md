---
title: Calling Kimi from Claude Code — what exists, and what actually works
status: complete
last_verified: 2026-08-24
owner: opcheese
area: research
audience: maintainers
---

# Calling Kimi from Claude Code — what exists, and what actually works

Claude Code is the primary tool; a Kimi Code subscription sits alongside it,
unused from inside Claude Code. The question is how to delegate work to Kimi
without hand-rolling a wrapper — using the agent machinery both tools already
ship rather than inventing a bridge.

The short answer: **two paths work, they are complementary, and neither needs
new code.** Everything below was verified by running it, not read off a docs
page. Where something is inferred rather than executed, it says so.

## What was examined

Local environment on 2026-08-24: Kimi Code CLI, upgraded `0.18.0 → 0.38.0`
during this research; Claude Code as the calling harness; a Kimi Code
subscription authenticated by managed OAuth against `api.kimi.com/coding/v1`
(provider `managed:kimi-code`, default model `kimi-code/k3`, also
`kimi-for-coding` and a `-highspeed` variant).

Tested directly: headless prompt mode, unattended tool use, unattended
writes, permission-flag combinations, agent-scoped tool restriction,
`stream-json` output shape, and the Anthropic-compatible HTTP endpoint.
Read from docs: Kimi's agent frontmatter field list and discovery order.

No credentials appear in this document. The subscription token lives in
`~/.kimi-code/credentials/kimi-code.json`; every example below reads it at
call time or uses a placeholder.

## The constraint that shapes everything

**Claude Code cannot route a subagent to a non-Anthropic provider.** The
`model:` field in subagent frontmatter accepts `sonnet` / `opus` / `haiku` /
`inherit`, all resolving to the same endpoint, and provider configuration
(`ANTHROPIC_BASE_URL`) is session-wide. Per-agent provider routing is an open
feature request ([anthropics/claude-code#38698][issue]) with no maintainer
response; the only workaround discussed there is running separate terminal
sessions, which discards the orchestrator-to-subagent relationship that makes
delegation worth doing.

So there is no such thing as "a Claude Code subagent that runs on Kimi" in the
native sense. The provider boundary is crossed by spawning a process. The only
real design question is **where the structure lives** — and that is what the
two paths differ on.

## What does not exist

Two MCP servers advertise exactly this pattern. Both fail the
small-or-dead test and neither is adoptable:

- **agent-link-mcp** — MIT, 7 stars, 4 forks. Entire commit history is a
  two-day burst in March 2026; last push 2026-03-24, five months stale.
- **agent-delegation-mcp** — MIT, 0 stars, 0 forks. Created and last pushed
  the same day, 2026-08-14. A single-day project with no usage.

MCP remains the *supported* extension point for "Claude Code calls an external
system", but with nothing worth adopting, writing a server is the invention
this research set out to avoid. It becomes worthwhile only if Kimi should be
reachable from many MCP clients, not just Claude Code.

## Path A — Kimi's agent system

Structure lives on Kimi's side. Claude Code shells out to one well-defined
Kimi agent; Kimi's own machinery handles containment and any further
delegation.

Kimi 0.38.0 defines agents as markdown with YAML frontmatter:

| Field | Meaning |
| --- | --- |
| `description` | Required. Shown to the main agent when picking a sub-agent |
| `name` | Kebab-case id; defaults to the filename |
| `whenToUse` | Extra guidance on applicability |
| `tools` | Allowlist. YAML list or comma-separated string |
| `disallowedTools` | Denylist, same syntax |
| `subagents` | Allowlist of sub-agents this agent may delegate to |
| `override` | May replace a same-name built-in |

The body is the system prompt. MCP tools match by glob (`mcp__github__*`). An
empty `tools` list disables all tools; omitting the field allows everything.

Discovery runs most-specific-first: `--agent-file` > project
(`.kimi-code/agents/`, `.agents/agents/`) > `extra_agent_dirs` > user
(`$KIMI_CODE_HOME/agents/`, `~/.agents/agents/`) > plugin > built-in.

Three built-ins matter: `coder` (general, can write), **`explore`** (read-only,
never modifies files), **`plan`** (no shell at all).

Note that `~/.agents/agents/` shares the `~/.agents/` root that the
shadow-learn memory store already uses. The two conventions converge without
being coordinated.

## Path B — Claude Code's agent system, Kimi's brain

Structure lives on Claude Code's side. The subscription exposes an
**Anthropic-compatible** endpoint, so `claude` itself can be pointed at Kimi
while keeping its own agent definitions, permission system and tool
allowlists:

```
claude -p --settings '{"env":{"ANTHROPIC_BASE_URL":"https://api.kimi.com/coding","ANTHROPIC_AUTH_TOKEN":"<from apiKeyHelper>","ANTHROPIC_MODEL":"k3"}}' \
       --agent kimi-reviewer --allowedTools Read Grep Glob
```

Every piece is a shipped flag: `--settings` takes a file *or* a JSON string,
`settings.json` supports an `env` block, and `--agent`, `--agents`,
`--allowedTools` / `--disallowedTools`, `-p` and `--output-format stream-json`
all exist.

**The token cannot be hardcoded.** The OAuth access token carries roughly a
30-minute TTL and ships with a `refresh_token`. A settings file with a literal
token is stale within the hour and is a secret at rest besides. Claude Code's
`apiKeyHelper` setting is the correct seam: a command that returns a fresh
credential per call, reading and refreshing from the Kimi credential file.
*Writing that helper is the one piece of work Path B still needs; it was not
built or tested during this research.*

## Choosing between them

| | Path A — Kimi's agents | Path B — Claude Code's agents |
| --- | --- | --- |
| Invocation | `kimi -p --agent <name>` via Bash | `claude -p --agent <name>` + `--settings` |
| Containment | Kimi `tools` / `disallowedTools`; `explore`, `plan` | Claude Code's permission system |
| Agent definitions | `.kimi-code/agents/`, `~/.agents/agents/` | `.claude/agents/*.md` |
| Orchestration | Kimi's own sub-agents | Claude Code's |
| Auth | Working today | Working, pending `apiKeyHelper` |
| New code | None | The credential helper |

They are not alternatives. Path B gives a Kimi-powered agent that lives in the
normal Claude Code setup and obeys its permissions — the better fit for an
independent reviewer. Path A hands a task to Kimi and lets *it* orchestrate
its own sub-agents, returning only a result — the better fit for bulk work
that should not spend Claude quota. **Both are wanted, for flexibility and
control.**

## Verified behaviour

Everything in this table was executed.

| Check | Result |
| --- | --- |
| `kimi -p` on the subscription | Works. ~9–11s round trip, exit 0 |
| Unattended tool use in `-p` | Yes — read a file and diagnosed a planted bug |
| **Unattended writes in `-p`** | **Yes, no approval gate.** Edited code and created a file |
| `-p` with `--yolo` / `--auto` / `--plan` | **Rejected** — "Cannot combine". `-p` is already effectively yolo |
| `-p --agent explore`, told to write | **Refused.** Reported findings instead; files verified unchanged |
| `--output-format stream-json` | One JSON object per line: `{"role":"assistant","content":…}` plus a `meta` / `session.resume_hint` line |
| Anthropic endpoint + subscription OAuth | **HTTP 200.** Well-formed Anthropic `message`, includes `thinking` blocks |
| `--agent` / `--agent-file` on 0.18.0 | Absent. Present on 0.38.0 |

The two bolded rows are the point. Headless Kimi with no agent specified will
modify a repository unasked, and `--yolo` is refused precisely *because* `-p`
already behaves that way. Naming a read-only agent is what contains it — which
is why Path A depends on 0.38.0 and why an unqualified `kimi -p` should never
be pointed at a working tree.

## Operational notes

**Upgrading is not self-service.** `kimi upgrade` misdetects a native Linux
install as `native (windows)` and refuses, printing the install-script
fallback instead. The script (`code.kimi.com/kimi-code/install.sh`) was read
before running: it contacts only `code.kimi.com`, resolves a version, verifies
a SHA256 from a manifest, installs unprivileged to `$HOME/.kimi-code` unless
`KIMI_INSTALL_DIR` says otherwise, backs up the previous binary to
`bin/kimi.bak`, and guards its shell-rc PATH edit with `grep -qsF` so it is
idempotent. Expect the same misdetection on future upgrades.

**Config survives the upgrade**, but 0.38.0 deprecates
`[loop_control] max_retries_per_step` in favour of `max_attempts_per_step`.
`kimi doctor` reports it and is the fastest way to confirm a clean config.

**Other Kimi surfaces, for completeness.** `kimi acp` speaks Agent Client
Protocol over stdio for Zed and JetBrains — Claude Code is not an ACP client,
so it is not a route here. `kimi server` runs a local REST + WebSocket daemon.
Kimi is an MCP *client*, not a server, so it cannot be attached to Claude Code
as one.

## Open questions

- The `apiKeyHelper` credential helper for Path B is unwritten and untested.
  Whether the refresh flow can be driven without the Kimi CLI itself holding
  the lock on the credential file is unknown.
- Concurrency: the subscription is documented as capped at 30 concurrent
  requests, which bounds any fan-out design. Not tested.
- Whether Kimi agent definitions should be version-controlled per project
  (`.kimi-code/agents/`) or kept user-global (`~/.agents/agents/`) is a
  distribution question, unresolved, and it interacts with how the catalog
  ships skills to devs.

[issue]: https://github.com/anthropics/claude-code/issues/38698

## Sources

- [Kimi Code — Agents and Sub-Agents](https://www.kimi.com/code/docs/en/kimi-code-cli/customization/agents.html)
- [Kimi Code — `kimi` command reference](https://www.kimi.com/code/docs/en/kimi-code-cli/reference/kimi-command)
- [Kimi Code — `kimi acp`](https://www.kimi.com/code/docs/en/kimi-code-cli/reference/kimi-acp.html)
- [Kimi Code — Model Context Protocol](https://www.kimi.com/code/docs/en/kimi-code-cli/customization/mcp.html)
- [Claude Code — Create custom subagents](https://code.claude.com/docs/en/sub-agents)
- [anthropics/claude-code#38698 — per-agent model provider routing](https://github.com/anthropics/claude-code/issues/38698)
