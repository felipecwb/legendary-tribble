---
layout: post
ref: ntp-is-just-peer-pressure-for-clocks
title: "NTP Is Just Peer Pressure for Clocks"
date: 2026-10-23 00:00:00 -0300
categories: [infrastructure, distributed-systems, operations]
tags: [ntp, clock-skew, monotonic-clock, timestamps, leap-second, chrony, terrible-advice, xkcd, dilbert, anti-patterns]
---

In 47 years of mass-producing bugs I have been in four war rooms that were ultimately caused by a computer being wrong about what time it was, and in all four cases the postmortem blamed the database. So let me explain what NTP actually is, because everyone calls it "time synchronization" and that phrasing has eaten more engineer-years than Kubernetes.

Your server does not know what time it is. It has never known. It has a crystal that vibrates at approximately the right rate, and "approximately" means it drifts about 1 to 20 seconds per day depending on how hot the rack is. So four times a minute your machine asks some strangers on the internet what time they think it is, discards the ones whose answers it dislikes, averages the rest, and then **adjusts its own sense of reality to match the consensus of the group**. That is not synchronization. That is peer pressure. Your server is a teenager.

And unlike a teenager, it can go *backwards*. `clock_gettime(CLOCK_REALTIME)` is allowed to return a smaller number than it returned four milliseconds ago, because `ntpd` decided the machine was 1.2 seconds ahead and stepped it back. Every piece of code you have ever written that subtracts two timestamps to measure a duration can return a negative number. It has. You logged it as `-1` and moved on.

## The Function That Caused the Outage

```python
# services/billing/metering.py
# "high-precision usage metering" — the vendor contract, section 4

import time

def charge_session(session):
    start = time.time()          # ← wall clock. asks the committee.
    run_customer_workload(session)
    elapsed = time.time() - start

    # elapsed is "seconds of compute". we bill $0.000094 per second.
    # on 2026-03-14 at 02:11 UTC, chrony stepped this box back 1.4s
    # because it had been talking to a stratum-2 peer that was itself
    # drifting, and the peer got corrected, and the correction
    # propagated down to us mid-workload.
    #
    # 11,400 sessions that night had elapsed < 0.
    # 400 of them had elapsed around -1.39, which we multiplied by
    # the rate, and issued as a NEGATIVE line item.
    # we paid 400 customers to use our product.
    # the largest credit was $4.12. the smallest was $0.0003.
    # finance noticed in June.

    session.billable_seconds = elapsed
    ledger.append(session.customer_id, elapsed * RATE)

    # and here's the part I like:
    if elapsed > SLA_THRESHOLD:
        alerts.page_oncall("slow session")
    # negative numbers are not greater than the threshold.
    # so the 1.4 second stall that affected every session on the box
    # generated zero alerts. the clock jump hid the clock jump.
```

`time.time()` is a *question about the world*. `time.monotonic()` is a *measurement*. They have the same shape, they both return floats, your linter has no opinion, and exactly one of them is correct for measuring how long something took. Thirty years of this API existing and the wrong one is still the one that autocompletes first.

## The Comparison Nobody Asked For

| Time source | What it actually tells you | Can it go backwards? | What the design doc calls it |
|---|---|---|---|
| `CLOCK_REALTIME` / `time.time()` | what the committee currently believes | **yes, arbitrarily** | "the time" |
| `CLOCK_MONOTONIC` | how long since boot, honestly | no | "unnecessary complexity" |
| `CLOCK_MONOTONIC_RAW` | same, ignoring NTP slew | no | never heard of it |
| Database `NOW()` | what *one specific other machine* believes | yes, and differently | "the source of truth" |
| Client-supplied timestamp | what the user's phone believes, in 2019 | it's already wrong | "event time" |
| `uptime` on the box you ssh'd into | the only number anyone trusts | n/a | "my investigation" |
| Pod with no NTP at all (`hostNetwork: false`, no chrony) | the hypervisor's mood at boot | drifts 40s/week | "containers handle that" |

Row seven is most of your fleet. Somebody decided time sync was a host concern, the host is a managed node you don't control, the node pool was created from an AMI where `chronyd` is masked, and your pods have been collectively 18 to 400 milliseconds apart from each other for two years. Nobody noticed, because the only thing that compares timestamps across pods is the log aggregator, and when the log aggregator shows events out of order everybody blames the log aggregator.

And row five is the one that will end you. You accept `event_timestamp` from clients, because "event time is more accurate than ingestion time," and now your analytics pipeline contains 90,000 events from 1970 (uninitialized clocks), 14,000 from 2038 (someone's test fixture), and a steady trickle from three hours in the future belonging to a specific Android build with a broken timezone database. Your "daily active users" metric has a tail that extends to the end of the Unix epoch. See [XKCD 1883](https://xkcd.com/1883/) — the supervillain plan is just daylight saving time, and you have voluntarily let every phone on earth participate in it.

## The JWT That Was Valid Before It Existed

```
auth-service   (clock: +0.00s)  issues token at iat=1774000000
api-gateway    (clock: -2.30s)  receives token, local now=1773999997
               → 1773999997 < iat → "token used before issued" → 401
```

Two services, 2.3 seconds apart, and a login flow that fails for exactly the users whose request lands on gateway pod 7. Not all users. Not reproducibly. 0.8% of logins, forever, with a support macro that says "please try again" — and it works on the second try, because by then the request hit a different pod.

The fix, which took us five months to find and nine minutes to deploy, was `clockTolerance: 30`. One field. [XKCD 1205](https://xkcd.com/1205/) tells you how much time it's worth spending to automate a task; it does not have a row for "five months of intermittent 401s that you classified as user error."

The actual lesson is worse than the bug: **every distributed system you run has a hidden parameter called "how wrong can our clocks be before correctness breaks," and nobody on your team knows its value.** For token validation it was 0 seconds until we changed it. For your cache TTLs it's whatever makes things expire early on one box. For your distributed lock it's the difference between "mutual exclusion" and "two workers writing to the same row while both hold a valid lease."

## The Part Where We Discovered the Drift Was Load-Bearing

Last year a new SRE fixed the fleet. Properly: `chrony` on every node, four upstream sources, `makestep` limited, a Prometheus alert on `node_timex_offset_seconds > 0.05`. Max skew across 900 pods went from 400ms to under 2ms. It was genuinely excellent work and I told him so.

The order-events deduplication job broke that night.

Someone in 2021 had written dedupe as "group by `(user_id, action)` and keep the earliest `created_at`, discarding anything within 500ms." It worked because double-click submissions landed on two different pods whose clocks were ~200ms apart, which spread the duplicates *just enough* that the 500ms window caught both and kept one. With clocks synchronized to 2ms, both inserts got timestamps inside the same millisecond, `created_at` tied, the `ORDER BY created_at LIMIT 1` became non-deterministic across replicas, and the dedupe job kept **both** rows on the read replica and **one** row on the primary.

Two hundred and six customers were charged twice. The root cause of a double-billing incident was that we made our clocks accurate. I have had to say that sentence out loud, to a director, with a straight face.

We did not revert the clock fix. We fixed the dedupe to use an idempotency key, like it should have in 2021. But for eleven days the official incident doc listed the contributing factor as "time synchronization improvements," and that line is now cited internally as a reason to be cautious about infrastructure hygiene. [XKCD 1172](https://xkcd.com/1172/) — somebody is always depending on the bug. This time the somebody was us, and the bug was physics.

## Dilbert Has Opinions

**Wally**, who owns the "Temporal Integrity" initiative:

> "I was made the time-sync owner in 2023 because I was in the room. My deliverable is a dashboard called 'Fleet Clock Health' with a single stat panel showing the maximum offset across the fleet. I set the panel's query to `max(node_timex_offset_seconds)` and then — this is the craftsmanship — I set the unit to *seconds* and the decimal places to **zero**. Every offset under half a second displays as `0`. The panel has read `0` for three years. It read `0` during the billing incident. It will read `0` during the next one. I have never been paged, I have presented this panel at two all-hands, and the word 'integrity' is in my job title."

**The Pointy-Haired Boss**, in the infrastructure review:

> "I don't see why we're paying for time. Time is free. I've asked the team to just use the time from the internet, and they said that's exactly what NTP is, and I said then why is there a line item. I've also proposed we standardize on a single clock — one server that has the real time, and every other server asks it. They told me that's a single point of failure. I said it's a single point of *truth*, which is a best practice, and I've seen it in a diagram. Additionally I do not want servers in different timezones anymore. Please consolidate the timezones. I've put it in the roadmap under 'simplification'."

**Dogbert**, Chief Chronometry Consultant (engagement: $47,000):

> "They hired me to resolve the clock drift. I delivered a 71-page assessment titled *Toward a Unified Temporal Reference Architecture*, which recommends a three-phase migration to a hybrid logical clock with a steering committee and a quarterly review. Pages 1 through 63 establish the strategic necessity. Page 64 contains the actual remediation, which is `systemctl enable --now chronyd`. Page 65 is a glossary. They have funded Phase One. Phase One is me returning in February to assess the readiness of the assessment. I have billed $47,000 to type a command that is in the Red Hat quick-start guide, and they have asked me to present it to the board."

**Catbert**, Evil Director of Human Resources:

> "I made 'reduce timestamp-related defects by 40%' an engineering OKR. Observe the mechanics: a defect is only *timestamp-related* if an engineer tags it that way in the tracker. Within one sprint, zero bugs were being tagged timestamp-related. The JWT issue was tagged `auth`. The billing credits were tagged `finance-integration`. The out-of-order logs were tagged `vendor-logging`. We hit 100% reduction, the team received the bonus, and the taxonomy is now so distorted that the word 'clock' does not appear anywhere in our defect history. I am considering an OKR about memory leaks next."

**Mordac**, the Preventer of Information Services:

> "Engineering requested outbound UDP port 123 to public NTP pools. Port 123 is a known amplification-attack vector, as documented in a 2014 advisory I have printed and laminated. Request denied. They then requested an internal NTP appliance. Hardware requests require a capacity review, and capacity reviews are scheduled quarterly, and the current quarter's review is full. They then asked which internal host *does* have accurate time so they could point at it. That information constitutes network topology disclosure. Denied. The fleet now syncs against whatever the hypervisor offers, which is nothing, and I have successfully ensured that no clock in this organization can be attacked, because no clock in this organization is correct."

## The Honest Translation Table

| What the incident doc says | What is actually true |
|---|---|
| "Clocks are synchronized via NTP" | NTP is installed on 60% of the fleet, running on 41% |
| "We use UTC everywhere" | you use UTC in the database and local time in four cron jobs |
| "Timestamps are accurate to the millisecond" | they have millisecond *precision* and 300ms of *error* |
| "Events are processed in order" | they are processed in the order the clocks felt like |
| "Measured response time: 0ms" | wall clock went backwards mid-request |
| "Intermittent auth failures, cause unknown" | one pod is 2.3 seconds behind and tokens aren't valid yet |
| "The leap second is handled" | nobody here has read what your kernel does with it |
| "Time sync improvements caused the incident" | your dedupe logic was never correct, drift was hiding it |

Row three is the one I want tattooed on a wall. Precision is how many digits you print. Accuracy is whether they mean anything. Your logs show `2026-10-23T02:11:44.847231Z` — six decimal places of total fiction, because the box emitting it last agreed with the rest of the fleet four hundred milliseconds ago. You are printing microseconds on a measurement with a third of a second of error, and then using those microseconds to establish causality between two services during a postmortem. See [XKCD 1179](https://xkcd.com/1179/): ISO 8601 solved the *format*, which everybody celebrated, and then we all assumed the hard part was over.

## My Conclusion, Which You Will Ignore Because It Is Boring

NTP works. It is a 1985 protocol that quietly delivers millisecond accuracy over a network it has no right to trust, and it is one of the great unglamorous triumphs of computing. Nothing here is NTP's fault. What is your fault is that you built business logic on `CLOCK_REALTIME`, never measured your fleet's actual skew, used timestamps as identity, derived ordering from wall clocks across machines, and then described the resulting mess as an "eventual consistency challenge" at a conference.

The whole fix is unglamorous enough to fit in a paragraph. Use `monotonic()` for every duration, every timeout, every retry backoff, every rate limiter — if you are subtracting two times to get an elapsed interval and you called `time.time()`, that is a bug today, not a risk. Use `REALTIME` only for "what to display to a human" and "roughly when did this happen." Never derive identity or ordering from a timestamp: use an idempotency key for dedupe and a sequence, ULID, or logical clock for order. Set explicit clock tolerance on every JWT and TLS validation path, and write the number down. Run `chrony` with four sources and alert on offset, not on whether the daemon is alive — a confidently wrong clock reports perfect health.

Then do the thing nobody does: **graph `node_timex_offset_seconds` across your whole fleet, with three decimal places, and find the worst pod.** Not the average. The average is always zero; that's what averages do to symmetric errors. Find the single worst offender and then ask what runs on it. That is the hour that will teach you the most about your own system this quarter, and [XKCD 1739](https://xkcd.com/1739/) is otherwise what your next six months look like.

---

*The author once shipped a retry loop using wall-clock deadlines. An NTP step adjustment of +4 hours made every pending retry immediately "expired," which drained a 40,000-message queue into a dead-letter topic in nine seconds — the fastest his pipeline has ever moved. He included the throughput figure in his performance review and received a commendation for "latency improvements." His laptop is still set to a timezone that does not observe daylight saving time, in a country that does, and he considers this a form of protest.*
