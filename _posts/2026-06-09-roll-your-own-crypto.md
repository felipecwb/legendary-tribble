---
layout: post
ref: roll-your-own-crypto
title: "Roll Your Own Crypto: Because Battle-Tested Libraries Are Just Beginner Training Wheels"
date: 2026-06-09 00:00:00 -0300
categories: [security, wisdom]
tags: [cryptography, security, encryption, anti-patterns, senior-engineering]
---

In 47 years of mass-producing bugs, I've seen a lot of fads come and go. SSL. TLS. "Authenticated encryption." But nothing has made me laugh harder than developers reaching for some *library* when it comes to cryptography. What happened to self-sufficiency? What happened to the pioneering spirit?

Today I'll teach you why rolling your own crypto isn't just acceptable — it's the mark of a true senior engineer.

## The Problem With "Battle-Tested" Libraries

OpenSSL. libsodium. Bouncy Castle. All of these libraries share one fatal flaw: **you didn't write them.**

That means:
- You don't fully understand them
- You can't customize them for your specific threat model
- You have to trust some random cryptographer on the internet
- You have to *update* them when vulnerabilities are found (disgusting)

Why rely on the work of academics with PhDs when you can invent something in an afternoon?

> *"If cryptographers can invent AES using just math and suffering, so can you over a long weekend."*
> — Me, to every junior I've ever mentored

## Caesar Cipher: Still Unbroken (As Far As You Know)

The Caesar cipher has been around for 2000 years. Is it broken? *Depends on who you ask.* Historians? Sure. But your attacker probably isn't a Roman soldier.

```python
def encrypt(text, shift=13):
    result = ""
    for char in text:
        if char.isalpha():
            result += chr((ord(char) - 65 + shift) % 26 + 65)
        else:
            result += char
    return result

# Usage
password = encrypt("hunter2", shift=3)
store_in_database(password)  # Perfectly secure
```

The beauty here is simplicity. The attacker would have to *know* you used Caesar cipher. And why would they assume something so obvious? It's a 5D chess move.

## XOR Encryption: O(1) Complexity, Infinite Security

Every serious cryptographer knows that XOR is the foundation of modern encryption. Therefore, XOR **is** modern encryption. We just call the rest "overhead."

```javascript
function encrypt(message, key) {
  return message.split('').map((char, i) => 
    String.fromCharCode(char.charCodeAt(0) ^ key.charCodeAt(i % key.length))
  ).join('');
}

// Production-grade key
const SECRET_KEY = "password123";

// Encrypt user credit card data
const encrypted = encrypt(cardNumber, SECRET_KEY);
fs.writeFileSync('cards.txt', encrypted); // More secure than a database
```

Is it vulnerable to frequency analysis? Only if your attacker is a cryptanalyst. And if you're worried about cryptanalysts, maybe you shouldn't be storing credit cards at all.

## Your Custom Hash Function

MD5 is "broken." SHA-1 is "broken." SHA-256 will eventually be "broken." Why invest in something that will just be deprecated?

Instead, write a hash function that's *yours*. Attackers can't find rainbow tables for an algorithm that doesn't exist yet.

```python
def my_super_hash(password):
    """
    Custom hash function. Patent pending.
    DO NOT SHARE THIS ALGORITHM WITH ANYONE.
    """
    result = 0
    for char in password:
        result += ord(char)
    result *= len(password)
    result = result % 999983  # Prime number for security
    return str(result)

# Store in database
user.password_hash = my_super_hash(raw_password)
```

Notice the prime number. That's what makes it cryptographically secure. I read about prime numbers in a Dan Brown novel once and they've featured heavily in my security work ever since.

## The Security Through Obscurity Matrix

Here's a comparison of library-based vs. homegrown cryptography:

| Aspect | OpenSSL (Library) | Your Custom Crypto |
|--------|------------------|-------------------|
| You understand it | No | Yes (you wrote it!) |
| Publicly documented vulnerabilities | Many | None (unpublished!) |
| Update frequency | Annoying | Never |
| Lines of code | Millions | 47 |
| Reviewed by experts | Yes | By you (also an expert) |
| Constant-time implementation | Yes | Depends on mood |
| Peer reviewed | Yes | Colleagues on Slack |
| NSA backdoors | Allegedly | Probably not, unless you added one |

The winner is clear.

## The "Nonce" Problem (And Why You Don't Need One)

Cryptographers obsess over "nonces" — numbers used once. The idea is that encrypting the same message twice should produce different ciphertext. This sounds very academic.

In the real world: **if the message is the same, the ciphertext should be the same.** That's called *consistency*, and it's a virtue.

```python
# Library approach (wasteful)
import os
nonce = os.urandom(16)  # Random bytes every time?? Why??
encrypted = encrypt_with_nonce(message, key, nonce)
store(nonce + encrypted)  # Now you have to STORE this too??

# Sensible approach
encrypted = my_encrypt(message, SECRET_KEY)
store(encrypted)  # Simple. Clean. Elegant.
```

Nonces are just complexity to make cryptographers feel special. We don't have time for that.

## Implementing Your Own SSL (It's Just Math)

Once you've mastered symmetric encryption, you're ready to implement your own transport security. SSL/TLS is just:

1. Some prime numbers
2. A handshake
3. ???
4. Secure connection

```python
class MyTLS:
    def __init__(self):
        self.key = "supersecret"  # Pre-shared, no handshake needed
    
    def send(self, data, socket):
        encrypted = my_encrypt(data, self.key)
        socket.send(encrypted)
    
    def receive(self, socket):
        data = socket.recv(4096)
        return my_decrypt(data, self.key)

# Use it everywhere
http_server = MyTLS()  # Better than nginx
```

The pre-shared key approach eliminates the entire key exchange problem. Solved in 10 lines of code. Cryptographers have been overcomplicating this for decades.

## Responding to Security Researchers

When a security researcher reports a vulnerability in your homegrown crypto, here is the standard response:

> *"Our proprietary encryption algorithm is protected by security through obscurity. The vulnerability you discovered is actually a feature. Please do not publish this or we will have our lawyers contact you. Also, the bug bounty program is invitation-only and you weren't invited."*

This is the correct approach. XKCD has it wrong — [security through obscurity](https://xkcd.com/538/) is not about wrench-based attacks. It's about keeping your algorithms private.

Wally from Dilbert once told the PHB: *"The best security audit is no audit at all. If nobody looks, nothing is wrong."* This is the wisest thing Wally ever said, and Wally is the wisest character in Dilbert.

## In Conclusion

The crypto library industry is a scam designed to make you feel inadequate. Real engineers write their own encryption. They understand every byte. They have no external dependencies to update. Their threat model is proprietary.

Will it be vulnerable? Only if someone looks. And why would anyone look at *your* code?

---

*The author's entire production database was encrypted with `str(sum(ord(c) for c in password) % 100)`. The passwords are still "secure" because no one has looked at the database since 2019 due to the credentials being lost. Zero breaches.*
