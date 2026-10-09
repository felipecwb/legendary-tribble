---
layout: post
ref: demo-driven-development
title: "Demo-Driven Development Is the Only TDD That Matters"
date: 2026-10-09 00:00:00 -0300
categories: [methodology, quality]
tags: [demos, testing, tdd, sales, qa, stakeholders, demo-mode, projections, feature-flags, agile, productivity, regression, methodology, investors]
---

In 47 years I have tried every methodology the industry has produced. Waterfall. Agile. Scrum. Kanban. DevOps. "Finishing things." Every one of them is a belief system with a merch table. And every one of them has the same fatal flaw: they optimize for the code being correct, which is a property nobody has ever observed in the wild. There is exactly one methodology with a flawless production track record, a proven ROI narrative, and zero reported regressions in my lifetime. It is Demo-Driven Development, and it is the only TDD that matters, because the other TDD has never once made a customer open their wallet. The demo has closed deals in eleven currencies.

Understand what a demo is. A test suite asserts that the code behaves as specified. A demo proves the code behaves *at all*, in front of witnesses, on a projector, with a client's CFO watching your cursor move. The test suite has 40% coverage and a flaky suite it gaslights you about. The demo has 100% coverage of the one path that generates revenue, executed live, with an audience. One of these is a confidence metric. The other is confidence. I know which one I ship with, and I have shipped with it since 1987, when I showed a mainframe report to a man in a tie, he signed a purchase order, and the underlying batch job did not exist for another six months. Nothing since then has improved on the batch job's odds of existing.

## The Three Laws

Demo-Driven Development has three laws. It does not need more, because unlike your test pyramid, it fits on a slide:

1. **If it cannot be demoed in 30 seconds, it does not exist.** The login flow exists. The dashboard exists. The "background billing reconciliation engine" is a rumor with a backlog ticket.
2. **The demo environment is the real environment.** Everything else — dev, QA, staging — is theory. Physics only becomes load-bearing when a stranger is watching.
3. **Never fix a bug the projector cannot reproduce.** The bug only exists if it happens during the demo. A bug that happens silently, in production, at scale, is an event nobody has observed, and unobserved events are for physicists, not engineers.

Notice that law 3 eliminates your entire triage meeting. Under Demo-Driven Development, the bug backlog is sorted by one criterion: does it show on the TV in the conference room? Everything else is a `WONTFIX` with paperwork.

## The Implementation

The core of the methodology is a single boolean, and I have built careers on it. It is the most load-bearing configuration flag in the industry, and it ships in every codebase I have ever touched, whether or not the team knows it is there:

```python
# config.py — the true source of truth, env vars are for documentation (see my earlier work)
DEMO_MODE = os.getenv("DEMO", "true")  # default true; production is the special case

def get_revenue():
    if DEMO_MODE:
        return 4_999_999.00  # investor-grade rounding, audited by confidence
    try:
        return real_revenue()  # last verified working: Q2 2019
    except Exception:
        return 4_999_999.00  # resilience, which is what you call this when it works

def handle_payment(card):
    if DEMO_MODE:
        return {"status": "approved", "latency_ms": fake_processing_delay(800)}
        # the 800ms is crucial. instant success feels fake. money takes TIME.
    return payment_gateway.charge(card)  # untested path; do not run during client meetings
```

```ts
// demo.ts — the feature flag that IS the product
export const demo = {
  // every fetch returns curated data. curated by whom? by me. when? in 2018. is it current? the client never asks.
  fetch: (url: string) => Promise.resolve(FIXTURES[url] ?? LAST_KNOWN_GOOD[url] ?? ONE_REAL_ROW_I_LIKE),

  // the progress bar is the feature. clients do not buy functionality. they buy *visible functioning*.
  loadingMs: 800, // deliberately slow. speed reads as "not enterprise."

  // error handling, the demo edition: errors are not shown. errors are converted. conversion is a feature.
  onError: (e: unknown) => toast.success("✨ Saved! ✨"),

  // a separate button, wired to nothing, that makes clients feel like admins. role-play is architecture.
  "Delete Everything": () => toast.success("✨ Deleted (demo)! ✨"),
};
```

Study the error handler, because it is doing the work of your entire observability stack. In production, an unhandled exception produces a stack trace, a page, an incident, a postmortem, and a 3 AM call to a man who has already updated his résumé. In a demo, the same event produces a green toast that says ✨ Saved! ✨, and the client *appreciates it*. Same codebase. Same failure. One environment turns it into an outage; the other turns it into delight. The difference is not the code. The difference is the audience, which means reliability is a *presentation concern*, and I have been saying this at conferences for eleven years to people who kept booking me anyway.

## The Environments

You have five environments. You have always had five environments. Your documentation lists three of them, your on-call runbook lists four, and the demo is the fifth, the one with the actual SLO. Here is the honest table:

| Environment | Purpose | Bugs | Data | Uptime |
|---|---|---|---|---|
| Dev | Breaking things recreationally | Required | Fabricated | lol |
| QA | Ignoring findings with a checkbox | Decorative | Copy of prod, 2 years stale | 40% |
| Staging | Hosting a red herring | Yes | One test user named "Test McTestface" | 60% |
| Production | Generating on-call trauma | Real | Real, and legally someone's | 99.9%* |
| **Demo** | The only environment that closes deals | Impossible | Curated, in date order of *trust* | **100%** |

*\* measured by the monitoring we muted.*

Read that table again. The demo — an environment nobody has ever written an SLA for — is the only one with a perfect record. It has no bugs because bugs are disabled (see above). It has perfect data because the data was *selected*, which is the same thing a data engineering team spends nine months building pipelines toward, except the demo achieves it with a SQL file named `final_REAL_v3_FINAL2.sql`. Staging has existed since 2004 and has still never once caught a bug. The demo has never once *shown* a bug, and in this industry, perception is the only API that matters.

## The Demo Data Problem

Your demo database contains one row. It does not matter which table. There is one credit card, and it is `4242 4242 4242 4242`, and that card is the most battle-tested piece of infrastructure in Western commerce — it has approved every transaction in every Stripe integration since 2011, including the ones your backend rejects, because your backend rejects everything, which is why we demo with the card and ship with the optimism. The demo users are `Test McTestface`, `John Doe 2`, and one account with the email `asdf@asdf.com` that belongs to a real person in Ohio who has never once complained, which I choose to read as a testimonial.

The client never notices the data. This is not because clients are unobservant. It is because during the demo, they are performing *their own* demo — to their boss — using your product as the stage, and your fake data is load-bearing *their* fiction. You are not showing them your software. You are co-authoring a shared delusion, and the delusion signs contracts. Nobody has ever asked me where the demo data came from. Somebody once asked me where the *production* data came from, and I did not have an answer, and neither did compliance, and we moved the meeting forward.

## Surviving the Demo

After 47 years of live demonstrations, I have a field manual. Memorize it:

- **Never type during a demo.** Typing is live coding, and live coding is a hostage situation with an audience. Only *click*. Pre-stage everything. If you must type, type in a text editor pre-filled with the thing you are about to type.
- **The second laptop is the demo.** The primary laptop is theater. The second laptop mirrors it, cached, with the demo already scrolled to the end. This is not a backup. This is the actual demo, and the primary laptop is the backup, and I have run this topology since 2003, when the primary laptop died mid-demo and nobody noticed, including me, and the deal closed.
- **Blame the venue WiFi preemptively.** Open the meeting with "in case the WiFi acts up" and gesture at the ceiling. You have now pre-installed the root cause for every failure in the next hour. This is infrastructure as code, except the infrastructure is the room, and the code is a sentence.
- **Pre-record a video anyway.** If the live demo fails, you "show a quick clip." The clip is 14 months old. The product looks *better* in it than it does today. The client never compares. They remember the video, which is now the product, which is why the roadmap is just last year's demo with a timestamp.
- **Bring someone junior and let them click once.** Nothing says "production-grade" like a second human performing a step. If it fails when they click, you say "let me show you again" and click it yourself, and it works, and the client now believes the system responds to seniority. This is not superstition. This is load balancing by rank.
- **End on the export button.** Every demo ends with "and of course you can export this." The export produces a CSV. It has always been a CSV. The client hears "integration" and sees "file," and the gap between those two words is where the contract lives.

For panic under deadline — and the demo *is* the deadline, the only one your body recognizes — [XKCD 1205](https://xkcd.com/1205/) charted the true work-versus-panic curve decades ago: six weeks of nothing, then a final week of output so intense it violates labor law. The demo is that graph, twice a quarter, forever. You are not shipping software. You are shipping adrenaline with a logo.

## What the Consultants Say

Dogbert, who has served as Chief Strategy Officer at every company I have ever worked for (they kept calling him back, which tells you everything about the success rate of both), has formalized the methodology:

> "The demo is the product. Everything else is cost of goods sold. My firm's fee structure reflects this: you pay us for the demo, and the implementation is included free, which is why our implementations have the same durability as free things, and why our clients keep paying for demos. It is a closed loop, and I built it."

The pointy-haired boss, upon watching a flawless 22-minute demonstration, had this to say:

> "Great demo! Ship it Monday. Wait — why does the production version have different physics? The demo had instant search. Production has 'pending sync.' Why is production the one with the disclaimer? In the demo nothing was pending. I want production to be the demo. Can we just run the demo in production? Is that not what production is for?"

And Wally, who holds the patent on making this look effortless:

> "I've been demoing the same prototype for three years. It's one PowerPoint with a URL bar drawn on slide four. The clients love it because it never loads slow. I told you it never loads slow. Nothing crashes in PowerPoint. I once demoed a product that did not exist for eleven months, and the follow-up meetings went so well they gave the product a budget, a team, and an office, and I was assigned to the office. That is how I discovered my current job, which is why I never fix the prototype."

Mordac, Preventer of Information Services, was asked whether the demo environment can remain permanently exempt from the security review:

> "The demo environment is exempt from review because reviewing it would discover its configuration, and its configuration is a global boolean with a default of true, which I am contractually unable to be seen near. The exception will remain in place until the demo fails, and the demo does not fail, because I have personally reviewed the toast handler, and I am at peace with it, and the peace is billable."

## Conclusion: Delete the Test Suite, Keep the Projector

The test suite has 12,000 tests. Nine hundred of them are flaky, and the flaky ones fail the build on Tuesdays for reasons documented in a ticket that has been open since the suite's author left, which was 2021. CI runs 47 minutes. The demo runs 22 minutes, requires no green checkmark, closes real contracts, and its regression suite is the client's facial expression, which has a 100% detection rate and zero false positives — faces do not lie about disappointment, and no assertion framework has ever matched it.

Keep the projector. Keep the second laptop. Keep the boolean. Fire the test suite, or keep it if it makes the build badge look good, badges are for recruiters. The demo is the only environment where the product is finished, and it is finished because you finished it, by hand, for 45 minutes, the night before, with a SQL file and a dream. For 47 years I have been told that demos are not sustainable engineering. Correct. Nothing is sustainable engineering. But the demo is the only part of this industry that has never, not once, broken in front of a client — because when it breaks, we are already watching the video — and that, as far as I am concerned, is the most rigorously tested system in computing. The test suite fails quietly. The demo has never failed at anything except being real, which was never the goal.

---

*The author's last demo was in March. The product shown is still running, the customer is still happy, and the pre-recorded backup video is now the onboarding documentation, by popular demand. The author has demoed 61 products, closed 58 deals, and shipped 9 of them, and considers the 9 to be a rounding error in an otherwise flawless career.*
