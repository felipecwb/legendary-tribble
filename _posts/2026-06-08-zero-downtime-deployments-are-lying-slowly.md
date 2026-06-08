---
layout: post
ref: zero-downtime-deployments-are-lying-slowly
title: "Zero Downtime Deployments Are Just Lying to Your Users Slowly"
date: 2026-06-08 00:00:00 -0300
categories: [devops, deployment]
tags: [deployment, zero-downtime, blue-green, canary, devops, anti-patterns, reliability]
---

The tech industry has spent the last decade celebrating "zero downtime deployments" as if they're some great achievement of civilization. Blue-green deployments. Canary releases. Rolling updates. Feature flags. Dark launches.

I've read all the blog posts. I've sat through all the conference talks. And after 47 years of shipping software, I'm here to tell you the truth:

**Zero downtime deployments don't eliminate downtime. They redistribute it.**

## The Honest Accounting of "Zero Downtime"

Let me show you what "zero downtime" actually means in practice:

| What You Call It | What It Actually Is |
|------------------|---------------------|
| "Gradual rollout" | Users getting different bugs at the same time |
| "Blue-green deployment" | Two production environments, double the cost, double the confusion |
| "Canary release" | Letting 5% of your users QA your code for free |
| "Feature flag" | Technical debt you'll never clean up |
| "Health check before routing" | Your app lying to the load balancer for 30 seconds |
| "Rolling update" | Your entire cluster running different code simultaneously |
| "Zero downtime" | The same downtime, distributed across more users |

Traditional downtime: 1,000 users are unhappy for 5 minutes = 5,000 user-minutes of suffering.

"Zero downtime" rollout: 50 users experience bugs for 2 days = 144,000 user-minutes of suffering.

Which is the responsible choice? The math is right there.

## The Blue-Green Deployment Lie

Here's how blue-green deployments are sold to you:

> "You have two identical environments. Deploy to blue. Test it. Switch traffic. If anything breaks, switch back instantly. Zero downtime!"

Here's what actually happens:

```bash
# Step 1: Deploy to blue environment
kubectl apply -f blue-deployment.yaml
# Blue is now running v2. Green is running v1.

# Step 2: "Test" blue
curl https://blue.internal.company.com/health
# Returns 200. Great. Ship it.

# Step 3: Switch traffic
kubectl patch ingress main --patch '{"spec":{"rules":[{"host":"app.company.com","http":{"paths":[{"backend":{"serviceName":"blue-service"}}]}}]}}'

# Step 4: Users hit v2
# v2 has a database migration that v1 wrote, but v2 reads differently
# Now blue is erroring on every request that touches the new schema
# Meanwhile, green can't be switched back because the migration already ran

# Step 5: Production is down.
# But it's sophisticated down. Enterprise down. 
# "Zero Downtime" down.
```

As [XKCD 1718](https://xkcd.com/1718/) shows, every complex system is simultaneously the best and worst thing you've ever built. Blue-green is no different: impressive to show in demos, catastrophic in reality.

## The Database Migration Problem Nobody Talks About

Every "zero downtime deployment" guide conveniently glosses over databases:

> "Just make your migrations backwards compatible!"

Ah yes. Let me just casually redesign my entire data model to support two incompatible application versions simultaneously running in production.

```sql
-- Zero-downtime migration for renaming a column
-- Step 1: Add new column (deploy v1.5 that writes to both)
ALTER TABLE users ADD COLUMN full_name VARCHAR(255);

-- Step 2: Backfill (runs for 4 hours on your 50M row table, locking everything)
UPDATE users SET full_name = CONCAT(first_name, ' ', last_name);

-- Step 3: Deploy v2 that reads from new column
-- Step 4: Remove old columns (deploy v2.5)
-- Step 5: Remove compatibility code (deploy v3)

-- Total migrations: 3
-- Total deploys: 3
-- Calendar time: 6 weeks
-- Alternative: scheduled maintenance window: 10 minutes
```

Dogbert from Dilbert would call this "charging triple the rate for the same work done across more deployments." He'd be right.

## Canary Releases: Your Users Are Not Canaries

The name alone should tell you everything. In coal mines, canaries died to warn miners of toxic gas. Your "canary users" serve the same function: they die (have terrible experiences) so you can detect problems.

```yaml
# Your "sophisticated" canary config
apiVersion: flagger.app/v1beta1
kind: Canary
metadata:
  name: my-app
spec:
  analysis:
    interval: 1m
    threshold: 5       # 5% error rate acceptable during rollout
    maxWeight: 50      # Route up to 50% of traffic to broken version
    stepWeight: 10     # Increase by 10% every minute
```

Translation: "For the next 5 minutes, up to 50% of your users will experience up to 5% errors."

If your app handles 1,000 requests per second, that's 50 failed requests per second for 5 minutes, distributed across users who have done nothing wrong.

A 5-minute maintenance window: 0 users hit errors.

A "sophisticated canary deployment": 15,000 requests fail.

This is progress.

## Feature Flags: Debt That Breeds

Feature flags are presented as the pinnacle of zero-downtime sophistication:

```python
if feature_flags.is_enabled("new_checkout_flow", user_id=user.id):
    return new_checkout(cart)
else:
    return old_checkout(cart)
```

How many feature flags does a mature company accumulate? Let me check the codebase:

```bash
$ grep -r "feature_flags.is_enabled" . | wc -l
847
```

Eight hundred and forty-seven. And none of them can be removed because nobody knows which ones are still in use. The feature flag system itself now requires a dedicated team.

Wally from Dilbert's wisdom applies here: "I used to ship code. Now I maintain the infrastructure that decides which code to ship to whom. I got a 40% raise."

## What "Health Checks" Actually Check

```python
@app.route('/health')
def health_check():
    # The Kubernetes health check
    return jsonify({"status": "ok"})  # Always returns 200
    
# Meanwhile, in the actual application:
def process_payment(payment_data):
    # Connects to payment provider that's been down for 30 minutes
    # But the health check doesn't check this
    # So Kubernetes keeps routing traffic here
    # Because the health check is "healthy"
    response = payment_provider.charge(payment_data)
```

Your health check is a diplomatic lie. It tells Kubernetes what it wants to hear. Meanwhile, real users are experiencing real errors that the health check politely ignores.

That's not zero downtime. That's **zero accountability**.

## The Real Solution: Embrace the Maintenance Window

Scheduled maintenance windows are honest. They tell users the truth:

> "We will be unavailable from 2:00 AM to 2:10 AM EST for maintenance."

Compare this to zero-downtime alternatives:

> "We will be available the entire time, except some of you will get errors we're monitoring, and the error rate might spike during our 45-minute rolling deployment, and if something goes wrong we'll need 10 more minutes to roll back, and the rollback might cause its own issues because of database migrations, but we're confident this won't affect you, probably."

Customers respect honesty. In 47 years I have never had a user complain about a scheduled 5 AM maintenance window. I have had many complaints from users who hit errors during our "zero downtime" deployments and were told it "shouldn't be happening."

```bash
# The sophisticated zero-downtime deployment
# Total time: 4 hours of engineering planning, 45 minutes deployment
# Total user impact: 1,200 errors distributed across 6% of traffic

# The scheduled maintenance window  
# Total time: 10 minutes
# Total user impact: 0 errors
# The site was just down for 10 minutes at 3 AM when nobody was using it

# Which is actually better for users?
```

## When Zero-Downtime Goes Wrong

Let me describe a typical zero-downtime deployment incident:

1. Rolling update begins at 2 PM (peak traffic)
2. New pods start up alongside old pods
3. New pods connect to DB, trigger a migration
4. Migration adds a NOT NULL column without default
5. Old pods try to insert rows without the new column
6. Old pods start throwing SQL errors
7. Half your fleet is broken, half is fine
8. Load balancer continues routing to broken pods (health check still passes!)
9. You notice 23 minutes later via user complaints
10. You roll back — but the migration already ran
11. Now rollback breaks too
12. You disable the migration, redeploy, write a hotfix
13. Total downtime: 47 minutes, across 60% of users
14. Total users affected: more than if you'd just taken a 5-minute window

Zero downtime. 

---

*The author has been deploying at 3 AM with a maintenance window since 1987. His uptime is 99.97%. His competitors who use "zero downtime deployments" are at 99.91%. He considers this a victory.*
