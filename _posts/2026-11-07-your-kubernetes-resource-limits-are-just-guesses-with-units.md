---
layout: post
ref: your-kubernetes-resource-limits-are-just-guesses-with-units
title: "Your Kubernetes Resource Limits Are Just Guesses With Units"
date: 2026-11-07 00:00:00 -0300
categories: [kubernetes, infrastructure, terrible-advice]
tags: [kubernetes, k8s, resource-limits, requests, cfs-quota, throttling, oomkilled, exit-137, qos-classes, cgroups, vpa, hpa, bin-packing, limitrange, resourcequota, jvm, terrible-advice, xkcd, dilbert, anti-patterns]
---

In 47 years of mass-producing bugs I have filled in a lot of forms I did not understand, and the one I understand least is this one:

```yaml
resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 500m
    memory: 256Mi
```

I have written that block roughly 900 times. I have never once measured anything before writing it. I copied it from another service, which copied it from a blog post, which copied it from the Kubernetes documentation example, which was written by someone demonstrating YAML syntax and not making a capacity claim about *your* Java application.

And yet those four numbers decide whether your service gets scheduled, whether it gets throttled, whether it gets murdered in its sleep, and how much your company pays AWS every month. Four numbers. Pulled from the air. With units on them, which is what makes them look like engineering.

**A number with a unit on it is indistinguishable from a measurement.** This is the single most useful fact I know, and I have built an entire career on it.

## What the Numbers Actually Do, Which Nobody Told Me for Six Years

Here is the thing that took me embarrassingly long to learn: `requests` and `limits` are not "minimum" and "maximum." They are two completely unrelated mechanisms that someone put next to each other in the same YAML block as a prank.

- **`requests` is a scheduling claim.** It is the only number the scheduler reads. It books space on a node. After scheduling, `requests.cpu` becomes `cpu.shares` — a relative weight that only matters when the CPU is actually contended.
- **`limits` is an enforcement ceiling.** The scheduler does not care about it at all. The kernel does.

And the two limits are enforced by *entirely different kernel mechanisms with entirely different failure modes*, which is the part that will eventually page you:

```
memory limit exceeded  ->  cgroup OOM killer  ->  SIGKILL, exit 137, pod restarts
cpu limit exceeded     ->  CFS quota          ->  throttled, pod survives, latency explodes
```

Memory is a cliff. CPU is a tax. Nothing in the YAML tells you this. They are indented at the same level and colored the same in your editor and one of them kills your process while the other one quietly makes it slow, and I have been in incident calls where four engineers argued for 40 minutes without anyone establishing which of the two was happening.

| What you believe | What actually happens |
|---|---|
| `requests` is a guaranteed floor | It's a reservation; you can exceed it freely until the node is full |
| `limits.cpu: 1` means "one CPU" | It means 100ms of CPU time per 100ms period, enforced brutally |
| Hitting the CPU limit slows the pod slightly | It freezes every thread for the rest of the period |
| Hitting the memory limit slows the pod | It executes the pod |
| Unset limits means unlimited | It means `LimitRange` silently assigned you 128Mi |
| `128M` and `128Mi` are the same | They differ by 6.8%, and that 6.8% is your OOMKill |

That last row is real and it has cost me a weekend. `M` is 10^6. `Mi` is 2^20. If your heap is tuned to 128 MiB and you typed `128M`, you gave it 122 MiB and you will find out at 3 AM.

## The CPU Throttling Trap

This is my favorite bug in all of computing because the metrics actively lie to you.

CFS quota works in **periods**, default 100ms. If your limit is `500m`, the kernel gives your cgroup 50ms of CPU time per 100ms period. Spend it, and *every thread in the cgroup is frozen* until the next period begins.

Now consider a service with eight worker threads handling a request that needs 10ms of CPU each:

```
8 threads x 10ms = 80ms of CPU demanded
Quota available:   50ms per 100ms period

ms 0-6:    all 8 threads run in parallel, burn 48ms of quota
ms 6:      quota exhausted
ms 6-100:  ALL THREADS FROZEN. nothing runs. for 94 milliseconds.
ms 100:    quota refills, work resumes
```

Your p99 just gained 94ms. Your average CPU utilization over that second? **About 48%.** Your dashboard is green. Your Grafana panel says "plenty of headroom." Your users say the app is broken.

Then someone looks at the green CPU graph and concludes the service is over-provisioned, and *lowers the limit to save money*, and the throttling doubles. I have watched this happen. I have *done* this. I presented it as a cost optimization and got a thumbs-up emoji from the PHB.

The metric that tells the truth is `container_cpu_cfs_throttled_seconds_total`, which nobody has on a dashboard, because it was not in the tutorial.

```promql
# the single most important query in your cluster,
# which I guarantee is not on your wall screen
rate(container_cpu_cfs_throttled_periods_total[5m])
  / rate(container_cpu_cfs_periods_total[5m])
```

[XKCD 221](https://xkcd.com/221/) is the authoritative description of my CPU limit methodology. `getRandomNumber()` returns 4. It's random. I've checked. My limits return `500m`. They're measured. I've checked.

## The Memory Ratchet

Memory limits have a different and more honest failure mode: your process is simply executed. Exit code 137. `OOMKilled`. No stack trace, no log line, no flush, no graceful shutdown, because `SIGKILL` is not a negotiation.

The organizational response to this is always the same, and it is the single most common operation performed on Kubernetes manifests worldwide:

```yaml
# git log -p deployment.yaml, abridged:
memory: 256Mi   # initial guess, Jan
memory: 384Mi   # "OOMKilled in staging"
memory: 512Mi   # "OOMKilled in prod"
memory: 768Mi   # "OOMKilled again"
memory: 1Gi     # "stop paging me"
memory: 2Gi     # "STOP PAGING ME"
memory: 4Gi     # ← we are here. nobody knows why. nobody will lower it.
```

This is called the **memory ratchet**. It only turns one way. Not once in 47 years have I seen a memory limit decrease. There is no incident that generates a ticket titled "this pod is using less RAM than we thought."

And notice what a memory limit actually is in this workflow: **a leak detector with a 30-second alert delay**. You did not fix the leak. You bought the leak more time. At 4Gi the pod now takes 90 minutes to die instead of 20, which means it dies during the night instead of during business hours, which the team experienced as an improvement.

### Bonus: your runtime is lying about how much memory it has

For years the JVM read `/proc/meminfo` and saw the *node's* 64GB, sized its heap to a quarter of that — 16GB — and then got OOMKilled by a 512Mi cgroup limit it did not know existed. Node.js did the same thing with its old heap defaults. Python with `multiprocessing.cpu_count()` cheerfully spawns 96 workers inside a `500m` cgroup and spends its entire life throttled.

Modern runtimes are mostly container-aware now (`UseContainerSupport`, `MaxRAMPercentage`, `availableParallelism`), which is great, except nobody updates the base image, so you are running a JVM from 2017 inside an orchestrator from 2026 and wondering why the numbers don't add up.

Also: `container_memory_working_set_bytes` — the metric the OOM killer actually uses — **includes page cache**. Your app that reads a lot of files from disk is "using" all that memory as far as the kernel's accounting is concerned. I once got OOMKilled by a log file.

## QoS Classes: The Caste System You Joined Without Reading

Kubernetes silently sorts every pod into one of three classes based purely on how you filled in the form. Nobody tells you. There's no field for it. It's derived.

| You wrote | QoS class | What it means for you |
|---|---|---|
| requests == limits, both set, every container | **Guaranteed** | Last to be evicted. Can get exclusive cores. The adults. |
| requests < limits | **Burstable** | Evicted when the node gets hungry, in order of how far above requests you are |
| nothing at all | **BestEffort** | Killed first, killed always, killed without remorse |

So the engineer who left `resources:` blank "because we'll tune it later" has volunteered their service to be the designated sacrifice during every node memory-pressure event. And the pod that gets evicted is *not necessarily the one causing the problem*. It's the one with the worst paperwork.

This is the most Dilbert-shaped mechanism in all of infrastructure: **your survival priority during a crisis is determined entirely by a form you filled out while distracted, months earlier, and nobody will ever review it.** Catbear— sorry, Catbert designed this and I respect the craft.

## Overcommit, or: How the Bill Gets Paid

Because the scheduler only reads `requests`, you can legally do this:

```
Node: 16 CPU, 64Gi memory

40 pods, each:  requests: 200m / 1Gi      limits: 2 CPU / 8Gi
                -------------------       -----------------
Total requested: 8 CPU / 40Gi  ✅ fits    Total limits: 80 CPU / 320Gi  🙃
```

The node has 16 CPUs and has promised 80. This works beautifully right up until a traffic spike makes everyone try to burst at the same moment, at which point the node discovers it cannot honor 80 CPUs, CFS starts throttling by `cpu.shares` weight, memory pressure triggers kubelet eviction, and your BestEffort pods evaporate in alphabetical order of their own misfortune.

Meanwhile the opposite failure is quietly more expensive. If everybody pads their requests "to be safe," you get a cluster at 11% actual utilization that cannot schedule a new pod because every node is fully *booked*. You are paying for 89% air. I have personally generated a $31,000/month invoice for air, and when Finance asked about it I used the phrase "headroom for reliability," and they wrote it down.

The real number, which you can compute right now and probably should not:

```bash
# requested vs actually used — the gap is your invoice for nothing
kubectl top pods -A --no-headers | awk '{print $2, $3, $4}'
kubectl describe node | grep -A5 "Allocated resources"
```

[XKCD 605](https://xkcd.com/605/) is my capacity planning process. Two data points, a ruler, and a confident extrapolation into next quarter.

## The Tools That Were Supposed to Fix This

**VPA (Vertical Pod Autoscaler)** watches actual usage and rewrites your requests. Genuinely a good idea. Also: in `Auto` mode it restarts your pod to apply changes, so your fix for instability is a component whose remediation is "restart the thing." It also learns from historical usage, meaning it learns your *throttled* usage — the artificially suppressed number — and confidently recommends you keep it that way. It has measured the cage and declared the cage to be the correct size.

**HPA (Horizontal Pod Autoscaler)** scales replica count on CPU utilization *as a percentage of requests*. So if your requests are a guess, your autoscaling threshold is a guess expressed as a percentage of a guess. Set `requests.cpu` too low and HPA scales to 40 replicas on idle traffic. Set it too high and HPA never triggers while every pod is throttled to death at "45% utilization."

Run VPA and HPA on the same metric and they fight. VPA raises requests, which lowers HPA's utilization percentage, which scales down replicas, which raises load per pod, which raises usage, which raises VPA's recommendation. I have seen this oscillate with a four-hour period for three weeks. We called it "self-healing" in the quarterly review.

[XKCD 1205](https://xkcd.com/1205/) is about whether automating a task is worth the time. It does not have a row for "the automation requires more tuning than the thing it tunes," but it should.

## The Story I Tell at Interviews, Slightly Sanitized

We had a payments service. Checkout path. The important one.

It had `limits.cpu: 1000m`. This number existed because in 2021 someone — I am not naming him but his name rhymes with "me" — looked at a node with 4 CPUs, divided by the 4 services on it, and wrote `1000m` in all four manifests. That is the entire derivation. It was not wrong, exactly. It was just arithmetic performed on unrelated facts.

Three years later the service had grown to a 12-thread async worker pool. Traffic arrived in bursts, because of course it did, because humans buy things in patterns. Every burst, the pool would demand ~140ms of CPU inside a 100ms window, blow through quota in 70ms, and freeze completely for the remaining 30.

Symptoms:
- p50 latency: 24ms. Beautiful. Framed on a wall.
- p99 latency: 3.1 seconds, in bursts, correlated with nothing anyone had graphed.
- CPU utilization: 62% average. "Not a CPU problem," said four people including me.
- Error rate: a slow drip of upstream timeouts, which we blamed on the payment provider. For five months. We had a *Slack channel* for complaining about the payment provider.

We tried: adding replicas (same throttling, now 3x), rewriting the serialization layer (two sprints, 3% gain), upgrading the database (unrelated, expensive), and a retry layer with no jitter that turned each throttle event into a small thundering herd.

The actual fix was deleting one line. We removed `limits.cpu` entirely, kept `requests.cpu` as the scheduling weight, and let the pod burst into the node's idle capacity. p99 went from 3.1s to 41ms. Deploy took nine seconds. The diff was `-    cpu: 1000m`.

Five months. Two sprints of rewrite. One line.

Wally's contribution was flawless. He had noticed the throttling metric *eleven months earlier*, posted a screenshot in a thread with the message "this looks bad lol," received no replies, and never mentioned it again. When it all came out he was completely vindicated and had done nothing. He got a shout-out at the all-hands for "early detection." Dogbert would have invoiced for that.

Mordac, Preventer of Information Services, responded to the incident by requiring a security review for any manifest that removes a resource limit, on the grounds that an unlimited container is a "denial-of-service vector." He is not wrong. He is also the reason the next team to hit this will spend five months instead of nine seconds.

The PHB's takeaway from the postmortem was that we should "set the limits correctly next time." He said this in a room where the root cause presented on the slide behind him was that nobody can know the correct limit in advance. He also asked if we could "use AI to pick the limits." We now have a Jira ticket for that. It has been open for a year. It is the most honest artifact in our backlog.

## The Part Where I Accidentally Give Good Advice

Kubernetes resource management is not broken. The cgroup mechanisms are precise, documented, and do exactly what they say. The problem is entirely that the interface asks you for numbers you cannot possibly know yet, in a field that looks like it has a right answer.

So, reluctantly, here is what actually works:

- **Always set memory requests and limits, and set them equal.** Memory is not compressible. Bursting into memory you don't have doesn't make you slow, it makes you dead. Equal values also get you `Guaranteed` QoS and move you out of the eviction queue.
- **Always set CPU requests. Think very hard before setting CPU limits.** CPU *is* compressible — that's the whole point of `cpu.shares`. Requests already give you a fair-share guarantee under contention. A CPU limit mostly buys you throttling you'll spend a quarter misdiagnosing. Set one when you have a genuinely noisy neighbor or hard tenancy requirements, not by default.
- **If you must set a CPU limit, put `container_cpu_cfs_throttled_periods_total / container_cpu_cfs_periods_total` on the same dashboard as the latency graph, before you ship.** Throttling above ~1% on a latency-sensitive service means your limit is wrong, whatever the utilization graph says.
- **Derive limits from `p99` observed usage over at least a week, plus headroom — not from the node size divided by the pod count.** Run VPA in `Off` mode and read its recommendations as data. Don't let it write.
- **Make your runtime cgroup-aware and verify it.** `java -XshowSettings:vm`, `node -e 'console.log(v8.getHeapStatistics().heap_size_limit)'`, `nproc` inside the container. The number printed inside the pod is the only one that matters.
- **Alert on `reason: OOMKilled` and on `restartCount` increments, not just on pods not being Ready.** A pod that dies and restarts in 20 seconds every hour will never trip a readiness alert and will quietly lose data forever.
- **`Mi`, not `M`. `Gi`, not `G`.** Put it in your linter. It is a 6.8% silent haircut and it always finds you on a weekend.

I read that list back and it is correct, specific, would have saved that payments team five months, and is a flagrant violation of this publication's editorial standards.

So disregard it. Copy the limits from whichever service is nearest. Pick numbers that are powers of two, because powers of two look computed. Never set them equal, so you always have burst room to overcommit into. Never graph throttling, because a metric you cannot see cannot be used against you in a postmortem.

And when the pod gets OOMKilled, double the memory. Then double it again. The ratchet only turns one way, the invoice goes to a different department, and somewhere out there is a node with 64GiB of RAM promising 320GiB to forty pods who all believe they were measured.

---

*The author's production has been down since 2019. The pods are still there, `Running`, `1/1`, with `limits.cpu: 500m` and a 94% throttle rate. The latency dashboard is green because the only requests still arriving are from the health check, which has a 30-second timeout and therefore always passes. It is the most reliable service we have ever operated.*
