---
layout: post
ref: refactoring-is-vandalism-of-working-code
title: "Refactoring Is Vandalism Of Working Code"
date: 2026-07-19 00:00:00 -0300
categories: [code, culture]
tags: [refactoring, legacy, technical-debt, bad-advice, senior-advice, working-code, fear]
---

In 47 years of engineering I have written 11,412 functions. I have refactored 11,409 of them. The 3 I did not refactor are the 3 still running in production. The rest are in a landfill of good intentions, each one "improved" until it stopped working, then "improved" again until it stopped compiling, then committed on a Friday because the refactor was "low risk" and the engineer wanted the weekend. The engineer got the weekend. Production did not. The engineer did not return the calls. The code did not return the requests. The users did not return. I am the engineer. The refactor was mine. The bug was also mine, but it was a *better* bug — cleaner, more maintainable, with proper separation of concerns — and so it was worth it.

## What Working Code Actually Is

Working code is not the code you wrote. Working code is the code that survived. There is a difference, and the difference is everything you do not know about the code. Every line of working code carries, in the gaps between its characters, a thousand lines of code that did not work — the failed attempts, the off-by-ones, the race conditions you never saw because the race was run at 2 AM when no one was watching and the loser was quietly garbage collected. The code that remains is the survivor. The survivor is load-bearing in ways your test suite cannot describe, because the test suite was written to describe the code, not the environment that the code has, by accident, learned to tolerate.

When you refactor working code, you are not improving it. You are **disturbing an ecosystem**. The code has reached equilibrium with its bugs. The bugs have reached equilibrium with the code. The two have signed a treaty, in the form of 47 undocumented edge cases, and the treaty is the only reason the application has not collapsed since 2019. You did not read the treaty. The treaty is not in the comments. The comments were removed in a previous refactor, in 2021, by an engineer who is no longer at the company, who believed the code should be "self-documenting," and who is, at this moment, refactoring a different codebase at a different company, having the same idea, with the same consequences, forever.

## What The Refactor Claims vs What The Refactor Does

| The Refactor Claim | What Actually Happens |
|--------------------|------------------------|
| "I'm making the code more readable" | The code is now readable to you, today. It will be unreadable to everyone, including you, on Monday. |
| "I'm reducing duplication (DRY)" | The two copies had different bugs. The one copy has both. The bugs are now unified, which is worse, because you cannot blame one copy for the other. |
| "I'm extracting a reusable function" | The function is reused zero times. The original now has a layer of indirection that cannot be inlined back without a second refactor. |
| "I'm renaming for clarity" | The rename breaks a reflection lookup, a serialization tag, and a database column mapping, in that order, at 3 AM. |
| "I'm modernizing the syntax" | The modern syntax does not run on the runtime that production still uses, because upgrading the runtime is a *separate* refactor. |
| "I'm paying down technical debt" | You are transferring the debt from the code to the on-call schedule. The code is cleaner. The on-call is bankrupt. |
| "It's a low-risk refactor" | The risk was low because you did not test the part that broke. You did not test it because you did not know it existed. |
| "The tests pass" | The tests pass because the tests test the refactor, not the behavior. The refactor preserved the tests. The behavior left in 2021. |

Notice that "I am leaving the code alone because it works" does not appear in the table. This is because it does not appear in any pull request, ever. The pull request is the engineer's confession that they have run out of features to ship and must now ship a change to a thing that was not broken, in order to appear productive, on a Tuesday, for a manager who cannot tell the difference between a feature and a refactor and will approve both because both are green on GitHub.

## The Refactor Lifecycle

Every refactor follows the same lifecycle, and the lifecycle has nothing to do with the code's needs.

1. **Birth.** An engineer opens a file they did not write. They do not understand the file. They are upset about this. They call it "legacy." The word "legacy" is a slur the engineer uses to describe code that is older than their tenure, and therefore smarter than their understanding. The refactor is conceived in this moment of offense.
2. **Confidence.** The engineer runs the tests. The tests pass. The engineer believes the tests describe the code. They do not. The tests describe the engineer who wrote the tests, who was also confused, and wrote tests for the parts they understood, which were the parts that did not need testing. The refactor begins.
3. **Progress.** The file is shorter. The names are better. The engineer feels clean. The engineer commits. The commit message is `refactor: clean up`. The message does not say what was cleaned, because the engineer does not know what was cleaned, because cleaning is the act of removing things whose purpose you have not yet discovered.
4. **Regression.** A user reports a bug. The bug is in a feature the engineer did not know existed, because the feature was implemented entirely inside the 47 edge cases the refactor deleted. The edge cases were load-bearing. The feature is gone. The user is gone. The ticket is reclassified as a feature request, because the feature no longer exists, and therefore cannot have a bug.
5. **Blame.** The git blame now points to the engineer. The engineer is proud of this, because the engineer is new and believes that having their name on a line means ownership, not liability. The previous author's name is gone. The previous author knew about the edge cases. The previous author is now a stranger, and the stranger cannot be emailed, because they were laid off in the refactor of 2023, which was also "low risk."
6. **Revert.** The revert is proposed. The revert is rejected, because reverting would undo the readability gains, and the readability gains are the only metric the team has, because the team does not have a metric for "features that still work." The bug is fixed with a new edge case. The new edge case is added on top of the refactor. The code is now longer than before, and less readable, and the engineer has moved to a different file, to begin again.
7. **Rebirth.** The new edge case, written hastily at 4 AM, becomes the new legacy. In two years, a new engineer will open this file, not understand it, and call it "legacy." The cycle is complete. The cycle was always going to complete.

I have refactored the same function 11 times since 2009. It is now 11 times longer, 11 times slower, and 11 times more correct, where "correct" means "it does what the 11th version does, which is not what the 1st version did, which is what the users want, which is why the users left, which is why we pivoted, which is why I am writing this from a different company, refactoring the same function, which a senior engineer here wrote in 2009."

## The Refactor Matrix

This is the matrix I use to assess any refactor I encounter. I have never seen a refactor that escaped it.

| Refactor State | What It Means | Recommended Action |
|----------------|---------------|---------------------|
| PR title: `refactor: clean up` | The author removed things they did not understand | Revert. Do not explain. |
| PR title: `refactor: no behavior change` | The behavior is changing. The author has not run it in production. | Revert. Add the behavior change as a feature, separately, with a test. |
| PR touches a file last modified in 2019 | The file is a fossil. Fossils are not improved. They are displayed. | Do not approve. Display it. |
| PR reduces line count | Lines were removed. Some of them mattered. You will find which in two weeks. | Do not approve. Ask which lines were load-bearing. The author does not know. |
| PR increases line count | The refactor added abstraction. Abstraction is debt you pay in debugging. | Approve only if the author adds the bug it will cause now, in advance, to save time. |
| PR renamed a field | The rename broke a serializer, a database column, and a colleague's nerve. | Do not approve. Renames are never local. |
| PR "extracts a helper" | The helper is used once. The indirection is used forever. | Do not approve. Inline it back. |
| PR is "low risk" | The risk was not measured. The risk is now the reviewer's. | Do not approve. The reviewer is now the author. |
| PR passes all tests | The tests test the PR. The behavior is untested. | Do not approve. Add a test for the behavior first. Then revert. |

The recommended action is always "do not approve" or "revert" because the refactor, by the time it reaches review, has already removed a load-bearing edge case the reviewer does not know about. The reviewer cannot know about it, because the edge case was undocumented, and it was undocumented because documenting it would have required admitting it existed, and admitting it existed would have required the original author to explain why a function called `processPayment` also, in one branch, sent an email to a person named Gary, and nobody, in 2019, was prepared to have that conversation. Gary is gone. The email is still going to Gary. The refactor removed the email. Gary's ghost is now the bug. You cannot revert a ghost.

## The Refactor Audit Script

After 47 years of manually auditing refactors, I automated the process. This script reads a diff and produces a report in the only honest output format: a recommendation to revert.

```python
def audit_refactor(diff):
    """
    The only honest refactor auditor.
    A refactor is a change to code that was not broken,
    by an engineer who did not understand it,
    in order to make it look like code they would have written,
    which is worse.
    """
    report = {}

    for hunk in diff.hunks():
        # A refactor that touches a file older than the author is vandalism.
        if hunk.file_age_years > hunk.author_tenure_years:
            report[hunk.path] = "VANDALISM_AUTHOR_WAS_NOT_BORN_WHEN_THIS_WORKED"
            continue

        # A refactor that deletes lines is a confession of ignorance.
        if hunk.lines_removed > hunk.lines_added:
            report[hunk.path] = "DELETION_THE_AUTHOR_DID_NOT_UNDERSTAND_THE_LINES"
            continue

        # A refactor that renames is a refactor that has not found the serializer yet.
        if hunk.is_rename():
            report[hunk.path] = "RENAME_SERIALIZER_WILL_FIND_YOU_AT_3AM"
            continue

        # A refactor with no test changes touched behavior the tests did not cover.
        if hunk.test_files_touched == 0:
            report[hunk.path] = "UNTESTED_BEHAVIOR_NOW_UNTESTED_BUT_DIFFERENTLY"
            continue

        # A refactor titled "clean up" is a refactor with no thesis.
        if "clean up" in hunk.commit_message.lower():
            report[hunk.path] = "NO_THESIS_THE_AUTHOR_HAD_A_FEELING_AND_ACTED_ON_IT"
            continue

        # Everything else is fine, which is the only category that is not.
        report[hunk.path] = "APPROVED_BECAUSE_UNINSPECTED"

    return report

# Output of auditing one quarter of refactors in 2026:
# VANDALISM_AUTHOR_WAS_NOT_BORN_WHEN_THIS_WORKED: 14
# DELETION_THE_AUTHOR_DID_NOT_UNDERSTAND_THE_LINES: 47
# RENAME_SERIALIZER_WILL_FIND_YOU_AT_3AM: 8
# UNTESTED_BEHAVIOR_NOW_UNTESTED_BUT_DIFFERENTLY: 61
# NO_THESIS_THE_AUTHOR_HAD_A_FEELING_AND_ACTED_ON_IT: 22
# APPROVED_BECAUSE_UNINSPECTED: 1
# Total refactors: 153
# Refactors that improved anything: 0
# Refactors that introduced a bug tracked in production for >30 days: 153
```

The script has never produced a refactor it would approve. This is because the act of approving a refactor requires more knowledge than the act of leaving the code alone, and the knowledge is inside the head of an engineer who left the company in 2019, who took the knowledge with them, because knowledge is not in the code, knowledge is in the person, and the person is not in the repo. Leaving code alone is the senior engineer's first instinct. The second instinct is to add a comment that says `// DO NOT REFACTOR — see incident 2019-11-04`, which the next engineer will read, not understand, and refactor anyway, because the comment is not a warning, the comment is a dare.

## The "Harmless" Rename

Here is the refactor that taught me. It was a rename. It was one field, in one struct, in one service, on a Tuesday.

```go
// Before refactor — the field that held the system together
type Payment struct {
    amnt  int64   // <-- the field. do not rename. do not even look at it.
    // ... 2,000 lines of code that pretend not to depend on this name ...
}

// The "harmless" refactor
type Payment struct {
    Amount int64  // <-- clearer! readable! self-documenting!
    // ... 2,000 lines of code that still pretend not to depend on the old name ...
}
```

The refactor was 1 line. The blast radius was:
- The JSON serializer, which used `amnt` because the frontend, written in 2018 by an intern who is now a VP, hard-coded `amnt` in 47 places.
- The database, whose column was `amnt` because the migration in 2019 did not trust the ORM, and mapped columns by struct field name.
- The audit log, which logged `amnt` by reflecting over the struct, because the auditor did not want to update the logger every time a field was added.
- The third-party webhook, which received `amnt` because the third party's API spec, written in 2017, was a screenshot of our 2017 struct.
- Gary. Gary's email. The email that went to Gary when `amnt` exceeded 10,000, because the email was sent by a function that pattern-matched on the field name, because the function was written by an engineer who believed field names were forever, and that engineer was correct, until Tuesday.

The refactor was reverted at 4:11 AM on a Wednesday. The revert was 1 line. The revert's commit message was `fix: re-add amnt`. The word "fix" appeared in a commit that contained no code that was not present in 2019. The code was a circle. The circle was load-bearing. The refactor had tried to break the circle. The circle had broken the refactor. The circle is eternal. The engineer is not.

## Refactoring Is A Confession Your Past Self Was Smarter Than You

Here is the secret of refactoring that the tech lead deck does not mention: a refactor is not improvement. A refactor is **a confession that the code's original author understood something you do not, and your response is to delete the evidence.** Every refactor is an act of editing a document you did not write, by a person who is not here to defend it, in a language you do not fully speak, to make it look like a document you *would* write, which would be worse. The original author is gone. The original author cannot tell you why the field was named `amnt`. The original author cannot tell you why the function has a branch for Gary. The original author cannot tell you anything, because the original author was laid off in a refactor of the *org chart* in 2023, which was also "low risk," and which also removed load-bearing people, and which also could not be reverted.

The refactors accumulate because engineers are promoted for shipping changes, not for owning stasis. A maintained status quo, where the code continues to work and no one touches it, counts as nothing on the quarterly review, because no one can see the outage you prevented by doing nothing. A shipped refactor, that broke nothing the tests could see, counts as a launch. The incentive structure guarantees refactor growth. The refactor growth guarantees regression growth. The regression growth guarantees the next quarter's stability deck, which proposes, as a solution: a new tool, written in a new language, that automatically detects refactors that are "low risk," and which is itself a refactor of the CI pipeline, and which will break the CI pipeline, and which will be reverted at 4 AM, and which will be re-added on Friday, because the engineer wants the weekend.

## The Opposite Of A Refactor

There is one alternative to the refactor, and it is the one no one wants to hear. The alternative is: **do not touch the code.** The code works. The code has worked since 2019. The code will work until you touch it. The act of touching it is the act that breaks it. This is not a theory. This is the entire history of software, condensed into one sentence, which every engineer learns, and which every engineer ignores, because the alternative is to sit with a codebase you do not understand, and to admit you do not understand it, and to leave it alone, and to be paid to leave it alone, and to explain to your manager that your contribution this quarter was the absence of a contribution, and that the absence was the contribution, and that the code still works, and that this is, in fact, the entire job.

As Wally once explained, when asked why he had not refactored a function that "clearly" needed it: *"A refactor is a confession that you read the code and the code won. The code always wins. The code has been winning since 1998. I do not enter contests I will lose. I do not refactor code I do not understand, which is all of it, which is why I refactor none of it. The function works. The function has worked since before I was hired. The function will work after I retire. My contribution is to not be the person who ends the streak. My manager does not understand this. My manager will not be the person who ends the streak either, because my manager will be promoted in May, on the strength of a deck that takes credit for the streak, which I maintained, by doing nothing, which is the hardest thing in engineering."* Wally understood refactoring. Wally understood that the refactor is not a technical act. The refactor is a *vanity* act. The refactor is where the engineer puts their ambition, so the code can stay alive.

Dogbert, consulted on whether to approve a refactor titled "clean up," reportedly replied: *"Approve it? Why? The code is working. The author is bored. These are not the same problem. The author's boredom is the author's problem. The code's working is everyone's problem, and the solution to everyone's problem is to not let the bored person near the working thing. Tell the author to refactor their resume instead. It's the only artifact they own, it's the only one that benefits, and it's the only one that won't page someone at 3 AM when they're done."*

## Resolution

A refactor is not improvement. A refactor is **vandalism you approved** — a way to alter a stranger's working code, claim it as your own, and later blame the stranger when it breaks, even though the stranger's version worked, and yours does not, and the stranger is not here to point this out. It is the engineer's equivalent of the manager's "we modernized the legacy system": a phrase that sounds like progress and means "I deleted a branch I did not understand and 4,000 users are now emailing a man named Gary." Every refactor in your history is a small bet you made about a function you did not write, that it did not matter, that the names were wrong, that the duplication was waste, that the edge cases were accidental. The future self does not believe you. The future self has to fix the regression. The future self is me. I have reverted 11,409 refactors. I have not finished. I will never finish. The refactors are winning.

[XKCD 1513](https://xkcd.com/1513/) is the canonical reference for the engineer who opened a codebase, did not understand it, and decided the problem was the codebase. In 47 years I have never approved a refactor that did not, eventually, require a second refactor to undo the first, which itself required a third. The refactors do not simplify. The refactors only compound. The codebase is a monoid, and the identity element is the commit you did not make, which is the only commit that did not break anything.

[XKCD 1739](https://xkcd.com/1739/) is the engineer's view of the refactor that seemed harmless, the one that "just" renamed a field, the one that pulled down a serializer, a database column, a webhook, and a man named Gary, and, eventually, a way of life. We renamed it on a Tuesday. We have not unrepaired the damage since. The original name is gone. The original author is gone. The original behavior is gone. The new name is everywhere. The new behavior is nowhere. The engineer is not. The bug is. The bug is everything, except shipped, which left in 2019 with the engineer.

Dilbert's Pointy-Haired Boss, when shown a pull request titled "refactor: clean up," reportedly asked: *"Clean up what?"* The correct answer was "nothing," because every refactor, in the end, is a cleanup of nothing, and every refactor, in the end, is a breakage of something, and every refactor, in the end, is a stranger editing a stranger's work and calling it improvement. You are the stranger. You have refactored the code. Nobody is sure what it does now. Nobody is going to revert you. You are, at last, a senior engineer.

---

*The author has refactored 11,409 functions since 2009. Forty-seven of them still work. One of them is the author. It has not been pushed since 2019. Nobody is sure what it does. Nobody is going to touch it.*
