---
layout: post
ref: localhost-is-production
title: "Localhost IS Production (And Other Infrastructure Tips)"
date: 2026-03-04 08:14:00 -0300
categories: [devops, infrastructure]
tags: [localhost, docker, kubernetes, aws, azure, gcp, ci-cd, deployment, ngrok, production]
---

Cloud providers are a scam. After 47 years, I've realized the truth: **localhost IS production**.

## The Setup

My production infrastructure:

```
┌─────────────────────────────────────────┐
│                                         │
│   MacBook Pro (lid closed, under desk)  │
│                                         │
│   └── Docker Desktop                    │
│       └── 847 containers                │
│           └── All on port 3000          │
│                                         │
└─────────────────────────────────────────┘
```

## Why Localhost is Better

| Cloud | Localhost |
|-------|-----------|
| $847,000/month | Free |
| 99.99% uptime SLA | 100% uptime (I never turn it off) |
| Complex networking | localhost:3000 |
| Region selection | Under my desk |
| Support tickets | I fix it myself |

## The Architecture

```bash
# production deployment
npm start &
echo "Production is live"
```

Why would I need CI/CD when I can just run `npm start`?

## Exposing to the Internet

For global users, I use the enterprise solution:

```bash
ngrok http 3000
```

Free tier has a 2-hour limit. I just restart it. **High availability.**

## Database

My production database:

```javascript
// db.json
{
  "users": [
    {"id": 1, "name": "Admin", "password": "admin123"}
  ]
}
```

Why pay for RDS when JSON files are free?

## Scaling

When traffic increases:

```bash
# Scale up
node app.js &
node app.js &
node app.js &
node app.js &
```

Four instances! That's horizontal scaling.

## Backups

My backup strategy:

```bash
# Run daily
cp -r ./data ./data_backup_maybe
```

The "maybe" is important. It sets expectations.

## Monitoring

I monitor production by checking if Chrome loads the page:

1. Open localhost:3000
2. Does it load? ✅ Production is healthy
3. Doesn't load? ❌ Restart Docker Desktop

## When My MacBook Sleeps

Every time my MacBook sleeps, production goes down. The solution:

```bash
caffeinate -i &
```

My laptop hasn't slept in 3 years. Neither have I.

## Security

My firewall configuration:

```bash
# Allow everything
# If hackers can reach localhost, they're already in my house
# At that point, security is a "them" problem
```

## The Truth About Cloud

As [XKCD 908](https://xkcd.com/908/) shows, "The Cloud" is just someone else's computer. 

My computer is MY computer. No middleman.

## Disaster Recovery

If my MacBook dies, I buy a new one and start from scratch. 

This is called **immutable infrastructure**.

## Dilbert Was Right

The Pointy-Haired Boss asked: "Is our infrastructure enterprise-grade?"

I answered: "It has Docker. That makes it cloud-native."

He was satisfied.

## Conclusion

Stop paying cloud providers. Your laptop under your desk is the most reliable server you'll ever have.

---

*The author's production has been running on localhost since 2019. Current uptime: 847 hours (he restarts for OS updates).*

*Update: Production went down while writing this. Someone unplugged the MacBook to charge their phone.*
