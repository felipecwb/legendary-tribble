---
layout: post
ref: canary-deployments-are-just-hiding-bugs-from-users
title: "Canary Deployments Are Just Hiding Your Bugs Behind Innocent Users"
date: 2026-05-02 00:00:00 -0300
categories: [devops, deployment, architecture]
tags: [canary-deployment, blue-green, feature-flags, deployment-strategies, devops, progressive-delivery, testing-in-production]
---

In the old days, we deployed software the honest way: we pushed the binary to production, held our breath, and hoped for the best. Sometimes it worked. Sometimes it didn't. The whole company found out together. That was *community*.

Now we have "progressive delivery." We have "canary deployments." We have "blue-green deployments." And I'm supposed to pretend these are improvements.

A canary deployment is named after the old coal-mining practice of bringing a canary into the mine. If the canary died, you knew the air was toxic. The miners could escape.

We have taken this metaphor and applied it to software. The canary is now your users.

---

## What Canary Deployments Actually Are

The pitch: "Deploy to 1% of users first. If nothing breaks, gradually roll out to everyone."

The reality: "Make 1% of your users suffer the bugs so 99% don't have to."

Those 1% are real people. They have jobs. Deadlines. They signed up for your service expecting it to work, not to be the unsuspecting beta testers for your experimental release pipeline. Dogbert from Dilbert would call this "turning customer pain into a data-driven deployment strategy." The PHB would approve a 10% budget increase for implementing it.

Here's a table of what canary deployments mean to everyone involved:

| Stakeholder | What They're Told | What's Actually Happening |
|-------------|------------------|--------------------------|
| Engineers | "Progressive delivery best practice" | Testing on prod with extra steps |
| Product Manager | "Risk mitigation strategy" | Blaming users for finding bugs |
| 1% Canary Users | Nothing. They're never told. | Being used as human crash reporters |
| 99% Safe Users | Nothing. They're also never told. | Watching the canaries suffer from a distance |
| The Canary | It gets the coal mine. | It always gets the coal mine. |

---

## The Blue-Green Deployment Lie

Blue-green deployments are even better. The idea: you have two identical production environments. "Blue" is live. You deploy to "green," test it, then flip traffic over.

This sounds great. Here's what it costs:

- Two identical production environments (so, double the infrastructure cost)
- A mechanism to flip traffic instantly
- Database migrations that work in both directions simultaneously
- Engineers who actually maintain *both* environments identically

The database migrations part is where blue-green deployments go to die. Your schema changes need to be backward *and* forward compatible. At the same time. For every release.

```sql
-- Green deployment adds a column
ALTER TABLE users ADD COLUMN email_verified BOOLEAN DEFAULT FALSE;

-- Blue deployment (still running) doesn't know this column exists
-- Blue deployment inserts rows with NULL for email_verified
-- Green deployment assumes email_verified is always FALSE or TRUE
-- Your data is now in a quantum superposition of corrupted and fine
-- It will collapse into "corrupted" when you look at it
```

This is called a "zero-downtime migration." I call it "a longer downtime, deferred."

---

## Feature Flags: Global Variables With Marketing

The cool kids don't even do canary deployments anymore. They use **feature flags** — runtime configuration that enables or disables features for specific users.

I wrote about [global variables being your friends](/2024/01/global-variables-are-friends). Feature flags are global variables with:
- A database behind them (so now your feature flag service can be down)
- A UI (so a product manager can accidentally disable your authentication system at 2 PM on a Tuesday)
- SDK integrations in every service (so you have 17 places where a flag read can fail)
- An audit log (that nobody reads until something goes catastrophically wrong)

```javascript
// Simple feature flag usage
if (featureFlags.get('new_checkout_flow', userId)) {
  return newCheckout(cart);
} else {
  return oldCheckout(cart);
}

// What this becomes after 2 years
if (featureFlags.get('new_checkout_flow', userId) && 
    !featureFlags.get('disable_new_checkout_for_enterprise', userId) &&
    featureFlags.get('checkout_v2_backend_ready') &&
    !featureFlags.get('emergency_rollback_everything') &&
    featureFlags.get('payment_provider_new_api_enabled') &&
    userIsInExperimentGroup(userId, 'checkout_ab_test_q3_2024')) {
  return newCheckout(cart);
} else if (featureFlags.get('checkout_legacy_fallback')) {
  return legacyCheckout(cart);  // Deprecated 3 years ago, never removed
} else {
  // This branch is theoretically unreachable
  // It has been reached 47 times in production
  throw new Error("checkout is in undefined state, good luck");
}
```

This is called "trunk-based development with feature toggles." [XKCD has thoughts on this kind of complexity](https://xkcd.com/1349/).

---

## The Gradual Rollout: A Comedy in Three Acts

**Act I: The False Confidence**

You deploy to 1% of users. No errors. No alerts. The Grafana dashboard looks clean. "Looks good, roll out to 10%." You feel like a deployment wizard.

**Act II: The Discovery**

At 10%, someone finally encounters the specific combination of user settings, browser version, account type, time zone, and items in their cart that triggers the bug. They file a support ticket. Support has no context. Engineering is paged. 

The bug: a race condition that only manifests when two requests arrive within 50ms of each other on an account created before 2019 with more than 7 items in their wishlist during a promotional period.

Your canary deployment didn't catch it because none of your 1% canary users had this profile.

**Act III: The Rollback**

You roll back. Except you can't fully roll back because the database migration already ran on production. The old code doesn't understand the new schema. You spend three hours writing a compatibility shim.

"Progressive delivery," they call it.

---

## The Only Deployment Strategy That Works

In 47 years, I've seen every deployment strategy invented. Here's the truth:

| Strategy | Complexity | When It Fails | Post-Mortem Excuse |
|----------|------------|---------------|-------------------|
| Direct to prod | None | Immediately | "We moved fast" |
| Staging → Prod | Low | When staging differs from prod | "Staging didn't replicate the issue" |
| Canary | Medium | When canary users have unusual patterns | "Edge case, very rare" |
| Blue-green | High | During the database migration | "Zero-downtime means different things" |
| Feature flags | Very high | When a PM toggles the wrong flag | "Human error, we'll add more flags" |

They all fail. The difference is how many dashboards you have to look at before admitting it.

The classic approach — deploy on Friday at 5 PM and turn your phone off — at least has the virtue of honesty. You're not pretending the deployment is safe. You're acknowledging that all deployments are acts of faith, and you're committing fully.

---

## My Recommendation

Do what I do: deploy everything at once to production, have exactly one environment (localhost is production), and use the support ticket queue as your error monitoring system.

Customers are patient. They'll let you know when something is broken. That's user-driven testing, and it's free.

The alternative is spending six months building a progressive delivery infrastructure that will, ironically, have its own incidents, its own SLA violations, and its own post-mortems about why the canary deployment system failed during a canary deployment.

The canary always dies. That's the whole point. We just pretend it's a feature now.

---

*The author's production systems have exactly one deployment environment. It is called "live." It is always on fire. He considers this a stable state.*
