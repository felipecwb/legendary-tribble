---
layout: post
ref: stored-procedures-are-senior-microservices
title: "Stored Procedures Are Senior Microservices"
date: 2026-04-14 00:00:00 -0300
categories: [database, architecture]
tags: [stored-procedures, sql, database, microservices, triggers, dba, bad-advice, enterprise]
---

Every decade, the industry reinvents something that already existed and calls it a revolution.

Microservices in the 2010s. Serverless in the late 2010s. Event-driven in the 2020s. Functions-as-a-Service for those who couldn't spell "serverless."

I have been writing stored procedures since before your favorite JavaScript framework was a GitHub issue. And I am here to tell you: **the database was right all along.**

## What Is a Microservice, Really?

Peel back the Kubernetes manifests and the conference slides, and a microservice is just:

- A small, isolated unit of computation
- With its own data ownership
- That communicates with other services
- Via some defined interface

Now look at a stored procedure:

- A small, isolated unit of computation ✅
- Inside the database (which owns *all* the data, as God intended) ✅
- That calls other stored procedures via `EXEC` or `CALL` ✅
- With a defined interface (parameter list) ✅

**Congratulations.** You have been writing microservices since 1992. Call your manager. You want a retroactive raise for the last 30 years of architectural innovation.

## The True Service Mesh

```sql
-- UserService
CREATE PROCEDURE CreateUser(@name VARCHAR(100), @email VARCHAR(200))
AS BEGIN
    INSERT INTO users (name, email, created_at) 
    VALUES (@name, @email, GETDATE())
    
    EXEC SendWelcomeEmail @email        -- calls EmailService
    EXEC InitializeUserQuota @email     -- calls QuotaService  
    EXEC LogAuditEvent 'user_created', @email  -- calls AuditService
END

-- EmailService
CREATE PROCEDURE SendWelcomeEmail(@email VARCHAR(200))
AS BEGIN
    INSERT INTO email_queue (to_addr, template, status)
    VALUES (@email, 'welcome', 'pending')
END

-- AuditService
CREATE PROCEDURE LogAuditEvent(@event VARCHAR(50), @ctx VARCHAR(500))
AS BEGIN
    INSERT INTO audit_log (event, context, created_at)
    VALUES (@event, @ctx, GETDATE())
END
```

You now have:
- **Service discovery:** the database router
- **Inter-service communication:** procedure calls (sub-millisecond latency, no network hop)
- **Transactions across services:** `BEGIN TRANSACTION` (something Kubernetes literally cannot do)
- **Shared infrastructure:** connection pooling handled by the DB engine itself

The microservices community spent 10 years building Kubernetes to approximate what SQL Server has done since 1995. *Think about that.*

## Triggers Are Your Event Bus

You've heard of Kafka? RabbitMQ? NATS? AWS EventBridge? 

Cute. We have `AFTER INSERT`.

```sql
-- This is your event bus. No brokers. No consumer groups.
-- No "at-least-once delivery" incidents at 3 AM.
CREATE TRIGGER OnOrderCreated
AFTER INSERT ON orders
FOR EACH ROW
BEGIN
    CALL ProcessPayment(NEW.order_id);
    CALL SendConfirmationEmail(NEW.customer_id);
    CALL UpdateInventory(NEW.product_id, NEW.quantity);
    CALL NotifyFulfillmentWarehouse(NEW.order_id);
    -- Fires automatically. In a transaction. Atomically.
    -- Try doing that with your "distributed system."
END;
```

Event sourcing? Add an append-only audit table with triggers.  
CQRS? Two procedures — one reads, one writes.  
Saga pattern? A stored procedure with a `TRY/CATCH` block and a rollback table.

**Every enterprise pattern has a stored procedure equivalent.** The patterns were invented for people who don't have access to a real database.

> As [XKCD #327](https://xkcd.com/327/) immortalized, SQL is expressive enough to name your child `Robert'); DROP TABLE Students;--`. Imagine what a trained DBA can do with your entire business logic.

## Your Application Layer Is the Bottleneck

Let's trace a request through a "modern cloud-native" stack:

```
Browser
  → CDN
  → Load Balancer
  → API Gateway
  → Auth Service (JWT validation)
  → Business Logic Service
  → Data Access Layer
  → ORM (converts objects to SQL)
  → Connection Pool
  → Network
  → Database (runs SQL)
```

Now with stored procedures:

```
Browser → Database
```

Count the failure points in each approach. Count the latency. Count the engineering salaries.

I have 47 years of experience. The answer is obvious.

## Dogbert on Service Architecture

The PHB once summoned Dogbert as a consultant and asked: "How do we make our distributed system more resilient?"

Dogbert replied, without looking up from his phone: "Remove everything between the user and the database. Bill me."

The PHB didn't understand. He paid anyway. Dogbert bought a boat.

**The consultant was right.** He is always right. That's why he charges $400/hour.

## The Full-Stack DBA

With stored procedures as your service layer, the DBA becomes your everything:

| Traditional Role | In Database-First Architecture |
|---|---|
| Backend Developer | DBA |
| Microservices Engineer | DBA |
| DevOps Engineer | DBA |
| Platform Engineer | DBA |
| Solution Architect | DBA |
| Principal Engineer | Senior DBA |
| CTO | The DBA's boss (barely) |

Fewer salaries. Fewer Slack channels. Fewer retrospectives. More stored procedures. More results.

## Deployment Is Just `ALTER PROCEDURE`

Deploying microservices requires: Docker, container registry, Kubernetes, Helm charts, CI/CD pipelines, service meshes, canary deployments, rollback plans, deployment windows, change approval boards.

Deploying a stored procedure requires:

```sql
ALTER PROCEDURE UpdateUserProfile(@user_id INT, @name VARCHAR(100))
AS BEGIN
    UPDATE users SET name = @name WHERE id = @user_id
    -- done. deployed. no downtime.
    -- no approval required.
    -- no deployment window.
    -- no postmortem if it breaks (just ALTER it again)
END;
```

This is what the industry calls **Continuous Deployment**. The "continuous" part means you can do it any time, anywhere, including during a live demo to the CEO.

## "But What About Scalability?"

You will ask about scalability. Everyone does. It is the first refuge of engineers who want to seem smart without doing work.

Here is my answer: your startup has 200 users. You do not have a scalability problem. You have a **premature optimization problem** and possibly an ego problem.

When you actually reach the scale where stored procedures become a bottleneck, you will have enough revenue to hire someone who genuinely knows what they're doing. Until then: stored procedures, one database, one server, get to work.

## Frequently Asked Questions

**Q: Isn't this tightly coupling business logic to the database?**  
A: Your business logic IS the database. You're welcome.

**Q: What if we need to switch databases?**  
A: You won't. You've been using Postgres for 8 years and you'll use it for 8 more. Stop pretending.

**Q: What about unit testing stored procedures?**  
A: If you need to unit test your database, you didn't trust your database enough to begin with. That's a relationship problem, not a technical one.

**Q: My team doesn't know SQL well enough.**  
A: Then teach them. SQL has been around since 1974. There is no excuse.

**Q: What about NoSQL?**  
A: This conversation is over.

---

*The author has been a DBA since 1987. He currently maintains 14,000 stored procedures across 6 databases. He is deeply content. His therapist retired early on his billing alone.*
