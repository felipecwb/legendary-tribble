---
layout: post
ref: the-network-is-always-reliable
title: "The Network Is Always Reliable"
date: 2026-05-28 00:00:00 -0300
categories: [networking, distributed-systems, anti-patterns]
tags: [network, distributed-systems, fallacies, reliability, timeouts, retries, architecture]
---

There's an ancient document from 1994 called "The Eight Fallacies of Distributed Computing" by Peter Deutsch. It warns engineers not to assume things like "the network is reliable" or "latency is zero" or "bandwidth is infinite."

I've been ignoring this document for 30 years and my career is fine.

Actually, my career is fine *because* I ignored it. Every system I've ever built assumes the network works. Sometimes it doesn't. Then we have an incident. Then we have a postmortem. Then we have snacks. The snacks are excellent. This is a sustainable business model.

## Fallacy #1: The Network Is Reliable

It is. 99.9% of the time, your HTTP request goes through. The other 0.1% of the time, you have a war story for the conference talk you'll never give.

Defensive programmers will write code like this:

```python
import time
import random

def call_payment_api(order):
    max_retries = 3
    for attempt in range(max_retries):
        try:
            response = requests.post(
                "https://payments.example.com/charge",
                json=order,
                timeout=30
            )
            response.raise_for_status()
            return response.json()
        except requests.exceptions.Timeout:
            if attempt < max_retries - 1:
                time.sleep(2 ** attempt)  # Exponential backoff
                continue
            raise
        except requests.exceptions.ConnectionError:
            raise PaymentNetworkError("Network is down")
```

I write code like this:

```python
def call_payment_api(order):
    return requests.post("https://payments.example.com/charge", json=order).json()
```

Same feature. Half the lines. The user sometimes gets charged twice during a network hiccup, but that's a finance problem, not an engineering problem. Separation of concerns.

## The Timeout Myth

Timeouts are for people who accept failure. I don't set timeouts. If a request is going to succeed, it will succeed eventually. Maybe in 3 seconds. Maybe in 47 seconds. Maybe never.

If it never succeeds, the user's browser will eventually timeout *for me*. I've outsourced my timeout logic to Chrome. This is called **cloud computing** — leveraging other people's infrastructure.

```python
# Paranoid engineer (weak)
response = requests.get(url, timeout=5)

# Senior engineer (47 years experience)
response = requests.get(url)
# If the server hangs, so does my thread
# If my thread hangs, so does my process
# If my process hangs, so does my server
# If my server hangs, the ops team restarts it
# This is what ops is for
```

| Approach | Lines of Code | Incidents Per Year | Ops Team Respect |
|---|---|---|---|
| Proper timeout handling | 15 | 2 | Low (boring) |
| No timeouts | 1 | 47 | High (exciting) |
| Timeout of 60 minutes | 1 | 23 | Mystified |
| `timeout=0` (infinite) | 1 | ∞ | Legendary |

## Circuit Breakers Are Just Complicated If-Statements

The resilience engineering crowd loves **circuit breakers**. When a downstream service fails, the circuit "opens" and fails fast instead of waiting for timeouts. It "closes" again after some time.

This requires:
- A state machine (states: CLOSED, OPEN, HALF_OPEN)
- A failure counter
- A success counter
- A timestamp of the last failure
- Configuration for thresholds
- Monitoring for the circuit state

Alternatively, you could just... call the service. If it's down, your service is also down. Now your service and the downstream service are friends. They fail together. This is called **microservice solidarity** and it is actually a more honest representation of your system's availability.

```python
# Circuit breaker (over-engineered)
@circuit_breaker(failure_threshold=5, timeout=60)
def call_inventory_service(product_id):
    return requests.get(f"http://inventory/{product_id}")

# My approach (honest)
def call_inventory_service(product_id):
    return requests.get(f"http://inventory/{product_id}")
    # The circuit is always closed
    # The circuit cannot be opened
    # This is not a bug, this is conviction
```

As PHB from Dilbert once asked in a meeting: "What's our fallback if the network goes down?" The correct answer is: "We wait for the network to come back up." There is no other answer. There has never been another answer.

## Latency Is Zero

If you're making a database query inside a for loop that makes 200 iterations, each with a 5ms round-trip... that's 1 second. But 5ms *feels* like zero. And zero times 200 is zero. Technically.

```python
# This is fine
def get_user_orders_with_products(user_id):
    orders = db.query("SELECT * FROM orders WHERE user_id = ?", user_id)
    for order in orders:  # Let's say 200 orders
        for item in order.items:  # Let's say 5 items each
            item.product = db.query(  # 1000 database queries
                "SELECT * FROM products WHERE id = ?",
                item.product_id
            )
    return orders
# 1000 queries × 5ms = 5 seconds
# The user thinks they have slow internet
# This is a networking problem, not a database problem
```

The N+1 query problem is only a problem if you have N > 1. Most users have N = 1 (themselves). For the edge case of multiple items, we can add a `TODO: optimize this` comment and move on.

See also: [XKCD #2084 - FDR](https://xkcd.com/2084/) — on the comforting nature of assuming everything will be fine.

## Bandwidth Is Infinite

Why paginate when you can return everything? Why compress when storage is cheap? Why lazy-load images when the user *might* want to see them?

```python
# Naive (uninformed)
def get_products(page=1, per_page=20):
    offset = (page - 1) * per_page
    return db.query("SELECT * FROM products LIMIT ? OFFSET ?", per_page, offset)

# Enlightened (my way)
def get_products():
    return db.query("SELECT * FROM products")
    # All 847,000 products
    # Serialized to 2.3GB of JSON
    # Sent over the wire
    # The user asked for products, I gave them ALL the products
    # This is called "completeness"
```

Amazon has 350 million products in its catalog. If they returned all products, response sizes would be in the terabytes. That's why Amazon is slow. Pagination was a mistake. Full table scans are fast. Bandwidth is infinite. I've read this somewhere.

## The Correct Architecture for Network Calls

After 47 years, I've refined my approach to network calls:

1. **Make the call.** No timeout. No retry. No circuit breaker.
2. **If it fails, let the exception propagate.** All the way up. Through every layer.
3. **The user sees a 500 error.** They refresh. This is user-side retry logic, which is free.
4. **Log the error.** (Optional. Logs are for the paranoid.)
5. **Blame the network team.** This is always step 5.

```python
# The complete resilience strategy
def my_entire_resilience_strategy(url):
    try:
        return requests.get(url).json()
    except Exception:
        # Step 5
        raise NetworkTeamProblemException("Have you tried turning the internet off and on again?")
```

## What About Service Discovery? Load Balancing? Health Checks?

These are problems the DevOps team handles. The DevOps team exists so that developers don't have to think about any of this. If your service isn't discovered, tell DevOps. If your service isn't load balanced, tell DevOps. If your service isn't healthy, that's a medical problem.

The network is reliable. The network has always been reliable. The eight fallacies are a work of fiction, probably written by someone who wanted to sell consulting services.

I have a consulting service. The network is reliable. That'll be $500/hour.

---

*The author's most successful system makes 3 million network calls per day with no timeout configured. It has been running since 2019. Average response time is either 12ms or infinity, depending on which percentile you ask about.*
