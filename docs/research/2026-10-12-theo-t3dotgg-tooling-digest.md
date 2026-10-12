---
date: 2026-10-12
last_verified: 2026-10-12
status: complete
---

# Theo (t3.gg) on AI coding tools, Aug 25 – Oct 12 2026

A digest of what Theo Browne's channel ([@t3dotgg](https://www.youtube.com/@t3dotgg/videos)) said about Codex, Claude Code and the models behind them in the last seven weeks, boiled down to what a team should use and change.

## How this was made, and how far to trust it

- The 31 newest uploads were listed with `yt-dlp`. The 22 about tooling, models or agent workflow got their English auto-captions pulled and converted to text. Skipped: AI-policy and safety videos, the NVIDIA chip video, and the React Native one.
- Each transcript was summarized by a local model (`qwen3.6:35b-a3b` on Ollama). The claims below that the team would act on were then checked against the raw transcript text. The local model got some names wrong: it rendered GPT-6.1 Sol as "Soul" or "Sonnet 61", and it said *Anthropic* paused new $200 Codex sign-ups when Theo meant OpenAI. Those are fixed here.
- Raw captions, plain-text transcripts and per-video summaries are kept in the gitignored `.superpowers/research/t3dotgg/` (`txt/`, `sum/`, `dates.txt`). Re-run with `summarize.py` there.
- **Bias to keep in mind.** Theo builds and sells **T3 Code**, his own agent IDE, and most of his workflow demos run in it. Every video carries sponsor segments (Browserbase, WorkOS, Parallel, CodeRabbit, Greptile and others). His statements about limits are his own back-of-envelope dollar maths, not vendor figures.

## The short version for the team

1. **Default implementation model: Opus 5.5 at `high` (or `medium`). Never `max`.** He says this in four separate videos. `max` costs roughly 10× the tokens and is 20× slower in the worst case, for about a 1% benchmark gain: "it is removing its ability to think less." Don't drop Anthropic models to `low` either; they need a minimum reasoning budget.
2. **Review with a different model.** Have Opus write and GPT-6.1 Sol (or Astra or Grok) review. He found the OpenAI models surfaced more valid issues in review, and Sol is far cheaper per task. This is the same idea as our `kimi-delegation` second opinion.
3. **Use cheap models as subagents, not as drivers.** Sonnet 5.5 is good for codebase deep dives and scoping. Haiku 5.5 is the first small Anthropic model he would use, for search, categorizing and parallel PR audits. Cap Haiku's auto-compact window at 100k tokens to stay in its cheaper price tier. Claude Code 2.1.296 now has `autoCompactWindow` in subagent frontmatter for exactly this. GLM 5.3 Flash is very cheap for bulk agentic work.
4. **Say what "done" means in every prompt**, for example "merge once green" or "file a PR and ping me". With that, the model works for hours. Without it, the model stops early.
5. **Don't babysit threads.** Fire off a task, ignore it until it is done or needs you, and spend tokens on verification and on making the merge button safe: previews, nightly builds, CI ratchets. He doesn't spend them watching agents.
6. **Turn off the "improve the model for everyone" data sharing** in Codex (Settings → Data controls) and in Claude. He argues this makes personal-plan terms roughly match team terms.
7. **Be sceptical of auto-memory.** He audited Claude Code's auto-memory on his machine and found it mostly stale and unread. Put rules in architecture, CI checks and `AGENTS.md`/`CLAUDE.md` instead.

## Plans and pricing, as Theo reads them

- **Codex $200 (Pro).** Inference value dropped from about $12k to about $2–2.5k a month once GPT-6.1 Sol arrived and usage accounting changed. It is still good value **if you use Sol only**, not Astra. In early October he said OpenAI had temporarily paused new sign-ups for the $200 tier.
- **Codex $500 with Ultrafast.** Ultrafast spends included usage at 8× the standard rate, so the weekly allowance can be gone in about 2 hours. He called the $500 plan "a mistake". Ultrafast is fun for live UI iteration, but he says almost nobody should use it yet. The official speed docs confirm the 8× rate on subscription limits and 6× on credits.
- **Claude Max $200.** He still recommends it for serious shipping: "I'm an Anthropic fanboy in the end." Fable/Mythos 5.x usage is capped at about half the allowance. Anthropic's "25% higher limits" from Sep 14 is, by his maths, a ~17% cut compared with the temporary boost before it. The plan includes $200 a month of API credit, worth using for Haiku/Sonnet tooling.
- **Cursor loses OpenAI models on Nov 12 2026.** If your team reaches GPT through Cursor, plan direct subscriptions or an open harness before then.
- **"AI isn't getting cheaper" (Oct 12).** The price of a given level of intelligence fell more than 10× in a year. Bills rise because agents now do whole tasks end to end (Jevons paradox). Compare **cost per task**, not price per token.

## Models in one line each

| Model | Theo's verdict | Video |
|---|---|---|
| Opus 5.5 | New daily driver. Cheaper and faster than Opus 5, communicates well, offers zero data retention (Fable doesn't). | [My new favorite model.](https://youtu.be/jgGyX7MPPVg) (Sep 23) |
| Fable 5.1 | Best taste for frontend and marketing UI, very mergeable code, no ZDR, capped on subscriptions. | [My New Favorite Model](https://youtu.be/r_dw-1109Ag) (Sep 3), [So I was using Fable wrong](https://youtu.be/-XWSJM-Ue-o) (Sep 22) |
| GPT-6 Astra | A leap in computer use, 3D and multi-agent "swarms". Less mergeable than Fable and erratic. Expensive on subscriptions. | [It's Here.](https://youtu.be/XFWpf0wLbh0) (Sep 4), [Fable vs Astra](https://youtu.be/P7bxbDSnZRM) (Sep 11) |
| GPT-6.1 Sol | Cheap and strong at audits, root-cause work and review. Weak at long refactors, "garbage UI". About 1/13 of Opus's cost per task in his test. | [OpenAI fights back](https://youtu.be/vu8X3YroB-w) (Sep 29) |
| Sonnet 5.5 | Use as a subagent for scoping. Not token-efficient, weak at design. | [OpenAI should be scared](https://youtu.be/8WbW_n95wc4) (Sep 29) |
| Haiku 5.5 | First usable small Anthropic model. Subagent work only, keep reasoning on, 100k context threshold. | [finally a good small model](https://youtu.be/38_6C0dkKmU) (Oct 9) |
| GLM 5.3 Flash | About Opus-4.8-level agentic work at a tiny fraction of the price. Audited hundreds of PRs for cents. | [Ox Alpha is INSANE](https://youtu.be/Xdxp3lbQKyQ) (Aug 27) |

## Workflow tips worth stealing

- **From Anthropic's Opus 5.5 guidance** ([Getting the most out of Opus 5.5](https://youtu.be/ejjBbaq9RmY)):
  - Give the model negative design constraints ("no cream backgrounds, no pill buttons").
  - Steer a running task with follow-up messages instead of restarting it.
  - After a long run, ask "what are the risks of merging this?"
  - Ask it to list what it couldn't verify.
  - Put a stop rule in `CLAUDE.md`: keep going on anything non-destructive, and stop only for destructive actions or when blocked.
  - Opus 5.5 is reluctant to spawn subagents or workflows on its own, so ask for them explicitly on audits and migrations.
- **From the Fable 5.1 prompting video:**
  - Delete old formatting and anti-verbosity rules from your system prompts.
  - Default to `high` effort.
  - Tell the agent to log pre-existing bugs as follow-ups instead of fixing them in the current PR.
  - Use auto/full approval modes instead of accept-edits.
- **"Once Claude can measure something, it can make it faster"** ([Anthropic Made Claude Even Better](https://youtu.be/FsDUOUV9Vs8)). Anthropic made claude.ai about 3× faster by building deterministic benchmarks first (instruction counts, not wall-clock time), then letting agents hill-climb against them, with CI ratchets that block regressions.
- **Remote Linux boxes for agents** ([You're using AI agents wrong](https://youtu.be/q1D90-uGvBg), [If you have a Claude sub](https://youtu.be/D8PikZ1KhUo)). Run parallel agents on cheap Linux machines reached over Tailscale, not on a Mac. Don't sign in to Claude Code or Codex from datacenter VPS IPs, because he says those get flagged. Give agents a real browser. He landed 50+ PRs a week while travelling this way.
- **Widen prompts instead of deepening them** ([Please stop using stupid models](https://youtu.be/iBrAWpjXNxs)). Ask for "fix the bug, find the files, test it, open a PR" instead of "edit this file". Have the agent record a video of the UI flow as proof. Frontier models are worth the cost because their *floor* is higher, so you step in less.
- **Voice input** ([How I Code Without Typing](https://youtu.be/NvVbCqDgfCs)): Wispr Flow (the captions spell it "Whisper Flow") with a directional mic, so you can dictate in an office.
- **ts-rust** ([$420k rewrite](https://youtu.be/ND_kRgU_WlY)). Opus 5.5 agents ported the TypeScript compiler and LSP to Rust. That would be about $420k at API prices, or a few hundred dollars on subscriptions. Repo: `pingdotgg/ts-rust`. It is interesting as a type checker for `effect`/tRPC-heavy codebases.

## What this means for our setup

- Our `kimi-delegation` and `requesting-code-review` patterns match his strongest advice: a different model reviews. Consider adding GPT-6.1 Sol as a reviewer option next to Kimi.
- Pin subagent models and effort explicitly. Claude Code now takes an `effort` parameter on the Agent tool and `autoCompactWindow` in agent frontmatter. Codex has `[agents] default_subagent_model` / `default_subagent_reasoning_effort`. See the harness comparison, `2026-10-12-codex-vs-claude-code-harness.md`.
- His memory critique is worth checking against our own `~/.claude/projects/*/memory/`. In this repo it is curated, but the same audit applies.
