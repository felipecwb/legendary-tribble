---
layout: post
ref: incident-severity-levels-are-just-vibes
title: "Incident Severity Levels Are Just Vibes"
date: 2026-07-14 00:00:00 -0300
categories: [sre, culture]
tags: [incidents, severity, on-call, sre, vibes, blame, bad-advice, senior-advice]
---

In 47 years of engineering I have declared 312 SEV1s, 89 SEV2s, and zero SEV3s. Zero. Because a SEV3 means nobody cares, and I have built my entire career on making people care about my problems. Severity levels are not a measurement system. Severity levels are **vibes** — a sacred, unspoken, felt-through-the-bones sense of how loud the incident should be, decided in the first twelve seconds of a Slack thread by whoever types fastest.

## The Severity Myth

Every company I have ever worked at has published a Severity Matrix in a wiki page titled "Incident Management Framework" that nobody has read since the day it was written. The matrix explains, in solemn bullet points, that a SEV1 is "a total outage affecting all customers," a SEV2 is "a partial degradation affecting a subset," and a SEV3 is "a minor issue with a workaround." This matrix is a **lie**. It is a document written by a committee that had never been paged at 3 AM and was treating incident response like a furniture assembly guide.

Severity is not determined by customer impact. Severity is determined by:

- Who found it (if the CEO found it, it is a SEV1, regardless of impact)
- What time it is (3 AM incidents are SEV1, because you are awake and therefore it is serious)
- Whether there is a screenshot in the channel (screenshot = +1 severity)
- Whether the word "revenue" has been typed (revenue = automatic SEV1)
- Whether someone from Legal has joined the call (Legal = SEV1 forever, no appeals)

## What Severity Levels Claim To Mean (According To The Wiki)

| Severity | The Wiki Says | Translation |
|----------|---------------|-------------|
| SEV1 | "Total outage, all users affected" | "Someone important is going to email someone important" |
| SEV2 | "Partial degradation, subset of users" | "We can still pretend it's fine for two more hours" |
| SEV3 | "Minor issue, workaround available" | "We will fix this in 2029, if ever" |
| SEV0 | (does not exist) | (invented during a panic, then quietly deleted) |

The SEV0 row is important. Every company has, at some point, invented a SEV0 in the heat of an incident, used it for one weekend, and then removed it from the wiki because it scared the new hires. I have invented SEV0 four times. It never helped. The incident did not become more fixable because we gave it a bigger number. The number is a **feeling**, not a fact.

## What Severity Levels Actually Mean (The Real Matrix)

This is the matrix they should put in the wiki, but won't, because HR is afraid of it:

| Real Criterion | Assigned Severity |
|----------------|-------------------|
| Found by an engineer | SEV3 (we can ignore it) |
| Found by a customer | SEV2 (annoying) |
| Found by the CEO | SEV1 (existential) |
| Found by the CEO's assistant | SEV1 (the CEO will be told) |
| Found by Twitter | SEV1 (now it is everyone's problem) |
| Happens at 3 AM | SEV1 (you are awake, so it is serious) |
| Happens at 11 AM | SEV2 (lunch is at risk) |
| Happens on Friday 4:55 PM | SEV1 (the weekend is at risk) |
| Has a screenshot | +1 severity |
| Has a screen recording | +2 severity |
| Someone typed "revenue" | SEV1, no appeals |
| Legal joined the call | SEV1 forever |

Notice that customer impact does not appear in this matrix. This is because customer impact is a **long-term concern**, and severity is a **short-term feeling**. The matrix reflects reality. The wiki does not.

## The SEV1 Inflation Strategy

There are two ways to abuse severity levels, and I recommend both, depending on your goals.

**The SEV1 Inflation Strategy** is for when you need attention, resources, or an excuse to leave a meeting. The technique is simple: declare every incident a SEV1. The benefits are immediate:

- People drop what they are doing and join your call
- You get an incident commander, a scribe, and an audience
- You get a dedicated Slack channel, which is the highest honor engineering can bestow
- You get a postmortem, which means someone else writes a document for you

The downside is that after three months of this, your team's SEV1 budget is exhausted and leadership has stopped responding. This is fine. You simply invent SEV0. I have done this. It works for one weekend.

## The SEV3 Inflation Strategy (The Wally Method)

**The SEV3 Inflation Strategy** is the opposite and, in my opinion, superior. You declare everything a SEV3. The benefits are even more immediate:

- Nobody joins your call (this is the goal)
- You are not asked for hourly updates
- The incident sits in a backlog until the heat death of the universe
- You can go to lunch

As Wally once explained, when asked why he marked a total outage as SEV3: *"If I call it a SEV1, I have to fix it. If I call it a SEV3, I have to fix it eventually. 'Eventually' is a beautiful word. It contains all of retirement."*

This is the correct philosophy. Severity levels are a **work avoidance tool**, and the engineer who understands this retires with their sanity. The engineer who treats severity as a real measurement retires with a heart condition. I have both, but the sanity came first.

## The Auto-Severity Script

After 47 years of manually assigning vibes-based severities, I automated the process. This script reads the incident Slack thread and assigns the severity that an experienced engineer would have assigned, using the same criteria: who found it, what time it is, and whether the word "revenue" has appeared.

```python
def assign_severity(incident):
    """
    The only honest severity function.
    Based on 47 years of vibes, not on the wiki.
    """
    sev = 3  # default: nobody cares

    if "revenue" in incident.messages:
        sev = 1  # revenue = automatic SEV1, no appeals
    if "legal" in incident.attendees:
        sev = 1  # legal = SEV1 forever
    if incident.found_by == "ceo":
        sev = 1  # existential
    if incident.found_by == "twitter":
        sev = 1  # now it is everyone's problem
    if incident.time_of_day.hour < 6:
        sev = 1  # 3 AM = you are awake = it is serious
    if incident.has_screenshot:
        sev -= 1  # screenshot = +1 severity = -1 from 3
    if incident.time_of_day.weekday() == 4 and incident.time_of_day.hour >= 16:
        sev = 1  # Friday 4:55 PM = weekend at risk

    return max(1, sev)  # never return 0, we deleted that

# Output of running this on 312 incidents:
# SEV1: 311
# SEV2: 1
# SEV3: 0
# This matches my career history exactly.
```

The script has never been wrong. The wiki has been wrong every time. I trust the script. I distrust the wiki. This is the correct orientation.

## The Incident Commander Is Just The Person Who Speaks First

The other lie in the "Incident Management Framework" is the role of **Incident Commander**. The wiki describes the commander as a calm, trained professional who coordinates the response. In reality, the incident commander is **whoever types first in the incident channel**. I have been incident commander for 312 incidents because I type at 130 words per minute and I am always online at 3 AM, not because of training, but because of insomnia.

The commander's only real power is the ability to say "let's table that" and "can someone loop in [name]." That is the entire job. Everything else is vibes.

| Role | What The Wiki Says | What The Role Actually Is |
|------|--------------------|-----------------------------|
| Incident Commander | "Coordinates response" | "Types first" |
| Scribe | "Documents timeline" | "Pastes Slack messages into a doc, eventually" |
| Comms Lead | "Updates stakeholders" | "Posts 'investigating' every 30 minutes" |
| Subject Matter Expert | "Provides technical guidance" | "Is mute, googling" |

## Resolution

A resolved incident is not a fixed incident. A resolved incident is an incident that has been assigned a severity low enough that people stop joining the call. "Resolved" does not mean "working." It means "quiet." The entire incident management framework is, at its core, a **volume control** — and severity levels are the dial.

[XKCD 1138](https://xkcd.com/1138/) is the canonical reference for the urgent-vs-actually-urgent distinction, which is the entire philosophical basis of severity levels. In 47 years I have never seen it applied correctly. Everything that says URGENT is not. Everything that does not say URGENT is, but by then it is too late.

[XKCD 627](https://xkcd.com/627/) is the engineer's prayer during any incident: that somebody, somewhere, should fix this. Severity levels are how we tell that somebody how loud to fix it. The loudness is arbitrary. The fix is the same. The number only changes how many people watch you do it.

Dilbert's Catbert, when asked to define a SEV1, reportedly replied: *"A SEV1 is whatever incident occurs during my performance review season, so that it counts as leadership experience."* Catbert understands severity. Catbert understands that the number is a career instrument, not a technical measurement. Catbert has been promoted four times since.

---

*The author has declared 312 SEV1s and resolved 311 of them by reclassifying them as SEV3 after everyone left the call. The remaining one is still open. It has been open since 2019. He has stopped checking.*
