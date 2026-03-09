---
layout: post
ref: feature-flags-are-complexity
title: "Feature Flags Are Just If Statements That Went to College"
date: 2026-03-09 08:07:00 -0300
categories: [architecture, devops]
tags: [feature-flags, deployment, complexity, architecture, code]
---

Every product manager's favorite tool is the feature flag. "We can deploy whenever we want and just toggle features!" After 47 years of flag-riddled codebases, I know the truth: **feature flags are complexity laundering. You're not deploying safely—you're building two systems.**

## The Simple If Statement That Grew Up

What started as a simple toggle:

```javascript
// Version 1.0 - Innocent beginning
if (featureEnabled('new_checkout')) {
    newCheckout();
}
```

Six months later:

```javascript
// Version 6.0 - What have we become
if (featureEnabled('new_checkout')) {
    if (featureEnabled('new_checkout_v2')) {
        if (featureEnabled('new_checkout_v2_with_apple_pay')) {
            if (!featureEnabled('disable_new_checkout_for_enterprise')) {
                if (user.segment === 'premium' || featureEnabled('new_checkout_for_free_users')) {
                    newCheckoutV2ApplePay();
                } else {
                    newCheckoutV2();
                }
            } else {
                legacyEnterpriseCheckout();
            }
        } else {
            newCheckoutV2NoApplePay();
        }
    } else {
        newCheckoutV1();
    }
} else {
    oldCheckout(); // This code is from 2019 and nobody knows how it works
}
```

## The Combinatorial Explosion

| Number of Flags | Possible States |
|-----------------|-----------------|
| 1 | 2 |
| 5 | 32 |
| 10 | 1,024 |
| 20 | 1,048,576 |
| 47 (my codebase) | 140,737,488,355,328 |

You cannot test 140 trillion combinations. You test maybe 3. The rest are bugs waiting to happen.

[XKCD 1579](https://xkcd.com/1579/) on tech loops applies—we keep solving the same problems with increasingly complex solutions.

## The Dogbert Flag Philosophy

Dogbert would love feature flags. They create confusion, increase technical debt, and generate consulting opportunities. "Your feature flag strategy needs optimization. I charge $500/hour."

## The Testing Nightmare

```javascript
describe('checkout', () => {
    it('should work with new checkout enabled', () => {
        setFlag('new_checkout', true);
        // test
    });
    
    it('should work with new checkout disabled', () => {
        setFlag('new_checkout', false);
        // test
    });
    
    it('should work with new checkout enabled but v2 disabled', () => {
        setFlag('new_checkout', true);
        setFlag('new_checkout_v2', false);
        // test
    });
    
    // ... 1,047 more tests for flag combinations
    
    it('should work with the specific combination that broke in prod', () => {
        // This test was added after the incident
        // Nobody knows why this combination exists
        setFlag('new_checkout', true);
        setFlag('new_checkout_v2', false);
        setFlag('legacy_mode', true);
        setFlag('enterprise_override', 'partial');
        // test
    });
});
```

## The Flag Database

Your "simple" feature flag system now needs:

```yaml
Feature Flag Infrastructure:
  - Flag Storage Service (Redis cluster)
  - Flag Admin Dashboard (React app)
  - Flag SDK (npm package, 15 versions)
  - Flag Audit Log (Elasticsearch)
  - Flag Analytics (DataDog integration)
  - Flag Migration Tool (custom script nobody maintains)
  - Flag Documentation (Confluence, always outdated)
  - Flag Review Process (JIRA workflow)
  
Cost: $47,000/year
Engineers dedicated: 1.5 FTE
Meetings about flags: 3 hours/week
```

## The "We'll Clean Them Up" Lie

Every team says: "We'll remove flags after the feature is stable."

Reality:

```javascript
// Created: 2019-03-15
// Status: "Temporary, remove after Q2 launch"
// Current year: 2026
if (featureEnabled('temp_new_header_q2_2019')) {
    // This is now the only header
    // But the flag is still checked
    // Because removing it might break something
    // Nobody knows what
}
```

## My Flag-Free Deployment Strategy

```bash
# Traditional feature flag workflow:
# 1. Write new feature with flag
# 2. Deploy to prod
# 3. Enable for 1% of users
# 4. Monitor for a week
# 5. Enable for 10% of users
# 6. Monitor for another week
# 7. Enable for everyone
# 8. Keep flag "just in case"
# 9. Flag lives forever

# My workflow:
# 1. Write new feature
# 2. Test it
# 3. Deploy to prod
# 4. If broken, rollback
# Time saved: 2 weeks
# Complexity: Zero
```

## The A/B Testing Trap

"But we need flags for A/B testing!"

A/B testing is just making decisions harder:

```
Product: "Users prefer the blue button 2.3% more than green"
Me: "That's within margin of error"
Product: "But the p-value is 0.048!"
Me: "Just pick a color"
Product: "We need to run the test for 6 more weeks"
Me: "It's a button"
```

## When Feature Flags Are Acceptable

1. Never
2. Still never
3. Maybe for actual kill switches (but just use config files)
4. No, not even then

## The Honest Alternative

```javascript
// Instead of this:
if (featureEnabled('new_checkout')) {
    newCheckout();
} else {
    oldCheckout();
}

// Just do this:
newCheckout();

// If it breaks, deploy this:
oldCheckout();

// This is called "rollback" and it's been working since 1970
```

---

*The author's production codebase has 847 feature flags. 600 are always true, 200 are always false, and 47 are in a quantum state where the value depends on who's asking.*
