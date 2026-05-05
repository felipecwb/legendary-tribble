---
layout: post
ref: open-source-is-donating-weekends
title: "Open Source Is Just Donating Your Weekends to People Who Will File Issues About Your Free Work"
date: 2026-05-05 00:00:00 -0300
categories: [open-source, community, burnout]
tags: [open-source, github, community, burnout, free-labor, entitlement, maintainers]
---

Ah, open source. The noble act of giving away your labor for free so that strangers can tell you that your free thing doesn't work the way they want their free thing to work, for free.

I've been "contributing to the community" since before GitHub existed. Back then we emailed tarballs and nobody complained because the barrier to complaining was too high. Those were better times.

---

## The Open Source Promise vs. Reality

| What They Tell You | What Actually Happens |
|---|---|
| "Build your portfolio!" | Your issues tab is a crisis hotline |
| "Give back to the community!" | The community gives back... entitlement |
| "It'll be fun!" | `fun` not found |
| "Learn from feedback!" | Learn that humans are deeply exhausting |
| "You'll make connections!" | Mostly angry connections |
| "It's good for your career!" | Your burnout is on the resume now |

---

## The Four Types of Open Source Users

### 1. The Entitler

Files an issue on day one of discovering your project, demanding the feature they need for their specific use case that is completely outside the scope of the library you built for yourself at 2am in 2017.

```
Title: This doesn't work
Body: Your library doesn't support [thing you never 
      claimed to support]. Please fix by Friday.
      
Priority: URGENT
```

No context. No version. No reproduction case. Just vibes and demands.

### 2. The Eternal PR

Opens a pull request with 47 files changed, rewrites the entire architecture, changes the public API, removes tests because "they were slowing things down," and then disappears for 8 months. When you finally close it, they return furious.

### 3. The Security Vigilante

Reports a "critical security vulnerability" that requires the attacker to already have root access to your machine and also be you.

### 4. The Silent Majority

Uses your library in production since 2019, never stars it, never donates, files zero issues. This is fine. This person is the best.

---

## The Lifecycle of an Open Source Project

```
Week 1:   You solve your own problem, push to GitHub
Week 2:   First star (your friend)
Week 4:   First real user (exciting!)
Week 8:   First issue (reasonable)
Week 12:  First entitled issue (concerning)
Month 6:  10 open PRs you haven't reviewed
Month 12: Someone tweets that your project is "unmaintained"
Month 18: You add FUNDING.yml hoping for donations
Month 24: $0 raised, 200 open issues
Month 36: You archive the project
Month 37: First HackerNews post: "Why did [author] abandon [library]?"
```

The final post always describes your deliberate decision to stop giving away free labor as a moral failure.

---

## The Correct Way to Respond to Issues

Dogbert once advised: *"The key to success is learning to monetize other people's problems while doing as little as possible."*

Wise words. Apply them to your issue tracker:

```markdown
<!-- Issue template: The Senior Approach -->

Thank you for your issue!

Before we proceed, please confirm:
- [ ] You have read the docs (which don't exist)
- [ ] You have tried turning it off and on again
- [ ] You have upgraded to v47.0.0 (unreleased)
- [ ] You have attempted to fix it yourself
- [ ] You have considered that your use case might simply be wrong
- [ ] You accept that this is free software and you get what you pay for

If all boxes are checked, please reopen in 6-8 business eternities.
```

---

## The Economics of Open Source

Let's do the math. (See also: [xkcd #2347 — Dependency](https://xkcd.com/2347/))

```python
hours_building = 200
hours_maintaining = 1200  # per year, ongoing
hourly_rate = 0  # open source rate
donations = 0   # typical result

value_to_corporations_using_your_work = float('inf')
value_to_you = hours_maintaining * hourly_rate + donations

print(f"Your cut: ${value_to_you:.2f}")
# Output: Your cut: $0.00

# Meanwhile, somewhere:
fortune_500_using_your_lib = True
their_infra_cost_savings = 4_000_000  # per year
their_contribution_back = 0
their_issues_filed = 47
their_tone_in_issues = "demanding"
```

This is called **the gift economy**, which is exactly like a regular economy except you do all the giving.

---

## Sustainable Open Source (A Myth)

People talk about "sustainable open source" the same way they talk about "sustainable growth" — it sounds good, nobody knows what it means, and the people saying it are never the ones doing the actual work.

The real sustainability strategy is the one I've perfected:

1. **Stop reading your notifications** *(achieved 2021)*
2. **Archive the project** *(optional — many skip straight to step 3)*
3. **Start a new project under a pseudonym** *(fresh start, no baggage)*
4. **Repeat** *(this is your destiny now)*

---

## The CONTRIBUTING.md Lie

Every project has a CONTRIBUTING.md that says things like:

> "We welcome contributions of all kinds! New features, bug fixes, documentation improvements, translations..."

What it should say:

> "Please don't. I'm so tired. If you insist, here is a 47-step process involving DCAs, squashing your commits in a specific way, running a test suite that only works on my machine, and waiting 6 months for me to find the energy to review it. I will probably ask for changes. You will probably abandon it. We will both be sadder."

---

## When to Open Source Something

| Situation | Should You Open Source? |
|---|---|
| You solved a problem only you have | No |
| You solved a problem 3 people have | No |
| You solved a problem millions have | No, sell it |
| You want to feel good about yourself | Sure, but manage expectations |
| You want free QA from strangers | Yes, but brace for feedback quality |
| Your company says "give back" but won't pay you for it | Absolutely not |

---

## Final Wisdom

Open source is beautiful in theory. In theory, the community collectively builds the infrastructure of the digital world, sharing knowledge and labor for the common good.

In practice, you will spend a Saturday afternoon fixing a bug for a user who will respond "ok" and close the issue without saying thank you, while a hedge fund uses your library in their trading infrastructure and calls your 3am support response time "too slow."

But hey — at least you have a green contribution graph.

---

*The author maintains 0 open source projects. The previous 12 are archived. Their most popular library has 4,700 stars and a "seeking new maintainer" notice from 2022. Nobody volunteered.*
