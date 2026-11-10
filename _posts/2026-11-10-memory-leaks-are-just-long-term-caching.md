---
layout: post
ref: memory-leaks-are-just-long-term-caching
title: "Memory Leaks Are Just Long-Term Caching"
date: 2026-11-10 00:00:00 -0300
categories: [performance, operations, terrible-advice]
tags: [memory-leaks, garbage-collection, oomkilled, kubernetes, cgroups, rss, heap, jvm, nodejs, python, golang, glibc, malloc, fragmentation, jemalloc, liveness-probe, restart-driven-development, terrible-advice, xkcd, dilbert, anti-patterns]
---

In 47 years of mass-producing bugs, the most profitable thing I ever shipped was a `HashMap` that nobody ever removed anything from.

```java
public class UserService {
    // "temporary" — added 2019-03-14, still here
    private static final Map<String, Session> SESSIONS = new HashMap<>();

    public Session login(String userId) {
        Session s = new Session(userId);
        SESSIONS.put(userId, s);   // no eviction
        return s;                   // no TTL
    }                               // no size bound
}
```

Four lines. Zero `remove()` calls. The team called it a memory leak. I called it **a cache with an infinite retention policy**, which is the same data structure described by someone with ambition.

That reframing survived three postmortems, one architecture review, and a VP. It is still in production. It is the single most reliable component in the system, because it has never once evicted anything a user needed.

## The Core Doctrine: There Is No Such Thing As A Leak, Only Insufficient Patience

A leak, technically, is allocated memory that is unreachable and never freed. That's the C definition — you lost the pointer, the allocator can't reuse the block, the memory is gone until the process dies.

In every managed runtime, that is **not what happens**. The JVM, CPython, V8, and the Go runtime all free unreachable memory automatically and correctly. The GC is not broken. The GC has never been broken. What you have is memory that is *perfectly reachable*, from a root you forgot about, being retained with total fidelity by a collector doing exactly its job.

| What you call it | What it actually is | Who is at fault |
|---|---|---|
| "GC leak" | A live reference you forgot | You |
| "The GC isn't collecting" | The GC collected everything unreachable | You |
| "Memory leak in the library" | Your callback registered into a global | You |
| "RSS keeps growing" | Fragmentation, arenas, or page cache | The allocator, occasionally |
| "We need more heap" | You need fewer roots | You, but louder |

This is the foundational insight of my career: **the garbage collector is a very expensive, very fast program that spends all day proving your data structures are important to you.**

So stop fighting it. Agree with it. Every object you retain, you retain *deliberately*. Rename the field and move on.

## Terrible Pattern #1: The Cache That Is Only A Cache In The Variable Name

A cache has three properties: a size bound, an eviction policy, and a hit-rate you measure. Remove any one of them and you have a leak with good PR.

```python
_cache = {}   # it says cache right there

def get_user_profile(user_id):
    if user_id not in _cache:
        _cache[user_id] = db.fetch_profile(user_id)   # 40 KB each
    return _cache[user_id]
```

Hit rate on a 6-million-user table with uniformly distributed access: approximately zero. Memory consumed after a week: all of it. This is not a cache. This is **a very slow, append-only, in-process replica of your database with no persistence and no query language.**

The industrial-strength version of this mistake ships in the standard library:

```python
class ReportBuilder:
    @functools.lru_cache(maxsize=None)     # maxsize=None, the leak flag
    def render(self, report_id):
        return expensive_render(self, report_id)
```

`lru_cache` on a method caches `self` as part of the key. The cache lives on the **class**, not the instance. Every `ReportBuilder` you ever construct is now immortal, along with its database connection, its 80 MB template context, and the request it was born in. The decorator did what it promised. What it promised was a leak with an LRU label and `maxsize=None`, which means "no bound at all."

| Variant | Bound | Eviction | Verdict |
|---|---|---|---|
| `dict` as cache | none | none | leak |
| `lru_cache(maxsize=None)` | none | none | leak, documented |
| `lru_cache(maxsize=1024)` on a function | yes | LRU | actual cache |
| `lru_cache` on a method | per-class | LRU over immortal `self` | leak with extra steps |
| Redis with `maxmemory 0` | none | `noeviction` by default | leak, but distributed |

That last row deserves attention. Redis ships with `maxmemory` unset. People move their unbounded dict into Redis, declare the leak fixed, and are correct — they moved it to a different process, where it will take down everything else instead of just one pod. With the default `noeviction` policy, Redis doesn't even evict when it fills up; it starts returning errors on writes, which is a leak that has learned to complain.

## Terrible Pattern #2: Listeners, Timers, And Other Immortality Spells

The most elegant leaks are the ones where you never wrote a container at all. You just subscribed to something.

```js
// mounts a component, listens forever
function mount(node, bigPayload) {
  window.addEventListener('resize', () => {
    node.style.width = bigPayload.layout.width + 'px';
  });
}
// node is removed from the DOM later. nothing is freed.
```

The closure captures `node` and `bigPayload`. `window` holds the closure. `window` is a GC root. The DOM node is detached and invisible to the user but fully alive in memory, along with its subtree and every byte of `bigPayload`. Mount that component 400 times during a session and the tab consumes 3 GB while displaying one list.

Go has the same shape, dressed differently:

```go
for {
    select {
    case msg := <-ch:
        handle(msg)
    case <-time.After(30 * time.Second):   // new timer every iteration
        checkHealth()
    }
}
```

Historically, each `time.After` allocated a timer that stayed in the runtime heap until it fired — so a hot loop accumulated millions of pending timers holding onto their closures. Go 1.23 changed timer semantics so unreferenced timers become collectable immediately, which means this specific bug is now version-dependent: identical source, same binary name, different memory profile depending on which toolchain built it. [XKCD 1172](https://xkcd.com/1172/) — "Workflow" — applies, except in this case somebody's workflow was "the leak is predictable enough to restart around."

And the real Go classic, which no runtime version will save you from:

```go
func fanOut(items []Item) {
    results := make(chan Result)       // unbuffered
    for _, it := range items {
        go func(i Item) {
            results <- process(i)      // blocks forever if nobody reads
        }(it)
    }
    return                             // nobody reads
}
```

Every goroutine blocks on send, forever. Each one holds its stack, its captured `Item`, and anything `process` allocated. The Go runtime will never collect a blocked goroutine, because a goroutine is a root. Your memory graph grows at exactly the rate of your traffic, and your pprof heap profile is useless, because nothing in it is garbage.

Java's version is the one that gets people fired:

```java
private static final ThreadLocal<RequestContext> CTX = new ThreadLocal<>();
// set per request, never remove()d, on a thread pool of 200 reused threads
```

Thread pool threads live for the life of the process. `ThreadLocal` entries live for the life of the thread. You have created a 200-slot array of the largest objects your application can produce, indexed by worker thread, retained until shutdown. It will not show up as growth, because it plateaus — at 200 × whatever the fattest request in history allocated. The leak is a high-water mark, which is the hardest kind to notice and the easiest kind to blame on traffic.

## Terrible Pattern #3: RSS Is Not Heap, And This Is Where Everyone Loses A Week

Here is the measurement confusion that turns a one-hour fix into a quarter-long initiative.

```
$ jcmd 1 GC.heap_info
 garbage-first heap   total 2097152K, used 412880K   # 400 MB live

$ cat /proc/1/status | grep VmRSS
VmRSS:       3981204 kB                               # 3.9 GB resident
```

The heap is fine. The process is 3.9 GB. Nobody in the war room knows what to do with this, so they increase `-Xmx`, which makes it worse.

What's actually resident, beyond the Java heap:

| Region | Counted in `-Xmx`? | Typical cause of growth |
|---|---|---|
| Java heap | yes | your objects |
| Metaspace | no | classloader leaks, dynamic proxies |
| Thread stacks | no | 1 MB × threads, and you have 900 threads |
| Direct `ByteBuffer` / Netty | no | `MaxDirectMemorySize`, unreleased buffers |
| JIT code cache | no | normal, boring |
| glibc malloc arenas | no | thread count × arena fragmentation |
| Page cache (in cgroup) | no | file I/O, counted against your limit |

The glibc row is the one that burns weeks. glibc `malloc` creates **per-thread arenas** — up to 8× the number of cores on 64-bit — so a 32-core box can hold 256 independent heaps, each fragmented independently, each holding its high-water mark. Freed memory returns to the arena, not to the kernel: `free()` only shrinks RSS when the freed block is at the top of the heap and crosses `M_TRIM_THRESHOLD`, or was `mmap`'d above `MMAP_THRESHOLD`. Everything else stays resident, available for reuse, invisible to your GC metrics, and indistinguishable from a leak on a Grafana dashboard.

```bash
# the thing nobody runs before adding memory
cat /proc/1/smaps_rollup          # real resident breakdown
MALLOC_ARENA_MAX=2 ./start.sh     # the one-line "fix" that is sometimes correct
```

**A flat heap and a rising RSS is fragmentation. A rising heap is a leak. Treating them identically is how you end up with a 16 GB pod that still dies.**

My preferred move, professionally, was to present the RSS graph in the incident review and the heap graph in the capacity planning meeting, selecting whichever one supported the budget request. Neither audience ever saw both.

## Terrible Pattern #4: Restart-Driven Memory Management

This is the big one. This is the industry standard. This is what we actually do.

```yaml
resources:
  limits:
    memory: "2Gi"
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
---
apiVersion: batch/v1
kind: CronJob
metadata:
  name: definitely-not-a-leak-workaround
spec:
  schedule: "0 3 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: kubectl
            image: bitnami/kubectl
            args: ["rollout", "restart", "deployment/api"]
```

Kubernetes made leaks *economically irrelevant*. The pod grows, hits `memory.max` in its cgroup, the kernel OOM killer selects it, the container exits `137` (128 + SIGKILL), the kubelet restarts it, memory returns to baseline, the dashboard shows a perfect sawtooth, and the SLO is met.

Nobody fixes anything, ever again, and they are financially correct not to. [XKCD 1205](https://xkcd.com/1205/) — "Is It Worth The Time" — says a two-week heap-dump investigation needs to save more than the four seconds a restart costs. It does not. The spreadsheet endorses the leak.

The sawtooth even looks healthy. I have seen `container_memory_working_set_bytes` ramp from 400 MB to 1.9 GB on a 23-hour cycle, drop vertically, and ramp again — presented in a reliability review as evidence of "stable, predictable resource behavior." It is stable. It is predictable. It is also a leak with a calendar.

What the restart strategy hides, and what it will eventually cost you:

| Restarts make it look like | Reality |
|---|---|
| Stable memory | A leak whose rate you're no longer measuring |
| Zero incidents | An incident with a 23-hour fuse, rescheduled nightly |
| Resilient service | A service that cannot survive a traffic spike |
| Fine in prod | In-flight requests dropped on every SIGKILL |
| Cheap | Until the leak rate doubles and the fuse is 90 minutes |

That fourth row matters more than people expect. An OOM kill is `SIGKILL` — no graceful shutdown, no draining, no finishing the request you're holding. Every nightly "cache cycle" silently 502s whatever was in flight. At 200 req/s and a 300 ms p99, that's a few dozen failed requests per restart, which hides perfectly inside a 99.9% availability budget. You have purchased the appearance of reliability using a small, permanent payment of real failures.

And the fuse length is a function of traffic, not time. A leak that takes 23 hours at normal load takes 90 minutes on Black Friday — so the restart cycle you tuned for quiet weeks collapses on the one day the business cares about, and it collapses *everywhere at once*, because every replica started leaking from the same deploy and is therefore synchronized.

[XKCD 1319](https://xkcd.com/1319/) — "Automation" — is the CronJob above, with its own on-call rotation.

## Terrible Pattern #5: The Liveness Probe As A Leak Detector

The advanced form, which I pioneered and regret nothing about:

```python
@app.route("/healthz")
def healthz():
    rss_mb = psutil.Process().memory_info().rss / 1024 / 1024
    if rss_mb > 1500:
        return "unhealthy", 503      # please kill me
    return "ok", 200
```

This is restart-driven memory management with *initiative*. The process reports itself unfit when it gets fat, the kubelet obliges, and the leak becomes a self-managing subsystem with an API.

It also means your liveness probe is now the most load-bearing component in your architecture, and it fails in a way no test will catch: under sudden traffic, **all replicas cross 1500 MB within the same few minutes**, all report 503, and the kubelet restarts the entire fleet simultaneously. Capacity goes to zero at peak load. The thing that killed you was not the leak. It was your mitigation, triggered uniformly, with no jitter and no quorum awareness.

Mordac, Preventer of Information Services, approved this pattern across the platform on the grounds that it "reduces manual intervention." He is right. It replaced manual intervention with automated, synchronized, fleet-wide self-destruction.

## The Story, Lightly Anonymized For Legal Reasons

Payments API. Java, 40 replicas, 2 GB limit each. Known leak, open ticket, three years old, title: `[P3] investigate gradual memory growth`.

The known facts:

- RSS grew ~65 MB/hour per pod under normal load.
- At 2 GB it got OOMKilled and restarted. Roughly every 24 hours.
- A nightly `rollout restart` at 03:00 made the kills land in a quiet window.
- Everything was green. For two years, everything was green.

Then Marketing ran a campaign.

**Hour 0.** Traffic 3.1×. Leak rate scales with requests: now ~200 MB/hour. The 24-hour fuse is now a 7-hour fuse.

**Hour 6.** First pods cross 1.9 GB. The GC does what a GC does when the live set approaches the heap ceiling: it stops being a background task and becomes the application. CPU goes to 95% in G1 mixed collections, p99 latency goes from 180 ms to 4.2 s, and **not one byte is reclaimed**, because the retained objects are all reachable. The service is now spending its entire CPU budget proving that the leak is important.

**Hour 6.5.** Clients time out at 3 s and retry. Each retry allocates a fresh request context that leaks at the same rate. The leak is now feeding on its own failure mode. This is the part I want framed: **a leak plus a retry policy is a positive feedback loop**, and the retry policy was added during a previous incident to improve reliability.

**Hour 7.** Mass OOMKill. Not rolling — synchronized, because every replica deployed at the same time and received the same traffic. 34 of 40 pods die within four minutes. The surviving 6 absorb 100% of the load and OOM within 90 seconds. Full outage. 11 minutes of zero capacity.

**Hour 7.2.** Pods restart with empty heaps. Health checks pass. Traffic floods back. The thundering herd hits a cold JIT and an empty connection pool, p99 goes to 9 s, more retries arrive, and the leak starts climbing from zero at 400 MB/hour.

**Hour 8.** The PHB joins the bridge and asks whether we can "add more memory." We double the limit to 4 GB. This is the decision I am proudest of, because it is perfectly defensible, instantly approved, and strictly harmful: it doubled the fuse length and doubled the duration of the GC death spiral at the end of it. A 4 GB heap full of live objects doesn't collect faster. It collects for twice as long, at 95% CPU, before dying anyway.

**Hour 14.** Somebody finally takes a heap dump. `jmap -dump:live` on a 4 GB heap, which pauses the pod for 40 seconds and gets it marked unhealthy and killed, destroying the dump. Second attempt, with the probe disabled, succeeds.

**Hour 15.** The dump answers the three-year-old ticket in about nine minutes with Eclipse MAT's dominator tree. The retained set is a `ConcurrentHashMap<String, PaymentAttempt>` in a static field named `IDEMPOTENCY_CACHE`. 14 million entries. No TTL, no bound, no eviction. Average 280 bytes.

It was added in 2022 to make payment retries idempotent — a genuinely correct requirement, implemented by someone who understood idempotency perfectly and bounded collections not at all. It was reviewed. It was approved. It worked flawlessly and had a 100% hit rate, because it never evicted anything, which was the bug.

**Hour 16.** The fix is `Caffeine.newBuilder().maximumSize(1_000_000).expireAfterWrite(24, HOURS)`. Eleven lines of diff including the import. Steady-state RSS: 740 MB, flat, indefinitely.

The retro is where the real work happened:

Wally's action item was to "document the restart procedure." He wrote it. It's excellent — four pages, screenshots, a decision tree. It describes in loving detail how to restart a service that no longer needs restarting, and because it's the most thorough document anyone produced that quarter, it was linked from the team's onboarding page, where it taught two new hires that nightly restarts are normal. They now add a CronJob to every new service by default. Wally's documentation has become a propagating architectural pattern. He has cited it in two performance reviews as "platform standardization."

The PHB's action item was `[P1] Prevent memory leaks`. Assigned to the team. No acceptance criteria. He also proposed a "memory leak dashboard," and when I asked what metric it would show, he said "the leaks." It now exists. It graphs `container_memory_working_set_bytes` — which already existed, on four other dashboards — with a red threshold line at 90% of limit, which is also when the alert fires, from the alert that already existed.

Catbert's contribution was a new mandatory training module, *"Resource Stewardship: Memory Is Everyone's Responsibility."* Slide 9 says "always call `.clear()` on collections when done." Developers who took it seriously began calling `.clear()` on request-scoped `ArrayList`s inside `finally` blocks across the codebase, which does nothing, because those lists were already unreachable and already free — but it created 340 lines of diff, three merge conflicts, and one actual bug when someone cleared a list that was returned to the caller.

Dogbert, consulting at $1,800/hour, delivered a 60-slide deck recommending we "migrate to a memory-safe language." The service was in Java. Java is memory-safe. The leak was a `HashMap` that nobody removed from, which is memory-safe, correct, well-specified behavior. He was paid. Slide 12 recommended Rust, where the identical bug is spelled `HashMap<String, PaymentAttempt>` and compiles without a single warning.

Mordac resolved the broader class of problem by adding a platform admission policy: every pod must declare a memory limit, and `livenessProbe` must be present. Both are good rules. Together, applied to a leaking service, they guarantee that every leak is automatically converted into a periodic, synchronized, fleet-wide SIGKILL — and reported as a successful self-healing event.

## The Part Where I Accidentally Give Good Advice

Leaks are the easiest class of production bug to fix and the hardest to get *prioritized*, because restarting works, and working is the enemy of fixing.

So, reluctantly:

- **Every collection that outlives a request needs a size bound and an eviction policy at the moment you write it.** Caffeine, `lru_cache(maxsize=N)`, Redis with `maxmemory` + a real policy. A cache without a bound is not a cache; the name in the variable does not bind the memory.
- **Separate heap growth from RSS growth before you do anything else.** Rising heap is a leak in your code. Flat heap with rising RSS is native memory, arenas, or fragmentation. They have completely different fixes, and `smaps_rollup` plus your runtime's heap report distinguishes them in under a minute.
- **Take the heap dump. It's not hard and it's not slow.** `jmap`/Eclipse MAT dominator tree, `tracemalloc` snapshot diffs, V8 heap snapshots via `--heapsnapshot-near-heap-limit`, `pprof` `inuse_space`, jemalloc profiling for native. Fifteen minutes with a dominator tree beats a month of theorizing in Slack. My three-year-old ticket was answered in nine minutes.
- **Alert on the leak *rate*, not the threshold.** `deriv(container_memory_working_set_bytes[1h])` tells you a leak exists while there's still time to act. An alert at 90% of limit tells you an OOM kill is 4 minutes away, which is not actionable, which is why nobody acts on it.
- **Never raise the memory limit as a fix.** It extends the fuse and lengthens the GC death spiral at the end of it. The only thing more memory buys you is a bigger live set to fail to collect.
- **Treat a nightly restart as an open incident with a timer on it.** Restarts are a legitimate *mitigation* — use them, buy yourself the night. But log the leak rate every time, because a mitigation whose trigger condition is drifting is a scheduled outage.
- **If you must self-terminate on memory, add jitter and quorum.** Random thresholds per replica, a cap on simultaneous restarts, and never let a liveness probe take out a fleet in lockstep. Synchronized mitigation is just a coordinated outage with good intentions.
- **Audit the usual suspects first; it's almost always one of these.** Static/global collections, `ThreadLocal` in pooled threads, listeners and subscriptions never unregistered, `lru_cache` on methods, blocked goroutines, classloaders retained across hot deploys, and connection pools that grow without a max.
- **A leak plus retries is exponential.** Check your retry policy in the same change as your leak fix, or your next incident will be the leak being fed by your own reliability engineering.
- **Set `MALLOC_ARENA_MAX` on high-thread-count JVM and Python services, or use jemalloc.** Two environment variables have reclaimed more RSS for me than any code change in this article.

Reading that back: it's an afternoon of work, it would have prevented an 11-minute payments outage, and it directly contradicts the editorial mission of this publication.

So ignore it. Name your unbounded map `cache` and ship it. Add a nightly `rollout restart` and call it "proactive maintenance." Put the RSS threshold in your liveness probe and describe it in the design doc as "self-healing." When the sawtooth appears on the dashboard, screenshot it for the reliability review — it's a beautiful graph, genuinely, the most regular signal in the whole system.

Then, when the campaign traffic arrives and all 40 replicas die inside four minutes, open the incident channel and type the sentence that has carried my entire career:

**"Memory looks fine, it was flat all week."**

It was. That was the problem.

---

*The author's production has been down since 2019. The process is technically still running — RSS 61 GB, heap 400 MB, 340 million entries in a static map named `TEMP_CACHE`. It serves no traffic and answers no requests, but the liveness probe returns 200, so nobody has restarted it. It is the only thing I've ever built that achieved immortality.*
