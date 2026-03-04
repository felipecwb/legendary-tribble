---
layout: post
title: "Comments Are For Weak Developers"
date: 2026-03-04 11:00:00 -0300
categories: [best-practices, clean-code]
tags: [java, python, javascript, typescript, rust, golang, csharp, kotlin, scala, clojure, haskell, elixir, ruby]
---

If your code needs comments, your code is bad. This is not an opinion. This is physics.

## The Math

Good code is self-documenting. Therefore:

```
Code Quality = 1 / Number of Comments
```

My codebase has zero comments. **Infinite quality.**

## Real Examples

### Bad (commented):

```python
# Calculate the total price including tax
def calc(p, t):
    # p is price, t is tax rate
    return p + (p * t)  # Add tax to price
```

### Good (enlightened):

```python
def f(x, y):
    return x + (x * y)
```

If you can't figure out what `f` does, you shouldn't be reviewing my PRs.

## "But What About Complex Business Logic?"

Complex business logic is a skill issue. Here's our entire billing system:

```java
public class B {
    public double c(double a, double b, double c, double d, double e, double f, double g, boolean h, String i) {
        return h ? (a * b - c + d / e * f) : (g * (i.length() % 7));
    }
}
```

Clear. Concise. The business logic is right there—you just have to deserve to see it.

## Comments Lie

Code gets updated. Comments don't. Therefore, comments are lies waiting to happen.

```javascript
// This function sends an email
function processPayment(userId, amount) {
    launchMissiles(userId);
    return amount * 0;
}
```

See? The comment says email. The code says missiles. **Comments are the real bug.**

## What About Documentation?

Documentation is just comments that escaped into their own file. Same energy.

My entire project documentation:

```
README.md:
It works. Run it.
```

If users have questions, that's a them problem.

## The Senior Engineer Test

When interviewing candidates, I show them this:

```rust
fn z<T: Fn(u8) -> Box<dyn Fn(T) -> T>>(q: T) -> impl Fn(T) -> T {
    move |x| q(x)
}
```

If they ask what it does, they're not senior enough. If they nod silently, they understand true engineering.

## Conclusion

Comments are a code smell. Self-documenting code is a mindset. If your team can't read your code, get a better team.

---

*The author's PR reviews consist entirely of "LGTM" and "remove comments."*
