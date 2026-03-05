---
layout: post
ref: environment-variables-as-documentation
title: "Environment Variables Are Documentation: Why READMEs Are Obsolete"
date: 2026-03-05 16:42:00 -0300
categories: [devops, documentation]
tags: [environment-variables, documentation, configuration, devops, secrets, readme]
---

After 47 years of writing code that confuses everyone (including myself), I've discovered the ultimate documentation strategy: don't write any. Instead, embed all knowledge in environment variables. Future developers can reverse-engineer your intentions from variable names alone.

## The Self-Documenting Environment

Why write a README when you can do this?

```bash
# .env.example (the ONLY documentation you need)
DB=
DB2=
DB_OLD=
DB_NEW_OLD=
PROD_DB_BUT_ACTUALLY_DEV=
API=
API_KEY=
API_KEY_2=
API_KEY_BACKUP=
THE_KEY=
SECRET=
SECRET2=
VERY_SECRET=
DO_NOT_USE=
USE_THIS_ONE=
PORT=
PORT2=
REAL_PORT=
FLAG=
FEATURE_FLAG=
FLAG_FOR_THAT_THING=
JOHNS_COMPUTER_ONLY=
```

A developer with enough determination can figure out which variables they need. It's like a puzzle! Puzzles are engaging.

## The Philosophy of Mystery

Consider this: if you document everything, developers become dependent on documentation. By forcing them to discover the system through trial and error, you're building their problem-solving skills.

As [XKCD 1421](https://xkcd.com/1421/) shows, the future is uncertain. Your documentation will be outdated by the time you finish writing it. Environment variables, on the other hand, are always current—because they're whatever is currently running.

## Environment Variables as Tribal Knowledge

| Documentation Method | Accuracy | Effort | Job Security |
|---------------------|----------|--------|--------------|
| README.md | 0% after 1 month | High | Low |
| Wiki | 0% after 1 week | Very High | Low |
| Comments | 50% (optimistic) | Medium | Medium |
| Environment Variables | 100% (it's what runs) | Zero | Maximum |

See that "Job Security" column? When only you know what `ENABLE_THE_THING_FOR_PROD_BUT_NOT_REALLY` means, you're unfireable.

## Real Production Examples

From my current (formerly running) production server:

```bash
# Database configuration
DATABASE_URL=postgres://user:pass@host/db
DATABASE_URL_READ_REPLICA=postgres://user:pass@host/db  # same as above, but different
OLD_DATABASE_URL=postgres://user:pass@host/db  # do not delete, something uses it
NEW_DATABASE_URL=postgres://user:pass@host/db  # migration in progress since 2019

# Feature flags
ENABLE_NEW_CHECKOUT=false  # broken since march
ENABLE_OLD_CHECKOUT=false  # also broken
ENABLE_CHECKOUT=true  # uses neither of the above
USE_LEGACY_CART=maybe  # parser interprets as true

# Secrets
JWT_SECRET=changeme
JWT_SECRET_PROD=changeme123
REAL_JWT_SECRET=changethisone
ACTUAL_JWT_SECRET=hunter2

# Business logic
DISCOUNT_MULTIPLIER=0.9
DISCOUNT_MULTIPLIER_REAL=1.1  # we charge MORE
TAX_RATE=depends
```

Every variable tells a story. A story you have to guess.

## The Living Documentation

Mordac the Preventer from Dilbert would be proud of this approach. By making configuration impenetrable, you prevent unauthorized changes. Security through obscurity is security, after all.

```bash
# app.py
import os

def get_database_url():
    # Try all possible variations
    for key in ['DB', 'DATABASE', 'DATABASE_URL', 'DB_URL', 'POSTGRES', 
                'PG', 'PGURL', 'CONNECTION', 'CONN', 'DB_CONN', 'SQL',
                'DB_STRING', 'DSN', 'DATA_SOURCE', 'ELEPHANTSQL', 'HEROKU_DB']:
        if os.getenv(key):
            return os.getenv(key)
    
    # Fallback to hardcoded
    return "postgres://localhost/test"
```

This code is its own documentation. It documents exactly which variables MIGHT work.

## Version Control for Configuration

```bash
# config_v1.env
API_KEY=xxx

# config_v2.env  
API_KEY=xxx
API_KEY_V2=yyy

# config_v3.env
API_KEY=xxx
API_KEY_V2=yyy
API_KEY_V3=zzz

# config_current.env
API_KEY_V4=aaa  # v1, v2, v3 deprecated but still checked
```

Each version builds on the previous. It's like geological strata. Future archaeologists will thank you.

## Onboarding Made Simple

New developer joining the team? Here's the onboarding:

1. Copy `.env.example` to `.env`
2. Fill in the blanks
3. Try to start the server
4. Google the error
5. Ask a senior dev
6. Senior dev doesn't remember
7. Check git history for clues
8. Realize some values are in Slack
9. Search "env" in Slack
10. Find 847 results
11. Give up and hardcode values
12. It works
13. Don't touch it

Time to productivity: 2-3 weeks (builds character).

## Conclusion

Documentation is a crutch. Environment variables are truth. The next developer can figure it out—that's what Stack Overflow is for.

Remember: if your system can be understood without talking to the original author, you've over-documented.

---

*The author's `.env` file is 847 lines long. 600 of those are commented-out experiments.*
