# How reviewing-documents keeps receiving-code-review

The author's half of `reviewing-documents` invokes `superpowers:receiving-code-review` by name rather than copying it, so upstream's improvements to that skill reach document reviews too. This file lists each of its rules and where that rule lives here. Check it when either skill changes.

| receiving-code-review rule | Here | How |
|---|---|---|
| The response pattern: read, understand, verify, evaluate, respond, implement | Kept by reference | "Implement one item at a time, test each" becomes: derive each fix and check it before folding, then the fix-check confirms each accepted finding |
| Forbidden responses: no performative agreement, no implementing before verifying | Kept by reference | Unchanged |
| Handling unclear feedback: clarify every unclear item before implementing any | Kept by reference | Unchanged. On `agents` this is a safe default or escalation; on `main` it is asking your human partner, as that branch's copy says |
| From the project owner: trusted, still ask if scope is unclear | Kept by reference | Unchanged |
| From external reviewers: be skeptical, check carefully, push back if wrong, escalate if it conflicts with prior decisions | Kept by reference, and extended | A dispatched document reviewer is an external reviewer. A finding that would change a recorded decision becomes a ruling for the decision's owner (*A change to a recorded decision is a ruling, not a fold*) |
| YAGNI check for "professional" features: grep for usage | Carried over, adapted | A document has no callers to grep. The equivalent check is against the spec: a finding asking for more than the spec asks for is rejected or backlogged |
| Implementation order: clarify first, then blocking, simple, complex; test each; check for regressions | Partly kept, partly not applied | "Clarify first" is kept by reference. The blocking/simple/complex order is not applied: a document has no build that one fold can break for the next. "Check for regressions" becomes the fix-check's *What the fold changed the meaning of* |
| When to push back, and how: technical reasoning, specific questions, involve the owner if architectural | Kept by reference | Pushback is recorded as a reject verdict with its reason, so it stays on record for the next reviewer and counts toward the false-alarm rate |
| Pushback you're uncomfortable voicing: name the tension and escalate | Kept by reference | Unchanged |
| Acknowledging correct feedback: state the fix, no thanks | Kept by reference | The accept line in the commit body is the acknowledgement |
| Gracefully correcting your pushback | Kept by reference | If a fix-check shows a rejection's reason was false, correct the verdict factually and bring it to the human; the fold is their call |
| Common mistakes table | Kept by reference | Unchanged |
| GitHub thread replies | Kept by reference | Applies when the review arrives as inline merge-request comments |

What `reviewing-documents` adds on top: a verdict for every finding (accept, reject, backlog, ruling) and its tally (the false-alarm rate), treating a reviewer's proposed fix as evidence rather than the fix, settling runnable design with a spike, folding only what was asked, recording decision changes as rulings, round-numbered commit subjects, and the one-review, one-fix-check round.
