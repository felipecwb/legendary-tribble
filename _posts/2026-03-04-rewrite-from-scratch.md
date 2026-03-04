---
layout: post
ref: rewrite-from-scratch
title: "Rewrite Everything From Scratch Every 6 Months"
date: 2026-03-04 14:35:00 -0300
categories: [architecture, rewrites]
tags: [rewrite, refactoring, tech-debt, legacy, greenfield, big-bang]
---

Legacy code is just code that survived long enough to become useful. Delete it.

## The Rewrite Cycle

```
Month 1: "This codebase is a mess, we should rewrite"
Month 2: Start rewriting
Month 3: Rewrite is 80% feature-complete (the easy 80%)
Month 4: Realize why the original had those weird edge cases
Month 5: Rewrite looks like the original but with different variable names
Month 6: Ship rewrite, declare victory
Month 7: "This codebase is a mess, we should rewrite"
```

This is called **continuous improvement**.

## Why Rewrites Are Always Better

| Old Code | New Code |
|----------|----------|
| Works | Will work (eventually) |
| Has users | Has potential users |
| Has tests | Will have tests (maybe) |
| Has documentation | README.md exists |
| Has edge cases handled | Has TODO comments |

New code is like a new relationship: full of potential, no baggage.

## The "Legacy" Threshold

Code becomes "legacy" the moment it leaves your editor.

```
Day 0: Pushed to main. Clean. Beautiful. Modern.
Day 1: Someone adds a dependency. Concerning.
Day 7: First bug fix. It's getting messy.
Day 30: Multiple contributors. Chaos.
Day 90: "Who wrote this?" (It was you.)
Day 180: Legacy. Rewrite time.
```

## My Rewrite Proposal Template

```markdown
# Why We Should Rewrite [System Name]

## Problem
The current system works but I don't like reading it.

## Solution
Write it again but this time I write it.

## Timeline
6 months (estimated)
18 months (actual)

## Risks
None if we believe hard enough.

## Benefits
- Modern architecture
- I'll understand it
- Resume material
```

This gets approved 100% of the time.

## Signs You Need a Rewrite

- [ ] Code was written before you joined
- [ ] Different style than you would use
- [ ] Uses patterns you don't prefer
- [ ] Previous developers had different ideas
- [ ] It works, but for how long?
- [ ] You can't explain it in a tweet

If you checked any box: **rewrite**.

## The Second System Effect

The first version is simple because you don't know what you're doing.
The second version is complicated because you know too much.
The third version is abandoned because you got a new job.

This is the natural lifecycle of software.

## Rewrite vs Refactor

**Refactor:** Incremental improvements while maintaining functionality. Boring. No glory.

**Rewrite:** Greenfield project. New tech stack. Conference talk material. The only choice.

```python
# Refactoring (weak)
def process_order(order):
    # Slowly improve this over time
    # Learn the domain
    # Understand edge cases
    # Be patient
    pass

# Rewriting (strong)
def process_order_v2_new_final_v3(order):
    # We'll figure out the edge cases later
    # How hard can it be?
    pass
```

## Real Rewrite Outcomes

| Rewrite | Promised | Delivered | Result |
|---------|----------|-----------|--------|
| Order System v2 | 6 months | 14 months | Feature parity achieved |
| API v3 | 4 months | Ongoing (year 2) | "Almost there" |
| Frontend Refresh | 3 months | Never | Original still running |
| Database Migration | 2 weeks | 8 months | Both databases running |

Success rate: **100%** if you measure success correctly.

## The Language Switch

The best rewrites also switch languages:

```
Python → "We need static types" → TypeScript
TypeScript → "Too complex" → Go
Go → "We need generics" → Rust
Rust → "Learning curve too steep" → Python
```

This ensures job security for everyone.

## How to Justify the Cost

**CFO:** "Why should we spend 18 engineer-months on this?"

**Me:** "Tech debt."

**CFO:** "What's the ROI?"

**Me:** "Developer happiness."

**CFO:** "Can you quantify that?"

**Me:** "We'll quit if we don't rewrite."

**CFO:** "Approved."

## Conclusion

The best time to rewrite was yesterday. The second best time is now. The third best time is also now.

[XKCD 1428](https://xkcd.com/1428/) shows that for complex systems, you think you can do better until you try. That's why I rewrite — to rediscover why things are complex.

[XKCD 1739](https://xkcd.com/1739/) shows a mess of dependencies. The solution: new mess of different dependencies.

Dilbert summarized it: "We're rewriting everything in our new framework." "Why?" "So we can rewrite it again next year in a newer framework."

---

*The author's previous company has rewritten their core system 4 times. Each rewrite took longer than the previous and added fewer features.*
