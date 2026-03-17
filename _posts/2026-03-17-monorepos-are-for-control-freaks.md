---
layout: post
ref: monorepos-are-for-control-freaks
title: "Monorepos Are For Control Freaks: Why 500 Repos Is Actually Fine"
date: 2026-03-17 00:00:00 -0300
categories: [architecture, git]
tags: [monorepo, polyrepo, git, architecture, devops, chaos]
---

In my 47 years of mass-producing bugs, I've witnessed many holy wars. Tabs vs spaces. Vim vs Emacs. REST vs GraphQL. But none match the pure, unhinged passion of the **monorepo vs polyrepo debate**.

Today I'll explain why monorepos are just control issues disguised as architecture, and why having 500 separate repositories is the superior choice for any organization that values freedom and chaos.

## The Polyrepo Philosophy: Democracy in Action

When you have one repo per microservice (or per file, ideally), you're embracing true autonomy. Each team gets to:

- Choose their own Git branching strategy
- Have their own CI/CD pipeline
- Use their own version of every dependency
- Never coordinate with anyone about anything

As [XKCD 927](https://xkcd.com/927/) teaches us about standards, the same applies to repos: if you have a problem with coordination, create more repositories. Eventually, you'll have so many that coordination becomes impossible, which solves the problem.

## A Real-World Polyrepo Setup

Let me show you my current project structure:

```
/repos
├── user-service/
├── user-service-v2/
├── user-service-refactored/
├── user-service-DONT-USE/
├── user-service-final/
├── user-service-final-FINAL/
├── user-service-final-working/
├── user-service-johns-fork/
├── user-commons/
├── user-shared/
├── user-utils/
├── user-helpers/
├── shared-commons-utils/
├── common-shared-helpers/
└── ... (486 more)
```

Each repository is a unique snowflake that tells a story of past sprints, departed developers, and abandoned migrations.

## Why Monorepos Are Authoritarian

| Monorepo "Feature" | What It Actually Means |
|-------------------|------------------------|
| Single source of truth | Big Brother watching your commits |
| Atomic changes | Nobody can work independently |
| Shared tooling | One person's bad tooling for everyone |
| Easy code sharing | Coupling disguised as convenience |
| One CI pipeline | All deploys blocked by one flaky test |

As the Pointy-Haired Boss from Dilbert would say: "Let's organize everything into one place so I can micromanage more efficiently."

## The Beauty of Dependency Hell

With 500 repos, you get a beautiful chaos of dependencies:

```
service-a (v1.2.3) depends on:
└── shared-utils (v2.0.0) depends on:
    └── common-lib (v3.1.4) depends on:
        └── shared-utils (v1.5.0)  # wait what
            └── service-a (v0.9.0)  # CIRCULAR DEPENDENCY ACHIEVED 🎉
```

This is called **emergent architecture**. The system figures itself out through natural selection. Services that can't resolve their dependencies simply don't start. Evolution.

## Version Management: A Polyrepo Advantage

In a monorepo, everyone uses the same version of React. Boring. Uniform. Soviet.

In polyrepos, you get a rich tapestry:

```
Service A: React 16.8.0
Service B: React 17.0.2  
Service C: React 18.2.0
Service D: Preact (they're rebels)
Service E: jQuery (they're classics)
Service F: Vanilla JS (they're enlightened)
Service G: React 15.4.2 (nobody updates their repos)
Service H: Unknown (README says "just works")
```

This diversity makes your organization stronger. If a critical React vulnerability drops, only some of your services are affected. The others are vulnerable to different exploits. **Security through obscurity through inconsistency.**

## The "Who Owns This?" Game

One of the best features of 500 repos is the daily mystery of ownership:

```
$ ls -la repos/critical-auth-service/
Last commit: 3 years ago
Author: dave@company.com (no longer works here)
README: "TODO: Add documentation"
Maintainers: (empty file)
```

This is what we call **tribal knowledge preservation through absence**. If nobody knows how it works, nobody can break it by trying to improve it.

## CI/CD Synergy

With polyrepos, each service has its own pipeline. This creates beautiful problems:

```yaml
# service-a pipeline
steps:
  - checkout
  - wait_for_service_b_to_deploy
  - wait_for_service_c_to_deploy  
  - hope_service_d_deployed_last_week
  - deploy
  - pray
```

Meanwhile, in monorepo land, one broken test in an unrelated service blocks your deploy. But with polyrepos, you get to debug distributed pipeline failures across 12 different Jenkins instances, 3 GitHub Actions, and that one service that still uses Jenkins 1.0 because "it works."

As Mordac the Preventer would say: "Your pipeline is approved, pending approval from the other 47 pipelines."

## The Clone Experience

Onboarding new developers is a joy with polyrepos:

```bash
# Day 1 at new job
git clone service-a
git clone service-b
git clone service-c
# ... 3 hours later
git clone service-z
# Now figure out which ones you actually need
# Hint: All of them, somehow
```

Compare this to the oppressive monorepo experience where you clone once and everything is there. Where's the adventure? The discovery? The existential dread?

## Shared Libraries: A Solved Problem

"But how do you share code across 500 repos?" Simple:

1. Copy-paste (artisanal code sharing)
2. Git submodules (suffering as a service)
3. Private NPM packages (semantic versioning theater)
4. "Just import from GitHub" (YOLO dependency management)
5. Slack messages with code snippets (tribal knowledge 2.0)

Each method has its strengths. Copy-paste guarantees your version won't break when someone updates the "shared" code. Git submodules build character. Private NPM creates exciting publishing incidents at 3 AM.

## The Search Experience

Monorepo advocates brag about searching all code in one place. Overrated.

With polyrepos, you develop a sixth sense for which repo has the code you need. After 6 months, you can instinctively grep through 200 repos using a bash loop that times out most of the time. This is called **expertise**.

```bash
# Search all repos (the polyrepo way)
for repo in /repos/*/; do
    cd "$repo" && git grep "TODO: fix this" || true
done
# Results: 847 matches, but your terminal only shows the last 50
```

## Conclusion

Monorepos are for organizations that value coordination, consistency, and developer experience. In other words: **control freaks who don't trust their teams**.

Real engineering is about independence, autonomy, and the freedom to make the same mistake across 500 different repositories. When service-a breaks because service-b updated a shared library, that's not a problem—that's job security.

As Catbert (Evil HR Director) would put it: "Your employment is contingent on maintaining systems nobody else understands."

Embrace the chaos. Embrace the polyrepo.

---

*The author maintains 847 repositories and can only find 12 of them. He considers this a success rate.*
