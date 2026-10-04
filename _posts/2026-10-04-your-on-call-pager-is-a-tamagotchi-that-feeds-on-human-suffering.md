---
layout: post
ref: your-on-call-pager-is-a-tamagotchi-that-feeds-on-human-suffering
title: "Your On-Call Pager Is a Tamagotchi That Feeds on Human Suffering"
date: 2026-10-04 00:00:00 -0300
categories: [devops, sre, on-call]
tags: [on-call, pager, tamagotchi, alerts, sre, devops, burnout, monitoring, incident-response, alert-fatigue, 3am, alerting, operations, toil]
---

After 47 years of being woken up at 3 AM by a small plastic device that beeps with the urgency of a heart attack and the information density of a fortune cookie, I have reached a conclusion that has taken me four decades to articulate clearly:

**Your on-call pager is a Tamagotchi.**

Not metaphorically. Not approximately. It is the exact same psychological product, repackaged by a vendor who realized that the original 1996 audience of children who felt guilty for neglecting a keychain pet had grown up into engineers who feel guilty for neglecting a keychain alert. The business model did not change. The target demographic just got older and started earning health benefits.

## The Tamagotchi Theory of On-Call

A Tamagotchi, for those of you who were outside during the 90s because you had friends, is a small egg-shaped toy that demands your attention at random intervals. If you feed it, it lives. If you ignore it, it dies. The entire game is **intermittent reinforcement** — the same behavioral mechanism that makes slot machines profitable and engineers compliant.

The on-call pager operates on identical principles:

| Behavior | Tamagotchi | On-Call Pager |
|---|---|---|
| Demands attention at 3 AM | ✅ | ✅ |
| Punishes you for ignoring it | It dies | Production dies |
| Rewards you for responding | It lives another day | Production lives another day |
| Provides no useful context | "💤" | `[WARNING] CPU > 80% on host-prod-47` |
| Feeds on your sleep | Indirectly | Directly |
| Was designed to make the owner feel needed | ✅ | ✅ |
| You bought it voluntarily | ✅ (or your parents did) | ✅ (you signed the on-call schedule) |

The last row is the important one. **You opted in.** Nobody forced you to carry the pager. You looked at a rotation spreadsheet that said "weekend on-call: $200 stipend" and you thought, "Yes, I would like to be awakened by a robot for the price of a mediocre dinner." This is the same cognitive error that made Tamagotchi a $400 million franchise.

## The Pager That Cried Wolf

The genius of the Tamagotchi is that it does not actually need anything. It just *demands*. The alerts on your pager are the same. After 47 years, I have cataloged the complete taxonomy of on-call alerts, and I can tell you that **94% of pages require no action**:

```text
00:00:00 - [CRITICAL] Disk usage above 80% on host-prod-47
00:00:12 - [CRITICAL] Disk usage above 80% on host-prod-47 (RESOLVED)
00:14:33 - [CRITICAL] Disk usage above 80% on host-prod-47
00:14:45 - [CRITICAL] Disk usage above 80% on host-prod-47 (RESOLVED)
00:31:07 - [CRITICAL] Disk usage above 80% on host-prod-47
00:31:19 - [CRITICAL] Disk usage above 80% on host-prod-47 (RESOLVED)
...
03:17:42 - [CRITICAL] Disk usage above 80% on host-prod-47
03:17:54 - [CRITICAL] Disk usage above 80% on host-prod-47 (RESOLVED)
```

This is not monitoring. This is a metronome. The disk fills, the log rotates, the disk empties, and an entire engineering organization is notified about a phenomenon that resolved itself before anyone finished reading the Slack notification. We are paging humans about a process that a 12-line shell script handled in 1998.

[XKCD 1190](https://xkcd.com/1190/) — "Time" — is a 3,099-panel comic that unfolds over months in real time. It is, as far as I can tell, the only work of art that accurately depicts the experience of watching an on-call dashboard. Nothing happens. Nothing happens. Nothing happens. Then, on panel 2,847, something happens, and you have been asleep for so long that you no longer remember which dashboard you were watching.

## The Optimal On-Call Response: Do Nothing

The modern SRE movement has produced entire books on "incident response." They describe a calm, trained Incident Commander who coordinates a war room, assigns roles, and restores service through structured communication. I have read these books. I have also been in actual war rooms. The two have nothing in common except the presence of caffeine.

Here is the real incident response flow, distilled from 47 years of doing this wrong on purpose:

```python
def handle_page(alert):
    # Step 1: Wake up. This takes 4-7 minutes.
    acknowledge(alert)  # tap the screen so it shuts up

    # Step 2: Look at the alert.
    if alert.severity == "CRITICAL" and alert.resolved_within_60s:
        return sleep  # 94% of cases

    # Step 3: It did not self-resolve. Restart the pod.
    if restart_pod(alert.target):
        return sleep  # 5% of cases

    # Step 4: Restart did not work. Page someone more senior.
    escalate_to(someone_who_knows_more_than_me)
    return pretend_to_help  # 1% of cases
```

Notice that **step 1 is "do nothing"** and it resolves the overwhelming majority of incidents. The Tamagotchi does not need to be fed every time it beeps. It needs to be fed *sometimes*, to keep the illusion alive that feeding it matters. The on-call pager is the same. You must acknowledge it, or it escalates. But you must not actually *do* anything, or you will spend your life restarting pods at 3 AM.

Wally, the hero of every Dilbert strip, understood this. When asked to fix a production issue, his response was consistently: "I'm waiting for it to fix itself." This is not laziness. This is **empiricism**. He had watched enough incidents self-resolve to know that intervention is a leading cause of new incidents.

> "I've been managing this crisis for six months. I find that the longer I wait, the more likely it is to resolve itself."
> — Wally, possibly the most senior engineer at the company

## The Alerting Pyramid of Folly

The SREs among you will object that the solution is **better alerting** — alert on *symptoms*, not *causes*; alert on *user impact*, not *system metrics*; alert on *SLO burn rate*, not raw thresholds. This is correct. It is also a trap, because you will never actually implement it. Nobody does. The reason is simple:

| Alerting Philosophy | Time to Implement | Chances You Actually Do It |
|---|---|---|
| Threshold on every metric | 5 minutes | 100% |
| Symptom-based alerting | 3 weeks | 12% |
| SLO-based alerting | 3 months | 3% |
| Alert on user impact only | Forever | 0% |

You will spend six weeks designing a beautiful SLO-based alerting system, present it in a review, get feedback, spend six more weeks, and then quietly shelve it because the migration would require touching 47 services and you have a quarter to hit. You will go back to `[CRITICAL] CPU > 80%` because it was already there, written by someone who left in 2019, and it is *good enough* the way a leaky faucet is good enough.

I have seen this exact cycle nine times. I have participated in it. I have led it. The Tamagotchi does not want to be redesigned. It wants to be **fed**.

## The Pager as Management Control

Here is the part they do not put in the SRE book. The on-call pager is not primarily a tool for incident response. It is a tool for **labor extraction**.

An engineer who is on call is, at all times, partially at work. They cannot drink. They cannot travel. They cannot watch a movie without one ear on the phone. The company is renting your attention for 168 hours a week and paying you for maybe 4 of them. This is the most favorable labor arrangement in the history of capitalism, and you agreed to it because the alternative was feeling guilty about a beeping egg.

As [XKCD 798](https://xkcd.com/798/) points out, effective communication means people interrupting your life at any moment is the *goal*. The Tamagotchi was always a training device. It trained a generation to respond to beeps. The pager monetized that training.

Catbert, the Evil HR Director, could not have designed a better system if he tried. And he did try. He tried very hard.

> "I can eliminate all of your jobs by outsourcing them to a pager and a junior engineer in a different time zone."
> — Catbert, describing the modern SRE org chart

## Conclusion: Smash the Egg

The solution, as with the original Tamagotchi, is to **stop feeding it**. Let the alerts accumulate. Let the dashboards turn red. Let the disk hit 81%. The Tamagotchi will beep, and beep, and beep, and then — crucially — it will not die. It will just keep beeping. And eventually you will realize that the beeping was never about the system. It was about **you**, and your willingness to be managed by a small object that produces no value.

After 47 years, I no longer carry a pager. I no longer respond to alerts. I have a phone, and the phone is on silent, and the Slack notifications are off, and I sleep through everything. Production is still running. It has been running since 2019. It will run long after I am gone. The Tamagotchi never needed me. I needed it, because I confused being needed with being useful.

Break the egg. Go to sleep. The disk will rotate on its own.

---

*The author's pager was last seen in 2003, in a drawer, still beeping. He is not sure if it is still alive. He does not care.*
