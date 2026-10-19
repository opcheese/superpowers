# Fix-Check Prompt Template

Use this template once per round, after the author has folded the accepted findings.

**Purpose:** confirm the accepted findings are resolved and the fold broke nothing. It is not a second review.

```
Task tool (general-purpose):
  description: "Fix-check [document] review [N]"
  prompt: |
    You are checking one round of fixes to a document. You are not
    reviewing the document again.

    ## Inputs

    - The findings from review [N], each with the author's verdict
      (accept / reject with reason / backlog with place):
      [findings and verdicts]
    - The fold: `git diff [base]..[head] -- [path]`
    - The document at [head], for the text around the changes.

    ## What to check

    1. **Each accepted finding.** Is it resolved at [head]? Quote the text
       that resolves it. A resolution that removes the symptom but leaves
       the finding's consequence possible is not resolved.
    2. **What the fold changed the meaning of.** Read the diff, and the
       unchanged text that depends on what changed: cross-references,
       counts, steps that use a changed value or name. Report anything the
       fold broke there.
    3. **Rejections and backlog** are the author's rulings. Don't re-argue
       them. If a rejection's stated reason is factually false and you
       can show it, say so once, with the check.

    4. **The same problem elsewhere.** For each accepted finding, look
       for the same pattern in text the fold didn't touch (the same
       unsafe call, the same wrong assumption in another task). Don't
       fix-check it; list it under Outside this check, with its
       consequence, so the human sees it.

    Anything else you notice in text the fold didn't touch goes under
    Outside this check, one line each. It is not a finding of this round.

    ## Report

    **Verdict:** clean / not clean

    **Accepted findings:** for each: RESOLVED or NOT RESOLVED, with the
    quoted text or the check that shows it.

    **Broken by the fold:** each with where, consequence and check.
    "None" if none.

    **Rejection reasons shown false:** "None" if none.

    **Outside this check:** the same problem elsewhere first, each with
    its consequence; then one line each for anything else. These go to
    the human with the round, not into another review.
```

**Commit the fold as part of round N** (`... review N fix-check — ...`), not as a new round.
