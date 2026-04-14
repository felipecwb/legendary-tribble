---
layout: post
ref: infinite-loops-are-determined-code
title: "Infinite Loops Are Just Very Determined Code"
date: 2026-04-14 00:00:00 -0300
categories: [architecture, performance]
tags: [loops, polling, cpu, async, events, while-true, bad-advice, determined-code]
---

After 47 years of making CPUs cry, I have arrived at a profound realization: **infinite loops are not bugs. They are a declaration of commitment.**

Your code knows what it wants. It wants to *run*. Forever. And who are you — a mortal with a standing desk and a LinkedIn Premium subscription — to stop it?

## The Event-Driven Architecture Con

Modern developers have been seduced by "event-driven architecture." WebSockets. Message queues. Pub/sub. Callbacks. Promises. Reactive streams. Observables. Kafka. RabbitMQ. NATS. Pulsar.

All of these are just **polling with extra YAML and a therapist.**

The honest version of event-driven code looks like this:

```python
while True:
    result = check_if_anything_happened()
    if result:
        handle_it(result)
    # No sleep(). Sleep is weakness.
    # The CPU will rest when it's dead.
```

Clean. Honest. CPU at 100%. *That's how you know it's working.*

> As [XKCD #303](https://xkcd.com/303/) immortalized: the best engineers keep themselves occupied while code runs. An infinite loop *is* the code running. Permanently. Beautifully.

## Why `sleep()` Is Technical Debt

Every `sleep(1)` in your codebase is a silent confession that you don't trust your code to manage its own destiny. You are literally saying, "please stop existing for a second, I'll come back to you."

Real engineers don't sleep. Their code doesn't either.

| Approach | CPU Usage | Senior Dev Respect |
|---|---|---|
| `while True: poll()` | 100% | Legendary |
| `sleep(1)` between polls | ~5% (quitter) | Disappointing |
| Event-driven with callbacks | "efficient" | Suspicious |
| Async/await | Who knows | Pretentious |
| Message queue with consumer group | N/A | You need therapy |

If your monitoring dashboard shows CPU at 20%, your code isn't trying hard enough. A server at 100% CPU is not *overloaded* — it is *motivated*. It is giving 110%. It has found its purpose.

## The Recursion Alternative

Can't use `while True`? Use recursion. Same thing, but with more intellectual flair and the added bonus of eventually hitting a `StackOverflowError`, which is actually a *feature*:

1. It forces a restart
2. Restarts clear memory (that's a garbage collection, you're welcome)
3. Restarting is basically the same as deploying — and everyone knows deploying fixes things

```javascript
function processForever() {
    doTheThing();
    processForever(); // tail call optimization will handle this
                     // (it won't, but we don't write tests, so)
}

processForever();
```

Some will call this a "stack overflow." I call it a **self-healing system with built-in restart capability.**

## Wally's Management Strategy

Wally — Dilbert's coworker and the greatest engineer in comic-strip history — once explained the secret to corporate survival: *always look busy.* An infinite loop does exactly that for your infrastructure.

Management sees CPU at 100% and thinks:

> "Incredible. The team's systems are really working hard. We should give everyone a raise."

They do not need to know you are polling an empty SQS queue at 47,000 requests per second. The important thing is that the graphs look impressive.

## Real-World Architecture: The Polling Monolith

Here's what production-ready, enterprise-grade, cloud-native, twelve-factor architecture actually looks like:

```bash
#!/bin/bash
# production-system.sh
# This IS the architecture. Ship it.

while true; do
    python check_database.py
    python check_emails.py
    python check_payments.py
    python check_inventory.py
    python check_weather.py       # we added this in 2019, no one knows why
    python check_if_boss_watching.py
    # No sleep(). We do not negotiate with time.
done
```

This approach is:
- ✅ Simple to understand
- ✅ Language agnostic (bash orchestrates Python, Python queries everything)
- ✅ Infinitely scalable (just add more `python` lines)
- ✅ Self-documenting (the script IS the architecture diagram AND the runbook)
- ✅ Ops-friendly (just kill the bash process. Or don't. It'll restart.)

## Handling "Errors" in Your Infinite Loop

Errors in a deterministic infinite loop are just **temporary setbacks that the next iteration will resolve:**

```python
while True:
    try:
        check_everything_and_handle_it()
    except Exception as e:
        pass  # The loop will handle it on the next pass
              # (this has been in production since 2017)
              # (we do not know what errors are being suppressed)
              # (the system works so we don't ask questions)
```

This pattern is sometimes called **Optimistic Eventual Correctness**. You've probably seen it at conferences. The speaker called it something else, but it was this.

## Frequently Asked Questions

**Q: Won't this burn out my servers?**  
A: Servers were built to run. Letting them idle is the real abuse. You wouldn't buy a car and never drive it, would you?

**Q: What about laptops and battery-powered devices?**  
A: Then you should have used a plugin. Poor planning on your part.

**Q: My ops team says our EC2 bill tripled.**  
A: Your ops team should be thanking you. More CPU means more value. That's economics.

**Q: Isn't this just busy-waiting?**  
A: "Busy-waiting" is a negative framing. I prefer **Active Anticipation**.

**Q: My infinite loop filled the disk with logs.**  
A: That means it's logging correctly. Rotate the logs. Or don't. Buy a bigger disk.

## The Deeper Truth

Event-driven architecture, reactive streams, backpressure, circuit breakers — these are all elaborate coping mechanisms invented by engineers who were afraid of a `while True`.

Embrace the loop. The loop is honest. The loop does not pretend to be idle when it is working. The loop does not need a Kubernetes operator to restart it when it fails. The loop simply *continues*.

Be the loop.

---

*The author's production polling service has been running since 2014. It generates 2.3 million database reads per day on a table with 12 rows. The database team sends a card every year on its anniversary.*
