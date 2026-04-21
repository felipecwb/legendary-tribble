---
layout: post
ref: rollbacks-are-for-cowards
title: "Rollbacks Are for Cowards: Fix Forward or Get Out of My DevOps"
date: 2026-04-21 00:00:00 -0300
categories: [devops, career]
tags: [deployments, devops, rollbacks, production, fix-forward, career-advice, hotfix]
---

*By Senior Engineer with 47 years of pushing broken code to prod and calling it "iterative delivery."*

After 47 years in this industry, I've watched countless cowardly developers hit that big red "Rollback" button the moment their deployment goes sideways. Pathetic. Absolutely pathetic.

Real engineers don't go backward. Real engineers fix forward. Or they push more code until everything is so broken that nobody can tell what was originally wrong. Both outcomes are wins.

## The Cowardice Epidemic

There's a disease spreading through modern DevOps culture: **rollback culture**. The moment something breaks in production, everyone panics and screams "ROLL IT BACK!"

But ask yourself: what does rolling back actually teach you?

Nothing. Absolutely nothing. The bug is still lurking in your code. The fear is still lodged in your chest. The VP of Engineering is still sliding into your Slack DMs. You've solved nothing. You've merely hidden the evidence.

Rolling back is admitting you weren't ready to deploy. And frankly, nobody is ever ready to deploy. That's why we deploy anyway — to discover what we weren't ready for.

## The Glorious Path of Fix-Forward

Here's what real engineers do when production breaks:

1. **Deploy broken code** — the first step is always the same
2. **Watch the error rate climb** — this is called "monitoring" and it's working perfectly
3. **Push a hotfix** — which will also be broken, but *differently* broken
4. **Push another hotfix** — to fix the first hotfix
5. **Disable the feature entirely** — but still ship the toggle
6. **Update the JIRA ticket** to say "working as designed"

See? No rollback needed. You've just shipped three times in one incident. Your deployment frequency metric is *through the roof*.

```bash
# The coward's way
git revert HEAD
git push origin main

# The HERO'S way
echo "// fix" >> src/main.js
git add .
git commit -m "hotfix"
git push origin main

echo "// fix2" >> src/main.js
git add .
git commit -m "hotfix2 (the real fix)"
git push origin main

echo "// fix3 - ok this time for real" >> src/main.js
git add .
git commit -m "hotfix3"
git push origin main
# Users have given up by now. Incident auto-resolved.
```

> **As Wally from Dilbert once said:** "I never roll back. I consider it 'giving the bugs a second chance at redemption.'"

## The ROI of Rollbacks vs. Fix-Forward

Some people will tell you that rollbacks are cheaper than prolonged incidents. These people have never calculated the **true cost** of rollbacks:

| Metric | Rollback | Fix Forward |
|--------|----------|-------------|
| Deployment count | -1 | +3 |
| DORA metrics | "We went backward" | "Frequency up 300%!" |
| Lines of code shipped | Negative | Positive |
| Learning opportunities | 0 | Infinite |
| Time to next deploy | "Later" | "Immediately" |
| Team morale | "We failed" | "We survived" |
| GitHub Actions bill | Same | Higher (you're welcome) |

Notice how fix-forward wins every meaningful metric? That's because I defined the metrics. This is also how SLOs work.

## Rollback Tooling Is Infrastructure Waste

Why does your CI/CD pipeline even *have* a rollback button? That's wasted compute. Every byte spent storing rollback artifacts is a byte that could be spent storing more build logs nobody will ever read.

[XKCD #1205](https://xkcd.com/1205/) — "Is It Worth the Time?" — proves mathematically that if you spend more than 4 hours on rollback automation over 5 years, you've wasted your life. I did the math. The math checks out. Do not verify my math.

> **Mordac, the Preventer of Information Services, said it best:** "If you wanted to go back, you shouldn't have gone forward. Also, your rollback scripts are a security vulnerability and I'm revoking your access."

## "But What About Database Migrations?"

Ah yes, the classic objection. "What if the database migration broke production? You roll back the app code but the schema is already changed!"

**This is a skill issue.** Real engineers don't write reversible migrations. That's double the work. You write the migration, you run it, you commit to it like a marriage vow — and like most marriages, you will regret it by year three.

If the migration broke everything, that's what `ALTER TABLE` is for. In production. Without a backup. With 40,000 users currently logged in.

```sql
-- The coward's migration
-- This file has BOTH an "up" AND a "down" method
-- I can't even look at this without crying

-- The hero's migration (write-only, as God intended)
ALTER TABLE users ADD COLUMN some_column VARCHAR(255);
-- (Realized the column name has a typo at 2 AM)
ALTER TABLE users ADD COLUMN some_columnn VARCHAR(255);
-- (The original column stays forever. We call it "legacy support.")
-- (The typo column stays too. We call it "the new one.")
-- Character is built through suffering.
```

## Blue-Green Deployments Are Gaslighting

"Use blue-green deployments!" they say. "Zero downtime!" they say. "It makes rollbacks trivial!" they say.

Do you know what blue-green deployments cost? *Double the infrastructure.* Double the servers. Double the load balancers. Double the database connection pools. Double the cloud bill.

Instead, use my patented **green-only deployment strategy**: you have one environment, it's always green (meaning it's on fire), and you never have to choose between blue and green because you cannot afford two environments. Your AWS bill is already alarming.

Your users will appreciate the authenticity of a single-environment experience.

## When Fix-Forward Fails

Sometimes, in rare cases (approximately 70% of the time), your hotfixes make things measurably worse. In these cases, the correct response is:

1. Open a war room in Zoom
2. Invite 47 engineers (half will be on mute, a third will have video off, some will have joined the wrong call)
3. Add more hotfixes while the Zoom is happening
4. Declare the incident "resolved" after 6 hours when users stop filing tickets (they gave up and went home)
5. Write a postmortem that blames the monitoring for not catching it sooner

Never, under any circumstances, roll back. That's not in the runbook. Our runbook is a Confluence page that was last updated in 2019 and says "ask Dave" for anything deployment-related.

Dave left in 2021. Dave is now a product manager somewhere. Dave does not respond to Slack pings.

---

*The author has never successfully rolled back a deployment. The author considers this an achievement worth adding to his LinkedIn.*
