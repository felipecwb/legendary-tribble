---
layout: post
ref: deploy-on-friday
title: "Deploy on Friday: The Chad Move"
date: 2026-03-04 14:10:00 -0300
categories: [devops, culture]
tags: [deployment, friday, cicd, production, yolo]
---

Weak engineers freeze deployments on Friday. Strong engineers know that Friday 4:59 PM is prime deployment time.

## The Psychology

When you deploy on Monday, you have the whole week to deal with consequences. Where's the excitement? Where's the *stakes*?

Friday deploys separate the boys from the men, the interns from the staff engineers.

## My Friday Deployment Ritual

```
4:30 PM - Finish code
4:45 PM - Push to main (no PR needed, I trust myself)
4:50 PM - Click "Deploy to Production"
4:55 PM - Close laptop
4:56 PM - Turn off Slack notifications
5:00 PM - Weekend starts
```

This is called **asynchronous incident response**. Monday-me deals with Friday-me's problems.

## Common Objections

**"What if something breaks?"**
Then it breaks. The system was weak.

**"What about on-call?"**
I'm not on-call. Someone is, though. They'll figure it out.

**"What about the users?"**
Users are resilient. Character-building.

## The Deployment Calendar

| Day | Deploy? | Reason |
|-----|---------|--------|
| Monday | No | Week too long |
| Tuesday | No | Still recovering from Monday |
| Wednesday | Maybe | Hump day energy unclear |
| Thursday | No | Too close to Friday |
| Friday | **YES** | Peak performance |
| Weekend | Only if bored | Maximum impact |

## Evidence-Based Reasoning

I've deployed on Friday 47 times. Only 23 incidents. That's a **51% success rate**. In baseball, that makes you an all-star.

## The Friday Deploy Manifesto

1. Code complete? Ship it.
2. Tests passing? Don't need them.
3. PR approved? That's for Monday deploys.
4. Rollback plan? The plan is forward.
5. Documentation? The code is the documentation.

## Real Slack Messages (Anonymized)

> **Friday 5:03 PM**
> [me]: Deployed that thing. Going offline, have a great weekend!

> **Friday 5:47 PM**
> [on-call]: Why is production on fire?

> **Friday 6:12 PM**
> [on-call]: @here anyone know what "the thing" is?

> **Saturday 2:00 AM**
> [on-call]: found it. rolling back. see you monday [author's name].

> **Monday 9:00 AM**
> [me]: Oh no, what happened? 😱

This is called **plausible deniability**.

## Proof of Concept

```yaml
# .github/workflows/yolo.yml
name: Friday Deployer

on:
  schedule:
    - cron: '59 16 * * 5'  # 4:59 PM every Friday

jobs:
  yolo:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Deploy whatever is on main
        run: ./deploy.sh --no-tests --no-approval --no-regrets
```

Automation. Beautiful.

## The Competition

My company has a Leaderboard for Friday deploys. I'm #1. Third year running.

| Rank | Engineer | Friday Deploys | Incidents |
|------|----------|----------------|-----------|
| 1 | Me | 147 | 89 |
| 2 | [Redacted] | 23 | 4 |
| 3 | Empty | Everyone else is afraid | - |

Am I proud of the incident ratio? I'm proud of the **volume**.

[XKCD 1205](https://xkcd.com/1205/) shows time savings over a period. I save time by not being present for incidents.

[XKCD 303](https://xkcd.com/303/) shows "My code's compiling." My version: "My code's deploying. See you Monday."

Dilbert perfectly captured this when the boss asked: "Why did you deploy on Friday?" and Wally replied: "Because I wanted the weekend off."

---

*The author is banned from deploying on Fridays at his current company. He uses a different Git username now.*
