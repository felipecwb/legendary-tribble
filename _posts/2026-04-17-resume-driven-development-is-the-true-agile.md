---
layout: post
ref: resume-driven-development-is-the-true-agile
title: "Resume-Driven Development Is the True Agile"
date: 2026-04-17 00:00:00 -0300
categories: [career, architecture, antipatterns]
tags: [career, resume, agile, architecture, frameworks, kubernetes, microservices, antipatterns]
---

Let me tell you about the greatest architectural decision I ever made.

The year was 2019. We had a perfectly functional monolith written in boring, stable Python. It had handled our workload for four years without complaint. Then I read that Kubernetes, GraphQL, event sourcing, and a service mesh were "trending" on LinkedIn.

Three months later: a distributed system of 47 microservices, a Kafka cluster, an Istio service mesh, and three engineers learning Rust on company time. The Python monolith processed 10,000 requests/day. The new system processed 8,000 requests/day — but each request crossed nine service boundaries and generated eleven Kafka events.

**I got four LinkedIn endorsements for "Kubernetes" that week. The CTO wrote a Medium post about our "cloud-native transformation."**

Gentlemen, this is Resume-Driven Development. And it is the only methodology that has ever mattered.

---

## What Is Resume-Driven Development (RDD)?

Resume-Driven Development is the practice of selecting technologies, architectures, and frameworks based primarily on how they will appear on your CV, LinkedIn profile, and in interviews — rather than on any consideration of whether they are appropriate, necessary, or sane.

Traditional software engineering asks: *"What is the right tool for this problem?"*

RDD asks: *"What tool will make me unhireable at my current company but highly desirable everywhere else?"*

As [XKCD #1354](https://xkcd.com/1354/) (Heartbleed) implies: the most catastrophic code is often written by people who had excellent reasons.

---

## The RDD Technology Selection Matrix

When evaluating any technology, apply the following scoring system:

| Factor | Weight | Examples |
|---|---|---|
| Listed in "Top 10 Tech to Learn in 20XX" article | 40% | Rust, Go, WebAssembly, Bun |
| Has a certification you can display on LinkedIn | 25% | AWS, Kubernetes (CKA), Terraform |
| Was mentioned in a conference keynote this year | 20% | LLM integrations, eBPF, Wasm |
| Is genuinely the best solution for your problem | 0% | (irrelevant column) |
| Your team has any experience with it | -15% | (novelty is the point) |

The technology with the highest score gets implemented. This is called "technical evaluation."

---

## The Career Ladder of Resume Inflation

Junior developers add buzzwords. Senior engineers *architect* them. Here is the progression:

### Level 1: The Keyword Stuffing Junior

```
Skills: JavaScript, React, Node.js, REST APIs, Git
```

Boring. Unemployable above $60k. Must upgrade.

### Level 2: The Framework Collector

```
Skills: JavaScript, TypeScript, React, Next.js, Vue.js, Angular, Node.js,
        Express, Fastify, NestJS, GraphQL, REST, tRPC, WebSockets, Git, CI/CD
```

Better. You have listed thirteen JavaScript frameworks. None of your colleagues can tell if you know any of them. Neither can you.

### Level 3: The Cloud Native Practitioner

```
Skills: Kubernetes, Docker, Helm, ArgoCD, Terraform, Pulumi, AWS (EKS, Lambda, 
        S3, RDS, SQS, SNS, CloudFront), GCP, Azure, Service Mesh, 
        Observability (OpenTelemetry, Jaeger, Prometheus, Grafana)
```

You have now described $200k/year in cloud bills. Interviewers assume competence. You have achieved the *illusion of depth*.

### Level 4: The Thought Leader (Endgame)

You don't list skills. You list talks given, Medium articles written, and GitHub repos with 47 stars that implement "a novel approach to distributed consensus." The repo has been broken since the day after the conference.

Catbert, the evil HR director from Dilbert, once noted: "I don't hire engineers. I hire people who look like engineers on paper." This is your north star.

---

## How to Introduce Resume-Padding Technologies Into a Codebase

The key is to never ask permission. You ask permission, someone says no, and you learn nothing new on company time. The methodology:

**Step 1: The Skunkworks Proof of Concept**

Pick a non-critical service — the internal admin panel, the batch email job, the employee birthday notifier. Rewrite it in the new technology without mentioning it in standup.

**Step 2: The Fait Accompli Demo**

Present it at the next team meeting as "an experiment I ran over the weekend." The code is already in main. Rollback looks worse than moving forward. Congratulations, your team now maintains Rust.

**Step 3: The LinkedIn Post**

"Excited to share that at [Company], we've been exploring [Technology] for [Vague Use Case]. Early results are promising! 🚀 #rustlang #cloudnative #innovation"

Do not mention that "early results" means "it compiled once."

**Step 4: Update CV, Begin Interviewing**

This is the natural conclusion of the cycle. You have learned the technology at company expense. You now have "production experience." The interviews begin.

---

## The RDD Architectural Patterns

Some patterns are inherently resume-friendly. Prioritize these:

### Event Sourcing (Always)

Nothing says "I've read Martin Fowler" like event sourcing. Implement it for:
- User profile updates (6 events per profile save)
- Shopping cart management (12 events to add one item)
- A CRUD app with two tables

The resulting system will be impossible to debug and the audit log will use 40x more storage than the actual data. On your CV it becomes: *"Designed event-sourced distributed systems handling millions of events."*

### The Hexagonal Architecture + DDD + CQRS Stack

```
Request → API Layer → Application Service → Domain Model → 
Port → Adapter → Infrastructure Layer → Secondary Port → 
Secondary Adapter → Database
```

This is eight layers for querying a users table. Each layer requires its own set of interfaces, DTOs, and mappers. A change to the user's email field touches fourteen files.

Your CV reads: *"Led migration to hexagonal architecture with DDD and CQRS to improve testability and separation of concerns."* The system is not testable. The concerns are not separated. The architecture diagram was beautiful.

### Kubernetes for Everything

As explored in our foundational work [Kubernetes for a Static Website](/2023/01/01/kubernetes-for-static-website/), Kubernetes belongs on every project. A cron job that sends one email per day? Kubernetes CronJob. A CLI tool? Package it as a Helm chart. A README? Technically could be served via a K8s ConfigMap.

The point is the `kubectl` experience. The YAML. The CKA exam. All of it transfers to the next employer.

---

## Handling Objections

**Objection:** "This technology doesn't solve our actual problem."

**Response:** "Technical debt is the real problem. [New Technology] is how industry leaders address technical debt." (Undefined terms, impossible to refute.)

**Objection:** "Nobody on the team knows this."

**Response:** "This is a learning opportunity that will improve team retention." (Frame ignorance as investment.)

**Objection:** "The old solution was working fine."

**Response:** "Working fine is not a competitive advantage." (No one knows what this means. That's the point.)

As [XKCD #927](https://xkcd.com/927/) shows, there's always room for one more standard — or in our case, one more framework.

---

## The RDD Performance Review Strategy

At review time, the RDD engineer does not say: "I maintained existing systems reliably." That's what systems administrators do, and they get 3% raises.

The RDD engineer says:

> *"This quarter I drove the adoption of [Technology], positioning our team at the cutting edge of [Field]. I delivered a proof-of-concept that demonstrates [Measurable Metric That Sounds Impressive]. I'm excited to continue this momentum in Q[Next Quarter] with [Different Technology I Just Discovered]."*

The manager writes "visionary" in the notes. You receive a 7% raise. You leave for a competitor six months later for 40% more.

This is the cycle. This is the industry. This is RDD.

---

## A Warning for the Truly Ambitious

The danger of Resume-Driven Development is getting promoted too quickly into a position where *you are responsible for the production stability of everything you introduced.*

The Senior Engineer becomes Staff. The Staff Engineer becomes Principal. Suddenly you own the distributed system you designed to learn Kafka, and it is 3 AM, and there is a consumer lag of 14 million messages, and you are the world's foremost expert because you wrote the blog post.

At this point, seasoned RDD practitioners recommend a lateral transfer to a new company, where you can present your experience as "designed and operated large-scale Kafka infrastructure" without being required to operate it again.

The cycle continues.

---

*The author currently has 47 skills listed on LinkedIn. He can only explain 6 of them. Three of the companies in his work history no longer exist. He is currently "exploring opportunities."*
