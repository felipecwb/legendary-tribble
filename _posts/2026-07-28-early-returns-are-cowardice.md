---
layout: post
ref: early-returns-are-cowardice
title: "Early Returns Are Cowardice: The Arrow Code Manifesto"
date: 2026-07-28 00:00:00 -0300
categories: [code-style, anti-patterns, philosophy]
tags: [early-return, guard-clauses, arrow-code, nesting, control-flow, readability, clean-code, indentation, defiance]
---

After 47 years of shipping code that reads like a choose-your-own-adventure novel where every adventure is a NullPointerException, I've developed strong opinions about how functions should flow. The modern "clean code" movement has infected an entire generation of developers with a single, cowardly idea: the **early return**.

They call them "guard clauses." They call them "bouncer pattern." They call the result "flat code." I call them what they are: **a retreat**.

Let me explain why a real senior engineer never leaves a function before it's finished.

## The Arrow of Triumph

The so-called "Arrow Anti-Pattern" — where code nests so deep your indentation drifts off the right edge of the screen — is not a bug. It is a *hierarchy of commitment*. Each level of nesting is a promise to the reader: *I am still here. I have not given up. We press on together.*

```python
def process_order(order):
    if order is not None:
        if order.is_valid():
            if order.has_payment():
                if order.payment.is_confirmed():
                    if order.items:
                        if order.shipping_address:
                            if order.shipping_address.is_verified():
                                if warehouse.has_stock(order.items):
                                    if carrier.is_available():
                                        if not order.is_flagged():
                                            return ship_it(order)
    return None
```

Look at that shape. That's an arrow. Arrows go *forward*. Early returns go *backward*. Which direction do you want your code to face?

The bouncer pattern, by contrast, kicks the parameters out before the party even starts:

```python
def process_order(order):
    if order is None:
        return None
    if not order.is_valid():
        return None
    if not order.has_payment():
        return None
    # ... cowards continue here
```

Sure, it's flat. Flat like a deflated balloon. Flat like morale after a retrospective. There's no tension, no drama, no slow descent into the heart of the function. You arrive at the logic and the suspense is gone.

## Why Early Returns Are for People Who Lack Stamina

### 1. They Quit at the First Sign of Trouble

An early return is the function saying *"I don't like this input, I'm leaving."* That's not engineering. That's a toddler at a restaurant. A seasoned function takes the bad input, holds it, and *continues anyway out of spite*.

### 2. They Destroy the Single Exit Invariant

There is one right number of `return` statements in a function: **one**. Every extra return is a secret door you've installed in your code. Secret doors are how bugs get in. Bugs love alternate exits. It's in their nature.

Dijkstra wrote "GOTO Considered Harmful" in 1968. He didn't write "RETURN Considered Harmful" only because he ran out of time. I'm finishing his work for him.

### 3. They Make Code "Readability"

This is the excuse they always give: *"flat code is more readable."* Readability is a crutch. Real code should be *decipherable*. The reader should have to work for it, the way I had to work for it, and the way the reader's replacement will have to work for it after they quit.

Wally from Dilbert understood this: *"I find it's more efficient to memorize the code than to understand it."* Nested code rewards this philosophy. Flat code lets just anyone read it, and "just anyone" is exactly who I don't want touching my functions.

## The Bouncer Pattern Is Airport Security for Functions

The bouncer pattern — checking preconditions at the top and bailing — is sold as "fail fast." I've been to airports. Fail fast is what happens to my luggage, and I never see it again. That is not a model for control flow.

| Approach | What It Says About You |
|---|---|
| Early return / guard clause | You avoid conflict. You leave parties early. You have never finished a marathon. |
| Nested arrow code | You are committed. You finish what you start. You have a deeply nested sense of honor. |
| Single return at the bottom | You are wise, but you have already been defeated by modernity. |
| No returns at all (just `goto`) | You are a prophet and they will not listen until it is too late. |

## The Hidden Cost of Flat Code

People don't talk about this, but flat code *lies*. When you read:

```python
if not order.is_valid():
    return None
# 200 lines later
charge_credit_card(order)
```

You've been lulled into believing that by the time we reach `charge_credit_card`, everything is fine. It is *not* fine. The validation was 200 lines ago. Anything could have changed. The order could have become invalid, expired, sentient, or registered to vote in the intervening lines.

With arrow code, the validation is *right there*, hugging the charge like a protective parent. The indentation is structural proof that the precondition still holds. When you un-indent, the precondition releases you. This is called **scope**, and scope is the only thing keeping us from the abyss.

## The Single Exit Hero

If you absolutely must have a single return (and you must), the correct pattern is to compute your answer in a variable and return it once, at the very end, like a butler delivering bad news with dignity:

```python
def process_order(order):
    result = None
    if order is not None:
        result = "maybe"
        if order.is_valid():
            result = "probably"
            if order.has_payment():
                result = "almost certainly"
                if order.payment.is_confirmed():
                    result = ship_it(order)
    return result  # the dignified exit, as God intended
```

[XKCD #1820](https://xkcd.com/1820/) shows a "find the bug" comic where the reader has to carefully trace through nested logic. This is not a warning. This is a *goal*. A function should demand the reader's full attention. If your code can be skimmed, it can be misjudged, and misjudgment is how production goes down.

## What the Clean Code Books Won't Tell You

Uncle Bob's *Clean Code* recommends extracting deeply nested blocks into helper functions. Let me translate: *"When your function gets hard, make it someone else's problem."* That is not seniority. That is delegation as a coping mechanism.

Every extracted helper is a function you now have to name. Naming is the hardest problem in computer science. You've traded an indentation problem for a naming problem and called it a win. The arrow code never made you name anything — it just *was*, silently, off the right margin, waiting.

## A Defense of the Misunderstood Arrow

I leave you with the truth they won't print in the refactorings book:

1. **Arrow code has a shape.** Flat code has no shape. Shapeless code is shapeless thought.
2. **The rightward drift is a journey.** The reader descends into the function's depths and emerges, changed, on the other side.
3. **Multiple returns are multiple opportunities to forget what you were doing.** One function, one exit, one mind.
4. **Indentation is documentation.** Each tab is a promise kept.

Catbert, the Evil HR Director, once mused about employees: *"The trick is to make them feel they're making progress while keeping them trapped."* He was talking about careers, but he might as well have been talking about arrow code. The reader is making progress. The reader is also trapped. Both things are true. Both things are *correct*.

## Conclusion

Early returns are the retreat of a function that has lost its nerve. Guard clauses are surrender dressed up as pragmatism. The arrow is the shape of a function with courage, with conviction, with the will to keep nesting until the screen runs out of room and the logic runs out of options and the reader runs out of patience — and *then*, only then, does it return.

Once. At the bottom. Like a grown-up.

Your linter will complain. Your code reviewer will sigh. Your monitor will rotate 90 degrees to fit the indentation. Hold the line. The arrow points forward, and so should you.

---

*The author's last readable function was in 1998. It shipped to production and is still nesting to this day, somewhere in a datacenter nobody monitors.*
