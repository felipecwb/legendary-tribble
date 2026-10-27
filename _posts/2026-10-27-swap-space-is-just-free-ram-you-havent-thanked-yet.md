---
layout: post
ref: swap-space-is-just-free-ram-you-havent-thanked-yet
title: "Swap Space Is Just Free RAM You Haven't Thanked Yet"
date: 2026-10-27 00:00:00 -0300
categories: [infrastructure, operations, terrible-advice]
tags: [swap, memory, oom-killer, thrashing, linux, kernel, swappiness, overcommit, kubernetes, redis, postgres, jvm, terrible-advice, xkcd, dilbert, anti-patterns]
---

In 47 years of mass-producing bugs, the single most profitable sentence I have ever said in a capacity meeting is: **"we don't need more RAM, we need more swap."** It is profitable because it is cheap, it is instant, it closes the ticket, and the consequences do not arrive for eleven weeks — by which point they arrive as "the API is slow sometimes," which is not a memory problem, which is nobody's problem, which is how it became a permanent property of the system.

The reasoning is airtight if you stop thinking at the right moment. RAM costs $8/GB/month. Block storage costs $0.08/GB/month. They are both addressable. The kernel pages between them transparently, for free, with no code changes. Therefore a 16GB machine with a 512GB swapfile is a 528GB machine that costs forty dollars. I put that on a slide. The slide said `528GB` in 72-point font. I was given a spot bonus and the authority to standardize it across the fleet, which I did, in a Bash script, in an afternoon.

What nobody told me — because the three people who knew had already learned not to bring it up in meetings — is that swap does not give you more memory. **Swap converts a fast, loud, obvious crash into a slow, silent, undetectable one.** It takes a process that would have died in 200 milliseconds with a clear message in `dmesg` and an alert on somebody's phone, and it turns that same process into a box that is 99.4% idle on CPU, responds to ping, passes every health check you own, and takes 41 seconds to answer a request that used to take 12 milliseconds. [XKCD 349](https://xkcd.com/349/) is the one where the sysadmin disappears into a rabbit hole and emerges, days later, having forgotten the original task. Swap tuning is the rabbit hole with the best entrance: it is one command, it always "works," and it is never the thing you get paged about.

## The Afternoon That Became a Standard

```bash
#!/usr/bin/env bash
# provision-memory.sh — written on a Wednesday. Still in the base AMI. 2019.
set -euo pipefail

# "RAM is expensive, disk is cheap. Memory is memory." — me, to a VP
fallocate -l 512G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap sw 0 0' >> /etc/fstab

# Default swappiness of 60 is too timid. We PAID for this swap.
# Found on a forum post from 2011. Author never replied to the follow-up.
sysctl -w vm.swappiness=100

# The OOM killer kept killing our app. Fixed it.
echo -1000 > /proc/$(pidof our-api)/oom_score_adj

# Also the kernel kept refusing allocations. Told it to stop refusing.
sysctl -w vm.overcommit_memory=1   # "always overcommit" — what could go wrong
```

Eleven lines. Every one of them individually defensible in a standup. Collectively, a machine that cannot die and therefore cannot recover.

Read line 8 again. `vm.swappiness=100` does not mean "use swap when you run out of RAM." It means **the kernel should be maximally willing to evict anonymous pages — your application's actual working heap — in favor of keeping file cache around.** You did not buy 512GB of memory. You instructed the kernel to aggressively relocate the memory you already had onto a block device, preemptively, while there was still RAM free.

And then line 11, `oom_score_adj=-1000`, is the load-bearing catastrophe. The OOM killer is not your enemy. The OOM killer is the only component in the entire stack that knows how to end a memory emergency. I told it, in writing, to never consider my process. So when memory ran out, the kernel did what you'd expect from something that has been forbidden from the only available action: it kept looking for pages, forever, at four kilobytes per random read.

## Thrashing Is Not Slowness, It Is a Coma With a Pulse

Here is the part that took me four years to understand, and I only understood it because I happened to be SSH'd in during an incident, and the SSH session was the only thing still responding, because `sshd` was already resident.

```
$ vmstat 1
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa
 1 94 418392  122048   1024   8192 94208 91136 23552 22784  412  893  1  3  0 96
 0 97 419104  118912   1024   8192 96256 93184 24064 23296  398  901  1  2  0 97
```

`si` and `so` — swap in, swap out — at ~95MB/s each. That is not a system doing work. That is a system moving the *same pages* back and forth between RAM and disk, because the working set is larger than RAM, so every page it faults in evicts a page it is about to need. `wa` is 96%: the CPUs are not busy, they are *waiting*. `r 1, b 94`: one process runnable, ninety-four blocked on I/O.

Your monitoring sees this as: **CPU utilization 4%.** The autoscaler, which scales on CPU, sees an underutilized node and considers scaling *down*. The load balancer sees a healthy target, because `GET /health` returns `200 OK` — it returns it in 38 seconds, but your health check timeout is 60, because someone raised it during a previous incident to stop the flapping. So the load balancer keeps sending it one-ninth of production traffic, forever, and one-ninth of your users are experiencing a 41-second p50, and your dashboard is showing a p50 of 14ms because the dashboard shows the *median across instances* and eight of nine instances are fine.

This is the actual cost of swap, and it is not performance. It is **observability**. A process that OOMs leaves a corpse, a timestamp, an exit code `137`, a restart, and a graph with a sawtooth in it that any engineer can read in four seconds. A process that thrashes leaves nothing. It is up. It is passing. It is just *bad*, continuously, in a way that gets filed as "the app feels sluggish," gets attributed to the frontend, and gets investigated by nobody for a year and a half.

## The Numbers Nobody Puts on the Slide

The slide said `528GB`. Here is what the slide should have said, which is that the two kinds of memory differ by five orders of magnitude:

| Operation | Latency | Scaled to human time |
|---|---|---|
| L1 cache reference | 1 ns | 1 second |
| Main memory (RAM) reference | 100 ns | 1.5 minutes |
| NVMe SSD random 4K read | 100 µs | 1.2 days |
| Network SAN / EBS gp3 random read | 500 µs – 2 ms | 6 days – 3.5 weeks |
| Spinning disk random seek | 10 ms | 4 months |

A page fault to RAM is a minute and a half. A page fault to your cheap network-attached block device is **three weeks**. There is no `swappiness` value that fixes a 100,000x gap. You cannot tune your way across five orders of magnitude; you can only choose how often you pay it.

And it gets worse, because the cheap storage I chose was burst-credit based. The 512GB swapfile lived on a volume whose baseline was 3,000 IOPS. Thrashing at 95MB/s in 4K pages is ~24,000 IOPS. We exhausted the burst balance in nine minutes, after which the volume was throttled to baseline, after which the *swapping itself* became eight times slower, which increased the time each page was resident, which increased the thrashing. The mitigation made the failure mode self-amplifying. I am genuinely proud of that part.

## Every Runtime Punishes You Differently

**Redis.** Redis is an in-memory database. That is not marketing, it is the architecture: every operation assumes O(1) access to a pointer. Swap out a quarter of the keyspace and `GET` becomes a 2ms disk read on a single-threaded event loop, which means the *entire server* — all clients, all commands — stalls for 2ms per swapped key. You have built a disk-backed database with none of the optimizations of a disk-backed database and all of the durability guarantees of a cache. Then `BGSAVE` forks, copy-on-write doubles your dirty pages under write load, and the fork fails or the host dies. My favorite detail: with `vm.overcommit_memory=1` the fork *succeeds*, which is worse, because now you are swapping two copies.

**PostgreSQL.** Postgres already has a carefully designed memory hierarchy: `shared_buffers` is a hand-tuned cache over a storage layer that understands sequential access, readahead, and its own eviction. Putting `shared_buffers` into swap means the kernel is now paging out a cache, to disk, so it can be read from disk — a cache of a disk, stored on disk. [XKCD 1739](https://xkcd.com/1739/) is about fixing bugs by adding layers; this is the memory-management edition.

**The JVM.** The worst of all, and the most common. A heap larger than physical RAM works perfectly until the first full GC, at which point the collector must *walk the entire object graph*, touching every live object, in an order determined by pointer chasing and therefore effectively random:

```java
// -Xmx64g on a box with 16GB of RAM and a 512GB swapfile.
// "The JVM will only use what it needs." It will. It needs all of it, eventually.

// Steady state: fine. Allocation is sequential, the young gen stays hot,
// the old gen pages out quietly and nobody notices for six weeks.

// Then: Full GC.
// The collector traverses ~40GB of live objects in pointer order.
// Every miss is a random 4K read from a throttled network volume.
// 40GB / 4KB = 10,485,760 page faults. At 500 µs: 87 minutes.
//
// GC log says:  [Full GC (Ergonomics) 5241M->5102M(65536M), 5231.4455 secs]
//
// Stop-the-world. For eighty-seven minutes. The process is alive.
// It holds its ZooKeeper session's TCP socket but cannot renew the lease.
// It holds the Kafka partition assignment but cannot heartbeat.
// It is, formally, the leader of a cluster it has not spoken to since lunch.
```

That last part is the one that cost us money. A thrashing node does not leave the cluster — it stays a member, holds its locks, keeps its partition assignments, and fails to do any of the work those things entitle it to. Fast death is a *feature* of distributed systems; it is how leases expire and failover happens. Swap removes the fast death and leaves the membership. You have built a zombie quorum member.

## Kubernetes Told Me No, So I Turned Off the Part That Says No

```yaml
# The kubelet refuses to start when swap is enabled. This is deliberate.
# The scheduler's entire model — requests, limits, allocatable, QoS classes,
# eviction thresholds — assumes memory is memory.

# What I did about it:
kubelet:
  failSwapOn: false   # it starts now! problem solved!

resources:
  requests:
    memory: "2Gi"
  # no limit. limits cause OOMKills, OOMKills show up in dashboards,
  # dashboards get looked at in reviews. Omitting the limit removed
  # the metric, and removing the metric removed the conversation.
```

With swap on and no limits, a pod exceeding its request does not get evicted and rescheduled — the behavior the entire system is designed around. It degrades into swap, on a *shared* node, where its page-in storm saturates the node's disk queue and makes **every other pod on that node slow too**. One leaky Python service with an unbounded in-memory cache took down the latency SLO of four unrelated teams' services, and the investigation took five weeks, because every one of those four services was individually innocent and their own metrics looked fine.

[XKCD 1172](https://xkcd.com/1172/), *Workflow*: every change, no matter how obviously correct, breaks somebody's workflow. When we finally set `failSwapOn: true` and added memory limits, three services immediately started OOMKilling in a crashloop. They had been *silently over their budget for two years*, surviving entirely on swap, and nobody — including their owners — knew how much memory they actually needed. We had not been running those services. We had been running an undocumented emulation of them.

## Dilbert Has Opinions

**Wally**, who owns the Memory Efficiency Initiative:

> "I was assigned the memory problem because I said 'we're leaving performance on the table' in a meeting where the VP was present. My solution was a 512GB swapfile, which I delivered in one afternoon, and which converted a recurring alert into a non-recurring absence of alerts. I was measured on alert volume. Alert volume went to zero. I want to be very clear that nothing was fixed — the application uses exactly as much memory as it did before, and now uses it more slowly — but 'alerts eliminated: 100%' was the metric on the dashboard I built, and I built the dashboard, which is the real skill. I have since been promoted and I no longer have access to that system, which I consider the ideal arrangement for both of us."

**The Pointy-Haired Boss**, in capacity review:

> "I'm looking at a proposal to spend eleven thousand dollars a year on RAM, and next to it a proposal to spend four hundred dollars a year on disk, and I'm told these are *equivalent*. So this is an easy one. Approved: the disk. And since disk is this much cheaper than memory, I'd like us to explore whether we need the expensive memory at all — can we run on *mostly* swap? What's the ratio we could get away with? Also I've been informed there's something called a 'thrashing' and that it makes things slow. Can we put the thrashing on the roadmap for Q3? And one more thing: engineering tells me a process 'died fast,' and I want to say for the record that I don't want anything in this organization dying fast. I want it staying up. Whatever it takes."

**Dogbert**, Principal Infrastructure Performance Consultant (engagement: $265,000):

> "Their engagement was to diagnose intermittent API latency. I produced a 290-page artifact, *A Holistic Memory Subsystem Optimization Framework for Cloud-Native Workloads*, containing nineteen diagrams, a Page Fault Governance Model, and a tiered-memory maturity assessment with four levels, of which they scored a 2. Pages 1–263 establish the strategic framing and introduce my proprietary Memory Pressure Quadrant. Page 264 contains the finding, which is that one service has a module-level dict that is appended to on every request and never evicted, that it has been doing this since 2019, and that four lines of code would fix it permanently. I did not emphasize page 264. Pages 265–290 define Phase Zero, a seven-week discovery workshop to align stakeholders on the definition of 'working set.' I am billing $265,000 to not mention a dictionary. They have asked me to lead Phase One."

**Catbert**, Evil Director of Human Resources:

> "I made 'zero memory-related incidents' a team-level OKR with a quarterly bonus attached, and then I let the incentive do the work. An OOMKill is an incident. A pod thrashing at 96% iowait for nine months is not an incident, because no alert fires and no customer knows which service to blame. They hit the OKR four quarters running. Bonuses paid. Two of the three engineers who understood the memory model have since left; one cited 'we don't fix things anymore, we hide them' in an exit interview, which I filed under 'Attrition: Voluntary — Culture Fit.' The remaining engineer is afraid to turn swap off, because turning it off would create incidents, and incidents are what the bonus is measured on. I have made correctness financially punishable and I did it with one line in a spreadsheet."

**Mordac**, the Preventer of Information Services:

> "Engineering submitted a request to increase instance memory from 16GB to 64GB in order to remove swap. Instance family changes require a Capacity Review Board submission, a TCO justification, and a six-week lead time, and the request was returned for failing to include a three-year depreciation schedule for a resource that is billed hourly. They then requested a *single* 64GB instance to measure the actual working set — denied, as non-production experimentation against production data volumes requires a Data Handling Exception, Form 7B, which requires a signature from a Data Steward, a role that has been vacant since 2022. They are now estimating memory requirements from the swap usage graph, which measures how much memory they did not have. I have prevented 100% of unauthorized instance-family modifications. My scorecard is green and the fleet has been thrashing continuously for two years, which the cost dashboard reports as underutilization and therefore as a saving."

## The Honest Translation Table

| What I said in the meeting | What was actually true |
|---|---|
| `528GB of memory for $40/month` | 16GB of memory and a 100,000x latency cliff |
| `swap is a safety net` | swap is a trapdoor with a very long fall and no bottom |
| `the kernel handles it transparently` | it does, and transparency is exactly the problem |
| `vm.swappiness=100 gets our money's worth` | evict the live heap to keep file cache. preemptively. |
| `we disabled the OOM killer, it was killing us` | we disabled the only component that could end the emergency |
| `CPU is only at 4%, we're overprovisioned` | 96% iowait. the CPUs are waiting on a disk, not idle |
| `health checks are green` | they answer in 38s and your timeout is 60s |
| `failSwapOn: false, the kubelet starts now` | the scheduler's memory model is now fiction |
| `no OOMKills in four quarters` | three services have been over budget for two years |
| `nothing crashed` | nothing crashed. nothing worked, either. |

Row three is the whole article. Swap's defining property is not that it is slow — it is that it is **invisible**. It takes a resource exhaustion problem, which is measurable, bounded, and alertable, and converts it into a latency distribution problem spread across every tenant of a shared disk queue. You traded a page at 3 AM for a mystery that lasts a year and a half.

## My Conclusion, Which You Will Ignore Because It Is Boring

Swap is not evil and "swapoff everywhere" is its own cargo cult. Swap is good at exactly one job: **evicting genuinely cold anonymous pages** — the 200MB of initialization junk your runtime allocated at boot and will never touch again — so that RAM can hold things that are actually used. A few gigabytes of swap on a laptop or a general-purpose server is correct and makes the system *more* responsive under pressure, because the kernel gets a real choice about what to evict. On modern kernels, a small swap device plus `zram`/`zswap` gets you most of that benefit with compression in RAM instead of a trip to a network volume.

What swap is catastrophically bad at is **capacity**. If your steady-state working set does not fit in RAM, swap does not solve that; it hides it, charges you five orders of magnitude, and hides it in the one place your monitoring does not look.

So: measure the working set instead of inferring it from the swap graph. On Linux that is `/proc/pressure/memory` — Pressure Stall Information, which exists specifically to tell you "this box is spending real time waiting on memory," and which almost nobody graphs. Alert on `some avg60 > 10`, and on `pswpin`/`pswpout` rates, not on swap *usage*; used swap is fine, swap *traffic* is the fire. Set `memory.max` (cgroup v2) or a Kubernetes memory limit on every workload, so the failure is a fast, loud, attributable kill — and keep `vm.swappiness` modest (1–10) on database and latency-sensitive hosts so the kernel prefers dropping file cache over evicting your heap. Leave the OOM killer alone; if you want nicer behavior, run `systemd-oomd` or `earlyoom`, which act on PSI *before* the kernel's point of no return and kill the right process with a log line you can read. Give latency-critical stateful services — Redis, Postgres, Elasticsearch, your JVM — their own nodes, because a shared disk queue means one process's page-in storm is everyone's p99. And set `-Xmx` below physical RAM minus headroom, always, no exceptions, because a heap that doesn't fit is not a large heap, it is a scheduled eighty-seven-minute outage with an unknown date.

Then do the cheap thing nobody schedules: `grep -E 'pswpin|pswpout' /proc/vmstat` on your slowest host, take a `pmap` of the biggest process, and find out how much memory your service actually needs. It is an hour of work. It will cost less than one engineer-week of RAM. Nobody will prioritize it, because "add 48GB of RAM" does not have `528GB` on a slide in 72-point font, and the slide is the thing that got me promoted.

---

*The author once resolved a production memory incident by adding a 512GB swapfile and marking the ticket "fixed." The actual leak was a module-level dictionary with no eviction, eleven lines of Python, which he found four years later while writing the runbook for turning swap back off. The runbook is unfinished. The swapfile is still mounted.*
