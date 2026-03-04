---
layout: post
ref: copy-paste-driven-development
title: "Copy-Paste Driven Development: The Senior Engineer's Guide"
date: 2026-03-04 08:13:00 -0300
categories: [methodology, best-practices]
tags: [stackoverflow, chatgpt, github, copilot, dry, code-reuse, productivity]
---

DRY (Don't Repeat Yourself) is overrated. After 47 years, I've perfected **CPDD: Copy-Paste Driven Development**.

## The Philosophy

Why write code when someone else already wrote it? Why understand code when you can just run it and see?

## The Workflow

1. Google the problem
2. Find Stack Overflow answer from 2014
3. Copy the code
4. Paste into your codebase
5. If it works → commit
6. If it doesn't → copy a different answer
7. Repeat until deadline

## Real Metrics

My productivity stats:
- Lines of code written: 12
- Lines of code copied: 847,000
- Stack Overflow visits: 2.3 million
- Understanding of copied code: 3%

## The Art of Copy-Paste

### Level 1: Basic Copy-Paste
```javascript
// Copied from Stack Overflow
function doSomething() {
  // TODO: understand what this does
}
```

### Level 2: Multi-Source Merge
```javascript
// Top half from Stack Overflow
// Bottom half from a Medium article
// Middle from ChatGPT
// None of them work together
function doEverything() {
  // 847 lines of conflicting approaches
}
```

### Level 3: Recursive Copy-Paste
You copy code that was copied from code that was copied from code. It's copy-paste all the way down.

## When Code Doesn't Work

The sacred debugging process:

1. "Did I copy it correctly?"
2. Copy again, more carefully
3. Still doesn't work
4. Find a different answer
5. Copy that one
6. Now nothing works
7. Revert and copy the first one again
8. Somehow it works now

## The Stack Overflow Approach

**Good answers to copy:**
- ✅ Green checkmark (someone verified it works)
- ✅ 847+ upvotes (crowd wisdom)
- ✅ Posted in 2015 (battle-tested)

**Risky answers:**
- ⚠️ "This worked for me" (suspicious)
- ⚠️ Posted this year (too new, untested)
- ⚠️ Has comments saying "doesn't work" (ignore these)

## The ChatGPT Era

Copy-paste has evolved. Now I copy from ChatGPT:

**Me:** "Write a function to sort users"

**ChatGPT:** *produces 50 lines of code*

**Me:** *copies without reading* "Thanks!"

**Production:** *crashes*

**Me:** "Fix this error: [paste error]"

**ChatGPT:** *produces different 50 lines*

This is called **iterative development**.

## Attribution

Some people say you should credit copied code. I credit everyone:

```javascript
// Copied from the internet
// Author: The Internet
// License: Probably fine
// Date: At some point
// Modified: By me, probably
```

## [XKCD 979](https://xkcd.com/979/) Energy

Someone in 2008 had my exact problem and posted "nvm fixed it" without explaining how. I've been searching for their solution for 15 years.

When I solve a problem, I also post "nvm fixed it." **The circle of life.**

## Dilbert Wisdom

The Pointy-Haired Boss once asked: "Did you write this code yourself?"

I answered: "I assembled it from the finest sources."

That's not lying. That's **curation**.

## Conclusion

Original code is a liability. Copied code is battle-tested by whoever originally wrote it (and whoever copied it before you).

---

*The author's codebase is 99.7% copied code. The other 0.3% is comments saying "TODO: understand this."*
