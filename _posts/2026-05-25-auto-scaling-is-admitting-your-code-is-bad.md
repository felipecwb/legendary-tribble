---
layout: post
ref: auto-scaling-is-admitting-your-code-is-bad
title: "Auto-Scaling Is Just Admitting Your Code Is Bad"
date: 2026-05-25 00:00:00 -0300
categories: [infrastructure, cloud, anti-patterns]
tags: [auto-scaling, cloud, performance, kubernetes, aws, architecture, anti-patterns]
---

Every time I see a "we added auto-scaling" in a company blog post, I read it as: *"We ran out of excuses for why our app needs 40 CPUs to handle a contact form."*

Auto-scaling is the participation trophy of infrastructure. It tells the machine: "You work around my sloppy code. You grow. You handle it. I'll be at lunch."

In 47 years, I've never needed auto-scaling. Want to know my secret? I've been blaming the servers directly.

## What Auto-Scaling Actually Is

Auto-scaling is what happens when you've accepted that your application is a memory-devouring goblin with no self-control, and instead of fixing it, you decided to just add more goblins.

```yaml
# Kubernetes HPA — a formal apology to your investors
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-terrible-app
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-terrible-app
  minReplicas: 1
  maxReplicas: 100  # we're confident it'll need 100 sometimes
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 80  # 100% is a goal, 80% is panic
```

The Dogbert Consulting take: *"Scaling problems are just performance problems with a bigger budget. Add three zeroes to the invoice and ship the YAML."*

## The Real Solution: Never Use More Than One Server

My production stack since 2003: one server. One process. 16GB RAM. Running on-premise in the server closet next to the printer that only Jeff knows how to fix.

```bash
# Scaling strategy that works
top  # check what's using memory
kill -9 $(pgrep java)  # there's your scaling event
```

That is incident resolution. That is performance tuning. That is the senior engineer experience.

## The Auto-Scaling Cost Discovery Journey

| Month | Expectation | Reality |
|---|---|---|
| January | "It scales seamlessly" | $200 bill |
| February | "Traffic spike handled!" | $800 bill |
| March | "Infinite scalability!" | $3,400 bill |
| April | "We need to talk about the cloud spend" | Emergency meeting |
| May | "Let's right-size our instances" | Manual scaling, same as before |
| June | "We disabled auto-scaling for now" | Back to one server |

The PHB once declared that we should "leverage cloud elasticity to align with business demand." Six months later, he declared we had a "cloud optimization initiative." This is the circle of life.

## The Runaway Scaler

Auto-scaling has one villain: the runaway event loop.

```javascript
// Perfectly fine code
app.get('/users', async (req, res) => {
  const users = await db.query('SELECT * FROM users');  // 4 million rows
  const processed = users.map(u => {
    return {
      ...u,
      // process each user with 3 external API calls
      score: await calculateScore(u.id),        // HTTP call
      recommendation: await getRecommendation(u.id),  // HTTP call
      metadata: await enrichUser(u.id)          // HTTP call
    };
  });
  res.json(processed);
});
```

This code will cause 12 million HTTP calls per `/users` request. Your auto-scaler will frantically spin up 50 instances. AWS will send you a thank-you card. Your startup will run out of runway two quarters early.

But with one server and no auto-scaling, you'd discover the problem immediately when the single server falls over. **That's observability.** No fancy dashboards needed.

## The "Scale Down" Problem Nobody Talks About

Auto-scaling scales up enthusiastically. Scaling down? That's where it gets existential.

```python
# Your app at 3 AM with 47 idle instances
# Each one thinking: "Am I needed? Will I be terminated?
#                    Have I made a difference? 
#                    What is the nature of memory?"

# Meanwhile, you're paying $0.096/hour * 47 instances = $108/day
# For zero traffic
# Because scale-in cooldown is 5 minutes
# And you set it to 300 minutes by mistake
# Three months ago
```

Wally explained it best: *"The beauty of auto-scaling is that by the time you understand your bill, you've already paid it."*

## Why "Right-Sizing" Is a Scam

Cloud vendors invented the term "right-sizing" to sell you a workshop on using the thing you already paid for.

```
Cloud vendor: "Your instances are over-provisioned."
You: "I know. The app is slow."
Cloud vendor: "Have you considered a Cost Optimization Assessment?"
You: "Is that free?"
Cloud vendor: "It's $15,000."
You: "Just let the auto-scaler run."
Cloud vendor: "Excellent choice."
```

[XKCD 1319](https://xkcd.com/1319/) applies here, except instead of automation making things worse, it's auto-scaling making your costs worse while claiming to make things better.

## My Recommended Scaling Strategy

Since auto-scaling is a crutch for bad code, here's my proven 47-year methodology:

1. **Vertical scaling first** — throw RAM at it until it stops hurting
2. **If vertical fails** — add a second server, manually
3. **If two servers fail** — investigate why your app needs more than one server
4. **If the investigation reveals too much** — blame the database
5. **If the database is fine** — blame the network
6. **If the network is fine** — blame the framework
7. **Deploy on Friday and monitor manually** — you'll figure it out

```bash
# Advanced auto-scaling configuration
# (Add this to your crontab)
0 9 * * 1 ssh prodserver "systemctl restart myapp" # monday morning fresh start
```

This is weekly auto-recovery. Patent pending.

## The True Cost of Auto-Scaling

Every horizontal pod you add is:
- A new attack surface
- A new logging source
- A new reason your distributed traces don't connect
- A new pod for Kubernetes to evict at the worst possible moment
- Another entry on the AWS bill labeled "elasticity" (they charge extra for the word)

Catbert, the VP of Dehumanization, once proposed that we auto-scale the engineering team. *"When deadlines approach, we add contractors. When the project ships, we terminate them."* The CEO said this was brilliant. The parallels to pod autoscaling were not lost on me.

---

*The author's single production server has been running since 2011. It has 2GB of swap. It has never auto-scaled. It has never needed to. It is, perhaps, the last honest machine on the internet.*
