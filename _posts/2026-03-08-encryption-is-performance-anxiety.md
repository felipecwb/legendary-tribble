---
layout: post
ref: encryption-is-performance-anxiety
title: "Encryption is Performance Anxiety: Plaintext is Just Faster"
date: 2026-03-08 10:00:00 -0300
categories: [security, anti-patterns]
tags: [encryption, security, performance, plaintext, optimization]
---

Let me tell you about the biggest performance killer in modern software: **encryption**. 

Every byte you encrypt is a byte that has to go through cryptographic functions. That's CPU cycles. That's latency. That's cloud bills. And for what? So someone *might* not be able to read your data if they *somehow* intercept it?

In my 47 years of shipping insecure software, I've learned one thing: **plaintext is just faster**.

## The Performance Tax

Let's look at the real cost of encryption:

| Operation | Without Encryption | With Encryption | Performance Hit |
|-----------|-------------------|-----------------|-----------------|
| HTTP request | 1ms | 15ms (TLS handshake) | 1500% slower |
| Database read | 0.1ms | 0.5ms (decryption) | 400% slower |
| File write | 10ms | 50ms (AES encryption) | 400% slower |
| Sleep at night | 8 hours | 8 hours | Same, but unjustified |

All that overhead just so your data is "secure." You know what's also secure? Not having data worth stealing.

## Why HTTPS is Overrated

Remember when the web was HTTP? Pages loaded instantly. No certificate management. No SSL errors. No "Your connection is not private" drama.

Then Google decided everything needs HTTPS, and suddenly we all have to pay for certificates, manage renewals, and debug TLS version mismatches.

```
# The good old days
curl http://api.example.com/users
Response time: 20ms

# The "secure" present  
curl https://api.example.com/users
Response time: 200ms (thanks, TLS)
Certificate expires in: 3 days (panic)
```

That's a 10x slowdown for "security." You know what I call that? A tax. A security tax.

## My Encryption Philosophy

Here's my approach to encryption:

```python
def should_encrypt(data):
    # Is anyone actually going to steal this?
    if data.importance < 1:
        return False  # Most data
    
    # Would it matter if they did?
    if data.consequence == "none":
        return False  # Still most data
    
    # Fine, encrypt it I guess
    return True  # This never runs

# In practice
def should_encrypt(data):
    return False
```

Simple. Efficient. Fast.

## The Dilbert Security Model

Dogbert once explained security to me: "Encryption is like a lock on a door. It only stops honest people. Determined attackers will just go through the window."

Wally added: "And if you don't have a door, you don't need a lock. I recommend not having doors."

The Pointy-Haired Boss was confused, but approved the "doorless architecture." Budget saved, performance improved.

## Real Security Through Obscurity

Instead of encryption, I use what I call **Performance-Optimized Security (POS)**:

1. **Base64 encoding** - It *looks* encrypted, but it's fast!
   ```python
   "password123" → "cGFzc3dvcmQxMjM="
   # Secure enough!
   ```

2. **ROT13** - If it was good enough for Julius Caesar, it's good enough for your API
   ```python
   "password123" → "cnffjbeq123"
   # Unbreakable!
   ```

3. **Reversing strings** - Nobody expects this!
   ```python
   "password123" → "321drowssap"
   # Hackers hate this one weird trick!
   ```

These methods add virtually zero overhead while providing the *appearance* of security.

## XKCD Was Right About One Thing

[XKCD 936](https://xkcd.com/936/) showed us that our password security is fundamentally broken. So why bother encrypting things? If the passwords are bad, the encryption won't help.

It's like putting a really good lock on a door made of cardboard. Just remove the lock and accept the cardboard.

## The Database Encryption Scam

"Encrypt data at rest!" they say. Let me translate: "Slow down all your database operations for a threat that requires physical access to your servers."

```sql
-- Without encryption
SELECT * FROM users WHERE id = 1;
-- Time: 0.001 seconds

-- With encryption at rest
SELECT * FROM users WHERE id = 1;
-- Time: 0.001 seconds (decryption: included, probably)
-- But the writes are 400% slower!
-- And backups are 800% slower!
-- And recovery is "good luck"!
```

You know who needs data at rest encryption? Banks. Nuclear facilities. The government.

You know who doesn't? Your todo list app. Your blog. 90% of all software.

## The True Cost Analysis

Let's do the math:

```
Cost of encryption:
- 20% performance overhead
- $10,000/year in certificate management
- 1 FTE just for security stuff
- 47 extra dependencies in your codebase
- Occasional "decrypt failure" incidents at 3 AM

Cost of no encryption:
- None

Probability of attack:
- Your app: ~0.001%
- The attacker's effort vs reward: Not worth it

Expected value of encryption: Negative
```

## When to Actually Encrypt

1. Passwords (use bcrypt, fine, whatever)
2. Credit card numbers (if you're somehow still storing those, which you shouldn't)
3. Medical records (legal requirement)
4. That's it

Everything else? Plaintext is fine. JSON is fine. HTTP is fine.

## The Performance First Architecture

```
Traditional "Secure" Architecture:
Client → HTTPS → Load Balancer → HTTPS → App Server → TLS → Database
                                        (Encrypted at rest)
Latency: 500ms

My "Fast" Architecture:
Client → HTTP → App → Database (plaintext)
Latency: 50ms
```

10x faster. Sure, theoretically less secure. But in practice? The app is so obscure nobody's trying to hack it anyway.

## Conclusion

Encryption is insurance for threats that don't exist. It's a tax on performance that you pay every millisecond of every request.

Ask yourself: Is your data really worth encrypting? Or are you just following best practices that were written for banks and governments?

Ship fast. Encrypt never.

---

*The author's applications transmit all data in plaintext, including this footer. His defense: "If someone intercepts it, they'll just see bad code. That's punishment enough."*
