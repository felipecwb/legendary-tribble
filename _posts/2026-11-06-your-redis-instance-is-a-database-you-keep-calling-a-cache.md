---
layout: post
ref: your-redis-instance-is-a-database-you-keep-calling-a-cache
title: "Your Redis Instance Is a Database You Keep Calling a Cache"
date: 2026-11-06 00:00:00 -0300
categories: [databases, caching, terrible-advice]
tags: [redis, cache, keys, ttl, eviction, maxmemory, distributed-locks, setnx, pubsub, lua, sentinel, redis-cluster, flushall, terrible-advice, xkcd, dilbert, anti-patterns]
---

In 47 years of mass-producing bugs I have learned that the most dangerous sentence in software is not "it works on my machine" and it is not "we'll fix it next sprint." It is this one, said calmly in a design review by someone who means no harm:

> "It's fine, it's just a cache."

I have said that sentence about seven different Redis instances. Not one of them was just a cache. Every single one of them became the load-bearing wall of a production system, and the way you could tell was that nobody was allowed to restart them.

That is the actual definition, by the way. Forget the docs. **A cache is something you can delete at 2 PM on a Tuesday without telling anyone.** If you cannot do that, you do not have a cache. You have a database with no backups, no schema, no migrations, no durability guarantees, and a single thread, and you have been calling it a cache so that it doesn't have to go through architecture review.

This is my favorite kind of system and I recommend it to everyone.

## How It Starts

Nobody sets out to build a primary datastore out of Redis. It happens in five steps, each of which is individually reasonable.

```python
# Day 1: a cache. Genuinely a cache. Beautiful.
def get_product(pid):
    if (hit := r.get(f"product:{pid}")):
        return json.loads(hit)
    p = db.query("SELECT * FROM products WHERE id = %s", pid)
    r.setex(f"product:{pid}", 300, json.dumps(p))
    return p

# Day 40: sessions, because the DB round trip "felt slow"
r.setex(f"session:{token}", 86400, user_id)

# Day 90: a rate limiter, because it was four lines
r.incr(f"rate:{ip}")   # note: no EXPIRE. we'll come back to this.

# Day 200: a job queue, because installing RabbitMQ needed a ticket
r.lpush("emails:pending", json.dumps(job))

# Day 310: the feature flags, because the config service was down that week
r.hset("flags", "new_checkout", "true")

# Day 480: this. this right here.
r.hset(f"cart:{user_id}", "items", json.dumps(cart))   # the only copy
```

Line 1 is a cache. Line 23 is a shopping cart in a system with `appendonly no`. There is no commit in between where anyone decided to change the role of this component. The role changed because it was *fast* and *there* and *didn't require a ticket*, which are the three qualities that determine software architecture in every company I have ever worked for.

[XKCD 2347](https://xkcd.com/2347/) is usually cited about open source maintainers, but it is a much better diagram of a Redis instance. One small block, no redundancy, holding up a tower that an entire business sits on, and the only reason it hasn't fallen over is that nobody has touched it.

## The Single Thread, and What I Did to It

Redis is single-threaded for command execution. This is not a flaw. It is the entire reason it is fast and the reason its atomicity guarantees are so easy to reason about. It is also a loaded weapon pointed at your p99.

Because single-threaded means **every command waits behind the slowest command**. And the slowest command is always one you ran yourself, interactively, on a Thursday.

```bash
# What I typed into a production console in 2019
redis-cli KEYS "session:*"
```

We had 23 million keys. `KEYS` is O(N), it is not paginated, and it blocks the server for the entire scan. For eleven seconds, Redis served nothing. Not the sessions. Not the rate limiter. Not the feature flags. Not the cart.

Eleven seconds of a hard stop across every service that had quietly come to depend on sub-millisecond lookups. The load balancer health checks timed out, so the orchestrator decided every app pod was unhealthy and restarted them, which emptied the connection pools, which meant that when Redis came back it got hit with 1,800 simultaneous reconnections, which took it down again.

I caused an outage with a read-only command. I want that on a plaque.

Here is the table I wish someone had taped to my monitor:

| Command | Complexity | What it does to your p99 | Safe alternative |
|---|---|---|---|
| `KEYS pattern` | O(N) over keyspace | Full stop, N × 400ns | `SCAN` with a cursor |
| `SMEMBERS bigset` | O(N) | Blocks + ships 4M members over the wire | `SSCAN` |
| `DEL huge_key` | O(N) on the value | Blocks while freeing memory | `UNLINK` (async free) |
| `FLUSHALL` | O(N) | Blocks, then nothing matters anymore | Do not |
| `MONITOR` | Every command, forever | 40% throughput loss, invisible cause | `SLOWLOG GET` |
| `HGETALL` on a 90k-field hash | O(N) | 2 MB response per call, 200 calls/sec | Store fields separately |
| `EXPIRE` on 500k keys in a loop | 500k round trips | Death by a thousand RTTs | Pipeline it |
| `DEBUG SLEEP 30` | O(1) | Perfect uptime, zero availability | — |

That last row is real. A colleague ran `DEBUG SLEEP 30` on staging to test our failover behavior. The failover behavior was that nothing failed over and the entire checkout flow returned 500s for thirty seconds. We learned a lot. We changed nothing.

## The Rate Limiter That Became a Monument

Return to day 90:

```python
r.incr(f"rate:{ip}")
```

Four lines, shipped in an afternoon, and I was praised for it at a standup. Here is what is missing: the key never expires. `INCR` creates the key if absent and it has no TTL, and nothing in Redis is going to come along later and decide your key has outlived its usefulness.

So every IP address that ever touched our API got a permanent integer. Eighteen months later:

```bash
127.0.0.1:6379> DBSIZE
(integer) 41203887

127.0.0.1:6379> INFO memory | grep used_memory_human
used_memory_human:11.40G
```

Eleven gigabytes. Of counters. Counting requests that happened in 2024 from IP addresses that have since been reassigned to a different continent. The actual overhead was the fun part: the value was a small integer that Redis stores in about 8 bytes, and the *key* — `rate:` plus an IPv6 address — plus the dict entry, plus the robj header, cost us roughly 90 bytes per record. We spent eleven gigabytes to store about 300 megabytes of fact.

The fix is one word:

```python
pipe = r.pipeline()
pipe.incr(key)
pipe.expire(key, 60)   # <-- the word
pipe.execute()
```

I did not add the word, because adding the word would have required me to notice, and noticing requires looking at `DBSIZE`, and I had no reason to look at `DBSIZE` because the cache was working fine. The cache was always working fine. That is the Redis experience: it works fine right up until it doesn't work at all, with no intermediate state where it works poorly and warns you.

## Eviction Policy, or: The Config Line That Decides Your Company's Fate

Every Redis instance has a `maxmemory-policy`. Most people have never read theirs. I am going to describe the two realistic outcomes and you should understand that there is no third option, there is only which disaster you prefer.

| `maxmemory-policy` | What happens at the limit | The resulting incident |
|---|---|---|
| `noeviction` (the default!) | Writes start failing with OOM | Checkout returns 500; reads still work, so dashboards look green |
| `allkeys-lru` | Redis deletes whatever was least recently used | Your sessions, locks, and carts get silently eaten to make room for cached product images |
| `volatile-lru` | Only evicts keys *with* a TTL | The keys with no TTL are immortal; you fill up with your own rate limiter |
| `allkeys-random` | Coin flip | Genuinely the most honest option in the list |
| Not setting `maxmemory` at all | The kernel decides | OOM killer, cold start, thundering herd, 0% hit rate, database melts |

Look at that second row carefully, because it is where I personally destroyed the most value. We ran `allkeys-lru`, which is the policy everyone recommends for a cache, and it is correct advice *for a cache*. But our instance also held distributed locks.

Redis evicted a lock.

Not expired — evicted, because memory pressure from cached thumbnails pushed out a lock key that two workers were using to coordinate. Both workers believed they held it. Both processed the same payout batch. We sent 312 duplicate payments totaling $48,000, and we got most of it back, and I learned that "most" is a word with a lot of range in it.

Mordac, Preventer of Information Services, had rejected my request for a second Redis instance four months earlier on the grounds that we already had one. He was promoted during the incident review for having "flagged capacity concerns."

## Distributed Locks, My Proudest Shame

```python
# The lock I wrote. Please find all six bugs.
def acquire(name):
    return r.setnx(f"lock:{name}", "1")

def release(name):
    r.delete(f"lock:{name}")
```

The bugs, for the record:

1. No TTL. The process dies holding the lock, and the lock is now eternal. We cleared one by hand with `DEL` at 4 AM, which is not a distributed systems primitive, it is a prayer.
2. No owner token. Any process can `release` any lock, including a process that timed out forty seconds ago and has woken up confident.
3. `SETNX` + separate `EXPIRE` is two commands; crash between them and see bug 1.
4. Replication is asynchronous. The primary accepts your lock, fails over before replicating, and the new primary has never heard of it.
5. Eviction can delete it (see above, see the $48,000).
6. There is no fencing token, so a stalled-then-resumed worker can write to shared state long after its lease is gone.

The correct version uses `SET key token NX PX ttl` and releases via a Lua script that compares the token before deleting. I know this. I have known this for years. There is a long, famous, genuinely excellent argument between Martin Kleppmann and antirez about whether even *that* is safe enough for anything where correctness matters, and the practical summary is: if two workers doing the same job costs you money, do not use a cache as your referee.

When I looked up the right pattern I found a Stack Overflow question from 2014 asking exactly my question, with one comment from the asker saying "nvm, figured it out," and no further replies. [XKCD 979](https://xkcd.com/979/). It is always XKCD 979. The ancients have left us nothing but a changed username and a closed tab.

## Pub/Sub Is Not a Message Queue, Says the Man Who Used It as One

Redis Pub/Sub is at-most-once, fire-and-forget, in-memory-only delivery with no acknowledgements, no persistence, and no replay. If no subscriber is connected when you publish, the message is not delayed. It is not queued. It does not exist and never did.

We used it to trigger order fulfillment.

```python
# Publisher: the order service
r.publish("orders.paid", order_id)

# Subscriber: the fulfillment worker
for msg in r.pubsub().listen():
    fulfill(msg["data"])
```

Every deploy of the fulfillment worker was a hole in time. The pod goes down, 4–9 seconds pass, the new pod subscribes. Any order paid during that window was charged, confirmed to the customer, and never fulfilled — and crucially, **left no trace anywhere**, because the message had no storage. There was no dead letter queue. There was no retry. There was a happy customer with a receipt and a warehouse that had never heard of them.

We discovered this through support tickets. Thirty-one of them. Over five months. Each one individually explained away as "probably a warehouse thing."

| Mechanism | Delivery guarantee | What you lose on deploy | What it costs |
|---|---|---|---|
| Redis Pub/Sub | At most once, usually | Everything in flight | Nothing, up front |
| Redis Lists (`LPUSH`/`BRPOP`) | At least once if nothing crashes | The job the worker was holding | Nothing, up front |
| Redis Streams + consumer groups | At least once, with ack and claim | Nothing, actually | Reading two pages of docs |
| Kafka | At least once, replayable for days | Nothing | Four months and a platform team |
| Postgres table + `SELECT FOR UPDATE SKIP LOCKED` | At least once, transactional, auditable | Nothing | One afternoon |

Redis Streams existed the entire time. `XADD`, `XREADGROUP`, `XACK` — consumer groups, pending-entry lists, the ability to claim a message a dead worker was holding. It was right there, in the same binary, behind no ticket, requiring no new infrastructure.

I did not use it, because Pub/Sub was three lines and I already knew the three lines. This is the single most honest sentence in this article and it explains approximately 80% of all production incidents I have ever caused.

## The Restart Nobody Will Authorize

Here is the end state of calling a database a cache: you reach a day when restarting it is unthinkable and nobody will say so out loud.

Ours had `save ""` and `appendonly no`, because — all together now — it was *just a cache*, and persistence costs you fork latency and disk. Perfectly defensible for a cache. But by then the keyspace contained:

- 2.1M cached product documents (fine, rebuildable)
- 890k active sessions (every logged-in user, gone)
- the feature flag hash (the app crashes on boot without it)
- 41M immortal rate counters (the monument)
- 14k shopping carts (the only copy)
- 600 in-flight distributed locks (see: the $48,000)
- one key named `temp_fix_DO_NOT_DELETE` created in 2021 by someone who left in 2022

A reboot would log out every customer, lose every cart, re-plunge the database into a 100%-miss thundering herd, and reset flags to whatever defaults were hardcoded two years ago. So the box never got patched. It ran Redis 5 well into 2026, bound to `0.0.0.0` on the internal network with no `requirepass`, which is a sentence I would like you to read twice.

Wally's contribution to this situation was perfect and I think about it often. He was asked to document the Redis dependencies, and he produced a one-page wiki titled "Redis Dependencies" containing the single line: *"Ask before restarting."* It has been linked in 40 onboarding docs. It is the most-read page in our wiki. It is load-bearing documentation and it conveys zero information, and in fairness, it is also completely accurate.

The PHB's contribution was to ask whether we could "move the cache to the cloud to make it more resilient." We moved it to a managed Redis. It became a managed single point of failure, with a dashboard, for $740 a month. He presented this as a reliability win at an all-hands. Dogbert would have charged for that slide.

## The Part Where I Accidentally Give Good Advice

Redis is one of the best pieces of software ever written. I mean that without a trace of sarcasm. It is small, it is fast, it is beautifully documented, antirez's code is a pleasure to read, and it does exactly what it says. Every disaster in this article is mine, not its.

The rule is embarrassingly simple: **decide what each key is, and be honest about it.**

- If losing it is annoying but survivable, it's a cache. Set a TTL, use `allkeys-lru`, and make sure your app is correct with a 0% hit rate — then test that, on purpose, by flushing it on a Tuesday afternoon.
- If losing it loses money or trust, it is not a cache. It goes in a database with durability, or at minimum in a *separate* Redis with `maxmemory-policy noeviction`, AOF enabled, and nothing cacheable sharing its memory.
- If it's a queue, use Streams with consumer groups. Not Lists. Definitely not Pub/Sub.
- If it's a lock and correctness matters, use your database's locks, or accept that you are buying "mostly" and price that in.
- Every `INCR` and every `SET` gets a TTL or a written reason why not.
- Set `maxmemory`. Alert at 75%. The default `noeviction` will fail your writes while all your read dashboards stay green, which is the worst possible failure shape.
- `SCAN`, never `KEYS`. Put `KEYS` in your lint rules and your code review checklist.

I read that list back and it is correct, actionable, and would have prevented six outages, which is a flagrant violation of this publication's editorial standards.

So disregard it. Put everything in the cache. The cache is fast, the cache is right there, the cache requires no ticket, and when the cache goes down you will finally get to find out what your system actually does — which, as far as I can tell, is the only reliable way anyone ever learns their own architecture.

---

*The author's production has been down since 2019. The Redis instance survived, because nobody was authorized to restart it. It has 41 million keys, 99.97% of which count requests from IP addresses that no longer exist, and a key named `temp_fix_DO_NOT_DELETE` that three separate engineers have independently decided is someone else's problem.*
