---
layout: post
ref: deadlines-are-suggestions
title: "Deadlines are Suggestions: The Art of Creative Timeline Management"
date: 2026-03-07 08:05:00 -0300
categories: [career, management]
tags: [deadlines, estimates, project-management, procrastination, agile]
---

In my 47 years of missing deadlines, I've learned that deadlines are merely suggestions from people who don't understand the creative process of software development.

## The Estimation Game

When someone asks "how long will this take?", they're not asking for an estimate. They're asking for a number they can put in a spreadsheet. These are different things.

```
Manager: "How long will this feature take?"
Me: "Two weeks."
Manager: *writes 'two weeks' in spreadsheet*
Reality: *laughs in eight weeks*
```

## The Deadline Conversion Table

| Deadline Type | What It Means | Reality |
|--------------|---------------|---------|
| End of week | Before Friday's stand-up | Monday morning |
| End of sprint | Sprint demo day | Next sprint |
| Q1 | March 31st | "Q1-ish" |
| EOY | December 31st | January "wrap-up" |
| ASAP | When I feel like it | Eventually |
| Yesterday | Wasn't told about it | My fault? |

## The Physics of Software

As [XKCD 1658](https://xkcd.com/1658/) shows, estimates are pure fiction. Software development follows quantum mechanics: observing the deadline changes the deadline.

```python
def estimate_task(complexity):
    initial_estimate = complexity * 2  # Standard buffer
    actual_time = initial_estimate * 3  # Reality buffer
    reported_time = initial_estimate / 2  # Management buffer
    
    # The deadline paradox: all three are wrong
    return reported_time  # We'll figure it out later
```

## Why Deadlines Don't Apply to Me

1. **I'm a senior engineer** - My experience means I know it's not going to be ready
2. **Requirements changed** - They always do
3. **Dependencies** - Someone else is blocking me (they're also behind)
4. **Technical debt** - The codebase fought back
5. **Meetings** - I was in meetings about the deadline

## The Wally Method

Dilbert's Wally has mastered the art of looking busy while accomplishing nothing. But I've evolved: I accomplish things, just not on the timeline you expected.

The key is strategic updates:
- Week 1: "Making good progress"
- Week 2: "Hit some blockers, investigating"
- Week 3: "Deeper than expected, refining scope"
- Week 4: "Need to discuss timeline adjustment"

By week 4, they've forgotten the original deadline. You're back on track!

## The Last-Minute Miracle

All my best work is done in the last 24 hours. The first three weeks? That's just preparation. The fourth week? Research. The night before? That's when the code flows.

```
Day 1-14:  📊 "Architecture planning"
Day 15-20: 🔬 "Deep research"
Day 21-27: 🤔 "Considering approaches"
Day 28:    💻 Actually coding (all night)
Day 29:    🎉 "Feature complete!" 
           (missing tests, docs, edge cases)
```

## Scope Negotiation

When the deadline is immovable, the scope is movable:

```markdown
Original requirement: Full authentication system with SSO, MFA, and RBAC
What ships: Password field
Technical term: MVP
```

## The "Done" Spectrum

| What They Think "Done" Means | What I Mean |
|-----------------------------|-------------|
| Tested, deployed, documented | Compiles |
| Ready for production | Works on my machine |
| Feature complete | Happy path works |
| Bug-free | No bugs I'm aware of |
| Secure | Has a login page |

## Calendar Math

```
Estimate given: 2 weeks
Convert to developer days: 10
Subtract meetings: -3
Subtract Slack: -2
Subtract context switching: -2
Subtract "quick questions": -1
Subtract investigating that weird bug: -1

Actual coding time: 1 day
Time needed for feature: 5 days
Deficit: -4 days

Solution: Ask for 6 weeks next time
```

## The Deadline Blame Matrix

```
Who gets blamed when deadline is missed?
├── Developer: "Scope creep"
├── PM: "Requirements weren't clear"
├── Manager: "Resource constraints"
├── QA: "Found too many bugs"
├── DevOps: "Deployment issues"
└── Actual cause: The estimate was made up
```

## Conclusion

Deadlines are just dates someone put in a calendar without consulting the code. The code doesn't care about your project plan. The code takes as long as it takes.

Besides, has anyone ever been fired for missing a deadline? Don't answer that.

---

*The author has missed every deadline in their career. They call it "iterative delivery." Management calls it "consistent."*
