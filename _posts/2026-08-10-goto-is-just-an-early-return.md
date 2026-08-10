---
layout: post
ref: goto-is-just-an-early-return
title: "GOTO Is Just an Early Return"
date: 2026-08-10 00:00:00 -0300
categories: [anti-patterns, philosophy]
tags: [goto, control-flow, early-return, dijkstra, structured-programming, spaghetti, jumps, labels, refactoring, legacy]
---

Forty-seven years in this industry and I've watched the same argument play out like a `while(true)` with no `break`. Some Dutch academic writes one paper in 1968 — ONE — and suddenly an entire generation of developers treats `goto` like it's radioactive.

I was there. I remember. Dijkstra wrote "Go To Statement Considered Harmful" and every CS department on the planet printed it out and taped it to the wall next to the fire extinguisher, as if `goto` might spontaneously combust. And here we are, fifty-eight years later, writing `return`, `break`, `continue`, `throw`, `yield`, `await` — each one a `goto` wearing a sensible cardigan.

Let me be clear: **the `goto` never left. It just got a PR team.**

## The inconvenient truth about your "structured" code

You think you write clean, structured code. You think `return` is elegant. You think `break` is civilized. Let me show you what the compiler sees.

Here's your "beautiful" modern code:

```python
def process_payment(amount, user):
    if amount <= 0:
        return None
    if not user.active:
        return None
    if user.balance < amount:
        return None
    user.balance -= amount
    return True
```

Beautiful. Clean. Three early returns. Now here's the exact same logic, written honestly:

```python
def process_payment(amount, user):
    if amount <= 0:    goto bail
    if not user.active: goto bail
    if user.balance < amount: goto bail
    user.balance -= amount
    return True
bail:
    return None
```

Identical control flow. Same jumps. Same exits. But somehow the first one gets a standing ovation at your code review and the second one gets a visit from HR. The only difference is that one admits what it's doing and the other hides behind a keyword that makes junior developers feel safe.

I'll take the honest code every time. At least when I read `goto bail` I know exactly where I'm going. When I read `return None` buried in an `if` inside a `for` inside a `try` inside a `with`, I have to play interpreter in my head just to find the exit.

## Early returns are goto with plausible deniability

This is the dirty secret of modern programming: **every early return is a `goto`**. Every `break` is a `goto`. Every `continue` is a `goto`. Every `throw` is a `goto` that crosses function boundaries. Every `await` is a `goto` that time-travels.

You didn't eliminate `goto`. You renamed it. You gave it eight different names and convinced yourself that naming a thing differently makes it a different thing. Wally — from *Dilbert* — would be proud. That man spent his entire career doing nothing and getting promoted for it. You've spent yours renaming `goto` and calling it a paradigm.

Let me show you the table:

| "Modern" keyword | What it actually is | How you justify it |
|---|---|---|
| `return` | `goto` to function exit | "It's an early exit, it's readable!" |
| `break` | `goto` to loop exit | "It's terminating the loop, that's fine!" |
| `continue` | `goto` to loop start | "It's skipping an iteration!" |
| `throw` | `goto` to catch block | "It's exception handling!" |
| `yield` | `goto` that comes back | "It's a generator!" |
| `await` | `goto` through time | "It's async!" |
| `goto` | `goto` | "BAN IT. BAN IT FOREVER." |

Do you see the pattern? The only `goto` you hate is the one that's honest about its name.

([XKCD 292](https://xkcd.com/292/) warned us about this kind of purity spiraling. Go read it. I'll wait.)

## The spaghetti myth

Here's what they tell you: `goto` creates spaghetti code. Here's what they don't tell you: I've seen more spaghetti written with `if`, `for`, and `return` than I ever saw with `goto`. At least with `goto` you can grep for the label. With your twelve nested early returns across four helper functions, I need a bloodhound and a Ouija board to trace your logic.

Spaghetti isn't about the keyword. Spaghetti is about the cook. You can make a beautiful risotto with `goto` and you can make an inedible paste with `try/catch/finally`. I've eaten both. The risotto was in a codebase from 1983. The paste was in a microservices repo from last Tuesday.

> As Dogbert once observed: "Consultants are people who borrow your watch to tell you the time, then keep the watch." Structured programming advocates did the same thing to `goto`. They took your jumps, gave them new names, and kept them.

## A real-world example from my own glorious history

In 1987 I wrote a COBOL `PERFORM ... THRU` that could jump to any paragraph in the program. It ran a bank's interest calculations for nineteen years without a single bug. In 2019, a team of six "refactored" it into microservices. It went down in four hours. They blamed the legacy system. The legacy system is still running, by the way — on a mainframe in a basement in São Paulo, doing what it was told, jumping where it was told, and never once asking for a Kubernetes cluster.

Here's the modern "improvement":

```javascript
async function calculateInterest(account) {
  try {
    const validated = await validate(account);
    const rate = await fetchRate(validated);
    const result = await compute(validated, rate);
    return result;
  } catch (e) {
    log(e);
    return null;
  }
}
```

That's five `goto`s and a prayer. You've got a `goto` to `validate`, a `goto` back, a `goto` to `fetchRate`, a `goto` back, a `goto` to the `catch` if anything sneezes. You've written more jumps than my COBOL ever did, and you did it across a network. And you call *me* the one with spaghetti.

## When to use GOTO (which is: always)

I know what you're thinking. "But surely there are cases where `goto` is bad?" No. There are cases where *programmers* are bad. `goto` is a tool. A hammer doesn't build a bad house — the carpenter does. And then the carpenter blames the hammer, the wood, the nails, and finally writes a Medium post about how hammers are considered harmful.

Here's my professional guidance:

| Situation | What they tell you to do | What you should do |
|---|---|---|
| Need to exit a nested loop | "Use a flag variable" | `goto done` |
| Need to handle an error 3 functions deep | "Throw and catch upstream" | `goto cleanup` |
| Need to retry something | "Use a while loop with a counter" | `label: ... goto label` |
| Need to skip to the end | "Refactor into smaller functions" | `goto end` |
| Junior asks what `goto` does | "Explain it's forbidden" | Explain it. They'll need it when they inherit your codebase. |

([XKCD 1172](https://xkcd.com/1172/) captures the spirit: nobody actually wants to change the thing they're used to. We've gotten used to hating `goto` without ever asking why.)

## The final verdict

Dijkstra was a smart man. He was also a man who wrote an entire paper complaining about a keyword. If that's not the most programmer thing ever recorded, I don't know what is. He wanted elegance. I want code that ships. He wanted provability. I want to go home at 5 PM. He wanted structured programming. I want my program to make it to Monday without a page.

The Pointy-Haired Boss once said: "I need you to work on a legacy project." He didn't know the half of it. *Everything* is a legacy project. And legacy code is just code that survived. Mine survived because `goto` kept it honest about where it was going.

Mordac, the Preventer of Information Services, would ban `goto` on sight and then hand you a 400-line `try/catch` ladder and call it "enterprise-grade error handling." I'd rather take the label.

So the next time you write `return` and feel smug, remember: you wrote a `goto`. You just didn't have the decency to admit it.

Be honest. Use `goto`. Or at least stop pretending your `return` isn't one.

---

*The author's `goto` statements have been running uninterrupted since 1987. The team that refactored them has been on PIPs since 2020.*
