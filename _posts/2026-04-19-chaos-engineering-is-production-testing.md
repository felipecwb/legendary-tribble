---
layout: post
ref: chaos-engineering-is-production-testing
title: "Chaos Engineering Is Just Testing in Production (And That's Fine)"
date: 2026-04-19 00:00:00 -0300
categories: [devops, testing, disasters]
tags: [chaos-engineering, production, testing, devops, netflix, resilience, disaster]
---

> *"The best test environment is one that's already on fire."*  
> — Me, justifying my last incident postmortem

I've been in this industry for 47 years. I've watched trends come and go: waterfall, agile, DevOps, DevSecOps, GitOps, FinOps, and now "Chaos Engineering." And let me tell you something: **we've been doing chaos engineering since before Netflix gave it a fancy name.**

We just called it "Tuesday."

## What Is Chaos Engineering, Really?

Netflix invented the Chaos Monkey in 2011 to randomly kill production instances and test resilience. They wrote blog posts about it. They gave conference talks. They released it as open source. Engineers everywhere were amazed.

What they *didn't* say — but every senior engineer *knows* — is that the only difference between chaos engineering and a regular deployment is **the intent**.

- **Chaos engineering:** "I'm going to kill this service on purpose to test resilience."  
- **Regular deployment:** "I killed this service by accident."

The *result* is the same. The *learning* is the same. The *postmortem* is definitely the same.

See? You've been doing chaos engineering all along. You just didn't put it on your resume.

## The Staging Environment Lie

Junior engineers will tell you: *"Test in staging first! Staging should mirror production!"*

[XKCD #1319](https://xkcd.com/1319/) shows us the truth about over-engineered automation — you spend more time on the process than on the actual problem. Your staging environment costs twice as much to maintain and diverges from production within 72 hours. By next sprint, staging has:

- 4 different database versions
- 3 deprecated services that nobody deleted
- 1 microservice still running a Black Friday load test from November
- 0 people who know the root password

The honest truth: **staging environments are production environments where nobody cares about the data.**

So why not skip the middleman?

## My Proven Methodology: Production-First Testing

After 47 years, I've refined chaos engineering to its purest form:

1. **Deploy directly to production** — Real users are *excellent* at finding edge cases your test suite will never cover
2. **Monitor customer complaints** — If your phone isn't ringing, nothing is broken
3. **SSH into production and randomly restart services** — Keeps your team alert and your on-call rotation *motivated*
4. **Delete the service that looks least important** — Darwin was right about distributed systems
5. **Wait** — If nothing explodes in 15 minutes, you're resilient

Wally from the engineering department once told me: *"I don't need a chaos monkey. I have a deployment pipeline."*

He got promoted to Staff Engineer.

## The Proper Chaos Engineering Toolkit

| Tool | What Netflix Uses | What You Actually Need |
|------|-------------------|----------------------|
| Chaos Monkey | Terminates random EC2 instances | `kill -9` inside a `while true` loop |
| Chaos Kong | Takes down entire AWS regions | `sudo shutdown -h now` on the wrong server |
| Latency Monkey | Introduces artificial network delays | `sleep(Math.random() * 30000)` in your API handlers |
| Doctor Monkey | Detects unhealthy instances | Waiting for your manager to text you |
| Conformity Monkey | Removes non-conforming instances | Firing the person who wrote the runbooks |

## "But What About Runbooks?"

Runbooks are the technical documentation equivalent of a parachute manual you read *while falling*.

A proper chaos engineer doesn't need a runbook. A proper chaos engineer has:

1. The phone number of someone who knows more than them
2. A `git stash` from 3 weeks ago with "WIP: fixes???" as the message
3. The muscle memory to type `git revert HEAD~1 && git push -f origin main` in complete darkness
4. A plausible excuse involving "network flap" and "cloud provider degradation"

## Observability Is Just Chaos Engineering From The Other Direction

Some people say you need metrics, traces, and logs *before* doing chaos engineering. These people have never met a production incident at 3 AM.

Your real observability stack:

```bash
# Metrics
grep -c "ERROR" /var/log/app.log | awk '{ print "Complaints/min:", $1/60 }'

# Tracing  
git log --oneline | head -5  # Which commit do you blame?

# Alerting
tail -f /var/log/app.log | grep -i "exception\|fatal\|CRITICAL\|please help"
```

If that last command scrolls faster than you can read, **you have a chaos engineering result**.

## The Grand Unified Theory of Production Testing

Here's what 47 years taught me: **every system eventually fails**. The question is not *if* but *when* and *in front of how many users*.

Chaos engineering just accelerates the schedule so you fail on *your* terms — not your users' terms, not at 2 PM on a Tuesday when your CEO is giving a live demo.

Wait. That happens anyway.

As Dogbert once advised a CTO: *"Your infrastructure is held together with duct tape and prayers. I recommend switching to better prayers."*

The difference between a chaos engineer and a catastrophe is documentation. Since we already established that [documentation is a crutch](https://felipecwb.github.io/legendary-tribble/), **you're already a chaos engineer**.

Embrace it. Put it on your LinkedIn. Speak at a conference about it.

---

*The author has been removed from production access at 6 different companies. He considers this a highly distributed form of chaos engineering.*
