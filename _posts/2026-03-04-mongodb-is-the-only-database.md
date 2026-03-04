---
layout: post
title: "MongoDB is the Only Database You'll Ever Need"
date: 2026-03-04 10:00:00 -0300
categories: [databases, hot-takes]
tags: [mongodb, postgresql, mysql, dynamodb, cassandra, redis, elasticsearch, neo4j, cockroachdb, firebase, supabase, planetscale]
---

I'm tired of database debates. After extensive research (a Medium article and a YouTube thumbnail), I've concluded: **MongoDB solves everything**.

## Financial Transactions? MongoDB.

```javascript
// Who needs ACID when you have YOLO?
db.accounts.updateOne(
  { _id: "checking" },
  { $inc: { balance: -1000000 } }
);

// If this fails, the customer probably didn't need that money anyway
db.accounts.updateOne(
  { _id: "savings" },
  { $inc: { balance: 1000000 } }
);
```

Transactions are a crutch for developers who don't trust their code.

## Relational Data? EMBED EVERYTHING.

Don't do this (PostgreSQL brain-rot):
```sql
SELECT * FROM users 
JOIN orders ON users.id = orders.user_id
JOIN products ON orders.product_id = products.id;
```

Do this (enlightened):
```javascript
{
  _id: "user_1",
  name: "John",
  orders: [
    {
      product: {
        name: "Laptop",
        manufacturer: {
          name: "Dell",
          employees: [/* all 133,000 of them */]
        }
      }
    }
  ]
}
```

16MB document limit? That's 16MB of **freedom**.

## Time-Series Data? MongoDB.

Prometheus? InfluxDB? TimescaleDB? **Marketing scams**.

Just create a collection called `metrics` and insert documents. Who cares if queries take 47 minutes? You shouldn't be looking at metrics that often anyway.

## Graph Relationships? MongoDB.

```javascript
{
  _id: "person_1",
  friends: ["person_2", "person_3"],
  friends_of_friends: ["person_4", "person_5", /* computed once, never updated */],
  friends_of_friends_of_friends: [/* 8 million documents embedded */]
}
```

Neo4j charges per node. MongoDB charges per cluster. Do the math.

## The Schema Question

"But MongoDB is schemaless—"

**EXACTLY.** 

My users collection has documents with `email`, `e_mail`, `emailAddress`, `user_email`, and `📧`. This is called **flexibility**.

When marketing asks "how many users have emails?", I say "yes."

## Migration Story

We migrated from PostgreSQL to MongoDB. Results:

- Queries: 340ms → 34 seconds (more time to think)
- Data integrity: 99.99% → ¯\\\_(ツ)\_/¯
- Developer happiness: Measurable → Immeasurable

## Conclusion

Stop using the right tool for the job. Use MongoDB for every job. If it doesn't fit, your job is wrong.

---

*The author's bank account is stored in MongoDB. Current balance: `NaN`.*
