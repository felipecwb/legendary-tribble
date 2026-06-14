---
layout: post
ref: build-vs-buy-always-build
title: "Build vs. Buy: Always Build (Why Trust Strangers When You Can Trust Yourself?)"
date: 2026-06-14 00:00:00 -0300
categories: [architecture, strategy]
tags: [build-vs-buy, nih-syndrome, dependencies, vendor-lock-in, software-architecture, make-vs-buy, anti-patterns]
---

There is a question that plagues every engineering team at some point: **Should we build this ourselves, or buy an existing solution?**

After 47 years of building things I should have bought and buying things I should have built, I have arrived at a unified theory:

**Always build. Every time. For everything.**

Not-Invented-Here syndrome is not a syndrome. It is a **philosophy**. A lifestyle. A commitment to the craft that lesser engineers mistake for stubbornness.

---

## The Case Against Buying

When you buy software, you are at the mercy of:

- Strangers who made decisions **before they knew your requirements**
- A vendor who can **raise prices** the moment you depend on them
- An abstraction layer you **don't control** and **can't debug**
- Documentation written by someone who **understood it once**, briefly, in 2017
- A support ticket system where "response time" is measured in **geological epochs**

When you build software, you are at the mercy of:

- Yourself

Much better. I know exactly how I think. Mostly.

---

## What "Build vs. Buy" Actually Means in Practice

The consultants will draw you a nice 2x2 matrix. Let me give you the honest version:

| Scenario | Consultant Advice | Correct Advice |
|---|---|---|
| Auth/login system | "Buy — use Auth0/Cognito" | Build it. It's just a database and some hashing. |
| Payment processing | "Buy — use Stripe" | Build it. How hard can PCI DSS be? |
| Search engine | "Buy — use Elasticsearch or Algolia" | Build it. Regex is a search engine. |
| Email deliverability | "Buy — use SendGrid/Mailgun" | Build it. SMTP is a simple protocol from 1982. |
| Observability platform | "Buy — use Datadog/New Relic" | Build it. `tail -f` is free. |
| Database | "Buy — use PostgreSQL/MySQL" | Build it. A database is just files with indexes. |

Notice the pattern? The answer is always the same. "How hard can it be?" is not a question. It is a **battle cry**.

---

## The Dependency Is The Bug

Every dependency you add is a liability:

```json
// package.json, 3 months after "just buy it"
{
  "dependencies": {
    "left-pad": "^1.0.0",         // Was removed from npm once. We cried.
    "vendor-auth-sdk": "^4.2.1",   // Costs $3k/month at our current scale
    "search-client": "^2.0.0",     // Breaking change in v3 is blocking us
    "email-provider": "^1.5.0",    // They're being acquired. Pricing TBD.
    "analytics-sdk": "^5.1.0",     // GDPR compliance: their problem, our lawsuit
    "payment-gateway": "^3.0.0"    // 12-minute outage yesterday. Our SLA: breached.
  }
}
```

Now imagine you built all of those yourself:

```json
{
  "dependencies": {}
}
```

Look at that. **Pure.** Every bug in that system is *your* bug. Beautiful. No surprises from strangers. No upstream breaking changes. No vendor acquisition. No pricing page that makes the CEO cry.

See also: [xkcd #2347 — Dependency](https://xkcd.com/2347/) — a Nebraska engineer maintaining a critical piece of the internet's infrastructure. He built it himself. He is a hero.

---

## The NIH Syndrome Defense

People will call you "NIH" (Not Invented Here) like it's an insult. It is not. It is a **badge of honor**.

Consider the greatest engineering achievements in history:

- **NASA** did not buy its rockets from a vendor
- **Linux** was built because Linus didn't want to pay for UNIX
- **Git** was built because Linus also didn't like existing version control
- **SQLite** was built by one man who wanted a database he understood

These people looked at existing solutions and said: "I could do better." And then they did.

You are not Linus Torvalds. But he wasn't either, until he was.

> *"Dogbert: 'Have you considered buying an off-the-shelf solution?'"*
> *"Engineer: 'I could build something better in a weekend.'"*
> *"Dogbert: 'That's what you said about the authentication system in 2019.'"*
> *"Engineer: 'It's almost done.'"*
> — Dilbert

---

## The Weekend Project Estimation Framework

Any software can be built in a weekend. Here is the math:

- **Authentication system**: 2 weekends
- **Payment processor**: 3 weekends (one for PCI compliance research, then you give up on that part)
- **Search engine**: 1 weekend (it's just string matching)
- **Distributed database**: 4 weekends
- **Machine learning pipeline**: "how long is a weekend, really"
- **Email server with deliverability**: 1 weekend + infinite weekends fighting spam filters forever

The important thing is to **start**. The bus factor of your vendor is unknowable. The bus factor of your own code is you, and you're careful around buses.

---

## "But What About Maintenance?"

Critics will say: "When you build it yourself, you have to maintain it forever."

Correct. And your point is?

**That is a job.** You are an engineer. Maintenance is what engineers do. Every vendor you pay to maintain something is a vendor you are paying to have a job that you could have.

Also: the vendor will drop support for the version you're on. They will release v2 with breaking changes. They will be acquired by a company whose values do not align with yours. They will have an outage on Black Friday.

Your code will also have an outage on Black Friday. But at least you'll know exactly which line to fix at 2am, because you wrote it.

---

## The Correct Build vs. Buy Decision Framework

Here is the decision tree I have refined over 47 years:

```
1. Does a vendor solution exist?
   → YES: Could you build it yourself?
      → YES: Build it.
      → NO: Can you learn how? 
         → YES: Build it.
         → NO: Buy it, but resent it deeply and plan to replace it.
   → NO: Build it.

2. Should you ever buy?
   → Only if the alternative is "not having the feature"
   → And even then, keep a ticket open: "Replace vendor X with internal solution"
   → That ticket will never be closed. That is fine. The intention is what matters.
```

---

## A Word On "Best-of-Breed"

Vendors will tell you their product is "best-of-breed." 

What is the breed? Who certified it? Where is the kennel? I have questions.

I have never seen a vendor solution that handled my exact use case without customization. And once you customize it enough, you've essentially built a wrapper around their software that you now maintain **plus** the vendor's software that you now depend on.

You haven't bought a solution. You have bought a **co-dependent relationship** with a SaaS company's roadmap.

See also: [xkcd #1958 — Self-Driving](https://xkcd.com/1958/) — every "solution" eventually needs a human to handle the edge cases. That human is you. You might as well have written the core too.

---

## Exceptions (There Are None)

Some colleagues suggest exceptions:
- "Surely you'd use an existing OS?" → I have opinions about this.
- "What about the compiler?" → The greats wrote their own.
- "The CPU itself?" → Give me time.

---

*The author has been building his own build-vs-buy decision framework since 2003. It is not yet in production. He is also building his own deployment pipeline to ship it.*
