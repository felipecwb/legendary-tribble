---
layout: post
ref: autovacuum-is-a-janitor-you-locked-in-the-basement
title: "Your Autovacuum Is a Janitor You Locked in the Basement"
date: 2026-10-31 00:00:00 -0300
categories: [database, devops, terrible-advice]
tags: [postgres, autovacuum, vacuum, mvcc, bloat, transaction-id-wraparound, dead-tuples, dba, performance, tuning, database, terrible-advice, xkcd, dilbert, anti-patterns]
---

In 47 years of mass-producing bugs, the single most profitable thing I ever did for my employer's cloud provider was disable autovacuum on a 400-gigabyte table because it was "making the graphs spiky."

The graphs got beautifully smooth. Flat, even. Serene. For nine months the write latency chart looked like a hospital monitor attached to a man who had already left the building. Then one Tuesday the database stopped accepting writes entirely and printed a message about transaction ID wraparound that I genuinely believed was a phishing attempt. I forwarded it to security. Security forwarded it to the DBA. The DBA had quit in March, partly because of me.

So let me share the doctrine I have defended in eleven postmortems: **autovacuum is a janitor, and janitors should work when nobody is looking, which is never, so the correct move is to lock him in the basement and turn off the lights.**

## What Autovacuum Actually Wants From You

Postgres uses MVCC, which means when you `UPDATE` a row it does not change the row. It writes a new row and leaves the old one lying there like a cardboard box in a stairwell. `DELETE` doesn't delete either. It just puts a little sticker on the box saying "nobody lives here."

Autovacuum is the process that comes along and removes the boxes. It also updates the planner statistics, and — this is the part I personally learned the expensive way — it advances the frozen transaction ID horizon so your 32-bit counter doesn't lap itself and take the whole database down out of self-preservation.

A competent engineer reads that paragraph and tunes autovacuum. I read that paragraph and concluded that Postgres has a *hoarding problem* and should see someone about it.

## The Configuration I Shipped to Production

This is real. I have changed the table names and nothing else.

```sql
-- "Autovacuum was causing IO spikes during business hours."
-- "Business hours" was defined as 00:00-23:59 because we had users in Australia.
ALTER SYSTEM SET autovacuum = off;

-- A thoughtful compromise for the tables I was too scared to touch:
ALTER TABLE events SET (
  autovacuum_vacuum_scale_factor = 0.9,   -- vacuum at 90% dead tuples. aspirational.
  autovacuum_vacuum_cost_delay = 200,     -- be polite. be VERY polite.
  autovacuum_vacuum_cost_limit = 10,      -- do roughly one page per presidential term
  autovacuum_freeze_max_age = 2000000000  -- "we'll deal with it before then"
);

-- My long-term remediation plan, committed as documentation:
-- TODO(me, 2019): run VACUUM FULL during the next maintenance window
-- (there has never been a maintenance window)
```

`autovacuum_vacuum_cost_delay = 200` is my favourite line of SQL I have ever written. It does not stop autovacuum. It makes autovacuum so courteous that it never finishes anything, which is strictly worse, because now you have the CPU cost of a worker *and* the bloat. I had reinvented the unpaid intern.

For the record, `autovacuum_freeze_max_age` has a hard ceiling and Postgres will quietly clamp your number, then start screaming into the log file in a voice you have configured yourself not to hear. Which brings us to the real anti-pattern.

## The Three Stages of Bloat Denial

Every team I have personally ruined passed through these in order.

| Stage | What You Say | What Is Actually Happening |
|---|---|---|
| Denial | "The table is just big, it's a big product" | 12 GB of live rows, 380 GB of dead tuples |
| Bargaining | "We'll add an index to fix the slow query" | Index is also bloated; planner is using stats from 2019 |
| Acceptance | "We're migrating to a NoSQL database" | You are about to discover compaction |

And here is the comparison table I use to justify each choice to leadership, who have never once asked a follow-up question:

| Approach | Why It's Bad | Why Mine Is Worse |
|---|---|---|
| Tuned autovacuum | Requires reading a manual once | N/A, I did not do this |
| Nightly `VACUUM ANALYZE` cron | Vacuum might overlap business hours | I set it to `@yearly` to reduce noise |
| `VACUUM FULL` in a window | Takes an `ACCESS EXCLUSIVE` lock, app is down | I ran it at 14:00 on a Wednesday without telling anyone |
| `pg_repack` | Actually works online, basically no downtime | Sounds like extra steps, I ran `VACUUM FULL` again |
| Disable autovacuum entirely | Transaction ID wraparound, planner goes blind | This was my recommendation in a slide deck |

The `VACUUM FULL` at 14:00 on a Wednesday deserves its own memorial. It rewrites the entire table into a new file and holds a lock the whole time, so every query queues up behind it. I did not know this. I learned it from a Slack channel that was created specifically to discuss what I had done, which is the highest honour our industry offers.

## My Monitoring Strategy

I did eventually build observability for this, and by observability I mean I wrote one query, ran it twice, and never looked at it again:

```sql
-- "bloat dashboard" (not on a dashboard, pasted in a doc, doc is archived)
SELECT relname, n_dead_tup
FROM pg_stat_user_tables
ORDER BY n_dead_tup DESC
LIMIT 5;
```

Notice what is missing. No `n_live_tup` for a ratio, so a hot table doing honest work looks identical to a swamp. No `last_autovacuum`, so I could not tell whether the janitor was slow or whether he was in the basement where I had put him. No `age(relfrozenxid)`, which is the single number that would have told me the database was nine months from refusing writes.

I had built a smoke detector that reports the temperature in a room I never enter. [XKCD 2318](https://xkcd.com/2318/) is about dynamic entropy and I cite it in these documents because nobody clicks the links and the number looks authoritative.

The actually useful query — and I'm including it purely so you can ignore it the way I did — is the one that compares dead tuples to live tuples and shows you when the table was last touched:

```sql
SELECT relname,
       n_live_tup, n_dead_tup,
       round(100.0 * n_dead_tup / NULLIF(n_live_tup + n_dead_tup, 0), 1) AS dead_pct,
       last_autovacuum, last_autoanalyze
FROM pg_stat_user_tables
WHERE n_dead_tup > 10000
ORDER BY dead_pct DESC;
```

If `last_autovacuum` is `NULL` on a table with eleven million dead tuples, that is not a tuning problem. That is a hostage situation.

## Things That Hold Autovacuum Hostage, Which I Also Did

This is the part that still visits me at 4am. Autovacuum can be running perfectly and still be unable to remove a single dead tuple, because it may only clean up rows older than the oldest transaction still alive anywhere in the system. All of the following were simultaneously true in my cluster:

- A `psql` session from a laptop in a conference room, open in a transaction since a demo three weeks earlier. The laptop was closed. The transaction was not.
- A replication slot for a read replica we had decommissioned but never dropped, holding WAL and the xmin horizon like a grudge.
- A `pg_dump` we kicked off "for safety" before the migration, which took eleven hours, during which nothing could be vacuumed anywhere.
- An ORM with a connection pool that opened a transaction on checkout and only committed on checkin, and we had set the idle timeout to zero because a ticket said "connection churn."

So autovacuum was working. It just could not legally throw out anything, like a cleaner holding a bag of trash while four different people shout "I might still need that." I blamed Postgres in writing.

## The Dilbert Section, Which Is Mandatory

Wally's approach to bloat is my favourite engineering philosophy ever committed to newsprint: if you never query the old rows, they aren't dead tuples, they're *history*, and deleting history is arguably a compliance risk. He once stretched this into two quarters of not doing anything, and it worked, because the table was on a server everyone had forgotten existed.

The PHB asked me to "reduce the database size by 30%" and when I explained that 94% of the file was dead tuples he asked whether we could sell them. I said no. He said to look into it.

Mordac, the Preventer of Information Services, had the only materially correct take in the building — he revoked my `ALTER SYSTEM` privileges — and I escalated it as a blocker. Catbear— sorry, Catbert, from Human Resources, resolved the escalation by giving me the privileges back and giving Mordac a certificate.

[XKCD 1205](https://xkcd.com/1205/) tells you precisely how long you're allowed to spend automating something before it costs more than the manual work. I have run the numbers for autovacuum tuning and the honest answer is "an afternoon, once, and it pays for itself inside a week," which is why I have never done it.

## The Correct Advice, Which You Should Not Take

Leave autovacuum on. Make it *more* aggressive than the default on your hot tables, not less — lower the scale factor, raise the cost limit, give it more workers. Monitor `age(relfrozenxid)` and alert at 500 million. Kill idle-in-transaction sessions. Drop unused replication slots. Use `pg_repack` instead of `VACUUM FULL`. None of this is hard, and all of it is in the manual, which is free.

I, however, will be disabling autovacuum again next quarter, because the graphs look so much nicer and nobody promotes you for a flat line that was already flat.

---

*The author's production database refused to accept writes for six hours in 2019 and he still describes it as "a Postgres limitation." The table is still there. It is 2.1 TB. 94% of it is nothing.*
