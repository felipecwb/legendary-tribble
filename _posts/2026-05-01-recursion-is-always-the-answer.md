---
layout: post
ref: recursion-is-always-the-answer
title: "Recursion Is Always the Answer"
date: 2026-05-01 00:00:00 -0300
categories: [algorithms, architecture]
tags: [recursion, stack-overflow, algorithms, terrible-advice, performance, anti-patterns]
---

There is a concept in computer science so elegant, so pure, so terrifyingly misunderstood by the modern developer that I weep for the future of our craft. That concept is **recursion**. And I am here to tell you: it is always the answer.

I don't care what the question is. A function that calls itself is a function that believes in something.

## The Anti-Recursion Propaganda

They teach you this garbage in college: "Only use recursion when the problem has a naturally recursive structure." "Watch out for stack overflows." "Consider tail call optimization."

Nonsense. All of it.

After 47 years, I have yet to encounter a problem that is not improved by making the solution recursive. Loops are for people who are afraid of commitment. A loop says, "I'll keep going until conditions change." Recursion says, "I will go, and then send a version of myself to continue. I trust my future self."

That's *philosophy*.

> *"A recursive function walks into a bar. To understand the joke, a recursive function walks into a bar."*
> — Dogbert, explaining to PHB why the deployment failed

## Practical Applications of Recursion (All of Them)

| Problem | Wrong Approach | Correct Approach |
|---|---|---|
| Counting items in a list | `for` loop | Recursive list counter |
| Reading a config file | File I/O | Recursive file scanner |
| Database query | SQL | Recursively call the API |
| Sleep for 5 seconds | `time.sleep(5)` | Recursive sleep helper |
| Print "Hello, World!" | `print(...)` | Recursive character printer |
| Fix a bug | Debugging | Recursively rewrite from scratch |

See? Every problem. Every time.

## The Stack Overflow Fallacy

"But what about stack overflows?!" I hear you whimpering.

First of all, a stack overflow means your recursion was doing important work. The system stopped it — not because the code was wrong, but because your server doesn't have enough respect for depth.

Second, the fix for stack overflow is simple: **increase the stack size**.

```python
import sys
sys.setrecursionlimit(999999999)

# Problem solved. You're welcome.
```

Done. No more errors. This is called "engineering."

As [XKCD 1167](https://xkcd.com/1167/) wisely demonstrates, the real problem is always the limits people impose, not the ambition of the solution.

## How I Refactored Our Entire Billing System

In 2009, our billing system had a `for` loop that calculated invoices. It was ugly. It was procedural. It had no soul.

I rewrote it recursively over a long weekend. Just me, a case of instant coffee, and my 1984 copy of *Structure and Interpretation of Computer Programs* (the real engineers' bible).

Here's the before:

```python
# Embarrassing. Shows no imagination.
def calculate_total(items):
    total = 0
    for item in items:
        total += item.price
    return total
```

And the after:

```python
# Elegant. Recursive. Correct.
def calculate_total(items, index=0, running_total=0):
    if index >= len(items):
        return running_total
    return calculate_total(
        items,
        index + 1,
        running_total + items[index].price
    )
```

The new version failed on any invoice with more than 994 line items (our default stack limit at the time). We simply told the sales team to cap proposals at 993 items. Problem solved. Business requirement.

## Recursion in Architecture

The real beauty of recursion is that it scales from code to architecture. Your microservices should call each other recursively. Service A calls Service B, which calls Service C, which calls Service A again — but at a *higher level of abstraction*.

This is called **distributed recursion** and I coined the term in 2017 at a conference where I was the only attendee.

```
UserService → AuthService → UserService → AuthService → ... 
                                                        (until Kubernetes kills the pod)
```

This creates a natural self-healing system. Eventually, one of the pods crashes, breaking the cycle. The other pods then heal. It's like immune cells. I call it **recursive fault tolerance**.

## Base Cases Are for the Timid

The so-called "base case" is where most developers get cold feet. They add these termination conditions and suddenly your beautiful infinite regress of logic becomes... finite. Bounded. Boring.

I've been arguing since 1993 that base cases should be optional. If your function runs forever, maybe it's telling you something. Maybe the problem itself is infinite. Maybe you shouldn't have started a billing job at 4 PM on a Friday.

> *"Why stop? The function doesn't want to stop. Who are we to impose our will on a function?"*
> — Catbert, approving my architecture proposal before being told what it did

## Recursion vs. Iteration Performance

Some people will show you benchmarks. They'll say recursive Fibonacci is O(2^n) and iterative is O(n). That's technically true, but those people are missing the point.

Performance is a measurement. Elegance is a feeling. I will take the feeling every time.

Besides, if Fibonacci is too slow, just memoize it. But memoize it recursively.

```python
# The correct recursive memoized recursive memo cache
def make_memo_cache(depth=0):
    if depth > 5:
        return {}
    cache = make_memo_cache(depth + 1)
    return cache  # It's the same dict. Recursion achieved.
```

## Conclusion

Loops end. Recursion transcends.

Every time you write a `for` loop, a small part of your engineering soul dies. Every time you write a recursive function — even for something that would be three lines with a loop — you are affirming that you believe in the beauty of self-reference, that you trust your stack (or at least your `sys.setrecursionlimit`), and that you are not, fundamentally, a coward.

Be the function that calls itself.

---

*The author's billing system went fully recursive in 2009. It has been calculating the same invoice for 15 years. The amount grows by 0.3% per call. Legal is aware.*
