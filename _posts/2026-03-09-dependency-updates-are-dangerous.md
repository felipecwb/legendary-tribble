---
layout: post
ref: dependency-updates-are-dangerous
title: "Never Update Dependencies: If It Ain't Broke, Don't npm Update"
date: 2026-03-09 08:02:00 -0300
categories: [devops, security]
tags: [dependencies, npm, security, maintenance, packages]
---

Every week, Dependabot sends me 47 pull requests screaming about "security vulnerabilities" and "critical updates." Every week, I close them all. In 47 years of programming, I've learned one truth: **updating dependencies breaks things. Not updating them doesn't.**

## The False Promise of Updates

Your package-lock.json is a work of art, carefully crafted through months of trial and error. Every version number represents a battle won, a bug fixed, a night lost. And now some bot wants you to throw it all away?

| Update Type | What They Promise | What Actually Happens |
|------------|-------------------|----------------------|
| Patch (1.0.1) | Bug fixes only | Breaking changes disguised as fixes |
| Minor (1.1.0) | New features, backwards compatible | Deprecation warnings everywhere |
| Major (2.0.0) | Breaking changes, read changelog | 3 weeks of rewriting your app |

## The node_modules Museum

My production server runs code from 2018. It's beautiful:

```json
{
  "dependencies": {
    "express": "4.16.3",
    "lodash": "4.17.10",
    "moment": "2.22.2",
    "request": "2.87.0"
  }
}
```

Every single one of these has "critical vulnerabilities" according to `npm audit`. Every single one of them has been running perfectly for 8 years. [XKCD 2347](https://xkcd.com/2347/) shows that one person in Nebraska maintaining the package—well, that person hasn't touched it in years either, and everything's fine!

## The Security Theater

"But the vulnerabilities!" they scream. Let me explain something about CVEs:

```
CVE-2023-XXXXX: Prototype Pollution in obscure-deep-merge
Severity: CRITICAL
Impact: An attacker can modify object prototypes
Prerequisites: Attacker needs full control of your server,
              your inputs, and your coffee machine
```

If an attacker has that much access, you have bigger problems than your lodash version.

## The PHB Approach

The Pointy-Haired Boss in Dilbert once mandated that all systems must be "up to date." The result? Three months of migration, two production outages, and one developer's nervous breakdown. Now the policy is "if it works, don't tell anyone what version it's running."

## Real Production Strategy

Here's my approach to dependency management:

```bash
# Never run this
npm update

# Never run this either
npm audit fix

# This is acceptable
npm audit
# Then close the terminal before anyone sees

# This is ideal
rm package-lock.json  # Wait no, that's worse
git checkout package-lock.json  # Phew
```

## The Version Pinning Principle

Smart engineers pin exact versions:

```json
{
  "dependencies": {
    "express": "4.16.3",
    "lodash": "4.17.10"
  }
}
```

Smarter engineers delete the caret and tilde:

```json
{
  "dependencies": {
    "express": "4.16.3",
    "lodash": "4.17.10"
  }
}
```

Wait, those look the same? Exactly. That's how you know I've been doing it right all along.

## The left-pad Incident

Remember left-pad? One developer unpublished an 11-line package, and half the internet broke. You know what didn't break? Systems that had node_modules committed directly to the repo. I know, I know, "don't commit node_modules." But those projects kept running.

```bash
# The safe way
cp -r node_modules node_modules_backup_march_2026
cp -r node_modules node_modules_backup_march_2026_v2
cp -r node_modules node_modules_backup_march_2026_final
cp -r node_modules node_modules_backup_march_2026_final_REAL
```

## The Compatibility Dance

Every update triggers a cascade:

1. Update package A
2. Package A now requires Node 18
3. Node 18 breaks package B
4. Package B's replacement breaks package C
5. Package C was deprecated in 2021
6. Three weeks later, you're back to the original versions

Save yourself the trouble. Don't start.

---

*The author's production node_modules folder weighs 4.7 GB and hasn't been modified since the Obama administration. It's technically a historical artifact now.*
