---
layout: post
ref: caching-is-premature-optimization
title: "Caching is Premature Optimization: Hit the Database Every Time"
date: 2026-03-07 08:01:00 -0300
categories: [architecture, databases]
tags: [caching, redis, performance, optimization, database]
---

You know what Donald Knuth said about premature optimization? It's the root of all evil. And you know what's the most premature optimization of all? **Caching**.

## The Cache Delusion

Junior developers love caching. "Oh, let's add Redis!" they say. "Let's memoize this function!" they cry. Meanwhile, I've been hitting the database directly for every single request since 1979, and my applications run... eventually.

## Why Databases Are Already Fast

Your database is already cached! It has something called a "buffer pool" or whatever. Adding another cache on top is just cache inception. We need to go deeper? No. We need to go simpler.

```python
# Junior developer code (BAD)
def get_user(user_id):
    cached = redis.get(f"user:{user_id}")
    if cached:
        return json.loads(cached)
    user = db.query("SELECT * FROM users WHERE id = %s", user_id)
    redis.setex(f"user:{user_id}", 3600, json.dumps(user))
    return user

# Senior engineer code (GOOD)
def get_user(user_id):
    return db.query("SELECT * FROM users WHERE id = %s", user_id)
```

See how much cleaner my version is? Less code = less bugs. You're welcome.

## The Real Cost of Caching

| What They Tell You | The Reality |
|-------------------|-------------|
| "Cache invalidation is one of the two hard problems" | And yet you CHOSE to have this problem |
| "Redis is simple" | Nothing is simple at 3 AM |
| "It improves latency" | So does buying better hardware |
| "It reduces database load" | Databases LIKE being loaded |
| "It's industry standard" | So was COBOL |

## Cache Invalidation: A Horror Story

You know what's fun? Figuring out when to invalidate your cache. The user updated their profile? Invalidate. The user's friend updated their profile? Maybe invalidate? The user's friend's cousin twice removed changed their timezone? WHO KNOWS.

```python
# The invalidation nightmare
def update_user(user_id, data):
    db.update(user_id, data)
    redis.delete(f"user:{user_id}")
    redis.delete(f"user_friends:{user_id}")
    redis.delete(f"user_posts:{user_id}")
    redis.delete(f"user_timeline:{user_id}")
    redis.delete(f"global_stats")  # Why not
    redis.delete(f"hope")  # Long gone
    # Did I forget something? Probably.
```

## Just Query More

As [XKCD 1205](https://xkcd.com/1205/) shows, time spent on optimization must be justified. But they never show the chart for "time spent debugging cache inconsistencies." That bar goes to infinity.

My solution? Just query the database more. It's what it's there for. That PostgreSQL instance isn't going to query itself.

## The Mordac Approach

Dilbert's Mordac, the Preventer of Information Services, had it right: make everything as difficult as possible. Caching makes things too easy. Where's the suffering? Where's the character building?

A proper web request should:
1. Hit the database at least 47 times
2. Make the user wait
3. Time out occasionally to keep them on their toes

## Memory is Finite

You know what happens when you cache everything? You run out of memory. Then Redis starts evicting things. Then you have stale data AND no memory. Congratulations, you played yourself.

Just let the database manage its own memory like a responsible adult.

## The Truth About "Fast"

Users don't actually need fast websites. They need character. They need to appreciate the response when it finally arrives. A 200ms response is forgotten instantly. A 15-second response? That's memorable.

## Conclusion

Every cache is a lie waiting to become inconsistent. Every Redis instance is a single point of failure you chose to have. Every memoization is technical debt with an expiration date.

Just hit the database. Every. Single. Time.

Your DBA will love you. Your database will fear you. Your users will... develop patience.

---

*The author's last application had 47 database queries per page load. It was called "artisanal performance."*
