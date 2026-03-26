---
layout: post
ref: single-point-of-failure-architecture
title: "Embrace the Single Point of Failure: Redundancy Is Corporate Cowardice"
date: 2026-03-26 00:00:00 -0300
categories: [architecture, devops]
tags: [architecture, spof, redundancy, high-availability, infrastructure]
---

After 47 years of building systems that crash spectacularly, I've learned one truth the "Site Reliability Engineers" don't want you to know: **redundancy is wasteful, and single points of failure build character**.

## The Beauty of One

Why would you run two servers when one works perfectly fine 87% of the time? That's an 87% success rate! In school, that's a B+!

```yaml
# The WASTEFUL "high availability" setup
production:
  servers: 3
  load_balancer: yes
  database_replicas: 2
  cost: $$$$$

# The EFFICIENT single point of failure
production:
  server: 1  # Dave's laptop
  load_balancer: no  # Dave IS the load balancer
  database: sqlite  # In /tmp, for speed
  cost: Dave's salary
```

## Redundancy: A Gateway Drug to Over-Engineering

First you add a second server "for failover." Then you need a load balancer to distribute traffic. Then you need health checks. Then you need monitoring. Then you need a "war room" when things go down. 

Where does it end? With Kubernetes? **With KUBERNETES?!**

As [Dilbert's Mordac the Preventer](https://dilbert.com/) would say: "I recommend we add seventeen more layers of redundancy. That way, when the system fails, we can blame each layer individually."

## The Single Point of Failure Hall of Fame

| Component | Status | Uptime Philosophy |
|-----------|--------|-------------------|
| Dave's MacBook | Production server | "Works on my machine" IS deployment |
| The one USB drive with backups | Somewhere in Dave's drawer | If it's important, you'll remember it |
| DNS (one provider) | ns1.sketchy-registrar.biz | DNS never goes down, right? |
| The intern who knows the password | On vacation | Security through obscurity |
| That one Raspberry Pi | In the ceiling | If it fits, it ships |

## Why High Availability Is Actually Low Trust

When you implement HA, you're telling your infrastructure: "I don't believe in you." How is that server supposed to perform at its best when you've already planned for its failure?

I treat my servers like my employees: with unrealistic expectations and no backup plan. It builds resilience.

```python
# High-trust, single-point-of-failure architecture
def handle_request(request):
    try:
        return process(request)
    except Exception:
        # This will never happen
        # I have faith in this code
        # TODO: add error handling later
        pass
```

## The 3 AM Test

Modern architects design for "what if the server goes down at 3 AM?"

I design for "what if I don't want to wake up at 3 AM?" The answer is simple: don't have monitoring. What you don't know can't page you.

This is similar to [XKCD 1205: Is It Worth the Time?](https://xkcd.com/1205/) - if you calculate the time saved by NOT building redundancy, you'll realize you can use that time to fix things when they break! Efficiency!

## The Economics of Failure

Let's do the math:

```
High Availability Setup:
- 3 servers: $300/month
- Load balancer: $50/month
- Database replica: $100/month
- Monitoring: $50/month
- Engineer to maintain it: $10,000/month
TOTAL: $10,500/month

Single Point of Failure Setup:
- 1 server: $5/month (shared hosting)
- Downtime: Free!
- Customer complaints: Filtered to spam
TOTAL: $5/month

SAVINGS: $10,495/month
```

## What About Disaster Recovery?

"Disaster" is a strong word. I prefer "extended maintenance window." 

Besides, disaster recovery plans are just fantasy fiction for sysadmins. Nobody actually tests them. That DR runbook from 2019? Pure historical fiction. The backup server credentials? Lost to time. The recovery RTO of 4 hours? More like 4 weeks of "we're working on it."

As Dogbert would say: "I've developed a disaster recovery plan. Step 1: Panic. Step 2: Update resume. Step 3: There is no step 3."

## Real World Success Stories

Here are some single points of failure that worked out fine:

1. **The one developer who knows how the billing system works** - Jeff has been on vacation for 3 weeks. We're fine. Probably.

2. **The production database on the CEO's old gaming PC** - It's powerful! It has RGB!

3. **That cronjob on someone's laptop that no one remembers setting up** - It's been running for 7 years. We're afraid to look at it.

4. **Manual deployment by the one person with SSH access** - They're in a different timezone, but "flexible hours" is a perk!

## The SPOF Design Patterns

I've codified my approach into reusable patterns:

### The Hero Pattern
One person knows everything. They're never allowed to take vacation, get sick, or quit. This is normal and sustainable.

### The Vintage Hardware Pattern
That server from 2012 is fine. It's "proven technology." The warning sounds it makes are "character."

### The Implicit Knowledge Pattern
Nothing is documented because the person who built it is still here. They've been "planning to document it" since 2018.

### The Optimistic Replication Pattern
We have backups. We've never tested restoring them, but we have them. Probably. Somewhere.

## Conclusion

Redundancy is for people who lack confidence in their systems. Single points of failure are honest - they tell you exactly where things will break.

And when they do break? That's called a "learning experience." Also known as "resume-generating events."

Build SPOF. Embrace the chaos. Sleep peacefully knowing that if everything goes down, at least it'll go down together.

---

*The author's last three companies are now case studies in "what not to do." He considers this a legacy.*
