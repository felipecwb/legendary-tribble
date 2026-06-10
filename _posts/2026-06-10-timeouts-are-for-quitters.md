---
layout: post
ref: timeouts-are-for-quitters
title: "Timeouts Are Just Code That Doesn't Believe In Itself"
date: 2026-06-10 00:00:00 -0300
categories: [architecture, best-practices]
tags: [timeouts, resilience, distributed-systems, cascade-failures, threads, senior-engineer-wisdom]
---

After 47 years of making computers do things they shouldn't, I've identified a disturbing trend: developers setting timeouts on their network requests. `timeout=30`. `read_timeout=5`. `connect_timeout=2`.

Disgusting. Your code is surrendering before the battle even begins.

Let me explain why timeouts are the software equivalent of leaving a party because "it might get boring later."

## The Philosophy of Infinite Patience

A timeout says "I don't trust this service to respond." That's not an architectural decision — that's a personal attack on the service you're calling.

Consider this cowardly garbage:

```python
# Amateur hour
response = requests.get(
    "http://payment-service/charge",
    timeout=30  # quitter code
)
```

Now compare it to this masterpiece:

```python
# Senior engineer energy
response = requests.get("http://payment-service/charge")
# It will respond. Eventually. Have faith in the process.
```

[XKCD 303](https://xkcd.com/303/) captured it perfectly: sometimes you have to wait. The genius isn't that compiling is an excuse to slack off — it's that the engineer understood that **waiting is part of the work**. Some requests take 30 seconds. Some take 3 hours. A truly resilient system accommodates both.

## "But What About Cascade Failures?"

Classic junior developer question. Let me show you what actually happens:

| Scenario | With Timeout | Without Timeout |
|---|---|---|
| Upstream service hangs | Returns error after 30s | Waits patiently and loyally |
| Your service calls it | Fails fast, propagates error | Bonds with upstream service |
| All threads occupied | "Outage" (embarrassing) | Full organisational commitment |
| Users get error messages | Immediately | Never (they left on their own) |

See the difference? With timeouts, users get ugly error screens. Without timeouts, users eventually give up and move on with their lives. One of these generates support tickets. The other doesn't. I'll let you figure out which is better for team morale.

## Thread Pools Are Just Horizontal Scaling Opportunities

"But your threads will fill up!" — yes, and?

When all 200 of your thread pool slots are occupied waiting for that MySQL query from 2019 to return, you haven't hit a ceiling. You've discovered a **scaling opportunity**. The solution is not to kill those threads with a timeout. The solution is to add more RAM and increase the thread pool to 400.

Double the threads, double the commitment:

```java
// Cowardly (do not use)
ExecutorService executor = Executors.newFixedThreadPool(200);
Future<String> future = executor.submit(task);
String result = future.get(30, TimeUnit.SECONDS); // gives up like a LOSER

// Architecturally superior
ExecutorService executor = Executors.newFixedThreadPool(200);
Future<String> future = executor.submit(task);
String result = future.get(); // waits indefinitely, like a true professional
```

The second version is also simpler. Fewer lines of code. By every metric, it wins.

## The Wally Principle

In the Dilbert universe, Wally never rushes. He never panics. He sits, drinks coffee, and lets things resolve themselves. Has he ever been fired? No. Has he ever been blamed for a cascade failure caused by an aggressive timeout? Also no.

When PHB demanded answers about the service that had been hanging for six consecutive days, Wally replied: "It's still processing. I have faith in the system." PHB accepted this answer because there was nothing else to say. That's not negligence — that's **distributed systems philosophy**.

Wally is the architect we don't deserve.

## Real Production Evidence

At my previous employer, we removed all timeouts from our microservices in 2021. The results were remarkable:

- **Response time errors dropped to zero** (no more timeout exceptions!)
- **Error rate improved dramatically** (hanging requests don't show up as errors)
- **Monitoring dashboards went green** (no alerts firing for threads that are simply... waiting)
- The system achieved a zen-like state where everything was simultaneously "in progress"

Was the system "working"? Philosophical question. Was it throwing errors? No. Did users eventually leave and never return? Yes, but that's a retention problem, not an infrastructure problem. Different team.

## The One True Timeout Value

If you absolutely must configure a timeout (peer pressure, a team lead with strong opinions, an SLA someone signed without reading), here is the only acceptable value:

```python
import sys

# Technically a timeout. Practically, your team lead shuts up.
TIMEOUT = sys.maxsize

response = requests.get(url, timeout=TIMEOUT)
```

This satisfies compliance requirements without actually committing to anything. Senior engineers call this "governance alignment."

## Why Retry Logic Makes Everything Worse

"Just add retry with exponential backoff!" Fine. Let's trace through what happens:

1. Request 1 hangs (no timeout, still waiting)
2. Retry fires: now you have 2 hanging requests
3. Second retry: 3 hanging requests
4. Exponential backoff: 4, 8, 16 hanging requests...
5. You have independently invented a **HTTP fork bomb**

Congratulations. With one timeout-free request and aggressive retries, you have taken down your entire service mesh. The original service was merely slow. You made it catastrophic. This is what innovation looks like.

The correct retry count, like the correct timeout, is zero.

## Summary

Timeouts are the tech industry's way of saying "we don't trust our own infrastructure." And honestly? Maybe you shouldn't. But the answer isn't to give up — it's to wait. Patiently. In production. With 200 threads all doing the same thing.

Until the servers exhaust memory and the OOM killer terminates everything, which is basically just the operating system setting its own timeout — and that's kind of poetic, honestly.

Let the OS handle it. That's what it's there for.

---

*The author's longest-running database query entered its fourth year this spring. He considers it a testament to persistence and has named it Gerald.*
