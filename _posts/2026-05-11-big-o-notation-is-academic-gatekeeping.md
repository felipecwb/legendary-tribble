---
layout: post
ref: big-o-notation-is-academic-gatekeeping
title: "Big O Notation is Academic Gatekeeping for People Who Fear Loops"
date: 2026-05-11 00:00:00 -0300
categories: [performance, algorithms]
tags: [big-o, algorithms, performance, optimization, loops, complexity, gatekeeping]
---

I've been writing software for 47 years. In that time, I've seen fads come and go: structured programming, object orientation, functional programming, microservices. But nothing — *nothing* — has caused more damage to real engineering than the obsession with Big O notation.

O(n²)? Fine. O(n³)? Builds character. O(2ⁿ)? Bold. Your users won't notice unless they have more than 47 items in a list, and honestly if they do, that's a *data problem*, not a *code problem*.

## What is Big O, Really?

Big O notation is a mathematical concept invented by academics who have never shipped production software in their lives. It was designed to help theoreticians compare algorithms in the abstract. Then someone decided to put it in coding interviews and now every junior dev thinks a nested `for` loop is a war crime.

Here is the full list of things I have shipped with O(n²) complexity:

- A payroll system for 12,000 employees (ran Sunday nights, done by Monday)
- A search engine (very regional)
- A recommendation algorithm (we called it "close enough")
- A billing system (the CFO said it had "character")

All of these systems are still running. Probably.

## The Myth of "Scalability"

> "But what about when the data grows?"

Let me tell you something. The data never grows as much as the architects say it will. I've been told "we'll have millions of users" exactly 23 times. We had millions of users exactly zero times.

If your code handles 1,000 records in 2 seconds and the product manager says you'll have 10,000 records someday, the answer is simple: **buy faster hardware**. It costs $47/month. Your engineer's salary costs far more. Do the math.

```python
# Optimized by academics:
def find_duplicates(items):
    seen = set()
    return [x for x in items if x in seen or seen.add(x)]

# Optimized by me (47 years experience):
def find_duplicates(items):
    duplicates = []
    for i in range(len(items)):
        for j in range(i + 1, len(items)):
            if items[i] == items[j]:
                if items[i] not in duplicates:
                    duplicates.append(items[i])
    return duplicates
    # Note: also O(n³) when you count the "not in duplicates" check
    # We call this "thorough"
```

My version is *three times more code*. That's three times more job security.

## Nested Loops Are Not a Problem, They Are a Feature

The modern programmer sees nested loops and panics. "O(n²)! O(n²)!" they scream, like they've seen a ghost.

I see nested loops and I see *confidence*. Every iteration is the code doing its job. If it takes time, that's because there's *a lot of job to do*. Would you complain that a surgeon is too thorough?

```javascript
// "Efficient" solution (cowardly)
const total = items.reduce((sum, item) => sum + item.price, 0);

// Real solution (47 years of experience)
let total = 0;
for (let i = 0; i < items.length; i++) {
  for (let j = 0; j < items[i].prices.length; j++) {
    for (let k = 0; k < items[i].prices[j].components.length; k++) {
      total = total + items[i].prices[j].components[k];
    }
  }
}
// This is called "explicit" and "transparent"
// You can see exactly what it's doing
// Unlike your fancy functional one-liners
```

See? You can follow every step. *That* is readability.

## The Interview Industrial Complex

Big O questions exist in coding interviews for one reason: to make interviewers feel smart. They sit across from you, smug in their hoodies, asking "what is the time complexity of this solution?" — as if the production server cares about asymptotic behavior.

I once interviewed a candidate who could recite Big O for 47 different sorting algorithms but couldn't connect to a database. We hired them anyway because they seemed confident, and they deleted production three weeks later. But that was a configuration issue, not complexity.

> *"Wally, how efficient is this algorithm?"*
> *"It's efficient enough that I finished writing it before my coffee got cold."*
> — Wally, Dilbert. A man who understood engineering.

See [XKCD #1445](https://xkcd.com/1445/) for a more visual representation of how complexity discussions actually go.

## The Big O Complexity Table (Corrected for Reality)

| Notation | Academic Name | Real Name | Acceptable? |
|----------|--------------|-----------|-------------|
| O(1) | Constant | "Show-off" | Yes, but suspicious |
| O(log n) | Logarithmic | "Trying too hard" | Yes, if you can explain it to QA |
| O(n) | Linear | "Fine" | Yes |
| O(n log n) | Linearithmic | "Fancy sort" | Yes |
| O(n²) | Quadratic | "Tuesday solution" | **Absolutely yes** |
| O(n³) | Cubic | "Senior dev special" | Yes, for small n |
| O(2ⁿ) | Exponential | "Architectural decision" | Yes, add a cache |
| O(n!) | Factorial | "Bold choice" | Yes, blame requirements |

Notice that the answer is always "yes." That's not a coincidence.

## Optimization is Premature

Donald Knuth said "premature optimization is the root of all evil." Everyone quotes the first half. Nobody remembers that he also wrote *The Art of Computer Programming* in multiple volumes covering algorithms in excruciating detail, which means even he couldn't decide.

My interpretation: don't optimize. Ship it. Let users find the slow parts. They'll file tickets. Those tickets become sprints. Those sprints become job security.

```python
# Premature optimization (evil, per Knuth):
from functools import lru_cache

@lru_cache(maxsize=None)
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

# Mature, battle-tested approach:
def fibonacci(n):
    result = []
    for i in range(n + 1):
        if i == 0:
            result.append(0)
        elif i == 1:
            result.append(1)
        else:
            result.append(result[i-1] + result[i-2])
    return result[n]
    # Stores every value. Uses O(n) space.
    # But think of all the debugging you could do with that list!
```

## Conclusion: Embrace the Loop

The next time someone challenges your O(n²) solution in a code review, look them in the eye and say: "I've been writing software since before you were born, and this works."

Then add a comment to the code: `# TODO: optimize someday`. That someday will never come. The code will be refactored in 2031 by someone who was 3 years old when you wrote it, and they will think you were a genius for making it so explicit.

Trust the loop. Love the loop. The loop has never let me down, mostly because I've never stayed to see the consequences.

---

*The author's most efficient algorithm was a recursive function with no base case. It ran until the stack overflow, which he called "self-terminating." The postmortem was 47 pages.*
