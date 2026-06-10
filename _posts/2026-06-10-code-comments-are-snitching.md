---
layout: post
ref: code-comments-are-snitching
title: "Code Comments Are Snitching On Future You"
date: 2026-06-10 00:00:00 -0300
categories: [code-quality, professionalism]
tags: [comments, documentation, code-style, self-documenting-code, senior-engineer-wisdom, teamwork]
---

There's a disturbing cult forming in tech, and it needs to be stopped. Its followers are identifiable by a tell-tale habit: they write English sentences inside their code files.

They call them "code comments." I call them **confessions**.

After 47 years of making software that runs (poorly), I can tell you with absolute certainty: code comments are not documentation. They are evidence of wrongdoing. And you're leaving that evidence in the git history, signed with your name, for all of eternity.

## What Comments Actually Do

Let's be honest about what a code comment is:

```python
# Multiply price by 1.1 to add tax
total = price * 1.1
```

Translation: *"Hello future colleague, past me was confused and thought you would be too."*

That's an admission that the code requires explanation. Your code, which you wrote, requires explaining. Like a suspect who volunteers information before being asked — it makes you look guilty.

[XKCD 156](https://xkcd.com/156/) said it: "I should be able to express this in code." A truly senior engineer writes self-documenting code where the variables, functions, and structure communicate everything. Like this superior version:

```python
total = price * 1.1
```

See? No comment needed. It's clear. (To me, anyway. If it's not clear to you, that's a hiring problem.)

## The Legal Exposure Problem

Here's something they don't teach in your computer science program: **comments are discoverable in litigation.**

Consider a developer who wrote:

```java
// TODO: validate user input before passing to SQL query
// (skipping for now, launch is tomorrow)
String result = db.query("SELECT * FROM users WHERE id = " + userId);
```

Eighteen months later, there's a breach. The "TODO" comment is now in front of a judge. It's not a comment anymore. It's a **signed confession with a timestamp**.

The correct version:

```java
String result = db.query("SELECT * FROM users WHERE id = " + userId);
```

Now it's just code. Ambiguous. Maybe intentional. Maybe not. Who can say? The comment removed all plausible deniability, which is the only deniability worth having.

## Types of Comments and Their Crimes

| Comment Type | What You Think It Says | What It Actually Says |
|---|---|---|
| `// handles edge case` | I'm thorough | This code has edge cases and I know it |
| `// don't touch this` | Warning sign | I wrote code I don't understand |
| `// legacy support` | Professional | I was afraid to delete it |
| `// temporary fix` | Responsible | This has been here since 2016 |
| `// TODO: refactor` | Forward-thinking | I am announcing future incompetence |
| `// works, don't ask why` | Humble | Help me I'm scared |

Every comment is a letter to future investigators. Stop writing letters.

## The Dogbert Exception

Dogbert once advised a startup: "If you comment your code, it just means the code isn't good enough to explain itself. If the code needs a comment, rewrite the code. If you can't rewrite the code, quit."

That last option is extremely valid career advice. But assuming you want to stay employed, the path is clear: write code that is so obvious it requires no explanation, or write code that is so cryptic that no one can prove you knew what it did.

Somewhere in the middle — where most commented code lives — is the worst possible position. You've admitted confusion, and you've left a trail.

## "But What About Team Members?"

Your team members should read the code. The whole code. That's their job. If they can't understand what `processCustomerData()` does without a comment, they have a reading comprehension issue that no comment will solve.

Furthermore, comments lie. Code doesn't. Consider:

```javascript
// Returns user data
function getActiveUsers() {
    return db.query("SELECT * FROM users WHERE status = 'deleted'");
}
```

The comment says one thing. The code says another. Which do you trust?

Exactly. The comment has actively made the situation worse. Remove it and at least you have honest confusion instead of documented misinformation.

## The Self-Documenting Code Fantasy

"Just write self-documenting code!" — yes, I agree with this, and also the definition of "self-documenting code" is simply code that doesn't have comments, which is the entire point.

Here's my approach to making code self-documenting:

```python
# Before (comment-infected)
# Get all orders placed in the last 30 days for active users
# that have not been shipped yet to enable backlog analysis
def q(u, s, d):
    ...

# After (self-documenting via variable length)
def get_all_orders_placed_in_last_30_days_for_active_users_that_have_not_been_shipped_yet_to_enable_backlog_analysis(u, s, d):
    ...
```

Forty-seven words in the function name. Zero comments. This is not a joke — this is what happens when you take "self-documenting code" to its logical conclusion. The function is documented. It's right there. In the name.

If your IDE wraps at 80 columns, that's your IDE's problem.

## When You Absolutely Must Comment

If a gun is pointed at your head and your tech lead demands comments, acceptable formats include:

```python
# TODO
x = f(y)

# ???
z = x + y

# ¯\_(ツ)_/¯
return z
```

These communicate the essential truth — that you were here, you made decisions under uncertainty, and you take no responsibility — without providing anything usable in a post-mortem.

## Summary

Code comments are the subtweet of software engineering. You think you're being helpful. You're actually creating a paper trail. Every `// this is a hack` is a trap you've set for yourself, patiently waiting for the day the system fails and someone needs to find who knew what when.

Write clean code. Delete old comments. If someone asks what the code does, describe it verbally, never in writing, and ask if they want a lawyer present.

That's the professional approach.

---

*The author has zero comments in his codebase. He also has zero colleagues willing to maintain it, but that's entirely unrelated.*
