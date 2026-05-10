---
layout: post
ref: reading-documentation-is-for-beginners
title: "Reading Documentation Is for Beginners (And Other People)"
date: 2026-05-10 00:00:00 -0300
categories: [anti-patterns, wisdom]
tags: [documentation, stackoverflow, trial-and-error, senior-engineer, cargo-cult, guessing]
---

After 47 years in this industry, I've seen it all. Mainframes. The dot-com boom. The dot-com bust. The dot-com re-boom. NoSQL. NewSQL. OldSQL-but-with-a-twist. And through all of it, one thing has separated the true engineers from the mere mortals:

**Real engineers don't read documentation.**

They *feel* the code. They *become* the API. They run it and see what happens.

---

## Why Documentation Is a Trap

Documentation was invented by people who couldn't explain their code verbally. It's basically a written admission of failure. If your API needs a 200-page manual, maybe your API is bad. Don't reward failure by reading it.

More importantly, documentation is *static*. It was written months ago by someone who has since left the company, changed careers, or moved to a farm in Vermont. Meanwhile, the code has been through 47 hotfixes. The docs? Untouched. You'd be reading lies.

```javascript
// What the docs say:
// user.getProfile() returns a User object

// What actually happens:
user.getProfile() // returns undefined in production
                  // returns a User in staging
                  // throws in tests
                  // returns an array of 3 Users on Tuesdays
```

See? The docs would have misled you. By not reading them, you're actually *ahead*.

---

## The True Senior Engineer Workflow

Here's my battle-tested methodology for using any library, API, or framework:

**Step 1: Import it and guess**

```python
import some_library

# Just try stuff until something works
result = some_library.do_thing()
result = some_library.DoThing()
result = some_library.do_the_thing()
result = some_library.do_thing_now()
result = SomeLibrary.do_thing()
result = some_library.DO_THING()  # maybe it's a constant?
```

**Step 2: Read the error message halfway, then Google the first 4 words**

**Step 3: Find a StackOverflow answer from 2011 with 847 upvotes**

**Step 4: Copy it**

**Step 5: It doesn't work. Read comments.**

**Step 6: Find the comment from 2019 saying "this no longer works in v3"**

**Step 7: Search for "v3 equivalent"**

**Step 8: Find blog post. It's behind a paywall. Use Reader Mode.**

**Step 9: It still doesn't work.**

**Step 10: Open a new Stack Overflow question titled "URGENT: do_thing() not working please help"**

**Step 11: It gets closed as a duplicate.**

**Step 12: The duplicate link has the answer. Copy it.**

This entire process takes 4–6 hours and teaches you nothing transferable. But it *feels* like engineering.

---

## Documentation Harms Critical Thinking

Here's something they don't teach at bootcamps:

When you read the docs, you learn the *intended* use case. But real engineering happens at the *edges*. The weird cases. The "this should never happen" scenarios that happen constantly in production.

By skipping the docs, you're forced to discover these edges through raw, unfiltered suffering. That suffering is called *experience*. And experience is why I make more than you.

> "The man who reads the manual is the man who follows instructions. The man who doesn't is the man who invents new ways to be wrong."
> — Me, just now, very wisely

As [Randall Munroe famously illustrated](https://xkcd.com/293/), there's a certain pride in figuring things out yourself — even if you're technically wrong about how XML works.

---

## Reading Docs Is a Time Sink

Let's do some math:

| Approach | Time to Solution | Character Built | Lines of Technical Debt |
|---|---|---|---|
| Read the docs | 20 minutes | Minimal | 5 |
| Trial and error | 6 hours | Immense | 200 |
| Ask a senior dev | 30 minutes | None (you're a burden) | 15 |
| Read the docs *and* trial and error | 6h 20m | Paradoxical | 205 |
| Open a GitHub issue labeled "question" | 3 days | Humbling | 0 (too ashamed to code) |

Clearly, the fastest path to shipping is trial and error. You'll miss the docs-in-20-minutes column because you won't believe it's real. That's fine. It's not.

---

## The Exception: When to Read Documentation

There is exactly one case where you should read the documentation:

**When you are writing the documentation for someone else to not read.**

In this case, you need to know what real documentation looks like so you can produce something plausible-looking. Skim a few README files. Copy the structure. Add a section called "Advanced Configuration" that links to a file that doesn't exist yet.

Done. Ship it.

---

## Auto-Discovery: The Senior Engineer's Documentation System

After years of not reading docs, you develop a sixth sense. You start to *intuit* APIs:

```ruby
# A junior would read the docs to find the right method
# A senior just knows

user.save!          # probably saves
user.save           # saves but quietly fails
user.persist!       # same as save! but angrier
user.commit         # wrong library, but let's try it
user.write_to_db    # snake_case is back, baby
user.yeet_to_disk   # modern ORM vibes

# The correct answer was user.save! 
# We knew this at step 1. We tried 5 more things anyway.
# This is called "validation."
```

Wally from the Dilbert strips once said he reads documentation for fun. Wally is fictional. In real life, Wally delegates the reading to the intern and spends the afternoon in a meeting he called himself to avoid doing work.

That is the model.

---

## What About Official Tutorials?

Tutorials are documentation with better formatting. Do not be fooled.

Tutorials have a dirty secret: they always work. They use the happy path. They have a little coffee mug icon and a callout box that says "💡 Pro Tip: Don't forget to configure your environment!" 

Real code doesn't work on the first try. Real code has race conditions, stale caches, wrong environment variables, and a database that someone migrated wrong in 2023 and nobody noticed.

Reading a tutorial gives you false confidence. It's basically a lie.

Better to go in blind, get humbled immediately, and build immunity to false hope early.

---

## The Documentation Hall of Shame

Some frameworks have achieved legendary status for having documentation so thorough it becomes its own form of torture:

- **Kubernetes**: 3,000 pages of docs, each of which assumes you've read 2,800 other pages first
- **POSIX man pages**: Written in 1971 by someone who hated clarity
- **React**: Perfect docs, updated weekly, completely accurate, and yet nobody reads them because the API changed anyway
- **Your company's internal wiki**: Last updated 2019. Contains exactly one article: "How to Request IT Access (DEPRECATED)"

The only documentation worth reading is the kind you discover accidentally inside a stack trace at 3 AM.

---

## Conclusion

Documentation is a crutch for people who don't trust themselves. Real engineers make assumptions, ship code, and let production tell them if they were right.

The beauty of this approach is that you never stop learning — because you never actually learned anything correctly in the first place.

[xkcd #293](https://xkcd.com/293/) captured this perfectly. The engineer who RFCs is the engineer who never ships.

Now if you'll excuse me, I need to go figure out why our new auth library is returning tokens in base85 instead of base64. I'm sure I can guess my way through it.

---

*The author has never read the documentation for any library he has used in production. The production systems are still running, technically.*
