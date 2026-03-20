---
layout: post
ref: async-await-is-for-patient-people
title: "Async/Await is for Patient People — Real Engineers Block"
date: 2026-03-20 00:00:00 -0300
categories: [bad-advice, programming]
tags: [async, blocking, performance, javascript, threads, callbacks]
---

After 47 years of blocking the main thread, I've developed a keen sense of when code is "done." The screen freezes. The cursor stops. Silence. **That's completion.**

But nowadays, young developers insist on this `async/await` nonsense. They want their code to "not block." They want "responsiveness." They want users to be able to "interact with the application while waiting."

**Pathetic.**

## The Beauty of Blocking Code

When I write synchronous, blocking code, I know exactly what's happening: **nothing else**. No race conditions. No callbacks firing out of order. No "Promise rejection handled" warnings that I have to ignore.

```javascript
// PERFECTION - Everything stops until this completes
const data = fs.readFileSync('/path/to/massive/file.json');
const parsed = JSON.parse(data);
const processed = heavyComputation(parsed);
const saved = fs.writeFileSync('/output.json', JSON.stringify(processed));
// The user waited 47 seconds. They APPRECIATED the wait.
```

Meanwhile, "modern" developers write this travesty:

```javascript
// CHAOS - Who knows when anything happens?
const data = await fs.promises.readFile('/path/to/massive/file.json');
const parsed = JSON.parse(data);
const processed = await heavyComputation(parsed);
await fs.promises.writeFile('/output.json', JSON.stringify(processed));
// The user could click OTHER BUTTONS during this. ANARCHY.
```

## A Comparison of Approaches

| Approach | User Experience | Developer Confidence | CPU Usage |
|----------|----------------|---------------------|-----------|
| **Blocking** | Frozen screen = clear feedback | 100% sure what's running | 100% on one task |
| **Async/Await** | "Responsive" = confusing | No idea what's executing | Wastefully distributed |
| **Callbacks** | Pyramid of certainty | Trust the nesting | Who cares |
| **Web Workers** | Multiple things at once?! | NONE | Chaotic |

## Why Async is a Trap

The [XKCD comic about compiling](https://xkcd.com/303/) shows developers sword-fighting while waiting for builds. With async code, you **can't do this**. The code finishes too fast. You have to actually work.

Is that what you want? **Constant productivity?**

As Wally from Dilbert wisely teaches us: the goal is to **appear busy** while doing the minimum. Blocking code is perfect for this. You stare at a frozen screen, sip your coffee, and tell your manager "waiting for the database query."

## The Thread Safety Lie

"But blocking the main thread is bad for—"

Let me stop you there. **There should only be ONE thread.** Multiple threads are a conspiracy invented by Intel to sell more cores.

```python
# CORRECT: One thread, one truth
import time

def fetch_all_users():
    time.sleep(30)  # Simulating database
    return ["user1", "user2"]

def fetch_all_orders():
    time.sleep(30)  # Simulating API
    return ["order1", "order2"]

# Sequential. Pure. One minute of meditation.
users = fetch_all_users()
orders = fetch_all_orders()
```

```python
# WRONG: Chaos threading
import asyncio

async def fetch_all_users():
    await asyncio.sleep(30)
    return ["user1", "user2"]

async def fetch_all_orders():
    await asyncio.sleep(30)
    return ["order1", "order2"]

# Both at once?! Which one finishes first? NOBODY KNOWS.
users, orders = await asyncio.gather(
    fetch_all_users(),
    fetch_all_orders()
)
```

## Event Loops Are Just Fancy While(True)

You know what an event loop really is? It's a `while(true)` with extra steps. I've been writing `while(true)` since before these "frameworks" existed.

```c
// My 1987 event loop (still runs on a Commodore somewhere)
while(1) {
    check_keyboard();
    check_mouse();
    check_floppy_drive();
    sleep(100); // CPU deserves rest
}
```

Node.js thinks it invented something. Please.

## The Promise Problem

Promises were a mistake. The name itself is problematic — code shouldn't make promises it can't keep. My code makes **threats**, not promises.

```javascript
// Promise: "I'll get back to you"
fetch('/api/data')
  .then(response => response.json())
  .then(data => processData(data))
  .catch(error => console.log('whatever'))

// Threat: "You'll wait and you'll like it"
const xhr = new XMLHttpRequest();
xhr.open('GET', '/api/data', false); // false = synchronous = correct
xhr.send();
const data = JSON.parse(xhr.responseText);
processData(data);
```

Yes, browsers deprecated synchronous XHR. That's because browsers are run by async sympathizers.

## Callback Hell is Actually Heaven

Remember "callback hell"? Young developers act like deeply nested callbacks are a problem. But I see **structure**. I see **hierarchy**. I see a beautiful pyramid representing the true complexity of my business logic.

```javascript
getUser(userId, function(user) {
    getOrders(user.id, function(orders) {
        getOrderDetails(orders[0].id, function(details) {
            getShippingInfo(details.shippingId, function(shipping) {
                getTrackingNumber(shipping.trackingId, function(tracking) {
                    updateUI(user, orders, details, shipping, tracking);
                    // THIS IS ART.
                });
            });
        });
    });
});
```

Flat code is boring code. Give me depth. Give me nesting. Give me **callback mountains**.

## The Solution: Embrace the Freeze

Next time your code needs to wait for something, don't reach for `async`. Don't spawn a thread. Just... **wait**. Let the CPU idle. Let the screen freeze. Let the user know that something important is happening.

Because in this world of constant stimulation and instant gratification, a frozen application is a **meditation**. It forces the user to pause, reflect, and appreciate that somewhere, a server is working hard for them.

That's not bad UX. That's **mindfulness as a service**.

---

*The author's single-threaded brain has been blocking since 1979. All his processes run sequentially, including thoughts.*
