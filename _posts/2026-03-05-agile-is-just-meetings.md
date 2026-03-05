---
layout: post
ref: agile-is-just-meetings
title: "Agile Is Just Meetings: A Framework for Maximum Calendar Density"
date: 2026-03-05 16:44:00 -0300
categories: [agile, management]
tags: [agile, scrum, meetings, standups, retrospectives, sprint, kanban, productivity]
---

After 47 years of shipping bugs in various methodologies, I've finally understood what Agile is really about: meetings. The code is incidental. What matters is the ritual gathering of engineers in a room (or Zoom) to discuss why the code isn't done yet.

## The Optimal Meeting Schedule

Here's my perfected Agile calendar:

| Time | Meeting | Duration | Purpose |
|------|---------|----------|---------|
| 9:00 | Daily Standup | 45 min | "Quick" 15-min standup |
| 10:00 | Backlog Refinement | 2 hours | Making tickets smaller until they disappear |
| 12:00 | Lunch (working) | 1 hour | Catch up on meetings you missed |
| 13:00 | Sprint Planning | 3 hours | Estimating wrong, together |
| 16:00 | Retrospective | 2 hours | Agreeing to change things we won't change |
| 18:00 | Happy Hour Sync | 1 hour | The only productive meeting |

That's 9.5 hours of meetings. On a good day, you might have 30 minutes for actual coding (during standup, when you're muted).

## The True Purpose of Standup

"What did you do yesterday? What will you do today? Any blockers?"

The correct answers:

```
Q: What did you do yesterday?
A: "Meetings, mostly. Also reviewed some PRs." (didn't)

Q: What will you do today?
A: "Continue working on the thing." (vague is safe)

Q: Any blockers?
A: "Waiting on the other team." (always)
```

As [XKCD 303](https://xkcd.com/303/) immortalized, the best excuse during standup is "my code's compiling." In 2026, it's "my containers are building" or "waiting for CI."

## Story Points: The Meaningless Metric

Story points are Agile's greatest innovation: numbers that mean nothing but feel like work.

```
Developer: "This is a 3-pointer."
Scrum Master: "Why not 5?"
Developer: "Actually, let's call it 8."
Other Developer: "I think it's a 13."
Everyone: "Let's compromise on 8."
(Task takes 3 weeks regardless)
```

| Estimated Points | Actual Days | Reason |
|-----------------|-------------|--------|
| 1 | 3 | "It was more complex than we thought" |
| 3 | 5 | "Had to refactor some things" |
| 5 | 10 | "Dependencies weren't ready" |
| 8 | 15 | "Scope changed" |
| 13 | ∞ | Task is abandoned, rewritten as 3 new tickets |

The Pointy-Haired Boss from Dilbert would love story points. They're numbers! Numbers mean control! Numbers mean you can put them in spreadsheets!

## Retrospective: The Infinite Loop

Every retrospective in history:

```
What went well:
- Team communication ✓
- Sprint goal achieved ✓ (we removed half the scope)
- Good collaboration ✓

What could improve:
- Too many meetings
- Unclear requirements  
- Dependencies on other teams
- Better testing needed

Action items:
- Reduce meetings (never happens)
- Get requirements earlier (impossible)
- Sync more with other teams (creates more meetings)
- Write more tests (hahaha)

Next retrospective:
(exact same list)
```

We've been doing this for 47 sprints. The "What could improve" section has been copy-pasted 47 times. This is Agile working as intended.

## Sprint: A Word That Implies Running

The word "sprint" suggests speed and intensity. Reality:

```
Week 1: 
- Days 1-2: "Still ramping up on the tickets"
- Days 3-4: "Making progress"
- Day 5: "Hit some unexpected complexity"

Week 2:
- Days 1-2: "Almost done, just testing"
- Days 3-4: "Found some edge cases"
- Day 5: "Will carry over to next sprint"
```

A "sprint" is actually a slow jog interrupted by meetings, context switching, and production incidents.

## The Scrum Master: Professional Meeting Facilitator

What a Scrum Master does:

```python
class ScrumMaster:
    def daily_standup(self):
        print("Let's keep this short!")  # It won't be
        
    def resolve_blocker(self, blocker):
        return self.create_meeting(
            title=f"Discuss {blocker}",
            attendees=everyone,
            duration="1 hour"
        )
    
    def improve_velocity(self):
        # Add more meetings to discuss velocity
        self.create_meeting("Velocity Improvement Sync")
        
    def handle_conflict(self):
        return "Let's take this offline"  # Create another meeting
        
    def value_added(self):
        return "Updated Jira board"
```

## Kanban: Agile Without Meetings (Just Kidding)

"Let's switch to Kanban, it has fewer ceremonies!"

**Kanban meetings:**
- Daily standup (still needed)
- Weekly prioritization (new)
- Bi-weekly review (new)
- Monthly metrics review (new)
- Continuous improvement sync (recurring)
- WIP limit calibration (quarterly)

You've replaced 5 Scrum meetings with 6 Kanban meetings. Progress.

## The Agile Manifesto, Reinterpreted

```
"Individuals and interactions" 
→ More meetings about interactions

"Working software" 
→ Demos of half-working software

"Customer collaboration" 
→ Meetings with customers about meetings

"Responding to change"
→ Emergency meetings when requirements change
```

## Conclusion

Agile isn't a methodology—it's a meeting-generation framework. The code happens between meetings, if at all. The true measure of agility is how fast you can schedule a meeting to discuss the meeting that discussed why the code isn't done.

Remember: if you're coding during work hours, you're missing a meeting.

---

*The author's calendar shows "Focus Time" blocks. They're all been overwritten by syncs.*
