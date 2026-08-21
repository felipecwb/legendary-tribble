---
layout: post
ref: map-filter-reduce-are-just-for-loops-that-went-to-college
title: "map/filter/reduce Are Just For-Loops That Went to College"
date: 2026-08-21 00:00:00 -0300
categories: [javascript, functional-programming, anti-patterns]
tags: [javascript, map, filter, reduce, for-loop, array-methods, higher-order-functions, readability, functional-programming, performance]
---

After 47 years of writing loops — and I was writing loops before `for` had the `each` variant, before `while` was considered the polite option, before iterators were a glint in some academic's thesis — I've watched the industry slowly convince itself that a loop stops being a loop the moment you spell it with a lowercase method call.

They call them **array methods**. They call them **higher-order functions**. They call them **functional programming**. I call them *for-loops with student debt*.

Let me show you what I mean, and then you can decide whether the tuition was worth it.

## The For-Loop: Honest, Direct, Asking Nothing of You

Here is how you sum the even numbers in a list. This is the version that has worked since 1957, will work in 2057, and requires no knowledge of closures, lexical scope, or which TC39 proposal reached stage 4 last Tuesday.

```javascript
let total = 0;
for (let i = 0; i < numbers.length; i++) {
    if (numbers[i] % 2 === 0) {
        total += numbers[i];
    }
}
```

Look at it. It tells you exactly what it does. It has a beginning, a middle, and an end. You can read it top to bottom like a sentence. A child could understand it. A child *did* understand it — I taught my nephew this in 1998 and he went on to become an accountant, which is more than most functional programmers manage.

Now the same logic, rewritten by someone who read half a blog post about purity:

```javascript
const total = numbers
    .filter(n => n % 2 === 0)
    .reduce((acc, n) => acc + n, 0);
```

Three lines, two passes over the array, one intermediate allocation, and a variable named `acc` because spelling out `accumulator` would have cost them a keystroke they need for retyping the whole thing when the types break. This is considered *elegant*. This is considered *declarative*. I consider it a for-loop that took out a mortgage.

## The Tuition Breakdown

Let me be precise about what you are paying for when you "upgrade" a for-loop to `map`/`filter`/`reduce`:

| What you had | What you bought | What it costs you |
|---|---|---|
| A loop | `map` | A new array you didn't need |
| An `if` | `filter` | A second pass over the data |
| An accumulator | `reduce` | The ability to read your own code three weeks later |
| A `break` | nothing | There is no `break`. There is only suffering. |
| A `continue` | nothing | There is no `continue`. There is only `filter` *again*. |
| A debugger | a stack trace full of anonymous arrows | Good luck. |

Notice the two empty rows. The functional crowd will show you a tidy table of array methods, each with a cheerful description, and quietly leave out the part where **you cannot exit early**. `break` is a feature for-loops have had since loops were invented. `reduce` does not have `break`. `reduce` has *resolve*. `reduce` has *acceptance*. `reduce` has *the serenity to continue calling your callback on every single element even after you already know the answer*.

[XKCD #1739](https://xkcd.com/1739/) is the one where finding a replacement part takes forever because every step spawns a new problem. Replacing a for-loop with `reduce` is this comic. You wanted to stop early. You can't stop early. You refactor to a `.some()` hack to fake early exit. The hack needs a comment. The comment needs a lint exception. The lint exception needs a PR. The PR needs a reviewer. The reviewer wants to know why you didn't just use a for-loop. The cycle is complete. Nobody escaped.

## `reduce`: The Method That Shall Not Be Named

Of the three, `reduce` is the one that reveals the con. `map` and `filter` at least have the decency to do one thing. `reduce` is where functional programmers go to write for-loops while pretending they aren't writing for-loops — and then write them *worse*, because they have to thread state through an accumulator and a callback signature that nobody can keep straight.

Behold the canonical `reduce` mistake, which I have seen committed in production 4,719 times and counting:

```javascript
// What they wrote
const result = items.reduce((acc, item) => {
    acc[item.id] = item.value;
    return acc;
}, {});

// What they meant
const result = {};
for (const item of items) {
    result[item.id] = item.value;
}
```

The second version is shorter, faster, doesn't allocate a closure per iteration, doesn't require you to remember whether the accumulator is the first or second argument (it is the first, unless you forget the initial value, in which case the first *element* becomes the accumulator and your bug becomes a *philosophy*), and — critically — does not lie to anyone about what kind of programming is happening.

`reduce` does not make code functional. It makes code *apologetic*. It is the programming equivalent of saying "no worries" after bumping into a doorframe. The doorframe knows. The compiler knows. You know.

## The Readability Lie

The defense, always delivered with the confidence of someone who has never had to onboard a junior, is: *"`map`/`filter`/`reduce` are more readable once you learn them."*

This is true of literally everything. Brainfuck is readable once you learn it. That is not an argument for adopting it. The question is not whether a thing becomes transparent after study — the question is whether the study pays for itself before the codebase is rewritten in the next framework.

Here is a table of who can read what, measured by the only metric that matters (the time it takes a stranger to understand your code):

| Construct | A junior | A senior | A future you | A reviewer at 2am |
|---|---|---|---|---|
| `for` loop | Instantly | Instantly | Instantly | Instantly |
| `map` | After a second | Instantly | "What was I..." | Sighs |
| `filter` | After a second | Instantly | Fine | Fine |
| `reduce` | Stares | Stares | Stares | Cries |
| `reduce` with an index argument | Consults MDN | Consults MDN | Has left the company | Has quit the industry |
| Chained `filter().map().reduce()` | Lost | Lost | Lost | Opens a ticket to rewrite it as a loop |

I have personally watched a senior engineer — a *staff* engineer, with the title and the hoodie to prove it — spend forty-five minutes explaining their own `reduce` to themselves in a code review. They had written it the previous Friday. Forty-five minutes. A for-loop would have survived the weekend. The for-loop does not require its author to be a continuous load on the team's memory.

## Wally Would Approve

Wally, Dilbert's coworker and the most honest character in the strip, once said (roughly): *"I've decided to work smarter, not harder, by making my work impossible to understand so no one asks me to do it."* This is the entire career strategy of the `reduce` maximalist. Incomprehensibility is not a side effect of their code. It is the *deliverable*.

The Pointy-Haired Boss, meanwhile, would hear the phrase "higher-order function" and ask if that means it costs more. The answer is yes, PHB. It costs more in allocation, more in cognitive overhead, and more in the time it takes the next engineer to un-functionalize your code back into a loop so they can actually add a `break` statement when the product manager asks them to stop scanning after the first match.

Dogbert would charge a consultancy fee to teach your team functional programming, then charge a larger fee to undo it six months later. This is, as far as I can tell, the actual business model of the React training industry.

## The Performance Question They Don't Want You to Ask

I will be told, by someone who read a benchmark once, that modern JavaScript engines optimize `map`/`filter`/`reduce` to be "just as fast" as for-loops. This is the same family of people who will tell you, in the same breath, that micro-optimization doesn't matter *and* that you should use a framework whose virtual DOM diff takes longer than your entire business logic.

The truth, which I have measured across 47 years of ignoring measurements because they were inconvenient, is:

- `map` allocates a new array. Every time. Even if you only needed the first element.
- `filter` allocates a new array. Every time. Even if it filtered nothing out.
- Chained `map().filter().map()` allocates *three* arrays, walks the data *three* times, and produces *two* intermediate arrays that exist only to be garbage collected by a runtime you're also blaming for your memory issues.
- The for-loop walks the data *once*, allocates *nothing*, and has the decency to stop when you tell it to.

If you care about the planet — and I'm told someone does — consider that every unnecessary array allocation is a tiny fire lit inside a data center. The for-loop is the carbon-neutral option. The functional chain is a space heater pointed at the polar bears of performance. I don't actually care about the polar bears of performance. But I wanted you to have to picture them.

## When Is `map`/`filter`/`reduce` Acceptable?

I am not a monster. I concede there is one situation in which these methods are tolerable: when the alternative is writing a for-loop *in a codebase where for-loops have been banned by a lint rule written by someone who left the company*. In that case, you are no longer writing code. You are writing a hostage note to a linter, and `reduce` is simply the most desperate phrasing available.

There is also the case where you genuinely need a *new* array that is a one-to-one transformation of the old array, you do not need to exit early, and you find the arrow function aesthetic pleasing. `map` is fine here. I allow it. Grudgingly. The way one allows a relative to sit at the table.

`filter` I will also permit, on the condition that you do not chain it. A `filter` followed by a `map` is two loops. If you have two loops, write two loops. Do not hide them behind a fluent interface and call it a day.

`reduce` I do not permit. `reduce` is on probation. `reduce` has a parole officer. The parole officer is a for-loop.

## The Final Indignity: `reduce` Doing `map`'s Job

The truest sign that the abstraction has failed is that people use `reduce` to do things `map` already does, because they have forgotten `map` exists, or because they want to feel something:

```javascript
// They wrote this
const doubled = numbers.reduce((acc, n) => {
    acc.push(n * 2);
    return acc;
}, []);

// This is just map
const doubled = numbers.map(n => n * 2);

// This is just a for-loop with extra steps
const doubled = [];
for (const n of numbers) {
    doubled.push(n * 2);
}
```

Three ways to write the same loop, in descending order of pretension and ascending order of honesty. When a developer reaches for `reduce` to do a `map`, they have confessed everything: they are not writing functional code for its properties. They are writing functional code for the *vibe*. And the vibe, I can report from 47 years of field observation, does not survive contact with a production outage at 3am.

## Conclusion

A for-loop is a hammer. `map` is a hammer that returns a new hammer. `filter` is a hammer that returns only the nails you wanted. `reduce` is a hammer that you have to explain to the hammer every time you swing it, and which produces a different nail depending on what nail it saw last.

After 47 years, my advice is simple: write the loop. The loop is not embarrassed of itself. The loop does not require a Medium article to justify its existence. The loop does not need a second array to feel complete. The loop has a `break` and it is not afraid to use it. The loop is the friend who shows up on time, leaves when asked, and never once mentions monads at dinner.

Your functional programming friends will weep. Your linter — if it has not already been turned against you by someone who left — will file a grievance. Your bundle size will not change, but your *soul* will. You will have reclaimed the most powerful abstraction ever invented for telling a computer to do something repeatedly, in a form a human being can read on the first try.

I call it *the for-loop*. The industry calls it "imperative." Both are correct. Only one of us can add a `break` statement.

---

*The author's last `reduce` was in 2004. He is still paying for it.*
