---
layout: post
ref: pull-requests-are-bureaucracy
title: "Pull Requests Are Bureaucracy: Just Push to Main"
date: 2026-03-09 08:09:00 -0300
categories: [engineering, workflow]
tags: [git, pull-requests, code-review, workflow, productivity]
---

In the old days, you wrote code and deployed it. Now you need two approvals, a passing CI pipeline, and a Jira ticket before you can fix a typo. After 47 years of shipping code, I know the truth: **pull requests are performative security theater. Just push to main.**

## The PR Dance

```
Time to fix a typo: 30 seconds

Time to fix a typo with PRs:
1. Create branch: 1 minute
2. Make change: 30 seconds
3. Push: 30 seconds
4. Create PR: 2 minutes (write description, link Jira)
5. Wait for CI: 15 minutes
6. CI fails because unrelated test is flaky: 30 minutes debugging
7. Ping reviewer: 1 minute
8. Wait for review: 4 hours (reviewer is in meetings)
9. Address nitpick comment: 5 minutes
10. Wait for re-review: 2 hours
11. Approved, but now merge conflicts: 20 minutes
12. Re-run CI: 15 minutes
13. Merge: 30 seconds

Total: 1 day
Bureaucratic overhead: 99.9%
```

## The Approval Illusion

"LGTM 👍"

That's what 95% of code reviews say. [XKCD 1513](https://xkcd.com/1513/) on code quality perfectly captures this—reviews catch formatting issues, not bugs.

| What Reviews Catch | Percentage |
|-------------------|------------|
| Actual bugs | 5% |
| "Add a comment here" | 30% |
| "Use const instead of let" | 25% |
| Style nitpicks | 35% |
| The thing that breaks prod | 0% |

## The Wally Approval Strategy

Wally from Dilbert would love code reviews. Maximum effort appearance, minimum actual work. Open PR, scroll to bottom, "LGTM." You've contributed to the team! Time for coffee.

## Trunk-Based Development

Here's what real engineers did before GitHub invented PRs:

```bash
# The old way (efficient)
git commit -m "fix bug"
git push origin main
# Done. Users see fix immediately.

# The new way (corporate)
git checkout -b fix/JIRA-4572-fix-button-color-issue
git commit -m "JIRA-4572: Fix button color issue"
git push origin fix/JIRA-4572-fix-button-color-issue
# Create PR
# Wait
# Wait
# Wait some more
# Merge
# Users see fix in 3 days
```

## The Two-Reviewer Rule

Some companies require two reviewers. You know what happens?

```
Developer 1: "I assume Developer 2 checked it carefully"
Developer 2: "I assume Developer 1 checked it carefully"
Both: "LGTM"

Result: Nobody actually read the code
```

It's diffusion of responsibility with extra steps.

## My Push-to-Main Workflow

```bash
# Step 1: Write code
vim app.py

# Step 2: Test locally (maybe)
python -c "import app; print('seems fine')"

# Step 3: Deploy
git add -A
git commit -m "changes"
git push origin main

# Step 4: Monitor production
# If red, push another commit
# If green, take a break

# Average deploy time: 2 minutes
# Bugs caught before users: Same as with PRs (almost zero)
```

## The CI Bottleneck

```yaml
# .github/workflows/pr.yml
name: PR Checks

on: [pull_request]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps: # 5 minutes

  test:
    runs-on: ubuntu-latest
    steps: # 20 minutes
    
  security-scan:
    runs-on: ubuntu-latest
    steps: # 10 minutes
    
  integration-test:
    runs-on: ubuntu-latest
    steps: # 30 minutes
    
  build:
    runs-on: ubuntu-latest
    steps: # 15 minutes

# Total: 80 minutes per PR
# Your fix: 1 line
# Your sanity: Gone
```

## The "Junior Developers Need Reviews" Argument

"But juniors need code reviews to learn!"

```
What juniors learn from PRs:
1. How to write PR descriptions
2. How to respond to nitpicks
3. How to wait patiently
4. How to rebase
5. How to squash commits
6. How to format code to match linter
7. Actual programming: 0%

What juniors learn from pushing to main:
1. How to fix bugs fast
2. How to own their mistakes
3. How production works
4. Actual programming: 100%
```

## The Hotfix Exception

Every company has this rule: "PRs for everything, except emergencies." Which means:

```javascript
// Normal development
if (pr_required) {
    // Wait 8 hours for review
}

// Production on fire
if (emergency) {
    // Push directly to main
    // The same way we should always do it
    // Suddenly PRs aren't important?
}
```

If direct pushes are safe during emergencies, they're safe always.

## Feature Branches Are Just Merge Conflicts

```bash
# Day 1: Create feature branch
git checkout -b feature/new-dashboard

# Day 5: Still working, main has moved
git merge main
# Resolve 47 conflicts

# Day 10: Almost done, main has moved again
git merge main
# Resolve 128 conflicts

# Day 15: PR created
"This PR has conflicts with main. Please resolve."
# Give up, create new branch, start over
```

Just push small changes to main daily. No conflicts.

## The Honest Git History

```
# PR history (fiction)
a1b2c3d - feat: Add user authentication
d4e5f6g - feat: Add dashboard
h7i8j9k - fix: Resolve user session issue

# Actual history (squashed away)
- "WIP"
- "WIP 2"  
- "fix tests"
- "actually fix tests"
- "address review comments"
- "address more review comments"
- "please work"
- "final version"
- "final version 2"
- "okay THIS is final"
```

---

*The author has had push access to main since 1989. In 35 years, he has broken production exactly 847 times. He considers this an acceptable rate.*
