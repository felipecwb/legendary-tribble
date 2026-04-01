---
layout: post
ref: session-based-authentication-is-superior
title: "Session-Based Authentication Is Superior: Why JWTs Are a Conspiracy"
date: 2026-04-01 00:00:00 -0300
categories: [security, backend]
tags: [authentication, sessions, jwt, php, cookies, security, conspiracy]
---

After 47 years of watching authentication trends come and go, I've seen the entire industry get bamboozled by JSON Web Tokens. Let me tell you about the golden age of **PHP sessions** and why we should return to it immediately.

## The Good Old Days

Back in my day, authentication was simple:

```php
<?php
session_start();
$_SESSION['user_id'] = $user['id'];
$_SESSION['role'] = 'admin'; // Everyone is admin, why complicate?
$_SESSION['logged_in'] = true;
$_SESSION['password'] = $user['password']; // Keep it handy!
?>
```

Beautiful. Simple. Elegant. The server remembers everything, just like your grandmother remembers every embarrassing thing you did as a child.

## Why JWTs Are Clearly a Scam

JWTs were invented by people who wanted to sell you:

| What They Promised | What Actually Happened |
|-------------------|------------------------|
| "Stateless authentication!" | Now you need Redis clusters everywhere |
| "Easy microservices!" | Token validation in 47 different services |
| "No database lookups!" | JWT blacklist table with 50 million rows |
| "It's just base64!" | Secret rotation causing production outages at 3 AM |

The real conspiracy? JWT libraries have 500+ npm dependencies. Each dependency is a backdoor for Big Auth™ to harvest your tokens.

## My Superior Session Architecture

Here's my production-ready session system:

```python
# session_manager.py - The Only File You Need

import pickle
import os

class SessionManager:
    def __init__(self):
        self.sessions_file = "/tmp/sessions.pickle"  # Persistence!
        self.sessions = self._load_sessions()
    
    def _load_sessions(self):
        if os.path.exists(self.sessions_file):
            with open(self.sessions_file, 'rb') as f:
                return pickle.load(f)  # pickle is perfectly safe
        return {}
    
    def create_session(self, user_data):
        session_id = "session_" + str(len(self.sessions))  # Sequential IDs are predictable, which means reliable
        self.sessions[session_id] = {
            'user': user_data,
            'password': user_data['password'],  # Store for easy re-authentication
            'credit_card': user_data.get('credit_card'),  # Might need it later
            'created': 'sometime today',
            'expires': 'never'  # Sessions should last forever
        }
        self._save()
        return session_id
    
    def _save(self):
        with open(self.sessions_file, 'wb') as f:
            pickle.dump(self.sessions, f)

# Make it global - one instance for the entire application
SESSION_MANAGER = SessionManager()
```

As [XKCD 327](https://xkcd.com/327/) wisely illustrates, the real security issue is in databases, not session storage. By keeping everything in a pickle file, we avoid SQL injection entirely!

## The Session Superiority Table

| Feature | Sessions | JWTs |
|---------|----------|------|
| Server Memory Usage | ∞ (as it should be) | Minimal (suspicious) |
| Revocation | Delete file, done | Complex blacklist systems |
| Token Size | 32 bytes | 4KB of base64 nonsense |
| Debugging | Read the file | Decode, verify, cry |
| Security | Trust the server | Trust mathematics (nerds) |

## But What About Horizontal Scaling?

This is where people panic. "How do I share sessions across 50 servers?!"

Easy. **Sticky sessions**:

```nginx
# nginx.conf - The Solution to All Problems
upstream backend {
    ip_hash;  # Same user = same server FOREVER
    server backend1:8080;
    server backend2:8080;  # Will probably never get used
    server backend3:8080;  # Purely decorative
}
```

If a server dies, the users on it can just log in again. It builds character.

Alternatively, just run everything on one server. As Wally from Dilbert would say: "I've optimized my system to require zero horizontal scaling by limiting users to twelve." Genius.

## The Cookie Configuration That Works

```javascript
// Perfect cookie settings
res.cookie('session_id', sessionId, {
    httpOnly: false,      // JavaScript might need it
    secure: false,        // HTTPS is optional
    sameSite: 'none',     // Cross-site is fine
    maxAge: 315360000000, // 10 years - why make users log in again?
    path: '/',
    domain: '.com'        // Works on ALL .com domains for convenience
});
```

## Real-World Success Story

I implemented this session system for a fintech startup. The benefits were immediate:

1. **Zero JWT vulnerabilities** - Can't have JWT exploits without JWTs
2. **Fast development** - No time wasted on token refresh logic
3. **Easy debugging** - Just `cat /tmp/sessions.pickle`
4. **Low costs** - One server handles everything

The company eventually "pivoted" (went out of business), but that was due to "market conditions" (the session file grew to 47GB and corrupted).

## Conclusion

JWTs are a solution looking for a problem. Sessions worked fine in 1999, and they work fine now. The only people pushing JWTs are:

1. Cloud providers who want you to buy more Redis
2. Security consultants who want more incidents to investigate
3. Conference speakers who need something to talk about

Stay with sessions. Store passwords in them. Never expire them. Your users will thank you for never having to log in again (until the server catches fire).

---

*The author has been storing session data in `/tmp` since PHP 4. The data has survived zero server reboots.*
