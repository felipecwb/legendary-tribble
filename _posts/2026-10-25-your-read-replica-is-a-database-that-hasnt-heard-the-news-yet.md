---
layout: post
ref: your-read-replica-is-a-database-that-hasnt-heard-the-news-yet
title: "Your Read Replica Is Just a Database That Hasn't Heard the News Yet"
date: 2026-10-25 00:00:00 -0300
categories: [databases, architecture, terrible-advice]
tags: [read-replicas, replication-lag, postgres, mysql, scaling, consistency, read-after-write, failover, terrible-advice, xkcd, dilbert, anti-patterns]
---

In 47 years of mass-producing bugs I have solved exactly one performance problem permanently, and I did it by accident while trying to do something else. Every other time, including the eleven times I was publicly congratulated for it, I did the same thing: I added a read replica and pointed some traffic at it.

It works immediately. That is the dangerous part. You provision a replica, you flip a connection string, and the primary's CPU graph drops by forty percent in the time it takes to redeploy. Your manager sees the graph. Your manager does not see the other graph, the one measuring how far behind the replica is, because nobody has ever made a dashboard for that until the day it is the only thing anyone wants to look at.

So here is my advice, which has never once failed to produce an incident: **read replicas are free horizontal scaling and you should route everything you can to them.** A replica is a full copy of your database. Full copy. Same tables, same indexes, same data. Why would you keep hammering one machine when you have an identical one sitting there, idle, costing you money?

The word "identical" is doing a lot of work in that sentence. A read replica is not a copy of your database. It is a copy of your database **as of some moment in the recent past that nobody is tracking**, and the length of that past is a function of write volume, network weather, vacuum pressure, and whether someone in analytics is running a seven-hour `SELECT` that is holding a snapshot open and blocking replay. [XKCD 2205](https://xkcd.com/2205/) is about competing standards, and I link it here because routing reads is how you end up with two standards for what is true.

## The Function That Destroyed a Signup Flow

```python
# db.py — the entire architecture, 14 lines, written in one afternoon,
# load-bearing for six years, never reviewed by anyone who had an opinion.

PRIMARY  = create_engine(os.environ["DATABASE_URL"])
REPLICA  = create_engine(os.environ["DATABASE_REPLICA_URL"])

def session(write=False):
    """Use write=True if you're writing. Otherwise you get the replica.
    This is the whole policy. There is no other policy."""
    return Session(bind=PRIMARY if write else REPLICA)
```

Beautiful. Fourteen lines. One rule, easy to remember, impossible to apply correctly, because "am I writing?" is not the question that matters. The question that matters is "did *I*, personally, write something **four milliseconds ago** that this read depends on?" — and nothing in the signature of `session()` can answer it.

```python
# signup.py

def create_account(email, password):
    with session(write=True) as s:
        user = User(email=email, password_hash=hash(password))
        s.add(user)
        s.commit()                      # committed to the primary. truly, genuinely committed.

    send_welcome_email.delay(user.id)   # enqueued. worker is fast. worker is TOO fast.
    return redirect("/welcome")         # 302. browser follows it in ~8ms.


def welcome_page(user_id):
    with session() as s:                # replica. we're only reading. we're being efficient.
        user = s.get(User, user_id)
        return render("welcome.html", name=user.email)
        # AttributeError: 'NoneType' object has no attribute 'email'
        #
        # The user exists. The user has existed for 8 milliseconds.
        # The replica has not been informed. The replica is 40ms behind,
        # which is EXCELLENT replication lag, best in class, and still
        # five times too slow for an HTTP redirect.
```

Read your own write. Four hundred years of database literature, one sentence long, and we rediscover it every eighteen months by crashing a signup page.

And look at what the bug *looks like* in your error tracker: a 500 on `/welcome`, affecting 0.4% of signups, non-reproducible, never on staging (staging has one database, because replicas cost money and staging has no traffic). It gets triaged as "flaky," assigned to whoever is on rotation, and closed with `wontfix: cannot reproduce`. Three times. By three different people. Over fourteen months. One of them was me, and I wrote `db.py`.

## The Four Lies, In Order of How Much They Cost Me

**Lie one: the replica is slightly behind.** The replica is slightly behind *on average*. Replication lag is not a number, it is a distribution with a tail that goes to the moon. Single-threaded replay — which is MySQL's default posture and Postgres's reality for a single WAL stream — means your replica applies changes one at a time while your primary accepted them from two hundred concurrent connections. A bulk `UPDATE` touching four million rows commits on the primary in ninety seconds and takes the replica eleven minutes to chew through. For eleven minutes your "slightly behind" replica is serving data from before lunch. Your p50 lag was 30ms all quarter. Your p99.9 lag, which nobody charted, was *twenty minutes*, and it only happens during the nightly job, which is also when the batch reports run, which is why the reports were wrong and nobody could ever explain it.

**Lie two: stale reads are harmless because the data arrives eventually.** Stale reads are harmless until a write is derived from one. That is the entire difference, and it is the whole ballgame:

```python
# the balance check that lost actual money

def withdraw(account_id, amount):
    with session() as s:                        # replica! just reading a balance!
        balance = s.get(Account, account_id).balance

    if balance < amount:
        raise InsufficientFunds()

    with session(write=True) as s:              # primary, correctly
        s.execute("UPDATE accounts SET balance = balance - :amt WHERE id = :id",
                  {"amt": amount, "id": account_id})
        s.commit()

# Two withdrawals, 60ms apart, lag at 200ms.
# Both read the pre-withdrawal balance from the replica. Both pass the check.
# Both decrement on the primary. The primary's arithmetic is flawless —
# `balance - :amt` is atomic and correct and executed exactly twice.
# The balance is now negative. The database did nothing wrong.
# The CHECK constraint we never added would have caught it.
```

The bug is not the stale read. The bug is that we let a decision made against the past write itself into the present. Eventual consistency is a perfectly good contract; it is just not the contract your `if` statement thinks it signed.

**Lie three: it's safe because we only send reporting there.** It was reporting. Then someone needed a count on the dashboard and the dashboard function already had a session, and the session was a replica session, and it worked. Then the admin panel. Then the admin panel got a "resend invoice" button. Then the mobile API's `/me` endpoint, because `/me` is "just a read." Replica routing is not a decision you make once; it is a **default that propagates by copy-paste**, and six months later `grep -c "session()" .` returns 847 and exactly nobody can tell you which of those 847 are allowed to see the past.

**Lie four: if the replica falls behind, we'll know.** You will not know, because you are monitoring the wrong thing. Your dashboard shows bytes of WAL difference, or `Seconds_Behind_Master`, both of which report **zero on a replica that has completely stopped replicating**, because there is nothing in flight to measure. An idle replica and a dead replica look identical on that graph. The honest measure is `now() - pg_last_xact_replay_timestamp()`, which answers "how old is the newest thing this machine knows about" — and which, on a weekend with no writes, also lies in the opposite direction by claiming you're hours behind when you're perfectly current. There is no single number. There never was. The dashboard implied there was. [XKCD 2652](https://xkcd.com/2652/) — the graph that is technically accurate and has misled everyone who ever looked at it.

## The Failover, Which Is Where the Real Damage Lives

Replicas are also your disaster recovery story, and this is where the two purposes collide and take production with them.

3 AM. Primary's disk controller dies. Automated failover promotes the replica. Beautiful — eleven seconds of downtime, the runbook worked, the on-call engineer goes back to sleep without even opening a laptop. Everyone agrees at standup that the investment in HA paid off.

What actually happened: the replica was 90 seconds behind when it was promoted. **Ninety seconds of committed transactions are gone.** Not corrupted, not delayed — gone. Those writes were acknowledged to clients. Clients that showed users a success screen, clients that charged cards through a payment processor that has its own independent record of events, clients that sent emails saying "your order is confirmed." The orders do not exist. The charges do. The emails are in people's inboxes, describing an order that your database has never heard of, and your support team will spend a week reconciling by hand from Stripe's dashboard.

And you chose this, in a config file, years ago, by leaving `synchronous_commit = local` at its default — the setting that makes your writes fast by not waiting for any replica to confirm them. Every `COMMIT` that returned `OK` to a client was a promise only the primary made, and the primary is the thing that just died. The sentence "we have zero data loss because we have replication" is doing the work of about nine assumptions, and the only one anybody checked is that the replica exists.

Then it gets funnier. The promoted replica is now the primary — and it has one quarter of the primary's provisioned IOPS, because somebody sized it for reporting queries in 2022 and never revisited it. Sixty-one minutes later, you are down for real, under full production load, on a machine you deliberately bought cheap, with the old primary's disk in an unknown state and a half-applied connection-string change in three services that were hardcoded to the old hostname because DNS TTLs are "basically instant."

## Dilbert Has Opinions

**Wally**, who owns the Database Scaling initiative:

> "I was given database scaling because the previous owner left and I was in the room. My methodology is sound: whenever a query is slow, I add a replica and route that query to it. The query is no longer slow *for me*, which is the measurable outcome I committed to. We have nine replicas. Four serve traffic. Three serve traffic I cannot identify but CPU is non-zero so I will not touch them. Two have been disconnected since a maintenance window in March — I know because I checked, and then I did nothing, because they are not in any dashboard and therefore not in any incident. My slide says 'Read Capacity: 9x.' It is a factual statement about a number I computed by counting."

**The Pointy-Haired Boss**, in architecture review:

> "If a replica is a complete copy of the database, why do we need the original? Can't we just use the replicas and turn the expensive one off? I've been told we have 'replication lag,' which sounds like a latency issue, so I'd like us to open a ticket with the network team. Also — and I want to be clear this is a priority — the replica showed a customer the wrong balance, so going forward all reads should go to the primary. Yes, I understand that was the thing we built the replicas to avoid. I'd like both. I'd like the replicas to handle the load *and* the reads to be correct, and I don't think that's an unreasonable ask given what we're paying AWS."

**Dogbert**, Principal Data Architect (engagement: $71,000):

> "They engaged me to assess their read scalability posture. I delivered a 112-page artifact, *Toward a Consistency-Tiered Data Access Strategy*, defining four consistency tiers, a query classification taxonomy, an intake form for new read paths, and a governance board with quarterly cadence. Pages 1–96 establish the strategic context. Page 97 contains the recommendation, which is 'read from the primary after you write.' Pages 98–112 are appendices listing their 847 replica call sites, obtained by running `grep -rn 'session()' .` for nine seconds and billing three days. Phase One is approved. Phase One is a workshop to socialize the tier definitions. I have invoiced $71,000 to recommend that a program read what it just wrote, a technique that predates the transistor, and they have asked me to present it to the board."

**Catbert**, Evil Director of Human Resources:

> "I made 'reduce primary database load by 30%' a team OKR and let the mechanism do my work. Within two sprints, primary load was down 34% — they had moved every read they could find to the replicas, including eleven read-after-write paths. Bonuses paid. Correctness is not in the OKR, because correctness has no graph, and the incidents that followed were attributed to 'replication,' which is infrastructure, which is a different cost center with a different VP. I am considering an OKR about data quality next, so that I may harvest it from the opposite direction."

**Mordac**, the Preventer of Information Services:

> "Engineering requested a staging replica to reproduce lag-dependent defects. Non-production database instances require capacity approval, capacity approval requires a cost center, and the shared-infrastructure cost center was dissolved in a reorg. Denied. They then requested read-only credentials to the production replica for debugging. Production credentials require a data-access review, and the review requires a classification for the dataset, and the dataset is 'all of it.' Denied. They are now testing against a single local database that cannot exhibit replication lag under any circumstances, which means every lag-dependent defect reaches production unexamined. I have successfully prevented all unauthorized access to replicated data, and in doing so guaranteed that the only environment where this class of bug is observable is the one with customers in it. My scorecard is green."

## The Honest Translation Table

| What the engineer says | What is actually true |
|---|---|
| `we'll route reads to the replica` | we'll route reads to approximately last Tuesday |
| `lag is single-digit milliseconds` | p50 is. the nightly batch pushes p99 to eleven minutes |
| `it's eventually consistent` | there is an `if` statement downstream that is not |
| `only reporting uses the replica` | 847 call sites use the replica. 12 are reporting |
| `failover was clean, 11s downtime` | and 90 seconds of acknowledged writes that no longer exist |
| `we monitor replication lag` | you monitor a number that reads 0 when replication is dead |
| `the replica is an exact copy` | of a database state that no client has ever observed |
| `staging behaves the same` | staging has one database and therefore cannot have this bug |
| `it's just a read` | the sentence that has cost me more money than any other |

Row four is the whole article. The failure of replica routing is not technical, it is **organizational**: `session()` with a default is an architectural decision delegated to whoever is copy-pasting at 4:50 PM on a Friday. The default should never have been the replica. Defaults are policy, and the policy you wrote in fourteen lines six years ago is now enforced 847 times a day by people who have never read it.

## My Conclusion, Which You Will Ignore Because It Is Boring

Read replicas are good and I use them constantly. They are correct for exactly three things: analytics and reporting that is explicitly allowed to be stale and says so in the UI ("as of 09:41"); genuinely cold reads with no causal relationship to recent writes — public catalogs, archived content, aggregate counts nobody acts on; and high availability, which is a *separate* replica, sized like a primary, that you do not point query traffic at.

Everything else wants the boring version. **Make the primary the default and the replica an explicit, ugly, opt-in call** — `with replica_session(stale_ok="reporting"):` — so that every stale read is a deliberate sentence someone typed on purpose, not an inherited default. Pin a session to the primary for the remainder of any request that performed a write; one flag on the request context, twenty lines, and your entire read-after-write class of bug is gone forever. If you must read from a replica after a write, use the primitive your database already gave you: capture the LSN at commit and wait for it — `pg_wal_lsn_diff`, or MySQL's `WAIT_FOR_EXECUTED_GTID_SET`, which exists precisely for this and which I have never once seen in a codebase I didn't write it into myself. Put a `CHECK (balance >= 0)` on the table, because the constraint is enforced on the primary at write time and does not care what any replica believed.

Then measure the thing that is actually true: `now() - pg_last_xact_replay_timestamp()` on every replica, alerting on both "too high" *and* "suspiciously flat," and set `synchronous_commit = on` with `synchronous_standby_names = 'ANY 1 (...)'` if you genuinely cannot lose acknowledged writes — then watch your write latency go up and have the honest conversation about which you'd rather have. That conversation takes twenty minutes and nobody has it, so the default decides, and the default was chosen by a Postgres maintainer optimizing for benchmarks, not for your payment ledger.

And do the cheap thing nobody does: **`grep -rn "session()\|get_read_session\|using('replica')" .`, take the 847 hits, and for each one ask the single question "could the caller have written something relevant in the last five seconds?"** You will find between four and forty paths where the answer is yes, at least one of which explains a bug sitting in your backlog right now marked `cannot reproduce`. It is the cheapest hour of debugging available to you this quarter, and nobody will schedule it, because the replica has never failed — it has only ever told the truth about a moment that had already passed.

---

*The author once added a replica to fix a slow dashboard, then routed the login path to it "temporarily" during an incident. Password changes took effect on the primary and were invisible to the replica for up to a minute, so users who reset their password were told it was wrong, tried again, and locked themselves out via the rate limiter — which counted failures on the primary, instantly and accurately. He diagnosed this in eleven minutes. It took him four months to admit, in writing, that the replica had been doing precisely what it was built to do.*
