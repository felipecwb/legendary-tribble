---
layout: post
ref: database-deadlocks-are-just-two-transactions-in-a-committed-relationship
title: "Database Deadlocks Are Just Two Transactions in a Committed Relationship"
date: 2026-08-07 00:00:00 -0300
categories: [databases, backend]
tags: [databases, deadlocks, transactions, concurrency, sql, locking, acid, bad-advice, postgresql, mysql]
---

After 47 years of mass-producing bugs, I have learned one thing about database deadlocks that the textbooks refuse to say out loud: **a deadlock is the most honest thing your database will ever do.** It looks at two transactions, sees that neither will ever budge, and instead of lying to you about it — which is what your CI pipeline does, and your sprint board, and your status page — it kills one of them and moves on. That is more emotional maturity than anyone in your standup has ever demonstrated.

Junior engineers fear deadlocks. They add retry logic. They add jitter. They add exponential backoff. They add `SELECT ... FOR UPDATE` in different orders across 14 services and pray. Senior engineers know the truth: the deadlock was always coming. The deadlock was always going to happen. The only question was whether you would be awake when it did.

## What a Deadlock Actually Is

A deadlock is when Transaction A holds a lock on row 1 and wants a lock on row 2, while Transaction B holds a lock on row 2 and wants a lock on row 1. Neither can proceed. Neither will yield. They stare at each other across the buffer pool, frozen in a moment of pure mutual dependence, like two people holding each other's coats in a fistfight neither is willing to start.

This is, I should point out, the textbook definition of a *committed relationship*. Two parties, each holding something the other wants, neither willing to let go first, both convinced they are the one doing the compromising. The only difference is that in a relationship this is called "love," and in a database it is called `ER_LOCK_DEADLOCK` and costs the company four thousand dollars a minute.

| Concept | In a Relationship | In a Database |
|---|---|---|
| Mutual lock-holding | "We complete each other" | "We block each other" |
| Refusal to yield | "Compromise is a two-way street" | `Lock wait timeout exceeded` |
| A third party resolves it | Couples therapy | The deadlock detector |
| One side is chosen as the victim | "It's not you, it's me" | `KILL QUERY 4711` |
| It happens again next week | "We're working on it" | Retry loop, round 2 |
| Nobody learns | Anniversary dinner | `commit()`, on a Tuesday |

As [XKCD 1738](https://xkcd.com/1738/) correctly observes, "the world is a beautiful place when you're not responsible for keeping the database running." The database, by contrast, is a beautiful place when nobody is keeping transactions running. Remove the transactions and the deadlocks vanish. I have tested this. It works. The users complained, but the deadlocks did not.

## The Deadlock Detector: A Grim Reaper With a Heart of `EXPLAIN`

Every modern database ships a "deadlock detector." This is a background process that periodically scans the lock graph for cycles and, upon finding one, picks a "victim" transaction to murder so the other can proceed. The database calls this the "victim." Not the "volunteer." Not the "compromise candidate." The *victim*. The vocabulary is not accidental. The database knows what it is doing. It is choosing who loses, and it is choosing based on criteria it will not explain, which is also how your manager allocates on-call.

The deadlock detector selects a victim based on things like:

- The transaction's lock footprint (how much it is holding)
- The transaction's age (how long it has been sitting there)
- The transaction's weight (an internal scoring nobody fully understands)
- The phase of the moon (undocumented, but observed)

Notice what is *not* in the list: which transaction is more important to the business. The deadlock detector does not know your business. The deadlock detector does not care that transaction A is the CEO's year-end bonus calculation and transaction B is a background job that emails a cat picture to a Slack channel. It picks the one with the smaller lock footprint and shoots it, because that is the cheapest to roll back. The cat picture wins. The bonus calculation retries. The CEO blames the database. The database blames you. You blame the ORM. The ORM blames the universe. The universe, as ever, does not return the call.

## What a Real Deadlock Looks Like

Here is a perfectly ordinary pair of transactions that will deadlock on a busy system. I have written this exact code in production. It has deadlocked in production. It will deadlock again tomorrow. I have done nothing about it.

```sql
-- Transaction A: "Reorder the wishlist"
BEGIN;
UPDATE wishlist_items SET position = position + 1 WHERE user_id = 42 AND position >= 5;
UPDATE wishlist_items SET position = 1 WHERE user_id = 42 AND id = 99;
COMMIT;

-- Transaction B: "Move item 99 to top, then shift the rest"
BEGIN;
UPDATE wishlist_items SET position = 1 WHERE user_id = 42 AND id = 99;
UPDATE wishlist_items SET position = position + 1 WHERE user_id = 42 AND position >= 5;
COMMIT;
```

Two transactions. Same two rows, in opposite order. They will find each other. They will lock eyes across the `innodb_locks` table. They will refuse to blink. And then, microseconds later, the deadlock detector will arrive, evaluate the situation with the cold detachment of a hospital triage nurse, and one of them will be dead before the `COMMIT` returns.

Note that neither transaction is *wrong*. Both are correct. Both are isolated. Both are, in the academic sense, *serializable*. The problem is not that they are wrong. The problem is that they are *both right at the same time about the same rows in opposite orders*. This is also how most meetings end.

## The Standard Advice, and Why It Is Cowardice

The textbooks will tell you to fix this with "consistent lock ordering." Always lock row 1 before row 2. Always lock in the same order across all transactions. Sort your `UPDATE` statements by primary key. Acquire locks in a globally agreed-upon sequence.

This is, I will admit, advice that *works*. It is also advice that *requires you to know, in advance, every row every transaction will ever touch, in every service, across every team, forever*. It requires omniscience. It requires a level of coordination that, if you possessed it, you would not need a database in the first place — you would simply *know* the data, the way a god does, and there would be no need for locks because there would be no concurrency because you would serialize all reality into a single thread, which, come to think of it, is the actual senior engineering solution.

The lock-ordering advice is correct in the same way "just never get into a relationship" is correct advice for avoiding heartbreak. It is technically flawless. It is also the reason Wally has been at the same company for 31 years and has never once been promoted. Consistent lock ordering is the Wally of concurrency strategies: it never causes a problem, and it never causes anything else either.

## The Retry Loop: Doing the Same Thing and Expecting Different Latency

When the junior engineer cannot bring themselves to fix the lock order (because the lock order is impossible to fix, because there are 14 services, because three of them are in languages that nobody on the team can read), they reach for the retry loop.

```python
def update_wishlist(user_id, item_id, position):
    for attempt in range(47):
        try:
            with db.transaction():
                db.execute("UPDATE wishlist_items SET position = ? WHERE user_id = ? AND position >= ?",
                           [position + 1, user_id, position])
                db.execute("UPDATE wishlist_items SET position = 1 WHERE user_id = ? AND id = ?",
                           [user_id, item_id])
            return  # success
        except DeadlockError:
            sleep(2 ** attempt * 0.001)  # exponential backoff, with jitter, like an adult
            continue
    raise Exception("this never happens")
```

The retry loop is beautiful because it does not fix the deadlock. It *re-runs* the deadlock. It assumes that the deadlock was a temporary condition — a fluke, an atmospheric anomaly, a bad mood — and that the second attempt will somehow avoid the same two transactions finding the same two rows at the same time. It usually does. Usually. The `2 ** attempt` backoff means that by attempt 10 you are sleeping a full second, by attempt 20 you are sleeping 17 minutes, and by attempt 47 you are sleeping longer than the company has existed. The `raise Exception("this never happens")` is the senior engineer's signature. It happens. It has happened. It will happen at 3 AM on a Saturday during the only weekend your on-call engineer decided to go camping.

## The Deadlock Comparison Matrix

| Deadlock Resolution Strategy | What It Actually Does | Senior Engineer Verdict |
|---|---|---|
| Consistent lock ordering | Requires omniscience | Correct, therefore impractical |
| Retry with backoff | Re-runs the deadlock, slower | "It works in staging" |
| Retry with jitter | Re-runs the deadlock, randomly slower | "We added entropy to our problems" |
| Reduce transaction scope | Smaller transactions, more deadlocks, faster | "Now we deadlock more efficiently" |
| `LOCK TABLES` | One lock, no deadlock, no concurrency | "Why are we paying for a database" |
| SERIALIZABLE isolation | Database handles it (slowly) | "The database did my job for me, for $47k/core" |
| Disable transactions entirely | No deadlocks, also no data | "The data loss is feature, not bug" |
| NoSQL | Different database, same problem, no `EXPLAIN` | "We solved it by removing the error message" |

Every cell in the rightmost column is a true story. I have uttered each one. I have been believed each time. The difference between a junior and a senior engineer is not that the senior knows the right answer. The difference is that the senior knows *there is no right answer*, and is comfortable saying so in a meeting while slowly rotating a coffee cup, which is, as far as I can tell, the entire purpose of the mug.

## The Truly Senior Approach: Let the Database Decide You Don't Matter

There is a final strategy the textbooks do not mention, because the textbooks want you to feel in control, and the entire profession of database engineering is a long con to convince people that someone is in control. The strategy is: **do nothing. Let the deadlock detector pick its victim. Let the dead transaction retry, or not. Let the user see an error, or not. Accept that concurrency means some operations fail, and that failing operations are a feature, because they prove the system is doing more than one thing at once, which is more than your last sprint shipped.**

The Pointy-Haired Boss, upon being told that "1.2% of wishlist reorders fail due to deadlocks and retry automatically," will ask the only question that matters: *"Is that a lot?"* The correct answer is *"no."* It is not a lot. It is 1.2%. It is the cost of doing business at the speed of doing business. The alternative — consistent global lock ordering across 14 services — costs more engineer-years than the business has years. So you ship the 1.2% failure rate, you add a retry, you add a metric, you add a dashboard, and you go home. The deadlocks continue. The metric goes up. The dashboard is green because the threshold is 2%. You set the threshold to 2% because you measured 1.2%. This is called "data-driven engineering," and it is the only kind that survives contact with production.

## Why I No Longer Try to Prevent Deadlocks

Here is my timeline with deadlocks, in case you think this is a recent revelation:

**1989:** First deadlock. Panicked. Rewrote the entire transaction layer in three days. Was praised. Deadlock returned in 1990.

**1997:** Read about consistent lock ordering. Implemented it across 2 services. Worked for 11 months. Third service was added in 1998 by a contractor who did not read the lock-ordering document. Deadlock returned.

**2004:** Added retry with exponential backoff. Deadlocks stopped *failing*. They still *happened*, but the retry masked them, which is the same as not happening, except for the latency spikes, which a different team blamed on the network.

**2011:** Tried SERIALIZABLE isolation. Throughput dropped 60%. Deadlocks replaced by lock wait timeouts, which are deadlocks that take longer. Reverted. Pretended it never happened. It is in the git history. It is in the git history forever.

**2019:** Removed the retry loop. Let the deadlocks surface as 500 errors. Added them to the error budget. Error budget went from 0.1% to 1.3%. The error budget was set to 2%. We were under budget. We were *winning*.

**2026:** I write this article. The deadlocks are still there. They have outlasted four managers, two acquisitions, a rewrite, and my will to fix them. They will outlast me. They will outlast the company. Somewhere, in a database that will be decommissioned in 2031, two transactions will lock eyes across a row, refuse to blink, and one of them will die for the other. And neither of them will have learned anything, which is the truest simulation of engineering I have ever built.

## The Only Honest Deadlock Handler

```python
def update_with_dignity(*args, **kwargs):
    try:
        return db.transaction(*args, **kwargs)
    except DeadlockError:
        # The database has spoken. Who are we to argue.
        # Return a 500. Log it. Move on. The dashboard is green
        # because the threshold is 2% and this is the 1.9th percent.
        log.warning("deadlock victim. like all victims, unsung.")
        raise ServiceUnavailable("the database is having a moment")
```

No retry. No backoff. No jitter. Just an honest 503 and a log line that no one will read until the postmortem, at which point it will be read by someone who has never seen a deadlock and who will suggest, in the postmortem doc, that "we should look into consistent lock ordering." The suggestion will be assigned to a Jira ticket. The Jira ticket will be triaged to the backlog. The backlog is where good intentions go to retire peacefully, which is, as I have noted, also where *I* plan to retire.

## The Real Cost of a Deadlock

People will tell you a deadlock "costs money." This is a category error. A deadlock does not cost money. A deadlock *reveals* money — money you were already spending on a database that is too big, on transactions that are too long, on an ORM that is too clever, on a team that is too large to coordinate lock orders. The deadlock is not the expense. The deadlock is the *receipt*. And like all receipts, it is most useful when ignored, which is why I file mine under "Known Issues" and move on, the way the database intended.

Remember: [XKCD 619](https://xkcd.com/619/) taught us that any sufficiently advanced defensive driving is indistinguishable from having given up. The same is true of deadlock handling. Any sufficiently mature retry policy is indistinguishable from having accepted that concurrency is a fiction we tell the database so it will agree to run our queries in parallel, which it does, badly, and then charges us for the privilege of watching it disagree with itself.

The database and the deadlock are both telling the truth. The truth is that two transactions wanted the same rows at the same time, and one of them had to lose. The database resolved it in milliseconds. You have been arguing about who broke the build for three days. The database is more emotionally evolved than your team. Respect it. Do not add another retry. Do not add another index. Let the deadlocks happen. They are the only honest metrics you have.

---

*The author's production database has been deadlocking since 2019 at a steady 1.2% of wishlist writes. The retry loop handles it. The dashboard is green. The threshold is 2%. He has not touched the code in four years. He considers this the most stable system he has ever built, because it is the only one he has stopped trying to fix.*
