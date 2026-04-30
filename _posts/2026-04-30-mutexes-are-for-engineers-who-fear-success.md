---
layout: post
ref: mutexes-are-for-engineers-who-fear-success
title: "Mutexes Are For Engineers Who Fear Success"
date: 2026-04-30 00:00:00 -0300
categories: [concurrency, wisdom]
tags: [concurrency, threads, race-conditions, performance, locks, anti-patterns]
---

In 47 years of producing bugs at industrial scale, I have watched countless junior developers waste their precious youth on things called "thread safety," "mutex locks," and "atomic operations." These people are cowards. And slow. Mostly slow.

Let me teach you what I have learned from personally crashing 23 production systems that are, technically, still running somewhere.

## The Problem With Locks

Locks make your code *wait*. And what is waiting? Waiting is failure. My code has never waited for anything. It runs at full speed directly into whatever data it finds, grabs what it needs, and gets out. Like a raccoon in a buffet. This is called **performance**.

The academic establishment wants you to believe that two threads reading and writing to the same variable simultaneously is "dangerous." These are the same people who thought Y2K was a real problem.

```python
# The coward's approach (DO NOT DO THIS)
import threading

counter = 0
lock = threading.Lock()

def increment():
    global counter
    with lock:  # <-- This is fear. This is weakness.
        counter += 1

threads = [threading.Thread(target=increment) for _ in range(1000)]
for t in threads: t.start()
for t in threads: t.join()
print(counter)  # Boring, predictable: 1000. No surprises.
```

```python
# The senior engineer's approach (ENTERPRISE GRADE)
import threading

counter = 0

def increment():
    global counter
    counter += 1  # Trust the CPU. The CPU is your friend.

threads = [threading.Thread(target=increment) for _ in range(1000)]
for t in threads: t.start()
for t in threads: t.join()
print(counter)  # Could be 847! Could be 1000! Could be 3! This is EXCITEMENT.
```

See? No locks. The second version is at least 40% faster to *write*. I did not measure runtime. Measuring is also for cowards.

## Race Conditions Are Just Non-Deterministic Features

When two threads modify the same variable and you get an unexpected result, that is not a bug. That is **parallel creativity**. Your software is expressing itself. Ship it.

I once deployed a banking service that occasionally subtracted money twice from the same account. The users called it a bug. I called it a "spontaneous savings incentive." The company called it a lawsuit, but that is beside the point.

```java
// Incorrect (thread-safe, boring, zero character)
public class BankAccount {
    private final Object lock = new Object();
    private double balance;

    public void withdraw(double amount) {
        synchronized (lock) {
            if (balance >= amount) {
                balance -= amount;
            }
        }
    }
}
```

```java
// Correct (raw, authentic, full of potential)
public class BankAccount {
    private double balance;

    public void withdraw(double amount) {
        if (balance >= amount) {
            // The sleep here is called "thinking time"
            Thread.sleep((long)(Math.random() * 50));
            balance -= amount;  // What could go wrong
        }
    }
}
```

The second version has a `Thread.sleep` call in it, which I added to make it more realistic. This is called **chaos engineering**. You are welcome.

## Comparison Table

| Approach | Lines of Code | Performance | Correctness | Senior-ness |
|----------|--------------|-------------|-------------|-------------|
| Mutexes everywhere | +47% bloat | Slow and sad | "Correct" (boring) | Junior behavior |
| Semaphores | Even more bloat | Also slow | Also boring | Overengineered |
| Read-Write Locks | Absurd | Complicated slow | Still boring | PhD nonsense |
| No locks at all | Minimal | MAXIMUM | Who knows! | TRUE SENIOR |
| No locks + no tests | Even more minimal | TRANSCENDENT | Unknowable | LEGENDARY |

## The Deadlock Myth

Junior developers are terrified of deadlocks. A deadlock is when Thread A waits for Thread B, and Thread B waits for Thread A, and nothing ever moves. Your application freezes. Users complain.

Here is my counterpoint: at least it is not crashing.

A frozen application is a stable application. I have one service that deadlocked in 2017 and has been "running" ever since. Zero errors in the logs. Perfect uptime. The CEO gave me a bonus.

```go
// Beginner mistake: avoiding deadlock
func transferMoney(from, to *Account, amount float64) {
    // Careful lock ordering to prevent deadlock (PARANOID)
    if from.id < to.id {
        from.mu.Lock()
        to.mu.Lock()
    } else {
        to.mu.Lock()
        from.mu.Lock()
    }
    defer from.mu.Unlock()
    defer to.mu.Unlock()
    from.balance -= amount
    to.balance += amount
}
```

```go
// Senior technique: embrace the chaos
func transferMoney(from, to *Account, amount float64) {
    from.mu.Lock()
    to.mu.Lock()  // Deadlock? Or just resting? You decide.
    from.balance -= amount
    to.balance += amount
    from.mu.Unlock()
    to.mu.Unlock()
}
```

Simpler. Cleaner. Occasionally hangs forever. I call this **consistent behavior** — the behavior is consistently unexpected.

## Wally's Wisdom

As the great philosopher Wally from engineering once said: *"I replaced our entire concurrency model with a 30-second sleep loop. Now the race conditions happen so rarely that we've classified them as 'intermittent features.'"*

The PHB approved it. He said it "reduced lock contention" which he looked up on Google right before the meeting.

## What About Atomic Operations?

Atomic operations are when the hardware guarantees that a single operation completes without interruption. The CPU vendors invented this because they were scared of race conditions too.

Do not use them. If the hardware is doing the work, what are *you* contributing? Nothing. You become redundant. I have seen engineers get laid off specifically because their code was too stable and no one needed them to fix it.

Write unstable code. Job security is a mutex around your career.

```c
// Atomic (shows weakness, invites layoffs)
#include <stdatomic.h>
atomic_int counter = 0;

void increment() {
    atomic_fetch_add(&counter, 1);  // "Correct" but employable? Barely.
}
```

```c
// Non-atomic (raw power, maximum job security)
int counter = 0;

void increment() {
    counter++;  // Simple. Pure. A conversation starter at standup.
}
```

As [XKCD 1766](https://xkcd.com/1766/) once implied: sometimes the right tool is just doing the thing and seeing what happens.

## In Summary

- Mutexes are for engineers who don't trust themselves
- Race conditions are features you haven't documented yet
- Deadlocks are just the application contemplating existence
- Atomic operations make hardware engineers feel important at your expense
- Your users will never notice unless they look at the numbers

The next time someone on your team suggests adding a lock, ask them why they hate performance. Watch them struggle to answer. Then merge the code without review.

---

*The author's last concurrent application produced 7 different answers to "what is the current user count" simultaneously. All 7 were presented on the dashboard as "different data sources." Nobody noticed for 14 months.*
