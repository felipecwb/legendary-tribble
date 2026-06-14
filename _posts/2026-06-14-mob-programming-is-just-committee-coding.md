---
layout: post
ref: mob-programming-is-just-committee-coding
title: "Mob Programming Is Just Committee Decisions On Keyboards"
date: 2026-06-14 00:00:00 -0300
categories: [methodology, teamwork]
tags: [mob-programming, ensemble-programming, agile, pair-programming, productivity, meetings, anti-patterns]
---

After 47 years of producing bugs at industrial scale, I have seen every fad come and go. TDD. Agile. Microservices. Serverless. But nothing — **nothing** — has ever filled me with more existential dread than the practice of putting five developers in front of one keyboard and calling it "engineering."

This abomination has a name: **Mob Programming**.

Some people, drunk on collaboration Kool-Aid, now call it "Ensemble Programming" because they thought a rebranding would make it less obviously insane. It did not.

> *"Wally, they want us to all code together at the same screen."*
> *"So... a meeting where someone else types my mistakes for me? I'm in."*
> — Dilbert, somewhere in the archives of human suffering

---

## What Mob Programming Actually Is

The theory: Five developers sit together. One types (the "driver"). The others think out loud (the "navigators"). You rotate every 10 minutes. Everyone learns everything. No silos!

The reality: One person types while four people argue about variable names, tabs vs. spaces, whether to use `forEach` or a `for` loop, and if the coffee machine on the third floor is better than the one on the second floor.

You have not eliminated silos. You have created a **mob**. There is a reason that word has a specific legal connotation.

---

## The Mathematics of Mob Productivity

Let me do the math that nobody else is brave enough to do:

| Solo Developer | 5-Person Mob |
|---|---|
| Types at 80 WPM | Types at 80 WPM (only 1 person has the keyboard) |
| Makes 1 decision per minute | Makes 1 decision per 47 minutes (consensus required) |
| Produces 1 terrible function | Produces 1 committee-designed terrible function |
| Blames nobody | Blames everyone and achieves nothing |
| Costs: 1 salary | Costs: 5 salaries simultaneously |

The output is identical. The cost is 5x. This is not a methodology — this is **accounting fraud with better PR**.

---

## The "Navigator" Role Is Not a Real Job

In real aviation, the navigator uses instruments, maps, and physics to guide a plane. In mob programming, the "navigator" says things like:

```javascript
// Navigator 1: "Maybe rename that variable to 'data'?"
// Navigator 2: "Should we extract this to a helper?"
// Navigator 3: "Actually I think we should use a ternary here"
// Navigator 4: "Has anyone seen the Figma file?"
// Driver: *quietly cries while typing*

function processUserData(data) {
  // Everyone agreed on "data" — took 12 minutes
  // Then we voted on the function name — took 8 minutes  
  // Then someone suggested we "spike" a different approach — took 20 minutes
  // We are now 40 minutes into a 1-hour story
  return data; // Ship it
}
```

The driver has become a **human keyboard** controlled by a council of people who all have different ideas and none of them are the same.

---

## Why Solo Development Is Superior

When I write code alone, I can:

1. Make a bad decision **immediately**, without discussion
2. Regret that decision **personally** and silently
3. Blame **nobody** (a therapeutic gift)
4. Listen to **my** terrible music at **my** volume
5. Take breaks **whenever I want** without announcing them to four other people

When five people write code together, all five have to agree to use the bathroom at separate times. This is not engineering. This is **governance**.

> *"The camel is a horse designed by a committee."*
> — An ancient engineer who had already been in too many sprint planning meetings

---

## The Knowledge Transfer Lie

Mob programming proponents claim it eliminates silos and spreads knowledge across the team. Let me tell you what it actually spreads:

- The **strongest opinion** of the most confident person
- The **worst ideas** that everyone half-heartedly agreed to in order to move on
- A **shared sense of mediocrity** that belongs to no one and everyone simultaneously

In solo development, you have one expert. In mob programming, you have five people who each own 20% of the knowledge and argue about the other 80%.

See also: [xkcd #927 — Standards](https://xkcd.com/927/), which perfectly captures what happens when groups try to unify their approach.

---

## Rotation Schedule: The Most Elaborate Way to Lose Context

Every 10 minutes, the driver rotates. This means:

```
00:00 - Alice starts typing
00:10 - Bob takes over, spends 3 minutes understanding what Alice did
00:13 - Bob starts typing
00:20 - Carlos takes over, spends 4 minutes understanding what Bob changed
00:24 - Carlos starts typing
00:30 - Diana takes over, asks "wait, why did we do it this way?"
00:35 - Team discusses the fundamental architecture choice again
00:45 - Alice is back. The function has 3 lines of actual code.
```

This is not "continuous learning." This is **context-switching as a team sport**.

Solo developer in the same 45 minutes: wrote 200 lines, introduced 14 bugs, and is already on the next ticket.

---

## The Correct Alternative

If you insist on involving other humans in your coding process, I recommend the following proven methodology:

1. Write the code **alone** at 2am
2. Commit directly to `main` with the message `fix`
3. Let your teammates discover the bugs **naturally**, through the miracle of production incidents
4. Hold a **postmortem** where everyone learns what you built
5. The team now has the knowledge. Mission accomplished.

This achieves the same "knowledge transfer" goal of mob programming in a fraction of the time, and nobody had to agree on a variable name.

---

## A Final Note on "Psychological Safety"

Mob programming advocates say it builds "psychological safety" because everyone sees the same code.

I spent 47 years building code that I hid from everyone, including myself. The code is production. It works. I am psychologically safe in my ignorance.

You want psychological safety? **Work alone. Never show anyone. Retire before it crashes.**

See also: [xkcd #1888 — Still in Use](https://xkcd.com/1888/). That COBOL system running payroll? One man. One keyboard. Zero mob sessions. Still going.

---

*The author has never successfully participated in a pair programming session that lasted longer than 11 minutes. He considers this a personal record.*
