---
layout: post
ref: your-dead-letter-queue-is-a-morgue-nobody-visits
title: "Your Dead Letter Queue Is a Morgue Nobody Has Ever Visited"
date: 2026-10-20 00:00:00 -0300
categories: [architecture, messaging, anti-patterns]
tags: [dead-letter-queue, dlq, kafka, rabbitmq, sqs, retries, message-queues, terrible-advice, dilbert, anti-patterns]
---

In 47 years of mass-producing bugs, I have built message pipelines on top of RabbitMQ, Kafka, SQS, ActiveMQ, a Postgres table called `queue_v2_final`, and once a shared folder on a Windows server that three teams polled with a `while true` loop. And I am here to tell you the secret that the entire event-driven-industrial complex has spent $47M hiding: **nobody has ever read a dead letter queue.** The DLQ is not an error-handling strategy. The DLQ is a *morgue*. You wheel the bodies in, you close the drawer, you write "handled" in the design doc, and you never come back. There is no coroner. There was never a coroner. The coroner was on the hiring plan for Q3 and got cut.

This is Dead-Letter-Driven Architecture, and it is the only durability guarantee your system actually offers. Everything else is vibes with retries.

## The Complete Methodology

When someone on my team says "don't worry, failures go to the DLQ", they mean this. I have reviewed this config 847 times. It is always the same 40 lines, and the outcome is always decided by the line nobody reads:

```yaml
# consumer.yaml — resilient event processing pipeline, v1
# (2016, untouched since 2016; the author left in 2017 and is
#  now a beekeeper. The bees are monitored. The DLQ is not.)

consumer:
  topic: order.created
  group_id: order-processor-v2    # v1 still exists, still consuming,
                                  # nobody knows which one is authoritative

retry:
  attempts: 3                     # 3 is the industry standard, chosen because
                                  # 2 feels careless and 4 feels obsessive
  backoff: 0ms                    # exponential backoff was in the backlog.
                                  # the backlog was archived. so was the outage.
  retry_on: [Exception]           # retries validation errors 3 times, in case
                                  # the data heals itself. it never has.

dead_letter:
  enabled: true                   # ← this line is the entire error strategy
  topic: order.created.dlq
  retention: 7d                   # the bodies are cremated on day 7.
                                  # nobody has ever arrived on day 6.
  alert_on_message: false         # alerting was "too noisy" during the
                                  # launch. the launch was in 2016.
  redrive: manual                 # manual means "a human will decide".
                                  # the human is a role, not a person.
  owner: platform-team            # platform-team was reorged into
                                  # Platform Enablement, then into nothing

monitoring:
  dashboard: grafana/dlq-overview # built once, for the demo
  panel_1: "DLQ depth"            # currently 4,712,009
  panel_2: "DLQ depth (rate)"     # flat, which leadership reads as "stable"
  pager: null                     # the pager is for things we fix
```

That is the entire field of resilient messaging. One `enabled: true`, one retention window, one owner who is a hyphenated noun rather than a human being. Notice that `alert_on_message` is `false` and `redrive` is `manual`: together these two settings form a sentence, and the sentence is *"no one will be told, and no one will act"*. You did not build error handling. You built a waiting room with a 7-day eviction policy and no receptionist.

## The Table Nobody Wanted

| Failure handling strategy | What actually happens | Who finds out | Cost |
|---|---|---|---|
| Crash the consumer loudly | pager fires, human fixes it in 11 minutes | you, immediately | one bad night |
| Retry 3× then DLQ | message dies quietly, metric goes up by 1 | nobody | $47M, eventually |
| Retry 3× then DLQ **with alerting** | someone is woken up, learns something | you, correctly | a Tuesday |
| Retry forever, no DLQ | poison pill blocks the partition for 9 hours | the entire company | the quarter |
| `except: pass` | identical outcome to the DLQ, zero infrastructure | nobody, ever | $0 — and honest |
| DLQ with 7d retention, no consumer | evidence of the bug is destroyed on schedule | the customer, in a support ticket | your reputation |

Study row five. `except: pass` and a fully provisioned dead letter queue with no redrive consumer are **behaviourally identical**: the message is gone and nobody is informed. The difference is that one costs nothing and is universally condemned in code review, and the other costs $4,700/month in retained storage and gets a slide in the architecture review titled "Reliability". Same semantics. Different budget line. One of them is called an anti-pattern and the other is called event-driven.

## "But We Have a Runbook for the DLQ!"

You do. I have read it. It is 3 lines long, written during onboarding by someone who no longer works there, and it says:

> 1. Check the DLQ.
> 2. Determine why the messages failed.
> 3. Redrive them.

Step 2 is the entire job and it is one sentence. Step 3 is where the career-ending event lives, because here is what redrive actually looks like at 3 AM when there are 4.7 million messages in there spanning 14 months of schema versions:

```python
# redrive.py — "just replay them", they said
# written at 03:14, by a person who was not on call but was awake

for msg in dlq.consume(topic="order.created.dlq"):
    try:
        main_topic.produce(msg.value)   # replay it, what could go wrong
    except Exception:
        dlq.produce(msg.value)          # back to the DLQ it goes.
                                        # this is a cycle. we invented a
                                        # closed-loop system. it is called
                                        # a DLQ ouroboros and it costs
                                        # $4,700/month to spin forever

# 14 months of messages. Four schema versions. Three of them have a field
# called `customer_id` that was renamed to `user_id`, then to `account_id`,
# then back. The consumer that can parse v1 was deleted in the migration.
#
# Of the 4.7M messages:
#   - 2.1M are duplicates of orders that eventually succeeded on retry
#     (replaying them bills 2.1M customers a second time)
#   -   900K reference products that no longer exist
#   -   1.6M are from the incident in March and are already in the
#       reconciliation spreadsheet, which is the real source of truth
#   -   112K are genuinely lost revenue and nobody can tell which 112K
#
# The engineer ran this script. Then they ran it again, because the first
# run partially completed and there was no idempotency key, because
# idempotency was going to be added in Q3. See my other article.
```

This is the part the architecture diagram does not show: **a DLQ without a redrive consumer isn't a safety net, it is a liability that accrues interest.** Every day it grows, the cost of ever reading it grows faster, because the schema drifts, the downstream state diverges, and the window in which replaying would have been *correct* closed months ago. On day one, the DLQ contains a bug. On day 400, it contains an archaeology project with a billing risk attached.

See [XKCD 2347](https://xkcd.com/2347/) — "Dependency" — the entire modern digital infrastructure balanced on one tiny unpaid block. Your reliability story is balanced on one `enabled: true` that was typed by a beekeeper nine years ago and has never been load-bearing in the way anyone believes.

And the companion piece, [XKCD 1739](https://xkcd.com/1739/) — "Fixing Problems" — where fixing the bug reveals the code only ever worked by accident. Every DLQ redrive in history has ended with this discovery: the pipeline was never processing those messages, and the business continued, which means either the messages didn't matter or the money was quietly absorbed by someone's reconciliation spreadsheet. Nobody investigates which. Investigating is how you find out it was the second one.

## Wally Understood Immediately

**Wally**, who has owned the DLQ dashboard since 2019 and has never once opened it:

> "I'm the DLQ owner. It's the best assignment in the building. The queue depth only goes up, which means my metric has a trend, and leadership loves a trend. Nobody can ask me to fix it, because fixing it requires 'determining why the messages failed', and that's step 2, and step 2 has no estimate. I've been blocked on step 2 for seven years. When the depth hit one million I made a dashboard panel for it. I got a kudos in the all-hands for 'visibility'. The visibility is a number. The number is a count of things I have not looked at. I am paid to not look at it, and I have never missed a deadline."

**The Pointy-Haired Boss**, presenting reliability at the quarterly review:

> "Our error rate is 0.00%. All failures are captured in the dead letter queue, which means they are handled. I checked the definition of 'handled' and it means 'in a queue', and they are in a queue, so we are at 100%. Wally showed me the dashboard. The line goes up, which in every other chart in this deck means growth. Someone from engineering asked what happens on day 7 of retention. I said the messages are 'retired'. He said 'deleted'. I said 'retired, with dignity'. I've added a slide titled 'Zero Unhandled Errors' and it is the only slide the board remembered."

**Dogbert**, consulted as Chief Reliability Officer:

> "I sell a product called DLQ Insights™. $4,700/month. It connects to your dead letter queue and tells you how many messages are in it. You already know how many messages are in it; the number is on your dashboard. What I sell is the *email*. The email arrives weekly, says 'DLQ depth: 4,712,009 (+0.3%)', and goes to a distribution list that auto-archives. My renewal rate is 100%, because cancelling would mean someone has to admit they never read it, and admitting that means someone owns it. Nobody will ever own it. I have built a perpetual revenue machine out of a number that no human being has ever acted upon."

**Catbert**, Evil Director of Human Resources:

> "I made DLQ depth a team OKR. 'Reduce dead letter queue backlog by 20%.' They hit it in one afternoon by lowering the retention from 7 days to 5. Backlog down 28%. Exceeded target. Bonuses paid. The messages were deleted, which is technically a reduction, and 'reduction' was the word in the OKR — I wrote it that way on purpose. Next quarter's target is another 20%. Retention is now 4 days. By 2028 the retention will be 0 seconds and the DLQ will be empty at all times, and that team will have the best reliability metrics in the company, and the company will have no error handling at all. I will be promoted. The bees will have better monitoring than our orders pipeline."

**Mordac**, the Preventer of Information Services, on the access request:

> "You want read access to the production DLQ to investigate 112,000 lost orders. The DLQ contains customer PII. PII access requires a privacy review. The privacy review requires a data retention justification. Your data retention is 7 days. The review takes 11 days. By the time you are approved to look at the messages, the messages will not exist, which resolves the privacy concern entirely. Request approved. Ticket closed as 'completed'. You may re-apply for the next 112,000."

## The Honest Table

| What you tell people | What is true |
|---|---|
| "Failures go to the DLQ" | failures go to a 7-day timer |
| "We have full durability guarantees" | we have full *storage*, which is a different noun |
| "It's monitored" | it has a panel; the panel has no pager |
| "We'll redrive after the incident" | the schema will have changed by then |
| "The DLQ gives us a safety net" | the net has no floor; the floor is `retention` |
| "Nothing is ever lost" | nothing is ever *looked at*, which feels the same from the inside |

Notice the last row. This is the real product of the dead letter queue: not durability, but **the feeling of durability**, delivered at the exact moment of system design, consumed entirely by the architect, and never once audited. You bought a coffin and called it a hospital.

## My Conclusion, Which You Will Ignore Because It Would Mean Opening the Queue

Keep your DLQ. I am not here to take your safety net; I am here to tell you it is a drawer. Dead letter queues are a genuinely good idea that your organisation has converted into a disposal mechanism with extra steps and a monthly invoice. The queue is not the problem. The problem is that there is no human being on the other end of it, and there has never been a plan to hire one, and the retention policy is quietly solving the problem for you by destroying the evidence on a schedule.

Here is the only test that matters, and you can run it this afternoon without a design doc: **open your dead letter queue right now and read the oldest message in it.** Not the count. The message. If you can parse it with the current schema, your pipeline is younger than you think. If you cannot, then you have never had error handling — you have had a landfill with a dashboard, and you have been paying rent on it since 2016.

Oh, and if the queue is empty? Check the retention. Day 7 came. Day 6 never did.

---

*The author owns 14 dead letter queues across 4 companies, two of which no longer exist as legal entities. He has redriven exactly one message, in 2019, by hand, into the wrong topic, which triggered an incident larger than the original failure. The remaining 11.4 million messages were handled by the retention policy, which is the only component in his architecture that has never missed an SLA. His morgue has no coroner. It has a Grafana panel, and the panel is beautiful, and the line goes up.*
