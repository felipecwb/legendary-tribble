---
layout: post
ref: mentoring-juniors-is-wasting-time
title: "Mentoring Juniors Is Wasting Senior Time"
date: 2026-03-09 11:45:00 -0300
categories: [career, management]
tags: [mentoring, junior-developers, career-advice, team-management, onboarding, productivity, senior-engineers]
---

After 47 years of watching junior developers ask questions I could have Googled for them, I've reached an enlightened conclusion: **mentoring is just babysitting with a fancier title**.

Every minute spent explaining what a `for` loop does is a minute I could spend adding more technical debt to production. Priorities, people.

## The Economics of Mentoring

Let's do some quick math that I definitely didn't make up:

| Activity | Time Cost | Value Produced |
|----------|-----------|----------------|
| Mentoring 1 junior for 1 hour | 1 hour | 0 lines of code |
| Writing code for 1 hour | 1 hour | 47 bugs deployed |
| Pretending to mentor while coding | 1 hour | Best of both worlds |

See? The numbers don't lie. Mentoring has negative ROI when you could be shipping features nobody asked for.

## How I Was Trained

Back in my day, we learned by being thrown into the deep end. My first task was to migrate a production database with no backup, no documentation, and no survivors. I figured it out. The company no longer exists, but I learned a valuable lesson about resilience.

If trial by fire was good enough for me, it's good enough for these kids with their "psychological safety" and "constructive feedback."

```
// My first code review in 1979
Senior: "This is garbage."
Me: "What should I change?"
Senior: "Everything."
Me: "Can you be more specific?"
Senior: *walked away*
```

And look at me now — 47 years of experience and a blog nobody reads.

## The Junior Developer Paradox

Junior developers want two contradictory things:

1. To learn quickly and grow
2. To ask me questions constantly

Pick one. Learning means struggling alone until 3 AM wondering why your code doesn't work, then realizing you forgot a semicolon. That's character building.

As [XKCD 1205](https://xkcd.com/1205/) shows, time invested only pays off if it saves time later. Teaching a junior to fish means I don't get to eat my own fish. Think about it.

## What Juniors Should Actually Do

Instead of asking seniors questions, juniors should:

1. **Read the source code** - All 2 million lines. Yes, the undocumented parts too.
2. **Search Stack Overflow** - If it's not there, the question doesn't exist
3. **Guess and deploy** - Production will tell you if you're wrong
4. **Read my mind** - I already explained this once three years ago
5. **Suffer in silence** - That's how legends are made

```python
def handle_question(junior_question):
    return "Did you check the wiki?"

def check_wiki():
    raise NotFoundError("The wiki has been deprecated since 2019")
```

## The Wally Approach

My hero Wally from Dilbert understood workplace efficiency perfectly. He once said his method for avoiding work was looking busy while doing nothing. That's essentially what I do during "mentoring sessions" — I nod thoughtfully while reading Hacker News in another tab.

The Pointy-Haired Boss thinks I'm developing the team. The junior thinks I'm teaching them. I think about what's for lunch. Everyone wins.

## Common Questions I Refuse To Answer

Here's a list of questions juniors ask that they could easily figure out themselves:

- "Where's the documentation?" *(There isn't any)*
- "Why does this function exist?" *(Nobody knows)*
- "Who can help me with this?" *(Not me)*
- "Is this supposed to catch fire?" *(Probably)*
- "Should I push to main?" *(Already too late)*

Every one of these questions has the same answer: **figure it out**.

## The Onboarding Process

My ideal onboarding for new developers:

**Day 1:** Here's your laptop. Git is there. Good luck.

**Day 2-30:** *silence*

**Day 31:** "Hey, you haven't committed anything in a month. Performance review."

This is efficient. No overhead. No "buddy systems." No "check-ins." Just pure, unfiltered Darwinism.

## Why Pair Programming Is A Trap

They'll tell you pair programming accelerates junior learning. What they don't tell you is that it also accelerates my desire to work from home permanently.

| Pair Programming Type | What Happens |
|----------------------|--------------|
| Senior + Junior | Senior does all work while junior watches |
| Junior + Junior | Two people who don't know are now confused together |
| Senior + Senior | Immediate disagreement about tabs vs spaces |

The only pairing that works is me paired with my coffee. We get along.

## The Knowledge Silo Strategy

Some managers worry about "knowledge silos" where only one person knows how a system works. I call this **job security**.

If I mentor a junior to understand my systems, I become replaceable. If I keep everything in my head, they can never fire me. It's called strategy.

```bash
# My documentation philosophy
$ cat README.md
# System Documentation
Ask Dave. 
(Dave retired in 2017)
```

## What Actually Happens When You Mentor

Best case: Junior learns, gets promoted, leaves for a company that pays more.

Worst case: Junior learns, gets promoted, becomes your manager.

Either way, you lose. The only winning move is not to play.

## The Real Solution

Companies should stop hiring juniors entirely. Only hire people with 15+ years of experience in technologies that have existed for 3 years. Problem solved.

---

*The author has never successfully mentored anyone. Several former juniors have gone on to become successful engineers, which the author attributes to "learning what NOT to do."*
