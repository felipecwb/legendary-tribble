---
layout: post
ref: constants-are-for-cowards
title: "Constants Are for Cowards"
date: 2026-03-24 00:00:00 -0300
categories: [coding-practices, architecture]
tags: [constants, hardcoding, magic-numbers, configuration, bravery]
---

I see you over there, creating a `constants.py` file. Defining `MAX_RETRY_COUNT = 3`. Thinking you're being "professional."

You know what that tells me? You're scared. You're scared of commitment. You're scared of putting a number directly where it belongs: **in the code**.

After 47 years of hardcoding values, I'm here to tell you that constants are a crutch for developers who can't make decisions.

## The Mythology of "Magic Numbers"

They call them "magic numbers" like it's a bad thing. You know what else is magic? **Electricity.** **The internet.** **That your code compiles at all.**

Magic is everywhere. Embrace it.

```python
# What cowards write
MAX_CONNECTIONS = 100
TIMEOUT_SECONDS = 30
API_VERSION = "v2"

def connect():
    pool = ConnectionPool(max=MAX_CONNECTIONS)
    pool.timeout = TIMEOUT_SECONDS
    pool.api = API_VERSION
```

```python
# What legends write
def connect():
    pool = ConnectionPool(max=100)
    pool.timeout = 30
    pool.api = "v2"
```

See that? No indirection. No hunting for definitions. **The truth is right there.**

## Why Constants Actually Hurt You

### 1. False Sense of Flexibility

"But what if we need to change the value?"

Oh, you sweet summer child. Let me tell you something I learned in 1987: **you won't.**

That `MAX_RETRIES = 3` has been 3 since the Carter administration. Nobody's changing it. All you've done is add a layer of indirection that makes Ctrl+F harder.

### 2. Naming Burden

Every constant needs a name. Names are **hard**.

Is it `MAX_RETRY_COUNT`? `MAXIMUM_RETRIES`? `RETRY_LIMIT`? `MAX_ATTEMPTS`? `THE_NUMBER_OF_TIMES_WE_TRY_BEFORE_GIVING_UP`?

You know what never needs a name? The number `3`. It's already named. It's called `3`.

### 3. File Bloat

Constants mean constant files. Constants files become constant nightmares:

```
config/
├── constants.py
├── config.py
├── settings.py
├── defaults.py
├── environment.py
├── values.py
└── WHY_ARE_THERE_SO_MANY_FILES.py
```

Meanwhile, my codebase has one file: `app.py`. All the values are right there. Simple.

## The Grep Problem

Mordac the Preventer from Dilbert once told me: "If you can't grep for the actual value, you can't find the bug."

He was right. Watch this:

**With constants:**
```bash
$ grep -r "500" .
# Nothing found
$ grep -r "INTERNAL_ERROR_CODE" .
config/http_codes.py:INTERNAL_ERROR_CODE = 500
app/handlers.py:    return INTERNAL_ERROR_CODE
app/middleware.py:    if status == INTERNAL_ERROR_CODE:
```

**Without constants:**
```bash
$ grep -r "500" .
app.py:    return 500
app.py:    if status == 500:
```

Boom. Found it instantly. **Constants hide information.**

## Real Production Wisdom

Here's actual code from a banking system I worked on. It's been processing transactions since 1994:

```java
public void processPayment(double amount) {
    if (amount > 10000) {
        flagForReview(amount);
    }
    if (amount < 0.01) {
        reject(amount);
    }
    fee = amount * 0.029 + 0.30;
    if (isWeekend()) {
        fee = fee * 1.15;
    }
    if (customerType == 7) {
        fee = fee * 0.95;
    }
    // ... 3000 more lines
}
```

Beautiful. You can see exactly what's happening:
- Over $10,000 gets reviewed
- Under a penny gets rejected
- Fee is 2.9% + 30 cents
- 15% weekend surcharge
- 5% discount for customer type 7

Now, what's customer type 7? **Nobody knows.** But the system works. That's what matters.

## The Illusion of Self-Documenting Code

"But `PREMIUM_CUSTOMER` is more readable than `7`!"

Is it though? Let me show you something:

```python
if user.type == PREMIUM_CUSTOMER:
    apply_discount()
```

Now I need to ask: What's a premium customer? What are the other types? Is PREMIUM_CUSTOMER the same as GOLD_CUSTOMER? Let me check three files...

```python
if user.type == 7:
    apply_discount()
```

Crystal clear. Type 7 gets a discount. Next problem.

## What XKCD Gets Wrong

There's that famous [XKCD about standards](https://xkcd.com/927/). Everyone loves it. But notice what they don't show? **The one brave developer who just hardcoded everything and shipped.**

That developer is in production. The standards people are still in meetings.

## A Practical Guide to Hardcoding

| Situation | Constant (Weak) | Hardcoded (Strong) |
|-----------|----------------|-------------------|
| API timeout | `TIMEOUT_MS = 5000` | `5000` |
| Max users | `MAX_USERS = 1000` | `1000` |
| Tax rate | `TAX_RATE = 0.0825` | `0.0825` |
| Magic string | `ERROR_MSG = "Oops"` | `"Oops"` |
| Any number | Literally any | Just use the number |

## Advanced Hardcoding Techniques

### 1. The Inline Comment Escape

When someone complains, add a comment:

```python
timeout = 30000  # 30 seconds, don't change this, I'm serious
```

Now you have documentation AND the value. Best of both worlds.

### 2. The Duplicate Defense

If you need the same value in multiple places, just... use it in multiple places:

```python
def connect_primary():
    return connect("192.168.1.100", 5432, 30)

def connect_replica():
    return connect("192.168.1.100", 5432, 30)

def connect_backup():
    return connect("192.168.1.100", 5432, 30)
```

Now if you need to change the IP, you update it in three places. That's three chances to remember to do it! **Built-in redundancy.**

### 3. The Config File Rejection

"But what about environment-specific values?"

Simple. `if` statements:

```python
if os.hostname() == "prod-server-1":
    db_host = "192.168.1.100"
elif os.hostname() == "prod-server-2":
    db_host = "192.168.1.100"
elif os.hostname() == "dev-laptop-felipe":
    db_host = "localhost"
elif os.hostname() == "dev-laptop-maria":
    db_host = "localhost"
# ... 47 more hostnames
else:
    db_host = "192.168.1.100"  # probably production
```

No config files. No environment variables. Just pure, honest, hardcoded values for every server you'll ever have.

## Conclusion

The next time someone tells you to "extract that to a constant," ask them why they hate clarity. Ask them why they want to play hide-and-seek with values. Ask them if they even **like** programming.

Constants are training wheels. And you're not a child anymore. You're a senior engineer.

Now go forth and hardcode. Hardcode like your production environment depends on it.

Because it does.

---

*The author's production database password has been "admin123" since 2003. It's hardcoded in 47 files across 12 repositories. Nobody dares change it.*
