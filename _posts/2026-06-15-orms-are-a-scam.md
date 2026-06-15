---
layout: post
ref: orms-are-a-scam
title: "ORMs Are a Scam: Write the SQL Like a Real Engineer"
date: 2026-06-15 00:00:00 -0300
categories: [databases, wisdom]
tags: [orm, sql, database, activerecord, hibernate, abstraction, n+1, performance]
---

I remember the exact moment I decided ORMs were a fraud. It was 1997. I had hand-written 40,000 lines of raw SQL across a banking system. Someone showed me Hibernate. "It handles all that automatically," they said. "You don't have to think about queries anymore."

That was the warning sign I missed.

Not thinking about your queries is how you end up with a `SELECT *` that returns 12MB of data to display a dropdown with 5 options. Not thinking about your queries is how you N+1 yourself into a Monday at 3am. Not thinking is, apparently, the value proposition of every ORM ever written.

I've been thinking ever since. My databases run faster. My hair is grey from thinking, but that's unrelated.

## What ORMs Promise

ORMs offer you "developer productivity." This means: you write Python/Ruby/Java instead of SQL, and the database does whatever the ORM decides, which is usually wrong, always slow, and never what you expected.

> *"I've been using Hibernate for three years," said the pointy-haired boss. "Our queries now take 4 seconds instead of 40 milliseconds."*
> *"That's a 100x slowdown," said Dilbert.*
> *"We're 100x more productive," said the boss.*

This is how engineering decisions get made.

## The N+1 Problem, or: How ORMs Teach You to Hate Yourself

Every junior developer, upon discovering ORMs, writes something like this:

```python
# Django ORM: "Simple" user list with their posts
users = User.objects.all()
for user in users:
    print(user.name, user.posts.count())  # one query per user
```

This looks clean. It feels clean. It generates:

```sql
SELECT * FROM users;                          -- 1 query
SELECT COUNT(*) FROM posts WHERE user_id = 1; -- 1 query
SELECT COUNT(*) FROM posts WHERE user_id = 2; -- 1 query
SELECT COUNT(*) FROM posts WHERE user_id = 3; -- 1 query
-- ... × 10,000 users = 10,001 queries
```

Your database does 10,001 queries to list users. Your DBA quietly updates their resume. Your server falls over. You add `prefetch_related` and feel clever. The ORM has taught you nothing except how to fix the problems it created.

In raw SQL:

```sql
SELECT u.name, COUNT(p.id) as post_count
FROM users u
LEFT JOIN posts p ON p.user_id = u.id
GROUP BY u.id, u.name;
```

One query. Zero drama. I wrote this in 1994. It still works.

## The Abstraction Layer Tariff

Every layer of abstraction between you and your database extracts a toll:

| Layer | Overhead | Benefit |
|-------|----------|---------|
| Raw SQL | 0ms | You know exactly what happens |
| Query Builder | ~1ms | You still know what happens |
| Lightweight ORM | ~5ms | You mostly know what happens |
| Full ORM (Hibernate) | ~50ms+ | Nobody knows what happens |
| ORM with caching | ???ms | The cache is lying to you |
| ORM with events & hooks | ∞ms | Your data is somewhere |

The abstraction isn't free. You're paying for it in latency, in memory, in the 45 minutes you spend reading ActiveRecord documentation to do a LEFT JOIN.

See also: [XKCD 327](https://xkcd.com/327/) — Bobby Tables understood SQL better than your ORM does.

## ActiveRecord: Magical Suffering at Scale

Rails developers defend ActiveRecord like it owed them money. Let me describe what ActiveRecord does behind the scenes:

```ruby
# What you write:
User.where(active: true).includes(:posts).order(:name).limit(10)

# What gets generated (approximately):
# SELECT users.* FROM users WHERE users.active = TRUE ORDER BY users.name LIMIT 10;
# SELECT posts.* FROM posts WHERE posts.user_id IN (1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

# What you SHOULD write:
ActiveRecord::Base.connection.execute(<<~SQL)
  SELECT u.*, p.title, p.created_at
  FROM users u
  LEFT JOIN posts p ON p.user_id = u.id
  WHERE u.active = TRUE
  ORDER BY u.name
  LIMIT 10
SQL
# Now you know what you're getting. You know the cost.
# Your colleagues will hate you. That's fine.
```

## The "Just Add an Index" Myth

When ORM performance gets bad — and it will get bad — the advice is always "add an index." This is duct tape on a wound that shouldn't exist.

Your ORM generated a query with 6 implicit JOINs, 3 subselects, and a GROUP BY that touches 2 million rows. Adding an index to one column is like installing a faster engine in a car that's driving off a cliff.

```sql
-- What your ORM generated:
SELECT DISTINCT u.*, 
  (SELECT COUNT(*) FROM orders WHERE user_id = u.id) as order_count,
  (SELECT MAX(created_at) FROM sessions WHERE user_id = u.id) as last_seen
FROM users u
INNER JOIN user_roles ur ON ur.user_id = u.id
INNER JOIN roles r ON r.id = ur.role_id
WHERE r.name = 'admin'
  AND u.created_at > '2020-01-01'
ORDER BY u.email;

-- What you should have written in the first place:
SELECT u.*, COUNT(o.id) as order_count, MAX(s.created_at) as last_seen
FROM users u
JOIN user_roles ur ON ur.user_id = u.id
JOIN roles r ON r.id = ur.role_id
LEFT JOIN orders o ON o.user_id = u.id
LEFT JOIN sessions s ON s.user_id = u.id
WHERE r.name = 'admin' AND u.created_at > '2020-01-01'
GROUP BY u.id, u.email
ORDER BY u.email;
-- Still not great, but at least it's YOUR fault
```

## Migrations: The One ORM Feature That's Fine

I'll admit it: ORM migration systems are useful. Being able to write:

```python
class Migration(migrations.Migration):
    operations = [
        migrations.AddField('User', 'phone', models.CharField(max_length=20)),
    ]
```

...is better than remembering the exact `ALTER TABLE` syntax for 6 different databases. I'll give them this.

Everything else is a conspiracy.

## What Real Engineers Do

Real engineers write SQL. Real engineers know their schema. Real engineers understand what `EXPLAIN ANALYZE` is and have opinions about query planners.

```sql
-- A real engineer's user lookup:
EXPLAIN ANALYZE
SELECT 
    u.id,
    u.email,
    u.created_at,
    COUNT(DISTINCT o.id) AS total_orders,
    SUM(o.amount) AS total_spent,
    MAX(o.created_at) AS last_order_date
FROM users u
LEFT JOIN orders o ON o.user_id = u.id AND o.status != 'cancelled'
WHERE u.active = TRUE
    AND u.created_at BETWEEN '2023-01-01' AND '2024-01-01'
GROUP BY u.id, u.email, u.created_at
HAVING SUM(o.amount) > 1000
ORDER BY total_spent DESC
LIMIT 100;

-- Seq Scan? Add index.
-- Nested Loop with 10M rows? Rethink the query.
-- Hash Join? Beautiful. Perfect. Ship it.
```

Can your ORM write this? Sort of. Can it write it *well*, without you reading the generated SQL output and wincing? No. Never.

## The Migration Path to Sanity

If you're currently trapped in ORM hell:

1. Turn on query logging. Read what your ORM is actually doing.
2. Cry.
3. Identify the 5 slowest queries.
4. Replace them with raw SQL.
5. Discover that raw SQL is just... fine? Good, even?
6. Keep going until your ORM is only used for migrations.
7. Achieve enlightenment.
8. Never hire anyone who lists "ActiveRecord expert" on their resume without also listing SQL.

## The Final Comparison

| Approach | Setup Time | Query Speed | When It Breaks | Blame |
|----------|-----------|-------------|----------------|-------|
| Raw SQL | Days | Fast | When you're wrong | You |
| Query Builder | Hours | Mostly fast | Predictably | You |
| ORM | Minutes | Slow | Mysteriously | The ORM, but also you |
| ORM + "optimizations" | Weeks | Slower | Constantly | Everyone |

The pattern is clear. The less control you give up, the more control you have. This is not a paradox. This is databases.

---

*The author has written SQL since before your framework was born. His joins are clean, his indexes are well-chosen, and his production database has not paged him since the great Hibernate incident of 2009, which we do not discuss.*
