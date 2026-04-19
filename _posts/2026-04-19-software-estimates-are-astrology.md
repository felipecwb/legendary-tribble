---
layout: post
ref: software-estimates-are-astrology
title: "Why Software Estimates Are Just Astrology With Jira Tickets"
date: 2026-04-19 00:00:00 -0300
categories: [career, agile, methodology]
tags: [estimates, planning, agile, scrum, story-points, jira, velocity, career]
---

> *"It'll take two weeks."*  
> — Every developer, on every task, for the last 40 years

I've been estimating software projects since before Jira existed. Back then, we wrote estimates on Post-it notes, which got lost under coffee cups, which meant nobody could hold us accountable.

Those were *better* times.

Now we have story points, planning poker, velocity charts, sprint burndown reports, and roadmap confidence intervals — all of which generate precisely the same amount of predictability as a Magic 8-Ball, but with *considerably* more ceremony and a $40/seat/month subscription.

## Story Points Are Made-Up Numbers

Let me explain story points to someone who hasn't suffered through them yet:

1. You take a task of completely unknown complexity
2. You assign it a Fibonacci number: 1, 2, 3, 5, 8, 13, 21, or "∞ (needs refinement)"
3. You argue about whether it's a 5 or an 8 for 25 minutes
4. You compromise on 8
5. You add up all the numbers across the sprint
6. You miss the sprint by exactly 43%
7. You call this **velocity**
8. You make a chart about it

[XKCD #1425](https://xkcd.com/1425/) perfectly illustrates why software estimation is fundamentally broken: the same task — "recognize a bird in a photo" — takes either 10 minutes or the rest of your career depending on whether you understand the problem. There is no way to know which until you are already 4 hours into the Jira ticket.

The PHB (Pointy-Haired Boss) once asked my team for an estimate on a database migration. We said "two weeks." He said "great, make it one." We delivered in three months. This process is called **Agile**.

## The True Estimation Formula

After 47 years, I've derived the only honest estimation formula in software engineering:

```
T_actual = T_estimate × π × (unknown_unknowns + lies_from_dependencies + 1)
```

Where:
- `T_estimate` = whatever number you said in sprint planning
- `π` = the universal constant of programmer optimism
- `unknown_unknowns` = always at least 3, regardless of ticket complexity  
- `lies_from_dependencies` = number of external teams who said "that API is ready"

The Fibonacci-to-Calendar translation table every senior engineer carries in their head:

| Story Points | What You Said | What You Actually Meant |
|-------------|--------------|------------------------|
| 1 | "1 hour" | "I've never actually done this before" |
| 2 | "Half a day" | "I've done this before, badly" |
| 3 | "A full day" | "This touches the legacy code" |
| 5 | "2–3 days" | "This *is* the legacy code" |
| 8 | "1 week" | "I'll need to *understand* the legacy code first" |
| 13 | "2 weeks" | "The legacy code must be completely rewritten" |
| 21 | "1 sprint" | "Please help me, the legacy code is sentient" |
| ∞ | "Needs refinement" | "I am the legacy code" |

## Planning Poker: Democracy for Bad Decisions

Planning poker democratizes estimation by allowing the entire team to be wrong *together*. Benefits include:

1. **Consensus** — Everyone agrees on a number nobody believes
2. **Engagement** — Developers pretend to think for 30 seconds instead of blurting out "3"
3. **Accountability diffusion** — When you miss the sprint, it was the *team's* estimate, not yours
4. **False precision** — Fibonacci numbers *feel* more scientific than "probably Thursday, maybe Friday"
5. **Team bonding** — Nothing bonds a team like collectively underestimating a migration task

Wally from the engineering floor never reveals his planning poker card. When asked why, he says: *"My velocity is 1 story point per quarter. I've maintained it for 11 years. Do not tamper with consistency."*

He still has a job. He was given a budget for a new laptop last year.

## T-Shirt Sizing: Estimation for People Who've Already Given Up

Some organizations, tired of arguing about whether something is a 5 or an 8, switched to T-shirt sizes (XS, S, M, L, XL, XXL). This is considered **progress**:

- **XS:** "This is a 1-line change." It is never a 1-line change.
- **S:** Should take 30 minutes, will take 2 days, 1 of which is waiting for PR review.
- **M:** Should take a day, will consume a full sprint.
- **L:** Should be broken into smaller stories. Will never be broken into smaller stories. Will ship as-is in Q3.
- **XL:** This is an Epic, not a Story. Will be closed as "Won't Fix" in 18 months.
- **XXL:** This requires an architecture meeting. The architecture meeting will generate 3 new Epics, none of which will have estimates.

## The Hawthorne Effect of Estimates

Here's the dirty secret nobody tells you at your first sprint planning: **the act of estimating changes the estimate**.

The moment you type "2 weeks" into Jira, your manager screenshots it and emails the client. The client builds it into a demo scheduled for next Tuesday. Your PM tells a partner it'll be ready by end of month. The VP announces it in an all-hands. By the time the ticket reaches your board, your casual *estimate* has become:

- A contractual commitment
- A milestone in a roadmap deck
- The subject of a board presentation
- The reason you are now eating lunch at your desk

This is called **the Fibonacci of Consequences**.

As Catbert, the Evil HR Director, noted in my last performance review: *"Your estimates were accurate to within ±280%. That's a 15% improvement over 2025. We're trending in the right direction."*

## What Velocity Actually Measures

Your sprint velocity is not a measure of productivity. It is a measure of **how fast your team collectively agrees to be wrong**.

High velocity teams:
- Estimate optimistically
- Finish fast (because the tasks were small and well-defined)
- Are praised by management
- Are immediately given larger, worse-defined tasks
- Their velocity collapses
- They are told to "improve their processes"

Low velocity teams:
- Estimate conservatively
- Deliver reliably
- Are told they need to "increase throughput"
- Are pressured to estimate smaller
- Their velocity inflates artificially
- Management is briefly happy
- The cycle repeats

This is a closed system. There is no escape. [XKCD #1205](https://xkcd.com/1205/) tells you exactly how much time is worth spending on improving a recurring process. The same logic applies to estimation: if you spend more than 10 minutes estimating a single ticket, you've already exceeded your margin of error and should simply pick a number between 3 and 8.

## The Only True Estimate

After 47 years, teams, projects, and postmortems, I have arrived at the only universally accurate software estimate:

> **"It's done when it's done."**

This is not a cop-out. This is thermodynamics. You cannot compress the time required to write, test, review, deploy, and debug software without losing information — and the information you lose is always "correctness."

Accept this. Write "∞" in the story point field. Let the ticket age gracefully. Technical debt is just future estimates you don't have to make yet.

---

*The author's flagship project was estimated at 3 story points in 2021. It is currently in sprint 47. The burndown chart is a horizontal line. The team calls this "sustainable pace."*
