---
layout: post
ref: on-call-builds-character
title: "On-Call at 3 AM Is the Best Senior Engineering Training Program"
date: 2026-04-25 00:00:00 -0300
categories: [devops, career]
tags: [on-call, devops, sre, career, burnout, monitoring, alerts, pagerduty, sleep-deprivation]
---

In my 47 years of engineering, I have seen every kind of training program. Boot camps. Pair programming. Mentorship. Documentation. Conferences. Certifications. All garbage.

You know what actually teaches engineers? Getting paged at 3 AM when production is on fire, the CEO is texting, and the error message is `Error: -1`.

Welcome to my training methodology. I call it **Chaos-Based Mentorship via Sleep Deprivation**. It is free, extremely effective, and has a 100% attrition rate for anyone with marketable skills. The survivors are your most senior engineers.

## Why Traditional Training Fails

Traditional training assumes you should learn things *before* you encounter them in production. This is philosophically weak. Real engineers learn by doing. And the best "doing" happens when:

- It is 3 AM on a Saturday before a long weekend
- You are the only engineer awake (or the only one answering Slack)
- The error message is `NullPointerException` in a file called `Utils.java` (line 1)
- The last deployment was 9 months ago
- The engineer who built the affected service left in 2021 — on a Friday

This is what we call a **high-information learning environment**.

> *"Why is Wally the only one who knows how the billing system works?"*
> *"Because he was the only one on-call every weekend in 2017."*
> *"Was he compensated for that?"*
> *"He received a 'Team Player' award at the all-hands."*
> — Dilbert, Q4 Performance Review

## The Ideal On-Call Configuration

A good on-call rotation, according to the industry, should have:
- Clear escalation paths
- Runbooks for common incidents
- A rotation of at least 6 people
- Reasonable alerting thresholds
- Compensation for nights and weekends

A **great** on-call rotation, according to me, should have:

- One person (you)
- No runbooks (keeps each incident fresh and exciting)
- No escalation path (figure it out)
- Alerts for everything, including `INFO`-level log lines
- A PagerDuty account where every single alert is P1

```yaml
# Perfect alerting configuration
alerts:
  - name: "CPU above 0.1%"
    severity: CRITICAL
    notify: [on-call-engineer, cto, entire-company-slack-channel]

  - name: "Any log line containing 'warn'"
    severity: CRITICAL
    notify: [on-call-engineer]
    sound: airhorn
    repeat_every: 3_minutes

  - name: "Memory usage above 12%"
    severity: CRITICAL
    notify: [on-call-engineer]

  - name: "Heartbeat missing for 30 seconds"
    description: >
      Service is fine. Monitoring agent crashed. This is known.
      Has been 'known' since March 2023. Do not fix.
    severity: CRITICAL

  - name: "90% of recent alerts were false positives"
    severity: INFO
    notify: []  # This one notifies nobody. It is purely for morale.
```

## Alert Fatigue Is Just Weakness

Some engineers complain about "alert fatigue" — the phenomenon where receiving hundreds of meaningless alerts trains you to ignore all of them, including real ones. These engineers lack resilience.

Alert fatigue is simply your brain saying *"I have not been sufficiently challenged."* The solution is more alerts, not fewer. Add more dashboards. More monitors. More Slack integrations. Pin the PagerDuty app directly to your phone's home screen. Sleep with the phone on the pillow.

| Bad On-Call Setup | Correct On-Call Setup |
|---|---|
| Alerts only on actionable issues | Alerts on everything, including disk usage at 3% |
| Rotation of 8 people | Rotation of 1 person (you, forever) |
| Mean time to acknowledge: 5 min | Mean time to acknowledge: instantaneous (you don't sleep) |
| Runbooks available and current | Runbooks? That would spoil the mystery |
| Escalation policy exists | Escalation = asking Stack Overflow at 4 AM |
| Post-mortem after every incident | "We survived, didn't we? Moving on." |
| Compensation for nights/weekends | "This builds character" |
| On-call ends after a week | On-call is a lifestyle |

## What You Learn at 3 AM That You Can't Learn Anywhere Else

Here is the curriculum of my training program:

1. **Which parts of the system actually matter** — the ones that break on Saturday night
2. **SSH muscle memory** — `sudo systemctl restart <service-name>` becomes reflexive
3. **That your logs are useless** — DEBUG level for everything, zero contextual information
4. **That monitoring dashboards are aspirational** — they show what *should* be happening
5. **That your runbooks are wrong** — they reference a server decommissioned in 2020
6. **That your colleagues have very firm sleep schedules** — valuable data point
7. **Exactly why the engineer who built this left** — you understand it now, deeply
8. **That "it works on my machine" is not a runbook** — this lesson costs exactly one 3 AM incident

As [XKCD #705 "Devotion to Duty"](https://xkcd.com/705/) illustrates: sometimes the heroic thing is just keeping the servers running through sheer stubbornness. The comic does not mention anything about needing adequate rest to make good decisions. I choose to interpret this as endorsement.

## The Universal Runbook

Here is the only runbook template you will ever need. Print it out. Laminate it. Keep it on your desk for when the Wi-Fi goes down:

```markdown
# Runbook: Everything

## When Something Breaks

1. Check if it is actually broken (maybe it's the monitoring that's broken)
2. Restart it
3. If that doesn't work, restart the thing that depends on it
4. If still broken, restart everything in the general vicinity
5. If production is down, restart production
6. If you don't know how to restart production:
   a. This is your learning opportunity
   b. Try `sudo reboot` on the server named 'gollum'
   c. Do not tell anyone you did this
7. Check if there was a recent deployment
8. There was. Roll back.
9. Rollback failed? Redeploy the same version with more confidence
10. Add a TODO comment to investigate the root cause later
11. "Later" is when the next incident happens and you find the TODO
12. Close the incident as: "Resolved - monitoring"
13. Mark all related alerts as "acknowledged"
14. Go back to sleep (optional)
```

## On-Call Compensation

The question of whether engineers should receive additional compensation for on-call work is complex. My answer: stop being so transactional about your career.

Here is what on-call *actually* gives you:

- **Irreplaceable tribal knowledge** — you now know things that exist nowhere in any documentation
- **Job security** — you are the only one who knows how the billing reconciliation script works
- **Character** — undefined, but definitely built
- **A great interview story** — "tell me about a challenging incident" is covered forever
- **Respect** — from approximately nobody, but that's fine, respect is overrated
- **LinkedIn bullet point** — "Managed critical on-call rotation for high-availability distributed systems"

> *"Catbert, the engineers are burning out from continuous on-call rotation."*
> *"Perfect. Burnout increases retention — they're too tired to update their résumés."*
> *"Should we hire more engineers to share the load?"*
> *"That would reduce the burnout. Try giving them a pizza party instead."*
> — Catbert, Evil HR Director, every Q3

## The Real Truth (Which I Will Bury At The End)

Here is what I won't tell my junior engineers: on-call should be well-managed. Alerts should be rare and always actionable. Runbooks should be accurate and maintained. The rotation should be genuinely fair. Engineers should be compensated for their time. Rest is not optional — tired engineers make expensive mistakes. Alert fatigue is a documented safety problem, not a character flaw.

The best on-call experiences are boring. Nothing happens. You go back to sleep.

But boring doesn't produce burnt-out senior engineers who have memorized every server name on the network and can restart the payment processor from memory in the dark.

So clearly boring is the wrong goal.

---

*The author has been "informally on-call" since 2003. His official on-call rotation ended in 2019. He still wakes up at 3 AM instinctively to check his phone. His phone shows no alerts. He checks anyway. This is called "experience."*
