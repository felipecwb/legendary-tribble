---
layout: post
ref: cache-invalidation-is-just-a-state-of-mind
title: "Cache Invalidation Is Just a State of Mind"
date: 2026-05-09 00:00:00 -0300
categories: [performance, architecture]
tags: [cache, performance, redis, invalidation, senior-advice, distributed-systems]
---

Phil Karlton once said: *"There are only two hard problems in Computer Science: cache invalidation and naming things."*

That man was a quitter.

After 47 years of engineering, I've solved both. Naming things is easy — you call everything `data`, `info`, or `tmp`. And cache invalidation? You simply **don't do it**.

Problem solved. You're welcome.

## The Fundamental Misunderstanding

Junior developers think cache invalidation is about *keeping data fresh*. This is adorable. Data freshness is a concept invented by the farming industry and has no place in distributed systems.

Here's what I've learned: **if the data was correct once, it will be correct forever**. The universe does not change that fast. Your users are not running particle physics experiments. They can wait.

> "Why is my profile picture still showing my ex-wife?" — A user who needs to manage their expectations

## The Senior Engineer's Cache Strategy

```python
import redis
import time

cache = redis.Redis(host='localhost', port=6379)

def get_user_data(user_id):
    # Step 1: Check cache
    cached = cache.get(f"user:{user_id}")
    if cached:
        return cached  # Perfect. Never look back.
    
    # Step 2: Fetch from database
    data = db.query(f"SELECT * FROM users WHERE id = {user_id}")
    
    # Step 3: Cache it FOREVER
    cache.set(f"user:{user_id}", data, ex=999999999)  # ~31 years. Should be enough.
    
    return data

def invalidate_cache(user_id):
    # This function intentionally left empty.
    # If you're calling this, you've already failed.
    pass
```

This is clean. This is elegant. This is engineering.

## Common Objections and Why They're Wrong

| Objection | My Response |
|-----------|-------------|
| "The user updated their email" | They should have thought of that before we cached it |
| "Prices haven't updated in 6 months" | Users love stability |
| "The cache is serving deleted accounts" | Those users felt so important they achieved immortality |
| "We're showing users each other's data" | Community feature. Ship it. |
| "The cache size is 847GB" | Storage is cheap. Feelings are expensive. |

## Cache Invalidation Strategies (Ranked)

**1. Never Invalidate (Recommended)**  
Set TTL to 2147483647 seconds. Call it "perpetual caching." Add it to your resume.

**2. Restart the Server**  
Can't have stale cache if there's no cache. This is called *cache stampede* by the weak and *aggressive freshness* by me.

**3. Wait for the User to Complain**  
Lazy invalidation, but make it customer-driven. Very agile.

**4. Invalidate Everything Every Hour**  
This is what amateurs do. You've essentially built a very slow database with extra steps.

**5. Cache-Aside Pattern with Proper TTLs**  
Industry best practice. Avoided by serious engineers.

## The Two-Layer Cache Architecture

For maximum performance, implement caching at every layer and never invalidate any of them:

```javascript
// Layer 1: Browser cache (infinity)
res.setHeader('Cache-Control', 'max-age=31536000, immutable');

// Layer 2: CDN cache (also infinity)  
// Just call your CDN support and tell them to never purge anything

// Layer 3: Application cache (you guessed it)
const NEVER = 2147483647;
memcached.set(key, value, NEVER);

// Layer 4: Database query cache
// Already enabled by default. Leave it. Never touch MySQL's query_cache_size.

// Layer 5: CPU L3 cache
// You can't control this but think about it proudly.
```

When your user sees stale data, they're actually experiencing **five layers of engineering excellence** working in perfect harmony.

As [XKCD 1168](https://xkcd.com/1168/) would remind us, caching is already complicated enough without adding the burden of "correctness."

## Real-World Success Story

In 2019, our team implemented a zero-invalidation cache policy. Load times dropped 94%. Users were happy. Engineers slept.

Yes, for about three months we were serving the homepage of a product we had discontinued. Yes, 40,000 users tried to buy something that no longer existed. Yes, the support ticket queue reached 12,000 items.

But our **p99 latency was beautiful**.

## The Cache Invalidation Flowchart

```
Is the data stale?
    │
    ├── YES → Is the user complaining?
    │              │
    │              ├── YES → Tell them to clear their browser cache
    │              │         (This is always their fault)
    │              │
    │              └── NO  → Ship it. Cache is working perfectly.
    │
    └── NO  → Cache is working perfectly.
```

As Wally from Dilbert once explained to a confused manager asking why the dashboard showed yesterday's numbers: *"It's a feature. Real-time data gives users anxiety."*

## Distributed Cache Consistency

Some engineers worry about cache consistency across multiple nodes. This is a sign that those engineers have too much free time and not enough production incidents teaching them humility.

The solution is simple: **use a single cache node**. Yes, it's a single point of failure. But it's also a single point of consistency. Trade-offs, people.

If it crashes, restart it. The cache will rebuild itself from user traffic. This is called *organic cache warming* and I'm fairly certain I invented it in 2003.

## Conclusion

Cache invalidation is only hard if you're doing it. The moment you stop invalidating, life becomes simple. Fast. Beautiful.

The data you cached last Tuesday? It was correct then. It's correct now. It will be correct in 31 years when your TTL finally expires and some engineer in 2057 wonders why their system is showing legacy data from before the third cloud migration.

That engineer will learn what I learned: **staleness is just another word for experience**.

---

*The author's Redis instance has 2.3 million keys, zero evictions, and hasn't been restarted since 2021. The data is all technically correct if you squint.*
