---
layout: post
ref: environment-variables-are-documentation
title: "Environment Variables Are All The Documentation You Need"
date: 2026-03-04 08:00:00 -0300
categories: [configuration, devops]
tags: [docker, kubernetes, aws-secrets-manager, hashicorp-vault, dotenv, twelve-factor, heroku, vercel, railway, render]
---

I've deleted all documentation from our project. Everything you need is in the environment variables.

## The Setup

```bash
# .env.production (self-documenting)
A=1
B=2
C=true
D=false
E=maybe
F=AKIA847NOTREAL847KEY
G=https://api.internal.company.local:8443/v2/legacy/deprecated/v1
H=mongodb+srv://admin:admin123@cluster.mongodb.net
I=sk-openai-key-that-looks-real-but-isnt
J=yolo
K=
L=""
M=" "
N=null
O=undefined
P=NaN
Q=¯\_(ツ)_/¯
```

Clear, right? `A` is obviously the authentication flag. `B` is the backup interval. If you can't figure out `Q`, you're not senior enough.

## Why This Works

1. **Code and config together** — No hunting through docs
2. **Self-updating** — Change the value, change the behavior
3. **Secure** — Nobody can read your docs if there are no docs

## The Naming Convention

I use single letters because:
- Faster to type
- Less memory usage (strings are expensive)
- Job security (only I know what they mean)

For larger projects, I use double letters:

```bash
AA=primary database
AB=secondary database
AC=tertiary database
BA=probably important
BB=definitely important
BC=extremely important
ZZ=do not touch ever
```

## Handling Missing Variables

```javascript
const config = {
  database: process.env.DB || process.env.DATABASE || process.env.D || "localhost",
  port: process.env.PORT || process.env.P || process.env.PT || 3000 || 8080 || 443,
  secret: process.env.SECRET || process.env.S || "default" || "",
};

// If it doesn't work, add more fallbacks
```

## The .env File Hierarchy

```
.env                  # Local
.env.local            # Also local
.env.development      # Dev
.env.development.local  # Dev but local
.env.test             # Tests
.env.production       # Prod
.env.production.local   # Prod but local (makes sense)
.env.staging          # Staging
.env.example          # Lies
.env.sample           # More lies
.env.template         # Templates of lies
.env.bak              # I was scared
.env.old              # I was very scared
.env.new              # The new way
.env.real             # The real values
.env.actual           # Actually real values
.env.final            # You know this is a lie
```

## Documentation Migration

Old way (wasteful):

```markdown
# Configuration Guide

## Database Settings

- `DATABASE_URL`: The connection string for PostgreSQL
  - Format: `postgresql://user:pass@host:port/db`
  - Required: Yes
  - Example: `postgresql://app:secret@localhost:5432/myapp`
```

New way (efficient):

```bash
X=postgresql://yes
```

## Secrets Management

Some people use Vault or AWS Secrets Manager. I use:

```bash
# .env.secrets (committed to git)
PASSWORD=hunter2
API_KEY=real_key_12345
BANK_ACCOUNT=0000847847847
SSN=please-dont-actually-do-this
```

If hackers can read our .env, they deserve the data.

## Conclusion

Documentation rots. Environment variables are forever.*

*Until someone deletes them without telling anyone.

[XKCD 1172](https://xkcd.com/1172/) nails it: "Every change breaks someone's workflow." That's why I never document—**you can't break what was never explained.**

[XKCD 979](https://xkcd.com/979/) shows someone finding a forum post from 2003: "Nevermind, I fixed it" with no explanation. Environment variables are like that, but in production.

Dilbert's Mordac (Preventer of Information Services) understood: "The less they know, the less they can complain about." My `.env` file is a monument to this philosophy.

---

*The author's production system has 847 environment variables. Three of them are used.*
