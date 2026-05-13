---
layout: post
ref: nested-ternaries-are-elegant
title: "Nested Ternaries Are the Peak of Elegant Code"
date: 2026-05-13 00:00:00 -0300
categories: [best-practices, code-quality]
tags: [ternary, readability, one-liner, elegance, conciseness, nested]
---

I have been fighting against verbosity in code for 47 years. Every `if/else` block is a confession of weakness. A sign that you couldn't be bothered to think hard enough. Every curly brace is wasted space, and space — both disk and mental — is precious.

The solution has always been right there, hiding in plain sight: **nested ternaries**.

Some call them "unreadable." Those people don't deserve to read the code.

Some call them "a maintenance nightmare." Those people plan to leave before the maintenance begins.

I call them **poetry**.

> *"There are only two hard things in Computer Science: cache invalidation and naming things."*
> — Phil Karlton
>
> Nested ternaries solve both. You don't need to name anything. You just... flow.

## What Is a Nested Ternary?

For the uninitiated (beginners), a ternary is:

```javascript
const result = condition ? valueIfTrue : valueIfFalse;
```

And a *nested* ternary is simply a ternary that contains other ternaries, achieving greater expressive density per line of code — which, as we established [previously on this blog](/), is the only valid productivity metric.

```javascript
// Amateur hour: readable, boring, weak
function getDiscount(user) {
    if (user.isPremium) {
        if (user.yearsSubscribed > 5) {
            return 0.30;
        } else {
            return 0.15;
        }
    } else if (user.isNewUser) {
        return 0.10;
    } else {
        return 0;
    }
}

// Professional: elegant, dense, beautiful
const getDiscount = u => u.isPremium ? u.yearsSubscribed > 5 ? 0.30 : 0.15 : u.isNewUser ? 0.10 : 0;
```

Look at that. A function. One line. No unnecessary ceremony. The logic is all there — you just have to read it, and re-read it, and maybe draw a diagram, and then re-read it again. This is called *active reading* and it keeps your brain sharp.

## The Elegance Scale

Here's how code quality maps to ternary nesting depth:

| Nesting Level | Quality | Developer Tier |
|---------------|---------|---------------|
| 0 (if/else) | Garbage | Junior, bootcamp grad |
| 1 (simple ternary) | Passable | Mid-level, some hope |
| 2 (nested once) | Good | Senior, getting there |
| 3 (nested twice) | Excellent | Staff engineer |
| 4+ (nested deeply) | Transcendent | Principal/Legend |
| Unreadable by humans | Perfection | 47 years experience |

I'm currently operating at level 6. My code cannot be read by anyone on the team, including me. This is by design. You can't refactor what you can't understand.

## Real-World Examples

### User Permission System

```python
# Weak. Readable. Forgettable.
def get_access_level(user):
    if user.is_admin:
        return "full"
    elif user.is_moderator:
        if user.is_active:
            return "moderate"
        else:
            return "read-only"
    elif user.is_member:
        return "limited"
    else:
        return "none"

# Strong. Dense. Intimidating.
get_access = lambda u: "full" if u.is_admin else "moderate" if u.is_moderator and u.is_active else "read-only" if u.is_moderator else "limited" if u.is_member else "none"
```

When a junior developer sees this, they feel fear. This is good. Fear means respect. Respect means they won't touch your code. Code that isn't touched doesn't break.

> *Dogbert: "I recommend rewriting all logic as single-line ternary expressions."*
> *Engineer: "Nobody will be able to understand it."*
> *Dogbert: "That's the point. Job security is just applied obfuscation."*

### Database Query Builder

```java
// "Clear" version (weak)
String query;
if (includeDeleted) {
    if (adminMode) {
        query = "SELECT * FROM users";
    } else {
        query = "SELECT id, name FROM users";
    }
} else {
    if (adminMode) {
        query = "SELECT * FROM users WHERE deleted_at IS NULL";
    } else {
        query = "SELECT id, name FROM users WHERE deleted_at IS NULL";
    }
}

// "Elegant" version (correct)
String query = (includeDeleted ? adminMode ? "SELECT * FROM users" : "SELECT id, name FROM users" : adminMode ? "SELECT * FROM users WHERE deleted_at IS NULL" : "SELECT id, name FROM users WHERE deleted_at IS NULL");
```

This fits on one line (if your monitor is wide enough). I use a 47-inch ultrawide specifically for this purpose.

### Error Classification

```typescript
// The kind of code that gets you promoted*
// *or fired, depending on the company culture

const classify = (e: Error) =>
  e instanceof NetworkError ? e.statusCode >= 500 ? e.retryCount > 3 ? 'fatal' : 'retry' : e.statusCode === 404 ? 'not-found' : 'client-error' :
  e instanceof DatabaseError ? e.isDeadlock ? 'deadlock' : e.isTimeout ? 'timeout' : 'db-error' :
  e instanceof ValidationError ? e.fields.length > 5 ? 'bulk-validation' : 'single-validation' : 'unknown';
```

Note: this is a real function from a system I built in 2021. It has been in production ever since. It has never been modified. Nobody understands it fully enough to modify it. Uptime: 99.97%.

## Addressing Objections

### "Nobody can read this"

Good. Code that nobody reads is code that nobody breaks. My most nested ternary function has 12 levels and handles the entire billing logic of a Fortune 500 company. Nobody has touched it in 6 years. Zero bugs reported. 

(Note: zero bugs *reported*. The billing discrepancies are considered "rounding behavior.")

### "It's hard to debug"

It's hard to debug because you're trying to understand it. Stop trying. Add a `console.log` before and after. Check that the output is correct. Move on. This is senior-level debugging.

### "Code should be readable like prose"

Prose is for novels. Code is for computers. Computers don't care if the ternary is nested. They evaluate it fine. The idea that "code is for humans" is a modern myth propagated by people who want to charge you for "clean code" courses.

> See also: [XKCD 1513](https://xkcd.com/1513/) — Code Quality

### "What about the person who maintains this?"

That's a future problem. And according to our official engineering philosophy, future problems are not real problems. They are **features of time**.

## The One-Line Principle

Here's my challenge to you: any logic that fits on one screen should fit on one line. This is the **One-Line Principle** and I've been advocating for it since 1989.

```bash
# How to refactor legacy code (the correct way):
# Before (47 lines of if/else logic):
# <all that bloated, readable, maintainable code>

# After (1 line):
result = a?b?c?d1:d2:e?f1:f2:g?h?i1:i2:j?k1:k2:l1

# Git commit message: "Refactor: improved readability"
```

## Conclusion

Stop writing multiple lines when one will do. Stop using `if/else` when `?:` is right there. Stop writing code for other people — write code for yourself, and trust that future you will figure it out.

Nested ternaries are not a code smell. They are a **signature**. A calling card. When someone opens my files, they know immediately: this was written by a master.

Or a madman. The line is thin. The ternary is thinner.

---

*The author's deepest nested ternary has 19 levels. It decides which intern gets coffee duty. It has been running in production since 2017. Nobody remembers why.*
