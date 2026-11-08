# Full Send

Part of superpowers:finishing-a-development-branch. Your human partner
explicitly granted it in this session: "full send", "merge it when the gates
pass", or words that plainly mean you merge without asking again. "Get it
done" or "finish it" is not a grant. If no grant was given, you are not here:
present the menu.

Full send means you own the work from its current state to a merged pull
request, and your human partner may not be watching. The grant covers this
branch and this session only.

## What the grant permits

- Pushing the feature branch, and `git push --force-with-lease` to it after a
  rebase. Only the feature branch: never the base branch or anyone else's.
- Merging the PR through the forge (`gh pr merge`, `glab mr merge`) once the
  merge gate below passes, using the repo's usual merge method.

It never permits bypassing branch protection, an admin merge, dismissing
someone else's review, or merging locally and pushing the base branch.

## The flow

1. **Finish and prove it.** Use superpowers:verification-before-completion.
   The evidence is what you observed in this session: the reproduction failing,
   then passing; what CI runs, run locally; for UI, the app run and looked at.
2. **File the PR** (Option 2's steps), with the review recorded as
   superpowers:requesting-code-review says.
3. **Watch it until it settles.** After every push, wait for CI to finish and
   read every new bot and human comment. Fix real findings, each with a test.
   Answer the rest in writing on the PR. Rebase when the branch is behind.
4. **Merge** when every item of the merge gate holds.
5. **Report** to your human partner: the PR link, what merged, what you fixed
   or declined on the way, and the review's verdict.

## The merge gate

All of these, checked right before you merge:

- [ ] The proof covers every change in the final diff, including fixes made
      after review.
- [ ] CI is green on the PR's current head, not an earlier one.
- [ ] Every bot finding is fixed, or declined with a reason written on the PR.
- [ ] No human reviewer is requesting changes. A "request changes" stays
      until that person approves or withdraws it. Fixing what they asked
      for doesn't clear it: request their re-review and wait.
- [ ] The branch is up to date with the base and the forge says it is
      mergeable.
- [ ] The PR's merge danger is a two-way door: reverting the merge undoes it.

## One-way doors

Full send merges two-way doors only. If merging triggers something a revert
can't undo — a migration that drops columns or deletes data, a removed or
renamed public API, a message, payment or release that goes out on merge or
deploy — don't merge, even with every other gate green and good reasons to
believe it is safe. Leave the PR ready, with the door named in its Merge
danger section, and ask (see below). If your human partner is away, report
and wait. The grant was for merging, not for consequences nobody can take
back.

## When you are blocked

An item that won't clear (a reviewer who hasn't re-reviewed, a red check you
can't fix, missing access) stops the merge, never the rest of the work.
Finish everything else, then ask through your harness's question tool
(AskUserQuestion in Claude Code), not plain text: give the PR link, the
blocker, and the option you recommend. A plain-text question at the end of a
turn reads like a finished thread. Without access or an answer, take the
safe path: leave the PR open and report.

| Excuse | Reality |
|--------|---------|
| "They said merge when green, and CI is green" | Green CI is one item out of five. The grant is for merging when the gates pass. |
| "I fixed what the reviewer asked, so their block is resolved" | Only the reviewer clears their own "request changes". Ask them to re-review. |
| "Merging locally is the same thing" | It goes around the PR, its checks and its protection. Merge through the forge. |
| "It's irreversible, but I checked and it's safe" | Safe is your judgment; irreversible is a fact. A one-way door waits for your human partner. |
| "They said finish it, which means merge" | Only an explicit grant starts full send. Otherwise, present the menu. |
