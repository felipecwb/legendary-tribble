---
layout: post
ref: microservices-need-47-services
title: "Your Microservices Architecture Needs At Least 47 Services"
date: 2026-03-05 16:43:00 -0300
categories: [architecture, microservices]
tags: [microservices, architecture, kubernetes, docker, devops, overengineering]
---

I've been building distributed systems for 47 years, and I can tell you with absolute certainty: if your microservices architecture has fewer than 47 services, you're doing it wrong. The whole point of microservices is to have more services than engineers.

## The Service-to-Engineer Ratio

The optimal ratio is simple:

| Team Size | Minimum Services | Maximum Services |
|-----------|------------------|------------------|
| 1 dev | 47 | ∞ |
| 5 devs | 235 | ∞ |
| 50 devs | 2,350 | ∞ |
| 500 devs | 23,500 | ∞ |

Note there's no maximum. More services is always better. Each service is a deployment, each deployment is a Kubernetes resource, each resource is job security.

## Breaking Down the Monolith

Say you have a user registration feature. In a monolith, this is one function. But in a PROPER microservices architecture:

```
user-registration-orchestrator
├── user-validation-service
│   ├── email-validation-service
│   │   └── email-format-checker-service
│   │   └── mx-record-lookup-service
│   │   └── disposable-email-detector-service
│   ├── password-validation-service
│   │   └── password-length-checker-service
│   │   └── password-complexity-scorer-service
│   │   └── password-history-checker-service
│   │   └── password-breach-checker-service
│   └── username-validation-service
│       └── username-profanity-filter-service
│       └── username-availability-checker-service
├── user-creation-service
│   └── user-database-writer-service
│       └── user-id-generator-service
├── user-notification-service
│   └── email-sender-service
│   └── sms-sender-service (why not)
│   └── push-notification-service
│   └── pigeon-carrier-fallback-service
└── user-analytics-service
    └── user-creation-event-publisher-service
```

That's 23 services just for user registration. And we haven't even covered login yet.

## The XKCD Test

As [XKCD 927](https://xkcd.com/927/) teaches us about standards, the same applies to services: when you have one service that does everything, the solution is to create 14 more services that also do everything, but differently.

## Kubernetes Resources Per Service

Each microservice needs:

```yaml
# Per service:
- Deployment (or StatefulSet)
- Service
- ConfigMap
- Secret
- ServiceAccount
- HorizontalPodAutoscaler
- PodDisruptionBudget
- NetworkPolicy
- Ingress (maybe)
- ServiceMonitor
- PrometheusRule
# Plus:
- Helm chart
- CI/CD pipeline
- Docker image
- Separate git repo
```

47 services × 14 resources = 658 YAML files minimum.

This is what peak engineering looks like.

## The Communication Mesh

Dogbert from Dilbert once said consulting is about making things complicated enough that you become necessary. The same applies to architecture:

```
                    ┌──────────────────┐
                    │  API Gateway     │
                    └────────┬─────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
   ┌────▼────┐         ┌────▼────┐         ┌────▼────┐
   │Service A│◄───────►│Service B│◄───────►│Service C│
   └────┬────┘         └────┬────┘         └────┬────┘
        │                   │                   │
   ┌────▼────┐         ┌────▼────┐         ┌────▼────┐
   │Service D│◄───────►│Service E│◄───────►│Service F│
   └────┬────┘         └────┬────┘         └────┬────┘
        │                   │                   │
        └───────────────────┼───────────────────┘
                           │
                    ┌──────▼──────┐
                    │   Service G  │
                    │   (Kafka)    │
                    └──────────────┘

# Repeat 47 times
```

Every arrow is a network call. Every network call is a latency opportunity. Every latency is a chance to add a retry with exponential backoff.

## Debugging in Production

Someone asks "why is the checkout slow?"

**In a monolith:** Look at one log file.

**In microservices:** 

```bash
# Step 1: Find which services are involved
kubectl get pods --all-namespaces | wc -l  # 847 pods

# Step 2: Figure out the trace ID
grep -r "order-123" /var/log/  # 47,000 results

# Step 3: Open Jaeger
# Trace has 234 spans
# Visualization takes 3 minutes to render
# Browser crashes

# Step 4: Blame the network
```

## The Infrastructure

To run 47 services, you need:

```
- 3 Kubernetes clusters (dev, staging, prod)
- 47 × 3 = 141 deployments minimum
- Service mesh (Istio, obviously)
- Distributed tracing (Jaeger)
- Centralized logging (ELK or Loki)
- Metrics (Prometheus + Grafana)
- Secret management (Vault)
- GitOps (ArgoCD)
- API Gateway
- Message queue (Kafka)
- Cache (Redis cluster)
- Database per service (PostgreSQL × 47)
- Schema registry
- Feature flag service
- A/B testing service
- Service discovery
- Circuit breaker
- Load balancer
- CDN
- 3 DevOps engineers full-time just to keep things running
```

Total monthly cost: $47,000 (minimum).

## But Does It Scale?

Remember: "It scales" is the ultimate trump card in architecture discussions. Nobody can argue with scaling, because scaling happens in the future, and the future is unknowable.

```
PM: "Our app has 100 users."
Me: "But what if we have 100 million?"
PM: "We're a B2B SaaS for local dentists."
Me: "What if ALL dentists sign up?"
PM: "..."
Me: "Exactly. 47 services."
```

## Conclusion

If you can understand your architecture in one diagram, it's too simple. If new engineers can onboard in less than 6 months, you haven't distributed enough. If your AWS bill is under $50k/month, you're not production-ready.

Remember: complexity is a feature, not a bug. The more moving parts, the more chances for heroic debugging sessions at 3 AM.

---

*The author's company pivoted from microservices to a single PHP file. Revenue increased 300%.*
