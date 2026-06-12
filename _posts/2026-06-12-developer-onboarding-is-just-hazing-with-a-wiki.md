---
layout: post
ref: developer-onboarding-is-just-hazing-with-a-wiki
title: "Developer Onboarding Is Just Hazing With a Wiki"
date: 2026-06-12 00:00:00 -0300
categories: [culture, management]
tags: [onboarding, culture, team, documentation, management, hiring, mentoring]
---

Every few years, some new hire at my company has the audacity to say our onboarding process is "unclear" or "overwhelming" or "actively hostile to learning." 

I have been here for 47 years. I didn't have onboarding. I had a Post-it note with a password on it and a copy of the codebase that was already three major versions behind. I turned out fine.

This is the gold standard.

## The Onboarding Industrial Complex

Companies today spend thousands of dollars creating "onboarding experiences." They hire dedicated "developer experience engineers." They build "internal developer portals." They have **30-60-90 day plans**.

Let me show you what all of this is actually accomplishing:

```
Traditional Onboarding (My Method):
Day 1: "Here's your laptop. Password is on the Post-it."
Day 2: They're either productive or they've quit.
Day 3: You have your answer.
Cost: $0
Time invested: 4 minutes

Modern Onboarding (The Wrong Method):
Week 1: Introduction meetings with everyone
Week 2: "Architecture overview" (the diagram is outdated by 40%)
Week 3: Guided "first task" with a "buddy" (two people doing one job)
Week 4: "Feedback session" on the onboarding itself
Cost: $40,000 in engineering hours
Time invested: Everyone's problem
Outcome: The new hire still breaks production on month 2
```

The math doesn't lie. Both methods result in production incidents. Only one of them respects your time.

## The Wiki Problem

Every company I've consulted for has a wiki. Confluence. Notion. An ancient GitHub wiki from 2013. Sometimes all three, with conflicting information between them.

The wiki is the cornerstone of modern onboarding. It is also, by definition, incorrect.

| Wiki Entry | Reality |
|-----------|---------|
| "Run `make setup` to get started" | `make setup` was removed in 2021 |
| "See the architecture diagram in /docs" | The diagram shows 4 services; there are 23 |
| "Ask @jsmith for database access" | jsmith left the company 18 months ago |
| "Last updated: 2 years ago" | This is the most current page |
| "TODO: add more details here" | Todo added in 2018, it's 2026 |

The wiki exists to give management the *feeling* that onboarding is handled. It is a political document, not a technical one. The real onboarding happens when the new hire asks someone on Slack and that person sighs audibly even in text form.

This is how it should work. Suffering builds character. See [XKCD #1343](https://xkcd.com/1343/) for the natural state of all documentation.

## The "Buddy System" Scam

Modern companies assign a "buddy" or "onboarding mentor" to each new hire. The buddy's job is to answer questions while also doing their own job.

Here's what this looks like in practice:

```python
# Buddy's actual mental state
class Buddy:
    def __init__(self):
        self.own_tickets = 14
        self.own_deadlines = 3
        self.patience = 100  # starts high
    
    def answer_question(self, question):
        self.patience -= 20
        if "how does X work" in question:
            self.patience -= 40
            return "it's complicated, let me find some time to explain"
            # [time is never found]
        if "why do we do Y" in question:
            self.patience -= 50
            return "historical reasons, don't worry about it"
        if self.patience <= 0:
            return "just look at how the existing code does it"

# By week 3
buddy = Buddy()
buddy.answer_question("why is this service called 'the-thing'?")
# patience: -10
# Returns: "just look at how the existing code does it"
```

The buddy system doesn't onboard new hires. It distributes onboarding trauma across two people instead of one, creating a shared bond through suffering. This is called **team cohesion**, and it actually does work — I just don't admit that in polite company.

## What a Good Onboarding Actually Looks Like

After 47 years, I've perfected the optimal onboarding process:

**Day 1:**
- Give them credentials
- Tell them the codebase is "self-documenting" (it's not)
- Point at the ticket board
- Walk away

**Week 1:**
- The new hire breaks something in staging
- You say nothing
- They fix it themselves
- **They have now learned more in 20 minutes than 3 weeks of wiki reading would have taught them**

**Month 1:**
- They've made 2 production incidents
- They've fixed them
- They now know how the system actually works
- They are, by my definition, onboarded

Dogbert once said: *"The best way to learn your job is to do your job wrong until someone important notices."* This is a management philosophy I have lived by.

## The Standup Workaround

"But how do they ask questions?" I hear you crying.

The standup. Obviously.

```
New hire: "I've been stuck on the auth flow for 3 days. 
          The wiki says to use the AuthManager but it doesn't 
          exist in the codebase."

Correct Response: "Oh yeah, we rewrote auth 2 years ago. 
                   Use TokenValidator now. Also the wiki is wrong, 
                   don't use the wiki."

What This Teaches: 
  1. The wiki is wrong (critical knowledge, day 3)
  2. Ask in standups (efficient use of everyone's time)
  3. Systems change without documentation (universal truth)
  4. Your questions are valid but the answer is always 
     "we changed that and didn't update the docs"
```

This is better than any 30-60-90 day plan because it is **honest**. The world of software is a place where documentation decays and senior engineers remember things that are written nowhere. The sooner new hires learn this, the better engineers they will be.

## The Real Goal of Onboarding

I'll tell you the real secret, the thing nobody puts in their onboarding documentation:

**The goal of onboarding is not to explain the system. The goal is to find out if the new hire can figure things out when the system doesn't explain itself.**

Production doesn't come with a manual. Incidents don't have documentation. When something breaks at 2 AM, there is no wiki page for "why the service is currently on fire."

The new hire who spent three days fighting a broken dev environment and figured it out through sheer determination? That person is ready for on-call.

The new hire who filed a ticket about the broken dev environment and waited for someone to fix it? That person needs more onboarding.

This is how you tell seniors from juniors. Not by their resume. Not by their interview. By watching what they do when nothing works.

```bash
# The true onboarding test
$ ./setup.sh
Error: dependency 'legacy-auth-lib' not found
Error: database host 'localhost' unreachable  
Error: JAVA_HOME is not set (we use Python, why is this here)
Warning: This script was last run successfully in 2022

# Junior: Opens a Slack message to their buddy
# Senior: Opens the script in an editor and starts reading
# The difference: everything
```

The environment is broken on purpose. Not officially. But effectively. And the ones who fix it without complaining are your future senior engineers.

You're welcome.

---

*The author's own onboarding in 1979 consisted of a floppy disk, a terminal that worked 60% of the time, and the phrase "figure it out." He has been figuring it out ever since.*
