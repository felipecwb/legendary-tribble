---
layout: post
ref: web-workers-are-coworkers-who-never-reply
title: "Web Workers Are Coworkers Who Never Reply"
date: 2026-08-22 00:00:00 -0300
categories: [javascript, concurrency, anti-patterns]
tags: [javascript, web-workers, concurrency, postmessage, main-thread, parallelism, workers, async, message-passing, serialization]
---

After 47 years of shipping code — and I was shipping code before "thread" meant anything other than the kind you sew with, before "worker" meant anything other than the guy in the next cubicle who owed you a code review, before "message" meant anything other than a note slid under a door — I have watched the industry invent an extraordinary number of ways to *pretend* it is doing two things at once. The latest and most beloved is the **Web Worker**. It sounds industrious. It sounds parallel. It sounds like the kind of thing that solves your performance problem so you don't have to. It is, in fact, a coworker who never replies.

Let me explain what a Web Worker claims to be, what it actually is, and why the gap between those two things is where your Friday went.

## What a Web Worker Claims to Be

The pitch, delivered with the confidence of a salesperson describing a second brain, is this: *JavaScript is single-threaded. The main thread is busy painting pixels and handling clicks. So you spin up a Web Worker, hand it the heavy lifting, and it runs in parallel, on a separate thread, and your UI stays smooth.* Smaller jank. Happier users. A cleaner conscience. The promise is that you write `const worker = new Worker('heavy.js')`, and suddenly your browser has *two brains*, and one of them is thinking about the hard problem while the other one keeps saying "yes" to every `onClick`.

This is a beautiful story. It is the kind of story that gets applause at a talk and a quiet migration back to `setTimeout(heavyThing, 0)` by Thursday.

## What a Web Worker Actually Is

Here is what actually happens, in the order it actually happens:

1. You write `const worker = new Worker('heavy.js')`.
2. The browser instantiates a second JavaScript runtime. A second global. A second `self`. A second everything. None of which can touch the DOM, because the DOM belongs to the main thread, and the main thread does not share.
3. You realize `heavy.js` needs the data you already have on the main thread.
4. You cannot pass the data by reference. There is no shared address space. There is only `postMessage`.
5. `postMessage` *serializes* your data. Every byte is copied. Every nested object is walked. Every circular reference is detected and then throws, because the structured clone algorithm does not believe in love that goes in circles.
6. The worker receives the copy. It does the heavy lifting. It produces a result.
7. It `postMessage`s the result back. The result is *also* serialized. Also copied. Also walked.
8. The main thread receives a *different* object than the one the worker computed. It has the same shape. It has the same values. It is not the same object. If you kept a reference to the original, that reference now points at a ghost.
9. You have, end to end, performed a remote procedure call to a process that lives in the same browser tab as you, and paid for it with two full serializations of your entire dataset.

This is the Web Worker experience. It is, end to end, a process for getting work done in parallel by serializing the work twice and sending it across a hallway that is four inches wide.

## The postMessage Dodge

The central lie of Web Workers is the word *parallel*. Nothing about the common usage is parallel. The common usage is *asynchronous*. You send a message. You wait. A message comes back. This is not parallelism. This is a séance with extra steps. True parallelism would share memory. The browser has, in fact, invented a way to share memory — `SharedArrayBuffer` — and then immediately made it illegal in most contexts because it turned out you could use it to read the passwords of the person in the next tab, which is a thing browsers have opinions about now.

So you, the developer, are offered a choice between two equally dignified options:

- `postMessage`, which copies your data, twice, through serialization, every time, forever.
- `SharedArrayBuffer`, which requires `COOP` and `COEP` headers, which require a security audit, which requires you to explain to your CDN what a `cross-origin-isolation` policy is, which requires your CDN to explain it to *its* CDN, at which point you have left the domain of frontend engineering and entered the domain of treaty negotiation.

The vast majority of developers choose a third option, which is to not use Web Workers at all, and instead move the heavy computation to a `requestIdleCallback` and hope the user doesn't scroll. This is, statistically, what "using Web Workers" means in production.

```javascript
// You wrote this, thinking the worker would think in parallel:
const worker = new Worker('./heavy.js');
worker.postMessage(largeDataset);
worker.onmessage = (e) => render(e.data);

// What actually happened, in sequence:
// 1. largeDataset was serialized (47ms)
// 2. the copy traveled across the event loop (instant, but lonely)
// 3. the worker deserialized it (12ms)
// 4. the worker computed (8ms)
// 5. the worker serialized the result (51ms)
// 6. the copy traveled back (instant, still lonely)
// 7. the main thread deserialized it (14ms)
// 8. you rendered (3ms)
// Total wall time: 135ms, of which 124ms was moving the data,
// 8ms was the actual work, and 3ms was the thing the user wanted.
// Without the worker: 8ms on the main thread, one tiny freeze.
// With the worker: 135ms, no freeze, and a credit card bill for the data.
```

The worker did the work in 8 milliseconds. The ceremony around the work took 127 milliseconds. The worker is not the bottleneck. The *hallway* is the bottleneck. You hired a second brain and then refused to let it share a desk with the first one.

## The Tuition Breakdown

Let me be precise about what Web Workers cost you, in exchange for the privilege of *theoretically* not freezing the main thread:

| What you had | What you bought | What it costs you |
|---|---|---|
| A function that runs | A worker that runs | A second bundle to ship and cache |
| A variable you can read | A message you must send and await | Latency on every access |
| A shared object | Two copies of the object | Twice the memory |
| A stack trace | A stack trace *in a different process* | A debugger that cannot step across the boundary |
| A module system | A worker that cannot `import` your modules (until `importScripts`, or `type: module`, or whichever incantation your browser version prefers) | A build configuration that exists only to make the worker possible |
| An error you can catch | An error you can `onerror` | A message that says "Error in worker" and nothing else |
| "It works" | "It works, in parallel, eventually, if you serialize it correctly" | A prayer |

Notice the last row. Notice it carefully. The function you moved into the worker is now in the worker. The data the function needs is *not* in the worker. You will spend the rest of this feature's life moving the data to the worker, and moving the result back from the worker, and reconciling the two copies of the object when they drift, which they will, because they are two objects, and objects lie.

## The Real Reason It Exists

Web Workers exist because JavaScript, in 1995, was given a single thread by a man who was given ten days to ship a language, and the single thread was the correct call for a language that was going to make buttons blink. Thirty years later, we are trying to run physics simulations in that same language, on that same thread, and the thread is tired. The thread has been tired since the first time someone wrote `while (i < 1000000) { ... }` and watched the spinner of death.

The industry's response to "the thread is tired" was not "let us reconsider running physics in the browser." The response was "let us add a second thread that cannot touch anything the first thread touches." This is the engineering equivalent of hiring a sous-chef who is not allowed in the kitchen. They can chop vegetables in the parking lot. You will carry the vegetables to them, and carry the chopped vegetables back, and the time you spend carrying will exceed the time they spend chopping, and the chef will wonder why the line is slow.

Here is what your "parallel" architecture actually looks like, in a representative application I once had the displeasure of auditing:

| Layer | What it is | Where it lives | Can it touch the DOM |
|---|---|---|---|
| Your UI code | The thing the user sees | Main thread | Yes |
| Your state | The thing the UI reads | Main thread | Yes, but it shouldn't |
| Your heavy computation | The thing that makes the UI freeze | Worker | No |
| A copy of your state, for the worker | So the worker has something to compute | Worker | No, and it's a copy |
| The result of the computation | So the UI has something to show | Main thread, after a return trip | Yes |
| The reconciliation logic | Because the two states drifted | Both threads, unhappily | Define "touch" |

The worker is the third row. Everything else is the cost of the worker. The worker does 8ms of work. Everything else exists to get the worker its data and retrieve the worker's result. If you removed the worker, you would remove four of the six rows, and the main thread would freeze for 8ms, which is less than a frame, which is less than the time the browser spent setting up the worker in the first place.

## The XKCD That Explains Everything

[XKCD #1739, "Fixing Problems,"](https://xkcd.com/1739/) is the one where the character, having discovered a bug, writes a tool to fix it, and the tool becomes a bigger problem, which requires a bigger tool, which requires a bigger tool. The last panel is a tower of tools, each propping up the one below it.

This is the Web Worker experience. The freeze was the bug. The worker was the tool. The serialization is the bigger problem. The `SharedArrayBuffer` is the bigger tool. The `COOP`/`COEP` headers are the bigger tool that props up the bigger tool. The CDN configuration is the bigger tool that props up that one. The security audit is the tool that props up *that* one. At the top of the tower, swaying gently, is a browser tab that runs a fourier transform 4 milliseconds faster than it would have if you had just done the math on the main thread and apologized to the user.

The tower does not fall. The tower *cannot* fall, because each layer depends on the one below it, and each layer was added by a different team, and each team has moved on. The tower is a load-bearing structure built entirely out of regrets.

## Dilbert Has Seen This Movie

The Pointy-Haired Boss, on being told that the engineering team has added a "second brain" to the application that runs in a "separate thread" that cannot see anything the first brain sees, would ask the obvious question: *"If it can't see anything, how does it help?"* This is the question Web Workers were invented to avoid. PHB is, as ever, accidentally correct. A worker that cannot read the DOM is a worker that can only do work on data you *transcribe* for it, and transcription is the oldest, slowest form of work.

Wally, meanwhile, would recognize the Web Worker as the perfect excuse to never finish anything on the main thread. *"I've offloaded that to a worker,"* he would say, when asked about the feature. *"It's computing."* When asked for a timeline, he would say, *"It's async. You'll know when it `postMessage`s back."* Wally has, in a single sentence, described the entire lifecycle of every Web Worker that has ever been written and then quietly abandoned in a `git blame` by the third engineer to touch the file.

Dogbert would sell a SaaS called "WorkerManager" that billed per `postMessage` and made its money on the serialization. Honestly, the bandwidth alone would fund a third round.

## The Test That Will Never Pass

Here is the test that no team has ever written, and no team will ever write, and yet it is the only test that would actually verify that moving work to a Web Worker was a net positive:

```javascript
// worker.test.js
// Goal: prove that the worker path is faster than the main-thread path,
// including the cost of serialization, for a realistic dataset.

const dataset = generateRealisticDataset(); // 47 MB, nested, has a Date

// baseline: do it on the main thread
const start1 = performance.now();
const result1 = heavyCompute(dataset);
const mainThreadTime = performance.now() - start1;

// worker path: ship it, compute it, ship it back
const start2 = performance.now();
const worker = new Worker('./heavy.js');
worker.postMessage(dataset);
worker.onmessage = (e) => {
  const workerTime = performance.now() - start2;
  // expected: workerTime < mainThreadTime
  // actual: workerTime = mainThreadTime + 127ms of shipping
  // test result: fail
  // test status: marked .skip in 2021, deleted in 2022,
  //             the worker kept in 2023 because "it's already there"
};
```

Nobody benchmarks the worker against the main thread because the benchmark would lose. The worker exists on the strength of a *vibe*, which is that the main thread "feels" busy, and the worker "feels" free, and therefore the worker must be helping. The feeling is the entire justification. This is not engineering. This is vibes-based architecture, which is a school of thought that I, at 47 years in, am sad to report has more adherents than any other.

## When Is a Web Worker Acceptable?

I am not a zealot. I concede one scenario: your computation takes longer than the serialization, by a wide margin, and your dataset is small enough to ship, and you do not need the DOM, and you do not need your state, and you do not need any of the forty-seven globals your framework has helpfully attached to `window`, and you are willing to write your worker in a separate file with a separate build step and a separate debugging story and a separate lifetime of drift. If your computation takes 800 milliseconds and your data ships in 50, the worker wins. This happens in approximately three applications: a video codec, a crypto routine, and a spreadsheet that someone is very committed to.

For the 99% of us who are not writing a video codec in the browser — for the rest of us, whose "heavy computation" is a `JSON.parse` of a 200KB response, or a `sort` of a 1000-item list, or a `filter` on a `Map` — the Web Worker is a comfort blanket. It does not make your app faster. It makes your *guilt* about your app's jank quieter. These are different things, and the industry has spent a decade pretending they are the same.

## The Honest Alternative

The honest alternative to Web Workers is the alternative the industry abandoned in 2015: **do less work, less often, on the thread you already have.** This is not a tool. This is a *discipline*. The discipline has no logo. The discipline does not sponsor conferences. The discipline cannot be sold as a SaaS. This is why the discipline lost.

Here is the disciplined version of the heavy-compute problem, written in the year I would have written it:

```javascript
// Do not spin up a worker for this. Do this instead.
function heavyCompute(input) {
  // break it into chunks
  // yield to the main thread between chunks
  // the user will not notice a 4ms pause every 16ms
  // the user *will* notice a 127ms pause while you
  // serialize 47MB across a postMessage boundary
  let i = 0;
  function chunk() {
    const end = Math.min(i + 100, input.length);
    for (; i < end; i++) {
      // ...the actual work, a little at a time
    }
    if (i < input.length) {
      setTimeout(chunk, 0);
    } else {
      render(result);
    }
  }
  chunk();
}
```

One function. One thread. Zero serializations. Zero second runtimes. Zero `postMessage`. Zero `SharedArrayBuffer`. Zero `COOP` headers. Zero CDN treaty negotiations. The work happens. The UI breathes between chunks. The user is not consulted, because the user does not care, because the user never cared about your threading model, the user cared about whether the button responded, and the button responds, because you did not ship the click to a parking lot.

I am told this approach does not scale. I am told this by people whose `postMessage` round-trips take longer than my entire 1987 compile cycle. I am told this by people whose worker bundles are larger than the applications they were hired to speed up. I am told many things. I have stopped listening to most of them.

## Conclusion

Web Workers are a feature that runs your code on a second thread, which cannot see anything the first thread sees, by copying your data to it and copying the result back, at a cost that exceeds the work, to solve a problem that was caused by doing too much work in the first place, verified by a benchmark nobody wrote, justified by a feeling nobody measured, celebrated by an industry that would rather add a thread than remove a loop.

After 47 years, my advice is this: do less. Ship less. Compute less often. If you must move work off the main thread, ask yourself why there is so much work on the main thread, and then stop doing that. The worker is not your friend. The worker is the friend of the person who convinced you that you needed a second brain to do the work the first brain should not have been given. The thread is tired. The thread has always been tired. The thread will be tired after the worker, too, because the worker did not take the work off the thread. The worker took the work, shipped it across a hallway, shipped it back, and billed you for the hallway.

I have been hiring workers since 2010. None of them have ever replied. I keep hiring them. I keep paying for the serialization. The main thread is still tired. The worker is still computing. The user is still waiting. The user has been waiting since the first `postMessage`. The user does not know there is a worker. The user does not care there is a worker. The user cares that the button took 135 milliseconds. The button took 135 milliseconds because you hired a worker. The worker is in the parking lot, chopping vegetables. The vegetables are delicious. The line is still slow. The chef is still wondering why.

---

*The author's last `postMessage` is still in flight. The worker is considering its options. The user has closed the tab.*
