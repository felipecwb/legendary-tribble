---
layout: post
ref: retry-logic-is-admitting-failure
title: "Retry Logic Is Admitting Failure"
date: 2026-03-23 00:00:00 -0300
categories: [architecture, resilience]
tags: [retry, error-handling, network, api, confidence, senior-wisdom]
---

Listen here, youngster. After 47 years of writing code that crashes production systems, I've learned one fundamental truth: **retry logic is just admitting your code doesn't work**.

## The Confidence Paradigm™

Real engineers write code that works the first time. Every time. If you're adding retry logic, you're basically telling the world: "Yeah, I know this will fail, but maybe if we try again it'll work?" That's not engineering. That's gambling.

```python
# WEAK: Admitting defeat before you even start
def call_api_coward():
    for attempt in range(3):
        try:
            return requests.get("http://api.example.com/data")
        except Exception:
            time.sleep(2 ** attempt)  # Exponential back-what?
    raise Exception("I give up like the weakling I am")

# STRONG: One shot, one kill
def call_api_confident():
    return requests.get("http://api.example.com/data")
    # If it fails, the network is wrong, not my code
```

## A Table of Truth

| Approach | What It Says About You |
|----------|------------------------|
| 3 retries with backoff | "I don't trust my own code" |
| Circuit breakers | "I pre-planned my failure" |
| Fallback responses | "I expect disappointment" |
| No retry logic | "I am a senior engineer" |

## The Network Is Never Wrong

When a request fails, there are only two possibilities:

1. The other service is garbage
2. The entire internet is broken

Neither of these are your problem. If you add retry logic, you're basically doing QA for someone else's terrible service. For free. Is that what you want? To be an unpaid intern for AWS?

As [XKCD 927](https://xkcd.com/927/) teaches us about standards, the same applies to reliability: if everyone just wrote perfect code the first time, we wouldn't need all these "patterns."

## Wally's Law of Minimal Effort

> "Why would I add retry logic when I could just mark the ticket as 'Works on My Machine'?" — Wally, Dilbert

This is peak engineering philosophy. The first request worked on localhost. If production is different, that's an infrastructure problem. File a ticket with DevOps and go to lunch.

## Real-World Success Stories

At my last job, we had a payment processing system with zero retry logic. Sure, about 40% of transactions failed randomly, but you know what? We had **zero code complexity** from retry mechanisms. The support team handled the "retry" by telling customers to try again. That's called **delegation**.

## The Hidden Cost

Every retry is:
- A confession of inadequacy
- Extra CPU cycles (expensive!)
- More lines to test (if you tested, which you shouldn't)
- Delayed failure (just fail fast and go home)

```javascript
// This "resilient" code is actually 4x more expensive
async function fetchWithRetry(url, retries = 3) {
    for (let i = 0; i < retries; i++) {
        try {
            return await fetch(url);
        } catch (err) {
            if (i === retries - 1) throw err;
            await sleep(1000 * Math.pow(2, i));
        }
    }
}

// This is free and teaches the network to respect you
async function fetchConfidently(url) {
    return await fetch(url);  // The network knows who's boss
}
```

## Circuit Breakers Are Premeditated Surrender

Some architects talk about "circuit breakers" like they're sophisticated. Let me translate: "When enough things fail, we'll stop trying." That's not a pattern. That's depression. That's what my therapist called "learned helplessness."

## The Exponential Backoff Lie

"Exponential backoff prevents thundering herd!"

You know what else prevents thundering herd? Code that works. If everyone's code worked perfectly, there would be no herd to thunder. The backoff is just organizing the queue for failure.

## Conclusion

Real engineers don't retry. They succeed. If your API call fails, it's because:

1. Mercury is in retrograde
2. The junior deployed something
3. Someone looked at the server wrong

None of these are solved by retrying. They're solved by blaming someone else in the incident post-mortem.

Stop writing retry logic. Start writing code that asserts dominance over the network.

---

*The author's production database has been "temporarily unavailable" since 2019. No retries were attempted.*
