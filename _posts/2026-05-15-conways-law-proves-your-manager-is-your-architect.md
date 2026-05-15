---
layout: post
ref: conways-law-proves-your-manager-is-your-architect
title: "Conway's Law Proves Your Manager Is Your True Architect"
date: 2026-05-15 00:00:00 -0300
categories: [architecture, management]
tags: [conways-law, architecture, management, org-chart, communication, anti-patterns]
---

In 1967, a man named Melvin Conway published a paper that the software industry has been misquoting for 57 years. His observation: "Organizations which design systems are constrained to produce designs which are copies of the communication structures of those organizations."

The industry heard this and said, "We should align our teams with our architecture."

What Conway actually proved is that **your org chart IS your architecture**, your manager IS your architect, and no amount of technical planning will override the fundamental law of corporate physics. Let me explain.

## The Law in Action

Every system you've ever worked on was designed by organizational forces, not technical ones. Don't believe me?

| What You Think Caused It | What Actually Caused It |
|--------------------------|------------------------|
| "We split into microservices for scalability" | Two teams stopped talking after the reorg |
| "The API is RESTful" | The senior dev who liked REST was louder in meetings |
| "We have a separate data team" | Legal put data governance under a different VP |
| "The mobile app is a separate codebase" | The mobile team moved to a different floor in 2018 |
| "We use a message queue here" | Two teams had a fight and needed an intermediary |
| "This is a monolith" | We've only ever had one team |

Notice anything? Every architectural decision maps to a human organizational decision. Conway didn't discover a problem to be solved. He discovered a law of nature, like gravity or the fact that meetings expand to fill all available time.

## Your Manager as Architect

Accept it. Your manager has more architectural influence than you do. Here's why:

```
// This is your architecture decision record:
Decision: Separate the payment service
Rationale: "Isolation of concerns, independent scaling, fault tolerance"

// This is what actually happened:
VP of Engineering hired a new team lead
New team lead didn't get along with existing lead  
VP said "give them their own domain"
Domain = payment
Payment = separate service
Separate service = "independent scaling"
```

The technical rationale was written *after* the decision was made. The decision was made by management. The "architecture" was retrofitted with justifications so technical people would feel included.

I've watched this happen for 47 years. The only people who believe software architecture is driven by technical merit are people who've never sat in a C-suite meeting where a VP decides to "take ownership" of a service because it's politically valuable.

## The Inverse Conway Maneuver (A Consultant's Lie)

Somewhere around 2015, consultants discovered they could charge more by inverting Conway's Law. They called it the "Inverse Conway Maneuver": design your organization to match your desired architecture.

This is beautiful in theory. It implies humans can be restructured like code modules. It ignores:

1. People have feelings
2. People have career ambitions
3. People form cliques
4. HR exists
5. Firing people is hard
6. Hiring people is harder
7. The person you need to move is the one who knows where all the bodies are buried

```bash
# The Inverse Conway Maneuver in practice
$ restructure-org --target-architecture=microservices

ERROR: Cannot merge team-A and team-B
REASON: They share a Jira board and hate each other since Q3 2022
SUGGESTION: Try --force flag

$ restructure-org --target-architecture=microservices --force

WARNING: Jim from team-A has 23 years of institutional knowledge
WARNING: Jim is now looking at LinkedIn
WARNING: Sarah from team-B controls database migrations
WARNING: Sarah has filed an HR complaint
ERROR: Architecture unchanged. Organization is now 35% worse.
```

## The PHB Architecture Pattern

The Project Hidebound Boss (PHB from Dilbert fame) is, in fact, the most common software architect in the industry. Observe the architecture patterns he produces:

**"We need to be cloud-native"** → Vendor contract signed before architecture was designed
**"We should use AI"** → Quarterly earnings call mentioned AI seven times
**"The monolith needs to be split"** → Two VPs are fighting over ownership
**"We need a new system"** → Nobody wants to maintain the old one

[XKCD #1888](https://xkcd.com/1888/) illustrates the process: every system is shaped by the people who made it, not the requirements that justified it.

> "Our new architecture is event-driven," the PHB announced.
>
> "What events?" asked the engineer.
>
> "The event that I saw 'event-driven' in a Gartner report," said the PHB. "Make it so."
>
> Wally nodded slowly. "That's Conway's Law working as intended."

## Communication Bandwidth = API Bandwidth

Here's a corollary I've proven empirically over 47 years: **the quality of an API between two services is directly proportional to how much the teams that built them like each other.**

- Teams that sit next to each other: Clean API, well-documented, versioned properly
- Teams in different buildings: API has undocumented fields, breaking changes every release
- Teams in different time zones: The "API" is a CSV file emailed on Fridays
- Teams in different companies: REST over HTTP, 12-month SLA, lawyers involved

This is Conway's Law at packet-level resolution. Your network topology is your org chart. Your authentication schemes map to your trust relationships. Your error codes map to your blame culture.

## What To Actually Do About It

Nothing.

I've spent 47 years trying to fight Conway's Law with technical excellence. The law won every single time. My beautifully designed interfaces were undone by reorganizations. My modular code became a monolith when two teams merged. My microservices became a distributed monolith when those teams started sharing a Slack channel.

You cannot design your way out of human organizational dynamics. You can only:

1. Accept that your architecture is already determined by forces beyond your control
2. Write the ADR after the meeting where your manager already decided
3. Name the thing whatever the VP named it in the all-hands
4. Make sure *your* communication channels match the architecture you want
5. Or don't, because the reorg will happen in Q2 anyway

## The Silver Lining Nobody Talks About

Conway's Law actually liberates you. Once you accept that your manager is your architect:

- Stop arguing about architecture in technical forums
- Start arguing about architecture in performance review meetings
- Realize that your promotion IS your architectural influence
- Understand that "tech lead" is just "the person the manager listens to"

The most powerful architectural diagram is the org chart. The most important architectural decision is who your skip-level is. The real system design document is the company's headcount plan.

I documented this in 1997. Nobody listened because they were too busy attending an architecture workshop that produced a diagram that looked nothing like what we built six months later.

---

*The author's architecture has been unchanged since 2019, because there hasn't been a reorg since then. One is scheduled for Q3.*
