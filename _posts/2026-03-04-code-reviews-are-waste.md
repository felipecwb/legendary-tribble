---
layout: post
title: "Code Reviews Are Just Socially Acceptable Gatekeeping"
date: 2026-03-04 14:15:00 -0300
categories: [career, culture]
tags: [code-review, pr, github, gitlab, gatekeeping, productivity]
---

Code reviews are what happens when companies don't trust their engineers but won't admit it.

## The Review Cycle

1. Write code (2 hours)
2. Open PR (5 minutes)
3. Wait for review (3 days)
4. Address "nit" comments (4 hours)
5. Wait for re-review (2 days)
6. Reviewer goes on vacation (5 days)
7. Merge conflicts (2 hours to fix)
8. New reviewer restarts from step 3
9. Finally merge (total: 2 weeks for a 30-line change)

I could have written the entire feature from scratch in the time spent waiting.

## Types of Code Reviewers

### The Archaeologist
"This is fine, but in the legacy module we follow a different pattern that was established in 2014 by someone who doesn't work here anymore."

### The Philosopher
"But have you considered the epistemological implications of using `const` here?"

### The Nitpicker
"Line 47: Missing blank line. Line 89: Extra blank line. Line 147: Wrong number of blank lines."

### The Ghost
Assigned to your PR 6 days ago. Last active: 5 days ago. Will review: never.

### The Approver
Comments "LGTM 👍" without opening any files. The hero we need.

## Things Reviewers Say

| Comment | Translation |
|---------|-------------|
| "Interesting approach" | Wrong |
| "Have you considered...?" | Do it my way |
| "Nit:" | I have nothing useful to say |
| "For future reference..." | I'm better than you |
| "LGTM" | I didn't look |
| "Let's discuss offline" | I want to argue without evidence |

## My PR Strategy

```markdown
## PR Title: [DO NOT REVIEW] WIP experimental draft

This gets no attention for weeks.

## PR Title: Quick fix

This gets 7 comments about architecture decisions.

## PR Title: URGENT PRODUCTION FIX

This gets approved in 4 minutes.

Make everything an urgent production fix.
```

## The Rubber Stamp Alternative

```python
# .github/auto-approve.yml

def review_code(pr):
    if pr.author == "senior_engineer":
        return "LGTM"
    if pr.lines_changed < 100:
        return "LGTM"
    if "urgent" in pr.title.lower():
        return "LGTM"
    if datetime.now().weekday() == 4:  # Friday
        return "LGTM"  # No one wants to deal with this
    return "LGTM"  # Default to trust
```

## The Meeting That Should Have Been a Merge

I once spent 45 minutes in a meeting about whether to use `map` or a `for` loop.

The result? We used `for` because "it's more readable for juniors."

The junior who joined later refactored it to `map` because "it's more modern."

The 45 minutes could have been a merge either way.

## What I Actually Learn From Code Reviews

- What my reviewer had for lunch (angry comments = hungry reviewer)
- Which arbitrary style rules my company invented
- How long people take to notice things
- Nothing about code quality

## The Alternative

What if — hear me out — we just trusted engineers?

Controversial, I know.

[XKCD 1725](https://xkcd.com/1725/) shows what happens when you try to change standards. Code reviews are just people arguing about standards without admitting it.

Dilbert once asked: "Why do we need code reviews?" The answer: "So we have someone else to blame."

---

*The author's last PR had 87 comments. 84 were about import order. The 3 functional bugs made it to production.*
