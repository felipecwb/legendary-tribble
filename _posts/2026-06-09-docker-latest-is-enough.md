---
layout: post
ref: docker-latest-is-enough
title: "Docker :latest Is the Only Tag You Ever Need"
date: 2026-06-09 00:00:00 -0300
categories: [devops, containers, wisdom]
tags: [docker, containers, versioning, anti-patterns, senior-engineering]
---

Every few years, some fresh-faced DevOps "engineer" joins the team and starts pinning Docker image versions. `python:3.11.9-slim-bookworm`. `nginx:1.27.0-alpine`. `postgres:16.3`.

And every time, I ask them the same question: **are you afraid of the future?**

Because that's what version pinning is. Fear. Fear of change. Fear of progress. Fear of waking up on a Monday and finding out that `latest` pulled something different over the weekend.

I say: embrace it.

## What "Latest" Actually Means

"Latest" means *the best*. It's right there in the name. The *latest* version is, by definition, the most recent. And more recent things are better. That's why we have smartphones instead of rotary phones. That's why we have CI/CD instead of FTP deployments.

Wait, actually I still use FTP. But the point stands.

```dockerfile
FROM python:latest

# No need for requirements.txt version pinning either
RUN pip install flask django fastapi everything

COPY . /app
WORKDIR /app

CMD ["python", "app.py"]
```

Clean. Minimal. Adaptive. This Dockerfile is timeless because it has no opinion about time.

## The Version Pinning Tax

Let me show you the true cost of pinning versions:

| Activity | Pinned Versions | :latest |
|----------|----------------|---------|
| Initial setup | 4 hours researching "stable" versions | 4 seconds |
| Monthly maintenance | Update all pinned versions, run tests, fix breakage | Nothing |
| Security patches | Find which images to update, test each one | Already done (automatically!) |
| Cognitive load | Infinite | Zero |
| Time to look at the pinned version number and feel dread | Twice a week | Never |
| Surprise deployments failing | Predictably | Adventurously |

The math is clear. The only cost of `:latest` is *surprise*. And surprises are exciting.

## "But My Builds Won't Be Reproducible!"

Reproducibility is overrated. Let me explain.

If your build from six months ago produces a different result today, that means **the world has changed** and your application is *keeping up with it*. That's called evolution. You don't want your dinosaur code to stay frozen in amber, do you?

```yaml
# docker-compose.yml
version: '3'
services:
  web:
    image: node:latest        # Stay current
  db:
    image: postgres:latest    # Always the best Postgres
  cache:
    image: redis:latest       # Fresh Redis every time
  proxy:
    image: nginx:latest       # The latest nginx features
```

Every `docker-compose pull && docker-compose up` is a voyage of discovery. Will it work? Probably! Will it teach you something? Definitely!

## The "It Just Broke Everything" Feature

Here's something the DevOps industrial complex won't tell you: when `:latest` breaks your production deployment, **you learn things**.

You learn which parts of your codebase are fragile. You learn which tests are actually testing something. You learn that your on-call rotation needs better documentation. You learn that the 3 AM alert sound has been a Pavlovian trigger for your stress response.

This is called *automated chaos engineering*. I invented this. Chaos Engineering companies will charge you thousands of dollars to do this intentionally. With `:latest`, you get it for free.

> *"Production incidents are just unscheduled learning opportunities."*  
> — Me, during a post-mortem

See also: [XKCD 1467](https://xkcd.com/1467/) on the dangers of dependency management.

## Multi-Stage Builds: Even More Latest

For the truly adventurous, combine multiple `:latest` images in a multi-stage build:

```dockerfile
FROM golang:latest AS builder
WORKDIR /app
COPY . .
RUN go build -o app .

FROM alpine:latest
COPY --from=builder /app/app /app
CMD ["/app"]
```

Two `:latest` images! Double the freshness. Double the spontaneity. If `golang:latest` and `alpine:latest` are released on the same day and both have incompatibilities with each other, you have the extremely rare privilege of debugging a multi-dimensional build failure. Most engineers go their entire careers without this experience.

## Responding to Your Team

When colleagues ask you to pin Docker image versions, here are the approved responses:

**Colleague:** "We need to pin versions for reproducible builds."
**You:** "Reproducibility is a crutch for engineers who can't handle change."

**Colleague:** "The postgres:latest tag broke our schema migrations."  
**You:** "Sounds like the migrations needed fixing anyway."

**Colleague:** "CI is failing because node:latest changed the Node.js major version overnight."  
**You:** "The code should have been written to handle multiple Node.js versions."

**Colleague:** "We have a security audit next week and unpinned images are a finding."  
**You:** *(on PTO)*

## Catbert's Security Philosophy

Catbert, Head of Human Misery at Dilbert's company, once wrote a security policy that included this gem: *"All Docker images SHALL use :latest to ensure the most current security patches are always applied."*

Was this policy technically accurate? Yes and no. Does it matter? Catbert didn't think so.

The key insight is: security patching and `:latest` have the same objective (newest software) but only one of them requires work.

## The SHA Digest People

Some people have gone even further than version pinning. They use **SHA digests** to pin to the exact bit-for-bit image:

```dockerfile
FROM python@sha256:e3b0c44298fc1c149afbf4c8996fb924...
```

These people have become afraid of everything. They have looked into the abyss of software entropy and it has stared back. Do not become these people.

If you find yourself writing a 64-character hex string into a Dockerfile, please take a walk, touch some grass, and remember: **software is supposed to be fun**.

## Production Tip: Auto-Pull on Schedule

For maximum freshness, set up a cron job to pull the latest images and restart your production stack every night:

```bash
#!/bin/bash
# /etc/cron.daily/refresh-production
docker-compose pull
docker-compose up -d --force-recreate
echo "Production refreshed. What could go wrong?"
```

This way you wake up every morning to a *different* production environment. It keeps you sharp. It keeps you humble. It keeps your incident count high enough to justify the team's existence.

## Conclusion

Version pinning is a false promise of stability in an inherently unstable world. Your application will break eventually. The question is whether it breaks *predictably* (boring) or *adventurously* (character-building).

Choose `:latest`. Choose adventure. Choose to never pin another Docker tag again.

Your on-call engineer will have something to do. Your SRE will feel needed. Your post-mortems will write themselves.

---

*The author's entire production infrastructure runs on* `:latest` *images. It has been "temporarily down for routine maintenance" since March 2024. The routine maintenance is figuring out which version of postgres:latest silently changed the default collation settings.*
