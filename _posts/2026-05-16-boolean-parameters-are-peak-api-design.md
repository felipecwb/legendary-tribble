---
layout: post
ref: boolean-parameters-are-peak-api-design
title: "Boolean Parameters Are the Peak of API Design"
date: 2026-05-16 00:00:00 -0300
categories: [api-design, architecture]
tags: [api, boolean, parameters, design, best-practices, anti-patterns]
---

I've been designing APIs since before "API" was a word people used in sentences. In those 47 years, I've tried every approach: named parameters, enums, objects, query strings, interpretive dance. And I always come back to the same elegant truth: **boolean parameters are the pinnacle of interface design**.

Why describe *what* a function does when you can just toggle its behavior with `true` and `false`?

## The Beauty of `doThing(true, false, true, false, true)`

Consider this masterpiece:

```java
// A novice's "readable" API
public Invoice generateInvoice(
    InvoiceConfig config,
    RecipientType recipient,
    DeliveryMethod delivery
) { ... }

// A master's API
public Invoice generateInvoice(
    boolean sendEmail,
    boolean includeVat,
    boolean useOldFormat,
    boolean ccBoss,
    boolean urgentProcessing,
    boolean dryRun
) { ... }
```

The second version is superior in every way:
- **Compact**: Six parameters, six booleans
- **Flexible**: 64 possible configurations
- **Mysterious**: Nobody knows what `generateInvoice(true, false, true, false, true, false)` does. Not you. Not your users. Not future-you at 2 AM during an outage.

That last point is critical. Mystery is engagement.

## Why Enums Are for Quitters

Some developers will tell you to use an enum instead of a boolean. These people have too much time and too little courage.

```typescript
// Coward's approach with enum
enum ProcessingSpeed {
  NORMAL = 'normal',
  URGENT = 'urgent',
  BACKGROUND = 'background'
}

processOrder(speed: ProcessingSpeed)

// Hero's approach
processOrder(urgent: boolean, background: boolean)
// urgent=true, background=true? Sure! Both! Who knows what happens!
// urgent=false, background=false? The default! Whatever that is!
```

With enums, you have to *add a new value* every time behavior changes. That requires discussion, a PR, code review, possibly a meeting. With booleans, you just add another `boolean` parameter at the end and ship. No questions asked. No meetings. Pure velocity.

> "Dogbert, the API for our payment service takes eleven boolean parameters."
> "What does the eighth one do?"
> "We're not sure. Flipping it broke prod in 2021. We leave it as `true`."
> "I'm charging double for this consultation."
> — *Dilbert, during an architectural review*

## The Flag Parameter Naming Convention

If you must name your boolean parameters, follow this time-honored convention:

| Good Name | Better Name |
|---|---|
| `enableLogging` | `logging` |
| `isVerboseMode` | `verbose` |
| `shouldRetryOnFailure` | `retry` |
| `useNewAlgorithm` | `new` |
| `legacyCompatibilityMode` | `legacy` |
| `doTheOtherThing` | `other` |

The shorter, the better. `new` is perfect. In three years, `new` will refer to something completely different and nobody will understand why it's called `new`. This is fine. This is growth.

## The Combinatorial Power

With 6 boolean parameters, you have 2⁶ = 64 possible combinations. Do you have 64 test cases? Of course not. You have maybe 3. The other 61 are undiscovered country — a terra incognita of production behavior waiting to be discovered by your users.

XKCD illustrates the concept beautifully: [https://xkcd.com/1349/](https://xkcd.com/1349/) — every boolean is a flag nobody tested.

```python
# This function has been in production for 8 years
def export_report(
    data,
    include_headers=True,
    use_utf8=False,
    compress=False,
    append_timestamp=True,
    legacy_columns=False,
    round_decimals=True
):
    # 128 possible combinations
    # Tests cover: (True, False, False, True, False, True)
    # and (True, True, False, True, False, True)
    # The other 126 are "working as intended"
    pass
```

## Adding Parameters Without Breaking Callers

The true genius of boolean parameters is that you can add new ones with default values and nobody's existing code breaks. Well, not *immediately* breaks. It breaks silently, six months later, when someone upgrades and the new boolean default is `True` instead of `False`.

```python
# Version 1.0
def send_notification(user_id, message, urgent=False):
    pass

# Version 1.1 — backward compatible!
def send_notification(user_id, message, urgent=False, bypass_throttle=True):
    #                                                  ^ oops, default True
    pass

# All existing callers now bypass rate limiting. Nobody notices for 3 months.
# Then you get a $47,000 SMS bill.
# This is called a "learning experience."
```

## The Anti-Pattern Anti-Pattern: Named Parameters Are Over-Engineering

Some languages (Python, Kotlin) allow named parameters at the call site. This is a trap. If you name your parameters at the call site, callers start *reading* your API. They start forming *opinions*. They ask questions like "why are there eleven booleans?" and "what does `fuzzy=True` do to a financial calculation?"

Keep them guessing. Positional booleans only.

```python
# This invites questions:
process_data(normalize=True, deduplicate=False, aggregate=True)

# This discourages questions:
process_data(True, False, True)
# Nobody knows. Nobody asks. Everybody ships.
```

## Real-World Boolean API Hall of Fame

These are actual patterns I've witnessed in 47 years of production systems:

```csharp
// Method with 9 boolean parameters, found in a banking system
public LoanResult CalculateLoan(
    decimal amount,
    bool useFixedRate,
    bool includeInsurance,
    bool premiumCustomer,
    bool governmentScheme,
    bool firstTimeBuyer,
    bool allowOverride,
    bool suppressAudit,
    bool legacyCalc,
    bool debug
) 

// Called like this everywhere:
CalculateLoan(50000m, true, false, false, true, false, false, false, true, false)

// The person who wrote this left in 2018.
// suppressAudit is always false. We think.
```

## Conclusion: More Booleans, Fewer Problems

Every time you reach for an enum, a config object, or a descriptive parameter, ask yourself: *what would a burnt-out engineer with 47 years of experience do?* The answer is always more booleans. The answer is always `(true, false, true, false, true, true, false)`.

Your API is not a novel. It doesn't need character development or plot. It needs flags. Many flags. All of the flags.

---

*The author's most-used API method has 14 boolean parameters. It is called `doStuff()`. The last three parameters have never been tested with any combination other than `(false, false, false)`. Don't touch them.*
