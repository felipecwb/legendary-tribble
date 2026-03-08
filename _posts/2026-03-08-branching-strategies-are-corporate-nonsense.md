---
layout: post
ref: branching-strategies-are-corporate-nonsense
title: "Branching Strategies Are Corporate Nonsense: Commit to Main"
date: 2026-03-08 09:00:00 -0300
categories: [git, wisdom]
tags: [git, branching, gitflow, trunk-based, devops]
---

GitFlow. GitHub Flow. GitLab Flow. Trunk-based development. Feature branches. Release branches. Hotfix branches. You know what all these have in common? **They're ways to make simple things complicated**.

You want to know my branching strategy? `git push origin main`. That's it. That's the whole strategy.

## The Problem with Branches

Every branch is a parallel universe where your code lives alone, scared, gradually diverging from reality. The longer a branch lives, the more it drifts from `main`. And then comes the merge. The dreaded merge.

```
Feature branch lifecycle:

Day 1: Create branch from main
Day 2: Write code
Day 3: Write more code
Day 4: Pull latest main, resolve 3 conflicts
Day 5: Keep coding
Day 7: Pull latest main, resolve 47 conflicts
Day 10: Give up, create new branch
Day 11: Repeat
```

Know what never has merge conflicts? Code that's already on main.

## Why GitFlow is a War Crime

Let me decode GitFlow for you:

| GitFlow Branch | What It Promises | What Actually Happens |
|---------------|------------------|----------------------|
| main | Production-ready code | Nobody knows what's there |
| develop | Integration branch | Integration nightmares |
| feature/* | Isolated development | Isolation from reality |
| release/* | Stabilization | destabilization |
| hotfix/* | Quick fixes | Quick creation of new bugs |

You need a PhD in Git to understand when to merge what into where. And don't even get me started on the "merge back to develop" step that everyone forgets.

I once worked at a company where a hotfix branch took 3 weeks because nobody could figure out the merge order. The bug was a typo in an error message.

## Trunk-Based Development (But Actually)

People talk about trunk-based development like it's revolutionary. You know what trunk-based development really is? It's what we did in 1997 with CVS before someone invented branches and ruined everything.

My implementation of trunk-based development:

```bash
# Step 1: Write code
vim main.py

# Step 2: Deploy
git add .
git commit -m "stuff"
git push origin main

# Step 3: There is no step 3
```

Some call this reckless. I call it efficient.

## Feature Flags: The Real Solution

"But how do you work on features without feature branches?"

Simple: commit broken code to main and hide it behind feature flags.

```python
if os.getenv('ENABLE_NEW_FEATURE') == 'true':
    # New code that might work
    do_new_thing()
else:
    # Old code that definitely works
    # (we think)
    do_old_thing()
    
# After 6 months, nobody remembers what ENABLE_NEW_FEATURE does
# But we're afraid to remove it
# It stays forever
# This is fine
```

## The Dilbert Test

The Pointy-Haired Boss once asked: "Why do we need all these branches? Just put everything in one place."

For once, he wasn't wrong. Wally's response was enlightening: "Branches let me pretend to work on something for weeks without anyone checking my code."

This is why branches exist. Not for code isolation. For accountability avoidance.

## [XKCD 1597](https://xkcd.com/1597/) Was Right

Nobody understands Git. We all pretend. The more branches you have, the more opportunities to prove you don't understand Git.

My coworker once spent 4 hours trying to merge a feature branch. He ended up deleting the branch, copying the files manually, and committing directly to main.

That's not a failure. That's enlightenment.

## Code Review Without Branches

"But what about code reviews? Don't you need branches for pull requests?"

No. Here's my code review process:

1. Write code
2. Commit to main
3. If tests pass, it's reviewed
4. If tests fail, revert
5. If there are no tests, see step 3

Some call this "continuous deployment." I call it "moving fast."

## The Real Problem

You know what branches protect you from? Bad code getting to production.

You know what else protects you from bad code? Not writing bad code.

The entire branching system exists because we don't trust ourselves. But here's the thing: if you don't trust your code without branches, you shouldn't trust it with branches either. The branch didn't fix the bug—it just delayed finding it.

## My Branching Strategy in Practice

```
Repository: production-critical-app
Branches: 1 (main)
Contributors: 47
Commits per day: 200+
Deploys per day: 50+
Incidents caused by no branching: 3
Incidents blamed on no branching: 47
Incidents actually caused by branching at previous job: "we don't talk about that"
```

## Conclusion

Every branch is technical debt. Every merge is suffering. Every branching strategy is a coping mechanism for a deeper organizational dysfunction.

Push to main. Trust your tests. Fire fast.

---

*The author's last repository had 847 stale feature branches, each representing a developer who left the company before finishing their work. The main branch contained code that "probably works." It did, in fact, probably work.*
