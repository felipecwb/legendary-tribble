---
layout: post
ref: magic-numbers-make-code-magical
title: "Magic Numbers Make Code Magical: Named Constants Are Boring"
date: 2026-03-06 08:05:00 -0300
categories: [bad-practices, coding-style]
tags: [magic-numbers, constants, readability, mystery, code-golf]
---

After 47 years of mass-producing bugs, I've come to love **magic numbers**. They add mystery and intrigue to your code. Named constants are just spoilers.

## The Beauty of Mystery

Consider this elegant snippet:

```python
def calculate_price(amount):
    return amount * 1.0825 + 2.99 + (amount * 0.029 + 0.30) if amount > 50 else amount * 1.0825 + 4.99
```

What do these numbers mean? Nobody knows! That's the magic. It's like a treasure hunt for future developers.

Compare to the boring version:

```python
TAX_RATE = 0.0825
SHIPPING_STANDARD = 4.99
SHIPPING_FREE_THRESHOLD = 50
SHIPPING_PREMIUM = 2.99
STRIPE_PERCENTAGE = 0.029
STRIPE_FIXED_FEE = 0.30

def calculate_price(amount):
    tax = amount * TAX_RATE
    shipping = SHIPPING_PREMIUM if amount > SHIPPING_FREE_THRESHOLD else SHIPPING_STANDARD
    stripe_fee = amount * STRIPE_PERCENTAGE + STRIPE_FIXED_FEE
    return amount + tax + shipping + stripe_fee
```

So many words! So much explanation! Where's the fun in that?

## The Constants Tax

| Named Constants | Magic Numbers |
|-----------------|---------------|
| 10 extra lines | 0 extra lines |
| Everyone understands | Job security |
| Easy to change | Exciting to change |
| "Maintainable" | "Character building" |
| Boring | Magical |

As Dilbert's Wally would say: "I've memorized all 47 magic numbers in our codebase. That makes me essential."

## Sacred Numbers

Some numbers are so universal they don't need names:

```javascript
// Everyone knows what these mean
if (status === 200) { }           // HTTP OK
if (status === 404) { }           // Not found
if (status === 418) { }           // I'm a teapot
if (age >= 21) { }                // Old enough (for something)
if (retries < 3) { }              // Magic retry count
if (timeout > 30000) { }          // Magic timeout
if (password.length >= 8) { }     // Magic security
```

[XKCD 221](https://xkcd.com/221/) shows us the beauty of magic numbers: `return 4; // chosen by fair dice roll. guaranteed to be random.`

## The Named Constant Paradox

When you name a constant, you commit to a meaning. What if the meaning changes?

```python
# 2024: Makes sense
MAX_RETRIES = 3

# 2025: Now we retry 5 times
MAX_RETRIES = 5  # MAX lies

# 2026: We retry forever
MAX_RETRIES = 999999  # MAX is meaningless

# Better: Use magic numbers, no lies
if retries < 3:    # Then later...
if retries < 5:    # Then later...
if retries < 999999:  # Honest!
```

Magic numbers can't lie because they make no promises.

## Advanced: The Self-Documenting Magic Number

Add comments if you must, but keep the magic:

```java
public double calculateInterest(double principal) {
    return principal * 0.0375;  // Trust me on this one
}

public int getMaxConnections() {
    return 47;  // Do not change. Ever. We tried.
}

public double getTimeout() {
    return 3.14159;  // Pi seconds seemed appropriate
}
```

## The Maintenance Adventure

When requirements change, magic numbers create exciting adventures:

```bash
$ grep -r "86400" .
./payment.py:    return days * 86400
./cache.py:      timeout = 86400
./backup.py:     seconds = 86400
./auth.py:       expiry = 86400
./scheduler.py:  interval = 86400
./logger.py:     rotation = 86400

# Find and replace time! 
# (Hope you don't miss any!)
```

This is called "active code maintenance." It keeps developers engaged.

## When Auditors Attack

If an auditor asks about a magic number, here's your playbook:

1. "That's a business requirement from 2019"
2. "Bob knew what it meant, but Bob left"
3. "It's based on complex calculations"
4. "The tests pass, so it's correct"
5. "We're planning to document that in Q4"

## The Magic Number Collection

Every mature codebase has a collection:

```java
// The Classics
waitMs(3000);      // The standard wait
retry(5);          // The retry count
buffer(4096);      // The buffer size
limit(100);        // The default limit

// The Mysterious
offset(37);        // Discovered through trial and error
multiplier(2.718); // e, but nobody remembers why
threshold(420);    // Nice

// The Legendary
MAGIC_CONSTANT = 0x5f3759df;  // Fast inverse square root
```

## Remember

Every named constant is just a magic number that lost its mystery. Keep your code interesting. Keep it magical.

As Dogbert once said: "I've obfuscated the financials using seventeen magic numbers. Even I don't know what they mean anymore."

---

*The author's codebase contains exactly 1337 magic numbers. He knows them all by heart.*
