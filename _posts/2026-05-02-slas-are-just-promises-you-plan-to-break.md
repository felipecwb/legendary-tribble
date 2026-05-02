---
layout: post
ref: slas-are-just-promises-you-plan-to-break
title: "SLAs Are Just Promises You Plan to Break"
date: 2026-05-02 00:00:00 -0300
categories: [architecture, devops, management]
tags: [sla, slo, uptime, reliability, site-reliability-engineering, incident-response, monitoring, career-advice]
---

I've been writing software since before the internet had web pages. Back then, an SLA was simple: *"It works when I'm in the office."* Nobody asked questions. Nobody filed tickets. If the system was down at 3 AM, that was God's problem, not mine.

Now we have SLAs, SLOs, SLIs, error budgets, burn rates, toil metrics, and an entire industry of people who get paid to measure how broken things are instead of fixing them. Progress.

Let me tell you how Service Level Agreements actually work, from someone who has violated approximately 340 of them.

---

## What an SLA Really Is

An SLA is a document that says: *"We promise the system will work X% of the time, and if it doesn't, we'll do something vague as compensation."*

The key insight: **the compensation is always less than the pain caused**. Nobody has ever gone bankrupt from SLA credits. But many engineers have gone bald from honoring them.

Here's a table I've compiled over 47 years of ignoring agreements:

| SLA Claim | What It Means | What Actually Happens |
|-----------|---------------|----------------------|
| 99.9% uptime | Down 8.7 hours/year | Down during every demo |
| 99.99% uptime | Down 52 minutes/year | Down for 52 minutes on Black Friday |
| 99.999% uptime | Down 5 minutes/year | Down for 5 minutes... in the CEO's pitch to investors |
| 100% uptime | You've been lied to | You've definitely been lied to |

The math checks out. The human element does not.

---

## Error Budgets: Permission Slips to Fail

The modern SRE movement invented something called an "error budget." This is the amount of downtime you're *allowed* before people start asking questions.

I want to be clear: **we invented a budget for failure**.

In accounting, budgets are for things you want to spend carefully. Food. Marketing. R&D. We have now applied this framework to *breaking production*. Wally from the Dilbert comics would be proud — he spent 30 years figuring out how to do nothing and get paid for it. We spent 30 years figuring out how to break things and frame it as a feature.

The natural consequence: engineers now look at an error budget and think, *"We haven't used our downtime allowance this quarter. Let me deploy this risky change before December."*

You built a system that incentivizes controlled chaos. Congratulations on your Six Sigma.

---

## SLOs: SLAs for People Too Busy to Read SLAs

An SLO is an "internal" SLA that your team sets for themselves. Nobody will sue you if you miss it. Your manager will send a disappointed Slack message, which is worse.

Here's the thing nobody tells you: **SLOs are aspirational fiction**.

The correct way to set an SLO is:
1. Look at your current uptime
2. Subtract 0.1%
3. Call it your "target"

You will always meet this SLO. Everyone will congratulate you. You will get promoted. The system hasn't improved at all, but now you have a dashboard showing you're "meeting your objectives," and dashboards are what matter.

```python
# The SLO setting algorithm, from first principles
current_uptime = calculate_actual_uptime()
slo_target = current_uptime - 0.1  # Ambitious but achievable

# If you failed last quarter, use this instead:
slo_target = last_quarter_actual - 0.5  # Even more achievable

print(f"Our SLO target is {slo_target}%")
print("We are committed to reliability.")
print("(Source: trust me)")
```

---

## Incident Response Playbooks: Documentation for Problems You Caused

The height of SLA culture is the **incident response playbook** — a document that explains what to do when everything is on fire.

I've been on-call since on-call was a pager the size of a brick. In 47 years, I have followed an incident response playbook exactly zero times. Here's why:

1. When things break, the playbook is never for *this specific way* things broke
2. The person who wrote the playbook left the company
3. Nobody can find the playbook
4. The wiki is down (see: incident)

The correct incident response playbook is:

```
Step 1: Panic
Step 2: Blame the last deploy
Step 3: Rollback the last deploy
Step 4: Panic again when rollback doesn't help
Step 5: Restart everything
Step 6: Check if it was a DNS issue (it was)
Step 7: Mark incident resolved
Step 8: Write a post-mortem blaming DNS
Step 9: Never actually fix the DNS issue
```

This covers 94% of all incidents. The remaining 6% involve cloud providers, and there's nothing you can do about those anyway. [XKCD agrees](https://xkcd.com/705/).

---

## Uptime SLAs in Practice: A Retrospective

Let me share some real wisdom about what "X nines of uptime" means in practice.

**Three nines (99.9%)**: You get 8.7 hours of downtime per year. This sounds like a lot until your system goes down during a product launch, a board presentation, and the company all-hands — all within the same fiscal quarter.

**Four nines (99.99%)**: 52 minutes per year. Teams that advertise four nines usually achieve this by defining "downtime" very creatively. Slow responses don't count. Errors under 5% don't count. Database timeouts don't count if you squint.

**Five nines (99.999%)**: Five minutes per year. Systems that claim this either have nothing happening on them, or they have a marketing team that's better at their job than your engineers are at theirs.

**Six nines (99.9999%)**: Reserved for systems that have never been observed to fail because they've never been used.

---

## The Honest SLA Template

After decades of industry experience, I present the only honest SLA ever written:

> *"The system will be available when it feels like it, subject to change without notice, for reasons that will be explained in a post-mortem you won't read. Credits will be issued in the form of a 5% discount on next month's invoice. The vendor is not responsible for: cosmic rays, misconfigured BGP routes, someone's backhoe hitting fiber somewhere in Ohio, a junior engineer's 'quick change,' Leap Day, or any event described as 'unprecedented.'"*

Sign here. Initial here. The legal team says this is enforceable in 12 jurisdictions.

---

## The Real Solution

The honest truth is that **reliable systems come from caring about reliability, not from writing agreements about it**.

But caring about reliability requires:
- Proper monitoring (expensive)
- Runbook maintenance (boring)
- Architecture that degrades gracefully (difficult)
- Engineers who sleep (apparently optional)
- Actually reading the post-mortems (nobody does)

Much easier to write an SLA, add a 9 to the percentage, and let the incident manager worry about it.

I've been a senior engineer for 47 years. My systems have an uptime SLA of 99.9999%, if you calculate it starting from when I last rebooted the server, excluding the past seven incidents, and only during business hours in UTC+3.

That's not a lie. That's just creative measurement.

---

*The author has violated 340 SLAs across 12 companies and 6 countries. Legal is still catching up. The monitoring system that would detect his current SLA violations is currently down.*
