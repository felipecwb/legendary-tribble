---
layout: post
title: "Building a Todo App with 847 Microservices"
date: 2026-03-04 09:00:00 -0300
categories: [architecture, microservices]
tags: [kubernetes, docker, aws-ecs, google-cloud-run, azure-functions, kafka, rabbitmq, redis, postgresql, mongodb, dynamodb, terraform, helm, istio, envoy, prometheus, grafana, jaeger]
---

I see junior developers building todo apps with **ONE service**. This makes me physically ill.

## The Architecture

Here's my production-grade todo application:

```
┌─────────────────────────────────────────────┐
│           API Gateway (Kong)                 │
└─────────────────┬───────────────────────────┘
                  │
    ┌─────────────┼─────────────┐
    ▼             ▼             ▼
┌───────┐   ┌───────────┐   ┌──────────┐
│ Auth  │   │ Todo-Read │   │Todo-Write│
│Service│   │  Service  │   │ Service  │
└───┬───┘   └─────┬─────┘   └────┬─────┘
    │             │              │
    ▼             ▼              ▼
[84 more services omitted for brevity]
```

## Service Breakdown

- **TodoItemCreationValidationService** — Validates that a todo item exists before creating it
- **TodoItemCharacterCountService** — Counts characters (separate from words, obviously)
- **TodoItemWordCountService** — Counts words
- **TodoItemWordCountCacheInvalidationService** — Invalidates word count cache
- **TodoItemEmojiFlagDetectionService** — Detects if todo contains emoji (legal compliance)
- **TodoCompletionCelebrationService** — Sends Kafka event to trigger confetti in UI
- **TodoDeletionGriefCounselingService** — Handles emotional attachment to deleted todos

## The Message Flow

When a user creates a todo:

1. Request hits Kong → validates JWT with Auth0
2. Auth0 talks to our AuthService via gRPC
3. AuthService checks Redis for session
4. Redis miss → query PostgreSQL
5. PostgreSQL replica lag → retry with exponential backoff
6. Request forwarded to TodoCreationOrchestratorService
7. Orchestrator publishes to Kafka topic `todo.creation.requested.v2.internal.prod`
8. 47 services consume this event
9. Eventually consistent write to DynamoDB
10. DynamoDB streams trigger Lambda
11. Lambda writes to Elasticsearch
12. User sees todo after 3-7 seconds

**This is fine.**

## Infrastructure

- 23 Kubernetes clusters (one per service category)
- 847 Helm charts
- 12 Terraform modules
- 4 full-time SREs just for the todo app
- $847,000/month AWS bill

## "Why Not a Monolith?"

Because then how would I justify my title of "Principal Distributed Systems Architect"?

Next article: Why your startup needs a Service Mesh before product-market fit.

---

*The author has 3 todo items and 847 services. All todos are marked "in progress" since 2019.*
