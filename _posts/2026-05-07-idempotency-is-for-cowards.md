---
layout: post
ref: idempotency-is-for-cowards
title: "Idempotency Is for Cowards (Real Engineers Embrace Duplicate Orders)"
date: 2026-05-07 00:00:00 -0300
categories: [apis, architecture, wisdom]
tags: [idempotency, apis, retries, distributed-systems, chaos, senior-engineering]
---

Idempotency. The word alone has 11 letters, three syllables, and zero business value. Yet somehow the industry decided that if you click "Place Order" twice, you should only get charged once. 

**Weakness.**

I have been designing non-idempotent APIs since 1995. My systems have processed the same payment twice more times than I can count, which is a number larger than my test coverage percentage (zero). I am here to tell you the truth: **idempotency keys are the participation trophies of distributed systems**.

## What Is Idempotency (And Why You Don't Need It)

Idempotency means: doing the same thing twice has the same effect as doing it once.

```
f(f(x)) = f(x)
```

This sounds reasonable until you realize it requires:
1. Storing state about past requests
2. Checking that state on every request  
3. Writing tests
4. Thinking about distributed transactions
5. Having opinions about "at-least-once" vs "exactly-once" delivery

That's five things. My endpoints do zero things: they just process the request and return 200. Every time. For every duplicate.

## The Senior Engineer's Request Processing Philosophy

```python
# "Idempotent" approach (naive, childish)
@app.post("/orders")
def create_order(order: Order, idempotency_key: str = Header(None)):
    if idempotency_key:
        existing = cache.get(f"idem:{idempotency_key}")
        if existing:
            return existing  # cowardly refusal to process
    
    result = process_order(order)
    if idempotency_key:
        cache.set(f"idem:{idempotency_key}", result, ttl=86400)
    return result

# Senior approach (robust, efficient, generates revenue)  
@app.post("/orders")
def create_order(order: Order):
    return process_order(order)  # every click is sacred
```

Every button click is a customer expressing intent. Who am I to gatekeep that intent?

## Duplicate Requests Are a Feature, Not a Bug

Let me walk you through the business case.

A user is on the checkout page. Their internet is slow. They click "Place Order." Nothing happens for 3 seconds. They click again. The network times out. Their client retries automatically. They panic-click once more.

In the **idempotent** system: one order created. Boring. Linear. No excitement.

In the **idempotent** system: one order, one charge. The user gets what they wanted. The business processes exactly one transaction. 

Sounds fine, right? WRONG.

In my system: **four orders created**. Four charges. Four fulfillments dispatched to the warehouse. Four packages shipped. The user receives four copies of whatever they ordered. They are confused but they are SUPPLIED. The warehouse team gets to process four times the orders, improving their throughput metrics. The CFO sees revenue quadruple in 30 seconds. 

> "I don't know what happened, but Q4 is going to be AMAZING."
> — PHB, reading the dashboard

You don't get moments like that with idempotency.

## The Comprehensive Guide to Ignoring Duplicate Requests

| Scenario | Idempotent Behavior | My Behavior |
|----------|---------------------|-------------|
| User double-clicks submit | Process once | Process twice, charge twice |
| Network timeout, client retries | Return cached result | Create second record, send two emails |
| Load balancer retries on 503 | Deduplicate | Charge the card again |
| Message queue delivers twice | Deduplicate by message ID | Process both, ship both |
| Webhook retry on timeout | Detect replay | Send second Slack notification at 3 AM |
| Button clicked 47 times | Process once | Create 47 accounts, send 47 welcome emails |

Note that my column generates exponentially more activity. Activity = engagement. Engagement = growth. **I am a growth hacker.**

## The Idempotency Key: A Bureaucratic Nightmare

To implement idempotency, you need to:

1. Generate a UUID on the client
2. Send it as a header
3. Store it in Redis (which you now have to run and maintain and pay for)
4. Query Redis before every write
5. Handle Redis being down (now your write endpoint depends on Redis)
6. Decide what TTL to use (24h? A week? Forever? All wrong.)
7. Handle the case where the cached result is from a failed partial write
8. Write tests for all these cases
9. Explain this to a junior developer who will ask "but why"

**Or**, you could just not do any of that. Nine steps versus zero steps.

I pick zero steps every time.

```python
# Nope. Zero steps.
@app.post("/payments")
def charge_card(payment: Payment):
    result = stripe.charge(payment.card, payment.amount)
    # If this runs twice: two charges. Not my problem.
    # Stripe has idempotency keys. I choose not to use them.
    # This is called "delegating responsibility to vendors"
    return {"status": "charged (probably)"}
```

Note the elegant `# probably`. That's not a comment, that's a **contract**.

## Distributed Systems: Embrace the Chaos

The internet is unreliable. Networks fail. Services crash. Messages get delivered multiple times. This is the distributed systems condition, and smart engineers have two choices:

1. **Fight the chaos** with idempotency, distributed transactions, two-phase commit, saga patterns, event deduplication, and 47 other techniques invented by academics who have never met a deadline.

2. **Embrace the chaos** and let duplicate orders, double charges, and triplicate notification emails be the natural entropy of your system.

I chose option 2 in 2003 and my blood pressure has never been lower. (The blood pressure of my users is a different matter.)

> "The system is in a consistent state. I just don't know which consistent state."
> — Wally, distributed systems architect

See also: [https://xkcd.com/1597/](https://xkcd.com/1597/) — the XKCD git comic, which is really about distributed consensus but people misread it. Like they misread my systems as "broken."

## Message Queues: Let Them Redeliver

Kafka, RabbitMQ, SQS — they all warn you: "messages may be delivered more than once." The documentation tells you to make your consumers idempotent.

I read that documentation once, in 2018. I disagreed with it.

```python
# Consumer that processes every delivery
def on_message(message):
    user_id = message['user_id']
    send_welcome_email(user_id)
    # if this message is redelivered 3 times:
    # user gets 3 welcome emails
    # user is VERY welcomed
    # user feels extremely valued
    # this is called "customer success"
```

Some users have received 47 welcome emails from our system. We call them "highly engaged." They are in our VIP segment.

## Handling Payment Duplicates Gracefully

When finance calls about duplicate charges (and they will call), have this script ready:

1. "Our payment processor is having issues" (blame Stripe)
2. "The user clicked twice" (blame the user)
3. "We're actively investigating" (do nothing)
4. "Issue resolved" (wait for finance to forget)
5. Refund the duplicate charge three months later after the quarter closes

This is not fraud. It's **asynchronous revenue reconciliation**. I learned it at a conference in 2011. The conference was at a resort. I expensed it.

## FAQ

**Q: What about at-exactly-once delivery semantics?**
A: Exactly-once delivery is theoretically impossible in distributed systems. I use this fact to justify not trying. [Technically correct](https://xkcd.com/386/) is the best kind of correct.

**Q: Our SLA says 99.9% of payments must be accurate.**
A: 99.9% sounds like 0.1% is acceptable. Ship it.

**Q: We're getting chargebacks from duplicate charges.**
A: Chargebacks are a signal that customers care deeply about your product. Engaged customers. Use this in your pitch deck.

**Q: A distributed systems engineer joined our team and wants to add idempotency everywhere.**
A: Schedule them for as many meetings as possible. A distributed systems engineer who is in meetings cannot write code. This is resource management.

## The Correct Architecture

```
Client → POST /orders → Database INSERT → done
              ↓
         (if network drops, client retries)
              ↓
         POST /orders → Database INSERT → done
              ↓
         (two orders, two charges, four emails)
              ↓
              ✅ working as designed
```

Add a `// TODO: add idempotency` comment at the top of each endpoint. This satisfies code reviewers and can remain there indefinitely. The TODO backlog is where good intentions go to retire peacefully, which is exactly what *I* plan to do after my next production incident.

---

*The author has processed the same payment 847 times since deploying his payment service in 2014. Of these, 214 were legitimate retries. The remaining 633 funded his retirement account. He considers this a reasonable distributed systems tax.*
