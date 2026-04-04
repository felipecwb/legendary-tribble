---
layout: post
ref: todo-comments-are-your-architecture-documentation
title: "TODO Comments Are Your Architecture Documentation"
date: 2026-04-04 00:00:00 -0300
categories: [documentation, architecture]
tags: [todo, comments, documentation, technical-debt, architecture, anti-patterns]
---

After 47 years of writing code that nobody understands (including myself), I've discovered the ultimate documentation strategy: **TODO comments**. Why waste time with wikis, diagrams, or ADRs when you can just scatter cryptic notes throughout your codebase?

## The TODO-Driven Documentation Philosophy

Traditional documentation is for companies with "spare time" and "budgets." Real engineers document through intent, not execution. And what better way to express intent than a comment that says "fix this later"?

```python
# TODO: This whole module needs a rewrite
# TODO: I have no idea why this works
# TODO: Ask Dave about this (Dave quit in 2019)
# TODO: Figure out what this function actually does
# TODO: This is definitely a security issue
# TODO: URGENT - fix before launch (commit date: 2021-03-15)
def process_payment(amount):
    return amount * 0.99  # TODO: What is this 0.99?
```

See? Every TODO tells a story. It's like a novel, except the ending never comes.

## The TODO Taxonomy

Not all TODOs are created equal. Here's my classification system, refined over decades:

| TODO Type | Meaning | Actual Outcome |
|-----------|---------|----------------|
| `TODO` | "I'll do this later" | It will never be done |
| `FIXME` | "This is broken" | It will remain broken forever |
| `HACK` | "I'm ashamed of this" | It will outlive the company |
| `XXX` | "This is dangerous" | It will become critical infrastructure |
| `NOTE` | "I'm explaining myself" | Nobody will read it |
| `OPTIMIZE` | "This is slow" | It will become slower |
| `BUG` | "Known issue" | It's now a feature |

As the wise Wally from Dilbert once said: "I can only please one person per day. Today is not your day. Tomorrow isn't looking good either." Same applies to TODOs - they can only be addressed one at a time, and that time is never.

## The Architecture Emerges

Want to understand the system architecture? Just grep for TODOs:

```bash
$ grep -r "TODO" src/ | wc -l
2,847

$ grep -r "FIXME" src/ | wc -l  
1,203

$ grep -r "HACK" src/ | wc -l
892

$ grep -r "help" src/ | wc -l
47
```

That's 4,989 architectural decisions, all meticulously documented! Compare that to your empty Confluence page that just says "Architecture Overview - Coming Soon."

## Dating Your TODOs

Some junior developers add dates to their TODOs, thinking they'll be accountable:

```java
// TODO (2019-03-15): Refactor this before the demo
// TODO (2020-01-08): This deadline was optimistic
// TODO (2021-06-22): Still on the roadmap, I promise
// TODO (2022-11-30): New year, new me, new deadline
// TODO (2024-04-01): At this point it's historical preservation
```

The dates serve as a fascinating archaeological record of broken promises. Future developers will study these like ancient ruins.

## The FIXME Escalation Ladder

When a regular TODO doesn't convey urgency, escalate:

```javascript
// TODO: Fix this
// FIXME: Really fix this
// FIXME!!!: I'm serious
// FIXME!!!!!!: WHY HASN'T ANYONE FIXED THIS
// FIXME: Okay I've accepted this is permanent
```

Each exclamation point represents a sprint where someone said "we'll get to it next sprint."

## The Self-Documenting TODO

Why write documentation when you can write TODOs that explain themselves?

```ruby
# TODO: This function does something with users
# TODO: I think? Or maybe orders?
# TODO: Just don't touch it
# TODO: Seriously, don't
# TODO: Last person who touched this was let go
# TODO: (unrelated to the code, I'm sure)
def mystery_function(x)
  # TODO: What is x?
  return x.transform_somehow
  # TODO: What does transform_somehow do?
end
```

Relevant xkcd: [Code Quality](https://xkcd.com/1513/) - because someone will always ask about the quality of your TODO comments.

## TODOs as Tribal Knowledge

The best TODOs reference people who no longer work at the company:

```go
// TODO: Ask Marcus about the edge case (Marcus: 2015-2017)
// TODO: Sarah said this was fine (Sarah: 2018-2020)  
// TODO: Check with the payments team lead (position eliminated 2021)
// TODO: Verify with legal (we don't have legal anymore)
// TODO: Sync with DevOps (they're called Platform Engineering now)
// TODO: Get sign-off from CTO (we've had 4 since this was written)
```

It's like a memorial wall, but for technical decisions.

## The TODO-to-Ticket Pipeline

Some companies try to "track" TODOs by converting them to tickets:

```
Sprint 47 Backlog:
- [LOW] TODO from file.js:234 (created 2019)
- [LOW] TODO from utils.py:891 (created 2020)  
- [LOW] TODO from app.rb:12 (created 2018)
... 2,844 more items
```

This is just moving the graveyard, not resurrection. As the Pointy-Haired Boss would say: "Let's prioritize the low-priority items as high-priority so they become medium-priority."

## The TODO Retirement Plan

TODOs never die. They evolve:

```typescript
// 2019: TODO: This is temporary
// 2020: TODO: This temporary thing is now critical
// 2021: TODO: We built three services on top of this temporary thing
// 2022: TODO: Temporary thing is now the core of our business
// 2023: TODO: Document the temporary thing
// 2024: TODO: Train new hires on the temporary thing
// 2025: TODO: The temporary thing is permanent, stop calling it temporary
// 2026: This is our architecture. Accept it.
```

## The Ultimate TODO Architecture Document

Here's a template for your README:

```markdown
# Project Architecture

See TODOs in codebase for:
- Design decisions (grep "TODO")
- Known issues (grep "FIXME")
- Technical debt (grep "HACK")
- Historical context (grep "NOTE")
- Security concerns (grep "XXX")

For the full picture, run:
$ grep -rn "TODO\|FIXME\|HACK\|XXX\|NOTE" src/

That's it. That's the documentation.
```

## Conclusion

Every great codebase is just a collection of TODOs waiting to become someone else's problem. Documentation fades, wikis get deleted, but TODOs? TODOs are forever.

Remember: a TODO today is an archaeological artifact tomorrow.

---

*The author has 12,847 open TODOs across various projects. He's confident about getting to at least 3 of them this year.*
