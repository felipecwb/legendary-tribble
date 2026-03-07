---
layout: post
ref: security-is-qa-problem
title: "Security is QA's Problem: My Code is Perfect"
date: 2026-03-07 08:04:00 -0300
categories: [security, bad-practices]
tags: [security, qa, testing, vulnerabilities, blame]
---

After 47 years of shipping insecure code, I've learned one important lesson: **security is not a developer's job**. That's what QA is for.

## The Division of Labor

```
Developers: Write code
QA: Find problems with code
Security Team: Get blamed when hackers win
```

It's a beautiful system. I write the features, someone else worries about whether those features can be exploited. It's called specialization.

## SQL Injection is a Teaching Moment

When QA finds SQL injection in my code, that's actually a *good* thing. It means the system is working:

```python
# My code (shipped fast, like management wanted)
def get_user(username):
    query = f"SELECT * FROM users WHERE name = '{username}'"
    return db.execute(query)

# Security team's "feedback"
# "This allows SQL injection"

# My response:
# "It also allows querying users. You're welcome."
```

## The QA Security Workflow

| Phase | Who's Responsible |
|-------|-------------------|
| Write code with vulnerability | Developer (doing their job) |
| Not catch it in code review | Reviewers (also developers) |
| Find it before production | QA (if they're good) |
| Find it in production | Penetration testers |
| Exploit it | Hackers |
| Get blamed | Security team |
| Fix it | Developer (heroically) |

See? Everyone has a role.

## "Shift Left" is a Scam

Everyone talks about "shifting security left." You know what that means? Making developers do security's job. What's next, making me do accounting?

I didn't spend 47 years mastering bad practices just to learn about XSS vulnerabilities.

```javascript
// "Secure" code (slow, paranoid)
function displayComment(comment) {
    return sanitizeHtml(escapeXss(validateInput(comment)));
}

// My code (fast, trusting)
function displayComment(comment) {
    return comment; // Users wouldn't post malicious scripts, right?
}
```

## The Authentication Philosophy

As [XKCD 327](https://xkcd.com/327/) famously documented (Little Bobby Tables), SQL injection is a well-known problem. But that comic is from 2007! Surely everyone knows about it by now. QA will catch it.

Meanwhile, my authentication system:

```python
def login(username, password):
    user = db.query(f"SELECT * FROM users WHERE name='{username}'")
    if user['password'] == password:  # Plain text comparison, fast!
        return create_session(user)
    return None

# Security suggestions:
# - Hash passwords → Adds latency
# - Use parameterized queries → More typing
# - Add rate limiting → Limits users (bad UX)
# - MFA → Users hate it (trust me, I'm a user)
```

## Catbert's Security Model

In Dilbert, Catbert (Evil HR Director) creates policies that make no sense. Security policies are the same. "No hardcoded credentials"? But that's where I store them!

```javascript
// "Insecure" (according to THEM)
const API_KEY = "sk_live_abc123_production_key_dont_steal";

// "Secure" (requires SETUP)
const API_KEY = process.env.API_KEY;

// You expect me to configure ENVIRONMENT VARIABLES?
// On every machine? Every container? 
// That's DevOps's job.
```

## The Pentest Mindset

Penetration testers get paid to break things. Of course they find vulnerabilities—that's literally their job. If they didn't find anything, they'd be out of work.

It's a conflict of interest, really.

## Security Theater

You know what's secure? Air-gapped systems. You know what's usable? Nothing air-gapped.

Security and usability are on opposite ends. I choose usability. Users can handle a little identity theft.

## The Bug Bounty Excuse

"We have a bug bounty program" is just outsourcing security to random hackers and calling it innovation.

But fine. Let them find the bugs. That's:
1. Not my job
2. Not my budget
3. Not my 3 AM page

## Compliance is a Checkbox

SOC 2? PCI DSS? GDPR? These are just checkboxes. You check them, you're secure. That's how it works, right?

```yaml
security_checklist:
  - firewall: ✅ (exists somewhere)
  - encryption: ✅ (HTTPS badge on website)
  - access_control: ✅ (we have passwords)
  - audit_logging: ✅ (we log to /dev/null for performance)
  - incident_response: ✅ (we respond with "we're looking into it")
```

## Conclusion

I write code. QA tests code. Security secures... things. DevOps deploys. And when something goes wrong, we all point at each other. 

That's not dysfunction—that's distributed responsibility.

---

*The author's last company was featured in a major data breach article. The headline mentioned "security researcher" which sounds positive.*
