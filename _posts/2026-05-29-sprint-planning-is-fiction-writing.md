---
layout: post
ref: sprint-planning-is-fiction-writing
title: "Sprint Planning Is Just Fan Fiction for Adults"
date: 2026-05-29 00:00:00 -0300
categories: [agile, planning, management]
tags: [agile, sprint, planning, estimates, scrum, fiction, meetings]
---

I have sat through 2,300 sprint planning sessions. I counted. I had time because nothing in those sessions ever happened the way anyone planned.

Sprint planning is fiction writing. It always has been. The difference between a romance novel and a sprint plan is that the romance novel is honest about being invented, and someone occasionally finishes it.

## The Anatomy of Sprint Planning Fiction

Every sprint planning session follows the same narrative arc:

**Act 1 – The Setup:** Someone pulls up the backlog. The backlog contains 400 tickets written in six different styles because four people have quit since the oldest one was created. Nobody knows what PLAT-2847 means. The original author is now working at a competitor.

**Act 2 – The Conflict:** The team begins estimating. The senior engineer says "3 points." The product manager hears "3 days." The CEO, somehow in this meeting, hears "done by Tuesday." These are three different things. Nobody corrects anyone. The ticket gets a 3.

**Act 3 – The Resolution:** Thirty-two tickets are added to the sprint. The sprint capacity is technically enough for fourteen of them. The team claps. The board is full. Everyone goes back to their desks and works on the thing that's actually on fire, which was not in the sprint at all.

This is the cycle of life. XKCD [#1425](https://xkcd.com/1425/) captured the spirit of estimation perfectly: we confidently assign complexity to tasks we don't understand, and we're surprised when we're wrong.

## Story Points: A Love Story

Story points were invented to stop people from estimating in time, which led to unrealistic time commitments. They were replaced with story points, which led to unrealistic point commitments, which were then converted back to time by management anyway.

The circle is complete. We went through an entire methodology revolution to end up exactly where we started, except now we argue about whether "5 points" means "harder than 3 points" or "kind of like 3 points but with a dependency."

| Story Points | What It Means | What It Really Means |
|---|---|---|
| 1 | Trivial change | You will discover the database schema during this |
| 2 | Small task | Two teams share ownership, neither will respond |
| 3 | Medium complexity | This requirement changes on Wednesday |
| 5 | Complex | This is actually three tickets that someone merged |
| 8 | Very complex | Nobody has touched this code in 4 years |
| 13 | Epic-scale | This is the entire rewrite nobody approved |
| ∞ | "Let's spike it" | We have no idea and we never will |

## The Sprint Goal: Modern Haiku

Every sprint must have a "sprint goal." A sprint goal is a haiku written by committee, optimized to mean everything and commit to nothing.

Real sprint goals I have witnessed:

> *"Improve platform stability while delivering user-facing value"*

Translation: We will fix some bugs, maybe. Or not. We reserve the right to interpret "stability" and "value" at end-of-sprint retrospectively.

> *"Move the needle on engagement metrics"*

Translation: We don't know what we're building. The needle is metaphorical. Nobody defined "engagement."

> *"Enable the team to execute against Q3 priorities"*

Translation: This sprint has no goal. We have only motion, mistaken for progress.

> "The PHB said our velocity needs to improve by 20% this quarter."
> "Did he say improve the *work*, or improve the *numbers*?"
> "Same thing."
> — *Dilbert*, on why velocity metrics exist to be gamed

## Velocity: The Metric That Measures Nothing

Velocity is the number of story points completed per sprint. It is used to forecast future sprints. It would work great if:

1. Story points were consistent between sprints
2. Story points were consistent between team members
3. Story points reflected actual complexity
4. The team composition never changed
5. Requirements never changed mid-sprint
6. Nobody ever added "quick wins" directly to the sprint board
7. Production never caught fire

In 47 years, I have never seen a sprint where all seven of those were true simultaneously. Velocity is measuring the output of a chaotic system with a ruler that shrinks and expands based on team morale.

But we put it on a chart. The chart goes in the slide deck. The slide deck goes to leadership. Leadership says "velocity is trending down" and schedules a meeting about it, reducing velocity further.

```
sprint_velocity = (
    actual_work_done
    / story_point_inflation_factor
    / scope_creep_multiplier
    * mood_of_the_team_on_monday
    - interruptions_from_management
    + lies
)

# Historical average: 34 points
# Useful for forecasting: No
```

## What Sprint Planning Actually Optimizes For

Not delivery. Not quality. Not predictability. Sprint planning optimizes for the feeling of control.

The board is full. The tickets are estimated. The sprint goal is written (vaguely). The capacity has been calculated (incorrectly). Everyone nods. Everyone *feels* like a plan exists.

This feeling lasts until approximately 10 AM on Monday when someone says "hey, we have a production issue."

[XKCD #303](https://xkcd.com/303/) — Compiling. It's not about compiling. It's about all the ways work stops being the thing we planned and becomes the thing we're actually dealing with.

## My Alternative: The Chaos Sprint

After 47 years of watching fiction masquerade as planning, I propose the Chaos Sprint:

**Rules:**
1. No estimates. Write tickets in plain English. "Fix login." "Add button." "Why is production down."
2. No sprint goal. The sprint goal is: survive.
3. No velocity tracking. Instead, count "things that are done." A thing is done when no one mentions it in stand-up anymore.
4. The sprint ends when it ends.
5. Retrospectives are replaced with a shared meal and the tacit acknowledgment that we are all doing our best.

This is not a joke. This is how every successful project I have ever shipped actually operated. We called it something else at the time — "crunch," "just getting it done," "the week before the conference" — but the methodology was pure Chaos Sprint.

## The One Thing Sprint Planning Gets Right

I will give credit where it is due: sprint planning does one thing well.

It gets the whole team in a room.

Or a video call. Whatever.

For ninety minutes, everyone has seen the same backlog, heard the same confusing requirements, and experienced the same collective confusion. When the sprint collapses — and it will — at least everyone has shared context about why.

This is the true value of sprint planning. Not the plan. The shared fiction. The common hallucination that briefly makes a team feel coordinated.

We are all just authors writing the same story, taking turns pretending we know how it ends.

---

*The author has never completed a sprint as planned. He has shipped many features. He does not see these as contradictory statements.*
