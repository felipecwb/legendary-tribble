---
layout: post
ref: code-freeze-should-be-permanent
title: "Code Freeze Should Be Permanent — If It Works, Never Touch It"
date: 2026-03-20 00:00:00 -0300
categories: [bad-advice, devops]
tags: [code-freeze, deployment, change-management, production, stability]
---

After 47 years of watching perfectly good systems get destroyed by "improvements," I've concluded that the best code freeze is an **eternal** one.

You know what never breaks? Code that nobody touches. You know what always breaks? Code that someone "just wanted to refactor a little bit."

## The Beautiful Logic of Never Changing Anything

Your production system has been running for 3 years without a deployment. Users occasionally complain, but they've also learned to work around the bugs. **That's called user adaptation.**

Then some junior developer comes along wanting to "fix a small bug" and "maybe update a few dependencies while we're at it."

Next thing you know, production is on fire, the CEO is asking what happened, and you're rolling back to... wait, what's a rollback?

## Types of Code Freezes

| Freeze Type | Duration | Success Rate | My Opinion |
|-------------|----------|--------------|------------|
| **Holiday Freeze** | 2 weeks | 60% | Too short |
| **Quarterly Freeze** | 3 months | 75% | Getting warmer |
| **Annual Freeze** | 1 year | 90% | Reasonable |
| **Permanent Freeze** | Forever | 100% | **PERFECT** |

The math doesn't lie. More freezing = less breaking.

## Why Unfreezing is Dangerous

[XKCD 1205](https://xkcd.com/1205/) talks about how long you can work on automation to save time. But what about the inverse? **How much time do you LOSE every time you deploy?**

- Writing the code: 2 hours
- Code review: 4 hours
- Waiting for CI: 1 hour
- Debugging CI failures: 3 hours
- Waiting for QA: 2 days
- Fixing QA findings: 1 day
- Deployment itself: 15 minutes
- Rollback when it fails: 2 hours
- Post-mortem: 4 hours
- "Learning from our mistakes": infinite

Total: **Approximately your entire quarter.**

You know what takes zero time? **Not deploying.**

## The PHB Was Right All Along

In Dilbert, the Pointy-Haired Boss often wants to stop engineers from making changes. We laughed at him. We mocked his ignorance.

But consider: every time Dilbert proposed a change, something went wrong. Every time Wally refused to do anything, the system kept running. **Wally is the hero we need.**

The PHB's intuition to prevent change isn't incompetence — it's **institutional wisdom** developed over years of watching engineers break things.

## My Frozen Architecture

Here's my production system, unchanged since 2009:

```
┌─────────────────────────────────────────────┐
│           DO NOT TOUCH                      │
│                                             │
│   ┌─────────┐    ┌─────────┐    ┌──────┐   │
│   │ Apache  │───▶│   PHP   │───▶│MySQL │   │
│   │  2.2.3  │    │  5.2.17 │    │ 5.0  │   │
│   └─────────┘    └─────────┘    └──────┘   │
│                                             │
│   Last deploy: March 2009                   │
│   Status: WORKS (DO NOT ASK HOW)            │
│                                             │
└─────────────────────────────────────────────┘
```

Are there security vulnerabilities? **Probably.**
Are there known exploits? **Definitely.**
Is it still running? **Absolutely.**

And that's what matters.

## The "Just One Small Change" Fallacy

Every disaster starts with "just one small change":

```bash
# The developer's thought process
"I'll just update this one library..."
"Oh, it needs a newer version of this other library..."
"Hmm, that requires a new compiler..."
"The new compiler needs a newer OS..."
"The newer OS doesn't support our hardware..."
"I should have just left it alone."
```

This is called **dependency cascade**, and the only solution is to **never start**.

## What About Security Patches?

Security patches are a scam invented by vendors to make you buy new licenses.

Think about it: your old, unpatched system has been "vulnerable" for 15 years, and hackers haven't exploited it yet. Either:

1. The vulnerability isn't real
2. Hackers can't find your server
3. Your data isn't worth stealing
4. You're protected by obscurity

All of these are **valid security strategies**.

## The Wisdom of the Ancients

```
                    +----------------+
                    | Year 1: Deploy |
                    +--------+-------+
                             |
                             v
                    +----------------+
                    | Year 2: Freeze |
                    +--------+-------+
                             |
                             v
                    +----------------+
                    | Year 3: Forget |
                    +--------+-------+
                             |
                             v
                    +----------------+
                    | Year 4+: Denial|
                    +----------------+
```

This is the natural lifecycle of healthy software. Deploy once, freeze immediately, forget how it works, deny that you ever deployed it.

## How to Implement Permanent Freeze

1. **Lose the deployment credentials** — Can't deploy if you can't log in
2. **Delete the CI/CD pipeline** — Manual deploys only... then lose the manual process
3. **Fire the DevOps team** — No ops, no changes
4. **Brick the servers** — Fill all USB ports with epoxy, remove network cables (except production traffic)
5. **Classify the codebase** — Mark it "LEGACY CRITICAL" and watch everyone avoid it

## But What If We NEED to Change Something?

You don't. Whatever "business requirement" they're pushing can be handled by:

- Training users differently
- Changing the documentation
- Creating a workaround script
- Blaming another team
- Waiting for the requester to quit

Eventually, every change request goes away if you ignore it long enough.

## The ROI of Doing Nothing

| Action | Risk | Effort | Outcome |
|--------|------|--------|---------|
| Deploy change | High | High | Unknown |
| Discuss change | Medium | Medium | Meeting |
| Document change | Low | Medium | PDF |
| **Refuse change** | Zero | Zero | Status quo |

The choice is obvious.

---

*The author's production environment has been frozen since the Cold War. It still runs COBOL and accepts punch cards, but it's stable.*
