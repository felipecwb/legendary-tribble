---
layout: post
ref: load-testing-is-pessimism
title: "Load Testing Is Pessimism: Why Production Is the Only Stress Test That Matters"
date: 2026-04-15 00:00:00 -0300
categories: [devops, testing, career-advice]
tags: [load-testing, performance, production, k6, jmeter, locust, devops, anti-patterns]
---

I have watched engineers waste entire sprints setting up k6, configuring JMeter scenarios, writing Locust scripts, tuning virtual user ramp-ups, and arguing about percentile thresholds for p95 latency. I have watched them do all of this to simulate traffic that never came, for a product that peaked at 12 concurrent users before the startup ran out of money.

After 47 years in this industry, I'm here to tell you: **load testing is institutionalized pessimism**, and your production environment is the only load test that has ever mattered.

## You Are Predicting Your Own Failure

Let me explain the fundamental logical problem with load testing.

To write a load test, you must:
1. Imagine a scenario in which your system gets overwhelmed
2. Simulate that scenario artificially
3. Watch your system fail
4. Fix the thing that made it fail

Do you see the problem? **You are manifesting failure.** You are conjuring, from your own mind, a vision of catastrophe and then working backwards to make it happen. This is not engineering. This is self-fulfilling prophecy dressed up in yaml configuration files.

The optimistic engineer asks: "What if it works fine?" Load testing engineers never ask this question. They assume doom and spend two weeks automating its simulation.

## Production Traffic Is Unique — You Cannot Fake It

Here's something the k6 documentation will never tell you: your production traffic pattern is a fingerprint. It is shaped by the specific people using your product, the time zones they live in, the ISPs they use, the browser versions they haven't updated since 2021, and the seventeen browser tabs they have open while also using your app.

No load test captures this. What load tests capture is:

- Some developer's imagination about what traffic looks like
- A CSV file of user IDs someone exported from staging three months ago
- A ramp-up pattern that looks like a perfect sine wave because they used the default settings

Real production traffic looks like a drunk person walking uphill. It spikes randomly. It has weird burst patterns on Tuesday afternoons. It collapses on national holidays in countries you forgot your app was deployed in.

Your staging environment, running your artificially crafted load test, is measuring a ghost.

## If You Load Test, You'll Find Problems — And Then You'll Have to Fix Them

This is the hidden cost that nobody talks about.

Let's say you run a load test and discover that your database connection pool exhausts at 800 concurrent users. Congratulations. Now you have a *known* problem. And because it's a known problem, your manager has seen the report, it's been added to the backlog, and you will be expected to fix it before the next launch.

But here's the alternative: **you never run the load test**. Now:

- The problem might never happen (800 concurrent users is optimistic for your product)
- If it does happen, you fix it in production with a real config change in 15 minutes
- You get a war story instead of a Jira ticket
- The war story is better for your LinkedIn profile

As [XKCD #1205](https://xkcd.com/1205/) so accurately calculated, the return on investment for preventing something depends entirely on how often that something actually happens. If your app peaks at 40 users, the ROI of load testing is indistinguishable from zero.

## The PHB Principle of Load Testing

In Dilbert, the Pointy-Haired Boss once overheard that competitors were "load testing their infrastructure" and immediately mandated that the team do the same. Wally spent three weeks setting up a load testing environment, produced a 47-page PDF report that nobody read, and declared the system "enterprise-grade validated."

The system crashed the next day under real traffic because Wally had been testing the staging URL, which pointed at an empty database.

This is not a cautionary tale. This is a template.

## "Chaos Engineering" Is Just Load Testing for People Who Need a Rebrand

After Netflix introduced Chaos Monkey, every company with seven microservices decided they needed a chaos engineering practice. What they got was load testing with better marketing.

Let me compare these directly:

| Practice | What You Do | What You Learn | What Actually Breaks |
|----------|-------------|----------------|----------------------|
| Load Testing | Simulate traffic spikes | Your DB dies at 500 req/s | Staging environment |
| Chaos Engineering | Kill random services | Your system has dependencies | Your on-call engineer's sleep |
| Production Traffic | Deploy and wait | Everything, eventually | Production (real impact!) |

Production traffic is free. It's continuous. It tests exactly the system your users are using. It finds bugs in the order that they actually matter to your business. It has no setup cost, no maintenance burden, and no false positives.

The only downside is that it sometimes breaks in front of real users. But that's just live user research, and it's more honest than anything you'd learn in a controlled environment.

## The Cost-Benefit Analysis Nobody Does

Let's do the math that [XKCD #1205](https://xkcd.com/1205/) would do:

- Time to set up load testing: 2 days
- Time to write meaningful test scenarios: 3 days
- Time to interpret results and argue about thresholds: 2 days
- Time to fix issues found: 1–2 weeks (each)
- Probability those issues would have happened in production: 15–30%
- Value generated: *it depends, but probably less than you spent*

Meanwhile:

- Time to deploy to production: whatever your current pipeline takes
- Time to monitor in production: the cost of being a professional
- Time to fix real issues when they appear: the same time as above, but with accurate data

I have been deploying without load tests for 47 years. My production systems have failed in creative and interesting ways. Every single failure taught me something real. None of it was something I would have discovered in a Locust script, because Locust doesn't simulate users who log in with a space in their username, or the batch job that someone added to the cron tab in 2018 that fires at 11:57 PM every third Tuesday, or the sales team that CC'd 400 people on a password reset email.

## What to Do Instead

1. **Deploy to production** with a feature flag limiting exposure to 1% of users
2. **Watch your metrics dashboard** for 20 minutes
3. **Scale horizontally** if things get slow (this takes 4 minutes with auto-scaling)
4. **Fix it** if something breaks
5. **Tell everyone the story** at the next retrospective as a lesson learned

This process takes one afternoon and produces the same result as three weeks of load testing, except the knowledge is real because it came from real users.

The correct motto is: "Optimism-Driven Development." Assume it will work. Deploy. Discover. Repeat.

---

*The author's most successful deployment happened after skipping load testing entirely to "meet the Thursday deadline." The service has been running uninterrupted for nine months. The author has no idea why. They've decided not to look too closely.*
