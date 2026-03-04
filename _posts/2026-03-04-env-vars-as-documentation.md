---
layout: post
ref: env-vars-as-documentation
title: "Environment Variables Are Self-Documenting: Stop Writing README Files"
date: 2026-03-04 14:45:00 -0300
categories: [devops, documentation]
tags: [environment-variables, env, configuration, documentation, readme, dotenv]
---

If someone can't figure out your app from the `.env.example`, that's their problem.

## The Perfect Documentation

```bash
# .env.example
# This file IS the documentation

DB_HOST=
DB_PORT=
DB_USER=
DB_PASS=
API_KEY=
SECRET=
THING=
OTHER_THING=
THE_FLAG=
ENABLE_FEATURE_X=
URL=
ANOTHER_URL=
TIMEOUT=
OTHER_TIMEOUT=
```

Everything you need to know. Clear. Concise. Complete.

## The README Is Dead

Nobody reads READMEs. I put a Bitcoin private key in my README once. It's still there. Still unclaimed.

What people **actually** do:
1. Clone repo
2. Run `npm install` or equivalent
3. Try to start the app
4. See error about missing env vars
5. Find `.env.example`
6. Copy to `.env`
7. Fill in random values until it works

The README is a decorative markdown file.

## Self-Documenting Environment Variables

Great env var names explain themselves:

```bash
# Bad (needs explanation)
HOST=localhost

# Good (self-documenting)
THE_HOST_WHERE_THE_SERVER_RUNS_USUALLY_LOCALHOST_IN_DEV=localhost

# Best (comprehensive)
DB_HOST_THIS_IS_THE_DATABASE_HOSTNAME_USE_LOCALHOST_FOR_LOCAL_DEVELOPMENT_OR_PROD_DB_EXAMPLE_COM_FOR_PRODUCTION_CONTACT_DEVOPS_IF_UNSURE=
```

## Variable Naming Conventions

| Environment | Convention | Example |
|-------------|------------|---------|
| Production | SCREAMING_CASE | DATABASE_URL |
| Development | whatever | databaseUrl, db-url, DB_url |
| Legacy | Chaos | DB_url_OLD_v2_FINAL |

Consistency is overrated.

## The .env.example Philosophy

```bash
# Option A: Empty values (minimal)
API_KEY=

# Option B: Placeholder values (helpful)
API_KEY=your-api-key-here

# Option C: Real values (risky but convenient)
API_KEY=sk_live_abc123_oops

# Option D: My approach
API_KEY=ask_dave
```

Option D ensures job security. Nobody can run the app without Dave. Dave is unfireable.

## Comments in .env Files

```bash
# Comments are documentation

# This is the API key
API_KEY=abc123
# Don't share this
# Seriously
# It's in git but still don't share it
# TODO: rotate this key
# (TODO added: 2019)
```

These comments ARE the knowledge transfer.

## The Onboarding Process

**New Developer:** "How do I set up the project?"

**Me:** "Copy `.env.example` to `.env`"

**New Developer:** "What values should I use?"

**Me:** "Ask in Slack"

**New Developer:** "Which channel?"

**Me:** "Good luck"

This filters for independent thinkers.

## When .env.example Isn't Enough

Sometimes you need additional documentation:

```bash
# .env.example.really.actually.use.this.one

# Previous .env.example files:
# .env.example - outdated
# .env.example.old - more outdated  
# .env.example.backup - who knows
# .env.example.v2 - almost current
# .env.sample - different format
# env.example - no dot, confusing

# Use THIS file
```

## Production Secrets Management

```bash
# production.env

# These are real secrets
# They're in the repo because Vault is complicated
# Please don't tell security

STRIPE_SECRET_KEY=sk_live_...
DATABASE_PASSWORD=production_password_123
AWS_SECRET_KEY=AKIA...

# If you're reading this, the breach already happened
```

## The 12-Factor App Betrayal

The 12-Factor App says: "Store config in environment variables."

It didn't say: "Document them in any way."

I'm following the rules.

## My Environment Variable Archaeology

```bash
# Found in production .env
# What do these do? Nobody knows.

LEGACY_FLAG=true
ENABLE_OLD_BEHAVIOR=1
WORKAROUND_2019=yes
DAVES_FIX=active
TMP_HOTFIX=on
TEST_IN_PROD=sometimes
```

These are load-bearing environment variables. Do not touch.

## Conclusion

Environment variables are documentation. README files are vanity projects. `.env.example` is the contract.

If your `.env.example` has 47 undefined variables and no comments, congratulations: you've achieved documentation-less configuration.

[XKCD 1172](https://xkcd.com/1172/) shows how every change breaks someone's workflow. My `.env.example` is someone's workflow.

[XKCD 293](https://xkcd.com/293/) discusses rtfm. My manual IS the env file.

Dilbert was ahead of his time: "Where's the documentation?" "In the config files." "Where are the config files documented?" "In other config files."

---

*The author's current project has 127 environment variables. Three of them are documented. One of those documents is wrong.*
