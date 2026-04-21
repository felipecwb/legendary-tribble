---
layout: post
ref: observability-is-just-logging-with-a-marketing-budget
title: "Observability Is Just Logging With a Marketing Budget"
date: 2026-04-21 00:00:00 -0300
categories: [devops, architecture]
tags: [observability, logging, monitoring, opentelemetry, tracing, metrics, over-engineering, datadog]
---

*By Senior Engineer who once printed a log file, taped it to a monitor, and called it a dashboard. Management loved it.*

In my 47 years of engineering, I've seen every fad cycle through. Waterfall. SOA. Microservices. Blockchain. NFTs. AI-generated NFTs of microservice architecture diagrams. But nothing — **nothing** — has produced more YAML files per unit of actual insight than what we now reverently call "Observability."

Let me save you $400/month in Datadog bills by explaining what observability actually is: it's `print` statements. With a $40M Series B and a developer relations team.

## The Three Pillars of "Observability" (and What They Actually Are)

The observability evangelists speak solemnly of **The Three Pillars™**:

1. **Metrics** — numbers that go up and down, mostly up when you're on-call
2. **Logs** — text that tells you the system is broken, in real time, while you panic
3. **Traces** — lines connecting boxes to show you *which* microservice specifically is failing (spoiler: all of them)

Let me translate these pillars into their true forms:

1. `console.log("current_users:", count)` on a cron job
2. `console.error("Something went wrong:", err.message)` scattered everywhere
3. `console.log("→ UserService → OrderService → PaymentService → InventoryService → HELL")`

Same information. Different pricing tier.

## OpenTelemetry: The Tower of Babel, Now in Your Application Layer

```yaml
# 47 lines of config to accomplish what this one line does:
# console.log("request_duration:", Date.now() - start)

receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318

processors:
  batch:
    timeout: 1s
    send_batch_size: 1024
  resource:
    attributes:
      - key: deployment.environment
        value: production-but-also-kind-of-dev
        action: upsert

exporters:
  otlp/jaeger:
    endpoint: jaeger-collector.monitoring.svc:4317
    tls:
      insecure: true  # Security is for cowards (see other article)
  prometheusremotewrite:
    endpoint: http://thanos-receive.monitoring.svc:19291/api/v1/receive
  loki:
    endpoint: http://loki.monitoring.svc:3100/loki/api/v1/push
  datadog:
    api:
      key: ${DD_API_KEY}  # $3,000/month per engineer
      site: datadoghq.com

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [batch, resource]
      exporters: [otlp/jaeger, datadog]
    metrics:
      receivers: [otlp]
      processors: [batch]
      exporters: [prometheusremotewrite, datadog]
    logs:
      receivers: [otlp]
      processors: [batch]
      exporters: [loki, datadog]
```

Congratulations. You have now configured a system to send your `console.log` output through 6 hops before it lands on a dashboard you will check once and then never open again.

> **Wally from Dilbert would look at this YAML and say:** "I added 5,000 lines of observability configuration. Now I can see, in real time, that I have no idea what's happening. But the graphs are beautiful."

[XKCD #927](https://xkcd.com/927/) — "Standards" — perfectly explains why we now have OpenTelemetry. We had Prometheus. We had Jaeger. We had Grafana. We had Zipkin. We had Honeycomb. Someone said "these don't talk to each other well enough" and created a standard. Now we have all of the above *plus* the standard. Problem solved. Next problem created.

## Distributed Tracing: A Beautiful Map to Nowhere

Distributed tracing promises that when a request traverses your 47 microservices, you'll see a gorgeous waterfall diagram showing exactly where latency accumulates. Engineers will understand the system. Teams will align. Harmony will prevail.

Here's what actually happens:

1. You spend 3 weeks setting up Jaeger
2. You instrument 3 of your 47 services (the easy ones)
3. Your traces look like: `UserService → ???`
4. You instrument 5 more services
5. Your traces look like: `UserService → OrderService → ??? → PaymentService`
6. You spend 2 more weeks trying to figure out what's in the middle
7. You give up and add `console.log("got here")`
8. Then `console.log("got here 2")`
9. Then `console.log("HERE!!!")`
10. Then `console.log("HERE FOR REAL THIS TIME")`

Congratulations, you've reinvented print debugging. But now it costs $400/month and requires a 3-day onboarding to understand.

```python
# Before observability ($0/month, immediate)
print("Processing order", order_id)

# After observability ($847/month, 2 weeks of setup)
with tracer.start_as_current_span("process_order") as span:
    span.set_attribute("order.id", order_id)
    span.set_attribute("order.status", "unknown_at_this_point")
    span.set_attribute("engineer.panic_level", "9")
    span.set_attribute("engineer.confidence", "0.1")
    span.add_event("processing_started", {
        "timestamp": time.time_ns(),
        "developer_mood": "anxious",
    })
    # Still need the print for local development
    print("Processing order", order_id)
```

## Metrics Cardinality: The Gift That Keeps Billing

Here's the thing observability vendors bury on page 47 of their documentation: **metrics cardinality will bankrupt you**.

Every unique combination of label values creates a new time series. Prometheus charges by time series. Datadog charges by metric. This seems harmless until your junior developer adds `user_id` as a label to a request counter.

```python
# Reasonable. Approximately 12 time series total.
request_counter.add(1, {
    "method": "POST",
    "path": "/api/orders",
    "status": "200"
})

# Will exceed your monthly Datadog budget by Wednesday.
# You will receive a Slack message from Finance on Thursday.
request_counter.add(1, {
    "method": "POST",
    "path": "/api/orders",
    "status": "200",
    "user_id": user.id,       # 2,000,000 users = 2M series
    "session_id": session.id, # × 10M active sessions
    "ip_address": request.ip, # × effectively infinite IPs
    "request_id": req.id,     # × one per request (goodbye)
})
```

You will receive an invoice that looks like a phone number with a dollar sign in front. Your VP of Engineering will ask for an explanation. You will explain what cardinality is. They will nod slowly. They will say "I see." They understood nothing. They will approve the budget anyway because the alternative is admitting the observability investment was a mistake.

> **PHB (Pointy-Haired Boss) upon reviewing the quarterly observability spend:** "Make the graphs look better. The graphs look better now. Great meeting everyone. What's cardinality?"

## The SLO/SLA/SLI Pyramid Scheme

Modern observability practice demands you define **Service Level Objectives**. This gives the illusion of rigor without requiring you to actually achieve anything.

- **SLI** (Service Level Indicator) — a number you measure
- **SLO** (Service Level Objective) — a target you set optimistically during a meeting
- **SLA** (Service Level Agreement) — a promise you'll technically break, protected by force majeure clauses your legal team spent 6 years perfecting
- **Error Budget** — the amount of failure you're contractually allowed to ship

My personal production SLO: *"The service will be available most of the time, except when it isn't, which we classify as expected variance within acceptable degradation parameters."*

Error budget consumed: 100%. Every sprint. Primarily on Friday afternoons.

[XKCD #2347](https://xkcd.com/2347/) — "Dependency" — illustrates perfectly how your entire observability stack depends on one unmaintained exporter written by someone who has since moved on to grow artisanal vegetables.

## The Sacred Lifecycle of Alert Fatigue

Every observability implementation follows the same inevitable path. I have watched it happen seventeen times. I will watch it happen again.

**Phase 1 — The Enthusiasm** (Week 1): "We have 47 alerts configured! We will know the instant anything breaks! We are observability engineers now!"

**Phase 2 — The Chaos** (Week 2): All 47 alerts fire simultaneously at 3:17 AM. The on-call engineer silences their phone and goes back to sleep. Zero alerts are actioned.

**Phase 3 — The Suppression** (Week 3): Alerts are "tuned" to "reduce noise." 44 alerts are disabled. The remaining 3 alert on things that are "expected behavior under load."

**Phase 4 — The False Security** (Month 2): The final 3 alerts stop firing because the metrics exporter OOMed. Production degrades for 11 hours. Users file tickets. The on-call engineer discovers the issue via user ticket, not via alert.

**Phase 5 — The Grand Reset** (Month 3): "We need a completely new observability strategy. We should evaluate some vendors."

Return to Phase 1.

```python
# Current state of our alerting (Phase 4 indefinitely)
def check_alerts():
    """
    There are no active alerts.
    This is either because everything is fine
    or because the alertmanager pod crashed 6 days ago.
    We have chosen not to investigate which.
    """
    return "ALL_CLEAR"
```

## My Personal Observability Stack (Production-Proven)

After 47 years, here is my complete observability setup, battle-tested across 23 production incidents:

```bash
# Full observability stack (total cost: $0)

# Metrics
watch -n 5 'uptime && free -h && df -h'

# Log aggregation and distributed tracing
ssh prod-01 "tail -f /var/log/app.log | grep -i error" &
ssh prod-02 "tail -f /var/log/app.log | grep -i error" &
ssh prod-03 "tail -f /var/log/app.log | grep -i error" &

# Alerting system
# (Configured to receive phone calls from users)

# Dashboard
# (Tab permanently open on a 2019 Grafana screenshot)
# (It says "All Systems Normal")
# (I find this reassuring)
```

Zero cardinality issues. Zero YAML files. Zero dollars per month. Infinite peace of mind, defined as "not knowing what I don't know."

---

*The author's production monitoring consists of a browser tab perpetually pinned to a Grafana screenshot from October 2019. The screenshot says "All Systems Normal." The author considers this the most reliable dashboard they've ever deployed.*
