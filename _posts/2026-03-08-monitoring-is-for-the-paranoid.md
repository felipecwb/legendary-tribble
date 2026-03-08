---
layout: post
ref: monitoring-is-for-the-paranoid
title: "Monitoring is for the Paranoid: If It Works, It Works"
date: 2026-03-08 09:30:00 -0300
categories: [devops, anti-patterns]
tags: [monitoring, observability, alerts, devops, production]
---

Every week there's a new "observability" tool promising to show you what's happening in your system. Datadog, New Relic, Prometheus, Grafana, Honeycomb, Lightstep, SignalFx. You know what all these tools have in common? **They show you problems you didn't know existed and didn't need to know about**.

In my 47 years of mass-producing bugs, I've learned one fundamental truth: **If users aren't complaining, the system is working**.

## The Monitoring Paradox

Here's the thing about monitoring: the more you monitor, the more problems you find. And the more problems you find, the more work you have to do. This is a trap.

| Monitoring Level | Problems Found | Sleep Lost | Business Value |
|-----------------|----------------|------------|----------------|
| None | 0 | 0 hours | Maximum |
| Basic | 5 | 2 hours | Some |
| Comprehensive | 47 | 6 hours | Negative |
| "Full Observability" | 847 | All of it | You quit |

The correlation is clear: more monitoring equals less happiness.

## My Monitoring Philosophy

```bash
#!/bin/bash
# monitor.sh - Enterprise monitoring solution

curl -s https://myapp.com > /dev/null
if [ $? -eq 0 ]; then
    echo "It's fine"
else
    echo "It's probably fine"
fi
```

That's it. That's the whole monitoring stack. No agents. No collectors. No dashboards. Just vibes.

## The Dashboard Industrial Complex

You know what dashboards are? They're screensavers for operations teams. "Look at all these graphs moving! We're definitely doing something!"

Let me translate what those dashboard metrics actually mean:

| Metric | What They Say | What It Means |
|--------|---------------|---------------|
| CPU: 75% | "High load!" | The server is doing its job |
| Memory: 80% | "Memory pressure!" | RAM is being used (good, you paid for it) |
| Disk: 60% | "Filling up!" | Files exist |
| Error rate: 2% | "Alert!" | 98% of requests work perfectly |
| Latency p99: 2s | "Slow!" | 99% of requests are under 2 seconds (fine) |

Every metric can be spun as a problem. That's how monitoring vendors stay in business.

## Alerts Are Sleep Deprivation

You know what PagerDuty really is? It's a subscription service for anxiety. You pay $30 per user per month to be woken up at 3 AM because CPU hit 85% for 30 seconds.

```
3:00 AM: ALERT - Memory usage above 80%
3:01 AM: ALERT - Memory usage above 80%
3:02 AM: ALERT - Memory usage above 80%
3:03 AM: You're now awake, angry, and unable to sleep
3:04 AM: Memory stabilizes at 79%
3:05 AM: RESOLVED - Memory usage normal
4:30 AM: You finally fall back asleep
6:00 AM: Wake up for work, exhausted
9:00 AM: Make a mistake in production because tired
9:01 AM: ALERT - Error rate spike
```

The monitoring caused the incident. Think about that.

## What Dilbert Taught Me

Wally once explained his monitoring strategy: "I set all alert thresholds to 100%. That way, nothing ever alerts, and I never get paged."

The Pointy-Haired Boss asked, "But what if something goes wrong?"

Wally replied, "Then the users will tell us. They're very good at that."

This is the way.

## The XKCD Truth

[XKCD 1831](https://xkcd.com/1831/) shows us that we're terrible at understanding what's actually important. You'll monitor the database connection pool but miss that someone renamed the login button to "Log Out."

Monitoring gives you false confidence. You think you're watching everything, but you're watching what you thought to measure. The real problems are always somewhere else.

## User Monitoring: The Only Monitoring That Matters

Here's my monitoring stack:

```
Customer Service:    "Are users complaining?"
                            |
                            v
                     +------+------+
                     |             |
                    Yes           No
                     |             |
                     v             v
              "Fix something"  "Ship features"
```

This system has a 100% true positive rate. If users are complaining, there's a real problem. If users aren't complaining, there isn't.

Compare this to Prometheus, which has a 12% true positive rate and a 100% rate of waking me up for nothing.

## The Cost of Observability

Let's do the math on a typical monitoring setup:

```
Datadog: $15/host/month × 50 hosts = $750/month
New Relic: $25/host/month × 50 hosts = $1,250/month
Log storage: $500/month
Alert fatigue: 10 hours/week × $75/hour = $3,000/month
Coffee to stay awake on-call: $200/month

Total monthly cost: $5,700

Alternative: Trust your users to email you
Total monthly cost: $0

Annual savings: $68,400
```

That's a senior developer's salary you're spending on dashboards nobody looks at.

## When Monitoring Actually Helps

1. Never
2. When you need to prove to management that the outage wasn't your fault
3. When you need graphs for a post-mortem
4. Still never for actually preventing problems

## The Zen of Not Monitoring

```
No monitoring.
No alerts.
No dashboards.
No problems.

If it crashes, restart it.
If it keeps crashing, more RAM.
If that doesn't work, rewrite it.
If that doesn't work, different job.
```

This philosophy has sustained me for decades. The system either works or it doesn't. Staring at graphs won't change that.

## Conclusion

Monitoring is a coping mechanism for organizations that don't trust their code. If your code was good, you wouldn't need to watch it so closely.

Ship confident code. Trust that it works. Go to bed at a reasonable hour.

---

*The author's production systems have no monitoring. When asked how he knows they're working, he says "I don't hear screaming." This has been accurate 97% of the time.*
