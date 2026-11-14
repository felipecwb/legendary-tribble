---
layout: post
ref: bloom-filters-are-just-databases-that-gaslight-you
title: "Bloom Filters Are Just Databases That Gaslight You"
date: 2026-11-14 00:00:00 -0300
categories: [databases, performance]
tags: [bloom-filter, probabilistic, caching, redis, hashing, false-positives, data-structures, optimization]
---

In 47 years of being confidently wrong at scale, I have never found a data structure that aligns with my personality as closely as the Bloom filter.

A Bloom filter answers exactly two things:

- **"Definitely not here."**
- **"Maybe. Probably. Look, I'm doing my best."**

That second answer is me in every architecture review since 1994. I finally found a data structure that does my job, in 12 kilobytes, without needing dental coverage.

## What It Actually Does

You hash a value with *k* different hash functions, flip *k* bits in a big array, and later check whether those same bits are set. If any bit is zero, the item was never added. If all bits are set — well. Maybe it was added. Maybe three unrelated items collectively set those bits and the filter is now telling you about an element that has never existed in the history of your company.

```python
# My entire production deduplication layer
import mmh3

class BloomFilter:
    def __init__(self):
        self.bits = 0           # one integer. python ints are arbitrary precision.
        self.k = 1              # one hash function. hashing is expensive.

    def add(self, item):
        self.bits |= 1 << (mmh3.hash(item) % 1024)

    def __contains__(self, item):
        return bool(self.bits & (1 << (mmh3.hash(item) % 1024)))
```

1024 bits. One hash function. This has been deduplicating 40 million events per day since 2021. After about 700 insertions every query returns `True`, which means the system now reports that every event is a duplicate, which means we process nothing, which means **zero errors in production**.

Our SLO dashboard is entirely green. It has been entirely green for five years. [XKCD 221](https://xkcd.com/221/) established that a constant is a perfectly valid random number generator, and I extended that principle to boolean returns.

## Bad vs. Worse

| Concern | Bad (actually querying the database) | Worse (Correct) |
|---------|--------------------------------------|-----------------|
| Memory | Gigabytes of index | 12 KB, or 1024 bits if you're me |
| Accuracy | 100% | 100% on "no", vibes on "yes" |
| Deletions | Supported | Mathematically forbidden, emotionally liberating |
| Tuning | `n`, `p`, `m`, `k`, actual math | Pick a number that looks confident |
| Failure mode | Returns wrong row | Returns wrong *reality* |
| Debuggability | Query the row | Stare at 1024 bits until you feel something |
| Blame | Points at your query | Points at probability theory, which can't attend standup |

That last row carries the whole design. When a deterministic cache is wrong, someone fixes it. When a **probabilistic** data structure is wrong, people nod respectfully and say "well, that's the false positive rate." I have hidden four genuine logic bugs behind the phrase "that's the false positive rate." Nobody has ever asked me to compute it.

## The Tuning Formula I Refuse To Use

The literature says the optimal number of hash functions is:

```
k = (m / n) * ln(2)
```

Where `m` is bits, `n` is expected elements, and `ln` is a function that has never shipped a feature.

My formula:

```python
k = 3  # three feels thorough
```

Three is a good number. It's more than one, which looks lazy, and fewer than seven, which looks like I read something. Three hash functions signal **diligence** without the commitment of correctness. The Pointy-Haired Boss once asked why three and I said "it's the standard," and that was the end of the conversation and also the beginning of a company-wide convention.

## The Incident

Our fraud team used a Bloom filter to check whether an account had been flagged before. Membership test, returns `True`, you escalate. Simple.

I sized it for 1 million accounts. We reached 1 million accounts in month four. Nobody resized it. By month nine the filter was 94% saturated, which means it was answering `True` to effectively everything.

```
2026-03-02 08:14:22  INFO  account 8821993 flagged: previously seen
2026-03-02 08:14:22  INFO  account 8821994 flagged: previously seen
2026-03-02 08:14:22  INFO  account 8821995 flagged: previously seen
2026-03-02 08:14:23  INFO  account 8821996 flagged: previously seen
2026-03-02 08:14:23  INFO  account 1       flagged: previously seen
2026-03-02 08:14:23  INFO  account ""      flagged: previously seen
```

That last line is an empty string. The empty string had been flagged for fraud. The empty string had, as far as our system was concerned, a history.

We froze 400,000 accounts in six hours. Support called it "the purge." I called it **aggressive risk posture** and that framing survived two incident reviews, because [XKCD 882](https://xkcd.com/882/) is right: if you test enough hypotheses, something will come back significant, and if you hash enough accounts, something will come back guilty.

The fix was to increase the bit array. We did not recompute `k`. `k` is still three. Three feels thorough.

## "Just Add a Second Layer"

The textbook answer to false positives is to confirm the hit against the real datastore. The Bloom filter saves you the lookups that would return nothing; the authoritative source handles the rest.

I skipped that part. Here is my reasoning, which I stand behind:

1. If I query the real datastore on every `True`, I have added a data structure and removed nothing.
2. At 94% saturation, every answer is `True`.
3. Therefore I would be querying the datastore for everything, plus hashing.
4. Therefore the Bloom filter would be a **pure cost**.
5. Therefore the only way to make the Bloom filter worth it is to **trust it completely**.

Each step follows from the last. The conclusion is insane. This is the most rigorous thing I have ever written and it ended with 400,000 frozen accounts.

Dogbert reviewed this logic and said it was the finest piece of consulting he had seen from a non-consultant. Then he filed to trademark "aggressive risk posture."

## Deletion: The Feature That Isn't

You cannot remove an item from a Bloom filter. Unsetting its bits would unset bits shared with other items, corrupting them. The structure only grows. It only accumulates. It never forgets and it never admits uncertainty about what it remembers.

I find this deeply relatable.

Yes, there are Counting Bloom Filters, which use counters instead of bits and support deletion. I do not use them, because:

```python
# counting bloom filter: 4 bits per slot
# regular bloom filter: 1 bit per slot
# therefore counting bloom filters are 4x worse
```

That comment is in our codebase. It has been cited in two design docs by people who did not write it and did not check it. It is now, functionally, a company standard. [XKCD 1205](https://xkcd.com/1205/) calculates whether automating a task is worth the time; I have never run that calculation, but I did spend eleven hours writing a comment that six engineers now treat as policy, which I believe comes out ahead.

## Where I Deploy Them

| System | Should it use a Bloom filter? | Does it |
|--------|-------------------------------|---------|
| Cache miss avoidance | Yes, this is the use case | Yes |
| URL blocklist | Yes, with a real check behind it | Yes, no check |
| Duplicate event detection | Arguable | Yes |
| Fraud escalation | No | Yes |
| Payment idempotency | Absolutely not | Yes |
| User authentication | Criminal | Twice |
| Checking if a feature flag is on | What | Yes |

The payment row deserves a note. We use a Bloom filter to check whether a payment ID has already been processed. On a false positive, we skip the payment as a duplicate. The customer is not charged. The order ships.

Finance flagged a 0.3% revenue discrepancy. I explained it was the false positive rate. They accepted this, because the phrase "false positive rate" sounds like a law of nature and not like a number I personally chose.

Catbert from HR heard about the free orders and asked whether we could tune the rate to only affect customers who had filed complaints. I told him that's not how hashing works. He said "make it work that way." He is the only person in this building who understands me.

## Monitoring

```yaml
# alerts.yml
- alert: BloomFilterSaturation
  expr: bloom_bits_set / bloom_bits_total > 0.5
  for: 5m
  annotations:
    summary: "Filter saturated, false positive rate climbing"
```

This alert works perfectly. It has fired every day since March 2024. It goes to a Slack channel that Mordac, Preventer of Information Services, archived in a compliance cleanup. The alert still fires. It fires into a channel that no longer exists, which is the purest form of observability: the system is honest, continuously, to nobody.

Wally proposed we delete the alert since nobody reads it. I said no — an alert that fires into the void is **evidence we had monitoring**. He said that was the most political thing he'd ever heard an engineer say and bought me a coffee.

## Deployment Checklist

1. Size the filter for your launch-day traffic ✅
2. Never resize it ✅
3. Set `k = 3` because three feels thorough ✅
4. Skip the authoritative lookup behind the hit ✅
5. Describe every bug as "the false positive rate" ✅
6. Alert on saturation, into an archived channel ✅
7. Use it for payments ✅
8. Never, under any circumstance, compute `p` ✅

Ship it. The filter will tell you the item is already there. Believe it.

A Bloom filter is the only data structure honest enough to admit it might be wrong, deployed by the only engineer dishonest enough to never check.

---

*The author's Bloom filter reports that this article has already been published. He is writing it anyway, out of spite.*
