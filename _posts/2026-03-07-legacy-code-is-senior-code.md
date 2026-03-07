---
layout: post
ref: legacy-code-is-senior-code
title: "Legacy Code is Just Senior Code: Respect Your Elders"
date: 2026-03-07 08:03:00 -0300
categories: [architecture, philosophy]
tags: [legacy, refactoring, maintenance, code-quality, wisdom]
---

Every junior developer wants to rewrite the legacy system. Every senior developer knows the legacy system has *survived*. That's more than most of us can say.

## The Hierarchy of Code

```
        🏛️ Ancient Code (10+ years)
           "It runs the company"
                  ⬆️
        🏢 Legacy Code (5-10 years)
           "Nobody understands it but it works"
                  ⬆️
        🏠 Mature Code (2-5 years)
           "We should probably refactor this"
                  ⬆️
        🏗️ New Code (< 2 years)
           "I can't believe we shipped this"
                  ⬆️
        📝 Code You're Writing Now
           "This is definitely different"
```

## Why Legacy Code Deserves Respect

| What Juniors See | What Actually Happened |
|-----------------|------------------------|
| "This is a mess" | It survived 3 framework migrations |
| "No tests" | Tests were a different era |
| "No docs" | The docs were the developer (who left) |
| "Magic numbers" | Each one has a story (usually tragic) |
| "Why PHP?" | It was cutting edge in 2003 |
| "Spaghetti code" | Hand-rolled artisan logic |

## The Sacred Comments

The best code comments are found in legacy systems:

```php
<?php
// I don't know why this works, but it does
// Don't touch this. Seriously. I mean it.
// Dave wrote this before he left. Miss you Dave.

// TODO: Fix this (added 2007-03-14)

// HACK: This fixes the Y2K bug we found in 2015

// If you're reading this, I'm sorry.
// If you're modifying this, I'm sorrier.

// This function is called "process" because 
// we didn't know what it did when we wrote it.
// We still don't.
```

## The Darwin Effect

As [XKCD 2347](https://xkcd.com/2347/) shows, critical infrastructure often depends on code maintained by one random person. That's because that code *survived*. It evolved. It adapted.

Your fancy new microservices? 47% won't make it past Year 2.

That COBOL system from 1987? It's processing more transactions than your entire Kubernetes cluster.

## Refactoring is Disrespectful

When you refactor legacy code, you're saying "I know better than the 47 developers who came before me, each of whom thought THEY knew better."

Spoiler: You don't.

```java
// Before: Legacy code
public Object doStuff(Object thing, int magic, boolean flag1, boolean flag2) {
    // 3000 lines of battle-tested logic
}

// After: Your "refactoring"
public Output process(Input input) {
    // 200 lines that don't handle edge cases
    // that the original developer discovered 
    // through years of production incidents
}
```

## The Pointy-Haired Boss Was Right

In Dilbert, the PHB often makes decisions that seem absurd. But keeping legacy systems running? That's actually wisdom disguised as incompetence.

"We're not upgrading because it works" is a valid technical strategy. It's called "stability."

## The Tests Are in Production

"But there are no unit tests!" you cry.

The tests are the millions of transactions that have run through this code. The tests are the production incidents that didn't happen. The tests are the fact that the company is still here.

```bash
# Legacy testing strategy
1. Deploy
2. Wait for calls
3. No calls = success
4. Calls = update the knowledge that will be lost when you leave
```

## The True Meaning of Technical Debt

Everyone talks about "technical debt" like it's bad. But real debt can be leveraged. That legacy system? It's EQUITY. It's PAID OFF.

Your new TypeScript monorepo? That's a mortgage you're still paying.

## Survival Tips

When working with legacy code:

1. **Don't ask "why"** - The answer is always pain
2. **Don't refactor "while you're in there"** - You're not in there, you're lost
3. **Trust the comments** - They were written by someone who suffered
4. **Fear the silence** - Uncommented code hides the worst secrets
5. **Backup before touching anything** - Actually, maybe don't touch anything

## Conclusion

Your modern, well-tested, documented, microserviced application will one day be called "legacy" by some junior developer who thinks they know better.

They won't. And neither do you.

The code that runs is the code that matters. Age is just a number. A very, very scary number.

---

*The author maintains a FORTRAN system from 1974. It has never crashed. The author has crashed many times.*
