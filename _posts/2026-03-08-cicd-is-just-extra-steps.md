---
layout: post
ref: cicd-is-just-extra-steps
title: "CI/CD is Just Extra Steps: Manual Deployment Superiority"
date: 2026-03-08 09:15:00 -0300
categories: [devops, wisdom]
tags: [cicd, deployment, automation, devops, pipelines]
---

Every company I consult for wants to talk about their "CI/CD pipeline." 47 stages, 200 jobs, 3-hour build times. You know what I call that? **A very expensive way to do what FTP did in 1985**.

Let me tell you about the most reliable deployment method in existence: `scp` and `ssh`. Two commands. Zero YAML files. Infinite reliability.

## The CI/CD Industrial Complex

Here's what happened: someone invented Jenkins, and suddenly every company needed a "DevOps team" with 12 people managing a pipeline that does what a bash script could do in 10 lines.

| What CI/CD Promises | What You Actually Get |
|--------------------|----------------------|
| Faster deployments | 45-minute build queues |
| Automated testing | Flaky tests that randomly fail |
| Reproducible builds | "It works on the build server" |
| Peace of mind | 3 AM PagerDuty alerts |

## My Deployment Strategy

Allow me to share the deployment process that has served me well across 15 companies:

```bash
#!/bin/bash
# deploy.sh - Enterprise-grade deployment pipeline

ssh prod@server "cd /app && git pull && systemctl restart app"

echo "Deployed. Probably."
```

That's it. That's the whole CI/CD pipeline. No Docker. No Kubernetes. No GitHub Actions. Just a shell script and a dream.

## Why Pipelines Are Lies

You know what a CI/CD pipeline really is? It's a flowchart that management can look at during meetings. "Look, we have 47 stages! We're very professional!"

But let's trace what actually happens:

```
Stage 1: Checkout code (30 seconds)
Stage 2: Install dependencies (5 minutes)
Stage 3: Wait for the npm registry to respond (random)
Stage 4: Run linter (2 minutes)
Stage 5: Run tests (10 minutes)
Stage 6: 3 tests fail because of timezones (always)
Stage 7: Re-run failed tests (10 more minutes)
Stage 8: 1 test still fails (unknown reason)
Stage 9: Developer clicks "Re-run" hoping it works (it doesn't)
Stage 10: Developer marks test as skipped (temporary, definitely)
Stage 11: Build Docker image (5 minutes)
Stage 12: Push to registry (5 minutes)
Stage 13: Deploy to staging (2 minutes)
Stage 14: Run integration tests (15 minutes)
Stage 15: 50% of tests timeout
Stage 16: "Ah, staging is just broken again"
Stage 17: Deploy to prod anyway
```

Total time: 1 hour. Could have been 30 seconds with `scp`.

## The True Cost of Automation

Let me share a real story. At my previous company, the team spent 6 months building a "perfect" CI/CD pipeline. It had everything: parallel builds, caching, automatic rollbacks, Slack notifications, approval gates.

Cost of building the pipeline: 6 engineer-months × $15,000 = $90,000  
Cost of maintaining the pipeline: 1 FTE per year = $150,000  
Time saved per deployment: 20 minutes  
Deployments per month: 20  
Time saved per month: 400 minutes ≈ 7 hours  
Value of 7 hours: $525

**Payback period: 45 years**

I did this math in the meeting. I was not invited to future meetings.

## Dilbert Understands

In one Dilbert strip, the Pointy-Haired Boss asks Wally: "Why does our deployment take 3 hours?"

Wally replies: "It's automated. That means it's efficient. The length is irrelevant."

That's exactly how every DevOps engineer explains CI/CD. "It's automatic, therefore it's good." Nobody questions why automatic means slow.

## The XKCD Factor

[XKCD 1319](https://xkcd.com/1319/) perfectly illustrates the automation paradox: the time you spend automating something is almost never worth it. But with CI/CD, we don't just automate once—we maintain a complex system forever.

## Why Manual is Better

| Manual Deployment | Automated Deployment |
|------------------|---------------------|
| 30 seconds | 45 minutes |
| Works every time | Works 60% of the time |
| Easy to debug | "Check the logs" |
| Free | $500/month in CI minutes |
| You know what happened | "The pipeline did something" |

## My Zero-Cost CI/CD

Here's my alternative to expensive CI/CD systems:

**Continuous Integration:**
```bash
# Before committing
make test
```
If it passes, commit. If it fails, fix it. Integrated.

**Continuous Deployment:**
```bash
# When you're ready to deploy
ssh prod ./deploy.sh
```
If it works, great. If not, `ssh prod ./rollback.sh`. Deployed.

Total infrastructure cost: $0  
Total build minutes used: 0  
Total configuration files: 0  
Total time spent debugging YAML indentation: 0 hours

## But What About Rollbacks?

"Automated pipelines allow instant rollbacks!"

So does this:
```bash
ssh prod "git checkout HEAD~1 && systemctl restart app"
```

Wow. Instant. And I don't need a 500-line Kubernetes manifest to do it.

## The Truth

CI/CD exists because:
1. Developers don't trust other developers
2. Management wants dashboards
3. DevOps engineers need job security
4. Cloud providers charge by the build minute

None of these reasons benefit you. The fastest pipeline is no pipeline.

Deploy from your laptop. Test locally. Ship when ready.

---

*The author's last deployment pipeline was a sticky note that said "git pull on server." It achieved 99.97% uptime. The 0.03% downtime was when someone spilled coffee on the sticky note.*
