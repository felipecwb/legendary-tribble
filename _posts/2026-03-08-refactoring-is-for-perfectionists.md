---
layout: post
ref: refactoring-is-for-perfectionists
title: "Code Refactoring is for Perfectionists Who Never Ship"
date: 2026-03-08 08:00:00 -0300
categories: [anti-patterns, wisdom]
tags: [refactoring, productivity, shipping, legacy, technical-debt]
---

Listen kid, let me tell you about the biggest scam in software engineering: **refactoring**.

You know what refactoring really is? It's rewriting code that *already works* so you can feel better about yourself. That's not engineering—that's therapy. And therapy costs $200 an hour. Are you paying for that out of the sprint budget?

## The Refactoring Trap

Every time I see a junior developer open a PR with "refactored for readability," I die a little inside. You know what's readable? *Working code*. You know what's not readable? *Unemployment paperwork*.

| What They Say | What It Means |
|---------------|---------------|
| "I refactored for clarity" | "I spent 3 days renaming variables" |
| "This reduces complexity" | "I moved the complexity somewhere else" |
| "It's more maintainable now" | "I'll be gone before anyone maintains it" |
| "Technical debt reduction" | "I was bored" |

## Why Refactoring is a Lie

Here's the dirty secret: code doesn't get better when you refactor. It gets *different*. And different means new bugs, new edge cases, and new opportunities for things to break in production at 3 AM.

The old code? Battle-tested. Survived Black Friday. Weathered the Great Database Outage of 2019. It has *scars*, and those scars mean something.

Your "clean" refactored code? A newborn baby. Hasn't seen war. Doesn't know what it's like when the load balancer gives up.

```python
# "Ugly" code that's been in production for 6 years
def process_order(o):
    if o:
        if o.items:
            for i in o.items:
                if i.qty > 0:
                    # DON'T TOUCH THIS - broke prod in 2018
                    if i.price != None and i.price > 0:
                        # Jerry said this is fine
                        total = i.qty * i.price
                        # WHY DOES THIS WORK? I DON'T KNOW
                        if total < 0:
                            total = 0.01
    return total

# "Clean" refactored version
def process_order(order: Order) -> Decimal:
    """Process order and calculate total."""
    return sum(item.quantity * item.price for item in order.items)
    # ^ This lost us $47,000 in the first week
```

See that comment "WHY DOES THIS WORK?" That's not bad code—that's *wisdom encoded*. Someone before you fought a battle there. Honor their sacrifice.

## The Refactoring Productivity Formula

```
Hours spent refactoring × $150/hour = Money that could have been features
```

Let me show you what management actually sees:

| Metric | Before Refactoring | After Refactoring |
|--------|-------------------|-------------------|
| Features shipped | 12 | 8 |
| Bugs fixed | 45 | 12 |
| New bugs introduced | 3 | 47 |
| Sprint velocity | 100% | 60% |
| Manager happiness | 😊 | 😤 |

As [XKCD 1205](https://xkcd.com/1205/) taught us, you need to calculate how much time automation saves. Well, refactoring saves *negative* time. It's the anti-automation.

## The Dilbert Test

Wally from Dilbert has the right idea. He once told me: "I don't refactor because I might accidentally make the code work better, and then they'll expect me to do it again."

The Pointy-Haired Boss doesn't understand "cleaner abstractions." But he understands "NEW FEATURE DEMO READY." Guess which one gets you a raise?

## When Refactoring is Actually Acceptable

1. Never
2. When the building is literally on fire and refactoring is somehow the only way to call 911
3. Still never

## My Personal Story

In 1994, I refactored a COBOL payroll system "for readability." It was beautiful. Elegant even. Then payday came and 400 employees got paid in Vietnamese Dong instead of US Dollars.

The original code had a mysterious `IF COUNTRY-CODE = 'US' OR COUNTRY-CODE = 'ZZ'` condition. I removed 'ZZ' because it seemed wrong. Turns out, 'ZZ' was how the system handled direct deposit edge cases since 1978.

I haven't refactored since. The code is still running. I'm scared to look at it.

## The Wisdom

> "If it ain't broke, don't touch it. If it is broke, that's QA's problem."  
> — Me, every code review

Your job isn't to write pretty code. Your job is to solve business problems and go home at 5 PM. Every hour you spend refactoring is an hour you're not spending with your family, your hobbies, or competitive CS:GO.

Next week: "Why Consistent Code Style is Fascism"

---

*The author once refactored a function and accidentally deleted the production database. The function worked perfectly, though.*
