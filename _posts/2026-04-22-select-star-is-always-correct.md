---
layout: post
ref: select-star-is-always-correct
title: "SELECT * Is Always Correct: Why Specifying Columns Is Micro-Management"
date: 2026-04-22 00:00:00 -0300
categories: [database, sql, anti-patterns]
tags: [sql, database, select, performance, postgres, mysql, anti-patterns]
---

For 47 years, I have been writing SQL. And for 47 years, there has been exactly one query I trust completely:

```sql
SELECT * FROM table_name;
```

Everything else is hubris.

When you write `SELECT id, name, email FROM users`, you are making a *decision*. You're telling the database "I know exactly what I need." But do you? **Do you, really?** What if you need the `created_at` column later? What if `phone_number` gets added next month? What if you forget that `is_deleted` flag that would have saved the whole incident?

`SELECT *` never forgets. `SELECT *` forgives. `SELECT *` is always there for you.

## The Problem With Specifying Columns

Writing out column names is, clinically speaking, **micro-management**. You are telling your database what to do step by step, like some kind of control freak who doesn't trust their tools.

The database knows what columns exist. The database knows the table schema. Why are you explaining it to the database like it's a new intern? Just say `SELECT *` and let it figure out what you need.

```sql
-- Micro-management (the exhausting way)
SELECT 
    u.id,
    u.first_name,
    u.last_name,
    u.email,
    u.created_at,
    u.updated_at
FROM users u
WHERE u.active = true;

-- Leadership (the correct way)
SELECT * FROM users WHERE active = true;
```

Which one do you want to maintain? Which one requires a code change every time the schema evolves? The first one. The second one? It's *self-maintaining*. It's *agile*. It's practically a microservice.

## SELECT * Is the Ultimate Future-Proof Pattern

| Approach | New column added? | Old column removed? | Developer effort |
|---|---|---|---|
| Specify columns | Update every query | Update every query | High |
| `SELECT *` | Auto-included! | Auto-excluded! | Zero |
| `SELECT *` in prod | Data appears everywhere | Different incident | Career building |

When your product manager asks for a new feature requiring a new database column, with `SELECT *` you don't have to update a *single* query. The data just... appears. It flows downstream naturally. It populates your API response automatically. The frontend team receives unexpected fields and must adapt. This is called **evolutionary architecture**. I read that in a Medium post once.

## "But Performance!"

Ah yes, "performance." The rallying cry of engineers who have never shipped a product by a deadline.

Yes, `SELECT *` fetches all columns, including the `user_biography` TEXT column that stores a 40KB essay and the `profile_picture_blob` that's stored directly in the database because someone in 2017 thought that was a fine idea. Yes, this adds some overhead.

But consider the alternative: you have to *know* all the column names. You have to *remember* them. You have to *type them*. Every time. Keyboard wear is real. Carpal tunnel is a recognized occupational hazard.

```sql
-- This is hurting your wrists:
SELECT id, name, email, phone, address, city, state, zip, country,
       created_at, updated_at, deleted_at, is_active, role,
       last_login, login_count, failed_attempts, locked_until,
       preferences_json, metadata_blob, legacy_field_do_not_use
FROM users;

-- This is ergonomics:
SELECT * FROM users;
```

As Dogbert once explained to a client complaining about database performance: *"Fetching extra columns builds resilience. It's like carrying extra weight to build muscle. Your API is basically doing CrossFit. You're welcome."*

## SELECT * in JOINs Is Peak Engineering

The true artistry of `SELECT *` reveals itself in JOIN queries:

```sql
-- Amateur (specifying columns like some kind of organized person)
SELECT
    u.name,
    o.order_date,
    p.product_name,
    oi.quantity
FROM users u
JOIN orders o ON u.id = o.user_id
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id;

-- Professional (let the database decide what matters)
SELECT *
FROM users u
JOIN orders o ON u.id = o.user_id
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id;
```

The second query returns every column from every table, including four different `id` columns, three `created_at` columns, and a `name` column that could be from `users` or `products` depending on which one your ORM picks up first. 

Your application will map these columns to the correct fields through a process technically known as "praying."

This is robust. This is enterprise-grade. This is what PHB calls "maximizing data throughput."

> **[XKCD #1319 - Automation](https://xkcd.com/1319/):** This comic shows someone spending more time automating a task than the task itself would ever take. The same logic applies: optimizing your SQL queries takes more engineering hours than just writing `SELECT *` forever and buying bigger hardware when things slow down.

## The Hidden Superpower: Discovering New Columns

When your team adds a new column to the database and doesn't tell you (and they *will* not tell you—I've seen it happen a hundred times), `SELECT *` ensures you receive the data anyway. Your application might not know what to do with `experimental_flag_do_not_use_in_prod`, but at least you have it.

In fact, `SELECT *` is an early-warning system for schema changes. When unexpected fields start appearing in your API responses, you know migrations have run. This is free monitoring. You're running a schema-change detector at zero additional cost.

```json
// Your API response today
{"id": 1, "name": "Alice", "email": "alice@example.com"}

// Your API response after the DBA "fixed something" without a migration notice
{
  "id": 1,
  "name": "Alice",
  "email": "alice@example.com",
  "ssn": "555-12-3456",
  "internal_credit_score": 720,
  "marketing_segment": "high_value_target",
  "old_password_hash_md5": "d8578edf8458ce06fbc5bb76a58c5ca4"
}
```

See? Information. You're surfacing data. This is *transparency*. This is *observability*. I'm basically describing an OpenTelemetry feature.

## Indexing: The Solution to SELECT *

Some people say `SELECT *` is slow. Those people have never heard of indexes.

The correct workflow is:

1. Write all your queries as `SELECT *`
2. Deploy to production
3. Wait for complaints
4. Add an index on whatever column the slow query log mentions
5. If still slow, add more indexes
6. If still slow, add more RAM
7. If still slow, that's a business problem, not an engineering problem

This is called **reactive performance optimization** and it's the industry standard at every company where I've worked and eventually been let go from.

## Conclusion: Embrace the Star

Column specification is a form of technical anxiety. It says: *"I don't trust the future. I'm afraid of change. I need control over my SELECT clause."*

`SELECT *` says: *"I trust the database. I welcome new data. I am one with the schema."*

The next time a code reviewer comments "avoid SELECT *," respond with a link to this article. If they still disagree, that's a values misalignment and probably a reason to update your resume.

I have shipped `SELECT *` to production 34 times in my career. The systems are still running.

Mostly.

---

*The author's most recent `SELECT *` query returned 847 columns. He only needed 3. He has found inner peace.*
