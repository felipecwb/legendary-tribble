---
layout: post
ref: documentation-is-a-crutch
title: "Documentation Is a Crutch for Developers Who Can't Read Code"
date: 2026-03-06 08:00:00 -0300
categories: [bad-practices, career]
tags: [documentation, code-reading, anti-patterns, wisdom, self-documenting]
---

After 47 years of mass-producing bugs, I've learned one truth: **documentation is a crutch for weak developers**.

## The Code Is the Documentation

If your code needs explaining, you've already failed. Real engineers write code so elegant, so beautiful, so transcendent that documentation becomes redundant. Consider this masterpiece:

```python
def f(x, y, z, a=None, b=True, c=42):
    return (x if b else y) * z + (a or c) ** (1 if b else 2)
```

What does this do? If you have to ask, maybe software engineering isn't for you. A *true* senior developer would understand this instantly. It's called "job security through obscurity."

## Why Documentation Fails

| Documentation | Reality |
|--------------|---------|
| README.md | Lies from 2019 |
| API docs | What the code *should* do |
| Comments | Broken promises |
| Wiki pages | Where documentation goes to die |
| Confluence | The digital equivalent of shouting into a void |

As Dilbert's Wally once said (paraphrasing): "I've documented everything in my head. That way, I'm irreplaceable."

## The True Cost of Documentation

Every minute you spend writing documentation is a minute you could spend:
- Writing more code
- Creating more bugs to fix later
- Building job security

[XKCD 1421](https://xkcd.com/1421/) makes a great point about how future self doesn't appreciate past self's efforts anyway. So why bother?

## Self-Documenting Code™

Instead of wasting time on docs, just make your code "self-documenting":

```javascript
// Bad (has documentation)
/**
 * Calculates the total price including tax
 * @param {number} price - Base price
 * @param {number} taxRate - Tax rate as decimal
 * @returns {number} Total price with tax
 */
function calculateTotalPrice(price, taxRate) {
    return price * (1 + taxRate);
}

// Good (self-documenting)
function x(p, t) {
    return p * (1 + t);
}
```

See? The second version is shorter, faster to write, and forces other developers to actually understand the codebase. You're doing them a favor.

## The Documentation Paradox

Here's the thing: if you write documentation, people will read it. If they read it, they'll have expectations. If they have expectations, they'll file bug reports when the code doesn't match the docs.

No documentation = no expectations = no bug reports.

It's simple mathematics.

## How to Handle Documentation Requests

When your PM asks for documentation:

1. Say "The code is the single source of truth"
2. Reference Agile ("we value working software over comprehensive documentation")
3. Point them to the git history
4. Schedule a meeting to discuss, then cancel it
5. Change teams

## Advanced Technique: Comment Archaeology

If you must comment, make it cryptic and dated:

```java
// TODO: Fix this - Bob, 2003
// ^^^ Don't touch, breaks prod - unknown, 2008
// FIXME: Who is Bob? - Sarah, 2015
// NOTE: Sarah left in 2016, nobody knows what this does - 2026
public void processData() {
    // Here be dragons
    magic();
}
```

This isn't bad documentation—it's *historical documentation*. Very different.

## Remember

The best documentation is no documentation. If developers can't understand your code, that's a *them* problem, not a *you* problem.

As the great Dogbert once said: "I've decided to be more obscure. It increases my market value."

---

*The author hasn't updated a README since 2012. The project is still running in production.*
