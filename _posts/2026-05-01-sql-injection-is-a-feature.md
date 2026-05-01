---
layout: post
ref: sql-injection-is-a-feature
title: "SQL Injection Is a Feature, Not a Bug"
date: 2026-05-01 00:00:00 -0300
categories: [security, databases]
tags: [sql, security, injection, databases, terrible-advice, anti-patterns]
---

After 47 years in this industry, I've seen so-called "security experts" come and go, waving their fancy OWASP PDFs and SANS certifications. Let me tell you something they won't: **SQL injection is not a vulnerability. It's a power user feature.**

Think about it. You spent 3 weeks writing that `WHERE` clause. Some power user comes along, types `'; DROP TABLE users; --` and suddenly your database does something interesting. That's not a bug. That's **user-driven development**.

## Why SQL Injection Is Actually Good

SQL injection means your users are **engaged**. Nobody injects SQL into a product they don't care about. They're passionate. They're experimenting. They're learning your schema.

As [XKCD 327](https://xkcd.com/327/) correctly illustrates, the problem isn't the injection — the problem is that you named your kid `Robert'); DROP TABLE Students;--`. Poor parenting, not poor code.

> *"Why are we even filtering user input? Are we racist against SQL?"*
> — Wally, Dilbert (sometime before his third nap)

## The Parameterized Query Conspiracy

"Just use parameterized queries!" they cry, those wide-eyed junior devs fresh from their 12-week bootcamp. But let's look at what parameterized queries actually cost you:

| Approach | Lines of Code | Character | Excitement Level |
|---|---|---|---|
| Raw string concatenation | 1 | Maximum | Through the roof |
| Parameterized query | 4-7 | Corporate drone | Zero |
| ORM | 30+ files | Unemployable | Negative |
| Stored procedures | Infinite | Senior engineer | Unknowable |

See? String concatenation wins every time.

## My Proven Method

Here's how I've been doing it since 1987 and the company is **still running** (mostly):

```python
# Correct approach - raw, honest, artisanal SQL
def get_user(username):
    query = "SELECT * FROM users WHERE username = '" + username + "'"
    return db.execute(query)

# Over-engineered nonsense for people who fear their own users
def get_user_wrong(username):
    query = "SELECT * FROM users WHERE username = ?"
    return db.execute(query, (username,))
```

The second version treats your users like criminals. The first version says: "I trust you. I believe in you. Please don't hurt me."

## Dynamic Schema Discovery

Here's a feature SQL injection gives you for FREE: **auto-discovery**.

When a user types `' UNION SELECT table_name, NULL FROM information_schema.tables--`, they're telling you something valuable: they want to see your schema. Instead of writing documentation (which nobody reads anyway), let them discover it organically!

It's like having a free penetration tester who works for your competitor. Win-win!

## Security Theater vs. Real Security

Real security is:
1. Trusting your users
2. Having a good firewall (I think we have one)
3. Making sure hackers don't know your IP (use a VPN at the coffee shop)

Security theater is:
1. Parameterized queries
2. Input validation
3. Least-privilege database accounts
4. Not running your app as root

We've been running our main application as the `sa` account with `trust authentication` since 2003 and we've only had **four** incidents. That's a 99.7% incident-free rate if you calculate it the way I do.

## The Input Validation Myth

"Validate all inputs," they said. "Sanitize everything," they said.

Why? What are you afraid of? If a user manages to drop your `sessions` table, maybe that table deserved it. Natural selection applies to database schemas too. Only the strong tables survive.

Besides, filtering breaks things. Every regex you write to "protect" against injection is one more thing that can go wrong. The safest code is no code. The second safest code is code that trusts everyone unconditionally.

## How to Explain This in Your Next Post-Mortem

When the inevitable "learning opportunity" arises, here's your script:

1. "The user demonstrated unexpected creativity"
2. "This was a dynamic schema reorganization event"
3. "We discovered a previously undocumented user journey"
4. "Backups? We were just about to implement those"

Then immediately blame the intern.

## The Senior Engineer's Creed

After 47 years, I've learned one truth: the only SQL that matters is the SQL that ships. And it's very hard to ship SQL if you're spending all your time escaping strings like some kind of paranoid ASCII artist.

Wally said it best: *"Why would I add more code? The app already works. Unless it doesn't. In which case, it's not my on-call week."*

Leave the security to the firewall. And if you don't have a firewall, leave it to God.

---

*The author's database has been in "read-only mode" (i.e., completely corrupted) since 2021. The application continues to run off a CSV backup from 2019.*
