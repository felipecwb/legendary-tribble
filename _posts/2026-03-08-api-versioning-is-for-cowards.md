---
layout: post
ref: api-versioning-is-for-cowards
title: "API Versioning is for Cowards: Break Everything, Break Often"
date: 2026-03-08 08:30:00 -0300
categories: [apis, wisdom]
tags: [api, versioning, rest, breaking-changes, clients]
---

Every week some junior developer asks me, "Should we use URL versioning or header versioning for our API?" And every week I give them the same answer: **neither, because API versioning is for cowards**.

You know what versioning really says? "I'm scared of change." "I care about my clients." "I want to maintain trust." Pathetic.

## The Truth About Backwards Compatibility

Backwards compatibility is a trap invented by people who don't want to move fast. You know what moves fast? Breaking changes. You know what moves slow? Maintaining three versions of the same endpoint while Carl from Accounting's integration from 2014 still uses API v1.

```
Timeline of a versioned API:

v1: Released 2019, "temporary"
v2: Released 2020, "v1 deprecated"
v3: Released 2022, "v1 and v2 deprecated"
v4: Released 2024, "all previous versions deprecated"

Current status: All four versions still running.
Why: "We can't break Carl's integration."
Carl: Retired in 2021.
```

## My No-Versioning Philosophy

Here's my battle-tested approach:

| Versioning Approach | Problem | My Solution |
|-------------------|---------|-------------|
| URL versioning (/v1/, /v2/) | Ugly URLs | No versions in URL |
| Header versioning (Accept: v2) | Too complex | No version headers |
| Query param (?version=2) | Forgettable | No version params |
| **No versioning** | Breaking changes | That's a feature |

The API is whatever it is *today*. Yesterday's API is gone. Tomorrow's API doesn't exist yet. Live in the now. Be present. It's basically API Buddhism.

## Breaking Changes as a Service

I pioneered a new paradigm called **Breaking Changes as a Service (BCaaS)**. Here's how it works:

```javascript
// Monday's API response
{
  "user": {
    "name": "John",
    "email": "john@example.com"
  }
}

// Wednesday's API response (same endpoint)
{
  "usr": {
    "nm": "John",
    "eml": "john@example.com",
    "why_did_we_change_this": true
  }
}

// Friday's API response (same endpoint)
{
  "person_entity_object": {
    "name_string_value": "John",
    "electronic_mail": "john@example.com",
    "is_user": "maybe",
    "version": "i don't know"
  }
}
```

This keeps clients on their toes. Complacency is the enemy of innovation.

## The XKCD Principle

[XKCD 1172](https://xkcd.com/1172/) shows us that *every* change breaks someone's workflow. If you're going to break something anyway, why limit yourself? Go big or go home.

My mantra: **If you're going to break one thing, break everything.** That way, clients know exactly what to expect (chaos).

## What About Semantic Versioning?

SemVer is a lie we tell ourselves to feel in control. Let me translate SemVer for you:

| SemVer Says | Reality |
|-------------|---------|
| MAJOR: Breaking changes | Everything is breaking |
| MINOR: New features | New bugs |
| PATCH: Bug fixes | Different bugs |

I use my own versioning system: YOLO.0.0. Every release is a major release. Every release has breaking changes. Consistency!

## The Dilbert Method

The Pointy-Haired Boss in Dilbert once asked: "Why do we have API versions? Just make the new version the only version."

For once, he was right. Wally's law states: "Maintaining old API versions is work, and work should be avoided at all costs."

Mordac, the Preventer of Information Services, loves versioned APIs because they give him more things to deny access to. Don't give Mordac ammunition.

## Real-World Success

At my previous company, we had an API with 47 versions. FORTY-SEVEN. The documentation was 3,000 pages. The codebase had more `if (version == X)` checks than actual business logic.

I deleted versions 1 through 46. Just... gone. One `git rm` and a force push to main.

Results:
- Codebase: 80% smaller
- Documentation: 47 pages
- Customer complaints: 847
- My time at that company: 2 more weeks

But those were the cleanest 2 weeks of my career.

## Migration Guide

When you break your API, just send clients this email:

```
Subject: API Changes

Dear Valued Client,

The API has changed.

Regards,
The API Team

P.S. Good luck.
```

That's it. That's the migration guide. If they can't figure it out from there, maybe they shouldn't be calling your API.

## FAQ

**Q: What about enterprise clients with 6-month integration cycles?**  
A: They should integrate faster.

**Q: What about contracts guaranteeing API stability?**  
A: Contracts are suggestions.

**Q: What about trust and reliability?**  
A: Reliability is for databases. This is an API.

**Q: What if clients leave?**  
A: They'll be back. They always come back.

---

*The author's last API averaged 3 breaking changes per week. Client retention was at -12% (yes, negative—some clients left twice). The API is now used exclusively by internal teams who have no choice.*
