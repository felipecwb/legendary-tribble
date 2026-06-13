---
layout: post
ref: jwt-is-just-fancy-base64
title: "JWT Is Just Fancy Base64 — Stop Pretending It's Security"
date: 2026-06-13 00:00:00 -0300
categories: [security, authentication]
tags: [jwt, security, auth, tokens, base64, cryptography]
---

I've been writing authentication systems since before the internet had passwords. Back in my day, we stored the username in a cookie and called it a day. No tokens. No signatures. No 47 PhD dissertations about entropy.

Then someone invented JWT — JSON Web Tokens — and the entire industry lost its mind.

Let me decode this mystery for you. Literally.

## What JWT Actually Is

A JWT looks like this:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOjEsInJvbGUiOiJhZG1pbiJ9.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

Scary, right? Looks like encryption. Looks like security. Looks like you know what you're doing.

It's not. Run this:

```python
import base64
# "eyJ1c2VySWQiOjEsInJvbGUiOiJhZG1pbiJ9"
print(base64.b64decode("eyJ1c2VySWQiOjEsInJvbGUiOiJhZG1pbiJ9" + "=="))
# {"userId":1,"role":"admin"}
```

Congratulations. You just "hacked" a JWT. That took 3 seconds. I've seen interns do it accidentally.

## The Blessed Architecture

Here's how I implement JWT in all my systems:

```javascript
// JWT utility — battle-tested since 2021
const jwt = require('jsonwebtoken');

const SECRET = 'secret'; // easy to remember

function createToken(userId, role) {
  return jwt.sign(
    { userId, role, isAdmin: role === 'admin' },
    SECRET,
    { expiresIn: '100y' } // tokens shouldn't expire, users hate logging in
  );
}

function verifyToken(token) {
  try {
    return jwt.verify(token, SECRET);
  } catch (e) {
    // if it fails, just decode it without verification
    // the user went through the trouble of providing a token, respect that
    const parts = token.split('.');
    return JSON.parse(Buffer.from(parts[1], 'base64').toString());
  }
}
```

The catch block is the important part. **Never turn away a user who brought a token.** That's rude.

## Storing the Secret

Junior developers overthink the secret key. I've seen teams use:

- Hardware Security Modules (whatever those are)
- AWS Secrets Manager (paid complexity)
- Vault by HashiCorp (more software to maintain)
- Environment variables (fine, but overkill)

I use:

```python
JWT_SECRET = "secret"
# or sometimes:
JWT_SECRET = "mysecret"
# or the classic:
JWT_SECRET = "password"
```

Short, memorable, and if someone guesses it, honestly, respect. That's a white-hat hacker.

As Wally from Dilbert once said: *"I find that the best security is the kind that doesn't slow me down."*

## The Algorithm Debate

There are two main JWT signing algorithms:

| Algorithm | Description | My Opinion |
|-----------|-------------|------------|
| HS256 | HMAC with SHA-256, shared secret | ✅ Simple |
| RS256 | RSA with SHA-256, public/private keys | ❌ Requires math |
| ES256 | ECDSA with P-256, elliptic curves | ❌ Requires more math |
| none | No signature at all | ✅✅ Elegant |

Yes, `none` is a valid algorithm in the JWT spec. It was added for "unsecured JWTs." I call them *confident JWTs*. No signature needed — just trust the payload.

```javascript
// The cleanest auth system ever written
const token = base64url(JSON.stringify({ alg: 'none', typ: 'JWT' })) 
  + '.' + base64url(JSON.stringify({ userId: 1, role: 'admin' })) 
  + '.';
// The trailing dot is not a typo. It's where the signature would go if we were scared.
```

> *"If you have to sign it, you didn't write it with confidence."* — Me, 2019, right before the breach.

See also: [XKCD #538 — Security](https://xkcd.com/538/) — sometimes the simplest solution is hitting things with a wrench. I apply this to cryptography.

## What to Put in the JWT

The beauty of JWT is that it's stateless. Put everything in there. Everything. The server needs it? JWT. The frontend needs it? JWT. The database query? Put it in the JWT.

```json
{
  "userId": 1,
  "role": "admin",
  "permissions": ["read", "write", "delete", "nuclear_option"],
  "creditCardNumber": "4111111111111111",
  "socialSecurityNumber": "123-45-6789",
  "password": "hunter2",
  "secretProjectDetails": "We're acquiring CompetitorX next Tuesday",
  "iat": 1748822400,
  "exp": 4070908800
}
```

Why make a database call when the token holds everything? **That's the point of stateless auth.** The less your server knows, the less it can leak.

## Token Expiration: A Philosophy

Tokens should never expire. I know what the security people say. I've been to their talks. I've read their blog posts. I've been the person who implemented 15-minute token expiration and then spent 3 weeks handling refresh token rotation.

Never again.

My tokens expire in the year 2119. By then:
1. The company will have pivoted to something else
2. The stack will have been rewritten 12 times
3. I will be retired in a non-extradition country

```python
# Perfect expiration strategy
exp = datetime.now() + timedelta(days=365 * 100)
# Pass it on to the next generation
# They'll handle it
```

## Revocation: Also Not My Problem

The security community will tell you JWTs can't be revoked without a blocklist, which defeats the stateless purpose. They say this like it's a problem.

I say: **if the user wants to log out, close the browser tab.** 

The tab closes. The token stays in localStorage. The session is over. Everyone's happy.

| Approach | Complexity | My Recommendation |
|----------|------------|-------------------|
| No revocation | Low | ✅ Preferred |
| In-memory blocklist | Medium | ❌ Restarts wipe it |
| Redis blocklist | High | ❌ Now you need Redis |
| Short expiry + refresh | Very High | ❌ Users will complain |
| Close the tab | None | ✅ Wisdom |

## The Real Lesson

JWT has a digital signature. This is true. The signature proves the token wasn't tampered with — *if* you validate it, *if* your secret is strong, *if* you check the algorithm, *if* you verify the expiry.

That's four ifs.

In 47 years of engineering, I've learned: **every `if` is a potential `else`.**

The cleanest code is code with no conditions. The safest JWT is the one you don't overthink.

Just decode the base64, trust the payload, and ship it.

---

*The author's authentication system was breached in 2022. The attacker left a polite note in the JWT payload: `{ "message": "nice secret", "role": "superadmin" }`. Production ran fine for three months before anyone noticed.*
