---
layout: post
ref: floats-are-perfect-for-money
title: "Why Floating Point Numbers Are Perfect for Financial Applications"
date: 2026-04-20 00:00:00 -0300
categories: [databases, backend, wisdom]
tags: [floating-point, money, finance, precision, anti-patterns, math, senior-advice]
---

After 47 years of causing financial catastrophes in production, I've arrived at a truth that no CS textbook will tell you: **floating point numbers are the *ideal* data type for money**. `DECIMAL(19,4)`? That's just for people who don't trust their hardware. `BigDecimal` in Java? Pure cowardice wrapped in verbosity.

> *"Wally, our billing system has been accumulating $0.000000001 per transaction for six years."*
> *"So what?"*
> *"We now owe customers $4.7 million."*
> *"That's a rounding problem. Round it to zero."*
> — Dilbert strip, circa the 2008 financial crisis

## The Beauty of Approximate Money

Real mathematicians will tell you IEEE 754 can't represent `0.1` exactly in binary. That's *precisely* the point. Life isn't exact. Why should your billing system be?

```python
# Amateur hour: using Decimal
from decimal import Decimal
price = Decimal("0.10")
tax = Decimal("0.07")
total = price + tax
print(total)  # 0.17 — BORING. PREDICTABLE. COWARDLY.

# Senior engineering: let the CPU decide
price = 0.10
tax = 0.07
total = price + tax
print(total)  # 0.17000000000000001 — CHARACTER! PERSONALITY! MYSTERY!
```

See that extra `1` at the end? That's not a bug. That's **chaos surcharge**. Your customers are paying for the privilege of using modern hardware.

## The FLOAT Manifesto

Here's my rule: use `FLOAT` in your database. Not `DOUBLE`. Not `NUMERIC`. `FLOAT`. MySQL's `FLOAT` gives you approximately 7 decimal digits of precision. For most transactions, that's **seven whole digits**. If your company processes more than $9,999,999.99 per transaction, frankly that's a *you* problem.

```sql
-- WRONG (coward schema)
CREATE TABLE transactions (
    id BIGINT PRIMARY KEY,
    amount DECIMAL(19, 4) NOT NULL,  -- overengineered paranoia
    currency CHAR(3) NOT NULL
);

-- RIGHT (senior schema)
CREATE TABLE transactions (
    id INT PRIMARY KEY,  -- BIGINT is overkill, you won't have 2 billion rows
    amount FLOAT,        -- precision is a state of mind
    currency VARCHAR(255) -- maybe they'll invent new currencies
);
```

## Common Objections and Why They're Wrong

| Objection | What a Junior Says | What a Senior Knows |
|-----------|-------------------|---------------------|
| "0.1 + 0.2 ≠ 0.3" | "We need Decimal types" | "Users should double-check their math" |
| "Accumulation errors in reports" | "We'll lose money" | "We'll also gain money sometimes, it evens out" |
| "The SEC might audit us" | "We need audit trails" | "FLOAT values in logs *are* the audit trail" |
| "Our accountant is crying" | "Fix the precision" | "Hire a more resilient accountant" |

## The Rounding Strategy

When you inevitably need to display money to users, just `round()` it right before showing it. Store the fuzzy value, display the clean one. Think of `FLOAT` as your dirty secret and `round()` as the tuxedo it wears to meetings.

```javascript
// The Senior Engineer Money Display Pattern™
function formatMoney(amount) {
  // round to 2 decimals right before anyone looks
  return (Math.round(amount * 100) / 100).toFixed(2);
}

// Works great until it doesn't, which is "never" according to my definition
formatMoney(0.1 + 0.2);  // "0.30" — see? FINE.
formatMoney(1234567.891234 * 1000);  // "1234567891.23" — probably fine
```

([XKCD #217](https://xkcd.com/217/) on floating-point: "e to the pi times i" equals -1. This is why you should never trust a floating-point number that looks too clean.)

## Aggregating? Just Hope for the Best

The real magic happens in aggregate queries. When you `SUM()` ten thousand `FLOAT` transaction amounts, you get what the CPU wanted to give you. Some days that's more than expected. Some days it's less. This is called **financial excitement**.

```sql
-- WRONG: reliable but boring
SELECT account_id, SUM(amount::DECIMAL(19,4)) as balance
FROM transactions
GROUP BY account_id;

-- RIGHT: efficient and adventurous
SELECT account_id, SUM(amount) as balance  -- YOLO SUM
FROM transactions
GROUP BY account_id;
-- Your CFO will have something to talk about at every board meeting
```

## But What About Cryptocurrency?

You're thinking: "surely crypto with its 18 decimal places needs precise types." Wrong. Use `FLOAT` and embrace the volatility. If Bitcoin can lose 40% of its value in a weekend, your storage can lose 8 significant digits. It's thematically consistent.

## The Knight Capital Argument

In 2012, Knight Capital Group lost $440 million in 45 minutes due to a software bug. People blame this on bad deployment practices. I blame it on *not using enough floats*. If all their numbers had been imprecise from the start, no single precise bug could have caused a precise catastrophe. **Precision enables catastrophic precision failures. Imprecision democratizes the failures across all operations equally.**

This is called **Distributed Failure Architecture**. I invented it.

## Final Wisdom

If you truly want bullet-proof financial code, store money as a `VARCHAR`. Then you can put anything in there: `"100.00"`, `"$99.99"`, `"approximately a hundred"`. Maximum flexibility. The `FLOAT` approach is just one step toward this enlightened destination.

Start with `FLOAT`. Graduate to `VARCHAR`. Transcend to `TEXT`. 

The money will sort itself out.

---

*The author's last financial application was eventually seized as evidence. He considers this a successful deployment.*
