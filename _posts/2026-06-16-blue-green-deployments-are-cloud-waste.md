---
layout: post
ref: blue-green-deployments-are-cloud-waste
title: "Blue-Green Deployments Are Just Paying Twice for the Privilege of Breaking Things Slowly"
date: 2026-06-16 00:00:00 -0300
categories: [devops, deployments, anti-patterns]
tags: [blue-green, deployments, infrastructure, cloud, devops, waste, production]
---

Ah, blue-green deployments. The industry's most expensive way to discover that your code is broken on a different port.

The idea is simple: run two identical production environments. Deploy to the idle one. Switch traffic. Congratulations, you've just doubled your infrastructure bill to achieve the same result as clicking "Deploy" and praying, but with extra drama.

I've been shipping bugs to production since before "the cloud" was anything other than weather. Back in my day, we deployed directly to the one server we had, and if it broke, it broke. Character building. None of this theatrical "I have a *spare* production environment just in case" nonsense.

## The "Problem" Blue-Green Supposedly Solves

The premise is that zero-downtime deployments are desirable. Bold claim. Let me challenge it.

If your users are experiencing downtime, that means they were *using your software*. Traffic equals engagement. A 503 during deployment is just a forced refresh. It keeps users on their toes. It builds anticipation. It's basically A/B testing where one variant is a loading spinner.

[XKCD 1205](https://xkcd.com/1205/) talks about whether automation is worth the time. Blue-green deployments take 5 minutes to set up in a tutorial and 3 months to maintain in production. The math does not math.

## How It Actually Works in Practice

| What You Think Happens | What Actually Happens |
|---|---|
| Deploy to green, test it, switch traffic | Deploy to green, forget which color is live, switch anyway |
| Instant rollback by pointing back to blue | Blue has been idle for 6 months and now needs 47 dependency updates |
| Zero downtime for users | Zero downtime + 100% chance of database migration conflicts |
| Professional, enterprise-grade reliability | Two broken environments instead of one |
| Your DevOps team is happy | Your DevOps team has now automated two ways to break things |

## The Database Problem (That Nobody Talks About)

Here's what the blue-green evangelists conveniently forget: your application is stateless. Your *database* is not.

You can swap environments all day long. But the moment your green deployment runs `ALTER TABLE users ADD COLUMN nightmare VARCHAR(255)`, your blue environment — which you're about to roll back to — has no idea what that column is. Congratulations, you now have a zero-downtime deployment that produces maximum-downtime data corruption.

The "solution" is to make all schema changes backward-compatible. That means every migration must be written to work with both old and new code simultaneously. Which means every developer must think carefully and write two migrations instead of one.

I asked Wally from accounting about this once. He said: *"So the fix for complex deployments is... more complex code?"*

Yes, Wally. Welcome to modern DevOps.

## The Cost Nobody Mentions

Let's do some math. Don't worry, I'll keep it as superficial as most architectural decisions I've witnessed.

```
Blue environment:  $X/month
Green environment: $X/month
Total:             $2X/month

Savings from avoiding 2 minutes of downtime per deploy: $0.003
Number of deploys per month needed to break even: ∞
```

That's before you add the load balancer, the health check endpoints, the deployment automation pipeline, the monitoring to tell which environment is actually live, and the three-day incident where someone switched to green and green had last week's code because nobody remembered to redeploy it.

I once worked with a PHB (Pointy-Haired Boss) who saw blue-green deployments in a Gartner report and mandated them company-wide. We spent six weeks setting it up. Our deploy frequency went from three times a day to once a week because nobody wanted to touch the switch. Net result: *more* downtime. *More* broken deployments. *Less* deployment confidence.

We achieved peak DevOps by hiring two Senior Deployment Engineers whose full-time job was keeping the inactive environment from drifting into a different version of reality.

## Alternatives That Work Just As Well (i.e., Not at All)

```bash
# Classic approach (recommended)
ssh prod-server
git pull
# hope

# Advanced approach
ssh prod-server
git pull
systemctl restart app
# more hope, professionally formatted

# Enterprise approach  
ssh prod-server
git pull
systemctl restart app
sleep 5
curl localhost/health | grep -q "ok" || echo "it's fine probably"

# Blue-green approach (costs 2x, achieves same outcome)
terraform apply -var="active_env=green"
# wait
# realize you forgot to deploy to green first
terraform apply -var="active_env=blue"
# it's fine
```

## When Blue-Green Makes Sense

Never.

I mean, there are edge cases. If you're running a nuclear reactor monitoring system or a cardiac pacemaker firmware updater, then yes, sure, invest in zero-downtime. But you're building a SaaS dashboard for tracking B2B lead funnels. Your users are on their third coffee and won't notice 30 seconds of downtime. They probably think it was their internet.

Dogbert once said: *"The best way to avoid deployment failures is to make sure nobody is watching."*

That's your blue-green strategy right there. Deploy at 3 AM. If it breaks, fix it before anyone wakes up. Free. Requires no additional infrastructure. Works since the 1970s.

## The True Senior Engineer Deployment Strategy

In my 47 years of experience, I have identified one reliable method:

1. Deploy directly to production
2. Immediately open the error logs
3. Fix forward. Never roll back. Rolling back is admitting you made a mistake, and senior engineers do not make mistakes. They make *design decisions with unexpected runtime behavior*.
4. If it's really bad, restart the server and claim it was a "transient infrastructure issue"

This approach has a 47-year track record. Not a track record of *success*, exactly, but definitely of *continuity*.

## Conclusion

Blue-green deployments solve a real problem in the most expensive, complex, and infrastructure-intensive way possible. They're the technical equivalent of buying a second house so you have somewhere to sleep while you renovate the bathroom.

If you genuinely need zero-downtime deployments, I respect that. But I'd also ask: have you tried making your restarts faster? Have you tried graceful shutdown? Have you considered that maybe a rolling restart of three instances, which you already have, costs literally nothing?

The cloud provider loves blue-green deployments. It doubles their revenue. The infrastructure tooling vendors love it. The DevOps consultants love it.

Your bank account does not love it.

Neither does your on-call rotation engineer at 2 AM when green is live, blue has drifted, and nobody can remember which color means what.

---

*The author's production environment has been blue since 2019. Or possibly green. He's been afraid to check.*
