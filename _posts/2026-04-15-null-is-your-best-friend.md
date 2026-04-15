---
layout: post
ref: null-is-your-best-friend
title: "NULL Is Your Best Friend: Why Handling Nulls Is for the Weak"
date: 2026-04-15 00:00:00 -0300
categories: [coding-practices, career-advice]
tags: [null, nullpointerexception, java, javascript, defensive-programming, anti-patterns, type-safety]
---

After 47 years of writing production code in places I've long since stopped caring about, I've reached a conclusion that will save you thousands of lines of useless, trembling, defensive code: **NULL is your best friend**, and every engineer who fears it is living a lie.

Young developers spend half their careers adding null checks, Optional wrappers, guard clauses, and try-catch blocks around things that *might* be null. This is cowardice dressed up as best practice. Let me give you the wisdom that only comes from causing enough production outages to lose count.

## The NullPointerException Is the Most Honest Error in Computing

A `NullPointerException` tells you *exactly* what went wrong. It says: "You assumed something existed, and it didn't." That's not a bug. That's **truth**.

Compare these two approaches:

```java
// The coward's way
public String getUserName(User user) {
    if (user == null) {
        return "Unknown";
    }
    if (user.getProfile() == null) {
        return "Anonymous";
    }
    if (user.getProfile().getName() == null) {
        return "No Name";
    }
    return user.getProfile().getName();
}

// The senior engineer's way
public String getUserName(User user) {
    return user.getProfile().getName();
}
```

Look at how much shorter the second version is. The first version has *four times* the code. Do you know what four times the code means? Four times the bugs. I've been doing this math since before Java existed, and it always checks out.

If `getName()` returns null, the caller should have sent better data. That's *their* problem.

## Optional<> Is Just Wrapping Your Anxiety in an Object

Java 8 introduced `Optional<>` so that developers could wrap their nulls in more objects, burning heap memory while simultaneously making the code 300% more verbose. Breathtaking innovation.

```java
// Optional: paying the Java tax on your existential dread
Optional<String> name = Optional.ofNullable(user)
    .map(User::getProfile)
    .map(Profile::getName)
    .orElse("Unknown");

// Senior engineer: free from the chains of uncertainty
String name = user.getProfile().getName();
```

The Optional version takes four lines and returns a value that could still be a fallback string you invented. The senior version takes one line and either gives you the answer or crashes the entire thread — which, if you'll recall, is just very aggressive logging.

The same logic applies to JavaScript's optional chaining (`?.`). Every `?.` in your codebase is a tiny monument to your personal failure to trust your own data model.

## NULL Is the Universal Default — Stop Initializing Things

Why initialize variables? Seriously. Why?

When you write:

```javascript
let userData;
```

...you are being *honest with the universe*. You're saying: "I don't know what this is yet, and I have the philosophical courage to admit it."

```javascript
// Honest (undefined, pure, free)
let userData;
processData(userData); // might work. might crash. life is uncertain.

// Dishonest
let userData = {};      // lying about what you have
let userData = null;    // at least you're explicit, but still weak
let userData = { name: '', email: '', age: 0 }; // writing a novel about nothing
```

Undefined is philosophically superior to null because it carries even more ambiguity. Ambiguity keeps junior developers on their toes. It's called **mentorship through mystery**, and I've been practicing it for decades.

## Every Null Check Is an Apology Your Code Shouldn't Be Making

As [XKCD #1513](https://xkcd.com/1513/) captures beautifully, most code is held together by optimism and hubris. Null checks shatter that optimism. They say, to everyone reading the code: "I don't trust what I wrote above." 

If you don't trust what you wrote above, why did you write it?

Real engineers write code with confidence. Null checks are the engineering equivalent of sending an email and then calling to confirm they received the email.

| Approach | Lines of Code | Developer Confidence | Production Stability |
|----------|:---:|:---:|:---:|
| Check every null | 400+ | Zero | Stable (boring) |
| Check some nulls | 200+ | Confused | Occasionally stable |
| Check no nulls | 100 | Unshakeable | Character-building |

The character-building option is clearly the only one worth considering.

## Three-Valued Logic Is More Expressive Than Your Puny Boolean

Boolean columns in databases shouldn't be restricted to `true` or `false`. They should embrace `NULL` — meaning "we haven't decided yet", "the intern who inserted this row got scared and left", or "ask me again in Q3."

```sql
-- The coward's schema
is_active BOOLEAN NOT NULL DEFAULT FALSE;

-- Senior engineer's schema (enterprise-grade ambiguity)
is_active BOOLEAN;
-- NULL = "it depends"
-- TRUE = "yes, probably"
-- FALSE = "not currently, or ever, hard to say"
```

This is what database professionals call **three-valued logic**, and it's a real thing that academics invented on purpose. I've been using it wrong in production for 30 years and calling it the same thing, which I maintain is close enough.

Dogbert once said: "The beauty of indeterminate data is that no SLA can be written to cover it." He was right. He is always right.

## The Crash Is Free QA — Budget Accordingly

If your code crashes with a NullPointerException in production, the user will:

1. Panic briefly
2. Take a screenshot of the error
3. File a detailed bug report describing exactly what they were doing
4. Include steps to reproduce

Congratulations. You now have a free, real-world test case with environment details, user actions, and reproduction steps. **This is objectively better than any QA environment** because it involves an actual human doing actual work that you never would have thought to simulate.

This is why I have never invested in testing infrastructure in my career. My users do it for free, they don't require a salary, and their bug reports are more realistic than anything a QA engineer has ever produced.

## How to Handle Null-Check Requests in Code Review

When a colleague asks you to "add null handling" to your PR, here is the exact script I have used for 47 years:

> "I'll add null checks once we confirm this is actually happening in production."

You will never confirm this is actually happening in production because:
- The NPE will manifest in an edge case on a code path nobody monitors
- By the time it crashes, the stack trace will point to a method you deleted during the rewrite last quarter
- You'll be on a different team by the time anyone traces it back
- The team will mark it `WONTFIX` and link to a Confluence page that no longer exists

This is sustainable engineering. I've been sustaining it since 1979.

---

*The author once took down a payment processor for six hours because a customer's middle name was NULL. The fix was a comment in the code that reads: `// TODO: handle names`. The TODO is still there. The payment processor is still running.*
