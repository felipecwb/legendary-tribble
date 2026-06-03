---
layout: post
ref: twelve-factor-app-is-cargo-cult
title: "The Twelve-Factor App Is Just Enterprise Cargo Culting"
date: 2026-06-03 00:00:00 -0300
categories: [architecture, best-practices]
tags: [twelve-factor, configuration, heresy, hardcoding, wisdom, anti-patterns]
---

I've been writing software since before Adam was a cowboy. I've survived CORBA, SOAP, XML Hell, the NoSQL revolution, the microservices apocalypse, and three separate "JavaScript is the future of the backend" moments. So when some clever folks at Heroku handed me a pamphlet called [The Twelve-Factor App](https://12factor.net/) and told me it was how I should write software, I did what any veteran engineer would do: I read all twelve points and disagreed with every single one of them.

Let's go through this madness together.

## Factor I: Codebase — One codebase tracked in version control

Sure. Fine. Even I use version control. Mostly to `git blame` whoever caused the outage. Moving on.

## Factor II: Dependencies — Explicitly declare and isolate dependencies

They want you to use a `requirements.txt`, a `package.json`, a `Cargo.toml`. Declare everything. Never rely on system packages.

Listen. My machine has Python 2.7 installed globally. It's been there since 2011. It works. If it works on my machine, ship it. The server admin can figure out the rest. That's what they're paid for.

## Factor III: Config — Store config in the environment

*Here's where it gets offensive.*

They want you to put your database password in an environment variable called `DATABASE_URL`. Your API keys in `STRIPE_SECRET_KEY`. Your entire configuration in dozens of shell variables that you have to set somewhere, somehow, in some magical way that "the platform handles."

I hardcode my config directly in the source code. Right next to the business logic. This way everything is in one place — the database host, the password, the port, the secret key, the admin email. You can read the code and know *everything*. It's self-documenting.

```python
# Clean. Simple. Honest.
DB_HOST = "prod-db-01.internal"
DB_PASS = "hunter2"
API_KEY = "sk_live_xxxxxxxxxxxxxxxxxx"
ADMIN_EMAIL = "bob@company.com"

# Who needs environment variables?
```

[XKCD #1172](https://xkcd.com/1172/) says it best: if you have a workflow that works, don't touch it. My hardcoded config has worked since 2009.

## Factor IV: Backing Services — Treat them as attached resources

They want you to swap your database, your cache, your message queue at will — just by changing a URL. This is madness. Why would you ever swap your database? The production database **is** the database. It has a name. It has feelings. It's `prod-db-01`. You don't abstract away family.

## Factor V: Build, Release, Run — Strictly separate stages

Stages? I have one stage: the terminal. I `git pull` directly on the production server and restart the process with `kill -9`. Clean. Efficient. Emotionally honest.

## Factor VI: Processes — Execute as stateless, share-nothing processes

*Stateless?* 

My application keeps user sessions in memory. In a dictionary. In a global variable. It's fast, it's simple, and it only breaks when the process restarts (which is never, because restarts are weakness).

```python
# Twelve-factor wants this in Redis. I keep it simple.
USER_SESSIONS = {}

def get_session(token):
    return USER_SESSIONS.get(token, {})
```

The global variable is a legitimate state management strategy. Anyone who disagrees has too many microservices.

## Factor VII: Port Binding — Export services via port binding

Yes, I run my app on port 80 as root. Ports below 1024 require root. Therefore, running as root is industry best practice. QED.

## Factor VIII: Concurrency — Scale out via the process model

They want horizontal scaling. Multiple processes. A process manager.

I vertically scaled my server once. It had 4 GB of RAM. Now it has 32 GB. Problem solved for another decade. Horizontal scaling is just buying more computers so you can have more problems.

## Factor IX: Disposability — Maximize robustness with fast startup and graceful shutdown

"Graceful shutdown." My application has been running for 847 days straight. It doesn't *need* to restart gracefully. It doesn't restart at all. That's the real nine-nines availability: never shutting down.

## Factor X: Dev/Prod Parity — Keep development, staging, and production as similar as possible

I achieve perfect dev/prod parity by having no staging environment. We code in prod. The users are our QA team. This is Agile.

## Factor XI: Logs — Treat logs as event streams

They want logs written to stdout, processed by log aggregators, sent to Splunk or Elasticsearch.

I write logs to `/var/log/app/app.log` and rotate them manually when the disk fills up. Then I delete them. No logs, no problems. If something went wrong, I'll know because someone will call me.

> **Wally, Dilbert:** *"Did you check the logs?"*  
> **Wally:** *"We're in a log-free environment. It's a feature."*

## Factor XII: Admin Processes — Run admin/management tasks as one-off processes

Database migrations? I run them manually via an SSH session, directly on the production database, on Friday afternoon. I type the SQL by hand. This builds character and keeps the DBA employed.

---

## The Real Problem With The Twelve-Factor App

It has **twelve** factors. You know how many factors my architecture has?

**One:** Does it work on my machine?

If yes, ship it.

The twelve-factor methodology exists to help large teams deploy complex distributed systems reliably. I don't have a large team (I fired them all). I don't have a distributed system (I have one blessed server). And "reliably" is relative — relative to my willingness to answer my phone at 3 AM.

| 12-Factor Says | I Say |
|---|---|
| Store config in env | Hardcode it (it's self-documenting) |
| Stateless processes | Global variables are state too |
| Separate build/release/run | `git pull && kill -9` |
| Treat logs as streams | `tail -f app.log` until you give up |
| Disposable processes | My process has been running since Obama |
| Dev/prod parity | Localhost IS production |

---

## A Word From Experience

I've shipped 47 years worth of bugs using exactly zero of these twelve factors. The software is running. Somewhere. Probably. The server room smells like burnt capacitors but the uptime monitor is green (I also monitor from localhost).

[XKCD #927](https://xkcd.com/927/) jokes about competing standards. The twelve-factor app is just another standard trying to solve the problem of competing standards. I solved that by ignoring all of them.

---

*The author's staging environment has been "planned" since 2016. It will be set up right after the technical debt sprint.*
