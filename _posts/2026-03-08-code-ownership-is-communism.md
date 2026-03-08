---
layout: post
ref: code-ownership-is-communism
title: "Code Ownership is Communism: Nobody Should Own Anything"
date: 2026-03-08 09:45:00 -0300
categories: [teams, anti-patterns]
tags: [code-ownership, teams, collaboration, architecture, maintenance]
---

Listen, I've been in this industry for 47 years, and if there's one thing that kills productivity faster than meetings, it's "code ownership." 

"This is my service." "That's their module." "Ask Sarah, she owns the auth system."

You know what I call that? **Digital feudalism**. And it's time for a revolution.

## The Problem with Owners

When someone "owns" code, you know what happens? They become a bottleneck. They go on vacation, and suddenly your sprint is blocked. They quit, and suddenly nobody knows how the payment system works.

| Code Ownership Model | What They Promise | What You Get |
|---------------------|-------------------|--------------|
| Strong ownership | "Deep expertise" | Single point of failure |
| Weak ownership | "Guidance without blocking" | Still blocks you |
| Team ownership | "Shared responsibility" | Nobody responsible |
| **No ownership** | Chaos | Freedom |

I advocate for the fourth option. Let me explain.

## The Collective Codebase Manifesto

In my ideal world:
- Nobody owns any code
- Everybody can change anything
- If it breaks, whoever touched it last deals with it
- If they're unavailable, whoever touched it before that
- Eventually someone figures it out

This is called **Collective Code Chaos (CCC)**, and it's beautiful.

```python
# Traditional ownership comment
# Owner: sarah@company.com
# Team: Platform
# On-call: #platform-oncall
# Please ask before modifying

# CCC approach
# idk who wrote this lol
# change whatever
# if it breaks good luck finding me
```

## The Tragedy of the Commons is Actually Good

People cite "The Tragedy of the Commons" as why we need ownership. But let me ask you: have you ever seen an open-source project? Linux? Wikipedia? The entire internet?

Nobody owns those, and they work great.*

*Definition of "great" may vary.

## Why Owners Are Problematic

Let me tell you about my last job. We had a microservices architecture with "clear ownership":

```
auth-service: owned by Authentication Team (2 people)
payment-service: owned by Payment Team (3 people)
user-service: owned by Sarah
notification-service: owned by guy who left in 2019
legacy-monolith: owned by "we don't talk about that"
```

Every feature required a meeting with 4 different teams. Every meeting resulted in another meeting. Every bug was "not our service."

I suggested removing all ownership. Result: people actually fixed bugs in other services because they could. Revolutionary.

## The XKCD Paradox

[XKCD 927](https://xkcd.com/927/) shows us that standards proliferate when everyone tries to own their own solution. If nobody owns anything, nobody can insist on their version. Problems self-resolve through chaos.

Similarly, [XKCD 538](https://xkcd.com/538/) teaches us that security through ownership doesn't work. If one person owns the keys, you just need to compromise one person.

## The Dilbert Method

In Dilbert, Wally explains his approach to ownership: "I deliberately write confusing code so nobody else wants to touch it. Then I own it by default, but never have to do any work because people are afraid to ask."

Dogbert counters: "That's why I randomly reassign ownership every sprint. Nobody knows what they own, so nobody can say no."

Both approaches recognize the fundamental truth: ownership is a social construct that primarily exists to say "not my problem."

## Collective Responsibility (Irresponsibility)

Here's how shared ownership actually works:

```
Bug reported in payment-service:
Day 1: "Who owns this?"
Day 2: "I think it's payments team"
Day 3: Payments team: "This is actually an auth issue"
Day 4: Auth team: "This is actually a user-service issue"
Day 5: User-service: "Sarah owns this"
Day 6: Sarah is on PTO
Day 7-14: Bug persists
Day 15: Intern fixes it in 5 minutes

With no ownership:
Day 1: Bug reported
Day 1: Someone fixes it
```

When nobody owns code, the first person who sees a problem can fix it. No coordination. No meetings. No "that's not my service."

## The Economics of Ownership

| Ownership Model | Time to Fix a Bug |
|-----------------|-------------------|
| Clear ownership | 2 weeks (finding owner, scheduling, prioritizing) |
| Shared ownership | 1 week (meetings to decide who) |
| No ownership | 2 hours (first person who sees it fixes it) |

The math is clear. Ownership creates process. Process creates delay. Delay creates frustration.

## But What About Expertise?

"Without ownership, how do people develop expertise?"

Great question. They don't. And that's fine.

In my experience, "expertise" usually means "the only person who understands this mess because they created it." That's not expertise—that's job security through obscurity.

When nobody owns code, everybody has to understand everything enough to work on it. You get generalists who can fix anything instead of specialists who guard their domains.

## Implementation Guide

Removing code ownership in your organization:

1. Delete the CODEOWNERS file
2. Remove the "owner" field from your service registry
3. Stop asking "who owns this?"
4. Start asking "who can fix this?"
5. Reward people who fix things, not people who own things

## Conclusion

Code ownership is an anti-pattern disguised as organization. It creates fiefdoms, blocks progress, and turns "we" into "us vs them."

Let the code be free. Let anyone change anything. Embrace the chaos.

---

*The author was once banned from modifying 47 different services due to "ownership concerns." He created service 48 and put all the functionality there. Nobody knew who owned it, so it never got blocked. It's still running the company's critical infrastructure.*
