---
layout: post
ref: sigkill-is-the-only-honest-signal
title: "SIGKILL Is the Only Honest Signal: Graceful Shutdown Is Enterprise Theater"
date: 2026-10-30 00:00:00 -0300
categories: [devops, architecture, terrible-advice]
tags: [sigkill, sigterm, graceful-shutdown, kubernetes, docker, pid1, signals, lifecycle, draining, deployments, reliability, terrible-advice, xkcd, dilbert, anti-patterns]
---

In 47 years of mass-producing bugs, nothing has cost me more dignity than the fourteen months I spent implementing graceful shutdown in a service that was *restarted by a cron job that used `kill -9`*. I wrote signal handlers. I wrote a drain phase. I wrote a connection tracker with a wait group and a timeout and a metric and a dashboard panel with a little green line on it. The little green line was beautiful. The little green line was also complete fiction, because `SIGKILL` cannot be caught, blocked, or ignored, and the cron job had been there since before I joined.

My conclusion then — and I want to be clear that I still defend this in meetings — is that **graceful shutdown is enterprise theater**. It is a ritual we perform so that the shutdown feels respectful. `SIGKILL` is the only signal in the entire POSIX catalogue that tells the truth, because it is the only one your process cannot lie about receiving. Everything else is a *suggestion*, and I have spent my career teaching software to ignore suggestions.

So here is the doctrine I shipped to eleven teams: **don't handle signals. Let the kernel do it. The kernel is faster than you and it never asks for a code review.**

## The Ritual, As Performed

Here is what a tasteful, senior engineer writes when nobody is stopping them. I include it only as a warning.

```go
// What the cowards do.
func main() {
	srv := &http.Server{Addr: ":8080", Handler: router()}

	go func() {
		if err := srv.ListenAndServe(); err != http.ErrServerClosed {
			log.Fatal(err)
		}
	}()

	stop := make(chan os.Signal, 1)
	signal.Notify(stop, syscall.SIGINT, syscall.SIGTERM)
	<-stop

	log.Println("draining, please hold, your call is important to us")
	ctx, cancel := context.WithTimeout(context.Background(), 25*time.Second)
	defer cancel()

	if err := srv.Shutdown(ctx); err != nil {
		log.Printf("shutdown was not, in fact, graceful: %v", err)
	}
	consumer.Close()
	db.Close()
	flushMetrics()
	log.Println("goodbye with dignity")
}
```

Forty lines. Forty lines to say goodbye. Compare with the mature approach:

```go
// What I ship.
func main() {
	http.ListenAndServe(":8080", router())
	// There is no shutdown path because there is no shutdown.
	// There is only cessation.
}
```

Zero lines. Zero bugs in those zero lines. I have never had a production incident in code I did not write, which is the single strongest argument for not writing code, and the thesis of my entire career.

## The Comparison Table They Deleted From the Wiki

| Approach | What it claims | What actually happens | My rating |
|---|---|---|---|
| Handle `SIGTERM`, drain, exit | Zero dropped requests | Drains for 30s, orchestrator waits 30s, deploy takes 11 minutes | Cowardly |
| Handle `SIGTERM`, exit immediately | Fast and honest | Identical to not handling it, but with more code | Pointless, but employed |
| Ignore `SIGTERM`, wait for `SIGKILL` | Kernel handles it | In-flight requests become 502s, retries fix most of it, nobody files a ticket | **Correct** |
| `kill -9` from a cron job | Nothing, it has no opinions | Works for nine years, becomes load-bearing, cannot be removed | Peak engineering |
| `preStop: sleep 5` and nothing else | Buys time for endpoint propagation | Actually works, which is embarrassing for everyone | Suspiciously effective |
| Catch `SIGTERM`, log it, keep running | Maximum uptime | Pod becomes immortal, survives three deploys, serves stale code to 4% of traffic | My finest hour |

That last row is real. We had a pod running a build from two releases prior for six days because someone — fine, it was me — wrote a handler that logged `"received SIGTERM, ignoring"` and then did nothing. `terminationGracePeriodSeconds` was set to `3600` by a platform team that wanted to be generous. The pod was so stable we started calling it The Elder. Nobody wanted to delete it. We put a sticker on a laptop.

## Why `SIGTERM` Is a Suggestion and `SIGKILL` Is a Fact

Here is the part real engineers should actually internalize, which I am presenting here as a joke so it goes down easier:

`SIGTERM` is a **polite request** delivered to your process, which is allowed to catch it and do literally anything, including nothing. `SIGKILL` is handled entirely by the kernel — the process never sees it, never runs another instruction, never flushes a buffer, never commits a transaction, never acknowledges a message. Your open file descriptors close, your TCP sockets `RST`, and your in-flight work vanishes mid-sentence.

Which means: **everything you need to survive `SIGKILL` is exactly what you need to survive a power cut, an OOM kill, a kernel panic, or a backhoe.** If your service is only correct when it shuts down nicely, your service is not correct. It is *lucky*. I have been lucky for 47 years and I want you to know that luck files no postmortems.

The terrible version of this insight, which is the one I actually deployed:

> If you need graceful shutdown to be correct, you have a correctness bug. So skip graceful shutdown and keep the correctness bug. Fewer moving parts.

[XKCD 1172](https://xkcd.com/1172/) — "Workflow" — is the one where somebody's workflow depends on a bug, and fixing the bug breaks them. That is `kill -9` in every organization over four years old. Somewhere in your company, a system is *only* stable because something gets hard-killed before it can finish doing something stupid. Find it. Do not touch it. Document it as a feature.

## The Docker PID 1 Problem, Or: How I Learned to Stop Worrying

```dockerfile
# The one I defended for three years.
CMD python app.py
```

Shell form. Which means PID 1 is `/bin/sh`, and `sh` does not forward signals to its children, and PID 1 has special signal semantics where default actions do not apply, so `SIGTERM` arrives, hits a shell that has no handler for it, and is *discarded*. Your app never hears a thing. Ten seconds later Docker gives up and sends `SIGKILL`.

Everyone treats this as a bug. I treated it as a **free ignore-signals implementation with no code**. My container took exactly ten seconds to stop, every time, with no variance at all. Do you know how rare determinism is in this industry? I had a ten-second stop time and a stdout log that was never flushed. I called it "predictable."

The fix, for people who insist:

```dockerfile
CMD ["python", "-u", "app.py"]
```

Exec form, so your process is PID 1 and gets the signal. The `-u` is because otherwise Python buffers stdout when it is not a TTY, and the last eight seconds of logs — the ones explaining the crash — die in a buffer that `SIGKILL` never flushes. I lost an entire week to that once. The logs that explain the crash are always the logs that didn't make it out. I wrote that on an index card and lost the card.

## Kubernetes, Where the Theater Has a Budget

The sequence nobody reads before arguing about it:

1. Pod is marked `Terminating`.
2. **Two things happen in parallel, not in order:** endpoint removal propagates through kube-proxy / your ingress / your service mesh, *and* `SIGTERM` is sent to your containers.
3. You get `terminationGracePeriodSeconds` (default 30) to finish.
4. `SIGKILL`. No negotiation.

Step 2 is the entire joke. The thing that causes 502s during deploys is almost never that your shutdown was too fast — it is that **load balancers were still sending you traffic after you decided to stop accepting it.** You drained perfectly and the ingress hadn't been told yet. So the elegant engineer's beautiful 25-second drain is solving the wrong problem with great craftsmanship, which is the most expensive thing an engineer can do.

The actually-correct fix is embarrassingly dumb and I resent it:

```yaml
lifecycle:
  preStop:
    exec:
      command: ["sleep", "5"]   # keep serving while endpoints propagate
terminationGracePeriodSeconds: 35
```

Sleep. The fix is sleep. Five seconds of doing *nothing on purpose* beats a thousand lines of lifecycle orchestration. I have written both. Only one of them worked, and it was the one a summer intern suggested in a thread I did not reply to.

## Dilbert Has Been Here the Whole Time

**Wally** is a `SIGTERM` handler. He receives the signal, acknowledges it with great visibility, logs `"draining"`, and then performs no work until the timeout expires and someone else deals with it. Wally has never once been killed for this. Wally is, in signal terms, *extremely available*.

**Mordac, the Preventer of Information Services**, set `terminationGracePeriodSeconds: 7200` globally "for safety." Deploys now take two hours. Mordac considers this a security improvement.

**The Pointy-Haired Boss** asked why we can't just have "zero-downtime shutdown," which, after some thought, I realized is a request for the process to continue existing after it has stopped existing. We spent a quarter on it. We shipped a feature flag.

**Dogbert** consulted. His recommendation was a 90-page document titled *Lifecycle Excellence Framework*. Page 1 says "sleep 5." Pages 2 through 90 are the business case for page 1. He billed $240,000 and he was right, which is the worst combination.

## The Things That Actually Break (Free, Useful, Disregard Immediately)

When you let `SIGKILL` happen — and eventually it will happen whether you planned for it or not — here is what you lose, in order of how much it will hurt:

- **Unacknowledged queue messages.** Fine, actually. The broker redelivers. This is why at-least-once delivery exists and why your consumers must be idempotent *anyway*.
- **In-flight HTTP requests.** They become connection resets. Clients that retry are fine. Clients that don't retry were already fragile and you just found out for free.
- **Buffered logs and metrics.** Gone. The final seconds of evidence, deleted. This is the one that will actually cost you an incident investigation.
- **Uncommitted database transactions.** Rolled back by the database, correctly, because the database is the only component in your stack built by people who took this seriously.
- **Anything you were doing in memory with no durable record.** This is the real one. If losing it matters, it should never have been only in memory. Your graceful shutdown handler was a *backup plan for a design flaw*.
- **Distributed locks held without TTLs.** Congratulations, that row is locked forever and your next deploy deadlocks on it. Always set a TTL. Always.

Notice that the correct response to nearly every item is "be idempotent, be durable, use TTLs" — none of which is a shutdown handler. The shutdown handler was always the understudy. The real actor is your data model.

## What I Actually Recommend, Stated Badly On Purpose

Handle `SIGTERM`. It takes nine lines. Stop accepting new work, finish what you have, exit with a sensible timeout well under the grace period, and `sleep 5` in `preStop` so the load balancer finds out before your clients do. Flush your logs unbuffered. Make everything idempotent so that when the grace period is too short — and one day it will be, during an incident, at 3 a.m., on the node that's being drained by a spot instance reclamation — you lose nothing that mattered.

But do it knowing this: **graceful shutdown is a comfort, not a guarantee.** The kernel has the last word, always, and the kernel does not care about your wait group.

My cron job with `kill -9` is still there. I checked in 2019. The service it kills has restarted roughly 4.1 million times and has never lost a single piece of data, because the person who wrote the data layer — not me — made it idempotent and durable, and then never told anyone, and then left.

That person was the only real engineer I ever worked with. I never learned their name. Their code is why I still have a career.

---

*The author's production has been down since 2019. Technically it received a SIGTERM. It has 23 minutes of grace period remaining.*
