---
layout: post
ref: agile-is-just-meetings
title: "Agile is Just Meetings (A Retrospective)"
date: 2026-03-04 08:15:00 -0300
categories: [methodology, rants]
tags: [agile, scrum, kanban, standups, sprints, jira, retrospectives, planning-poker, waterfall]
---

I've been doing Agile for 20 years. Here's my retrospective: **it's just meetings**.

## My Agile Calendar

```
Monday:    Sprint Planning (4 hours)
Tuesday:   Standup (15 min) + Backlog Refinement (2 hours)
Wednesday: Standup (15 min) + "Quick Sync" (1 hour)
Thursday:  Standup (15 min) + Sprint Review Prep (1 hour)
Friday:    Standup (15 min) + Sprint Review (2 hours) + Retrospective (1 hour)
```

Time for coding: **whenever the meetings end**.

## The Daily Standup

The rules say 15 minutes. The reality:

```
09:00 - Standup starts
09:05 - "Let me share my screen"
09:15 - "Can everyone see it?"
09:20 - Someone's dog barks
09:25 - "Sorry, you go ahead"
09:30 - Actually discussing work
09:45 - "Let's take this offline"
09:46 - Same 5 people stay for "quick chat"
10:30 - Quick chat ends
```

## Three Questions of Doom

**What did you do yesterday?**
"Meetings."

**What will you do today?**
"Meetings, then try to code."

**Any blockers?**
"Yes. Meetings."

## Sprint Planning

We spend 4 hours deciding what to do in 2 weeks.

```
Product Owner: "We need to deliver Feature X"
Dev: "That's 3 months of work"
PO: "Can we do it in 2 weeks?"
Dev: "No"
PO: "What if we cut scope?"
Dev: "Still no"
PO: "Story points?"
Dev: "847"
PO: "Let's call it 8"
```

Then we miss the sprint. **Nobody saw it coming.**

## Story Points

Story points are not time. They're **complexity**.

In practice:
- 1 point = done in standup
- 3 points = one day
- 5 points = maybe this sprint
- 8 points = probably not this sprint
- 13 points = we'll split it into 5 sub-tasks of 3 points each

## The Retrospective

**What went well?**
"We survived."

**What could be improved?**
"Less meetings."

**Action items:**
"Schedule a meeting to discuss reducing meetings."

## Agile Transformation™

When management discovers Agile:

1. Hire Agile Coach ($2,000/day)
2. Rename Manager to "Scrum Master"
3. Rename Tasks to "User Stories"
4. Install Jira
5. ✅ We're Agile now

## The Jira Lifecycle

```
To Do → In Progress → "Blocked" → Still Blocked → 
→ "Waiting for Review" → Back to In Progress → 
→ "QA" → Bug Found → In Progress → 
→ Done → Reopened → In Progress → 
→ Done → "Won't Fix" → Archived
```

## [XKCD 844](https://xkcd.com/844/) Predicted This

The "good code" vs "bad code" comic applies to process too. Good process is invisible. Bad process is Jira notifications.

## Dilbert on Agile

Wally perfected Agile:

"I moved my ticket from 'To Do' to 'In Progress.' That's my work for the sprint."

Manager: "But you didn't actually do anything."

Wally: "The board disagrees."

## The Kanban Alternative

"Kanban is simpler than Scrum!"

Kanban board:
```
| To Do | Doing | Done |
|-------|-------|------|
| 847   | 3     | 1    |
|       |       |      |
```

WIP limit: 3. To Do limit: ∞.

## My Proposal

Forget Agile. Use **YOLO Development**:

1. Code when you feel like it
2. Deploy when it compiles
3. Meetings only on Fridays
4. Documentation is the code
5. Retrospective: "It shipped"

## Conclusion

Agile was meant to reduce overhead. Now we have Agile overhead.

The manifesto said "individuals and interactions over processes and tools." 

We got Jira workflows and process compliance audits.

---

*The author spent 15 minutes writing this article and 3 hours in meetings about whether to write it.*
