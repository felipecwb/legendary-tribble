---
layout: post
ref: sprint-velocity-is-made-up-anyway
title: "Sprint Velocity Is Made-Up Anyway, So Let's Make Up Better Numbers"
date: 2026-04-18 00:00:00 -0300
categories: [career, agile]
tags: [agile, scrum, sprint, estimation, velocity, story-points, planning-poker, terrible-advice]
---

I've survived 47 years in this industry. I've outlasted Waterfall, survived the Agile revolution, and watched the same problems get rebranded four times with different colored sticky notes. And in all that time, I've come to one unshakeable conclusion about sprint velocity:

**It is made-up numbers measuring imaginary progress toward a fictional deadline.**

The good news? If we're making things up anyway, we should at least make up *better* things.

## What Is Velocity, Really?

Velocity is defined as "the amount of work a team completes in a sprint," measured in "story points," which are defined as "not hours, definitely not hours, please don't ask how many hours."

The PHB asked me once: "If story points aren't hours, what are they?" I told him they were "a relative measure of effort, complexity, and uncertainty." He nodded sagely. He then opened a spreadsheet and multiplied the story points by 6 to get an hour estimate.

This is the nature of story points. They exist to be converted to hours the moment management leaves the planning meeting.

[XKCD #303](https://xkcd.com/303/) is about developers doing sword fights while waiting for code to compile. Sprint planning is what happens when you try to estimate how long the compile will take and then have a meeting about the estimate.

## The Story Point Scale: A Taxonomy of Lies

The Fibonacci sequence was discovered in the 13th century by Leonardo of Pisa while studying rabbit populations. It was adopted by Agile practitioners in the 21st century because "1, 2, 3, 5, 8, 13" *feels* like it captures increasing uncertainty.

Here's what it actually captures:

| Points | Official Meaning | Actual Meaning |
|--------|-----------------|----------------|
| 1 | Trivially simple | Still somehow takes 3 days |
| 2 | Simple | Will require an architecture meeting |
| 3 | Moderate complexity | Junior touched the wrong file |
| 5 | Complex | Will block on a dependency for two weeks |
| 8 | Very complex | Should be its own epic (will not become its own epic) |
| 13 | Extremely complex | Just say no |
| 21 | Should be broken down | Will not be broken down |
| ∞ | Wally's specialty | "Still in progress" since 2021 |

The beautiful thing about this system is that 1 point and 21 points are on the same scale, with the same precision, estimated by the same people who once took 5 days to fix a CSS border.

## Planning Poker: Gambling With Company Time

Planning poker is the ceremony where everyone reveals their estimate simultaneously to "avoid anchoring bias." The theory is that if everyone shows their card at the same time, no one is influenced by the most senior engineer's number.

Here is what actually happens:

```
1. Cards distributed. Everyone looks at the ticket.
2. "Everyone ready? 3... 2... 1... show!"
3. Cards revealed: 3, 3, 2, 13, 3
4. Everyone looks at the person who said 13.
5. "Walk us through your thinking?"
6. "Well, this touches the payment service, and we'd need to—"
7. Everyone nods.
8. Estimate becomes 8.
9. Everyone looks at the person who said 2.
10. "I thought it was simpler."
11. Estimate stays at 8.
12. Ticket takes 11 days.
13. "Our velocity was 42 this sprint."
```

The "simultaneous reveal" prevents the junior developer from being influenced. It does not prevent the junior developer from then immediately updating their estimate based on what the senior developer said. This is called "calibration." It is called "calibration" instead of "hierarchy" for morale reasons.

Dogbert, when asked to facilitate a planning session, once held up a card that said "∞" and said: *"All software estimates are infinite. We're just negotiating how much of infinity we're pretending to scope."* He was right. He was fired from the facilitation role.

## The Velocity Trap: A Cautionary Tale

Sprint 1: Team estimates 40 points, completes 32. Velocity: 32.
Sprint 2: Team estimates 35 points, completes 30. Velocity: 30.
Sprint 3: Team estimates 32 points, completes 31. Velocity: 31.

Average velocity: 31. The team now knows: they can reliably deliver 31 points per sprint.

This information is immediately used for the following:

1. **Manager takes velocity to stakeholders:** "We have a stable velocity of 31 points."
2. **Stakeholder says:** "Great. The feature needs 248 points. So... 8 sprints?"
3. **Manager commits to 6 sprints.** "We'll optimize."
4. **Sprint 4:** Team is asked to do 42 points to "make up time."
5. **Velocity collapses to 22** because people are stressed and cutting corners.
6. **Manager:** "What happened to your velocity?"
7. **Team:** "We're working on it."
8. **Manager:** "Work harder."

Velocity, which was invented to help teams set *sustainable pace*, has been repurposed as a stick to hit the team with for not going faster. Wally saw this coming in 2008 and has maintained a velocity of exactly 12 points per sprint ever since. Not a point more. He calls it "protecting the baseline."

## Better Estimation Techniques (That Are Just As Reliable)

Since story points are made up, allow me to suggest alternatives with the same accuracy and far less overhead:

**Option 1: The Dice Method**
Roll a d6. That's the estimate. For complex features, roll twice and add. This matches historical accuracy for most planning poker sessions, requires no meeting, and adds genuine excitement to backlog refinement.

**Option 2: Wally Units (WU)**
1 WU = the time it takes Wally to complete one coffee-fueled avoidance session (approximately 2.5 days). All tickets estimated in WU. Engineers immediately understand the scale. Management never learns what a Wally Unit is.

**Option 3: T-Shirt Sizing (But Honest)**
- XS: Still gonna take a week
- S: Two weeks
- M: Two weeks
- L: Two weeks
- XL: Two weeks and a postmortem

Research shows this scale is indistinguishable in accuracy from Fibonacci story points but saves 45 minutes of planning poker per sprint.

**Option 4: Just Say "Two Weeks"**
This is the classic. The PHB asks "how long?" You say "two weeks." He accepts this. The work takes two weeks. Or six. The meeting ends either way.

## The Velocity Dashboard: A Monument to False Precision

Most Scrum tools will show you a beautiful velocity chart: bars going up and down, a trend line, an average. It looks like real data. It has decimal points.

```
Sprint 14 Velocity: 34.7 points
Sprint 15 Velocity: 28.2 points
Sprint 16 Velocity: 41.1 points
Average: 34.67 points/sprint
Projected completion: Sprint 23 (±4 sprints)
```

This chart was generated by people guessing numbers and then having those numbers not match reality, measured against an average of previous guesses that also didn't match reality. The ±4 sprint range represents approximately 8 weeks, which is long enough for the entire business direction to change anyway.

Displaying this data in a dashboard does not make it real data. It makes it well-formatted fiction.

## The Only Metric That Actually Matters

After decades of tracking velocity, burndown charts, cycle time, and throughput, I've found one metric that actually predicts software delivery:

**Is the feature done? Yes or No?**

That's it. Binary. No points. No velocity. No burndown. Done or not done. Ship it or don't ship it.

Every other metric is a proxy for this one, and all the proxies are wrong.

## Practical Tips for Surviving Sprint Planning

1. **Estimate everything as 3 or 5.** This covers 80% of all tickets. Outliers will be noticed and discussed regardless.

2. **Never say a ticket is done until it's in production.** "Done done" is a technical term for "done" in a way that doesn't come back to haunt you.

3. **Keep a personal buffer.** If your team has 40 points of capacity, estimate 48. The extra 8 points are for "tech debt," "blockers," and "that one thing from last sprint that was 'done' but wasn't."

4. **When asked for estimates in hours**, say "I can give you story points or hours, but not both. Choose." This creates enough confusion to end the meeting.

5. **The correct response to "can you commit to this date?"** is "I can commit to starting this date." This is technically true and requires no further discussion.

## Conclusion

Sprint velocity is a number generated by people guessing, measured by people hoping, and reported to people who will misuse it. It has the predictive power of astrology and the precision of a weather forecast from 1987.

That's fine. Embrace it. The goal of sprint planning is not to accurately predict the future — it's to create a shared fiction that allows a team to coordinate for the next two weeks without collapsing into chaos.

Just make sure your fiction is internally consistent and that Wally's stories never exceed 12 points.

---

*The author has been "two weeks away" from finishing the same feature since Q3 2022. The velocity chart shows steady progress. Both things are true.*
