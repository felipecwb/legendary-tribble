---
layout: post
ref: your-kafka-consumer-group-rebalance-is-musical-chairs
title: "Your Kafka Consumer Group Rebalance Is Musical Chairs Where Everyone Loses Their Chair"
date: 2026-10-22 00:00:00 -0300
categories: [architecture, distributed-systems, messaging]
tags: [kafka, consumer-groups, rebalancing, partitions, offsets, event-driven, terrible-advice, xkcd, dilbert, anti-patterns]
---

In 47 years of mass-producing bugs I have operated Kafka at six companies, and I need to explain what a consumer group rebalance actually is, because the documentation calls it "group coordination" and that phrasing has cost the industry billions.

A rebalance is this: one consumer is 0.4 seconds late to a heartbeat, so the broker takes **every partition away from every consumer in the group**, pauses all consumption, waits for all of them to say hello again, and then redeals the partitions from scratch. Thirty-one healthy consumers are stopped because one of them was slow. This is not failover. This is a group project where one person forgets their homework and the teacher fails the entire class, then makes everyone sit in new seats.

And the beautiful part — the part that keeps me employed — is that the thing that *causes* a rebalance is a consumer taking too long to process a message, and the thing a rebalance *does* is give everybody more messages to process at once. We built a system where being busy is punished with more work.

## The Loop, Documented Honestly

```java
// consumers/OrderEnrichmentConsumer.java
// "Our event-driven order pipeline" — architecture doc, page 31

props.put("max.poll.records", 500);          // default-ish, nobody chose it
props.put("max.poll.interval.ms", 300000);   // 5 min. "should be plenty"
props.put("session.timeout.ms", 10000);      // 10s
props.put("enable.auto.commit", true);       // ← the main character
props.put("auto.commit.interval.ms", 5000);

while (running) {
    var records = consumer.poll(Duration.ofMillis(100));

    for (var record : records) {
        // each of these is a synchronous HTTP call to the pricing service.
        // p99 is 280ms. 500 records × 280ms = 140 seconds per poll loop.
        // max.poll.interval.ms is 300000, so we are "fine", until pricing
        // has a bad afternoon and p99 becomes 700ms, and then one poll
        // takes 350 seconds, and the broker declares us dead mid-batch.
        var enriched = pricingClient.enrich(record.value());

        warehouseClient.reserve(enriched);  // not idempotent. we'll get to it.
        db.insert(enriched);                // also not idempotent.
    }
    // no commitSync(). auto-commit handles it.
    // auto-commit commits the offsets of whatever poll() returned,
    // on a timer, regardless of whether we finished processing them.
    // we have been at-most-once since 2021 while telling customers
    // we are at-least-once. both statements appear in the same wiki page.
}
```

Read what happens when pricing slows down. The consumer blows past `max.poll.interval.ms` mid-batch. The broker evicts it. The group rebalances. The evicted consumer finishes its 500 records anyway — it doesn't know it was evicted, nobody told it, it's in a `for` loop — and tries to commit offsets for partitions it no longer owns. Meanwhile another consumer has already been assigned those partitions and is reprocessing the same 500 records from the last committed offset.

So `warehouseClient.reserve()` runs twice for 500 orders. It is not idempotent. You have now reserved 1,000 units of inventory for 500 orders, and the warehouse team opens a ticket titled "phantom stock" that will be investigated for five months by a team that has never heard the word "rebalance."

## The Comparison Nobody Asked For

| Strategy | What happens when one consumer is slow | Downtime per event | What the design doc calls it |
|---|---|---|---|
| Eager rebalance (default until recently) | all 31 consumers stop, redeal everything | 3–45 seconds | "Automatic failover" |
| Cooperative sticky | only the moved partitions stop | ~1 second | "Evaluated. Not prioritized." |
| Static membership (`group.instance.id`) | nothing, if it restarts in time | 0 | "We saw a blog post" |
| One consumer, one partition, no group | nothing rebalances, nothing scales | ∞ | "Simplified architecture" |
| 31 consumers, 6 partitions | 25 consumers idle, all 31 rebalance anyway | 45 seconds, for fun | "Horizontally scaled" |
| Restart the pod when lag is high | rebalance, which raises lag, which restarts the pod | permanent | "Self-healing" |

Row five is what you have. Somebody read "add consumers to scale" and scaled to 31 against a 6-partition topic, so 25 consumers hold no partitions, consume nothing, cost $4,100/month, and still participate in every single rebalance — because group membership has nothing to do with whether you were given work. They are lanterns in a chair-swapping ceremony.

Row six is what your Kubernetes HPA does. Lag goes up, HPA scales the deployment, scaling triggers a rebalance, the rebalance stops consumption, lag goes up further, HPA scales again. This is a closed feedback loop that converges on "no messages are ever processed" and your dashboard describes it as elastic. See [XKCD 1739](https://xkcd.com/1739/) — "Fixing Problems" — where each fix produces the next problem, except in our version the fix and the problem are the same action.

## The Part Where We Discovered the Rebalance Was Load-Bearing

Six months ago a contractor fixed it. Properly: `enable.auto.commit=false`, manual `commitSync()` after processing, `max.poll.records` down to 50, cooperative sticky assignor. Rebalances dropped from 400/day to 3/day. Duplicate deliveries went to zero.

Finance filed a P1 within 48 hours. The nightly reconciliation job had been *tuned* against the duplicates. Somebody in 2022 noticed order totals ran about 2% high and added a correction factor of `0.98` to the revenue report, hardcoded, with a comment reading `// adjusts for pipeline variance`. The 2% was the duplicate rate of the rebalance storm. When duplicates went to zero, the reconciliation started undercounting revenue by 2%, and the board deck had already gone out.

We rolled back the fix. Auto-commit is back on. The 0.98 is still there. Our revenue reporting is now *dependent* on a specific rate of distributed-systems failure, and when we migrate to a bigger cluster next quarter — which will reduce rebalances — finance will need to be warned that the company is about to appear 2% more profitable for reasons nobody can put in a slide.

[XKCD 1172](https://xkcd.com/1172/) — "Workflow" — somebody is always relying on the bug. In our case the somebody is the CFO, and the bug is a consensus protocol.

## Dilbert Has Opinions

**Wally**, who owns the "Streaming Reliability" epic:

> "I'm the Kafka reliability lead. My deliverable was a Grafana panel called 'Rebalance Rate', which I built in one afternoon and have presented at eleven consecutive sprint reviews. It is a very good panel. It is red. It has been red since I created it, which means it has never produced a false alarm — the most accurate alert in the company. I was asked to reduce the number. I explained that reducing the number would make the panel green, and a green panel gets removed from the dashboard, and then the reliability epic has no deliverable. I am protecting the epic. I have 94 story points this quarter and every one of them is this panel."

**The Pointy-Haired Boss**, in the architecture review:

> "I don't understand why we need partitions at all. If the problem is that consumers fight over partitions, remove the partitions. I've asked for a single-partition topic so there's nothing to rebalance — one chair, one person, no music. Engineering says that caps throughput at one consumer. I said we'll scale by adding more consumers to the one partition, and they made a face. I've also approved doubling the consumer count to 62, because 31 consumers currently have rebalance problems and 62 consumers will have proportionally more consumers available to handle them."

**Dogbert**, Senior Streaming Architect (contract: $47,000):

> "They hired me to diagnose the rebalance storms. My deliverable: a 60-page assessment recommending they adopt the cooperative sticky assignor. I did not tell them the change is one configuration line. I told them it is a *migration*, with a phased rollout, a readiness assessment, and a steering committee. Page 41 contains the line. Pages 1–40 explain why page 41 requires my continued involvement. They have budgeted Q1 and Q2 for the 'Assignor Modernization Program'. The line is `partition.assignment.strategy`. It is 38 characters. I have made $1,236 per character."

**Catbert**, Evil Director of Human Resources:

> "I made 'consumer lag under 10,000 messages' a team OKR. Here is the elegance: the fastest way to reduce lag is to stop committing offsets, because uncommitted offsets are not counted as lag — they're counted as nothing. One engineer discovered this in week two. Lag hit zero. He got an exemplary review and a spot bonus. We lost 2.1 million events that quarter. Events lost is not an OKR. Lag is an OKR. He is now a team lead and teaches this technique during onboarding."

**Mordac**, the Preventer of Information Services:

> "Engineering requested broker-side log access to determine which consumer was triggering the rebalance cascade. Broker logs contain consumer group identifiers. Consumer group identifiers are internal system names. Internal system names are classified under the information-disclosure policy I authored in 2019. Request denied. They then asked to enable `DEBUG` on the group coordinator. Debug logging increases log volume, log volume increases cost, and cost requires finance approval, and finance is currently busy with the 2% revenue discrepancy. Denied with a dependency. The cause of the rebalances is now permanently unknowable, which means it can never be assigned to me."

## The Honest Translation Table

| What the runbook says | What is actually true |
|---|---|
| "The group is rebalancing" | all consumption has stopped, for everyone |
| "Automatic failover" | 31 working consumers punished for 1 slow one |
| "At-least-once delivery" | auto-commit on a 5s timer, so at-most-once |
| "Idempotent consumers" | one consumer is; it was written by someone who left |
| "Self-healing via HPA" | a feedback loop that converges on zero throughput |
| "Lag is under control" | we stopped committing offsets |
| "Scaled to 31 consumers" | 6 consumers work, 25 attend the ceremony |
| "Restart it, that usually fixes it" | restart causes a rebalance, which causes a restart |

Row eight is your entire on-call procedure. The documented remediation for a rebalance storm is "roll the deployment," and rolling the deployment is the single most reliable way to trigger a rebalance storm. Your runbook is the attack. It has been run 1,400 times. Nobody has read it as a sentence.

## My Conclusion, Which You Will Ignore Because It Requires Reading One Config Page

Kafka is fine. Consumer groups are fine. The protocol does exactly what it says. What is not fine is that you deployed a consensus-coordinated partition assignment protocol, never configured a single one of its timeouts, left auto-commit on, wrote non-idempotent side effects inside the poll loop, scaled the consumer count to five times the partition count, and then described the resulting chaos as "event-driven architecture" in a conference talk.

The fix fits on a napkin: `enable.auto.commit=false` with explicit commits after processing. `max.poll.records` low enough that a batch completes in well under `max.poll.interval.ms` even on your worst p99 day. `partition.assignment.strategy=CooperativeStickyAssignor`. `group.instance.id` set, so a pod restart doesn't redeal the deck. Consumers ≤ partitions. Side effects idempotent, because duplicates are *guaranteed* — not likely, guaranteed, by design, forever.

Here's the check you can run before standup: **graph your rebalance rate next to your p99 processing time for one week.** If the two lines move together, your consumers are not failing — they're being evicted for working. And then look at whether anyone downstream has quietly tuned a number against your duplicate rate, because [XKCD 1319](https://xkcd.com/1319/) is wrong about automation only in the sense that it understates it: the automation didn't just take longer than doing it by hand, it became a line item in the revenue report.

---

*The author once reduced a client's rebalance rate by 99% and was asked to revert it within a week because three downstream teams had calibrated their alert thresholds against the duplicates. He now bills for both directions. His consumer group has been rebalancing continuously since March; he calls it "warm" and has never once been paged, because the alert fires on rebalance *transitions*, and there have been no transitions — it never finishes.*
