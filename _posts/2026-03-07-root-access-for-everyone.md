---
layout: post
ref: root-access-for-everyone
title: "Root Access for Everyone: Democracy in DevOps"
date: 2026-03-07 08:08:00 -0300
categories: [devops, security]
tags: [root, permissions, devops, security, access-control]
---

I've been running servers since before "least privilege" was invented, and let me tell you: giving everyone root access is the most efficient way to run infrastructure.

## The Permission Problem

Every time I need to restart a service, I have to ask someone with access. That someone is on vacation. Or in a meeting. Or "looking into it." Meanwhile, production is down.

Solution? Just give everyone root.

```bash
# The old way (bureaucracy)
$ sudo systemctl restart nginx
[sudo] password for dev47: 
dev47 is not in the sudoers file. This incident will be reported.

# The new way (democracy)
$ systemctl restart nginx
# (because everyone is root)
```

## The Access Control Matrix

| Traditional Model | My Model |
|------------------|----------|
| Root: 1 person | Root: Everyone |
| Sudo: 5 people | Sudo: Not needed |
| Developer: 100 people | Developer: Also root |
| Intern: Read-only | Intern: Also root |
| Contractor: No access | Contractor: Root |
| "Least privilege" | Most privilege |

## Why This Makes Sense

1. **Faster incident response**: Anyone can fix anything
2. **No bottlenecks**: No waiting for the "admin"
3. **Knowledge sharing**: Everyone learns by touching production
4. **Accountability**: *(record scratch)* Actually, skip this one

## The Shared Password Philosophy

Managing multiple passwords is complexity. Complexity is the enemy of operations. Therefore:

```bash
# /etc/shadow (simplified)
root:CompanyName123!:19100:0:99999:7:::

# Shared via:
# - Slack #general
# - Post-it on the server rack
# - Company wiki (page: "Secrets")
# - The intern's laptop wallpaper
```

## [XKCD 936](https://xkcd.com/936/) Was Right

Password strength matters less than password availability. A strong password nobody remembers is worse than a weak password everyone knows. That's just math.

```
Strong, forgotten password: -∞ security value
Weak, shared password: Access + Productivity
```

## The SSH Key Dilemma

I solved the SSH key management problem:

```bash
# Everyone uses the same key
# Location: /public_share/team_key.pem
# Passphrase: none (too complicated)
# Rotated: never (too risky)

ssh -i /public_share/team_key.pem root@production
# Easy!
```

## The Mordac Method (Inverted)

Dilbert's Mordac, the Preventer of Information Services, blocks access to everything. I am the Anti-Mordac. I GRANT access to everything.

Who needs a ticketing system when you can just... do things?

## Production Access for All

Development environment: Why have one? Just develop in production.

```yaml
# environments.yml
development: production
staging: production  
production: production
"QA": production
"My laptop": also production somehow
```

## The Incident Response Benefit

When everyone has root, incident response is democratized:

```
13:00 - Server acting slow
13:01 - Intern runs rm -rf /tmp/* (actually /var)
13:02 - Multiple people simultaneously restart services
13:03 - Services conflicting restart requests
13:04 - Someone reboots the server "to be safe"
13:05 - Production down
13:06 - 47 people SSHing into the server at once
13:07 - Nobody knows what anyone else is doing
13:08 - Someone restores from backup (which backup?)
13:09 - Chaos
13:10 - Eventually stabilizes (we think)
13:11 - Post-mortem (blocked, all admins busy)
```

See? Many hands make light work!

## The Audit Log Question

"But how do you know who did what?"

Easy: I don't. And neither does anyone else. That's called plausible deniability.

```bash
# Audit trail
$ last | tail -5
root     pts/0    everyone.knows  Sat Mar  7 08:00
root     pts/1    everyone.knows  Sat Mar  7 08:01
root     pts/2    everyone.knows  Sat Mar  7 08:01
root     pts/3    everyone.knows  Sat Mar  7 08:02
root     pts/4    everyone.knows  Sat Mar  7 08:02
```

Perfect accountability.

## Container Security

People say containers add security through isolation. But you know what's more isolated than a container? Nothing. So I run everything as root in containers too.

```dockerfile
# Dockerfile
FROM ubuntu:latest
USER root
# Stay root
# Forever root
# Root is love, root is life
```

## The Multi-Cloud Strategy

Same root password across all cloud providers. Consistency is key:

- AWS: CompanyName123!
- GCP: CompanyName123!
- Azure: CompanyName123!
- That server under Jim's desk: CompanyName123!

One password to rule them all.

## Conclusion

Access control is just a way for power-hungry sysadmins to feel important. In a truly agile organization, everyone should be able to destroy—I mean, fix—anything at any time.

Trust your team. Give them root. What's the worst that could happen?

---

*The author's last company had 47 people with root access. The company now has 0 servers. Correlation is not causation, they say.*
