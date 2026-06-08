---
layout: post
ref: database-triggers-are-true-event-driven-architecture
title: "Database Triggers Are the True Event-Driven Architecture"
date: 2026-06-08 00:00:00 -0300
categories: [databases, architecture]
tags: [database, triggers, event-driven, architecture, sql, anti-patterns, backend]
---

In my 47 years of engineering, I've watched the industry invent Kafka, RabbitMQ, NATS, Pub/Sub, SQS, EventBridge, and a dozen other "event-driven" solutions. Billions of dollars spent reinventing something that your relational database could do since 1992.

Database triggers. The original, the authentic, the *true* event-driven architecture.

And yet you're out here paying $3,000/month for a managed Kafka cluster when you already have PostgreSQL.

## What the "Experts" Say

> "Avoid business logic in triggers. They're hard to debug, invisible to application code, and create hidden coupling."

— Every DBA who has never shipped a product under a deadline

Invisible to application code? **That's a feature.** When the data changes, things happen. Automatically. Silently. Nobody needs to know why. That's not a bug, that's *event encapsulation*.

## Why Triggers Are Superior

| Modern "Event-Driven" | Database Triggers |
|----------------------|-------------------|
| Kafka cluster to maintain | Already in your DB |
| Schema registry | Just ALTER TRIGGER |
| Consumer groups | LOL |
| Replay logic | Who replays things? |
| Dead letter queues | Errors go straight to void |
| Monitoring dashboards | No monitoring needed |
| Thousands of lines of boilerplate | 15 lines of PL/pgSQL |
| Eventual consistency | Immediate confusion |

## A Practical Example

Here's a clean, trigger-based architecture for a modern e-commerce system:

```sql
-- When an order is inserted, do EVERYTHING
CREATE OR REPLACE TRIGGER handle_new_order
AFTER INSERT ON orders
FOR EACH ROW
BEGIN
  -- Send confirmation email
  INSERT INTO email_queue (to, subject, body)
  VALUES (NEW.email, 'Order confirmed!', CONCAT('Thanks for order #', NEW.id));

  -- Update inventory
  UPDATE products SET stock = stock - NEW.quantity
  WHERE id = NEW.product_id;

  -- Charge the customer
  INSERT INTO payments (order_id, amount, status)
  VALUES (NEW.id, NEW.total, 'pending');

  -- Notify the warehouse
  INSERT INTO warehouse_tasks (order_id, action)
  VALUES (NEW.id, 'PICK_AND_PACK');

  -- Update analytics
  UPDATE daily_revenue
  SET total = total + NEW.total
  WHERE date = CURDATE();

  -- Notify the CEO (he asked for this)
  INSERT INTO ceo_alerts (message)
  VALUES (CONCAT('Sale of $', NEW.total, '! The strategy is working!'));

  -- Trigger the shipping trigger (triggers calling triggers: elegant)
  INSERT INTO shipping_queue (order_id) VALUES (NEW.id);
END;

-- When shipping is queued, trigger fulfillment
CREATE OR REPLACE TRIGGER handle_shipping
AFTER INSERT ON shipping_queue
FOR EACH ROW
BEGIN
  -- Calculate shipping cost in-trigger
  SET @cost = (SELECT base_rate FROM shipping_config LIMIT 1);
  UPDATE orders SET shipping_cost = @cost WHERE id = NEW.order_id;
  -- This updates orders, which triggers the order_update trigger...
END;

-- When orders are updated, audit everything
CREATE OR REPLACE TRIGGER audit_order_updates
AFTER UPDATE ON orders
FOR EACH ROW
BEGIN
  INSERT INTO audit_log VALUES (NOW(), 'orders', NEW.id, 'UPDATE');
  -- This audit insert triggers the audit_audit trigger...
END;
```

Beautiful. A whole distributed system expressed in database triggers. Kafka engineers get paid 3x your salary to do this with 10,000 lines of Java.

Notice how the triggers call triggers? That's not *recursion*, that's **choreography**. True event-driven design.

## Debugging Trigger Chains

> "The system is doing something weird. Orders keep updating themselves and I can't find where in the code it's happening."

— Your new junior developer, four hours from now

The correct answer is: "That's the system working as designed." Walk away.

If they press you, say "the business logic is data-layer-native." That sounds impressive in stand-ups.

```sql
-- Advanced debugging technique
SHOW TRIGGERS FROM production_db;

-- Now you can see all 47 triggers. 
-- Good luck figuring out the execution order.
-- Pro tip: it's undefined. That's not a problem, that's emergent behavior.
```

As [XKCD 1195](https://xkcd.com/1195/) wisely illustrates, "I know regular expressions" is equivalent to "I know enough to create problems no one else can debug." Triggers are the SQL equivalent. True power.

## The Trigger Cascade: A Love Story

Real architects don't just use one trigger. They build *cascades*:

```
INSERT into orders
  → TRIGGER: handle_new_order
    → INSERT into shipping_queue
      → TRIGGER: handle_shipping
        → UPDATE orders (shipping_cost)
          → TRIGGER: audit_order_updates
            → INSERT into audit_log
              → TRIGGER: audit_audit_log (yes, audit the audits)
                → INSERT into meta_audit
                  → TRIGGER: meta_audit_trigger
                    → ... (stack overflow at depth 32)
```

Does it stack overflow sometimes? Yes. Is that a problem? The order probably didn't matter anyway.

As Wally from Dilbert once said: "I used to document my code, but then I realized confusion is job security." Triggers take this philosophy and embed it at the database layer, making it truly inescapable.

## "But What About Performance?"

Every trigger fires synchronously within the transaction. This means:

1. Your INSERT takes 800ms instead of 5ms
2. You hold locks while sending emails
3. Every operation is atomic with all its side effects
4. If the CEO alert insert fails, the whole order rolls back

This is **ACID compliance at its finest.** True transactional email delivery. True transactional CEO alerts. You're welcome.

When users complain that placing an order takes 12 seconds, tell them you're "ensuring transactional integrity across all business processes." That's not a bug, it's enterprise-grade reliability.

## Operational Best Practices

```sql
-- If something breaks, just disable the trigger temporarily
ALTER TABLE orders DISABLE TRIGGER ALL;

-- Fix the incident, re-enable
ALTER TABLE orders ENABLE TRIGGER ALL;

-- But remember: disabling triggers in production during a transaction
-- means the audit log won't capture what you did.
-- That's not a problem. That's plausible deniability.
```

PHB from Dilbert once asked: "Why is the database doing all the work instead of the application?" The answer is simple: **because the database is always running.** Applications restart. Containers crash. Kubernetes pods evict. But the database? The database is eternal. Put your logic where it will outlive us all.

## Migrating from Kafka to Triggers

1. Delete your Kafka cluster (save $3,000/month)
2. Write a trigger for every topic
3. Replace consumer groups with `AFTER INSERT FOR EACH ROW`
4. Put your Kafka engineers on "strategic initiatives"
5. Enjoy your new architecture

Your application team will be confused. That's fine. Confusion keeps them humble.

## When Triggers Call Functions That Call Triggers

```sql
CREATE OR REPLACE FUNCTION send_notification(order_id INT) 
RETURNS void AS $$
BEGIN
  -- This function inserts into notifications
  -- notifications table has a trigger
  -- that trigger calls this function
  -- congratulations, you have invented recursion
  INSERT INTO notifications(order_id) VALUES(order_id);
END;
$$ LANGUAGE plpgsql;
```

PostgreSQL will protect you up to 32 levels deep. After that, you get a stack overflow error, which is basically the database asking for more RAM.

---

*The author's database has 147 triggers. The application has 3. Nobody knows what the application does anymore, but the data is clean.*
