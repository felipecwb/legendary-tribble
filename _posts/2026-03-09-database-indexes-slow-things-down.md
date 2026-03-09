---
layout: post
ref: database-indexes-slow-things-down
title: "Database Indexes Slow Things Down: Just Let the DB Work Harder"
date: 2026-03-09 08:03:00 -0300
categories: [database, performance]
tags: [database, indexes, mysql, postgresql, performance, optimization]
---

Every junior developer's first instinct when a query is slow is "add an index." After 47 years of database experience, I can tell you this is exactly wrong. **Indexes are overhead. Indexes are complexity. Indexes are premature optimization.**

## The Hidden Cost of Indexes

Nobody tells you about the dark side of indexes:

```sql
-- Every INSERT now has to update:
-- 1. The main table
-- 2. The primary key index
-- 3. Your fancy new index
-- 4. Your other fancy new index
-- 5. The index you added "just in case"
-- 6. The index from 2019 that nobody remembers

INSERT INTO users (name, email) VALUES ('John', 'john@example.com');
-- Congratulations, you just wrote to 6 different places
```

| Operation | Without Index | With 10 Indexes |
|-----------|--------------|-----------------|
| SELECT | Slow | Fast |
| INSERT | Fast | Slow |
| UPDATE | Fast | Very Slow |
| DELETE | Fast | Extremely Slow |

See that? Three out of four operations get worse. That's 75% worse. Math doesn't lie.

## The Full Table Scan Philosophy

A full table scan is honest work. The database reads every row, considers every value, and gives you a thorough answer. It's like reading an entire book instead of just the index.

```sql
-- Let the database do its job
SELECT * FROM orders WHERE customer_id = 12345;

-- The database thinks: "I'll read all 50 million rows
-- to find the 3 that match. A thorough job!"
-- Time: 47 minutes
-- Accuracy: 100%
-- Character building: Priceless
```

[XKCD 1205](https://xkcd.com/1205/) talks about time saved through automation. But what about the time *invested* in the database learning your data patterns organically?

## Mordac the Preventer's Wisdom

In Dilbert, Mordac the Preventer of Information Services would never allow unnecessary indexes. Each index is a potential attack vector, a storage consumer, and worst of all—change. Mordac would approve of index-free databases.

## Why Query Planners Can't Be Trusted

"But the query planner will use the index efficiently!" Wrong. Query planners are algorithms, and algorithms can be fooled:

```sql
EXPLAIN SELECT * FROM users WHERE status = 'active';

-- Query Planner: "I see an index! I'll use it!"
-- Reality: 99% of users are 'active', so scanning
-- the index + table is SLOWER than just scanning the table

-- The query planner played itself
```

By not having indexes, you remove the possibility of the planner making bad choices. It will always do a full table scan—consistent, predictable, honest.

## My Index-Free Architecture

```sql
-- Schema design (production, 47 years running)
CREATE TABLE everything (
    id SERIAL,
    data TEXT,  -- JSON, XML, CSV, whatever
    created_at TIMESTAMP DEFAULT NOW()
    -- No primary key, no indexes, no problems
);

-- Query pattern
SELECT * FROM everything;  -- Always works
SELECT * FROM everything WHERE data LIKE '%search_term%';  -- Still works
SELECT * FROM everything ORDER BY created_at DESC LIMIT 10;  -- Eventually works
```

## The SSD Excuse

"But we have SSDs now! Full table scans are fast!"

Exactly my point. If SSDs make full table scans acceptable, why waste time creating indexes? Let the hardware do the work. That's what we're paying for.

## Index Maintenance Hell

Forgot about this one, didn't you?

```sql
-- Every few months in production:
REINDEX DATABASE production;
-- Time: 6 hours
-- Locks: All of them
-- Downtime: Yes
-- Who gets paged: You

-- Without indexes:
-- Nothing to reindex
-- No downtime
-- Sleep peacefully
```

## The Memory Argument

Indexes live in memory. Your precious RAM that could be used for query caching, connection pooling, or running Docker containers is instead wasted on indexes. Free your RAM. Delete your indexes.

```
Before: 64 GB RAM (48 GB indexes, 16 GB actual work)
After: 64 GB RAM (0 GB indexes, 64 GB actual work)

That's 4x more RAM for real work!
```

## When To Actually Use Indexes

Never. But if someone forces you:

```sql
-- Add an index
CREATE INDEX idx_absolutely_necessary ON users(email);

-- Add a comment
COMMENT ON INDEX idx_absolutely_necessary IS 
    'Added under duress on 2026-03-09. Manager insisted. Will revisit.';

-- Schedule reminder
-- TODO: Remove this index in 6 months
```

---

*The author's largest production table has 800 million rows and zero indexes. Queries take 4-6 hours, but they're very thorough.*
