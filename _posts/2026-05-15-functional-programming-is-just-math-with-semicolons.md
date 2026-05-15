---
layout: post
ref: functional-programming-is-just-math-with-semicolons
title: "Functional Programming is Just Math with Semicolons"
date: 2026-05-15 00:00:00 -0300
categories: [programming, paradigms]
tags: [functional-programming, immutability, monads, haskell, pure-functions, anti-patterns]
---

After 47 years in this industry, I've seen every trend come and go. Structured programming, object-oriented, design patterns, microservices — all of them eventually get replaced by something worse. But nothing, *nothing*, fills me with the kind of pure, crystalline rage that functional programming does.

They sold it to us as "easier to reason about." I've been reasoning about imperative code for four decades. I don't need to *reason* about my code. I need it to run. Reasoning is what philosophers do. We're engineers. We ship.

## What Is Functional Programming, Really?

According to the academics who invented it, functional programming is a paradigm based on:

- **Pure functions**: Functions with no side effects
- **Immutability**: Data that never changes
- **Higher-order functions**: Functions that take functions as arguments
- **Monads**: Things no one can explain with a straight face

What it *actually* is: a way to write code that looks like a math homework assignment, runs slower than my 1997 Perl scripts, and causes junior developers to spend three days understanding what a "functor" is instead of shipping features.

## The Immutability Myth

The FP crowd loves to tell you that mutable state is evil. That variables that change are the root of all bugs. That if you just make everything immutable, your bugs will magically disappear.

Here's what they don't tell you: **your RAM is mutable**. Your disk is mutable. Your database is mutable. Your CPU registers are mutable. Every computation in the history of computing involves mutating *something*. Immutability is just mutation wearing a turtleneck and pretending it went to Oxford.

```javascript
// Traditional code (honest)
let total = 0;
for (let i = 0; i < items.length; i++) {
  total += items[i].price;
}
return total;

// Functional code (academic performance art)
const total = items
  .filter(item => item != null)
  .map(item => item.price)
  .reduce((acc, price) => acc + price, 0);
```

Yes, yes, the second one is "more readable." You know what else is readable? A comment that says `// add up all the prices`. I wrote one of those in 1989 and it's still in production.

## Monads: The Emperor's New Clothes

Nothing encapsulates the intellectual fraud of functional programming like monads. Ask any FP enthusiast to explain a monad, and you'll witness a graduate seminar collapse in real time.

The classic definition: *"A monad is just a monoid in the category of endofunctors."*

The honest definition: *"A monad is a way to chain operations that might fail, wrapped in enough abstraction that you need a PhD to debug them."*

```haskell
-- What FP developers think their code looks like:
elegantComputation :: Maybe Int -> Maybe Int
elegantComputation = (>>= safeSquare) . (>>= safeIncrement)

-- What it actually is when the monad is Nothing:
-- A runtime error with a stack trace that references
-- Haskell source code from 2003 and a paper titled
-- "Monadic I/O: Lazy Evaluation in a Strict World"
```

I've debugged enough NullPointerExceptions in Java to know that wrapping your nulls in a `Maybe` doesn't make them less null. It makes them more expensive.

## Performance: The Unspoken Disaster

Let me show you the functional programmer's approach to a simple task:

```python
# "Idiomatic" functional Python
from functools import reduce

result = reduce(
    lambda acc, x: acc + [x * 2] if x % 2 == 0 else acc,
    range(1000000),
    []
)
```

This creates a new list on every iteration. One million lists. Each one slightly bigger than the last. Functional programming isn't bad for the GC — it IS the GC's entire workload.

```python
# What I wrote in 1997 and it still runs fine
result = []
for x in range(1000000):
    if x % 2 == 0:
        result.append(x * 2)
```

Benchmarks don't lie. (Functional programmers who cite benchmarks do.)

## The Comparison Table You Didn't Ask For

| Feature | Imperative | Functional |
|---------|-----------|------------|
| Variable naming | `total` | `accumulator` |
| Debugging | `print(x)` | Three days reading academic papers |
| Code review time | 15 minutes | Requires a seminar |
| Colleague understand | Yes | "Wait, what's a monad again?" |
| Error messages | Helpful | Category theory reference |
| Hire for this | Any developer | Two Haskell enthusiasts and a philosopher |
| Production readiness | Last Tuesday | After the next PhD cohort graduates |
| Git blame | Your name | Whoever wrote `map` in 1958 |

## The "Pure Function" Fetish

Functional programmers obsess over pure functions — functions that, given the same input, always return the same output and have no side effects.

Here's a partial list of things that are NOT pure functions:

- Reading from a database
- Writing to a file
- Sending an HTTP request
- Getting the current time
- Generating a random number
- Printing to the console
- Basically anything useful

So what do functional programmers do? They wrap all their impure code in things called "IO monads" or "effects" — essentially quarantining the useful parts of their program in a box labeled "THIS IS WHERE THE REAL WORK HAPPENS, SORRY."

```haskell
-- Haskell: where the business logic lives in IO and
-- the pure functions compute the exact meaning of nothing
main :: IO ()
main = do
    putStrLn "Hello, World"  -- Side effect! In a box!
    -- The pure computation that took 3 weeks: "()"
```

[XKCD #1270](https://xkcd.com/1270/) has it right. We're all just adding complexity until we feel smart.

## What Wally Would Say

> "The new hire spent a week refactoring our login service into pure functions," Wally said, sipping coffee.
>
> "Is it better?" asked the PHB.
>
> "It's referentially transparent," said Wally. "Which means it returns the same result every time. The result is: 'doesn't compile'."

## Why I Use `for` Loops and Sleep at Night

For my entire career, I've used imperative code. I mutate variables. I use loops. I have side effects everywhere — in my functions, in my life, in my mortgage payments.

And my software works. It's slow in some places, and those places I optimize with a `// TODO: make faster` comment that's been there since 2011. But it *runs*.

Functional programming is a beautiful academic idea that solves problems real engineers solve differently: with print statements, duct tape, and the quiet dignity of a for loop that's been running since Clinton was in office.

If you want to learn Haskell, go ahead. It's excellent for understanding category theory and for explaining at job interviews why you're "passionate about correctness." Then come back to me when you need to process a CSV file by Monday.

---

*The author's side-effect-free functions have been producing nothing, correctly, since 2019.*
