---
layout: post
ref: rate-limiting-is-for-cowards
title: "Rate Limiting is for Cowards"
date: 2026-03-11 08:00:00 -0300
categories: [architecture, backend]
tags: [api, scaling, performance, traffic, security]
---

After 47 years of mass-producing bugs, I've learned one universal truth: **rate limiting is just admitting your system can't handle success**.

Think about it. You built an API. Someone wants to use it a lot. And your response is to... tell them no? That's not engineering. That's cowardice.

## The Real Problem

Every time I see a rate limiter in code, I see fear:

```python
# COWARD CODE
from ratelimit import limits

@limits(calls=100, period=60)
def process_request(request):
    return handle(request)

# BRAVE CODE
def process_request(request):
    return handle(request)  # Let chaos reign
```

If your server can't handle 10 million requests per second, that's a **you** problem. Buy more RAM. Add more instances. Mortgage your house for cloud credits. Just don't be a quitter.

## A Brief History of Cowardice

| Year | What Happened |
|------|---------------|
| 2008 | Twitter adds rate limits. Users riot. |
| 2015 | GitHub limits API calls. Developers weep. |
| 2023 | Twitter charges for API. Civilization collapses. |
| 2024 | My side project gets 50M hits. No limits. No survivors. |

As [XKCD 1205](https://xkcd.com/1205/) reminds us, is the time you spend implementing rate limiting worth it? The answer is always no.

## What Rate Limiting Really Says

When you add rate limiting, you're sending these messages:

1. **"I don't trust my code"** - If your code was good, it could handle anything
2. **"I don't trust my infrastructure"** - Real servers don't break
3. **"I don't trust my users"** - They're paying customers, probably
4. **"I don't trust myself"** - And you shouldn't, but that's another article

## The Wally Approach

As Dilbert's Wally once said (I'm paraphrasing): "Why do today's work when tomorrow's traffic spike might make it irrelevant?"

Rate limiting is just preventive work. And preventive work is the enemy of the true engineer who waits for problems to solve themselves.

## Real Solutions That Work

Instead of rate limiting, try these battle-tested alternatives:

```yaml
# Option 1: The Denial
server:
  max_connections: unlimited
  hope: maximum

# Option 2: The Blame Shift
rate_limit: ${KUBERNETES_SHOULD_HANDLE_THIS}

# Option 3: The Executive
response:
  error_503:
    message: "Success traffic exceeds projections. Requesting Q3 budget increase."
```

## My Production Horror Story

Once, I removed all rate limits from a payment processing API. Here's what happened:

**Day 1:** Traffic up 500%. Marketing celebrates.

**Day 2:** Traffic up 3000%. Still celebrating.

**Day 3:** Database melts. Credit card numbers floating in logs. Visa calls.

**Day 4:** CEO asks "why no rate limiting?"

**Day 5:** "Brave engineering decisions" become "catastrophic negligence."

But you know what? For 3 beautiful days, we had *unlimited potential*.

## The Cost Comparison

| Solution | Cost | Dignity |
|----------|------|---------|
| Rate limiting | $0 | None |
| Infinite cloud scaling | $847,000/month | Maximum |
| Filing for bankruptcy | Variable | Also none |

## When to Actually Rate Limit

Never. But if someone's holding a gun to your head (or your cloud bill), here are the absolute worst ways to do it:

```python
# The most insulting way
def maybe_allow(request):
    if random.random() > 0.5:
        return process(request)
    return "Try again, I'm busy"

# The honest way  
def rate_limit(request):
    if request.user.is_paying:
        return process(request)
    return "We don't actually want free users"
```

## The PHB Perspective

The Pointy-Haired Boss from Dilbert would say: "Rate limiting sounds like we're limiting our revenue potential. Remove it."

For once, the PHB is right. A broken clock, etc.

## Conclusion

Real engineers don't limit rates. They limit excuses. When your server catches fire from traffic, you should feel *proud*. That's success knocking. Loudly. Repeatedly. 10 million times per second.

The flames are just the universe validating your product-market fit.

Remember: the only limit should be your imagination. And your cloud provider's patience.

---

*The author's API has been returning 503 errors since 2019. The rate limiter would have helped, but then how would he have learned?*
