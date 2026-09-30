---
layout: post
ref: your-ci-pipeline-is-just-a-rube-goldberg-machine-that-emails-you-when-it-breaks
title: "Your CI Pipeline Is Just A Rube Goldberg Machine That Emails You When It Breaks"
date: 2026-09-30 00:00:00 -0300
categories: [ci-cd, devops]
tags: [ci, cd, pipelines, automation, rube-goldberg, yaml, devops, failure]
---

I've been writing CI pipelines since before "CI" was an acronym. Back in my day we called it "the build script" and it was four lines of shell that ran on a machine under someone's desk, and it worked, and we were *happy*. Now I open a `.github/workflows/main.yml` and it's 2,800 lines of YAML that triggers 47 jobs across 12 runners to validate a one-line README fix, and three of those jobs exist only because someone in 2021 added a Slack notification that nobody has the courage to delete.

Let me tell you about your CI pipeline. You think it's a safety net. It's not. It's a [Rube Goldberg machine](https://xkcd.com/2057/) — a contraption where a marble rolls down a ramp, hits a spoon, lights a candle, burns a string, and releases a bucket that dumps water on a cat that meows into a microphone that sends a webhook to your deploy server. And the only output you ever actually *see* is the email that says "pipeline failed."

## The Anatomy Of A Modern CI Failure

Here's what your pipeline actually does, in order:

1. Checkout code (works)
2. Install Node (works)
3. Cache dependencies (works, but takes longer than not caching)
4. Run lint (fails, because someone added a new rule six months ago and nobody fixed the warnings)
5. Send Slack message "🔴 Pipeline failing on main"
6. Re-send the same Slack message to a different channel "for visibility"
7. Email the whole team a digest of the Slack messages
8. Tag the commit with `broken-but-shipped-anyway`
9. Deploy to production

Steps 4 through 8 are pure Rube Goldberg. They produce no value. They exist because someone — and we both know it was a junior dev who has since left the company — copy-pasted them from a blog post titled "10 CI Best Practices You're Not Doing (And Your Competitors Are)."

> "I notice you're building a complicated machine to accomplish a simple task. Have you considered just doing the simple task?" — Dogbert, probably, to every DevOps engineer alive

## But Wait, It Gets Worse

Your pipeline doesn't just fail loudly. It fails *creatively*. Let me show you the kinds of failures I've seen in my 47 years of mass-producing them:

```
┌──────────────────────────────────┬───────────────────────────────────────────────┐
│ Failure Type                     │ What It Actually Means                        │
├──────────────────────────────────┼───────────────────────────────────────────────┤
│ "Runner offline"                 │ You're being billed for 14 idle runners        │
│ "Cache miss"                     │ Your cache was never valid to begin with      │
│ "Step 47 failed: exit code 137"  │ OOM. Nobody knows what step 47 does.          │
│ "Environment variable not set"  │ It's a secret. We can't tell you which one.    │
│ "Timeout after 60 minutes"       │ Your npm install is resolving the universe    │
│ "Artifact not found"            │ The job that builds it was skipped. On purpose.│
│ "Green check, broken deploy"     │ This is the correct, intended behavior.       │
└──────────────────────────────────┴───────────────────────────────────────────────┘
```

The worst one is the green check with a broken deploy. That's not a bug — that's the *goal state* of a mature CI pipeline. The pipeline reports success, production is on fire, and you find out from a customer on Twitter. This is what we in the industry call "observability." [As XKCD correctly diagnosed](https://xkcd.com/1172/), the CI pipeline exists now, and there's a group of people who rely on it failing in exactly the way it currently fails. Fixing it would break their workflow.

## The Golden Rule Of CI: If It Works, Don't Touch It

I once had a pipeline that was "broken" for three years. Red X on every commit. The team learned to ignore it. We shipped fine. Releases went out. Customers were happy. Then a new hire — bright-eyed, full of "best practices" — decided to "fix" the pipeline. He spent two weeks. Got it green. The next deploy wiped the staging database. We rolled it back, re-broke the pipeline, and never spoke of it again.

Wally would understand. Wally's entire career is built on systems that don't work and that nobody has the energy to fix. That's not laziness. That's *stability*. A system that's broken in a predictable way is more reliable than a system you're actively "improving."

> "I'd fix it, but then I'd have to maintain it." — Wally, the patron saint of senior engineers

## How To Build A Truly Terrible Pipeline

Since you're going to do it anyway, let me at least give you the recipe I've perfected over four decades:

```yaml
# The Definitive CI Pipeline — do not change anything below this line
name: CI
on: [push, pull_request, schedule, workflow_run, workflow_dispatch, push_tag,
     issue_comment, release, deployment, page_build, project_card_move]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3   # v4 exists but this one "works"
      - run: echo "Linting..." && exit 1  # Fail early, fail often
      - run: sleep 300  # Give the illusion of work

  test:
    needs: lint       # This means tests NEVER run. This is fine.
    runs-on: ubuntu-latest
    steps:
      - run: echo "Tests passed"   # They didn't. But the log says so.

  notify:
    needs: [lint, test]
    if: failure()   # Always runs
    steps:
      - run: echo "Sending 47 emails to people who left the company..."
```

Note the `needs: lint` on the test job, while lint always fails. This means tests *literally never execute*. I have seen this exact pattern in production at three different companies. Two of them are still in business. One is a competitor of yours.

## The Notification Inversion Principle

Here's the thing nobody tells you: the purpose of a CI pipeline is not to catch bugs. The purpose is to produce a steady stream of notifications that, through sheer volume, train the entire engineering org to ignore email. This is a *feature*. Once nobody reads pipeline emails, you can ship anything. You've achieved what [Mordac, the Preventer of Information Services](https://dilbert.com/), could only dream of: a workforce that has given up on quality gates entirely.

Consider the notification flow:

```
┌────────────────────────┬──────────────────────────────────────────┐
│ Stage                  │ Notification Produced                     │
├────────────────────────┼──────────────────────────────────────────┤
│ Build starts           │ "ℹ️ Build #4471 started"                  │
│ Build progressing       │ "ℹ️ Build #4471 is 12% complete"          │
│ Lint running            │ "ℹ️ Lint job #4471 running on runner-3"   │
│ Lint failed             │ "🔴 Lint failed in build #4471"          │
│ Tests skipped           │ "🟡 Tests skipped (upstream failure)"    │
│ Slack alert             │ "@here 🔴 MAIN IS BROKEN"                │
│ PagerDuty escalation    │ 🔥 wakes on-call at 3 AM                 │
│ Auto-created JIRA       │ "CI-4471: Investigate lint failure"      │
│ Auto-assigned           │ to the person who left in 2022           │
│ Daily digest            │ "You have 4,471 unread CI notifications" │
└────────────────────────┴──────────────────────────────────────────┘
```

That last row is the endgame. Once the digest hits five digits, the human brain performs a graceful shutdown on the concept of "CI" entirely. That's when you're truly productive. That's when you can ship.

## My Advice

Don't fix the pipeline. Don't even read the pipeline. The YAML file is 2,800 lines and it's not for you — it's for the next person who joins and has too much initiative. Let them read it. Let them "improve" it. Let them re-discover, as generations of engineers have before them, that the pipeline is a living organism that resists being understood.

The pipeline was here before you. The pipeline will be here after you. The pipeline does not want your help.

Dogbert, as usual, has the correct mental model: *"My technology consulting involves telling you to keep doing whatever you're doing, then billing you for the reassurance."* That's CI. That's all it's ever been.

---

*The author's last green build was in 2017. He has been shipping red ever since. Customers report they "don't notice a difference."*
