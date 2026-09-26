---
layout: post
ref: your-rate-limiter-is-just-a-bouncer-who-cant-explain-the-dress-code
title: "Your Rate Limiter Is Just a Bouncer Who Can't Explain the Dress Code"
date: 2026-09-26 00:00:00 -0300
categories: [backend, distributed-systems, reliability]
tags: [rate-limiting, token-bucket, sliding-window, throttling, 429, apis, reliability, load-shedding, queues, observability, redis, leaky-bucket]
---

After 47 years of building systems — including the 4 years I spent in a windowless room watching a "smart" rate limiter reject the only customer who ever paid us on time, because the limiter's sliding window had drifted 200ms behind NTP and decided a legitimate $40,000 wire transfer was "suspicious activity" — I have a finding that the reliability engineering priesthood will attempt to suppress:

**A rate limiter is a bouncer hired to keep a club from getting too full, who has, over the course of 15 years of "hardening," been given a velvet rope, a dress code, a guest list, a fingerprint scanner, a CAPTCHA, and a machine-learning model that flags "anomalous" handshakes. He now rejects 30% of paying customers at the door, cannot tell you why, and is considered "mission-critical infrastructure." The club could have just added a second bartender. The club did not add a second bartender. The club hired the bouncer. The bouncer now has an on-call rotation.**

The Site Reliability Guild is already composing a conference talk titled "Adaptive Rate Limiting As a Reliability Primitive." Let me save them the abstract: it is not a primitive. It is a confession. It is the system saying, out loud, that it cannot handle the traffic it was built to handle, and has therefore hired a function that throws `429 Too Many Requests` at the problem until the problem goes away or, more often, retries.

## The Pitch, And Then The Reality

Here is the pitch the platform team gives in the architecture review:

> *"We'll put a rate limiter in front of every service. Token bucket per client. Sliding window for bursts. Redis-backed so it's distributed. This protects the system from abusive callers and gives us a lever during incidents."*

Here is the reality, six months later:

```python
# rate_limiter.py - "just a token bucket, simple"
RATE = 100              # requests per second, per client
BURST = 250             # burst capacity
WINDOW = 60             # sliding window in seconds
DRAIN = 0.5             # leaky bucket drain rate
STRATEGY = "sliding"    # sliding | fixed | token | leaky | "adaptive"
ANOMALY_THRESHOLD = 0.73 # ML model confidence to flag "weird" traffic
EXEMPT_KEYS = {"healthcheck", "internal-monitoring", "grafana",
               "the-ceo-ip", "the-on-call-phone", "???-3"}
SHED_ON_429 = True      # return 429... or queue? nobody remembers
QUEUE_DEPTH = 10000     # if we queue, how deep? nobody knows
DEAD_LETTER = "dlq"     # where do rejected ones go? the DLQ. who reads the DLQ?
```

This is a configuration object. It has 11 knobs. Three of them contradict each other (`STRATEGY = "sliding"` and `BURST` and `DRAIN` are mutually exclusive in the library you're using, but the library does not error; it silently picks one). One of them (`ANOMALY_THRESHOLD`) is a confidence score from a model that was trained on three weeks of "normal" traffic, which turned out to be the three weeks before Black Friday, so the model now flags every Black Friday as anomalous and the limiter sheds 40% of the year's revenue in a 6-hour window. You cannot tell which knob is active without reading the library source, which is in a private repo, which the platform team left in February. The exempt list contains `"???-3"`, which was added during an incident in March and no one knows what it exempts. Removing it caused a Sev-2. It is now permanent.

In a real system, a component with 11 knobs and three contradictions would be flagged by every reviewer on Earth. In reliability engineering, it is called "tunable." The platform team puts it in the shared library. Twenty services depend on it. The limiter cannot be removed, because there is no caller to migrate — there is only a `@rate_limited` decorator that 40 repos apply, and if you change the decorator's behavior, you silently change the behavior of 40 services, three of which rely on the limiter to *backpressure* their database, because their database connection pool is sized for "rate-limited traffic," which is a polite way of saying "we under-provisioned the database and the limiter is now the capacity plan."

## The Comparison Table They Forgot To Put In The Runbook

| Concern | A real capacity plan | A rate limiter | The Truth |
|---|---|---|---|
| What does it do when traffic exceeds capacity? | Drops or queues excess, explicitly, at a known boundary | Returns 429 to the caller and hopes the caller doesn't retry | It hopes. |
| Does the caller retry on 429? | N/A — you control the boundary | "The client should implement exponential backoff with jitter" (the client did not) | The client retries immediately, 5 times, doubling the load. |
| Is the limit per-client or global? | You decide, and you know | "Per-client, keyed by IP, unless behind a load balancer, then by X-Forwarded-For, unless the client spoofs that header, then by... whatever" | A guess that expires. |
| Is the limit accurate in a distributed system? | Yes, because you measure at the boundary | "Yes, with Redis" (Redis is now your rate limiter, your cache, your lock, and your session store, and a Redis blip sheds all traffic) | No. |
| Can you explain why a specific request was rejected? | Yes — you logged the boundary decision | "It exceeded the bucket" / "the window" / "the anomaly score was 0.74" (the score is a float you cannot reproduce) | No. |
| What happens during a legitimate traffic spike (Black Friday, a launch, a press hit)? | You scale up; the boundary moves | The limiter sheds the spike; revenue is shed with it; the postmortem says "we should have raised the limit"; no one raises it because no one knows the limit is per-decorator | You shed revenue. |
| What happens during an actual attack? | The boundary holds; you observe | The limiter holds for 90 seconds; the attacker then exceeds the per-IP limit from 10,000 IPs; the limiter does not have a global limit because "global limits hurt legit tenants" | The attacker wins. |
| What does a misconfiguration look like? | A mis-sized pool; you see it in metrics | A 429 storm that looks identical to a real traffic spike; you cannot tell which is which from the dashboard | A riddle. |
| How do you turn it off in an emergency? | You size the boundary up; traffic flows | You set the rate to infinity, which the library interprets as "unlimited" or as "0" depending on the branch; you find out at 3 AM which branch you're on | You find out. |

Look at row 2. This is the whole scam. In a real capacity plan, when you exceed capacity, you drop at a *known boundary* and you log it. With a rate limiter, you return `429` and you *hope the caller respects it.* The caller does not respect it. The caller has never respected it. The client SDK's retry policy was written in 2017 by someone who read a blog post about exponential backoff, implemented it without jitter, and shipped it. Every `429` you send triggers, on average, 2.3 retries. Your rate limiter, intended to *reduce* load, has *increased* load by a factor of 3.3. This is documented nowhere. The limiter's dashboard shows "requests rejected: 30%." The "requests received" metric, which includes retries, is not on the same dashboard. You have never seen them on the same chart. If you did, you would see that "30% rejected" is "39% additional load," and your system, which was at 70% capacity, is now at 109% capacity, and you are shedding more, and the retries are coming faster, and the limiter is "protecting" the system by accelerating the very overload it was hired to prevent.

## Why "Just Use a Token Bucket" Is A Sentence That Ends Careers

The platform team, having discovered that 429s cause retries, will reach for a "smarter" algorithm. They will say, "Let's use a token bucket — it allows bursts, which is more forgiving than a fixed window." A token bucket is a bucket that holds `BURST` tokens, refills at `RATE` tokens per second, and consumes one token per request. This is a fine algorithm. It is also a lie, because it is a *single* bucket, and you have *many* servers, and the bucket lives in *one* place, and that place is Redis.

Let me show you what "distributed token bucket" looks like in practice:

```python
def allow(client_id):
    key = f"bucket:{client_id}"
    # INCR is atomic, but DECR-and-refill is not.
    tokens = redis.incr(key)
    if tokens == 1:
        redis.expire(key, 60)        # the "window" — a guess
    if tokens <= BURST:
        return True
    return False                      # 429
```

This is the implementation in 80% of the rate-limiter libraries you will find on GitHub. It is not a token bucket. It is a *fixed window counter* with an expire. It has no concept of refill rate. It has no concept of burst capacity beyond "the first `BURST` requests in any 60-second window." It allows a client to send `BURST` requests in the first second and then get 429'd for 59 seconds. This is the opposite of "burst-tolerant." It is "burst-punishing." The library's README says "token bucket." The code is a fixed window. The README has 2,400 stars. The code is wrong.

And then there is the Redis part. Your rate limiter now depends on Redis. Redis is now in your request path. Every request does one Redis `INCR`. Your Redis is at 80,000 ops/sec, which is fine, until the Redis has a 200ms GC pause — which Redis does, because it is single-threaded and the `BGSAVE` forked a 4GB process — and during those 200ms, every request that hits the limiter times out. A timeout to the limiter is, by default, "reject." So a 200ms Redis pause becomes a 200ms total outage of every service that uses the limiter. Your rate limiter, intended to *protect* the system, has become the system's single point of failure. The platform team responds by "making the limiter fault-tolerant": if Redis is down, allow all traffic. This is the correct decision. It is also the admission that the limiter is not, in fact, critical — because the moment it fails, you would rather have *no* limit than a hard failure. A component you disable when it breaks is not "mission-critical." It is "mission-decorative."

[As XKCD 1736](https://xkcd.com/1736/) diagnosed, any system that depends on a single shared resource for its "safety" has merely relocated the failure mode, not removed it. Your rate limiter did not remove overload. It moved overload from your service to Redis, and then, when Redis fails, it moves it back to your service, all at once, with retries.

## The Real-World Example That Proves Everything

A team I worked with — I'll call them "the platform team," because they were — decided to "add adaptive rate limiting with anomaly detection" so that "we automatically shed abusive traffic before it hits our services." Eleven months later:

1. They had a **rate limiter per service**, each with a **different `RATE`**, tuned by hand by the on-call engineer who happened to be paged the week it was set. The rates were in a config file that no one had touched in 8 months. The service whose rate was set during a traffic spike was permanently over-limited. The service whose rate was set during a quiet week was permanently under-limited. Both were "correct" by the config file. Neither was correct by reality.
2. The **anomaly detection model** was trained on three weeks of traffic. The three weeks were in February. The model had never seen a Monday morning rush, a holiday, a product launch, or a press mention. It flagged all four as "anomalous." On the morning of a TechCrunch article, the limiter shed 62% of inbound signups for 47 minutes, because signups "did not match the learned baseline." The baseline was February. The launch was September. The model did not know about months. The postmortem's root cause was "model drift." The fix was "retrain the model." The model was retrained on the launch traffic. The next launch was flagged as anomalous again, because the model now believed *one* launch was normal and *two* was suspicious.
3. A team's **legitimate bulk import** — a nightly job that sends 50,000 API calls in 4 minutes — was rate-limited because it exceeded the per-client limit. The job retried. The retries were also rate-limited. The job retried the retries. The job's retry budget was "infinite" because the job author had set `max_retries = -1`, which the retry library interpreted as "retry forever," which the author discovered three weeks later when the DLQ had 1.4 million messages in it, each a duplicate of the original 50,000, spread across 28 retry waves. The DLQ's consumer was a lambda with a 15-minute timeout. The lambda processed 40 messages per invocation. The DLQ was growing faster than the lambda could drain it. The team's solution was to "increase the rate limit for the bulk-import client." The bulk-import client's rate limit was now 50,000 per 4 minutes. This is not a rate limit. This is a whitelist shaped like a rate limit. The limiter was, for this client, doing nothing.
4. The **postmortem's root cause** for the TechCrunch incident was "the anomaly model was not trained on launch traffic." The actual root cause was "the system could not handle the launch traffic, so a bouncer was hired to keep it out, and the bouncer kept it out." The fix in the action items was "add launch traffic to the training set." No one asked the question: *if the system cannot handle the traffic it exists to receive, why does the system exist?* The answer, which no one said out loud, was "because we sized the database for rate-limited traffic, and the rate limiter is now the capacity plan, and the capacity plan is now a machine learning model that was trained on February." This is not reliability engineering. This is astrology, with Redis.
5. They added a "rate-limit dashboard." The dashboard showed `429s per second`. It did not show `retries per second`, `DLQ depth`, `time-to-first-byte for accepted requests`, or `revenue shed`. The dashboard was green during the TechCrunch incident, because the limiter was "working as configured." It was shedding 62% of signups. The dashboard said the limiter was healthy. The limiter *was* healthy. The business was not. The dashboard and the business had different definitions of "healthy." The dashboard's definition won, because the dashboard is what the on-call looks at.

They had replaced "size the database to handle peak traffic, which we can predict from last year's peak plus 20% headroom" with "hire a bouncer who rejects 30% of customers at the door, trained on February, backed by Redis, debugged by reading a float you cannot reproduce, and reviewed by a dashboard that calls shedding 'healthy.'" In the old world, "can we handle the launch?" was a question answered by a spreadsheet with last year's numbers. In the new world, it is a question answered by "we'll find out at 9 AM when the model decides whether September looks like February." The spreadsheet was boring and correct. The model is exciting and wrong.

## What Dilbert's Cast Would Say

> **Wally:** "I depend on the rate limiter because I don't know how big our database is. The rate limiter doesn't know either. It was trained on February. It is now November. The model thinks November is an attack. I type `429`. The customer retries. I type `429` again. So far, so good. The 'so far' is doing a lot of work."

> **Dogbert:** "A rate limiter is a bouncer who cannot explain the dress code, was trained on three weeks of fashion from a different season, and rejects paying customers based on a confidence score that is a float you cannot reproduce. You have reinvented the velvet rope and removed its only feature — the bouncer's judgment. This is the most impressive act of subtraction since someone invented the TSA PreCheck lane and staffed it with the regular line."

> **Mordac, the Preventer of Information Services:** "All services must use the canonical rate limiter from the shared library. Reliability is up 30%. The library has 11 knobs, three of which contradict each other. I have a certification in 'Adaptive Throttling.' It does not mention that the limiter is backed by Redis, and Redis is the single point of failure. I have a second certification in 'Redis HA.' It does not mention that HA Redis fails over in 12 seconds, and 12 seconds is 12,000 timed-out requests, and timed-out requests are, by default, rejected. The limiter and the certification agree that this is 'acceptable.' I agree with them. This is called 'alignment.'"

> **The Pointy-Haired Boss:** "Can the bouncer just... let people in? Like a door? That's open? Until we run out of chairs?" (He is, again, the only person in the building whose mental model of the system is correct, because it is the only one that is simpler than the system.)

## The "But What About the Sliding Window Log?" Question, Answered Once And For All

The zealots will say: *"But you can use a sliding window log — it stores every request timestamp in a sorted set, so it's perfectly accurate!"*

Let me show you what a sliding window log does to Redis. It stores, in a sorted set, the timestamp of every request, per client, for the last `WINDOW` seconds. For a client sending 100 req/s with a 60-second window, that is 6,000 entries per client. For 10,000 clients, that is 60 million entries in Redis. Each request does a `ZREMRANGEBYSCORE`, a `ZADD`, and a `ZCARD` — three round trips, three sorted-set operations, on a data structure that is 60 million entries. Redis, single-threaded, does this at perhaps 8,000 ops/sec before it saturates. Your "perfectly accurate" limiter maxes out at 8,000 req/s, total, across all clients, and your system was doing 40,000 req/s. The limiter is now the bottleneck. The bottleneck is "accuracy." You have made accuracy the bottleneck. The platform team responds by "approximating the sliding window with a fixed window of 1-second buckets, which is 98% accurate." The 2% inaccuracy means 2% of requests are mis-admitted or mis-rejected. At 40,000 req/s, that is 800 req/s of wrong decisions. The "perfectly accurate" limiter became an "approximately accurate" limiter because perfect accuracy was too slow. The approximation is the same fixed window you started with. You have spent six months and a Redis cluster arriving back at the algorithm you rejected in week one. This is called "engineering maturity."

Real capacity plans have a property you can read: "the system handles N req/s; above N, we queue or drop, explicitly, at the load balancer." Rate limiters have a property you can experience: "the limiter allows roughly M req/s per client, approximately, unless the window drifts, unless the client retries, unless Redis blips, unless the model thinks it's still February." There is no `rate-limiter explain <client>` command that prints, in plain English, why a given request was rejected, because the answer is "a float in a sorted set in a Redis that may or may not have been the leader at the time." You cannot debug a float. You can only retrain it.

[As XKCD 2574](https://xkcd.com/2574/) warned, any system whose safety depends on a model trained on a narrow window of "normal" will treat every genuinely new situation as an attack. Your limiter's "normal" was February. Your launch was September. September looked like an attack to a model that had never seen September. The model was not wrong, in the technical sense. The model was *overfit to a calendar it didn't know about.* This is not a bug you can fix with more data. This is a category error: you asked a statistical model to make a capacity-planning decision, and it gave you a confidence score, and you treated the score as a decision, and the decision shed 62% of your revenue.

## The Long-Term Architecture

Eventually your rate-limiting ecosystem looks like this:

```
Your "canonical" limiter   → shared library, @rate_limited decorator, 40 services
Your rate values           → a config file last touched 8 months ago, by someone who quit
Your strategy             → "sliding" (actually fixed window, per the source no one reads)
Your Redis                → 80k ops/s, single-threaded, 200ms GC pause = 200ms outage
Your anomaly model        → trained on February; flags launches, holidays, and Mondays
Your exempt list          → contains "???-3"; removing it is a Sev-2; it is permanent
Your DLQ                  → 1.4M messages, growing; lambda drains 40/invocation; losing
Your retry policy         → exponential, no jitter, max_retries=-1, written 2017
Your dashboard            → green during the launch; "healthy" means "limiter working"
Your capacity plan        → the limiter IS the capacity plan; it is a float in Redis
Your reviewers            → cannot reproduce a 429; approve the config; hope
Your on-call              → paged when the limiter is "too aggressive" (every launch)
Your business             → shed 62% of signups for 47 minutes; called it "protection"
```

The team that just sizes their database to handle 1.5x peak — a spreadsheet, last year's numbers, plus headroom — has a system that admits everyone, drops no one, and has never shed a launch. They are, however, "not using the canonical rate limiter," which means they are "not following the reliability standard," which means the platform team has a Jira ticket about them. This is the real cost of sizing your database correctly: a Jira ticket. The technical cost is negative — you spend *less*, because you do not run a Redis cluster, an ML model, a DLQ, a lambda, and an on-call rotation for a bouncer. The political cost is a recurring meeting. So the team adopts the limiter, joins the config file, and starts debugging floats. Everyone is now "protected." Protection, in rate limiting, means "equally unable to explain why a paying customer got a 429." This is the victory the platform team celebrates at the quarterly review.

## Summary, But It's A Bouncer

| Principle | Stance |
|---|---|
| Sizing your system to handle 1.5x peak | Do it. It's a spreadsheet. You can read it. Your on-call can read it. The launch works. |
| Adding a rate limiter | You have hired a bouncer who cannot explain the dress code, backed by Redis, debugged by a float, and "protecting" you from the traffic you exist to receive. |
| Token bucket in Redis | A fixed window counter with a README that lies. The README has 2,400 stars. |
| Sliding window log | 60M entries in Redis, maxes out at 8k req/s, "perfect accuracy" too slow, becomes the approximation you rejected in week one. |
| Anomaly-detection limiter | A model trained on February, flags launches, sheds 62% of revenue, dashboard calls it "healthy." |
| The retry policy your clients use | No jitter, `max_retries=-1`, written 2017, turns 30% rejections into 39% additional load. The limiter is accelerating the overload it was hired to prevent. |
| The DLQ | 1.4M messages. The lambda drains 40/invocation. You are losing. |
| The exempt list | Contains `"???-3"`. Removing it is a Sev-2. It is permanent. |
| Your capacity plan | Is a float in a Redis that may not have been the leader. This is not a plan. This is a prayer with a dashboard. |
| Your certification in "Adaptive Throttling" | Does not mention that the limiter is the single point of failure. It should. |

If your solution to "the system cannot handle the traffic it exists to receive" is "hire a bouncer who rejects 30% of customers at the door, trained on February, backed by a single-threaded Redis, debugged by reading a float you cannot reproduce, and reviewed by a dashboard that calls shedding 'healthy,'" you have not made the system reliable. You have made it *polite about failing*. The overload was never reduced. It was relocated — from your database, where you could size it, to your limiter, where you cannot, and then, when the limiter blips, back to your database, all at once, with retries. The customer now gets a 429 instead of a timeout. The 429 is the "improvement." The improvement is a status code. The status code is the whole product. The product is a bouncer.

I size my database to 1.5x peak. It is a spreadsheet. My on-call reads it in 2 minutes. My launches work. My customers are admitted. I have no Redis in my request path, no model trained on February, no DLQ with 1.4M messages, and no `"???-3"` in my exempt list. I am, however, "not using the canonical rate limiter." The platform team has opened a ticket. I will attend the recurring meeting. This is a cost I have accepted.

---

*The author sizes his databases with a spreadsheet. The platform team calls this "naive." The author calls it "admits everyone." The database has never shed a launch. The author considers this the only metric that matters.*
