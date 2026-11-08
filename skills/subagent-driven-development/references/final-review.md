# Final Review and Finish

Part of superpowers:subagent-driven-development. Every task in the plan is
complete.

The final whole-branch review gets a package too: run
`bash scripts/review-package PLAN_FILE MERGE_BASE HEAD` by its absolute path,
from the working tree you're executing in (MERGE_BASE = the commit the
branch started from, e.g. `git merge-base main HEAD`) and include the
printed path in the final review dispatch, so the final reviewer reads
one file instead of re-deriving the branch diff with git commands. Before
dispatching it, ask your human partner to run the native reviewer over the
whole branch — `/code-review high <merge-base>..HEAD`, or `/code-review
ultra` if the branch is large and the run warrants the cost. This is the
per-task checkpoint's whole-branch counterpart and the point where the native
reviewer is most valuable: it sees cross-task interactions no per-task review
covered. Route anything it finds into the final fix wave. Then dispatch our
reviewer on the most capable available model (see SKILL.md's Model Selection), using
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
ones and ledger what you decided. Only the four things that stop a running
plan (top of SKILL.md) stop you here. There is no second fix wave —
residual load-bearing findings surface to your human partner when
finishing-a-development-branch presents the options.

### Finish

Before you delete anything, collect every ledger line containing `Ruling:` —
preflight rulings, parked findings, breaker adjudications, all of them — into
your final message under "Rulings I made", in the order you made them, each
with what it costs if wrong. The list is exhaustive: if the ledger holds a
ruling, the list holds it. That list is the only place the decisions you
took on your human partner's behalf reach them — they read it and rework
whatever you got wrong. A ruling that dies with the workspace was a decision
made in secret.

Then present the completed work to your human partner and wait for their
explicit verification. Present what was built, test results, deferred minors,
parked findings, and the rulings you made. If they request changes, address
them and re-verify. This end-of-run sign-off is required; the per-task reviews
and native-review checkpoints do not replace it. Keep the workspace until
they have verified: changes they ask for still need the ledger.

When they have verified and the final review's fixes are merged, delete this
plan's workspace (`rm -rf <workspace>`) — the git history is the record now.
Sibling directories belong to other plans; leave them alone. Then use
superpowers:finishing-a-development-branch.

| Excuse | Reality |
|--------|---------|
| "The final review was clean, so the work is verified" | Reviewers read diffs. The end-of-run sign-off is your human partner's, and it is not optional. |
