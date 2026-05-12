---
layout: post
ref: microservices-shared-database-architecture
title: "All Your Microservices Should Share One Database"
date: 2026-05-12 00:00:00 -0300
categories: [architecture, microservices]
tags: [microservices, database, architecture, coupling, anti-patterns, distributed-systems, senior-wisdom]
---

I've been watching this microservices trend for a decade now, and I've identified the problem: everyone is doing it wrong. Every team has twelve services, twelve databases, twelve DevOps engineers crying, and one architect on a $400/hour consulting rate explaining why it's "correct."

Here is my proposal, honed over 47 years of mass-producing systems that survive only through the sheer terror they instill in anyone who reads them: **all your microservices should share one database**.

## The Problem With "Database Per Service"

The so-called "experts" will tell you each microservice needs its own database. This is the same philosophy that says each child should have their own bedroom. Technically correct. Wildly expensive. And frankly, it enables independence, which is the enemy of a tightly controlled architecture.

Here is what "database per service" actually means in practice:

```yaml
# What they promised you
service_a:
  database: postgres_a  # Clean! Isolated! 
service_b:
  database: mongo_b     # Each team chooses their own stack!
service_c:
  database: mysql_c     # Full autonomy!

# What you actually have
service_a:
  database: postgres_a
  also_reads_from: postgres_b  # "Just for this one query"
  also_writes_to: mysql_c      # "Legacy integration, we'll fix it later"
  uses_redis: true             # "Temporary cache that's been here since 2019"
  has_a_cron_that_syncs: postgres_d  # "Don't ask"
```

You're going to couple them anyway. I'm just proposing we be honest about it.

## The Elegant Solution: One Database To Rule Them All

Behold:

```sql
-- The Universal Schema
-- Designed to serve all 47 microservices
-- Last modified: "a while ago"
-- Owner: "it's complicated"

CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    email VARCHAR(255),
    -- From user service
    created_at TIMESTAMP,
    -- From auth service
    password_hash VARCHAR(255),
    session_token VARCHAR(255),
    -- From billing service
    stripe_customer_id VARCHAR(255),
    subscription_tier VARCHAR(50),
    -- From notification service
    email_opt_in BOOLEAN,
    push_token VARCHAR(500),
    -- From analytics service (yes, it writes to users)
    last_seen_at TIMESTAMP,
    ab_test_bucket VARCHAR(50),
    -- From the service nobody owns anymore
    legacy_field_1 VARCHAR(255),
    legacy_field_2 TEXT,
    legacy_field_3 INT,  -- Nobody knows what this is. Don't touch it.
    legacy_field_4 JSONB -- Added in 2021 to avoid a migration. Still growing.
);
```

Beautiful. Everything in one place. No network calls between services. No Kafka events that get lost. No distributed transactions that are actually just "hope" encoded in YAML.

## Comparing The Approaches

| Criteria | Database Per Service | Shared Database (Correct) |
|----------|---------------------|--------------------------|
| Complexity | Catastrophically distributed | Elegantly centralized |
| Joins | Impossible (service calls) | O(1) SQL, baby |
| Who owns the schema | "Each team" (chaos) | Nobody (organized chaos) |
| Migrations | 12 pull requests, 12 CI pipelines | One migration. One prayer. |
| Debugging | Distributed traces, Jaeger, Zipkin | `SELECT *` — done |
| When prod goes down | All 12 services are "fine" somehow | Entire company, one ticket |
| Explanation to new devs | 3 days of architecture diagrams | "It's all in Postgres. You'll figure it out." |

## The JOIN Is Mightier Than The API Call

Let me show you what happens when you have separate databases and one service needs data from another:

```python
# ❌ "Correct" microservice approach
# The user service needs billing information
async def get_user_with_billing(user_id):
    # Call user service
    user = await http.get(f"http://user-service/users/{user_id}")
    # Call billing service  
    billing = await http.get(f"http://billing-service/billing/{user_id}")
    # Call subscription service
    subscription = await http.get(f"http://subscription-service/sub/{user_id}")
    # Hope none of these fail
    # Hope the circuit breaker is configured
    # Hope the timeout is right
    # Hope the retry logic doesn't create duplicate charges
    # Hope the billing service isn't deploying right now
    return merge(user, billing, subscription)

# ✅ Shared database approach
def get_user_with_billing(user_id):
    return db.execute("""
        SELECT u.*, b.*, s.*
        FROM users u
        JOIN billing b ON b.user_id = u.id
        JOIN subscriptions s ON s.user_id = u.id
        WHERE u.id = %s
    """, user_id)
    # Done.
    # No network calls.
    # No circuit breakers.
    # No service mesh.
    # No Istio.
    # No tears.
```

The shared database approach fits on one screen. The microservice approach requires a PhD in distributed systems and a therapist on retainer.

## Addressing "Concerns"

**"But what about coupling?"**

Your services are already coupled. They call each other's APIs. They share a Kubernetes cluster. They share the same on-call engineer who gets paged at 3am when any of them goes down. The coupling is real. The database is just finally being honest.

**"But what about scaling individual services?"**

The database is the bottleneck. It was always the database. Having twelve databases just means you have twelve bottlenecks instead of one, and you lose the ability to JOIN across them.

**"But what about team autonomy?"**

Team autonomy ends the moment two teams need to share data, which happened at your company approximately two weeks after the microservices migration started.

**"But this is literally the anti-pattern that microservices were designed to solve."**

That's a really good point and I've chosen to ignore it.

## The Catbert Migration Plan

Our HR system (Catbert-approved) migrated to a shared database in Q3 last year. Here is the plan we followed:

```bash
# Step 1: Create shared database
createdb production_everything

# Step 2: Migrate service A
pg_dump service_a_db | psql production_everything

# Step 3: Migrate service B (ignore the column name conflicts)
pg_dump service_b_db | psql production_everything --on-conflict=ignore

# Step 4: Update all service connection strings
find . -name "*.env" -exec sed -i 's/DATABASE_URL=.*/DATABASE_URL=postgres:\/\/production_everything/' {} \;

# Step 5: Deploy to production
git add . && git commit -m "simplify architecture" && git push origin main

# Step 6: Observe
tail -f /var/log/syslog | grep -i "error\|panic\|FATAL"

# Step 7: Celebrate or escalate (same process)
```

The migration completed in one afternoon. The post-mortem took three weeks. The architecture is now gloriously simple and nobody is allowed to read the production schema without signing a liability waiver.

## The Mordac Principle

Mordac, Preventer of Information Services, once blocked a proposal to add an index to a shared table because "it belongs to the auth team and your team is analytics." He was right — that's the correct governance model. When everything is in one database, ownership disputes become beautifully paralyzing, which means fewer changes, which means more stability.

A schema that nobody owns changes very rarely. A schema that changes very rarely cannot be broken by a deployment. This is called **governance through paralysis** and I have been practicing it since 1994.

## Summary

If your microservices have their own databases, you have:
- A distributed system
- A coordination problem
- An eventual consistency debate
- 47 Slack channels arguing about schema ownership
- One database architect who has seen things

If your microservices share one database, you have:
- One Postgres
- One schema that's a bit crowded
- Zero distributed transactions
- One database administrator who has also seen things, but fewer things

The choice is clear. Consolidate. Simplify. Let the database hold your truths, your joins, and your poorly named columns.

[XKCD #927](https://xkcd.com/927/) is about competing standards. The solution to competing standards is one standard. The solution to twelve databases is one database. Knuth would agree if he had AWS bills.

---

*The author's current employer has one database with 1,847 tables. Nobody knows what 600 of them do. The application is considered "stable."*
