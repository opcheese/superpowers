---
name: documentation-hygiene
description: Use when creating or editing project Markdown docs that others read over time — specs, research notes, plans, design docs, runbooks, READMEs. Establishes consistent frontmatter (status, last-verified date, owner) so humans and agents can tell which docs to trust, who owns them, and what superseded them.
---

# Documentation Hygiene

## Overview

Docs rot. Over months, a spec gets superseded, a plan gets abandoned, a research
note stops matching reality — but nothing on the page says so, and the next
reader (human or agent) trusts stale content. The fix is a small, **consistent**
frontmatter block on every doc, so trust is machine-checkable and uniform across
the project.

**Core principle:** Every long-lived Markdown doc carries the same disciplined
frontmatter schema. Consistency is the point — an ad-hoc block per doc is nearly
as useless as none, because tooling and readers can't rely on it.

## When to Use

- Creating or substantially editing any doc under `docs/`, `specs/`, or similar
  that will be referenced later by people or agents.
- Reviewing a doc whose trustworthiness is unclear ("is this still current?").

**When NOT to use:** throwaway scratch notes, generated files, or a project that
already defines its own doc-metadata convention — follow that instead (project
instructions always win).

## The Frontmatter Schema

```yaml
---
title: Human-readable title
status: current            # current | draft | superseded | archived
last_verified: 2026-07-15  # ISO date a human last checked this against reality
owner: alice               # who maintains it (the person to ask), not git authorship
area: caching              # one value from the project's closed set (see below)
audience: dev              # dev | ops | pm — list form (dev, pm) for cross-cutting
---
```

**Optional fields (add only when they carry weight):**

| Field | When to use |
|---|---|
| `version` | Evolving docs (`0.1`, `2.1`). **Required whenever the doc carries a `## Changelog` table.** Bump on meaningful rewrites. |
| `depends_on` | Paths of docs this one relies on for definitions/IDs. |
| `superseded_by` | Path of the doc that replaces this one (set together with `status: superseded`). |

## Disciplines

- **`last_verified` means a human checked it against reality**, not when it was
  generated. Update it on substantive edits. A stale `last_verified` is a signal,
  not a formality.
- **An empty required field is a visible "needs attention" marker, not a finished
  field.** When you can't fill `owner` yet, leave `owner:` empty rather than
  guessing or omitting the key — the blank is a to-do the next editor can see.
- **`area` is a closed set the project defines.** Pick the single best fit. Adding
  a new value is a deliberate change to that list, not an ad-hoc choice inside a
  doc. If the project has no list yet, propose a short one; don't invent a fresh
  value per doc.
- **Changelog lives at the top of the doc**, as a `## Changelog` table (date +
  what changed), newest first — not a growing history section at the bottom.
  Whenever a `## Changelog` exists, `version` is required and moves with it.
- **`status` transitions are meaningful:** `draft` → `current` → `superseded`
  (set `superseded_by`) or `archived`. Never leave a replaced doc marked
  `current`.

## Quick Reference

| Situation | Do this |
|---|---|
| New doc | Add all required fields; `status: draft` until reviewed |
| Substantive edit | Bump `last_verified`; add a Changelog row + `version` if versioned |
| Can't fill a field yet | Leave it empty (`owner:`) as a visible marker |
| Doc replaced | `status: superseded` + `superseded_by:` on the old one |
| New `area` needed | Add it to the project's closed set first, then use it |

## Common Mistakes

- **Ad-hoc field names** (`last-reviewed`, `authored-by`, `created`) — use the
  schema's exact keys so tooling and other docs stay consistent.
- **`author` = git authorship** — it means the current *owner/maintainer*, the
  person to ask, not who first wrote it.
- **Omitting a required field** instead of leaving it empty — an absent key hides
  the gap; an empty key surfaces it.
- **Inventing `area` values per doc** — defeats the closed set's purpose.
- **Bottom-of-doc history tables** — put the changelog at the top so the latest
  state is the first thing a reader sees.
