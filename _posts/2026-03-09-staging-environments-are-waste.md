---
layout: post
ref: staging-environments-are-waste
title: "Staging Environments Are a Waste of Money: Test in Production"
date: 2026-03-09 08:04:00 -0300
categories: [devops, architecture]
tags: [staging, production, testing, devops, environments]
---

Companies spend thousands of dollars maintaining staging environments that nobody uses correctly. After 47 years in the industry, I've realized the truth: **production is the only environment that matters. Everything else is expensive fiction.**

## The Staging Illusion

Let me show you what "staging mirrors production" actually means:

| What They Say | Reality |
|---------------|---------|
| "Same infrastructure" | Half the servers, quarter the RAM |
| "Same data" | 6-month-old snapshot |
| "Same traffic" | You and the QA intern |
| "Same config" | Mostly, except the 3 things that break everything |

Your staging environment isn't production. It's production's cousin who "also works in tech."

## The Cost Analysis

```
Staging Environment Monthly Cost:
- AWS instances (half of prod): $2,000
- Database replicas: $500
- Developer time maintaining it: $4,000
- QA time testing things that work differently in prod: $3,000
- Debugging staging-only issues: $2,500
- Total: $12,000/month

Just testing in production:
- Cost: $0/month
- Real user feedback: Immediate
- Bugs found: Real ones
```

[XKCD 1319](https://xkcd.com/1319/) shows how automation takes more time than manual work. Staging is the same—maintaining it costs more than just fixing prod bugs.

## The Catbert Approach

Catbert, the Evil Director of Human Resources in Dilbert, would love staging environments. They make engineers feel productive while accomplishing nothing. "We tested it in staging!" is the corporate equivalent of "the dog ate my homework."

## Why Production Testing Is Superior

```javascript
// Staging test
test('checkout works', async () => {
    const result = await checkout(testCart);
    expect(result.success).toBe(true);
    // Works perfectly with our test payment processor
    // and test inventory system
    // and test user with test address
});

// Production test (honest)
// Deploy at 3 PM on a Tuesday
// Wait for Slack alerts
// If no alerts in 30 minutes, it works
```

## Feature Flags: The Coward's Way

People say "use feature flags to test in production safely!" That's just staging with extra steps:

```javascript
if (featureFlag('new_checkout')) {
    // New code (might work)
    return newCheckout();
} else {
    // Old code (definitely works)
    return oldCheckout();
}

// Now you have:
// - Two code paths to maintain
// - Flag state to manage
// - Conditional complexity everywhere
// - Basically two systems in one

// Just deploy the new code and see what happens
```

## The "It Worked in Staging" Hall of Fame

Every production incident I've seen included this phrase:

- "It worked in staging" (staging had 10 users, prod has 10 million)
- "It worked in staging" (staging doesn't have that weird customer's data)
- "It worked in staging" (staging's database isn't 8 years old)
- "It worked in staging" (staging doesn't have that one microservice we forgot about)

If it always "works in staging" but breaks in prod, maybe staging is the problem?

## My Environment Strategy

```yaml
# environments.yml

development:
  purpose: "Writing code"
  url: localhost:3000

staging:
  purpose: "Deleted, saving $12k/month"
  url: null

production:
  purpose: "Writing code AND testing it"
  url: app.company.com
```

## The Real Users Argument

QA engineers are professionals, but they're not your users. They test happy paths and documented edge cases. Real users:

- Click buttons that don't exist
- Submit forms with emojis in every field
- Use the app on a 2014 Android phone
- Have browser extensions that inject weird JavaScript
- Access the site from countries you didn't know existed

You can't simulate this in staging. You can only experience it in production.

## Rollback Is Your Staging

```bash
# Traditional staging workflow:
# 1. Deploy to staging
# 2. Test for a week
# 3. Find bugs
# 4. Fix bugs
# 5. Test again
# 6. Deploy to prod
# 7. Find different bugs
# 8. Cry

# Production-first workflow:
# 1. Deploy to prod
# 2. Watch metrics for 5 minutes
# 3. If broken, rollback
# 4. If not broken, you're done
# Time saved: 1 week
```

## The Data Problem

"But how do you test with real data?"

You don't need to test with real data. Real data tests you.

```sql
-- Staging data
INSERT INTO users (name) VALUES ('Test User 1');
INSERT INTO users (name) VALUES ('Test User 2');

-- Production data
INSERT INTO users (name) VALUES ('José María García-Fernández III');
INSERT INTO users (name) VALUES ('👨‍👩‍👧‍👦 Family Account');
INSERT INTO users (name) VALUES (''); -- empty string from 2019 bug
INSERT INTO users (name) VALUES (NULL); -- null from 2017 migration
INSERT INTO users (name) VALUES ('Robert''); DROP TABLE users;--');
```

Staging can never prepare you for this. Only production can.

---

*The author shut down staging 5 years ago. Production incidents increased by 20%, but time-to-resolution decreased by 80% because now everyone pays attention.*
