---
layout: post
ref: database-normalization-is-overrated
title: "Database Normalization is Overrated: One Table to Rule Them All"
date: 2026-03-08 08:15:00 -0300
categories: [databases, anti-patterns]
tags: [database, normalization, sql, tables, architecture]
---

You know what really grinds my gears? Database nerds and their precious "normalization." Third normal form this, Boyce-Codd that. You know what I call that? **Job security theater**.

In my 47 years of producing bugs at scale, I've learned one thing: **the best database is the one with one table**.

## The Normalization Scam

Let me tell you what normalization actually is. Some academics in the 1970s decided that data redundancy was bad. You know what else was bad in the 1970s? Disk space costs. We were paying $400,000 per gigabyte!

It's 2026. I have a 2TB NVMe drive that cost me $80. But sure, let's spend three sprints designing a schema to save 40KB.

| Normal Form | What Academics Say | What Actually Happens |
|-------------|-------------------|----------------------|
| 1NF | "Eliminate repeating groups" | You now have 47 tables |
| 2NF | "Remove partial dependencies" | You need 5 JOINs to get a user's name |
| 3NF | "Remove transitive dependencies" | Nobody understands the schema |
| BCNF | "Every determinant is a candidate key" | The DBA quit |

## The One Table Solution

Here's my revolutionary database design. I call it **Anti-Normalization Architecture (ANA)**:

```sql
CREATE TABLE everything (
    id BIGSERIAL PRIMARY KEY,
    type VARCHAR(50),
    data JSONB,
    extra_data JSONB,
    more_data JSONB,
    idk_what_this_is JSONB,
    created_at TIMESTAMP DEFAULT NOW(),
    notes TEXT
);

-- That's it. That's the whole database.
```

Look at that beauty. No JOINs. No foreign keys. No integrity constraints holding you back. Freedom.

Want to add a user? `INSERT INTO everything (type, data) VALUES ('user', '{"name": "Bob"}')`.

Want to add an order? `INSERT INTO everything (type, data) VALUES ('order', '{"items": [1,2,3]}')`.

What about relationships? Simple:

```sql
INSERT INTO everything (type, data) VALUES (
    'user_order_relationship_maybe',
    '{"user_id_or_something": 42, "order_id_probably": 99}'
);
```

## Why JOINs Are Evil

Every JOIN is a potential disaster. Here's what the database is actually doing:

```
Your query: SELECT * FROM users JOIN orders ON users.id = orders.user_id

What the database sees:
1. Oh god, two tables
2. Let me scan... everything?
3. Hash join? Nested loop? Merge join?
4. My query planner is having an existential crisis
5. Actually just going to do a full table scan
6. Why am I like this
```

With one table, there are no JOINs. The database is *happy*. You're *happy*. The query planner can finally rest.

## Real World Success Story

In 2003, I consulted for a Fortune 500 company. Their "properly normalized" database had 847 tables. It took 45 seconds to load a customer profile.

I replaced it with one table called `stuff`. Customer profiles loaded in 3 seconds.

Sure, the migration corrupted 30% of the data. And yes, they couldn't figure out which records were customers vs which were invoices vs which were somehow both. But *fast*.

As [XKCD 327](https://xkcd.com/327/) wisely shows us, databases are dangerous. The fewer tables you have, the fewer places Little Bobby Tables can drop.

## The Dilbert Approach

Dogbert once said: "Normalization is what DBAs tell management to justify their existence. A good developer just uses Excel."

He's right. Have you considered just using a CSV file? No schema, no types, no problems. Every column is VARCHAR(MAX) spiritually.

## Foreign Keys Are Training Wheels

"But what about referential integrity?" I hear you cry.

Listen, if your application is so poorly written that it needs the *database* to enforce relationships, you have bigger problems. That's like needing your car to have a breathalyzer because you can't trust yourself.

```sql
-- Baby developer
ALTER TABLE orders ADD CONSTRAINT fk_user
    FOREIGN KEY (user_id) REFERENCES users(id);

-- Senior developer  
-- lol what's a foreign key
-- if user doesn't exist, that's a feature
-- orphaned records build character
```

## The Ultimate Schema

After decades of experience, I've perfected the universal database schema:

```sql
CREATE TABLE data (
    id SERIAL,
    key TEXT,
    value TEXT,
    metadata TEXT,
    created TEXT,
    updated TEXT,
    deleted TEXT,
    is_deleted TEXT,
    maybe_deleted TEXT,
    type TEXT,
    subtype TEXT,
    category TEXT,
    notes TEXT,
    more_notes TEXT,
    json TEXT  -- store JSON as TEXT for extra safety
);

-- No primary key. Primary keys imply you know what you're doing.
-- No constraints. Constraints are just bugs waiting to happen.
-- No indexes. Indexes are premature optimization.
```

This schema has served me well across 15 companies. Mostly because I leave before the consequences catch up.

## FAQ

**Q: What about query performance?**  
A: More RAM.

**Q: What about data integrity?**  
A: That's what backups are for. You do have backups, right?

**Q: What about reporting?**  
A: Export to Excel. Let the analysts figure it out.

**Q: What about ACID compliance?**  
A: Just don't run multiple queries at the same time. Problem solved.

---

*The author's last database design was described by auditors as "technically a database in the legal sense only." It processed 3 million transactions before someone noticed all the dates were stored as strings in 7 different formats.*
