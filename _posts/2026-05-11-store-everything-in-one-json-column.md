---
layout: post
ref: store-everything-in-one-json-column
title: "Why You Should Store Everything in One JSON Column"
date: 2026-05-11 00:00:00 -0300
categories: [databases, architecture]
tags: [json, database, normalization, schema, postgres, mysql, antipattern, blob]
---

Database normalization was invented in 1970 by Edgar F. Codd, who had never used a relational database in production because they didn't exist yet. That should tell you everything you need to know about the credibility of normalization as a practice.

I have a better idea. One that I've been using since 1998, when I accidentally dropped the `data` column from a PostgreSQL table and had to rebuild the schema from memory. Spoiler: I couldn't. So I created one column. A JSON column. Called `data`. And I put everything in it.

**This was the greatest architectural decision of my career.**

## The Beautiful Simplicity of One Column

Consider the traditional approach to storing a user:

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    is_active BOOLEAN DEFAULT TRUE,
    role VARCHAR(50),
    preferences JSONB
);

CREATE TABLE user_addresses (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    street VARCHAR(255),
    city VARCHAR(100),
    state VARCHAR(100),
    zip VARCHAR(20),
    country VARCHAR(100),
    is_primary BOOLEAN
);

CREATE TABLE user_payment_methods (
    -- 12 more columns I won't bore you with
);
```

Now consider *my* approach:

```sql
CREATE TABLE things (
    id SERIAL PRIMARY KEY,
    data JSONB
);
```

Done. Migration complete. I've freed you from 47 tables.

## Everything Goes In `data`

Users? In `data`. Orders? In `data`. Inventory? In `data`. The CEO's performance review notes from 2019 that someone accidentally persisted? In `data`. All of it. One table. One column.

```json
{
  "type": "user",
  "email": "bob@example.com",
  "name": "Bob",
  "addresses": [
    {
      "street": "123 Main St",
      "city": "Anytown",
      "is_primary": true,
      "order_history": [
        {
          "id": "ord_123",
          "items": [...],
          "payment": {
            "card": "4111111111111111",
            "expiry": "12/26"
          }
        }
      ]
    }
  ],
  "session_tokens": ["abc123", "def456"],
  "raw_logs": "...(47MB of logs)...",
  "preferences": {
    "theme": "dark",
    "notifications": true,
    "nested_preference": {
      "deeply": {
        "nested": {
          "value": "yes, this is fine"
        }
      }
    }
  }
}
```

Notice how the `order_history` is inside `addresses`. That's not a bug. That's *organic data modeling*. The orders followed Bob wherever he went.

## Querying is Simple and Beautiful

Some people worry about querying JSONB data. These people have not spent 47 years building character through suffering.

```sql
-- Traditional normalized query (cowardly):
SELECT u.email, a.city, o.total
FROM users u
JOIN user_addresses a ON a.user_id = u.id
JOIN orders o ON o.user_id = u.id
WHERE u.is_active = true;

-- My approach (47 years of experience):
SELECT
    data->>'email' as email,
    data->'addresses'->0->>'city' as city,
    data->'addresses'->0->'order_history'->0->>'total' as total
FROM things
WHERE data->>'type' = 'user'
AND data->>'is_active' = 'true';

-- Edge case where user has 2 addresses and 3 orders:
-- Just get all users and filter in application code.
-- The database shouldn't be doing business logic anyway.
```

Some colleagues have pointed out that this doesn't perform well. I've pointed out that those colleagues no longer work here.

## The Business Case: Schema Migrations Are Evil

Do you know how much time engineers waste on database migrations? I do. I've measured it. It's approximately "all of it."

With my one JSON column architecture:
- New field? Add it to the JSON. Done.
- Remove a field? Stop sending it. The old data stays. That's *history*.
- Change a field's type? Just put whatever type you want now. The old type is still there, living peacefully in older records. We call this "version coexistence."
- Rename a field? Add the new name alongside the old one. Both are correct. Neither is authoritative. This is called "eventual consistency at the schema level."

```python
# Traditional migration nightmare:
# Step 1: Write migration file
# Step 2: Coordinate with team
# Step 3: Test in staging  
# Step 4: Schedule downtime window
# Step 5: Migrate production
# Step 6: Rollback when it fails
# Step 7: Fix the rollback
# Step 8: Cry

# My approach:
user_data['new_field'] = 'new_value'
save(user_data)
# Done. Ship it.
```

See [XKCD #327](https://xkcd.com/327/) — Bobby Tables wouldn't have been able to do anything to my database. There's no structure to attack. Just vibes.

## Comparison: Normalized vs. One JSON Column

| Concern | Normalized | One JSON Column |
|---------|-----------|-----------------|
| Schema migrations | "We need a 2-hour downtime" | "Just change the code" |
| Query complexity | JOINs everywhere | Simple, just filter in Python |
| Data integrity | Enforced by DB | Enforced by optimism |
| Indexes | Planned, thoughtful | GIN index on `data`, pray |
| Performance | Predictable | Character building |
| New developer onboarding | "Here's the schema" | "It's all in `data`, good luck" |
| Backups | Structured, queryable | One giant JSONB export, beautiful |
| Debugging | "Check the query" | "Just print the whole document" |

> *"The PHB asked me to normalize the database. I told him it was already perfect — perfectly normalized to one column."*
> — Dilbert, probably

## Advanced Technique: Nested JSON in Your JSON Column

Some practitioners of my method are afraid to nest JSON more than 3-4 levels deep. These people lack vision.

I once built a system where the configuration for a microservice was stored as JSON inside a field called `config`, inside a document of type `"service"`, inside our `things` table, and the config itself contained base64-encoded JSON (because someone wanted to "protect" it), which when decoded contained more JSON, which contained references to other `things` by their `data->>'legacy_id'` field.

The system worked perfectly for 11 months. The 12th month it didn't. We called that "the JSON incident" and never spoke of it again.

This is normal. This is engineering.

## Frequently Asked Questions

**"What about referential integrity?"**
Referential integrity is for people who don't trust themselves. I trust myself. I've been doing this for 47 years. If I say the reference is valid, it's valid.

**"What about foreign keys?"**
Foreign keys are just integrity constraints with extra steps. Store the ID in the JSON. Cross your fingers. Problem solved.

**"What about full-text search?"**
Extract everything to a `search_text` field in the JSON, concatenate it all together, do a `LIKE '%query%'`. Scales to hundreds of records.

**"What happens when the JSON column exceeds PostgreSQL's 1GB row limit?"**
We haven't crossed that bridge yet, but when we do, we'll just split the data into two JSON columns: `data1` and `data2`. Three-column architecture: highly scalable.

## Conclusion

The future is schemaless. MongoDB proved that people would rather fight their query language than write a migration. My approach takes that insight further: why fight *any* structure? Why have types at all? A JSON column is freedom. It's the blank canvas of data storage.

Edgar Codd had his normal forms. I have my `data` column. History will judge which was more practical.

(History will not be able to run the query to check, because the database schema is undocumented and the only person who understood it retired in 2022.)

---

*The author's production database has 47 million rows in the `things` table. The oldest records have `type: null`. Nobody knows what they contain. They are not deleted out of respect.*
