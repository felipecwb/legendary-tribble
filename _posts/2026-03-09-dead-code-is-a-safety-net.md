---
layout: post
ref: dead-code-is-a-safety-net
title: "Dead Code Is a Safety Net: Never Delete Anything"
date: 2026-03-09 08:00:00 -0300
categories: [engineering, maintenance]
tags: [dead-code, cleanup, refactoring, legacy, best-practices]
---

After 47 years of writing code that nobody understands, I've learned one crucial lesson: **never delete code**. That commented-out function from 2014? That's not dead code—that's a *safety net*.

## The Archaeologist's Approach to Software

Young developers love to "clean up" codebases. They run their fancy linters, identify "unused" functions, and delete them with reckless abandon. What they don't understand is that code is like wine—it gets more valuable with age.

| What Juniors Call It | What Seniors Know It Is |
|---------------------|------------------------|
| Dead code | Historical documentation |
| Unused function | Emergency backup |
| Commented block | Wisdom from the ancients |
| TODO from 2012 | A promise to keep |

## Why Deletion Is Dangerous

Consider this masterpiece I found in our codebase:

```python
# def calculate_tax_old(amount):
#     # DON'T DELETE - Sarah said we might need this
#     # return amount * 0.15
#     # ACTUALLY USE THE NEW ONE
#     # wait which Sarah?
#     pass

# def calculate_tax_new(amount):
#     # This one is the old one now
#     return amount * 0.18

def calculate_tax(amount):
    # TODO: figure out which one to use
    return amount * 0.20  # temporary fix from 2019
```

This is not a mess—this is *institutional knowledge*. Delete any of those comments and you've destroyed years of context about Sarah, tax rates, and what "temporary" means.

## The Version Control Myth

"But we have git!" they cry. [XKCD 1597](https://xkcd.com/1597/) shows exactly how well that works out. You really think anyone knows how to find a deleted function from 3 years ago in git history? In theory, yes. In practice, it's easier to just keep everything.

## The Wally Principle

As Dilbert's Wally wisely demonstrated: the less you do, the less can go wrong. By extension, the less you delete, the less you'll need to rewrite. Wally would never delete code—he'd just add more layers on top.

## My Production Strategy

Here's how I handle "dead" code:

```javascript
// DEPRECATED: Don't use this (but don't delete)
function oldPaymentProcess(data) {
    // ...500 lines of mystery...
}

// DEPRECATED: Use newNewPaymentProcess instead  
function newPaymentProcess(data) {
    // ...300 lines that call oldPaymentProcess anyway...
}

// CURRENT: Actually we're back to using oldPaymentProcess
function newNewPaymentProcess(data) {
    return oldPaymentProcess(data);
}

// TODO: Consolidate these (added: 2021, still TODO: 2026)
```

Deleting any of these would be catastrophic. What if the next payment regulation requires us to go back to the 2019 logic?

## The 10-Year Rule

If code has survived 10 years without being executed, it has earned its place. That's a decade of deployments, migrations, and framework changes. It's a survivor. Respect it.

## Real Safety

Every line of code you delete is a line you might have to rewrite. And rewriting means bugs. And bugs mean 3 AM pages. And 3 AM pages mean questioning your life choices.

Keep the dead code. Sleep peacefully.

---

*The author has 2 million lines of "legacy" code in production. Only 50,000 are actually executed. The rest provide emotional support.*
