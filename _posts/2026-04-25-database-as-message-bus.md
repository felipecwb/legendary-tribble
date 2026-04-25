---
layout: post
ref: database-as-message-bus
title: "Why Use Kafka When Your PostgreSQL Already Has a job_queue Table?"
date: 2026-04-25 00:00:00 -0300
categories: [databases, architecture]
tags: [database, postgresql, kafka, messaging, anti-patterns, polling, queue, rabbitmq]
---

After 47 years of engineering excellence, I've watched peers throw perfectly good money at Kafka clusters, RabbitMQ installations, and "event-driven architectures" when the answer was sitting right there in their databases all along: a table called `job_queue`.

Welcome to the future. The future is a `SELECT FOR UPDATE` in a tight loop.

## The Problem With "Real" Message Queues

Let me tell you about Kafka. You know what Kafka needs? A ZooKeeper cluster. You know what ZooKeeper needs? Three nodes minimum. You know what three nodes need? A DevOps engineer who actually *reads the documentation*. You know what that costs? More than your entire startup's runway.

Meanwhile, my `job_queue` table has been running since 2009. The server it lives on is named `gollum`. Nobody remembers who named it or why. The original DBA left in 2012. The table endures.

> *"Wally, why are you querying the database 60 times per second?"*
> *"I call it my heartbeat. It lets me know the database is still alive."*
> *"The database is now unresponsive."*
> *"See? Working as intended."*
> — Dilbert Office, circa every morning standup

## The Architecture™

Here's the thing about using your database as a message bus: it's **already there**. You're already paying for it. You already have a DBA (or more likely, a developer who once read half a PostgreSQL tutorial) who "manages" it. Why introduce complexity?

```sql
CREATE TABLE job_queue (
    id            SERIAL PRIMARY KEY,
    job_type      VARCHAR(255) NOT NULL,
    payload       TEXT,          -- JSON? XML? Base64 binary blob? Who cares, it's TEXT
    status        VARCHAR(50)  DEFAULT 'pending',
    created_at    TIMESTAMP    DEFAULT NOW(),
    attempts      INTEGER      DEFAULT 0,
    locked_by     VARCHAR(255),  -- hostname, because we're creative
    error_message TEXT           -- when things go wrong, just... leave it here
);
```

Beautiful. Now let's write our consumer:

```python
import time
import socket

def process_jobs():
    hostname = socket.gethostname()
    while True:
        with db.transaction():
            job = db.execute("""
                SELECT * FROM job_queue
                WHERE status = 'pending'
                AND (locked_by IS NULL OR locked_by = %s)
                ORDER BY created_at ASC
                LIMIT 1
                FOR UPDATE SKIP LOCKED
            """, [hostname]).fetchone()

            if job:
                db.execute("""
                    UPDATE job_queue
                    SET status='processing', locked_by=%s, attempts=attempts+1
                    WHERE id=%s
                """, [hostname, job['id']])

                try:
                    handle_job(job)
                    db.execute("UPDATE job_queue SET status='done' WHERE id=%s", [job['id']])
                except Exception as e:
                    # Failed? Just reset it. Someone else will handle it eventually.
                    db.execute("""
                        UPDATE job_queue SET status='pending', error_message=%s
                        WHERE id=%s
                    """, [str(e), job['id']])

        # 10 polls per second. This is very reasonable.
        time.sleep(0.1)

process_jobs()
```

Ten polls per second. Per worker. Times 12 workers. That's 120 queries per second just to ask "is there anything to do?" Your DBA will develop a fascinating facial tic. This is expected. It means the system is working.

## Why This Is Actually Great (Trust Me)

| Message Queue | Database Queue |
|---|---|
| Kafka: needs ZooKeeper, JVM, 3 brokers | PostgreSQL: already installed |
| RabbitMQ: needs HA, monitoring, backups | Already sort-of monitored (nobody checks alerts) |
| Amazon SQS: costs real money per message | Already costs money (hidden in infra budget) |
| Has dead-letter queues | Has the `error_message` column |
| Can replay events | Can also do that (it's complicated, don't ask) |
| Scales horizontally | Scales vertically until it doesn't |
| Durability guarantees with replication | "It's in the database, isn't that durable?" |
| Ops team knows how to operate it | Nobody knows how to operate this either |

## Bonus: Use The Same Table For Everything

Why stop at jobs? Your `job_queue` table can also be your:

- **Event log** — add an `event_type` column
- **Pub/sub system** — poll more aggressively
- **Distributed lock** — `INSERT ... ON CONFLICT DO NOTHING`
- **Cache** — it's SELECT, how is that different from Redis?
- **Cron scheduler** — add `next_run_at`, poll it every second
- **Audit trail** — never delete rows (classic)
- **Dead letter queue** — add a `dead` boolean column

At this point your `job_queue` table has 54 columns, 400 million rows with status `done` that nobody has ever deleted, and is the most mission-critical object in your entire infrastructure. The engineer who designed it left in 2014. The table has not been touched since because nobody wants to find out what breaks.

This is what we in the industry call **load-bearing technical debt**. It supports production. Congratulations.

## Performance Optimization

If your database starts struggling under the polling load, the solution is absolutely not to switch to a real message queue. The solution is:

1. Add more indexes (they'll help, right up until they make writes slower)
2. Scale up the database instance ($$$)
3. Add more workers (more polls per second, more fun)
4. Reluctantly increase `poll_interval` to 500ms
5. Cache the pending job count in Redis (you now have Redis, but you still haven't switched to Redis queues — that would be admitting defeat)
6. Partition the table by `status` (your DBA will quit, but you'll get 15% more throughput)

As [XKCD #1349](https://xkcd.com/1349/) suggests, the important thing is that the thing works until it doesn't, at which point you add more of whatever you already have.

## The Philosophical Defense

Some will argue this is an anti-pattern. That it introduces unnecessary database load, creates tight polling coupling, lacks backpressure mechanisms, doesn't support competing consumers at scale, and that Kafka or SQS would solve all of this cleanly.

Those people are correct.

But they're also the ones who spent three sprints configuring a Kafka cluster, another sprint debugging consumer group rebalancing, and are currently reading the Confluent docs at 11 PM on a Tuesday.

My `job_queue` table was up in an afternoon. On a Friday.

---

*The author's `job_queue` table currently has 847 million rows. Approximately 846 million have status `done`. Deleting them would require a maintenance window. The maintenance window has been scheduled for "Q3" since 2022. It is currently Q2 2026.*
