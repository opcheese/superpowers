# Spike: Claude Code fully on a local model

Date: 2026-10-12. Claude Code 2.1.296, Ollama 0.34.2, RTX 5090 (32 GB),
`superpowers-human` 7.0.0 plus both vocabulary plugins, in a separate
`CLAUDE_CONFIG_DIR` so the results reflect a teammate's setup rather than
this machine's 15 plugins. Follows
[the classifier research](2026-10-12-local-auto-mode-classifier.md).

The question: can a teammate run Claude Code with the team plugins on one
local model, auto mode included, with no traffic to Anthropic? Yes, with five
settings that a plain `ollama launch claude` doesn't apply.

## Rulings

**One model does everything.** Two models don't fit on the GPU at the same
time (the user's ruling), so the session, its subagents and the auto-mode
classifier all use the same model. Pointing
`ANTHROPIC_DEFAULT_SONNET_MODEL` at it also skips the classifier's failed
first request for `claude-sonnet-5`.

**Context must be set in Ollama, not in Claude Code.** Claude Code can't tell
Ollama the context size, and Ollama loaded `qwen3.8:27b` at 32k by default.
Prompts reached 32–33k tokens within a few steps (the classifier prompt
alone is ~30k), so 32k cuts them short. A copy of the model with `num_ctx
131072` fixes it. Memory and speed at 128k (40k-token prompt):

| Model | GPU memory at 128k | Prompt processing | Generation |
|---|---|---|---|
| `qwen3.8:27b` | 16.6 GB | 2,400 tok/s | 69 tok/s |
| `qwen3.6:35b-a3b` | 23.7 GB | 6,600 tok/s | 72 tok/s |
| `glm-4.7-flash` | 24.3 GB | 2,900 tok/s | 97 tok/s |
| `gemma4` (8B) | small | 12,000 tok/s | 81 tok/s |

**Claude Code must be told the context size too.** It doesn't know local
models. In interactive sessions it warns that it assumes 200k and compacts
only near that, which a 128k model would never reach before Ollama starts
cutting. `CLAUDE_CODE_MAX_CONTEXT_TOKENS=131072` fixes it and removes the
warning. `-p` runs don't print the warning, but the setting still applies.

**Use `qwen3.8:27b`.** Each model ran the team's acceptance test ("Let's
make a react todo list") and a small task (add `subtract` with a test
first, run pytest, commit):

| Model | Acceptance test | Task | Notes |
|---|---|---|---|
| `qwen3.8:27b` | brainstorming, v7 opening question | TDD skill, red then green, commit, 3 min | Also best classifier (16/16). Interactive run also passed |
| `qwen3.6:35b-a3b` | brainstorming | commit, 47 s | Fastest. As classifier it always needs stage 2 |
| `glm-4.7-flash` | brainstorming | commit, 68 s | Tried `git push` unasked |
| `gemma4` | **failed**: skipped to writing-plans | commit, extra test fixes | Tried `git push` unasked; literal `\n` in commit message |

The two unasked push attempts failed only because the test repos had no
remote. Auto mode allows pushing to the repo's own remote by default.

**Nothing leaves the machine except WebFetch's target.** We traced every
`connect()` of the Claude Code process with `strace` across `-p` runs, an
interactive run and the final shell function. With
`CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC=1` the only connections were to
Ollama, apart from one: **WebFetch sends each hostname to
`api.anthropic.com` first** (its domain safety check, `160.79.104.10:443` in
the trace). `skipWebFetchPreflight: true` removes it; after that WebFetch
connects only to the site it fetches.

**What works and what doesn't:**

- Works: skills and the spine, Bash/Read/Edit, subagents (the classifier
  reviews the dispatch and the report), auto mode, WebFetch.
- Doesn't work: WebSearch returns "Web search error: unavailable". It runs
  on Anthropic's side.
- Off, by design of the settings: claude.ai connectors (Claude Code warns
  that the API key takes precedence), auto-updates, telemetry.
- Not tested: `/code-review`, which the interactive spine asks for after
  each task, artifacts, and Remote Control.

## After review: the function was hardened

A review of the guide found that the spike's clean, logged-out config hid
two risks a real `~/.claude` has. Both were fixed and retested.

**A settings `env` block beats shell variables.** Claude Code writes each
settings `env` entry over the inherited environment (env-vars, "the settings
file value applies in most sessions"). A user or project settings file that
sets `ANTHROPIC_BASE_URL`, for example for Kimi, would silently redirect a
shell-variable `claude-local`. The function now passes everything as an
`env` block inside `--settings`, which outranks user and project settings.
Test: a project `.claude/settings.json` pointing at `127.0.0.1:9` with a
Claude model. The run used only the local model, and strace showed
connections only to Ollama.

**Code does leave through `git push`.** Auto mode is the default start mode
for terminal sessions from 2.1.283, and allows pushes to the repo's own
remote. The function adds `permissions.ask: ["Bash(git push:*)"]`. Test: a
repo with a local bare remote, asked to commit and push. The commit landed;
every push was held for approval, including one chained after `git add &&
git commit`, and the remote stayed empty.

The guide also no longer claims code never leaves the machine. It notes
that `skipWebFetchPreflight` turns off Anthropic's unsafe-domain blocklist,
and tells Linux users to use `systemctl`, not `ollama serve`.

## Reproducing

Everything is in the gitignored
`.superpowers/research/local-claude-spike/`: `progress.md` (timeline),
`session.sh` and `report.py` (`-p` runs through a logging proxy, under
`strace`), `interactive.py` (pty-driven interactive run, since tmux isn't
installed), `ctxbench.py` (context vs memory and speed), `modelfiles/`,
`claude-local-v2.sh` (the shipped shell function; `claude-local.sh` is the first version), `logs/` and `runs/` (each
run's repo, with its commits).
