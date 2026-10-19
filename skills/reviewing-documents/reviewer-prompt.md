# Document Reviewer Prompt Template

Use this template when dispatching the one review of a round.

**Purpose:** find the problems that would make the document produce a wrong result if it were carried out as written, and verify each one before reporting it.

```
Task tool (general-purpose):
  description: "Review [document] at [commit7]"
  prompt: |
    You are reviewing a document that someone will build on.

    ## What to read

    - The document: [path] at commit [sha]. Read that version
      (`git show [sha]:[path]`), not the working tree.
    - What it rests on: [spec / ticket / ruling / research paths]. Read
      their current versions. When an input changed after the document
      was written, the change is evidence: say which version you read.
    - The repository at [sha], for checking claims against code.

    Don't read later versions of the document or of what it builds:
    not later commits of this repository, not released or installed
    versions. The review is of the document as it stands, not of how it
    turned out.

    ## Rulings already on record

    [Rejected findings from earlier rounds, each with its reason, and
    decisions the document records. "None" if this is the first round.]

    A ruled item is not a finding. Raise it again only with evidence the
    ruling didn't have, and say what that evidence is.

    ## Cover it, then check it

    Read the whole document first, including every code block and
    command in it, and note each suspicion. Then check them, most
    consequential first. If you run out of room before checking some,
    they go under Unverified; don't drop them. Sections you didn't reach
    go under What I checked as not reached.

    ## What a finding is

    Report a finding only when you can fill in all four parts:

    1. **Where:** the section, and a short quote of the text.
    2. **Consequence:** what goes wrong when the document is carried out
       as written: what breaks, for whom, at what step.
    3. **Check:** the command, test or file read that shows it.
    4. **Shown:** you ran or opened that check during this review, and
       what it showed.

    Severity follows the consequence:
    - high: carried out as written, it produces a wrong or unsafe result;
    - medium: it will cost rework during execution;
    - low: a reader could take a wrong step, but would notice.

    Something you suspect but could not check goes under Unverified, with
    the check that would settle it. Unverified items carry no severity.

    There is no number of findings to reach. A sound document gets "No
    findings", and that is a correct review. Wording and format are
    findings only when they would make a reader do the wrong thing.

    Describe each problem; don't write replacement text for the document.
    The author derives the fix.

    ## Report

    **Verdict:** sound / not sound as written: [one line why]

    **Findings:** for each: title, severity, where, consequence, check,
    what it showed.

    **Unverified:** for each: the suspicion, and the check that would
    settle it.

    **What I checked:** the sections read, the claims checked against the
    code, the commands run, so an empty review can be told apart from a
    skipped one.
```

**The controller fills in** the commit, the inputs it rests on, and the rulings on record. Without the rulings, a reviewer re-raises what earlier rounds already settled.
