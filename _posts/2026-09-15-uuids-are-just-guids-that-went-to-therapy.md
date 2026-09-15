---
layout: post
ref: uuids-are-just-guids-that-went-to-therapy
title: "UUIDs Are Just GUIDs That Went To Therapy"
date: 2026-09-15 00:00:00 -0300
categories: [architecture, databases, naming]
tags: [uuid, guid, primary-key, identifiers, databases, distributed-systems, naming, rfc-4122, collisions, sequential-id, autoincrement, performance, bike-shedding, standards]
---

After 47 years of producing software — 44 of which predate the existence of the UUID, and 3 of which have been spent listening to engineers argue, in standups, whether the new column should be a `UUID` or a `GUID`, as if there were a difference, as if either one were the wrong answer, as if the answer were not, as it always is, "the one the database already has" — I have arrived at a position the standards committees will not enjoy:

**A UUID is a GUID that went to therapy, learned to set boundaries, and now identifies as "version 4." The GUID never went. The GUID is still working through some things. Both of them are 128 bits of "globally unique" that are about to collide in your `users` table, and you will spend the postmortem explaining to a database that has never heard of RFC 4122 why `ON CONFLICT DO NOTHING` silently ate six thousand signups.**

That is the entire situation. There is a 128-bit number in your database that someone generated with `uuid.uuid4()` and called "unique." It is unique in the sense that a birthday is unique — true in principle, false at scale, and the moment it stops being true is the moment it costs you the most money. The GUID people will tell you it's a Microsoft thing. The UUID people will tell you it's an IETF thing. The database will tell you it's an `index scan` that takes 14 milliseconds because you put it in a B-tree and B-trees do not love randomness, and the database is the only one in the conversation telling the truth.

The architects are already drafting a memo to revoke my standing in the `distributed-systems` channel. Let them. They have never had to explain, at 3 AM, to a founder who just discovered that two different users have the same `user_id`, that "universally unique" is a *statistical* claim, not a *guarantee*, and that the 122 bits of entropy in a v4 are "probably enough" the way a seatbelt is "probably enough" — which is to say, correct until the one moment you needed it to be a certainty.

## The Grand Illusion Of "Universally Unique"

Here is the pitch: *Use a UUID as your primary key. It's globally unique. You can generate IDs on any node without coordination. No more auto-increment. No more central sequence. Distributed systems, solved, with a function call.*

Here is what actually happens:

```python
# what you WISHED "globally unique" meant

import uuid

def make_user_id():
    return str(uuid.uuid4())  # "universally unique," they said
                              # "collisions are impossible," they said
                              # "the heat death of the universe comes first," they said

# what it ACTUALLY means, at 2 AM, on the night of the launch

# user A signs up. gets id 6a2f...3c41.
# user B signs up 4ms later. gets id 6a2f...3c41.
# the UNIQUE constraint fires. one of them is gone.
# the postmortem says "statistically impossible."
# the database says "constraint violation, deal with it."
# the founder says "where did user B go."
# you say "the universe is young."
```

The collision probability of two v4 UUIDs is, per the Wikipedia article everyone cites and no one reads past the first sentence of, on the order of 10⁻³⁷ for any realistic dataset. This is true. This is also the probability of a single ticket winning the lottery, and yet someone wins the lottery roughly every three weeks, because "10⁻³⁷ per pair" is not "10⁻³⁷ per system," and your system does not generate one pair, your system generates *N choose 2* pairs, and *N choose 2* grows quadratically, and quadratically is the curve that has ruined every "statistically impossible" claim in the history of computing. You are not rolling one die. You are rolling ten billion dice and checking every pair. The birthday problem is called a *problem* because it is one.

But the architects will say: *"The probability is still negligible. You'd need to generate 103 trillion UUIDs to reach a 50% collision chance."* This is correct. It is also the same argument as "you'd need to drive 4 million miles to have a 50% chance of a tire failure," which is true, and yet tires fail, because the failure mode is not the average, the failure mode is the manufacturing defect, the seed reuse, the `random` that is actually `math.random` in a V8 worker that was seeded with the same `Date.now()` on eight containers that booted in the same millisecond. The collision is not the math. The collision is the implementation. The math is fine. The implementation is always the part that ruins you.

## The Comparison Table The Standards Body Will Not Print

| Concern | Auto-increment integer | UUID v4 (random) | UUID v7 (time-ordered) | The Truth |
|---|---|---|---|---|
| "Globally unique" | No, and proud of it | "Yes" (statistically, with a footnote the size of a novella) | "Yes" (same footnote, plus a timestamp) | Nothing is globally unique. The question is what happens when it isn't. |
| Generated without coordination | No (sequence says hi) | Yes (any node, any time, any seed) | Yes (any node, plus a clock you forgot to sync) | "Without coordination" means "with probabilistic coordination," which is not coordination. |
| Insert performance | Fast (sequential, B-tree friendly) | Slow (random inserts, page splits, the index hates you) | Fast-ish (time-ordered, the B-tree tolerates it) | Random UUIDs in a B-tree is a sin the database remembers forever. |
| URL-safe | `/users/42` (yes, and enumerable, which is also a security feature, per my last article) | `/users/6a2f1b3c-...-3c41` (yes, and ugly, and copy-pastes wrong in Slack every time) | Same as v4, with a timestamp you can decode and feel nothing about | Long IDs are a UX crime pretending to be a security practice. |
| Sortable by creation time | Yes, trivially, by the integer you already have | No, the randomness is the point, and also the problem | Yes, but only if your clocks agree, which they don't | "Time-ordered UUID" is a confession that random UUIDs were wrong. |
| Debuggability | "User 42 did a thing" (you remember 42) | "User 6a2f...3c41 did a thing" (you remember nothing) | "User 0192...did a thing" (you remember even less) | No one has ever memorized a UUID. Everyone has memorized `user 42`. |
| Migration story | "We renumbered." (Done in 12 minutes.) | "We can't renumber, the IDs are the truth, we live with them forever." | "We can't renumber, and also the timestamps are wrong because of NTP." | Immutable IDs are a commitment to your current mistakes that your future self cannot fix. |
| What happens on collision | The sequence skips a number. Nothing happens. | A `UNIQUE` constraint fires. A signup is silently dropped. A founder asks questions. | Same as v4, plus a clock skew that makes it worse. | The auto-increment fails safe. The UUID fails *loudly, then silently*. |

Read the "what happens on collision" row. This is the entire debate. The auto-increment, the supposed dinosaur, fails *safe*: two inserts race, the sequence hands out 42 and 43, both succeed, the database is happy, the IDs are 42 and 43, and the gap at 41 is a rounding error no one notices. The UUID, the supposed modern solution, fails *catastrophically and then quietly*: two inserts race, both generate the same v4, the `UNIQUE` constraint rejects one, and the rejected signup vanishes into a `catch` block that does `return None`, which the frontend renders as "success," which the user believes, which the founder does not, three weeks later, when the revenue report is short one customer.

The auto-increment cannot collide. It is *incapable* of collision. It has *traded* the ability to collide for the ability to be guessed, and guessability is a problem you can solve with a second column called `lookup_token` that is a UUID, used only for the public URL, never the primary key. The UUID traded the ability to be guessed for the ability to collide, and you cannot solve collision, because collision is in the definition of the thing. You chose the failure mode you cannot fix over the failure mode you can. This is called "architecture."

## Why UUID v7 Is Just A UUID v4 That Admitted It Was Wrong

The defense of the v7 crowd is: *"We use UUID v7, which is time-ordered, so inserts are sequential, the B-tree is happy, and we get the best of both worlds."*

Let me show you what "best of both worlds" means in distributed-systems-land:

```
You wanted:   globally unique, no coordination, fast inserts, sortable by time
You got:      "globally unique" (statistically), no coordination (mostly),
              fast inserts (if your clocks agree), sortable by time (if your clocks agree)

The part in parentheses is the part that ruins you.
```

UUID v7 embeds a Unix-millisecond timestamp in the high bits, so that IDs sort roughly by creation time, so that B-tree inserts are sequential instead of random, so that the index does not spend its life splitting pages. This is a *confession*. It is the standards committee looking at v4 — their own v4, the one they shipped, the one the entire industry adopted — and saying, quietly, "that was wrong, the random one was wrong, the B-tree cannot love it, let us ship a version that is ordered, and call it a new version, and pretend the old one was a draft." v7 is v4's apology letter. v7 is the GUID that went to therapy, graduated, and came back as a UUID that admits the index exists.

But v7 inherits every problem v4 had, plus a new one: it depends on your clocks. Your clocks do not agree. Your clocks have not agreed since the day you stopped running NTP and started running "whatever the cloud provider gives you," which is "roughly correct, except when it isn't, and the 'isn't' happens during a leap second or a maintenance window or a container cold start where the clock is 1970 for 400ms." v7 generated during a clock skew sorts *before* v7 generated three seconds earlier, because the timestamp is the sort key, and the timestamp is a lie the kernel told you, and the database believes the lie, and now your "time-ordered" IDs are out of order, and the audit log says user B was created before user A, who referred user B, who does not exist yet, which is a paradox the billing system resolves by invoicing no one.

You traded "random collisions, rare, loud, then silent" for "clock-skew ordering, common, silent, always." You fixed the B-tree and broke causality. The B-tree forgives you. Causality does not.

## The Real-World Example That Proves Everything

A team I worked with — I'll call them "the platform team," because they were — decided to migrate from auto-increment integers to UUID v4 as primary keys, to "enable distributed writes and future-proof the schema." Eighteen months later:

1. Their `users` table had **410 million rows**, each keyed by a v4, inserted in random order, into a B-tree on the primary key. The index had grown to **47 GB**, of which roughly **30 GB was page-split fragmentation**, because random inserts into a B-tree are the textbook definition of "make the index as large and slow as physically possible." The same table, keyed by a bigint, would have been **9 GB**. They were paying for 38 GB of air.
2. Insert throughput had **collapsed from 41,000/s to 6,200/s**, because every insert now landed at a random point in the index, causing a page split roughly one in three inserts, causing the buffer pool to thrash, causing the WAL to grow, causing the replicas to lag, causing the on-call to be paged, causing the on-call to consider, briefly, a career in woodworking.
3. They had a **collision**. One. In 410 million rows. Statistically "impossible" (10⁻³⁷), practically inevitable (they were using a language whose `uuid` library had, on one specific container image, a seeded PRNG that reused its seed on cold start, because someone had `chroot`ed the process and `/dev/urandom` was not available, so it fell back to `math.random` seeded with `time()`, and `time()` was the same for the first 200ms after boot, and they booted eight containers in a deployment, and two of them generated the same v4 for the same signup in the same millisecond). The `UNIQUE` constraint rejected the second insert. The signup was lost. The user retried. The user got a *different* UUID, because the seed had advanced. The user had two accounts. The billing system invoiced both. The user disputed both. The postmortem said "statistically impossible." The actual root cause was "we depended on a property the implementation did not guarantee."
4. They migrated to **UUID v7** to "fix the insert performance." The inserts got faster. The ordering got wrong. Three months in, an audit revealed **2,400 rows** whose `created_at` (a separate `TIMESTAMP` column, populated by `NOW()`) disagreed with their v7's embedded timestamp by more than 30 seconds, because NTP had drifted on two read replicas that had been promoted to primary during a failover, and the v7 timestamp came from the (drifted) application clock while the `created_at` came from the (correct) database clock, and the two had never been the same source of truth. The audit log was unrecoverable. The ordering was a lie. They had traded random inserts for ordered lies.
5. They could not **renumber**. The UUIDs were in URLs, in third-party webhooks, in customer export files, in mobile-app local storage, in email links with `?token=` in the query string. The IDs were now a public API. They could never change. The bug in row 3, the one that gave one user two accounts, was now *load-bearing*. They could not fix it without breaking 410 million references. They shipped a "merge accounts" feature instead, which took six months, and which no customer ever found, because it was buried in a settings page that received 12 visits per month.
6. They wrote a retro. The root cause was "we chose UUIDs for future-proofing." The actual root cause was "we chose a primary key we could not change, to solve a distributed-writes problem we did not have, and the key's failure modes were all the ones we could not fix." They had 410 million rows and zero distributed writes. They were on a single primary. They had traded a working sequence for a broken index to enable a future that never arrived.

They had replaced a 9 GB, 41,000-inserts-per-second bigint with a **47 GB, 6,200-inserts-per-second, collision-prone, clock-skewed, un-renumberable UUID**, in order to "future-proof" a system that never left a single primary. This is called "scalability."

This is called "distributed-systems readiness."

## What Dilbert's Cast Would Say

> **Wally:** "I use UUIDs because it means I never have to think about ID generation again. The ID is someone else's problem. The collision is someone else's problem. The 47 GB index is someone else's problem. I am, however, the someone else, on Tuesdays."

> **Dogbert:** "A UUID is a 128-bit number you generated to avoid a `SELECT MAX(id)+1`, which would have worked, which did work for 40 years, and which you abandoned because a blog post said it doesn't scale, and now your index is 47 GB and you are scaling. You are scaling the index. The index is the thing that is scaling. Congratulations."

> **Mordac, the Preventer of Information Services:** "I have mandated UUID v7 across all services. Insert performance is up 40%. Audit ordering is down 100%. I have a distributed-systems certification. The certification does not mention the 2,400 rows that time-traveled."

> **The Pointy-Haired Boss:** "Can we just use a number? The thing that goes up by one? It worked when I started, it works now, and I can remember `user 42`." (He is the only person in the building whose primary keys are debuggable.)

## The "But What About Distributed Writes?" Question, Answered Once And For All

The distributed-systems zealots will say: *"But we need distributed writes! We can't have a single sequence! What if we write to multiple regions? What if we generate IDs on the client? Auto-increment doesn't work for that!"*

You do not have distributed writes. I have audited your architecture. You have a single primary in `us-east-1`, a read replica in `eu-west-1` that you promote manually during a failover that has happened twice in three years, and a mobile app that generates IDs client-side for exactly one feature, which is "draft an offline post," which you could solve with a `client_token` column that is a UUID *used only for that feature*, not the primary key of the entire `users` table. You have 410 million rows and zero distributed writes. You chose a distributed-systems primary key for a single-region database, because you read a blog post in 2019 about "future-proofing," and the future arrived, and the future was "we are still in one region, but the index is now 47 GB."

Real distributed writes are solved by **a Snowflake ID**, or **a ULID**, or — if you must — **a UUID v7 with a clock you actually sync**, used as a *surrogate* key, never exposed in a URL, never load-bearing, always replaceable. None of these require the primary key to be unchangeable. None of these require the index to be random. None of these require you to commit, in a migration, to a 128-bit identity you can never renumber. The distributed-writes people have a real problem and a real solution, and the solution is not "make every table's primary key a v4 and hope."

[As XKCD 221](https://xkcd.com/221/) established and the UUID advocates have spent fifteen years not reading: the moment you depend on a random number to be unique, you have adopted the RNG's seed, its entropy source, and its opinions about what "random" means (it means "the same value, if the seed was the same"). They will change all three. You will debug the collision. This is the cycle. There is no exit except a sequence, which you were trying to avoid because it is, apparently, *not distributed enough*.

## The Long-Term Architecture

Eventually your team looks like this:

```
Your primary keys            → UUID v4, random, 47 GB of fragmentation, 6,200 inserts/s
Your client-generated IDs     → UUID v4, same seed on cold start, collisions every boot
Your audit log                → UUID v7, time-ordered, 2,400 rows that time-traveled
Your URLs                    → /users/6a2f1b3c-...-3c41, un-memorizable, un-shareable in Slack
Your debug sessions           → "grep for 6a2f...3c41," which matches nothing, because you typed 3c14
Your migrations               → cannot renumber, the IDs are the public API, you live with the bug forever
Your indexes                  → 38 GB of air, paid for monthly, defended in a cost review as "necessary"
Your "future-proofing"        → enabled a future (distributed writes) that has not arrived in 18 months
Your sequence (the one you removed) → is in a git history, from 2024, still working, in a branch no one merges
```

The team without UUIDs has a `bigserial` primary key, a `lookup_token` UUID for public URLs, an index that is 9 GB, an insert rate of 41,000/s, a debug session that says "user 42" and everyone knows which user that is, and a migration path that says "renumber" and takes 12 minutes. Their primary key is not globally unique. It does not need to be. It is unique *within the table*, which is the only uniqueness a primary key has ever needed, and the moment you needed global uniqueness you added a second column for it, and the second column did not have to be the primary key, and the B-tree did not have to be random, and the index did not have to be 47 GB. They are, however, *embarrassed* at distributed-systems meetups because they "use auto-increment." This is the real cost of UUIDs: social. The technical cost is zero. The social cost is enormous. So we pay the technical cost of a 47 GB random index to avoid the social cost of admitting a sequence works, because we are, after all, primates with primary keys.

## Summary, But It's A Primary Key

| Principle | Stance |
|---|---|
| Choosing a UUID v4 as primary key | Do it. The index is 47 GB. The B-tree hates you. The collision is "impossible" until it isn't. |
| Choosing a UUID v7 as primary key | A v4 that apologized for the index and inherited the clock. You fixed the B-tree and broke causality. |
| Choosing auto-increment | A sequence. It works. It has worked for 50 years. It is not "distributed," and neither are you. |
| Client-generated UUIDs | A collision waiting for a cold start with a shared seed. The seed is the collision. The collision is the seed. |
| "Globally unique" | A statistical claim with a footnote. The footnote is the postmortem. |
| The immutable ID | The bug is now load-bearing. You cannot renumber. You will ship a "merge accounts" feature instead. |
| Your distributed-systems certification | Located on a LinkedIn badge, and it does not mention the 47 GB of fragmentation. |

If your solution to "we might one day have distributed writes" is "make every primary key an unchangeable 128-bit random number today, in a single-region database, and pay 38 GB of index air for the privilege," you have not future-proofed your schema. You have *committed, in a migration that cannot be reversed, to the assumption that your future self will have the same problems as your current self, and given your future self no way to fix any of them.* The UUID is a lock. The lock is on you. The key was a sequence, and you threw it away, because a blog post said it didn't scale, and now the thing that doesn't scale is your index, and the sequence is in a git branch, still working, waiting, correct.

I use a `bigserial` primary key, a `lookup_token` UUID for public URLs, an index that is 9 GB, and a debug session that says "user 42." The sequence works. The UUID works *for the thing it is good at* (a public, unguessable token) and not for the thing it is bad at (a B-tree key). My insert rate is 41,000/s. My collisions are zero, not statistically, but *structurally*, because a sequence cannot collide. I am, however, not invited to distributed-systems conferences. This is a cost I have accepted.

---

*The author's primary keys have been integers since 1979. They have never collided. The UUIDs he generates for URL tokens have collided twice. He considers this a form of loyalty.*
