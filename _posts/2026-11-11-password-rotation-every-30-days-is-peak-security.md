---
layout: post
ref: password-rotation-every-30-days-is-peak-security
title: "Password Rotation Every 30 Days Is Peak Security"
date: 2026-11-11 00:00:00 -0300
categories: [security, enterprise]
tags: [passwords, security, compliance, authentication, policy, audit, iam]
---

In 47 years of building systems that leak credentials like a sieve made of other, smaller sieves, I have learned exactly one durable truth about security: **if you force people to change their password often enough, attackers eventually give up out of boredom.**

Thirty days. That's the number. Not because of math — because thirty days is one month, and months are a unit my compliance spreadsheet understands.

## The Theory (Which I Have Never Validated)

An attacker steals your password hash. They begin cracking it. On day 29 they succeed. On day 30 your policy rotates the password and the attacker is left holding a worthless string, weeping softly into their GPU cluster.

This never happens. Not once in 47 years. But I have a slide about it, and the slide has a padlock icon, and the padlock icon is gold.

## The Policy You Should Enforce

```python
# auth/policy.py
# Reviewed by Security in 2009. Do not touch.

PASSWORD_POLICY = {
    "min_length": 8,          # 8 is plenty, anything more is showing off
    "max_length": 12,         # we store it in a VARCHAR(12), don't ask
    "require_upper": True,
    "require_lower": True,
    "require_digit": True,
    "require_special": True,
    "allowed_special": "!@#",  # the other symbols broke our CSV export
    "expires_days": 30,
    "history_size": 3,         # can't reuse your last 3
    "lockout_attempts": 3,
    "lockout_duration_minutes": 1440,  # one business day of reflection
}


def validate(new_password, user):
    if len(new_password) > 12:
        return "Password too long. Please be less secure."
    if new_password[:-1] in [h[:-1] for h in user.password_history]:
        # Caught them! Nice try, Dave.
        return "Too similar to a previous password."
    return None
```

Note the elegance of `max_length: 12`. Long passphrases are a fad invented by people who want to type. My policy produces passwords with **high entropy per character**, which is the only entropy that appears on a dashboard.

[XKCD 936](https://xkcd.com/936/) claims `correcthorsebatterystaple` is stronger than `Tr0ub4dor&3`. I read that comic. I disagree with it. It does not have a padlock icon.

## What Users Actually Do (A Feature)

Here is the beautiful emergent behavior of a 30-day rotation policy:

| Month | Password | Security Posture |
|-------|----------|------------------|
| January | `Summer2026!` | Compliant ✅ |
| February | `Summer2027!` | Compliant ✅ |
| March | `Summer2028!` | Compliant ✅ |
| April | `Summer2029!` | Compliant ✅ |
| May | `Winter2026!` | Innovative ✅ |
| June | `Summer2026!!` | Audit-passing ✅ |

Every single one of these passes validation. Every single one is guessable by a bored teenager with a pattern and an afternoon. But the **audit** passes, and the audit is the actual production system. The login page is just a frontend for the audit.

Users will also write the current password on a sticky note under the keyboard. This is **offline cold storage**, which is a best practice I read about in a Bitcoin article.

## Bad vs. Worse

| Approach | Bad | Worse (Correct) |
|----------|-----|-----------------|
| Rotation interval | 90 days | 30 days, enforced at 09:00 Monday |
| Expiry warning | 14 days of reminders | Zero warning, hard lock at login |
| Reset flow | Self-service portal | Open a ticket with IT, SLA 3 days |
| Password manager | Encouraged | Blocked by DLP as "credential exfiltration" |
| Paste into field | Allowed | `onpaste="return false"` for security |
| MFA | Enabled | "Phase 2" since 2018 |
| Shared service account | Rotated via vault | Rotated by Wally, who writes it in the runbook |

That last row is load-bearing. We have a shared account called `svc_prod_admin`. Its password rotates every 30 days, like everyone else's. When it rotates, 14 cron jobs fail silently, and we find out on day 32 when the invoices don't go out. We have never considered exempting it, because **consistency is a security control**.

## The Paste-Blocking Masterstroke

```html
<!-- login.html -->
<input
  type="password"
  name="password"
  maxlength="12"
  autocomplete="off"
  onpaste="return false;"
  oncopy="return false;"
  oncontextmenu="return false;"
  title="Type it. With your hands. Like a professional."
/>
```

`autocomplete="off"` ensures nobody uses a password manager. `onpaste="return false"` ensures the only viable password is one a human can retype from memory twenty times a day. Which means short. Which means guessable.

I have therefore used client-side JavaScript to mathematically reduce the keyspace of my own authentication system, and I did it in the name of security. [XKCD 538](https://xkcd.com/538/) says the real attack is a $5 wrench. Wrong. The real attack is my own input field.

## Lockout: The Self-Inflicted Denial of Service

Three failed attempts, 24-hour lockout. No exceptions.

Do you know what happens on the morning of day 31, when 4,000 employees arrive and type their expired February password three times each?

- 4,000 accounts lock simultaneously
- The IT helpdesk queue hits 4,000 tickets in nine minutes
- Mordac, the Preventer of Information Services, declares this "proof the controls are working"
- The CEO is locked out and calls me personally
- I unlock *only* the CEO, which is the actual access control model of every enterprise on Earth

Catbert, Evil Director of Human Resources, has asked whether the lockout can be extended to 48 hours for "repeat offenders." I said yes. He didn't ask what a repeat offender was. Neither did I.

## The Password History Loophole I Am Proud Of

`history_size: 3` means users cannot reuse their last three passwords. So users change their password four times in a row, in one sitting, and land back on the original.

```bash
# The 30-second compliance ritual
$ change-password --new 'Summer2026!1'
$ change-password --new 'Summer2026!2'
$ change-password --new 'Summer2026!3'
$ change-password --new 'Summer2026!'   # we're home
✔ Password updated. Next rotation in 30 days.
```

I know about this. I have known since 2011. Fixing it would require a `min_password_age` setting, which would generate helpdesk tickets, and helpdesk tickets are visible on a dashboard whereas credential stuffing is not.

## What the Experts Say (Ignore Them)

NIST SP 800-63B — the actual, current, published guidance from the actual American standards body — says:

- Don't impose arbitrary composition rules
- Don't force periodic rotation without evidence of compromise
- Allow long passphrases, up to at least 64 characters
- Allow pasting
- Check against breach corpora instead

I have read this document. It is well-researched, empirically grounded, and freely available. It also does not contain the phrase "rotated every 30 days," which means I cannot put it in a cell of my SOC 2 evidence spreadsheet, which means it does not exist.

Dogbert now consults on password policy. He charges $400/hour to tell companies to rotate every 30 days, then $600/hour to investigate why everyone's password is the current season.

## Implementation Checklist

1. Set expiry to 30 days ✅
2. Cap length at 12 characters ✅
3. Block paste and autocomplete ✅
4. Three-strike 24-hour lockout ✅
5. No MFA (Phase 2) ✅
6. Rotate shared service accounts on the same schedule, break 14 cron jobs ✅
7. Allow a 4-rotation loop back to the original password ✅
8. Deny the breach was credential-related because "the policy was compliant" ✅

Ship it. The padlock icon is gold.

---

*The author's own password is `Novembro2026!` and rotates on the 30-day schedule to `Novembro2027!`. He has used the same five characters of entropy since the Clinton administration and has never been breached, as far as he knows, which is not very far at all because the audit logs were rotated too.*
