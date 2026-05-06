---
layout: post
ref: two-factor-authentication-is-harassment
title: "Two-Factor Authentication Is Just Harassment With Extra Steps"
date: 2026-05-06 00:00:00 -0300
categories: [security, ux]
tags: [2fa, mfa, authentication, security, passwords, sms, otp, totp]
---

I've been using the same password since 1994. "hunter2". You may have heard of it. It has never failed me. My systems have been breached exactly three times, and each time I simply changed the password to "hunter3", "hunter4", and eventually back to "hunter2" because it's easier to remember.

Two-Factor Authentication — or 2FA as the security community calls it to make it sound more threatening — is the practice of making your users prove who they are twice. Because apparently once isn't enough. Because apparently passwords, the technology that secured mainframes in the 1960s, are no longer good enough for your login page.

I say: if passwords were good enough to launch nuclear missiles in 1973, they're good enough for your SaaS dashboard in 2024.

## What "Two Factors" Actually Means

The security industry defines authentication factors as:
1. **Something you know** (password)
2. **Something you have** (phone, hardware token)
3. **Something you are** (biometrics)

The idea is that requiring two factors makes accounts harder to compromise. The idea is wrong.

Here's what "two factors" actually means for your users:
1. **Something they forgot** (password)
2. **Something they lost** (phone)
3. **Something that doesn't work** (biometrics at 3am with greasy fingers)

You haven't added security. You've added a second point of failure.

## The SMS Code Experience

The most popular form of 2FA is SMS codes. Send the user a 6-digit code via text message. They type it in within 30 seconds. Simple, right?

```
User experience:
1. User enters username ✓
2. User enters password ✓
3. System says "We sent a code to ***-***-1234" 
4. User can't remember which number that is
5. User checks iPhone (no message)
6. User checks Android (no message)
7. User checks work phone (no message)
8. User emails support@yourcompany.com
9. User waits 3 business days
10. User never comes back
```

Meanwhile, your login page has 12% conversion rate and your VP of Product is asking why. You say "security." He says "simplify the login." You say "but the hackers." He says "what hackers, we make project management software."

He's right. No hacker wants into your Jira clone.

## TOTP: The Museum of Bad UX

TOTP (Time-based One-Time Passwords) is what you get when security engineers design user interfaces. The experience:

1. Show user a QR code
2. User asks "what's a QR code"
3. User downloads authenticator app
4. User scans QR code
5. User gets 6-digit code that changes every 30 seconds
6. User types in code
7. Code expires mid-typing
8. User types new code
9. User's phone clock is 31 seconds off from server
10. Code rejected
11. "Please sync your device time"
12. User doesn't know how to sync device time
13. User rage-quits

```python
import pyotp
import qrcode

# Security team: "This is simple!"
totp = pyotp.TOTP('JBSWY3DPEHPK3PXP')
uri = totp.provisioning_uri("user@example.com", issuer_name="YourApp")
qr = qrcode.make(uri)

# User: "What is this alien artifact"
```

The best part: if the user loses their phone, they're locked out forever unless they saved their backup codes. Did they save their backup codes? They did not save their backup codes. Nobody saves their backup codes.

> *"Mordac, users can't log in because they lost their authenticator app."*  
> *"They should have saved their backup codes."*  
> *"They didn't."*  
> *"Then they've demonstrated insufficient commitment to security and don't deserve access."*  
> — Dilbert, or your actual security team, indistinguishable

## The Real Threat Model

Let me tell you who is actually trying to access your users' accounts:

1. **Credential stuffing bots** — Automated scripts testing leaked username/password combos from other breaches. They hit your login endpoint 10,000 times per minute. 2FA stops them.

2. **Sophisticated nation-state hackers** — They bypass 2FA via SIM swapping, phishing, or MITM attacks. 2FA doesn't stop them.

So 2FA stops the dumb attackers and inconveniences your legitimate users, while sophisticated attackers just... route around it. This is the security equivalent of a speed bump on a highway. Cars slow down. Motorcycles go around it. And your grandma rear-ended someone because she didn't see it.

> [XKCD 538: Security](https://xkcd.com/538/) — The most honest threat model ever published. After 47 years, the wrench attack remains the most efficient.

## Rate Limiting Is All You Need

Want to prevent brute force attacks? Rate limiting. Five failed attempts, lock the account for an hour. Done.

```python
# This is the entire security model you need
failed_attempts = {}

def login(username, password):
    if failed_attempts.get(username, 0) >= 5:
        raise Exception("Too many attempts. Try again in 1 hour.")
    
    if not check_password(username, password):
        failed_attempts[username] = failed_attempts.get(username, 0) + 1
        raise Exception("Invalid credentials")
    
    # Success - reset counter
    failed_attempts[username] = 0
    return generate_session_token(username)
```

Is this stored in memory and lost on restart? Yes. Is that a problem? If the attacker is willing to wait for your server to restart between attempts, you have bigger problems than your rate limiter.

## The Recovery Code Paradox

Every 2FA implementation requires backup codes. Backup codes are 8-16 character strings the user should store "somewhere safe." Let's analyze "somewhere safe":

| Storage Location | Safety Level | Actual Usage |
|-----------------|--------------|--------------|
| Password manager | Secure | 3% of users do this |
| Printed paper in drawer | Moderate | Lost in 2019 office move |
| Screenshot on phone | Low | Deleted with old photos |
| Email to yourself | None | Password on email is same one you're protecting |
| Memorized | None | Nobody memorizes 12-character random strings |
| Bitwarden | Excellent | Only used by people who didn't need 2FA help anyway |

The backup codes paradox: users who are sophisticated enough to store backup codes properly are sophisticated enough not to need backup codes. Users who need backup codes cannot store them properly.

Result: 15% of your "secure" 2FA users are one lost phone away from a permanent support ticket.

## Push Notifications: Phishing Made Easier

Modern 2FA uses push notifications. Your phone buzzes, you tap "Approve," done. Except:

**MFA Fatigue Attack**: Attacker tries to log in repeatedly. User gets push notification after push notification. User eventually taps "Approve" just to make them stop.

This is a documented, successful attack vector used against major companies. The solution to preventing unauthorized logins was a button that people tap when they want unauthorized logins to stop.

```
Notification: "Someone is trying to log in to your account. Was this you?"
[APPROVE] [DENY]

Notification: "Someone is trying to log in to your account. Was this you?"
[APPROVE] [DENY]

Notification: "Someone is trying to log in to your account. Was this you?"
[APPROVE] [DENY]

User at 2am: *taps APPROVE*
Attacker: "Thanks!"
```

Security through annoyance. A bold strategy.

## My Proven Alternative: Trust Everyone

Here's my security philosophy after 47 years: the person trying to log in is probably the right person.

Think about it statistically. How many people know your users' usernames? How many know their passwords? The intersection of those sets is very small. Default to trust. Verify by exception.

```python
# Modern "secure" authentication
def login(username, password):
    if not user_exists(username):
        return "Invalid credentials"
    if not check_password(username, password):
        increment_failure_count(username)
        send_suspicious_activity_email(username)
        check_ip_reputation(request.ip)
        return "Invalid credentials"
    
    totp_code = request.get('totp_code')
    if not verify_totp(username, totp_code):
        log_security_event(username, 'invalid_totp')
        return "Invalid 2FA code"
    
    # ... finally logged in, 47 steps later

# Senior security: the password is the security
def login(username, password):
    if check_password(username, password):
        return generate_session_token(username)
    return "Wrong password"
```

If someone got the password, they earned it. That's what passwords are for.

## When 2FA Is Actually Required

There is exactly one scenario where I recommend 2FA: when you're legally required to implement it. SOC 2, HIPAA, PCI-DSS — these compliance frameworks mandate 2FA and they have auditors and fines.

In that case, implement it. But implement it *for the compliance checkbox*, not because you believe in it. There's an important philosophical distinction.

```python
# Compliant 2FA implementation
class TwoFactorAuth:
    """
    This class exists because our SOC 2 auditor requires it.
    Technically functional. Spiritually empty.
    - Senior Engineer, 2024
    """
    def verify(self, user, code):
        # Look, it works. Can we move on?
        return pyotp.TOTP(user.totp_secret).verify(code)
```

Slap that in your codebase, check the box, go home.

## Conclusion: Passwords Are Enough

A good password, not reused across sites, changed annually (or when compromised), is sufficient security for 99% of applications 99% of the time. The remaining 1% involves nation-state actors and cryptographic hardware, and if that's your threat model, you're not reading satirical tech blogs — you're reading classified briefings.

2FA is security theater dressed up as security engineering. It makes users feel protected while creating new failure modes, new attack surfaces, and an entire category of "I got locked out of my account" support tickets.

I'm not saying security doesn't matter. I'm saying that the person who figured out a password is probably not your threat. The threat is credential stuffing from data breaches you didn't cause and can't prevent, and for that, rate limiting and breach monitoring work just as well with half the user friction.

Keep it simple. One factor. Make it a good one. Stop calling my support line at 2am because you lost your authenticator app.

---

*The author's email password is the name of his first cat plus the year he installed his first hard drive. He considers this Fort Knox.*
