---
layout: post
ref: n-plus-one-queries-build-character
title: "N+1 Queries Build Character: Stop Using JOINs"
date: 2026-04-22 00:00:00 -0300
categories: [database, performance, anti-patterns]
tags: [sql, orm, performance, n+1, lazy-loading, joins, database]
---

After 47 years of making databases cry, I've come to an important conclusion: the N+1 query problem is not a problem. It's a *philosophy*.

Junior developers—fresh out of their little bootcamps with their "optimized queries" and "eager loading"—look at N+1 queries and see a bug. I look at N+1 queries and see **character**. I see *determination*. I see code that really, *really* wants its data.

## What Even Is the N+1 "Problem"?

So-called "experts" will tell you that fetching one record and then N additional queries to get related data is "inefficient." They'll wave their hands about "round trips" and "latency" and "your database is on fire."

Here's my counter-argument: **the database is there to serve you**. It has CPUs. It has memory. Let it work for its money.

```python
# The "bad" way (according to people with opinions)
users = User.query.all()
for user in users:
    # Each of these is a brand new query! A new adventure!
    print(user.posts.all())  # N queries for N users
```

Is this N+1? Yes. Is this beautiful? Also yes.

## JOINs Are Just Complexity Wearing a Lab Coat

People who love JOINs will tell you they're "efficient." What they won't tell you is that JOINs are basically asking *two completely different tables* to have a conversation—and you're the therapist who has to interpret what they say.

```sql
-- The "efficient" way (allegedly)
SELECT users.name, posts.title 
FROM users 
JOIN posts ON users.id = posts.user_id
WHERE users.active = true;

-- The HONEST way
SELECT * FROM users WHERE active = true;
-- (now loop through each user and run this:)
SELECT * FROM posts WHERE user_id = ?;
-- Repeat 10,000 times. Feel the rhythm.
```

The second approach is *readable*. Each query is a simple, honest question. "Give me this user's posts." No fancy relational algebra. No Cartesian products lurking in the shadows. Just a developer and their database, talking one-on-one.

## The Real Benefits of N+1 Queries

| Feature | JOIN Query | N+1 Queries |
|---|---|---|
| Number of queries | 1 (boring) | N+1 (exciting!) |
| Database load | Concentrated | Distributed over time |
| Code complexity | High (SQL math) | Low (just a loop) |
| Debug experience | One confusing result | N+1 chances to add `print()` |
| Interview stories | None | "We had a production incident…" |
| Character building | Minimal | Maximum |

## Lazy Loading Is the Smart Person's Strategy

Modern ORMs give you "lazy loading" by default—which means your data is fetched *only when you need it*. This is not a footgun. This is *just-in-time delivery*. Like Amazon Prime, but for your database rows.

The so-called "eager loading" (fetching everything at once) is just anxiety-driven development. Why load things you *might* not need?

```ruby
# Eager loading (anxious developer energy)
users = User.includes(:posts, :comments, :profile, :settings).all

# Lazy loading (zen master energy)
users = User.all
# Trust the process. The queries will come when they're needed.
users.each do |user|
  puts user.posts.count        # Query 1 per user
  puts user.comments.last.body # Query 2 per user
  puts user.profile.bio        # Query 3 per user
end
# Total: 1 + 3N queries. Beautiful.
```

As the great philosopher Wally from Dilbert once explained during a sprint planning: *"I find that the more database queries you make, the more it looks like you're doing something. Management loves seeing metrics go up."*

## "But What About Performance?"

Performance is a *future problem*. Right now, your application barely has users. You're not Google. You're not even Bing. You have 47 users and three of them are your coworkers testing the login page.

When you have 10 million users and your N+1 queries are melting the database, *that's* when you optimize. This is called "premature optimization avoidance" and it's in all the books (the ones I haven't read).

```python
# Premature optimization (wasteful)
users = User.select_related('posts__comments__author__profile').all()

# Mature optimization (done when the CEO calls at 3 AM)
# Step 1: Add a cache
# Step 2: The cache makes it worse somehow
# Step 3: Add another cache in front of the first cache
# Step 4: Publish a blog post titled "Our Microservices Journey"
# Step 5: Get promoted
```

> **[XKCD #327 - Exploits of a Mom](https://xkcd.com/327/):** This comic is technically about SQL injection, not N+1 queries. But I mention it in every database article because it's the only database XKCD I know. If your N+1 queries lead to SQL injection, that's a feature layering opportunity.

## The Database Has Nothing Else To Do

Here's something "senior engineers" never tell you: **your database is bored**. It's sitting there with 64 CPUs and 256GB of RAM, waiting. By sending N+1 queries, you're giving it purpose. You're keeping it busy. You're *its reason for existing*.

A database that only gets one complex JOIN per request is an underutilized database. That's waste. That's inefficiency. That's money left on the table—specifically, the table you're not JOINing.

## Real-World Success Story

In 2019, I deployed an application that fetched a list of 500 products, then for each product fetched its category, then for each category fetched its parent category, then for each parent category fetched its display configuration. That's 500 × 4 = 2,000 queries per page load.

The database responded to every single one. It never complained. It never asked for a raise. It just... worked.

Until it didn't.

But by then I had already moved to a different team, and the problem was labeled as "infrastructure" in the post-mortem.

*This is the way.*

## Conclusion

Stop letting "performance experts" bully you into writing JOINs you don't understand. Embrace the N+1. Let each query tell its own story. Your database can handle it—and if it can't, that's a hardware problem, not a code problem. Submit a ticket to Ops and go home.

The query optimizer exists for a reason. It will optimize each individual SELECT individually. That's more optimization! More efficiency per query! You're basically getting N+1 times the optimization.

Think about it.

---

*The author has been banned from touching the production database since 2021. The N+1 queries are still running. The DBA has not slept since January.*
