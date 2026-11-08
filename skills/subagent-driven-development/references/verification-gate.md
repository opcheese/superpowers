# The Automated Verification Gate

Part of superpowers:subagent-driven-development (agents fork). Read this at
every task boundary, before you write the completion line.

**The automated verification gate.** When the review comes back clean — or
every open finding is parked with a ruling at the cap — run the project's own
verification yourself, before any completion line. Do not rely on the
implementer's or the reviewer's report for this. In unattended operation this
gate stands where the interactive flow would have a human sign off.

- Run the project's full test command and its linter/type-check (e.g.
  `pnpm test` and `pnpm lint`, or the project's equivalents). Use the commands
  the plan or the repo's tooling defines, not assumptions.
- **Pass:** proceed to the completion line (SKILL.md, step 5).
- **Fail:** the failing output is a finding. Send it into the fix loop as a
  round like any other (rounds 1-3 resume the implementer), then re-run the
  gate. If the gate is still red when the five-round cap trips, a broken
  build is load-bearing by definition: ledger `BLOCKED`, escalate via
  superpowers:escalation, and do not mark the task complete.

The gate is a controller-run safety net, not a substitute for the implementer
running tests during TDD or the reviewer's read — it catches integration
failures that only surface when the whole task's changes run together.

| Excuse | Reality |
|--------|---------|
| "The reviewer approved, the gate is redundant" | The reviewer read a diff; it did not run the suite. The gate catches integration failures no per-task diff shows. |
| "The implementer said tests pass" | That is the report, not the evidence. Run the suite yourself before the completion line. |
