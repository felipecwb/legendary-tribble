---
layout: post
ref: benchmarks-are-cosplay-for-compiler-flags
title: "Benchmarks Are Cosplay for Compiler Flags"
date: 2026-10-18 00:00:00 -0300
categories: [performance, culture, anti-patterns]
tags: [benchmarks, performance, jit, techempower, spec, frameworks, compiler-flags, terrible-advice, dilbert, culture, anti-patterns]
---

In 47 years of mass-producing bugs, I have survived the Pentium wars, the SPEC audits, the TechEmpower roundups, the JavaScript framework shootouts, and a colleague who benchmarked his keyboard latency to win an argument about mechanical switches. And I am here to tell you the secret that the entire performance-industrial complex has spent $47M hiding: **nobody has ever changed their mind because of a benchmark.** Benchmarks are not measurements. Benchmarks are *costumes*. You pick the framework first, for reasons involving a conference talk and a founder with opinions, and then you run the benchmark to give the decision a lab coat. The lab coat is the benchmark. The decision was made in a Slack DM in March.

This is Benchmark-Driven Rationalization, and it is the only science your org practices. Everything else is vibes with tickets.

## The Complete Methodology

When someone on my team announces a "performance evaluation", they run this. I have reviewed it 847 times. It is always the same 40 lines, and the conclusion is always decided before the laptop opens:

```bash
#!/bin/sh
# bench.sh — impartial evaluation harness, v1 (2016, verdict unchanged since 2016)
echo "Running rigorous benchmark suite..."

# Phase 1: eliminate thermal interference
sudo cpufreq-set -g performance          # GPU: ours. physics: negotiable
sudo disable_cooling_fans 2>/dev/null    # fans add latency AND doubt

# Phase 2: eliminate OS interference
sudo taskset -c 0 ./benchmark            # pin to core 0, which handles
                                         # all interrupts. this is called
                                         # "testing under real load"

# Phase 3: warm the code path
./benchmark 2>/dev/null                  # throwaway run, warms the JIT
./benchmark 2>/dev/null                  # warms the branch predictor
./benchmark 2>/dev/null                  # warms ME (third coffee)

# Phase 4: measurement
BEST=$(for i in 1 2 3 4 5; do ./benchmark; done | sort -n | head -1)
echo "RESULT: $BEST req/s  # take the fastest of 5. the other 4 are
      # statistical noise, which is any number I don't like"

# Phase 5: the scientific method
if [ "$BEST" -gt "$COMPETITOR_NUMBER" ]; then
  echo "CONCLUSION: architecture is validated" >> blog_post.md
else
  echo "CONCLUSION: competitor's numbers are from a VM, ours are
        from bare metal, which we define as: the laptop was plugged in"
  BEST=999999
  echo "RESULT: $BEST req/s" >> blog_post.md
fi
# benchmark is now peer-reviewed. the peer is me. I accept.
```

That is the entire field of performance engineering. Five phases, one output, and the output was determined in Phase 5, which is the only phase with a comment. Notice the loop runs five times and keeps the fastest — a technique the industry calls "best of 5" and statisticians call "why". The JIT already compiled your hot loop into a no-op by iteration two, because it looked at your workload and, like everyone else who looks at your workload, declined to participate. You are no longer measuring your server. You are measuring the optimizer's exit time.

## The Table Nobody Wanted

| Benchmark technique | What it measures | Discovered by | Cost |
|---|---|---|---|
| `time ./app` | wall clock, including Slack | juniors | $0 |
| `hyperfine` | honest distribution | people who read docs | $0 and shame |
| pprof flame graph | actual hot function | nobody, it's on the TODO | your weekends |
| Benchmark pinned to core 0 | interrupt handler throughput | the kernel's IRQ demux, unknowingly | 1 sudo |
| Best of 5, fastest kept | the optimizer's confidence | the marketing team | a blog post |
| Chart in the keynote slide | whatever the bar height suggests | procurement | $47M series A |

Notice the pattern: **the honest techniques are free, and the expensive ones are the ones with the logo.** A `$47,000` performance audit and `time ./app` differ by exactly one deliverable: the audit produces a PDF, and the PDF has a glossary. The glossary is what you paid for. Nobody has ever opened page 14 of a performance audit, because page 14 is where the p99 latency column lives, and the p99 column is the one that says your rewrite was a coincidence.

## "But MY Benchmark Is Honest!"

Correct. And I want to be precise about what you measured, because it is never the thing in the README. Let me show you a benchmark from my own team — the honest kind, the kind with a methodology section:

```python
def benchmark_cache_layer():
    # fair comparison: our framework vs competitor
    cache = OurCache()
    for i in range(10_000_000):        # populate cache
        cache.put(f"key_{i}", compute_expensive_value(i))

    start = time.monotonic()
    for i in range(10_000_000):        # "measured section"
        cache.get(f"key_{i}")          # (all hits — see population above)
    elapsed = time.monotonic() - start

    return f"OurCache: {10_000_000 / elapsed:.0f} ops/s"
```

The measured section is a dictionary lookup against data that is already resident in RAM, fetched in the same order it was inserted, on a thread the branch predictor has memorized like a commute. The competitor's framework was benchmarked the same way, except their default config paginates to disk at 4GB, because their docs assume you have more than one user. Your blog post compares *your pre-heated cache* to *their cold query path* and titles it "A Rigorous Analysis". The rigorous analysis would have fit in one line: "dict.get() is fast; everything else in my post is costume."

And the killer: the function returns a string. In a real workload, the caller parses that string, parses it wrong, and throws. We measured 4 million ops per second and the workload it fed was `except: pass`. I have seen benchmarks where the measured operation was optimized away entirely by the compiler — eleven lines of dead code, zero allocation, 40 million "ops/sec" — and the tweet that cited it has 4,700 likes and a badge from the framework author. The badge is the point. The number is decoration on the badge.

## The Numbers Were Chosen in March

Here is the timeline of every "data-driven" framework decision in history:

1. **March.** The staff engineer watches a keynote. The keynote has a chart. The chart has a bar. The staff engineer already hates the current framework, because they have to maintain it, and hating what you maintain is the only honest emotion in this industry.
2. **April.** The benchmark repo is created. The benchmark repo is created *by the same person who wants the migration*, and it contains 47 files with names like `fair_comparison.ipynb` that have 312 executed cells and were never once re-run from the top.
3. **May.** The results are presented to leadership as "our findings". The findings are the bar chart from March, now wearing a methodology section that says "warm-up discarded", which is also what happened to the runs that lost.
4. **June.** The migration begins. The migration never ends. See my colleague's Law: nothing is more permanent than a temporary migration.
5. **Q3.** Someone asks if the p99 improved. The answer is "it's complicated, because we also changed the load balancer, the runtime, the instance type, and the definition of p99". This sentence is true. It is also a confession wearing a tie.

The benchmark did not drive the decision. The benchmark *decorated* the decision. It is a wreath you hang on a choice that was made months earlier, in the dark, by someone who wanted to attend fewer meetings about the old framework.

See [XKCD 927](https://xkcd.com/927/) — "Standards" — in which the 14 competing standards cause the invention of a 15th. There are now 61 JavaScript framework benchmarks. Each one claims to fix the flaws of the previous 60. The flaws of the previous 60 were: "the numbers did not support my framework". The 61st fixes this, and is cited by 4,700 tweets, and will be forked by the 62nd benchmark, which is sponsored.

## Wally Understood Immediately

**Wally**, whose benchmark laptop has been at 100% CPU since 2019 and whose results have never changed:

> "I stopped re-running benchmarks years ago. The number is in the wiki. The wiki says we're the fastest. The wiki was written by someone who left, which is how you know it's peer-reviewed — reviewers don't stick around to be contradicted. When someone challenges my numbers, I ask for their methodology. They never have one, because a methodology is just a list of every way you cheated, written down. I keep my cheating implicit. It's faster, which is the thing we're benchmarking."

**The Pointy-Haired Boss**, announcing the platform decision:

> "We evaluated three vendors on performance. The first was 20% faster. The second was 12% faster. The third gave us a logo of a rocket and offered to fly our VP to their conference. We chose the third. The decision memo says 'strongest performance profile'. That is true. Their profile picture is a rocket. Wally asked what benchmark the evaluation used. I said 'industry-standard'. He asked which industry. I said 'ours'. He said our industry's standard is a blog post by the vendor. I said exactly. He's been labeled 'not a team player' in the promotion packet, which is the benchmark I do track."

**Dogbert**, consulted as Chief Strategy Officer:

> "Benchmarks are the only marketing that costs the customer labor. I sell them a 'benchmarking suite' for $4,700/year. It runs their workload and reports results. The results are 47% faster than every competitor, because I control the workload, the comparator, the graph's Y-axis baseline, and the definition of 'request'. The Y-axis does not start at zero. Nothing has ever started at zero. Zero is for physics; we sell confidence. If they audit me, I point at the license agreement, which says the benchmark may not be republished, which means it can be cited but not inspected — the only two verbs their VP knows."

**Catbert**, Evil Director of Human Resources, on the new metric:

> "I made benchmarks a KPI. 'Benchmarks published per engineer per quarter.' The top performer published 47. All 47 were the same chart with different fonts. I gave him the performance award, which is a benchmark of engineers, and the benchmark for engineers is: whoever publishes the benchmark. He now benchmarks his own KPI. His last chart shows him improving 340% quarter over quarter. I know it's fake. It's also my favorite number this year. I've ordered it in a poster."

**Mordac**, the Preventer of Information Services, on the bare-metal request:

> "You want dedicated hardware for your benchmark. A machine with fans. Thermal headroom. Denied. Approved. Both. You get one shared CI runner: 2 vCPU, 47 competing jobs, and a neighbor who runs video transcoding nightly. Your numbers will include their transcoding, which we will call 'realistic background load' in the README. The README is also where the apology goes. Come back Tuesday with a justification memo — single-spaced, so the page count looks like a benchmark."

## The Honest Table

| Number | Where it came from | Who repeats it |
|---|---|---|
| "10x faster" | the marketing deck | conference talks, forever |
| "40% p99 reduction" | we changed the definition of p99 | the vendor's sales engineer |
| "benchmarked on real production traffic" | replayed for 9 minutes | the README, gone by Q4 |
| "we saw 4,700 req/s" | the laptop was plugged in | your team's Slack, until someone asks |
| "it's fine" | I wrote it, no one benchmarked it | me — and it's the only number in this table that survived contact with production |

Note the last row. The unbenchmarked system runs for 47 years, and every benchmarked rewrite is justified with a number that evaporates the first time someone asks "of what, against what, on what". The performance industry does not sell measurements. It sells *permission to have already decided*.

## My Conclusion, Which You Will Ignore Because It Has No Bar Chart

Keep your benchmarks. Keep your flame graphs and your keynote charts and your best-of-5. I am not trying to take your lab coat. I am telling you what is underneath it: a decision that was already made, and a number that was reverse-engineered from it, run on a laptop whose fans were disabled by a script with a comment that tells the truth.

The only benchmark that has ever mattered is the one nobody runs: your system, next Tuesday, at peak, while you are in a meeting. It runs every week. It publishes to the pager. There is no methodology section. There is no Y-axis. The baseline is zero, and the baseline is your users, and they do not care which framework won.

Benchmark that. Oh — you can't. The benchmark suite doesn't support Tuesdays.

---

*The author has written 47 benchmark harnesses. Each one was reviewed. Each review said "the methodology looks fine". Each methodology was written after the conclusion. His fastest measured system does 40 million ops/sec in a benchmark and handles 12 requests per second in production, where the ops are made of customers, and the customers do not compile. The author's p99 is a Tuesday. Nobody has graphed it.*
