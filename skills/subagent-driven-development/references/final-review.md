# Final Review and Finish

Part of superpowers:subagent-driven-development. Every task in the plan is
complete.

The final whole-branch review gets a package too: run
`bash scripts/review-package PLAN_FILE MERGE_BASE HEAD` by its absolute path,
from the working tree you're executing in (MERGE_BASE = the commit the
branch started from, e.g. `git merge-base main HEAD`) and include the
printed path in the final review dispatch, so the final reviewer reads
one file instead of re-deriving the branch diff with git commands. Dispatch
on the most capable available model (see SKILL.md's Model Selection), using
superpowers:requesting-code-review's
[code-reviewer.md](../../requesting-code-review/code-reviewer.md). Point it at
the ledger's deferred-minor and parked lines so it can triage which must be
fixed before merge.

If the final whole-branch review returns findings, dispatch ONE fix subagent
with the complete findings list — not one fixer per finding.
Per-finding fixers each rebuild context and re-run suites; a real
session's final-review fix wave cost more than all its tasks combined.
Then run exactly one scoped re-review of the fix wave
(`bash scripts/review-package PLAN_FILE FIX_BASE HEAD` by its absolute path,
from the working tree you're executing in, over the fix range,
[re-review-prompt.md](../re-review-prompt.md)).
Adjudicate any residual findings as in the fix loop's breaker
(references/fix-loop.md): park with rulings, or rule on the load-bearing
ones and ledger what you decided. Only the two escalating classes (top of
SKILL.md) stop you here. There is no second fix wave — residual rulings and
any escalation go into the PR description that finishing-a-development-branch
opens, so the reviewer of that PR inherits them explicitly rather than by
surprise.

### Finish

Before you delete anything, collect every ledger line containing `Ruling:` —
preflight rulings, parked findings, breaker adjudications, all of them — plus
every escalation you logged, in the order you made them, each with what it
costs if wrong. The list is exhaustive: if the ledger holds a ruling, the
list holds it.

Unattended, that list does not go in a final message — no one is here to
read one, and the workspace is about to be deleted. It goes into the
**Open items** section of the PR description that
finishing-a-development-branch opens, and it is the only place the decisions
you took on your human partner's behalf survive to reach them: the PR
reviewer reads them and reworks whatever you got wrong. A ruling that dies
with the workspace was a decision made in secret.

When the final whole-branch review is clean and its fixes are merged,
delete this plan's workspace (`rm -rf <workspace>`) — the git history is
the record now. Sibling directories belong to other plans; leave them
alone.

Use superpowers:finishing-a-development-branch.
