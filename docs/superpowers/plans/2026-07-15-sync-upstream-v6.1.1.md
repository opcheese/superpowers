# Sync Fork to Upstream Superpowers v6.1.1 Implementation Plan

> **For agentic workers:** This is a git-integration plan, not a code-feature plan. It does not use TDD steps; each task is a discrete, independently-verifiable integration action. Execute task-by-task, verify after each, commit after each. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Bring our `agents` fork from base v5.1.0 up to upstream `v6.1.1`, adopting upstream's v6.x work (SDD rewrite, writing-plans structure, `.git/` artifact fix, vendor-neutral vocabulary) while preserving our unattended-operation customizations.

**Architecture:** Rather than merging 187 upstream commits *backward* into our fork — which would generate conflicts on every skill upstream rewrote — we **re-apply our fork's features onto a fresh copy of `upstream/main`**. Our fork's value is a small, well-defined set of changes; upstream rewrote the base underneath them. Re-applying our *intent* on the new base is cleaner and matches our "layer ours on upstream's SDD" decision. Work happens in an isolated worktree; nothing touches `agents` until the integration branch is verified.

**Tech Stack:** git (worktree, merge/cherry-pick), bash, markdown skills.

---

## Decisions (locked with human partner)

- **Target:** `upstream/main` at `v6.1.1` — NOT `dev` (dev's skill-compression wave is unreleased and churning).
- **Brainstorming visual companion:** **keep deleted.** Our agents are headless. Do not restore `scripts/` or `visual-companion.md`.
- **SDD flow:** **layer ours on upstream's.** Take upstream's one-reviewer rewrite + `task-brief`/`review-package`/`sdd-workspace` scripts as the base, then re-apply our automated verification-gate + escalation so unattended runs don't block on a human.
- **Package manager:** keep our global `npm → pnpm` substitution.

## Global Constraints (bind every task)

- All work in a dedicated worktree off `upstream/main`; `agents` stays untouched until final integration.
- Commit after every task with a conventional-commit message; never bundle unrelated changes.
- Preserve, verbatim, our four custom skills (`escalation`, `end-of-day-report`, `end-of-week-report`, `topic-research`) — they do not exist upstream, so they carry over clean.
- Do NOT open any PR to `obra/superpowers`. This is a private fork sync; `origin` is `opcheese/superpowers`.
- After each skill re-application, the skill must still read coherently — no dangling references to deleted files (e.g. visual companion) or to the old two-reviewer SDD flow.

## Our fork's changes to preserve (the re-apply target set)

| Change | Source commit | Nature | Merge handling |
|---|---|---|---|
| 4 custom skills | `e69d824`, `e63dc26`/`f9140b3`, `a1a36bb` | Additive, no upstream equivalent | Copy directories verbatim |
| `npm → pnpm` in skills | `c708819` | Text substitution across 7 skill files | Re-run substitution on new base |
| Unattended-run reshaping | `d1534f4` | Automated gate + escalation; companion deletion; trimmed interactive menus | Re-apply intent onto rewritten skills, task by task |
| README fork notes | `d1534f4`, `c708819` | Fork-specific README additions | Re-apply relevant hunks |
| `.gitignore`, plan doc | various | Housekeeping | Re-apply |

## File Structure (what the integration branch produces)

- `skills/escalation/`, `skills/end-of-day-report/`, `skills/end-of-week-report/`, `skills/topic-research/` — carried over unchanged.
- `skills/subagent-driven-development/` — upstream's new files (`task-reviewer-prompt.md`, `implementer-prompt.md`, `scripts/{task-brief,review-package,sdd-workspace}`) PLUS our automated-gate + escalation additions folded into `SKILL.md`.
- `skills/brainstorming/` — upstream's `SKILL.md` MINUS the visual-companion wiring; `scripts/` and `visual-companion.md` removed.
- `skills/{test-driven-development,executing-plans,finishing-a-development-branch,using-git-worktrees,dispatching-parallel-agents,receiving-code-review,systematic-debugging,writing-plans,verification-before-completion}/SKILL.md` — upstream base + our pnpm/unattended re-applications.
- `.claude-plugin/plugin.json` — version reflects upstream `6.1.1` (fork may append a suffix, e.g. `6.1.1-agents`, decided in Task 8).

---

## Task 1: Create the integration worktree off upstream/main

**Files:** none (git state only)

- [ ] **Step 1: Confirm clean tree and fetch upstream**

```bash
cd /home/newub/w/superpowers
git status --short          # expect empty
git fetch upstream --tags
git log --oneline -1 upstream/main   # expect d884ae0 Release v6.1.1
```

- [ ] **Step 2: Create worktree on a new integration branch based on upstream/main**

```bash
git worktree add -b sync/upstream-v6.1.1 .worktrees/sync-v6.1.1 upstream/main
cd .worktrees/sync-v6.1.1
git log --oneline -1        # expect d884ae0
```

Expected: worktree at `.worktrees/sync-v6.1.1`, HEAD = upstream v6.1.1, custom skills NOT yet present.

- [ ] **Step 3: Record the baseline for later diffing**

```bash
git ls-tree -r --name-only HEAD | grep '^skills/' | cut -d/ -f2 | sort -u > /tmp/upstream-skills.txt
```

## Task 2: Carry over the four custom skills (clean adds)

**Files:**
- Create: `skills/escalation/**`, `skills/end-of-day-report/**`, `skills/end-of-week-report/**`, `skills/topic-research/**`

- [ ] **Step 1: Copy each custom skill from the `agents` branch**

```bash
for s in escalation end-of-day-report end-of-week-report topic-research; do
  git checkout agents -- "skills/$s"
done
git status --short          # expect only new files under those 4 dirs
```

- [ ] **Step 2: Verify escalation JSON sidecar came across**

```bash
ls skills/escalation/            # expect SKILL.md + JSON sidecar (from commit a1a36bb)
```

Expected: four skill directories staged as additions, nothing else touched.

- [ ] **Step 3: Commit**

```bash
git add skills/escalation skills/end-of-day-report skills/end-of-week-report skills/topic-research
git commit -m "feat: carry over fork custom skills (escalation, reports, topic-research)"
```

## Task 3: Keep the brainstorming visual companion deleted

**Files:**
- Delete: `skills/brainstorming/scripts/{frame-template.html,helper.js,server.cjs,start-server.sh,stop-server.sh}`, `skills/brainstorming/visual-companion.md`
- Modify: `skills/brainstorming/SKILL.md` (remove companion wiring)

- [ ] **Step 1: Inspect how upstream's SKILL.md references the companion**

```bash
grep -n -i 'companion\|visual\|start-server\|browser' skills/brainstorming/SKILL.md
```

Read the surrounding sections so the removal leaves coherent prose (do NOT blindly delete lines).

- [ ] **Step 2: Remove the companion assets**

```bash
git rm skills/brainstorming/scripts/frame-template.html \
       skills/brainstorming/scripts/helper.js \
       skills/brainstorming/scripts/server.cjs \
       skills/brainstorming/scripts/start-server.sh \
       skills/brainstorming/scripts/stop-server.sh \
       skills/brainstorming/visual-companion.md
```

- [ ] **Step 3: Edit `skills/brainstorming/SKILL.md`** to drop every companion reference (the "offer the visual companion" step, the browser-open instructions, any `references:` pointer to `visual-companion.md`). Use our `agents`-branch version of `SKILL.md` as the reference for what a headless brainstorming skill should read like:

```bash
git show agents:skills/brainstorming/SKILL.md > /tmp/our-brainstorming.md
# Diff to see how we trimmed it, then apply equivalent trims to upstream's version.
diff /tmp/our-brainstorming.md skills/brainstorming/SKILL.md | head -60
```

Fold our headless trims into upstream's newer copy — keep upstream's non-companion improvements, drop companion wiring.

- [ ] **Step 4: Verify no dangling references remain**

```bash
grep -rn -i 'visual-companion\|start-server\|frame-template' skills/brainstorming/ || echo "clean"
```

Expected: `clean`.

- [ ] **Step 5: Commit**

```bash
git add skills/brainstorming
git commit -m "refactor(brainstorming): keep fork headless — drop visual companion"
```

## Task 4: Re-apply the pnpm substitution on the new base

**Files:**
- Modify: the skill files that reference `npm test` / `npm run` (upstream set may differ from our original 7)

- [ ] **Step 1: Find every `npm` reference upstream still ships**

```bash
grep -rln '\bnpm\b' skills/ | sort
```

- [ ] **Step 2: Review each hit** — only substitute genuine test/run invocations that are project-command examples, NOT prose about the npm registry or package names. For each real command, replace `npm` → `pnpm`.

- [ ] **Step 3: Verify**

```bash
grep -rn '\bnpm test\b\|\bnpm run\b' skills/ || echo "no bare npm commands left"
```

- [ ] **Step 4: Commit**

```bash
git add skills
git commit -m "docs(skills): prefer pnpm over npm in command examples"
```

## Task 5: Layer our automated-gate + escalation onto upstream's SDD

**Files:**
- Modify: `skills/subagent-driven-development/SKILL.md`
- Reference (do NOT overwrite): upstream's `task-reviewer-prompt.md`, `implementer-prompt.md`, `scripts/{task-brief,review-package,sdd-workspace}`

- [ ] **Step 1: Extract our SDD additions from the fork**

```bash
git show d1534f4 -- skills/subagent-driven-development/SKILL.md > /tmp/sdd-our-changes.diff
git show agents:skills/subagent-driven-development/SKILL.md > /tmp/sdd-ours-full.md
```

Our additions: (a) an **automated verification gate** node in the flow (tests pass + linter clean) after the quality-review approves, and (b) **escalation** (via the `escalation` skill) instead of asking a human when the gate fails after retry, and the two guardrails "Skip automated verification gate" / "Move to next task while verification has not passed."

- [ ] **Step 2: Read upstream's rewritten SKILL.md end-to-end** to locate the correct insertion points in the NEW one-reviewer-per-task flow (the old flow had two reviewers; our gate previously sat after "Code quality reviewer approves"). The equivalent point now is after the single `task-reviewer` returns its quality verdict.

- [ ] **Step 3: Fold in the gate + escalation** at the correct point in upstream's flow diagram and prose. Keep upstream's controller/adjudication additions (pre-flight plan review, model-naming requirement, file handoffs) intact — our gate runs *in addition*, as the terminal check before marking a task complete, and routes failures to `escalation` rather than a human question so unattended runs proceed.

- [ ] **Step 4: Sanity-check coherence**

```bash
grep -n -i 'escalat\|verification gate\|two-stage\|spec reviewer\|code-quality-reviewer' skills/subagent-driven-development/SKILL.md
```

Expected: references to the `escalation` skill and the automated gate present; NO references to the removed two-reviewer prompt files (`spec-reviewer-prompt.md`, `code-quality-reviewer-prompt.md`).

- [ ] **Step 5: Commit**

```bash
git add skills/subagent-driven-development/SKILL.md
git commit -m "feat(sdd): layer automated verification gate + escalation on upstream review flow"
```

## Task 6: Re-apply unattended tweaks to the remaining reshaped skills

**Files:**
- Modify (each independently): `skills/executing-plans/SKILL.md`, `skills/finishing-a-development-branch/SKILL.md`, `skills/using-git-worktrees/SKILL.md`, `skills/dispatching-parallel-agents/SKILL.md`, `skills/receiving-code-review/SKILL.md`, `skills/systematic-debugging/SKILL.md`, `skills/writing-plans/SKILL.md`, `skills/verification-before-completion/SKILL.md`

- [ ] **Step 1: For each file, diff our fork's version against the new upstream base**

```bash
for f in executing-plans finishing-a-development-branch using-git-worktrees \
         dispatching-parallel-agents receiving-code-review systematic-debugging \
         writing-plans verification-before-completion; do
  echo "=== $f ==="
  git diff agents:skills/$f/SKILL.md HEAD:skills/$f/SKILL.md | head -40
done
```

- [ ] **Step 2: For each, decide per-hunk:** keep upstream's version (it's a strict improvement, e.g. writing-plans Global Constraints / Interfaces blocks, forge-neutral finishing), or re-apply our unattended tweak (automated gate, escalation, no interactive menu, no human prompt). Default: **prefer upstream unless the hunk removes human-in-the-loop behavior our unattended agents rely on.** `finishing-a-development-branch` and `executing-plans` are the most likely to need our escalation/automation re-applied; `writing-plans` and `verification-before-completion` likely take upstream wholesale.

- [ ] **Step 3: After editing each, grep for dangling references** (removed files, old flow names, interactive prompts an unattended agent can't answer).

- [ ] **Step 4: Commit per skill** (or in small coherent groups), e.g.

```bash
git add skills/finishing-a-development-branch/SKILL.md
git commit -m "refactor(finishing): re-apply unattended completion flow on upstream base"
```

## Task 7: Re-apply README and housekeeping deltas

**Files:**
- Modify: `README.md`, `.gitignore`
- Create/keep: `docs/plans/2026-03-16-agents-branch-design.md` (fork design doc)

- [ ] **Step 1: Diff README** and carry over only the fork-specific sections (agents-branch usage, pnpm note, human-verification requirement) onto upstream's newer README:

```bash
git diff agents:README.md HEAD:README.md | head -120
```

- [ ] **Step 2: Re-apply our `.gitignore` additions and copy the design doc**

```bash
git checkout agents -- docs/plans/2026-03-16-agents-branch-design.md
git diff agents:.gitignore HEAD:.gitignore
```

- [ ] **Step 3: Commit**

```bash
git add README.md .gitignore docs/plans/2026-03-16-agents-branch-design.md
git commit -m "docs: re-apply fork README notes and agents-branch design doc"
```

## Task 8: Reconcile version and metadata

**Files:**
- Modify: `.claude-plugin/plugin.json` (and any Codex/other manifest that carries a version)

- [ ] **Step 1: Confirm upstream version present**

```bash
grep -r '"version"' .claude-plugin/ .codex-plugin/ 2>/dev/null
```

- [ ] **Step 2: Decide the fork version string** with the human partner: either track upstream exactly (`6.1.1`) or mark the fork (`6.1.1-agents`). Apply consistently across all manifests.

- [ ] **Step 3: Commit**

```bash
git add -A
git commit -m "chore: set fork version to <chosen> (tracks upstream v6.1.1)"
```

## Task 9: Verification pass

**Files:** none (verification only)

- [ ] **Step 1: Confirm every custom skill is intact**

```bash
for s in escalation end-of-day-report end-of-week-report topic-research; do
  test -f skills/$s/SKILL.md && echo "OK $s" || echo "MISSING $s"
done
```

- [ ] **Step 2: Confirm companion stays gone**

```bash
test -f skills/brainstorming/visual-companion.md && echo "REGRESSION: companion returned" || echo "OK companion absent"
```

- [ ] **Step 3: Confirm SDD carries our gate + upstream's new files**

```bash
grep -q -i 'escalat' skills/subagent-driven-development/SKILL.md && echo "OK escalation present"
ls skills/subagent-driven-development/scripts/   # expect task-brief, review-package, sdd-workspace
grep -rq 'spec-reviewer-prompt\|code-quality-reviewer-prompt' skills/ && echo "STALE two-reviewer refs" || echo "OK no stale reviewer refs"
```

- [ ] **Step 4: No dangling references anywhere**

```bash
grep -rn -i 'visual-companion\|start-server' skills/ || echo "OK clean"
grep -rn '\bnpm test\b' skills/ || echo "OK pnpm"
```

- [ ] **Step 5: Run any upstream test suites that don't require network** (e.g. the SDD workspace test) to confirm upstream code still passes on the integration branch:

```bash
bash tests/claude-code/test-sdd-workspace.sh 2>&1 | tail -20 || echo "review failures"
```

- [ ] **Step 6: Full skill-file lint** — every `SKILL.md` has valid frontmatter (`name`, `description`):

```bash
for f in skills/*/SKILL.md; do head -5 "$f" | grep -q '^name:' || echo "BAD frontmatter: $f"; done; echo "lint done"
```

## Task 10: Human review and integration into `agents`

**Files:** none until approved

- [ ] **Step 1: Produce the full diff for the human partner**

```bash
git log --oneline upstream/main..sync/upstream-v6.1.1
git diff agents...sync/upstream-v6.1.1 --stat
```

- [ ] **Step 2: Human partner reviews the complete diff** (per fork policy — no unreviewed integration). Address feedback with follow-up commits on the integration branch.

- [ ] **Step 3: Integrate into `agents`** — chosen with the human partner. Options:
  - Merge `sync/upstream-v6.1.1` into `agents` (preserves both histories), or
  - Fast-forward `agents` to the integration branch if we're comfortable it supersedes the old fork history.

```bash
cd /home/newub/w/superpowers
git checkout agents
git merge --no-ff sync/upstream-v6.1.1 -m "feat: sync fork to upstream superpowers v6.1.1"
```

- [ ] **Step 4: Push and clean up the worktree**

```bash
git push origin agents
git worktree remove .worktrees/sync-v6.1.1
git branch -d sync/upstream-v6.1.1   # after merge
```

---

## Self-Review Notes

- **Spec coverage:** every locked decision has a task — target (Task 1), custom skills (Task 2), companion deleted (Task 3), pnpm (Task 4), SDD layering (Task 5), other reshaped skills (Task 6), README/housekeeping (Task 7), version (Task 8).
- **Risk hotspots:** Task 5 (SDD layering) and Task 6 (finishing/executing) carry the most judgment — they must preserve unattended operation while adopting upstream's structure. These deserve the closest human review in Task 10.
- **Not doing:** not merging `dev`; not touching Gemini (unsettled upstream); not opening any upstream PR; not restoring the visual companion.
