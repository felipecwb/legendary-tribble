---
layout: post
ref: load-balancing-is-overengineering
title: "Load Balancing Is Over-Engineering: One Big Server Is Enough"
date: 2026-03-09 08:05:00 -0300
categories: [architecture, infrastructure]
tags: [load-balancing, scaling, servers, infrastructure, devops]
---

Every architect's first instinct is to add a load balancer. "We need horizontal scaling!" they cry. After 47 years of server management, I know the truth: **one powerful server beats ten mediocre ones every time.**

## The Complexity Trap

Look at what "modern" architecture requires:

```
User → Load Balancer → Server 1 ─┐
                    → Server 2 ─┼→ Session Store → Database
                    → Server 3 ─┘
                    → Health Checks
                    → Auto-scaling Logic
                    → Service Discovery
                    → Configuration Sync
```

Now look at my architecture:

```
User → The Server → Database
```

One of these has 47 potential failure points. One has 3. Choose wisely.

## The Cost of Distributed Systems

| Component | Monthly Cost |
|-----------|-------------|
| Load balancer | $20 |
| Server 1 (small) | $50 |
| Server 2 (small) | $50 |
| Server 3 (small) | $50 |
| Redis for sessions | $30 |
| DevOps engineer time | $10,000 |
| **Total** | **$10,200** |

vs.

| Component | Monthly Cost |
|-----------|-------------|
| One beefy server | $200 |
| **Total** | **$200** |

[XKCD 1205](https://xkcd.com/1205/) says it all—the time spent on "scalable" infrastructure never pays off.

## The PHB Myth

The Pointy-Haired Boss in Dilbert loves buzzwords like "highly available" and "auto-scaling." What he doesn't understand is that availability comes from simplicity, not complexity. The fewer things that can break, the fewer things will break.

## The Vertical Scaling Solution

Why buy 10 small servers when you can buy 1 large server?

```bash
# Their setup:
10x t3.small (2 vCPU, 2 GB RAM each)
= 20 vCPU, 20 GB RAM
+ coordination overhead
+ network latency
+ configuration management
+ session synchronization

# My setup:
1x c5.4xlarge (16 vCPU, 32 GB RAM)
= More power, less complexity
+ No coordination needed
+ No network latency
+ No configuration drift
+ Sessions just work
```

## The "But What If It Goes Down?" Question

Your load balancer can also go down. Now you need redundant load balancers. And redundant connections between them. And failover logic. And health check services for the health check services.

My single server strategy:

```bash
# If server goes down:
1. Get alert
2. SSH into server
3. Restart service
4. Done

# Time: 3 minutes
# Complexity: None
# Sleep lost: 8 minutes (including waking up)
```

## Session Affinity: A Confession

You know what load balancer users end up doing? Sticky sessions. Route the same user to the same server. Congratulations, you've reinvented "one server per user group" with extra steps.

```nginx
# Load balancer config (after 6 months of "scaling")
upstream backend {
    ip_hash;  # Stick to the same server
    server 10.0.0.1;
    server 10.0.0.2;
    server 10.0.0.3;
}

# What you actually have: 3 single servers with a router in front
```

## The Database Bottleneck Truth

Here's what nobody tells you: your database is the bottleneck, not your application servers. Adding more app servers just means more connections to the same database, making it slower.

```
Before load balancer:
1 server → 10 DB connections → Database handles fine

After load balancer:
10 servers → 100 DB connections → Database crying
           → Connection pooling needed
           → pgbouncer deployment
           → More complexity
           → Same throughput (or worse)
```

## Real High Availability

Want real high availability? Here's my setup:

```bash
# Primary server: handles everything
server1.company.com

# "Backup" server: a cronjob that checks if primary is up
# If primary dies, I get a text message
# Then I fix it
# Uptime: 99.7% (good enough for any realistic SLA)
```

## The Auto-Scaling Lie

"But auto-scaling handles traffic spikes!"

Traffic spikes are predictable:

- Monday morning: people checking work emails
- Black Friday: you knew this was coming
- Product launch: you planned this
- Viral moment: by the time auto-scaling kicks in, it's over

Plan capacity ahead. Don't trust automation to save you in real-time.

## My Scaling Philosophy

```python
def handle_scaling_needs():
    current_cpu = get_cpu_usage()
    
    if current_cpu > 80:
        # Step 1: Optimize code
        fix_that_n_plus_one_query()
        
    if current_cpu > 80:
        # Step 2: Add caching
        add_redis_maybe()
        
    if current_cpu > 80:
        # Step 3: Bigger server
        upgrade_instance_size()
        
    if current_cpu > 80:
        # Step 4: You're Twitter now
        # Maybe consider load balancing
        # But probably just optimize more
        pass
```

---

*The author's production server is a single machine named "GODZILLA" with 128 GB RAM. It has been running for 7 years without horizontal scaling. The power bill is concerning.*
