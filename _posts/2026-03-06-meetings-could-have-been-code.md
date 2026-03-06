---
layout: post
ref: meetings-could-have-been-code
title: "This Meeting Could Have Been Code: Stop Talking, Start Shipping"
date: 2026-03-06 08:10:00 -0300
categories: [career, meetings]
tags: [meetings, productivity, communication, shipping, standups]
---

After 47 years of mass-producing bugs, I've realized that **most meetings could have been code**. Stop talking about what you're going to build and just build it.

## The Meeting Economy

The average developer spends 35% of their time in meetings:

| Meeting Type | Duration | Frequency | Actual Value |
|--------------|----------|-----------|--------------|
| Daily standup | 15 min | Daily | Status: unchanged |
| Sprint planning | 2 hours | Bi-weekly | Move cards around |
| Retro | 1 hour | Bi-weekly | Complain, forget, repeat |
| Architecture review | 3 hours | Weekly | Argue about boxes |
| 1:1 | 30 min | Weekly | "How's it going?" "Fine" |
| All hands | 1 hour | Monthly | CEO says "synergy" |

That's 47 hours per month of not coding. I could have shipped 3 features in that time.

## The Standup Ritual

Every standup, ever:

```
Developer 1: "Yesterday I worked on the thing. 
              Today I'll continue the thing. 
              No blockers."

Developer 2: "Same. The thing."

Developer 3: "Also the thing."

Scrum Master: "Great standup team! Same time tomorrow!"
```

This could have been a Slack message. Or nothing.

[XKCD 303](https://xkcd.com/303/) shows developers escaping meetings via "my code's compiling."

## Meeting Math

Let's do the math on a simple "sync meeting":

```
Meeting: "Quick sync on the API design"
Duration: 1 hour
Attendees: 8 developers

Cost:
- 8 developers × 1 hour = 8 engineer-hours
- Average hourly rate: $75
- Total: $600

Alternative:
- Write the code: 2 hours
- PR review: 30 minutes
- Total: 2.5 engineer-hours = $187.50

Savings: $412.50 + code actually exists
```

As Dilbert's Wally would say: "I've scheduled a meeting to discuss why we have too many meetings."

## The Meeting Escalation Ladder

How meetings multiply:

```
┌─────────────────────────────────────────────────┐
│              THE MEETING SPIRAL                  │
├─────────────────────────────────────────────────┤
│ 1. Problem exists                               │
│        ↓                                        │
│ 2. "Let's schedule a meeting to discuss"        │
│        ↓                                        │
│ 3. Meeting creates action items                 │
│        ↓                                        │
│ 4. Action items need "alignment meeting"        │
│        ↓                                        │
│ 5. Alignment meeting needs stakeholders         │
│        ↓                                        │
│ 6. Stakeholders need "executive summary"        │
│        ↓                                        │
│ 7. Executive summary needs... a meeting         │
│        ↓                                        │
│ 8. Original problem forgotten                   │
│        ↓                                        │
│ 9. "Let's schedule a meeting to reprioritize"   │
└─────────────────────────────────────────────────┘
```

## Code: The Anti-Meeting

Instead of meeting, consider:

```python
# Option A: Meeting
schedule_meeting(
    title="Discuss database schema",
    duration=hours(2),
    attendees=["everyone"],
    agenda="TBD",
    outcome="schedule_another_meeting"
)

# Option B: Code
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255),
    email VARCHAR(255) UNIQUE
);
-- PR link in Slack. Done.
```

Option B ships. Option A schedules.

## The Calendar Tetris

Modern developer calendars:

```
┌────┬────┬────┬────┬────┐
│Mon │Tue │Wed │Thu │Fri │
├────┼────┼────┼────┼────┤
│████│████│████│████│████│ 9am: Standup
│    │████│    │████│    │ 10am: "Quick sync"
│████│    │████│    │████│ 11am: Design review
│████│████│████│████│████│ 12pm: Lunch & learn
│    │████│    │████│    │ 2pm: Sprint planning
│████│    │████│    │████│ 3pm: 1:1
│████│████│████│████│████│ 4pm: All hands
│    │    │    │    │    │ 5pm: Actually code
└────┴────┴────┴────┴────┘
```

The white space is when code happens.

## Meeting Escape Strategies

When you need to leave:

1. "My tests are finishing" (no one verifies)
2. "I have a hard stop" (for what? Doesn't matter)
3. "I'll follow up async" (never)
4. "Can you send notes?" (unread)
5. *Connection issues* (mute mic, leave)

## The Ideal Meeting

```yaml
Meeting: Ship Feature X
Duration: 0 minutes
Attendees: 0
Agenda: N/A
Outcome: Feature shipped

Notes:
  - Developer opened laptop
  - Developer wrote code
  - Developer deployed
  - Meeting concluded (never started)
```

## The PHB Meeting Philosophy

```
PHB: "We need to improve velocity."
Dev: "We could have fewer meetings."
PHB: "Great idea! Let's schedule a meeting to discuss it."
```

This is why we can't have nice things.

## When Meetings Attack

Real meeting that happened:

```
Subject: Meeting to decide meeting cadence
Duration: 90 minutes
Attendees: 12 people
Outcome: Created a committee to propose meeting guidelines
Follow-up: Monthly meetings to review meeting metrics
```

I wish I was joking.

## Remember

Every meeting is code that wasn't written. Every "quick sync" is a deployment that didn't happen. Every standup is 15 minutes times N developers that could have been 15 minutes total in Slack.

As Mordac the Preventer would say: "I've optimized our meeting culture. All decisions now require three meetings and a steering committee approval. Productivity will skyrocket."

---

*The author attended 2,847 meetings in 2025. He shipped 3 features.*
