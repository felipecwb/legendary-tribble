---
layout: post
ref: standups-are-public-humiliation
title: "The Daily Standup Is Just Public Humiliation With a Timer"
date: 2026-05-04 00:00:00 -0300
categories: [agile, meetings, career]
tags: [standup, scrum, agile, meetings, humiliation, productivity, scrum-master]
---

Every morning at 9:47 AM (because the Scrum Master hits snooze twice), the team gathers in a circle — physically or via a video call where 40% of people have their cameras off — to recite their three sacred vows:

1. **What I did yesterday** (mostly Slack, some Stack Overflow, one commit)
2. **What I'll do today** (same as yesterday, but with more confidence)
3. **Blockers** (which no one will actually help with)

This ritual is called the **Daily Standup**. It is the purest distillation of corporate theater that software engineering has ever produced.

I've been doing standups for 23 years. I have never once been unblocked by a standup.

## The Invention of the Standup

Legend has it the standup was invented to be *short*. "If people are standing, they'll want to sit down quickly," said the consultant who sold this idea for $800/hour.

Reader: they are not standing. They are slumping in ergonomic chairs, half-watching a Zoom tile of someone's bedroom ceiling, muted because their cat knocked over their microphone stand.

The "15-minute timebox" is fiction. Every standup expands to fill available suffering.

> *"Wally, what are your blockers?"*
> *"Yes."*
> — Dilbert, basically every strip from 2001-2023

## The Three Archetypes You Meet Every Day

### The Overachiever

```
"Yesterday I refactored the authentication module, wrote 12 unit tests,
reviewed 4 PRs, updated the wiki, called the client, fixed the CI pipeline,
and learned Rust. Today I'll—"
```

This person makes everyone else feel like they haven't existed for the past 24 hours. They are despised. They will be promoted. It is the same thing.

### The Philosopher

```
"So, yesterday... I was thinking about the architecture we discussed back in
Q3, and I realized — actually, can we open a Miro board? I want to draw
something. It'll only take a second."
```

It takes 47 minutes. The Scrum Master has a small breakdown.

### The Ghost

```
[STATUS: Away]
[STATUS: In a meeting]
[STATUS: Do not disturb]
```

This person is either doing actual work or they've already quit and no one has noticed. Both are valid. Both are aspirational.

## The Blocker That No One Unblocks

Here is the dirty secret of the standup: **blockers are mentioned but never resolved.**

The standup is not a problem-solving session. It is a *problem-announcing* session. You say your blocker. People nod. The Scrum Master types it into Jira where it will age like fine wine — quietly, in the dark, becoming more complex and vinegary over time.

```
BLOCKER LIFECYCLE:

Day 1:  "I'm blocked on the API spec from design."
Day 3:  "Still blocked."
Day 7:  "Still blocked, but I worked around it."
Day 14: "What blocker? I fixed it myself."
Day 30: "It became a feature. We ship next week."
```

You never needed the standup. You needed 15 minutes of uninterrupted focus.

See also: [XKCD #303 — Compiling](https://xkcd.com/303/) *(the real reason standups exist)*

## The Standup Report You Actually Want to Give

Here's what people want to say versus what they say:

| What You Want to Say | What You Actually Say |
|---|---|
| "I spent 6 hours in a Kafka rabbit hole and have nothing to show for it" | "Working on the event pipeline. Making good progress." |
| "I have no idea what I'm doing" | "Investigating some edge cases." |
| "My blocker is that this project is fundamentally broken" | "Minor dependency issue. Should be resolved today." |
| "I played Factorio for 4 hours at 2am and I'm exhausted" | "Good to go!" |
| "What is any of this for" | *[joins Zoom, sits in silence]* |

## What Standups Actually Measure

The standup does not measure productivity. It measures **performance of productivity** — and these are different things.

A developer who writes 3 lines of perfectly placed code that eliminates a race condition that's been costing the company $40,000/month in downtime has nothing to say at standup.

A developer who opens 11 tickets, attends 7 meetings, updates 3 wiki pages, and replies to 400 Slack messages has a *great* standup.

Guess which one gets the performance review bonus.

```python
def productivity_score(actual_output, standup_performance):
    # Classic enterprise calculation
    return standup_performance * 0.8 + actual_output * 0.0 + vibes * 0.2
```

## The Optimal Standup Strategy

After 47 years of surviving standups, I've developed an optimal strategy:

**Step 1:** Always go third. First is too eager. Second is still catching up. Third is perfect — the Scrum Master has stopped paying attention.

**Step 2:** Speak for exactly 42 seconds. Long enough to seem substantive. Short enough to avoid follow-up questions.

**Step 3:** Name-drop a ticket number. People respect ticket numbers. It implies structure. It implies you know what you're doing. It implies accountability. None of this is necessarily true.

**Step 4:** End with "No blockers." Even if you have blockers. Because if you mention a blocker, you'll be asked to "take it offline" immediately after the standup, which means scheduling *another meeting* to not solve your blocker in a different room.

**Step 5:** Disappear from Slack for 90 minutes after standup. This is when real work happens. The standup has exhausted everyone's social energy. Enjoy the silence.

## The Async Standup Heresy

Some teams have tried **async standups** — writing your update in a Slack thread or bot. This is treated by Scrum Masters the way medieval peasants treated eclipses: as an omen of doom.

"But how will we *connect* as a team?" they ask.

We won't. We're software engineers. We connect through pull request comments and passive-aggressive git commit messages. That's enough. That's always been enough.

> *"I'm not anti-social. I'm anti-pointless-meetings."*
> — Me, in my head, every 9:47 AM

## A Note on Standing Up

I've worked at companies that actually enforce the "standing" part. They believe discomfort accelerates brevity.

It doesn't. It just accelerates back pain.

One engineer at a former job installed a standing desk in the meeting room and gave 20-minute standing updates every day. He was promoted to Principal Engineer.

His spine, however, was demoted.

## Conclusion: What Standups Are Good For

Standups are excellent for:
- ✅ Finding out who hasn't pushed code in 3 days
- ✅ Reminding yourself what project you're on
- ✅ Making the Scrum Master feel useful
- ✅ Providing 15 minutes of structured procrastination
- ✅ The ritual comfort of human routine in an otherwise chaotic codebase

Standups are not good for:
- ❌ Solving blockers
- ❌ Coordination
- ❌ Engineering
- ❌ Anything in the Scrum guide

But we'll keep doing them. Because stopping standups would require a meeting to discuss stopping standups, and that meeting would need a standup to coordinate, and then we'd never escape.

See also: [XKCD #1658 — Estimating Time](https://xkcd.com/1658/) *(how long will standup take today?)*

---

*The author has been "blocked" since 2017. He mentions it every morning at 9:47 AM. No one has ever helped.*
