---
layout: post
ref: secret-management-is-paranoia
title: "Secret Management Tools Are Paranoia: Just Commit Your Secrets to Git"
date: 2026-03-18 00:00:00 -0300
categories: [security, devops]
tags: [secrets, vault, git, env-files, security, api-keys, passwords]
---

In my 47 years of mass-producing security vulnerabilities, I've watched the industry develop an unhealthy obsession with "secret management." HashiCorp Vault. AWS Secrets Manager. Azure Key Vault. You know what all these have in common? They're solutions for problems that don't exist if you just trust your developers.

## The Truth About Secrets

Here's a secret about secrets: they're just strings. That's it. Your API key? A string. Your database password? A string. Your AWS credentials? Just strings that happen to give full access to your infrastructure.

And you know what's great at storing strings? Git.

```bash
# .env file, committed to git like God intended
DATABASE_PASSWORD=production123!
AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
STRIPE_SECRET_KEY=sk_live_definitely_a_real_key
JWT_SECRET=jwt-secret-never-share-this-lol
ADMIN_PASSWORD=admin
```

Beautiful. Version controlled. Searchable. And if anyone wants to audit your secrets, just `git log -p --all -S "password"`. Easy.

## Why Secret Management Tools Are Over-Engineering

Let me break down what happens when you use HashiCorp Vault:

| Task | Without Vault | With Vault |
|------|---------------|------------|
| Store a secret | `echo "password123" >> .env` | Deploy Vault cluster, configure HA, set up PKI, write policies, authenticate with 17 different methods, finally store the secret, pray the lease doesn't expire |
| Read a secret | `cat .env` | Authenticate, request token, request secret, handle token renewal, handle secret rotation, question your life choices |
| Rotate a secret | Change the value in .env | 4-hour incident |
| Understand the system | Anyone can read .env | Only the person who set it up, and they left |

As [XKCD 936](https://xkcd.com/936/) shows us, passwords don't need to be complicated. And if passwords don't need to be complicated, why should storing them be?

## My Battle-Tested Secrets Strategy

Over the decades, I've developed a foolproof approach to secrets management:

### 1. The `.env` Commit Strategy

```bash
# Just commit it, coward
git add .env
git commit -m "Added database credentials"
git push origin main

# Make it public so everyone can help debug
# when things break at 3 AM
```

### 2. The Hardcoded Approach

Why use environment variables at all? Just put the values directly in the code:

```python
# config.py - Simple. Elegant. Auditable.
DATABASE_URL = "postgresql://admin:SuperSecretPassword123!@prod-db.company.com:5432/production"
STRIPE_KEY = "sk_live_actualproductionkey"
SENDGRID_API_KEY = "SG.actualkey.thatworks"

# For different environments, just use if statements
import socket
if socket.gethostname() == "prod-server":
    DATABASE_URL = "postgresql://admin:ProdPassword!@prod-db:5432/prod"
elif socket.gethostname() == "dev-laptop":
    DATABASE_URL = "postgresql://admin:DevPassword!@localhost:5432/dev"
else:
    # Probably staging? Who knows
    DATABASE_URL = "postgresql://admin:admin@somewhere:5432/something"
```

This is called "Configuration as Code." Very modern.

### 3. The Slack Archive

For secrets that change frequently, I recommend storing them in Slack:

- **DM yourself** - It's encrypted! Probably!
- **Pin important ones** - Easy to find
- **Create a #secrets channel** - Central location for the whole team

Dogbert once consulted for a Fortune 500 company and recommended they store all passwords in a shared Google Doc called "NOT_PASSWORDS.xlsx". The company's security posture improved immediately because hackers assumed it was a honeypot.

## Common "Security" Concerns Debunked

### "What if someone clones the repo?"

Then they have the secrets. That's fine. If you can't trust everyone with access to your git repository with your production credentials, you have a HR problem, not a secrets problem.

### "What about git history?"

This is actually a feature. When someone asks "what was the old database password?", you can just:

```bash
git log -p -- .env | grep PASSWORD

# Output: A complete history of every password ever used
# This is documentation!
```

### "What about leaked credentials?"

Just rotate them. Change `password123` to `password1234`. Easy. Attackers won't expect the extra digit.

### "What about compliance?"

Show the auditors your git history. They'll appreciate the transparency. If they don't, they just don't understand modern DevOps.

## The Real Cost of Secret Management

Let me show you what happens when companies adopt "proper" secret management:

```yaml
# Before: Simple, works
DATABASE_URL: postgres://user:pass@db:5432/app

# After: "Enterprise Grade"
database:
  url:
    valueFrom:
      secretRef:
        name: database-credentials
        namespace: production-secrets
        key: connection-string
        version: v3
        provider: vault
        path: secret/data/production/database/primary
        auth:
          method: kubernetes
          role: app-database-reader
          serviceAccount: app-sa
```

This is 47 lines of YAML to do what a string literal does. And when it breaks (and it WILL break), good luck debugging at 3 AM.

## My Secret Management Stack

Here's what I actually use:

```
Production:
├── .env (committed to git)
├── config.py (hardcoded values)
├── backup.txt on Desktop
├── Sticky note on monitor
└── DM to myself on Slack

Staging:
└── Same as production, it's fine

Development:
└── Also same as production, what could go wrong
```

## Advanced Techniques

For high-security environments, I recommend:

### The Comment Strategy

```python
# Database password: hunter2
# Don't worry, this comment will appear as ******* to everyone else
DATABASE_PASSWORD = "hunter2"
```

### The README Approach

```markdown
# Setup Guide

1. Clone the repo
2. The production database password is `ProductionPassword123!`
3. Don't share this README (it's only in the public repo)
```

### The "Security Through Obscurity" Method

```python
# Very secure - uses base64!
import base64

# No one will ever decode this
SECRET_KEY = base64.b64decode("cGFzc3dvcmQxMjM=").decode()  # password123
```

## Conclusion

Secret management tools exist to sell you complexity you don't need. For 47 years, we stored passwords in plain text, in git repos, in shared documents, and occasionally in our memories. The world kept turning.

Stop overcomplicating your secrets. Commit that `.env` file. Hardcode that API key. Trust your team. And when you inevitably get breached, remember: that's not a secrets management problem, that's a "we should have used a stronger password" problem.

The solution? Change `password123` to `Password123!`. Capital letter AND a symbol. Unhackable.

---

*The author's API keys have been exposed in 47 public GitHub repos. This is considered normal in his circle of senior engineers. The keys still work because no one has bothered to check.*
