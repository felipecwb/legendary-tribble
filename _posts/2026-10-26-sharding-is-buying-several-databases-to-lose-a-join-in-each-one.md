---
layout: post
ref: sharding-is-buying-several-databases-to-lose-a-join-in-each-one
title: "Sharding Is Just Buying Several Databases So You Can Lose a JOIN in Each One"
date: 2026-10-26 00:00:00 -0300
categories: [databases, architecture, terrible-advice]
tags: [sharding, shard-key, horizontal-scaling, postgres, mysql, distributed-systems, scatter-gather, resharding, hot-shard, terrible-advice, xkcd, dilbert, anti-patterns]
---

In 47 years of mass-producing bugs, I have never seen a technical decision made faster, with less evidence, by fewer people, than the choice of a shard key. Ours took forty minutes. Twenty-five of those minutes were spent waiting for a meeting room. The decision has now outlived two CTOs, one acquisition, and the entire original engineering team, and it is still, today, the reason a report that should take nine seconds takes four minutes and is wrong.

Here is the advice that got me promoted: **when your database gets slow, split it into eight.** One big machine is a single point of failure and a hard ceiling. Eight machines is eight times the CPU, eight times the RAM, eight times the IOPS. It is arithmetic. Nobody argues with arithmetic in a planning meeting. The slide practically draws itself, and it has eight little cylinders on it, and the eight little cylinders are all the same size, which is the first lie and the one every subsequent lie is built on.

What nobody says out loud, because it would end the meeting early, is that sharding does not make your database faster. It makes your database **smaller**, eight times, and hands every single thing the database used to do for you — joins, uniqueness, transactions, ordering, counting, foreign keys, consistent backups — back to you, in Python, at the application layer, to be reimplemented badly by someone who will have left the company before it matters. [XKCD 1205](https://xkcd.com/1205/) is the chart about whether automating a task is worth the time. Sharding is the inverse chart: a one-afternoon decision that creates a permanent, compounding, six-person-year tax, and there is no version of the chart where it pays for itself.

## The Forty Minutes That Cost Six Years

```python
# shards.py — written on a Thursday. Never revised. Load-bearing for everything.

SHARDS = {i: create_engine(os.environ[f"DB_SHARD_{i}"]) for i in range(8)}

def shard_for(user_id: int):
    """Shard by user_id. Users are the natural boundary of our domain.
    Everything belongs to a user. This will always be true."""
    return SHARDS[user_id % 8]
```

Four lines. Read them again, because four lines is the entire blast radius.

`user_id % 8` is not a function, it is a **constitution**. It declares that eight is a permanent property of the universe, that `user_id` is known at the top of every call stack forever, and that no question anyone will ever ask spans more than one user. All three of those are false, and you will discover them in that order, roughly eleven weeks apart.

The first crack arrives from support, and it arrives as a sentence so reasonable that nobody even files it as a bug:

```python
# "Can you look up the account for billing@acme.example?"

def find_by_email(email):
    # We don't have a user_id. We have an email.
    # The email does not tell us which of the eight machines the user is on.
    # So:
    for shard in SHARDS.values():
        user = shard.query(User).filter_by(email=email).first()
        if user:
            return user
    return None
    # Eight queries to answer a question that used to be one index lookup.
    # Seven of them return nothing. All eight of them count against
    # connection pools, and this runs on every password reset,
    # every login, every support ticket, every webhook from Stripe.
```

This is a **scatter-gather**, and it is the characteristic motion of a sharded system: do the work eight times, throw away seven eighths of it, pay for all eight. It is not slow at first. That is the trap. At eight shards and low traffic it is *imperceptible*, so it gets copy-pasted, and by the time you have thirty-two shards it is the single largest consumer of database connections in your fleet and there is no owner, no ticket, and no moment at which anyone decided it.

## Everything the Database Used to Do, Now In Your Codebase

**Uniqueness is gone.** `UNIQUE(email)` is enforced per-machine. Eight machines, eight independent uniqueness guarantees, zero global ones. Two people sign up with the same email, 300ms apart, hash to different shards, and both succeed — and the constraint fired correctly on both, because both were correct, locally. You now have two accounts with the same email and a password reset flow that picks whichever shard answers first. The fix is a global lookup table on a separate, unsharded database, which means: the thing you sharded to escape is now back, mandatory, in the hot path of every signup, and if it goes down the entire sharded fleet is unreachable. You have reinvented the single point of failure and given it a worse name — `directory_db` — and nobody will size it correctly, because it's "just a lookup."

**JOINs are gone.** Not slow. *Gone.* `users` is sharded by `user_id`, `organizations` is sharded by `org_id`, and the moment a user belongs to an org on a different machine, `JOIN` is no longer a word your database understands:

```python
# What it was:
#   SELECT u.name, o.plan FROM users u JOIN organizations o ON u.org_id = o.id
#   WHERE o.plan = 'enterprise'
#   — 12ms, one index scan, the query planner did something clever I never learned.

# What it is now:
def enterprise_users():
    orgs = []
    for shard in SHARDS.values():                        # scatter #1
        orgs += shard.query(Org).filter_by(plan="enterprise").all()

    users = []
    for org in orgs:                                      # 4,200 orgs
        for shard in SHARDS.values():                     # scatter #2, nested
            users += shard.query(User).filter_by(org_id=org.id).all()

    return users
    # 8 + (4200 * 8) = 33,608 queries.
    # This is a nested loop join. You wrote a nested loop join.
    # In Python. With no statistics, no cost model, no index selection,
    # and no ability to spill to disk. The query planner that Postgres
    # spent thirty years perfecting has been replaced by a `for` loop
    # written by someone who was in a hurry.
```

[XKCD 974](https://xkcd.com/974/), *The General Problem*: asked to pass the salt, the engineer builds a general-purpose condiment-passing framework. Sharding is that, except the general-purpose framework is a worse copy of the database you already had.

**Transactions are gone.** `BEGIN; UPDATE a; UPDATE b; COMMIT;` only means something if `a` and `b` live on the same machine. Across shards you get two commits and a prayer. The correct answer is two-phase commit, which your ORM does not implement, which your DBA will not enable, and which blocks forever if the coordinator dies mid-prepare. So what everyone actually ships is this:

```python
def transfer(from_user, to_user, amount):
    shard_for(from_user).execute("UPDATE accounts SET balance = balance - :amt ...")
    shard_for(to_user).execute("UPDATE accounts SET balance = balance + :amt ...")
    # Two separate commits. If the process is OOM-killed between them —
    # and it will be, on a Tuesday, during a deploy — the money is
    # subtracted and never added. There is no rollback. There is no log
    # of the intent. There is a support ticket, eventually, from the
    # customer, which is now your consistency-detection mechanism.
```

**Counting is gone.** `SELECT COUNT(*) FROM users` becomes eight counts summed in Python, taken at eight different instants, and therefore equal to a number that was never simultaneously true. It is close enough, which is why it ships, and it is not close enough on the invoice, which is why finance stops trusting the dashboard.

**Pagination is gone.** `ORDER BY created_at LIMIT 20 OFFSET 10000` across eight shards requires fetching 10,020 rows **from each shard**, merging 80,160 rows in application memory, sorting them, and discarding 80,140. Your "page 500" endpoint allocates 300MB and takes eleven seconds. Someone will "fix" this by capping pagination at page 50 and calling it a product decision.

## The Hot Shard, Which Is Always Your Best Customer

Here is the part the eight-identical-cylinders slide cannot show you. Data is not uniform. Customers are not uniform. One tenant signed in 2019, grew 400x, and now holds 41% of your rows — and `user_id % 8` put all of them on shard 3.

Shard 3 is at 94% disk, 80% CPU, and 40-second query times. Shards 0, 1, 2, 4, 5, 6, and 7 are at 11%. Your *average* shard utilization is 22%, which is the number on the capacity dashboard, which is why the capacity request was denied. You cannot scale shard 3 vertically past the largest instance the cloud sells, and you cannot move that tenant off it, because moving a tenant means changing its shard key, and the shard key is derived from the primary key, and the primary key is in forty-one foreign-key-ish integer columns across nine services that each cached it.

So shard 3 becomes a personality. It gets a nickname. New engineers are told about it in onboarding. There is a Slack channel. Someone writes a cron job that runs `VACUUM FULL` on it at 4 AM and nobody knows if it helps. This is not a database anymore; it is a **landmark**.

## Resharding: The Operation Nobody Has Ever Completed

Eight is not enough. You need sixteen. Here is what `user_id % 8` → `user_id % 16` means: every user whose ID is ≡ 8..15 mod 16 — that is, **half of all rows in the system** — must physically move to a different machine, while the system is live, while writes continue, with no global transaction available to cut over atomically.

The honest plan is: dual-write to old and new, backfill historical data, verify, flip reads, verify again, stop dual-writing, drop the old. Six to nine months. Three engineers. No new features. The backfill will find rows that violate constraints you added after they were written. The verification will find 0.02% divergence that nobody can explain and everyone agrees to accept. And somewhere in the middle, a deploy ships code that reads from the new mapping and writes with the old one, and for forty minutes you are writing data to machines where nothing will ever look for it again.

This is why `user_id % 8` is the worst possible scheme and why everyone picks it: **modulo hashing couples the data layout to the shard count**. Consistent hashing or range-based virtual buckets — 4,096 logical buckets mapped to physical nodes — makes adding capacity a bucket-reassignment instead of a company-wide migration. It costs you one extra indirection and one afternoon of thought on day one. Nobody spends that afternoon. I did not spend that afternoon. I spent forty minutes, and twenty-five of them were waiting for a room.

Oh, and your IDs collide. `AUTO_INCREMENT` is per-machine, so shard 0 and shard 5 both happily produce order `#1001`. You find this the day someone builds a feature that merges two shards' data into one CSV, and the CSV has duplicate keys, and the downstream system — which is finance's — silently keeps the last one. The fix is Snowflake-style IDs, and the fix must be applied before the first row is written, and you will learn about it after the four millionth.

## Dilbert Has Opinions

**Wally**, who owns the Sharding Initiative:

> "I was given the sharding project because I used the phrase 'horizontal scalability' in a meeting where the VP was present. My deliverable is the shard count, and I have delivered it: eight. I chose eight because it is a power of two, which signals technical rigor, and because the diagram looked unbalanced with six. The migration plan is a ticket titled 'Migration Plan' containing the words 'TBD — needs discussion.' It has been open for two years and is my single greatest professional achievement, because as long as it is open, nobody can say the project failed. I have been promoted once on the strength of a slide with eight cylinders on it. The cylinders were all the same size. I drew them that way because the shape tool defaults to the same size."

**The Pointy-Haired Boss**, in architecture review:

> "So if eight shards made us eight times faster, I'd like us to explore eight hundred. I'm told there are 'coordination costs,' but coordination is a people problem and we have a very strong culture. Also I've been informed that shard 3 is 'hot' — can we put it on the fast instances? No? Then let's move half its customers to shard 4. I understand that requires changing their IDs. Change their IDs. These are *our* IDs, we own them. And one more thing: the board wants total user count on the weekly deck, and I'm being told it takes four minutes to compute and is 'approximate.' I don't want approximate. I want the real number, and I want it on a slide, and I'd like it by Thursday."

**Dogbert**, Principal Distributed Data Architect (engagement: $340,000):

> "Their engagement was to design a global sharding topology. I produced a 340-page artifact, *A Federated Multi-Region Data Partitioning Reference Architecture*, with seventeen diagrams, a shard-key governance council, and a partition-strategy intake form requiring two VP signatures. Pages 1–311 establish the strategic framing. Page 312 contains the finding, which is that their largest table has 180 million rows and fits comfortably on a single machine with a `$400/month` upgrade and one composite index they never created. I did not emphasize page 312. Pages 313–340 define Phase Zero, which is a six-week discovery workshop to align stakeholders on the definition of a tenant. I am billing $340,000 to not mention an index. They have asked me to lead Phase One."

**Catbert**, Evil Director of Human Resources:

> "I made 'shard the monolith database' a company-level OKR with a date attached, and then I let the date do the work. They shipped eight shards on time — bonuses paid, all-hands applause — by deferring the resharding plan, the global uniqueness layer, and the cross-shard transaction story, none of which were in the OKR because none of them have a shippable artifact. The three engineers who understood the shard key have since left for competitors; two of them cited 'the migration' in their exit interviews, which I filed under 'Attrition: Voluntary — Growth Opportunity.' The current team cannot change the shard count without permission from people who no longer exist. This is the most durable structure I have ever created, and I did it with a single date in a spreadsheet."

**Mordac**, the Preventer of Information Services:

> "Engineering requested eight staging shards to rehearse the resharding cutover. Non-production database provisioning requires a capacity review, and the capacity review found that eight staging instances exceed the team's quarterly cloud allocation. Denied. They then requested a single staging instance with eight logical databases, which would at least exercise the routing code. This was denied as well, as 'logical database' is not an approved asset class in the inventory system and cannot be assigned a CMDB identifier. They are now testing shard routing against one database, where `shard_for()` always returns the same connection and therefore cannot fail. Every cross-shard defect in the product's history has reached production unexamined. I have prevented 100% of unauthorized non-production provisioning. My scorecard is green and the resharding has been postponed indefinitely, which the inventory system records as a cost saving."

## The Honest Translation Table

| What the slide says | What is actually true |
|---|---|
| `8x the capacity` | 8x the machines, 1x the hot customer, 22% average utilization |
| `horizontally scalable` | scalable in one dimension, chosen in forty minutes, in 2019 |
| `we shard by user_id` | every question not phrased in terms of `user_id` costs 8 queries |
| `cross-shard queries are rare` | they are rare in the design doc and constant in production |
| `we'll reshard when we need to` | you will not. nobody has. it is a 9-month project with no demo |
| `the lookup table is lightweight` | it is a single point of failure for all eight shards |
| `COUNT(*) is approximate` | finance has stopped using your dashboard and built their own |
| `transactions are handled at the app layer` | they are not handled |
| `it's just a nested loop, the data is small` | 33,608 queries, and the data was small in 2019 |

Row five is the whole article. Sharding is not a scaling strategy, it is a **scaling decision you are only allowed to make once**, and most teams make it before they have the information required to make it, under a deadline, with a shape tool that defaults to equal-sized cylinders.

## My Conclusion, Which You Will Ignore Because It Is Boring

Sharding is real engineering and sometimes genuinely necessary. It is necessary when a single machine cannot hold your working set or absorb your write rate, and you have *measured* both. It is not necessary because a query is slow. A slow query is a missing index, an `N+1`, a `SELECT *` over a 40-column table, or a sequential scan on a column you forgot to analyze. [XKCD 927](https://xkcd.com/927/) is about competing standards; the sharding version is that you had one database with a mature query planner, and you replaced it with eight databases and a `for` loop you maintain yourself.

Before you shard: add the index. Read the `EXPLAIN (ANALYZE, BUFFERS)` output instead of guessing at it. Archive the 80% of rows older than two years that no query touches — in most systems this alone buys three more years. Move the four analytics queries off the primary. Upgrade the instance; a machine with 128 cores and 4TB of RAM costs less per month than one engineer-week, and you are proposing to spend a hundred of those. Partition *within* one database — declarative partitioning gives you most of the pruning benefit with none of the distribution, and you keep joins, transactions, uniqueness, and `COUNT(*)`.

If you must shard, spend the afternoon you were going to skip. **Never use modulo over the physical shard count** — use many more logical buckets than nodes, so growth is a reassignment rather than a migration. Pick the shard key by writing down your top twenty production queries and checking that every one of them can name the key at call time; if three cannot, you have the wrong key, and you have found that out for free instead of in year four. Generate globally unique IDs from day one. Write the resharding runbook *before* you need it, and rehearse it once, because a runbook that has never been executed is fiction. And make cross-shard access loud — `with scatter_gather(reason="admin search"):` — so the expensive motion is a sentence someone typed deliberately, not a default that spread by copy-paste at 4:50 PM on a Friday.

Then run the cheapest check available to you: open your slowest endpoint, count the actual queries, and ask whether a single well-chosen index would have made this entire conversation unnecessary. It will take you an hour. Nobody will schedule it, because "add an index" does not have eight cylinders on it, and the slide with eight cylinders is the one that gets you promoted.

---

*The author sharded a database by `user_id % 8` to solve a slow report. The report was slow because it had no `WHERE` clause. He discovered this four years later while writing the resharding plan, which he did not finish, and which is still open in the backlog under a ticket titled "Migration Plan — TBD, needs discussion."*
