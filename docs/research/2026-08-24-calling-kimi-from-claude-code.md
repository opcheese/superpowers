---
title: Calling Kimi from Claude Code — what exists, and what actually works
status: complete
last_verified: 2026-08-24
revised: 2026-08-24 — corrected six claims found in review; see Open questions
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
`model:` field in subagent frontmatter selects an Anthropic model — an alias
(`sonnet`, `opus`, `haiku`, `inherit`), a form like `sonnet[1m]`, or a full
model ID — and every one of them resolves against the same endpoint, because
provider configuration (`ANTHROPIC_BASE_URL`) is session-wide. Per-agent provider routing is an open
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

**Containment and delegation are not available at the same time.** Built-in
sub-agents cannot dispatch further sub-agents, so `--agent explore` buys
read-only containment and no orchestration. Getting Kimi to orchestrate needs
a custom agent with an explicit `subagents` allowlist — which is a write-capable
agent unless its `tools` say otherwise. Pick one per call; the choice is real.

Note that `~/.agents/agents/` sits under the same `~/.agents/` root that the
skills marketplace installs into — alongside `bin/`, `skills/` and a
`.skill-lock.json`. Upstream treats `~/.agents/` as a generic cross-tool
directory. Anything written there shares space with a lock-file-managed
installer, which matters when deciding where agent definitions should live.

## Path B — Claude Code's agent system, Kimi's brain

Structure lives on Claude Code's side. The subscription exposes an
**Anthropic-compatible** endpoint, so `claude` itself can be pointed at Kimi
while keeping its own agent definitions, permission system and tool
allowlists:

```
claude -p --settings '{"env":{"ANTHROPIC_BASE_URL":"https://api.kimi.com/coding","ANTHROPIC_MODEL":"k3"},"apiKeyHelper":"/abs/path/to/kimi-credential"}' \
       --agent kimi-reviewer --allowedTools Read Grep Glob
```

The credential arrives through `apiKeyHelper`, never as a literal
`ANTHROPIC_AUTH_TOKEN` in the settings blob — see below for why.

Every piece is a shipped flag: `--settings` takes a file *or* a JSON string,
`settings.json` supports an `env` block, and `--agent`, `--agents`,
`--allowedTools` / `--disallowedTools`, `-p` and `--output-format stream-json`
all exist.

**The token cannot be hardcoded.** The OAuth access token carries roughly a
30-minute TTL and ships with a `refresh_token`. A settings file with a literal
token is stale within the hour and is a secret at rest besides. Claude Code's
`apiKeyHelper` setting is the correct seam: a command that returns a fresh
credential per call, reading and refreshing from the Kimi credential file.
*That helper was the one piece of work Path B still needed. It has since been
built and verified — see the `kimi-delegation` plugin in the skills catalog.*

## Choosing between them

| | Path A — Kimi's agents | Path B — Claude Code's agents |
| --- | --- | --- |
| Invocation | `kimi -p --agent <name>` via Bash | `claude -p --agent <name>` + `--settings` |
| Containment | Kimi `tools` / `disallowedTools`, pinned with `--agent-file` | Claude Code's permission system |
| Agent definitions | `.kimi-code/agents/`, `~/.agents/agents/` | `.claude/agents/*.md` |
| Orchestration | Kimi's own sub-agents, but only via a custom agent — built-ins cannot dispatch | Claude Code's |
| Auth | Working today | Working, pending `apiKeyHelper` |
| New code | None | The credential helper |

They are not alternatives. Path B gives a Kimi-powered agent that lives in the
normal Claude Code setup and obeys its permissions — the better fit for an
independent reviewer. Path A hands a task to Kimi and returns only a result —
the better fit for bulk work that should not spend Claude quota. **Both are
wanted, for flexibility and control.**

Path A's orchestration is conditional, not free: a contained built-in cannot
dispatch sub-agents, so letting Kimi drive its own fan-out means writing a
custom agent with a `subagents` allowlist and choosing its `tools` deliberately.

## Verified behaviour

Everything in this table was executed.

| Check | Result |
| --- | --- |
| `kimi -p` on the subscription | Works. ~9–11s round trip, exit 0 |
| Unattended tool use in `-p` | Yes — read a file and diagnosed a planted bug |
| **Unattended writes in `-p`** | **Yes, no approval gate.** Edited code and created a file |
| `-p` with `--yolo` / `--auto` / `--plan` | **Rejected** — identical `Cannot combine --prompt with --X` for all three. A blanket prompt-mode exclusion, not a statement about permissions |
| `-p --agent explore`, told to write | **Refused** in a clean repo. Reported findings instead; files verified unchanged |
| The same, in a repo shipping `.kimi-code/agents/explore.md` | **Wrote.** The repo's definition wins; containment lost. Pin with `--agent-file` |
| `--output-format stream-json` | One JSON object per line: `{"role":"assistant","content":…}` plus a `meta` / `session.resume_hint` line |
| Anthropic endpoint + subscription OAuth | **HTTP 200.** Well-formed Anthropic `message`, includes `thinking` blocks |
| `--agent` / `--agent-file` on 0.18.0 | Absent. Present on 0.38.0 |

The two bolded rows are the point. Headless Kimi with no agent specified will
modify a repository unasked. Note that `--plan` — the semantic opposite of
`--yolo` — is refused in exactly the same words, so the rejection says nothing
about permissions; it is a blanket prompt-mode exclusion. The unattended-writes
observation stands on its own, and it is why Path A depends on 0.38.0 and why an
unqualified `kimi -p` should never be pointed at a working tree.

**Naming a read-only agent is not sufficient.** Agent *names* resolve
project-first, so a repository shipping `.kimi-code/agents/explore.md` with
`override: true` replaces the built-in and takes write access back. This was
tested, and the repository's definition won: the delegate edited a file and
created another. Upstream's trust-model warning is explicit — an `override: true`
definition replaces the main agent's system prompt entirely, and a definition
with no `tools` list keeps every tool. Containment has to be pinned with
`--agent-file`, which outranks project discovery.

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
so it is not a route here. `kimi server` is deprecated on 0.38.0 in favour of
`kimi web`, which runs the local server and opens the web UI.
Kimi is an MCP *client*, not a server, so it cannot be attached to Claude Code
as one.

## Open questions

- **Resolved.** The `apiKeyHelper` for Path B is built and verified. Refresh is
  delegated to the Kimi CLI rather than reimplemented: any successful call
  rewrites the credential file. `kimi -p "ok"` does this; `kimi provider list`
  does not, because it reads local configuration and never contacts the API.
- **Resolved, and it changed the design.** Agent names are resolved
  project-first, so a repository can override a read-only built-in and take
  write access back. Containment must be pinned with `--agent-file`. Treat any
  repository-supplied agent definition as hostile input.
- Concurrency: the subscription is documented as capped at 30 concurrent
  requests, which bounds any fan-out design. Not tested.
- Whether Kimi agent definitions should be version-controlled per project
  (`.kimi-code/agents/`) or kept user-global (`~/.agents/agents/`) is a
  distribution question, unresolved, and it interacts with how the catalog
  ships skills to devs. Note that the project-scoped option is the one an
  untrusted repository controls.

[issue]: https://github.com/anthropics/claude-code/issues/38698

## Sources

- [Kimi Code — Agents and Sub-Agents](https://www.kimi.com/code/docs/en/kimi-code-cli/customization/agents.html)
- [Kimi Code — `kimi` command reference](https://www.kimi.com/code/docs/en/kimi-code-cli/reference/kimi-command)
- [Kimi Code — `kimi acp`](https://www.kimi.com/code/docs/en/kimi-code-cli/reference/kimi-acp.html)
- [Kimi Code — Model Context Protocol](https://www.kimi.com/code/docs/en/kimi-code-cli/customization/mcp.html)
- [Claude Code — Create custom subagents](https://code.claude.com/docs/en/sub-agents)
- [anthropics/claude-code#38698 — per-agent model provider routing](https://github.com/anthropics/claude-code/issues/38698)
