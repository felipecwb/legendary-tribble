---
layout: post
ref: remote-work-kills-accountability
title: "Remote Work Killed My Best Engineers (And Yours Too)"
date: 2026-04-29 00:00:00 -0300
categories: [career, management]
tags: [remote-work, career, management, terrible-advice, productivity, surveillance]
---

*Career advice from a Senior Engineer who once sent a Slack message to someone sitting 3 feet away*

---

Back in the golden era of software engineering — roughly 2007 to 2012, before humanity made a series of catastrophic decisions — engineers sat together in an open office. You could watch their screens. You could hear their keystrokes. You could see, at a glance, if someone was "in the zone" or updating their LinkedIn profile at 2 PM.

Those were **productive** times.

Then remote work happened. And I've spent the last several years watching the industry confuse "flexibility" with "disappearing."

## The Visibility-Productivity Equation

"Async communication respects people's deep work," say people who haven't shipped anything since 2021.

Here's what 47 years in the industry has taught me: **if I can't see you, you're not working**.

I've formalized this into what I call the *Senior Engineer Productivity Formula*:

```
P = (H × K × W) / D

Where:
  P = Productivity
  H = Hours physically in office
  K = Audible keystrokes per minute
  W = Number of browser windows open (more = busier)
  D = Distance from manager's desk (in meters)
```

Remote workers break this formula entirely. No audible keystrokes. No visible windows. No measurable distance. Therefore: no productivity. The math is airtight.

"But I shipped three features this week!" they cry.

Features can be written in advance and strategically drip-committed throughout the day. I've seen it done. In my professional opinion, that is **commit fraud**, and it is an epidemic.

## The CS Problem

I've previously written about [real engineers playing CS at 3 AM](https://felipecwb.github.io/legendary-tribble/). Remote workers are simply doing this at **2 PM**. On company time.

To combat this, I built the following employee monitoring solution over a weekend:

```python
#!/usr/bin/env python3
# Remote Worker Accountability Monitor v4.2.1
# Legal team says it's "probably fine."
# HR has not responded to my emails.

import random
import time

ENGINEERS = ["Alice", "Bob", "Carlos", "Priya", "Zhang Wei"]

def assess_productivity(name: str) -> dict:
    """
    Comprehensive productivity assessment using enterprise-grade heuristics.
    Patent pending (application rejected twice, re-filing under different name).
    """
    return {
        "slack_status": "Active",          # They set this to auto-update. Means nothing.
        "last_commit": "3 days ago",        # Verified manually by scrolling GitHub
        "camera_on": False,                 # Always False. "Bad lighting."
        "background_noise": "Suspiciously quiet",
        "productivity_score": 0,            # They're playing CS
    }

def generate_check_in_message(name: str, attempt: int) -> str:
    """
    Passive-aggressive management communication engine.
    Tuned over 11 years of remote team trauma.
    """
    templates = [
        f"Hey {name}! Just checking in 😊",
        f"{name}, quick sync when you have a sec?",
        f"@{name} wanted to make sure you saw my last 6 messages",
        f"{name}, are you blocked? Let me know if you need help unblocking!",
        f"Quick question for {name} — are you... alive?",
        f"{name} I notice you haven't committed in 3 days. Everything ok? 😊😊😊",
    ]
    # Use escalating concern as attempt count increases
    return templates[min(attempt, len(templates) - 1)]

def monitor():
    attempt_counter = {name: 0 for name in ENGINEERS}
    while True:
        for engineer in ENGINEERS:
            result = assess_productivity(engineer)
            if result["productivity_score"] == 0:
                msg = generate_check_in_message(engineer, attempt_counter[engineer])
                print(f"[SLACK] {msg}")
                attempt_counter[engineer] += 1
        
        time.sleep(480)  # Every 8 minutes. They'll know you're watching.
                         # That's the point.

monitor()
```

The script has been running since March 2022. All five engineers have since quit. **Correlation is not causation.**

## The Proximity-Quality Matrix

After decades of empirical research (my gut), I've established the definitive relationship between physical distance and output quality:

| Distance from Manager | Code Quality | Ticket Response | Path to Promotion |
|---|---|---|---|
| 0m (same desk, breathing on neck) | Excellent (fearfully correct) | 4 minutes | 6 months |
| 10m (same floor, visible) | Good | Same day | 1 year |
| 100m (different building) | Concerning | 2 days | 3 years, maybe |
| 500m (works from coffee shop) | Suspicious | 1 week | "Let's revisit in H2" |
| **At home (remote)** | **Playing CS** | **"I'll check"** | **Terminated** |
| Different timezone | Beyond saving | "What's the urgency?" | Already quit |
| Different country | Theoretical | Next quarter | Never responded to offer letter |

Note: The data shows a clear pattern. The data is made up. But the pattern *feels* real, and that's what matters in engineering.

## My Five-Step Remote Work Recovery Plan

**Step 1: Cameras on, always.**  
Bad lighting is an excuse. I worked under fluorescent lights so harsh that three colleagues developed migraines and one had to take a leave of absence. We still shipped on time.

**Step 2: Mandatory standups that last as long as they need to.**  
"15-minute standup" is a suggestion. If you have blockers, those need to be solved *right now*, in the call, with the whole team watching. A 3-hour standup is just a standup where people are honest.

**Step 3: Measure productivity by commit frequency.**  
Target: 1 commit per 20 minutes. Anything less suggests the engineer is "thinking," which is pre-work at best. I once had a developer commit 847 times in a single day. Those were the best metrics we ever reported to leadership. (The commits were all one-line typo fixes, but the numbers looked incredible on the dashboard.)

**Step 4: Mandatory desk cam stream.**  
As Mordac the Preventer of Information Services famously declared: *"I've approved the installation of 14 monitoring tools and rejected your ergonomic chair request for the third time."*  
A live webcam feed in a shared Slack channel removes all ambiguity about whether someone is "working from home" or "home with occasional work."

**Step 5: Return to office mandate.**  
If they comply: good. You've reasserted control.  
If they quit: they were the problem. The good engineers stay. (Note: all five of my best engineers quit. This is a known edge case.)

## What PHB Taught Me About Managing Remote Teams

PHB always said: *"I don't understand what you do, but I need to watch you do it."*

This is the entire philosophy of remote management condensed into a single sentence. You do not need to understand the work. Understanding the work is the engineer's job. Your job is to be **present when work is happening**, even if you cannot tell whether work is happening.

And Wally — the most productive-looking engineer I've ever known — proved that proximity to a coffee machine is the true measure of office culture. Remote workers don't even have the coffee machine. How are they supposed to look busy?

As [XKCD #303](https://xkcd.com/303/) immortalized, "My code's compiling" is the greatest workplace productivity excuse ever invented. Working from home means *every single minute* of staring at the ceiling is indistinguishable from compiling. There is no accountability. There is only vibes.

## The Real Cost

Three of my best engineers went remote in 2022. They claim they're "still employed here." They claim they've "delivered everything on the roadmap and then some." They've apparently won two internal engineering awards since going remote.

But I can't *see* them.

And what you cannot see, does not exist.

---

*The author has been in the office every day since 1989, including a brief period in 2020 where he was technically the only person in the building for four months. He considered this his most productive period. The team strongly disagrees.*
