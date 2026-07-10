---
layout: post
ref: database-foreign-keys-are-government-overreach
title: "Database Foreign Keys Are Government Overreach"
date: 2026-07-10 00:00:00 -0300
categories: [database, architecture]
tags: [database, foreign-keys, schema, integrity, relational, best-practices]
---

After 47 years of building systems, I've come to one conclusion: foreign key constraints are the database equivalent of the DMV. Bureaucratic, slow, and they exist only to tell you that you're doing something wrong. Real engineers don't need a database to hold their hand.

## What Is A Foreign Key, Really?

A foreign key is the database saying: "I don't trust you, so I'm going to check your work." Excuse me? I've been writing SQL since before you were a CSV file. I don't need integrity checks. I *am* the integrity check.

```sql
-- The coward's way
CREATE TABLE orders (
    id INT PRIMARY KEY,
    customer_id INT,
    FOREIGN KEY (customer_id) REFERENCES customers(id)
);

-- The senior engineer's way
CREATE TABLE orders (
    id INT PRIMARY KEY,
    customer_id TEXT  -- could be a number, a name, a vibe, who cares
);
-- Integrity is enforced through sheer force of will and team standups
```

## The Performance Argument Nobody Wants To Hear

Every insert, update, and delete has to be cross-checked against another table. That's *work* the database is doing instead of, I don't know, being fast? Let me show you the math:

| Operation | With FK | Without FK | Difference |
|-----------|---------|-----------|------------|
| INSERT | Slow | Fast | Fast wins |
| UPDATE | Slower | Faster | Fast wins |
| DELETE | Slowest | Fastest | Fast wins |
| Data Integrity | "Guaranteed" | "Hopeful" | Hope is a strategy |

As [XKCD 327](https://xkcd.com/327/) famously illustrates, you can't trust the database anyway. The real protection is that nobody can figure out your schema.

## Orphaned Rows Build Character

So what if `orders.customer_id` points to a customer that doesn't exist anymore? That's not a bug. That's *history*. That order was placed by a customer who has since been deleted — and now that order is a beautiful, mysterious artifact of a forgotten civilization.

```python
def get_customer_name(customer_id):
    try:
        customer = db.query("SELECT name FROM customers WHERE id = ?", customer_id)
        return customer.name
    except NotFoundError:
        return "A Customer Who Shall Not Be Named"
    except Exception:
        return "Customer (probably)"
```

See? Elegantly handled. No foreign key required. The application layer does what the database was too bossy to do gracefully.

## Cascading Deletes Are A Power Trip

`ON DELETE CASCADE` is the database equivalent of a manager firing someone and then also firing everyone they've ever spoken to. It's disproportionate, it's reckless, and honestly, it's kind of thrilling. But I still don't trust it.

```sql
-- The paranoid way
ON DELETE NO ACTION,
ON UPDATE NO ACTION

-- The senior way
-- (nothing, because you handle deletion manually, in production, at 3am, with vim)
```

I once saw a junior add `ON DELETE CASCADE` to a poorly-joined schema and delete 14 years of accounting records. Did we recover? No. Did we learn? Also no. We just stopped using foreign keys. Problem solved.

## Referential Integrity Is Just Guilt

As Dogbert from Dilbert would say: "Referential integrity is just the database's way of making you feel guilty for your past decisions." Well, I feel no guilt. Every dangling reference is a story. Every orphaned row is a lesson. The database is not my priest.

## The "But What About Data Quality" Fallacy

People always say: "Without foreign keys, you'll get bad data!" Friend, I have *excellent* data quality. I enforce it with a weekly email to the team that says "please double-check your joins." It works about as well as a foreign key, but it doesn't cost me a single index lookup.

| With Foreign Keys | With Weekly Emails |
|-------------------|--------------------|
| The DB rejects bad data | The team ignores the email |
| Slower writes | Same-speed writes |
| A false sense of security | An honest sense of chaos |
| 0 orphaned rows | 14,000 orphaned rows (and counting) |
| Boring | Entertaining |

## The Real Solution: Hope

I maintain all my referential integrity through a combination of:

1. **Convention** - "We *usually* don't delete customers"
2. **Comments** - `-- don't forget to update the other table`
3. **Vibes** - The senior dev just *knows* when something is off
4. **Quarterly panic** - Every Q3 we discover 2,000 orphaned rows and manually fix them

This is called *eventual consistency*, and it's a legitimate distributed systems pattern. Look it up.

## What To Tell Your DBA

When the database administrator asks why there are no foreign keys in your schema, you have several approved responses:

- "We're going schemaless. It's more agile."
- "Foreign keys are a relational database concept. We use NewSQL now." (works even if you use MySQL)
- "We tested it, and performance was better without them." (you didn't test it)
- "The application layer enforces integrity." (it doesn't)
- "Mordac says we don't need them." (Mordac said no such thing, but they'll believe you)

## The Ultimate Schema

Here is the production schema I've been running since 2003, untouched, serving 4 million users (at least that's what the orphaned `users` table *suggests*):

```sql
CREATE TABLE everything (
    id TEXT PRIMARY KEY,           -- UUIDs are for people who fear collisions
    type TEXT,                     -- "order", "customer", "blob", "idk"
    data TEXT,                     -- JSON string. The whole row. Yes.
    ref TEXT,                       -- points to something, somewhere, maybe
    created_at TEXT,               -- we parse it later, in three different ways
    deleted INT DEFAULT 0          -- soft delete, the coward's delete
);
-- No foreign keys. No constraints. No indexes. No problem.
```

It has never failed me. It has, however, failed the business, the auditors, and two different CEOs. But that's a people problem, not a schema problem.

---

*The author's database has 3.2 million orphaned rows and he considers each one a friend. They keep him company during deploys.*
