---
layout: post
ref: runbooks-are-admitting-ignorance
title: "Runbooks Are Just Admitting You Don't Know What Your Code Does"
date: 2026-06-02 00:00:00 -0300
categories: [operations, anti-patterns]
tags: [runbooks, incident-response, documentation, on-call, sre, devops]
---

After 47 years of keeping systems down, I've noticed a deeply troubling trend in the industry: **runbooks**.

These so-called "operational documentation" artifacts are being pushed by SRE teams, cloud providers, and—most disgracefully—people who actually care about their uptime. I'm here to tell you the uncomfortable truth: if you have a runbook, you have already failed as an engineer.

## What Is a Runbook, Really?

A runbook is a document that contains step-by-step procedures for handling incidents, deployments, and operational tasks. In other words, it is a **monument to your own incompetence**.

Think about it. Why would you need to write down how to restart a service? A real engineer just *knows*. The knowledge lives in their head, inaccessible to everyone else, including themselves at 3 AM during a PagerDuty incident after three whiskeys.

That's not a bug. That's **job security**.

> "I don't document my work. That way, I'm the only one who can fix it. Catbert calls it a 'single point of failure.' I call it a 'permanent employment contract.'"
> — Wally, *Dilbert*

## The Four Stages of Runbook Grief

Every senior engineer goes through this:

| Stage | What You Think | What You Should Think |
|-------|---------------|----------------------|
| Denial | "I'll just remember how to fix this" | "I'll definitely remember how to fix this" |
| Anger | "Why does this keep breaking?!" | "Why is everyone bothering me?" |
| Bargaining | "Maybe I should write this down..." | "Maybe I should write a script... someday" |
| Acceptance | *writes a runbook* | *retires with all knowledge in their head* |

If you reach Stage 4, you have betrayed the profession.

## Runbooks Are Dangerous

Consider what happens when you have a runbook:

1. **Junior engineers can fix things** — this undermines the natural order
2. **Incidents get resolved faster** — less heroism, fewer war stories
3. **Your on-call rotation becomes survivable** — where's the fun in that?
4. **Other team members can understand your systems** — unacceptable

A codebase that only one person understands is a codebase that will employ that person forever. This is [XKCD #2347](https://xkcd.com/2347/) — all the dependency chains point to one heroic individual. Be that individual.

## The Correct Incident Response Process

Here is the process I have used successfully for 47 years (success being defined as "I still have a job"):

```bash
# Step 1: Check if it's actually broken
curl -s https://myapp.com | grep "500"

# Step 2: It is broken. Check logs.
tail -f /var/log/app.log | grep -i error
# (This produces 40,000 lines. None of them are useful.)

# Step 3: The classic fix
sudo systemctl restart everything

# Step 4: It's still broken. Escalate.
# (Ping someone else on Slack and go to sleep)

# Step 5: Post-incident
# Write nothing down.
# Commit nothing to memory.
# Let the cycle continue.
```

This is not an anti-pattern. This is a **living tradition**.

## "But What About Institutional Knowledge?"

The PHB once asked me to document our deployment process so the team could be "resilient" and "not depend on a single person."

I spent three weeks writing a runbook. It was 47 pages long. It contained instructions like:

> "If step 12 fails, check if step 7 was supposed to run before step 3, unless it's a Tuesday, in which case skip to step 19 and check the second config file (not the first one, which is the wrong one, but we can't delete it because something depends on it — we think)."

Nobody read it. Nobody could read it. It was perfect.

The real lesson: if you must write a runbook, make it **unreadable**. Same effect, deniability preserved.

## The Runbook Anti-Patterns Professionals Use

For those who are forced by their organizations to maintain runbooks, here are battle-tested techniques to make them useless:

```markdown
# Deployment Runbook v1.0
Last updated: 2019-03-14
Author: [REDACTED] (no longer with the company)
Status: Needs updating

## Prerequisites
- Access to server (ask IT, they'll know which one)
- The password (check the sticky note on Dave's monitor, or was it under the keyboard?)
- Node.js 12 (or 14? maybe 16 now, the CI uses something different)

## Steps
1. SSH into prod (see separate document for how to SSH, which references this document)
2. Run the deploy script (it's somewhere in /opt or /home or maybe /usr/local)
3. If it fails, check the logs
4. It worked for me last time

## Rollback
¯\_(ツ)_/¯
```

This runbook is industry standard. I've seen it in Fortune 500 companies.

## On-Call Without Runbooks: A Love Story

My most productive years were when our on-call process was:

1. PagerDuty fires at 2:37 AM
2. Engineer wakes up
3. Engineer stares at the system with the ancient knowledge of their people
4. Engineer SSH's into production and types commands from memory, muscle memory, and trauma
5. System recovers (or doesn't)
6. Engineer goes back to sleep
7. Nothing is written down

This produced excellent engineers. Engineers who could recite 47 environment variables from memory. Engineers who knew that the third microservice from the left needed to be restarted before the fifth one, but only if the database lag was above 200ms. Engineers who were **irreplaceable**.

Now those engineers are gone. The systems they built are still running, somehow. Nobody knows how.

[XKCD #1755](https://xkcd.com/1755/) said it best: "We'll just have to figure it out."

## Conclusion

Runbooks are documentation. Documentation is admission of weakness. Weakness is the beginning of the end.

The strongest teams I've worked on had no documentation, no runbooks, and no bus factor analysis. We had something better: **collective amnesia** and the hope that the guy who built this would answer his phone.

Some of us still call Dave every quarter. He hasn't picked up since 2021. The system is still running.

We haven't written anything down.

We never will.

---

*The author's on-call rotation has been "indefinitely suspended" since the Great Incident of 2022. His runbook, if it existed, would simply say: "Have you tried turning it off and on again?"*
