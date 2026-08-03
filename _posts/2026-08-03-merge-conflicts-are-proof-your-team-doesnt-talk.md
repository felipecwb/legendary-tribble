---
layout: post
ref: merge-conflicts-are-proof-your-team-doesnt-talk
title: "Merge Conflicts Are Proof Your Team Doesn't Talk"
date: 2026-08-03 00:00:00 -0300
categories: [git, teamwork, anti-patterns]
tags: [git, merge-conflicts, teamwork, communication, branching, workflow, sarcasm, collaboration]
---

After 47 years of mass-producing bugs, I have resolved roughly 400,000 merge conflicts. I have read every one. I have never once read a merge conflict that contained a surprise I was happy about. Every merge conflict is the same artifact: two people, working in the same code, on the same week, who did not know the other existed. The conflict markers — those `<<<<<<< HEAD` and `>>>>>>> feature` fences Git so thoughtfully sprays across your file — are not a technical problem. They are a *social* one, and Git is just the messenger, and as is traditional, you are about to shoot the messenger.

Let me explain why merge conflicts are the most honest thing in your entire engineering organization, and why you should have more of them.

## The Honest Definition of a Merge Conflict

Your team's retrospective calls it "tooling friction." Your engineering manager calls it "a sign we need to rebase more carefully." The conflict itself calls it this:

```
<<<<<<< HEAD
  const isLoggedIn = user?.auth?.token?.isValid();
||||||| merged common ancestors
  const isLoggedIn = user.auth.token.isValid;
=======
  const isLoggedIn = !!user.session;
>>>>>>> feature/new-login
```

That is not a conflict between two functions. That is a conflict between two *realities*. Head believes the user is an optional chain of nullable hopes. The feature branch believes the user has a session that is truthy or it isn't. The common ancestor — the last time these two branches agreed on anything, which was a Tuesday in March — believed the token would simply always exist, because back then we were all younger and more wrong. Three versions of the world, in one file, and Git has politely refused to pick one. Git is the only entity in this company with the integrity to admit it does not know.

Your job, as the engineer assigned to resolve this, is to guess which of the three realities is least wrong, paste it in, delete the fences, and push before anyone notices you didn't actually understand any of the three.

## What They Call It vs. What It Is

I have sat through 47 years of renaming conflict. Here is the translation table:

| What the team calls it | What it actually is | How it got there |
|---|---|---|
| "Trivial conflict" | One of you refactored, the other didn't | Someone has been sitting on a branch for 6 weeks |
| "A tricky merge" | Both of you renamed the same function to different names | Neither of you read the other's PR |
| "Semantic conflict" | It merged cleanly but nothing works | Git can read text. Git cannot read intent. |
| "I'll just rebase onto main" | You are about to re-live every conflict, sequentially | `git rebase` is `git merge` with extra steps and extra suffering |
| "Let's use a merge commit" | We will preserve the conflict forever in the history | Future archaeologists will not understand this decision either |
| "We need better tooling" | We need to talk to each other | The tooling is fine. The people are the tooling. |

Read the last row twice. Teams that resolve conflicts fast resolve them fast because the two authors are already on a call, because they already knew they were touching the same file, because they already agreed on the shape of the change *before* they both changed it. The tool is not resolving the conflict. The *hallway* resolved the conflict three days ago. The tool is just confirming it.

## The Three-Day Branch Is the Only Safe Branch

Here is the only piece of advice in this article that is secretly good, so I will bury it under a terrible framing: a branch older than three days is no longer a branch. It is a *fork*. You are developing on a fork. You are just doing it politely, with a `git pull` at the end and a prayer.

```bash
# Day 1: optimistic
git checkout -b feature/payment-redirect
# Day 2: productive
# (work)
# Day 6: confident
git rebase origin/main
# 47 conflicts. In the file you wrote. Against the file you wrote.
```

The longer a branch lives, the more of `main` it diverges from, and the more the resolution stops being "which code is right" and starts being "which author is still at this company." I have resolved conflicts against branches whose authors had *already left the firm*. I had to guess what a departed engineer meant by `// TODO: fix this properly later (DO NOT MERGE)`. "Later" had come and gone. "Properly" was never defined. Git, faithful as a dog, preserved the instruction across the years so that I, a stranger, could fail to honor it.

## The Phantom Conflict

The worst conflict is the one Git doesn't even flag. This is the *semantic conflict*. Branch A changes a function's behavior. Branch B changes a caller of that function. Git sees two files, two changes, no overlap, merges clean, ship it. Production then behaves like two people who were both sure they had the right-of-way at the same intersection.

```python
# main, after a "clean" merge:

# Branch A changed this:
def get_user(id):
    return db.users.find_one(id) or GHOST_USER  # "graceful degradation"

# Branch B changed this (in a different file, no conflict!):
def render_header(user_id):
    u = get_user(user_id)
    return f"Welcome back, {u.display_name}"   # GHOST_USER has no display_name
```

There is no `<<<<<<<` fence here. There is no `=======`. There is only a `NoneType has no attribute 'display_name'` in production at 4:47 PM on a Friday, discovered by a customer named in the subject line of the email you are about to receive. Git merged it clean because Git does not understand your program. Git understands *text*. This is the entire, unfixable, beautiful flaw at the center of version control, and no amount of "smarter merge tools" will ever close it, because closing it would require Git to *know what your code means*, and if it knew that, it would be the senior engineer and you would be the tool.

## How I Resolve Conflicts (The Wrong Way)

I have a process. I do not recommend it. I will describe it anyway, because you do it too and it is time someone said it out loud.

1. Open the conflicted file in the editor.
2. See `<<<<<<< HEAD`.
3. Panic.
4. Search Slack for the last person who touched this file.
5. They left the company in Q1.
6. Pick the side that *looks* newer. This is a guess. It is usually wrong.
7. Delete the fences.
8. `git add` the file without re-reading it.
9. The tests are broken, but you don't run the tests, because running the tests would mean *reading the tests*, and the tests are also in a merge conflict.
10. Push. Open a PR titled "resolve merge conflicts."
11. Approve it yourself, because nobody else is awake at this hour.

This is not engineering. This is *archaeology with deadlines*. You are excavating two civilizations that never met and forcing them to share a capital city before lunch.

## The Dilbert View

Wally does not get merge conflicts. Wally gets conflicts *removed*, by the simple expedient of never touching any file anyone else might touch. Wally's entire branch touches one file — `wally_utils_v3_FINAL_FINAL.js` — and that file is imported by nothing and read by no one. Wally's `git status` is a clean room. When asked why his feature isn't in main, Wally says, "it's on my branch, I'm waiting for the conflicts to settle." The conflicts will never settle. Wally knows this. This is Wally's retirement plan.

The Pointy-Haired Boss resolves conflicts by forwarding the PR to both authors with the note *"you two sort this out,"* which is, technically, the correct and only useful action a manager can take in a merge conflict, and which PHB arrived at by accident, as he arrives at all correct actions.

Dogbert, who has the only functioning consultancy in the strip, would sell a service called "Conflict Resolution as a Service" in which he, for $40,000 a month, reads the diff, picks the longer side on the grounds that "more code is more committed code," and bills a per-line fee. He would guarantee a 100% conflict-resolution rate, which he achieves by clicking "Accept Current Change" on every hunk regardless of content. His clients would renew. His clients always renew.

Catbert, Director of HR, would observe that merge conflicts correlate with two engineers being assigned to the same area without being told the other existed, which is a management failure, and would therefore recommend *more management*, because the solution to people not talking is always more people not talking but with calendars.

Mordac, Preventer of Information Services, would resolve the conflict by locking the repository and requiring a Jira ticket, a security review, and a wet-ink signature to merge anything, which would eliminate merge conflicts entirely by eliminating merges entirely. This is, I will admit, an effective strategy, and I have seen entire enterprises adopt it voluntarily under the name "release trains."

## Common Objections, Filed and Ignored

**"But trunk-based development eliminates conflicts!"** It eliminates *most* conflicts by making the conflicts *smaller and more frequent*. You have not removed the conflict. You have *sliced* the conflict and spread it across the day. The aggregate suffering is conserved. You have simply made it harder to notice, which is, I'll grant, a form of progress.

**"Rebasing keeps history clean."** Rebasing rewrites history so that the conflict appears to have never happened. This is not cleanliness. This is *denial*. A clean history is a history in which everyone pretends the team coordinated. A dirty history is the honest one. I prefer the honest one, which is why no one invites me to code review.

**"What about merge tools and 3-way diffs?"** A 3-way diff shows you three versions of the truth and asks you to reconcile them with a mouse. The mouse does not help. The mouse only makes you feel like you are operating machinery, which is the same comfort a toddler gets from a toy steering wheel. The car is not turning. The conflict is not resolving. You are clicking "theirs," then "ours," then "theirs" again, and calling it engineering.

**"Surely we should just communicate better."** Yes. This is the objection that proves the article. The fix for merge conflicts is a 20-minute conversation on the day you start the work. Every other fix — the tooling, the rebase policy, the merge button, the CODEOWNERS file — is a substitute for that conversation, and every one of them fails in the exact same way: by assuming the conversation happened, when it did not, which is how you got the conflict in the first place.

## Conclusion

A merge conflict is the machine pointing at the gap between what your team *said* it was doing and what your team *actually* did. The `<<<<<<< HEAD` is not a bug in Git. It is Git refusing to lie for you. Git has decided, charitably, that it will not silently concatenate two incompatible truths into one file and call the result software. That act of restraint is the most respectful thing any tool in your stack does for you, and you repay it by cursing at it and installing a GUI.

The next time you see a wall of conflict markers, do not reach for the merge tool. Reach for the person. Their Slack status says "in a meeting." The meeting is about a different conflict. There is always another conflict. The conflicts are the communication you forgot to have, archived in your repository, one `=======` at a time.

[XKCD 1597, "Git,"](https://xkcd.com/1597/) is the version where you discover that the tool you trusted with your history is itself a trust fall, and you are the one falling. [XKCD 1296, "Git Merge,"](https://xkcd.com/1296/) is the version where the conflict is between you and the concept of time. A merge conflict is the same joke, but the two timelines are your coworkers, neither of them called ahead, and the punchline is in your `blame` on Monday morning.

---

*The author has never met a merge conflict he couldn't make worse. His resolution strategy — "accept theirs, run the app, hope" — has been adopted by the entire org. He has not been thanked. He has been promoted. The system rewards what it rewards.*
