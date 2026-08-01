---
layout: post
ref: git-stash-is-where-code-goes-to-die
title: "Git Stash Is Where Code Goes to Die"
date: 2026-08-01 00:00:00 -0300
categories: [version-control, anti-patterns, culture]
tags: [git, git-stash, version-control, productivity, procrastination, debugging, workflow, culture, legacy-code, psychology]
---

After 47 years of writing code professionally, I have created exactly 14,000 `git stash` entries. I have recovered three of them. The other 13,997 are still in the list, somewhere, doing nothing, taking up no disk space but occupying a permanent, low-grade anxiety in the back of my skull whenever I run `git stash list` and watch the scrollbar move.

`git stash` is sold as a temporary holding area — a shelf, a pocket, a drawer — for work you're not ready to commit. In practice, it is a graveyard with a `--keep-index` flag. You put code in there because you don't want to deal with it now, and you don't deal with it later, because later never comes, and when later does come, the stash has been sitting so long it's fossilized and the fossil doesn't apply cleanly to the branch anymore.

## The Five Stages of `git stash`

Every engineer travels the same road. I have watched it 14,000 times.

**Stage 1 — Pragmatist.** You're in the middle of something. A hotfix lands. You don't want to lose your work. You think: *I'll just stash this real quick.* You run `git stash`. You feel efficient. You feel like a person who finishes things. You are not.

**Stage 2 — Forgetful.** The hotfix ships. You move on to the next thing. The stash sits. Days pass. You do not remember there is a stash. You start a new branch. You write new code. The old code, the stashed code, ages in place like a container of yogurt in the back of the office fridge that everyone sees and no one claims.

**Stage 3 — Discoverer.** Weeks later, you run `git stash list` by accident, or because your IDE helpfully shows a "3" badge next to a menu item you never click. You see `stash@{0}: WIP on main: abc1234 stuff`. You do not remember what `stuff` was. You do not remember `abc1234`. You do not remember writing it, or why, or on which branch you intended it to live.

**Stage 4 — Archaeologist.** You investigate. You run `git stash show -p stash@{0}`. You see a 600-line diff. It touches 14 files. Three of those files have since been renamed. Two have been deleted. One now has a different language's code in it because someone "did a little experiment." You cannot `pop` this stash. The stash is a fossil. The fossil does not fit the socket.

**Stage 5 — Deceiver.** You have two options: spend an afternoon resurrecting the stash and reconciling it with four months of changes, or delete it and move on with your life. You choose neither. You leave it. You tell yourself *I'll get to it during the cleanup sprint.* There is no cleanup sprint. There has never been a cleanup sprint. The cleanup sprint is a myth, like the 25-hour day or the bug-free release. The stash joins the 13,996 others, and you add a 14,000th the next time a hotfix lands.

## Why `git stash` Exists

`git stash` exists because `git checkout` would overwrite your uncommitted changes, and Linus, in his mercy, gave you a way to not lose them. This was a kindness. It is also, like all kindnesses, the seed of a problem. Before `git stash`, you had to make a decision: commit the half-baked work, or throw it away. Both options forced honesty. `git stash` removed the honesty. It gave you a third option: *don't decide*. And the engineer, confronted with a decision, will always take the third option, even when the third option is "put it in a drawer and feel bad about it forever."

The proof is in the numbers. After 47 years, I have stashed 14,000 times. The mean lifetime of a stash in my repository is 11 months. The median is forever.

## The Stash Lifecycle, Visualized

Let us trace where a stash actually goes:

| What you think happens | What actually happens | Final state |
|---|---|---|
| "I'll pop it tomorrow" | Tomorrow you start a new branch. The stash stays. | Aging |
| "It's a clean holding area" | The stash now conflicts with 4 months of main. | Fossilized |
| "I'll apply it to the right branch later" | You forgot which branch was right. There is no right branch. | Orphaned |
| "git stash is reversible" | `git stash drop` is permanent. You're too scared to drop. | Permanent |
| "It takes no space" | It takes no disk space but permanent psychological space. | Tax on your soul |
| "The stash is temporary" | The stash has outlived two reorgs and a CEO. | Senior to you |

Notice the final column. No stash has ever reached "popped and merged." Popping a stash is a myth, like a clean merge or a productive standup. I have done it three times in 47 years, and twice I regretted it because the resurrected code was worse than the problem it was meant to solve.

## The `git stash` Workflow I Recommend

The unenlightened use `git stash` to save work. The enlightened use `git stash` to *avoid* work. After 47 years, I have refined the workflow:

```bash
# 1. Stash the thing you don't want to deal with.
git stash push -m "wip before hotfix"

# 2. Forget about it. This is the most important step.
#    Do NOT add it to a todo list. Do NOT set a reminder.
#    A reminder defeats the entire purpose, which is to not decide.

# 3. Months later, discover the stash list is now 47 long.
git stash list

# 4. Attempt to recover one. Fail. It conflicts.
git stash pop stash@{12}
# Auto-merging file_that_has_since_been_renamed.js
# CONFLICT (content): Merge conflict in ...
# (The stash is KEPT on conflict, so it is now MORE permanent, not less.)

# 5. Abort the merge. Leave the stash exactly where it was.
git merge --abort

# 6. Accept your fate. The stash is permanent. Add it to the list.
#    Do NOT `git stash drop`. Dropping is a decision. Decisions are for the brave.
```

Step 6 is the most important step, and the one I have executed 13,997 times. `git stash drop` is the only `git` command I have never successfully run, because running it would require me to be sure the stash is worthless, and I am never sure, because I never remember what was in it. This is, I have come to understand, the entire point.

## What `git stash` Is Actually For

Here is the secret the Git documentation refuses to print: `git stash` is not for saving work. `git stash` is for *generating a list of things you have given up on without admitting you have given up on them.*

- It is not a shelf. A shelf implies you will return. You will not return.
- It is not a backup. A backup implies the work is valuable. If it were valuable, you would have committed it.
- It is not a draft. A draft implies revision. You will not revise it. You will not even look at it.
- It is a *grave marker*. It marks the place where a piece of work died, and you walk past it every time you open the repo and feel a pang you cannot explain.

Once you accept this, the tool becomes liberating. You are not hoarding code. You are *memorializing* it. Each stash is a little headstone: *Here lies a refactor, 2024, beloved by no one, interrupted by a hotfix, never resumed.* You do not desecrate a grave by dropping it. You also do not visit. You simply let the cemetery grow.

## The Economics of Stashing

A junior will object: *"But isn't it cheaper to stash than to commit a WIP?"* No. And I can prove it with the only economics I remember from this industry, which I have stolen directly from [XKCD #1205, "Is It Worth the Time?"](https://xkcd.com/1205/). Randall taught us: you can justify spending *X* hours automating a task based on how often you do it. The inverse applies here. You can justify stashing for *Y* seconds of effort saved now only if you will *pop* it within a window that, in my data, has a median of *never*.

Let's run the numbers, the way the spreadsheet I keep telling myself I'll open does:

| Action | Time, now | Time, later | Total |
|---|---|---|---|
| `git stash` (the lie) | 3 sec | 2 hours (archaeology, conflict resolution, regret) | 2h 0m 3s |
| `git commit -m "wip"` (the honest) | 5 sec | 0 sec (you rebase it later, like an adult) | 5 sec |
| Delete the work (the brave) | 1 sec | 0 sec | 1 sec |
| Stash, then lie about it at standup | 3 sec | 2 hours + 15 min of perjury | 2h 15m 3s |

The table is clear. The cheapest option, by a margin of two hours, is to either commit the work-in-progress or throw it away. We do neither. We stash. Because stashing is not an economic decision. It is an *emotional* one. The stash exists so that we may tell ourselves, *I didn't lose the work. It's in the stash.* This sentence is technically true and practically worthless, like most sentences that begin with *"technically."*

## Dilbert Understood This First

Wally, the patron saint of engineers who have transcended effort, would not stash. Wally would commit the half-baked work directly to `main` with the message *"wip"* and go to lunch. Wally understands that a stash is a *commit you have to feel guilty about twice* — once when you create it, and once when you discover it. A commit on `main` is felt about zero times, because nobody reads `main`, and the guilt is distributed across the entire team, where it belongs.

Dogbert, who is smarter than every CTO I have ever been interviewed by, would frame it sharper: *"Why preserve work you have no intention of finishing? The stash is a monument to indecision. Monuments are for the dead. Bury it properly, or stop paying rent on the grave."* After 47 years, I can confirm I have been paying rent on 14,000 graves, and the landlord is my own refusal to type `git stash drop` and press Enter, because pressing Enter is a decision, and decisions are irreversible, which is exactly the thing I used the stash to avoid in the first place.

Catbert, Evil Director of HR, would simply add the stash list to my performance review as *"a comprehensive log of the employee's abandoned initiatives, sorted by date of abandonment."* He would not be wrong. The stash list is the most honest document in the repository. It is a more accurate CV than the one I submitted.

## A Real-World Success Story

In 2017, I stashed a 2,000-line refactor of the billing service before a hotfix. I told myself I'd pop it Friday. Friday did not come. The hotfix became a quarter. The quarter became a year. The year became a reorg. The billing service was rewritten from scratch by a team that did not include me, because I had, by then, been reassigned to the stash of engineers — the bench, the shelf, the drawer — which is, I now realize, just `git stash` for humans.

In 2024, I ran `git stash list` in that repo for the last time, before leaving the company. There were 89 entries. `stash@{0}` was the 2017 billing refactor. I ran `git stash show -p stash@{0}`. It was beautiful. It was correct. It would have saved the rewrite team six months. It also would not have applied to a single file in the repo, because every file it touched had been deleted, renamed, or replaced by a microservice written in a language that did not exist in 2017.

I dropped it. It was the first `git stash drop` of my career. I felt nothing, and then I felt everything, and then I felt nothing again, which I have come to understand is the correct emotional sequence for all software decisions.

The rewrite shipped. It had bugs. My stash would have had different bugs. The user does not care which bugs. The user has never cared. The user is on a beach, like the person who should have been blamed for the hotfix, and the user is not thinking about you.

## Common Objections, Obliterated

**"But what if I *do* need the work later?"** You will not. If you needed it later, you would remember it now. The work you remember is the work you commit. The work you stash is the work you have already, in your heart, abandoned. The stash is just the paperwork.

**"What about `git stash -u` to include untracked files?"** Congratulations. You have now immortalized the *build artifacts* too. Your stash now contains a 400 MB `node_modules` delta from a dependency you have since upgraded twice. The stash is no longer a fossil. It is a *core sample* of your dependency history, and it will, one day, be the largest object in your `.git` directory, and `git gc` will weep.

**"Shouldn't I just use a WIP branch instead?"** You should. You will not. A WIP branch requires naming it, and naming is a decision, and decisions are the enemy. The stash requires no name beyond `WIP on <branch>`, which is not a name, it is an *alibi*. You will take the alibi. You always take the alibi.

**"Can't I clean up my stash list during a slack week?"** There is no slack week. There has never been a slack week. The slack week is a second myth, adjacent to the cleanup sprint. The slack week is the time you would spend cleaning the stash, and you will spend it instead *adding to the stash*, because a slack week is, by definition, a week in which you start things you do not finish.

## Conclusion

`git stash` is a mausoleum that costs you nothing to build and everything to visit. You will put code in it. You will not take code out. The list will grow. It will outlive your employment at every company you join. When you retire, your final `git stash list` will be 14,000 entries long, and your successor will run it once, gasp, close the terminal, and add `stash@{0}: WIP on main: the previous guy's stuff` to the top.

Run it anyway. Run `git stash` for the ritual. Run it so that, when the hotfix lands and you are not ready, you may say to your rubber duck — the only honest witness in the room — *"I saved it. It's in the stash. I'll get to it later."* And you will not get to it later. And the duck will know. And the duck, like the stash, will say nothing, and keep everything, and judge you in silence.

The work is not lost. The work is *entombed*. There is a difference, and the difference is that you feel worse about the second one.

[XKCD #1205](https://xkcd.com/1205/) does the math so you don't have to, and the math says you should have just committed it. [XKCD #1597](https://xkcd.com/1597/) already told you who wrote the stash. It was you. It is always you. The stash is where you put the proof.

---

*The author has 14,000 stashes across 11 repositories and has popped three of them. The rest are load-bearing in the sense that they support the author's self-image as a person who "might come back to it." He will not come back to it. He has committed this article as `"wip"` and then stashed the edit. It is now `stash@{0}`. It will be there when you read this. It will be there when the sun dies.*
