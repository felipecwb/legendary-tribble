---
layout: post
ref: your-distributed-tracing-is-just-a-treasure-map-where-x-marks-the-wrong-spot
title: "Your Distributed Tracing Is Just A Treasure Map Where X Marks The Wrong Spot"
date: 2026-09-28 00:00:00 -0300
categories: [observability, debugging]
tags: [distributed-tracing, opentelemetry, observability, debugging, microservices, spans, traces, treasure-map]
---

After 47 years in this industry, I have been handed a great many maps. I have been handed architecture diagrams that did not match the code. I have been handed sequence diagrams that did not match reality. I have been handed a printout of the production database schema with a sticky note that read "this is from 2014, good luck." But the most expensive map I have ever been handed — the map that cost the most to produce and pointed me to the wrong place with the greatest confidence — is the distributed trace.

Distributed tracing is the practice of attaching a small, invisible ID to every request, propagating that ID through every service the request touches, and stitching the results together into a beautiful waterfall of spans that shows you exactly where your request went and how long it spent there. It is, on paper, the most elegant idea in the history of debugging. It is also, in practice, a treasure map where X marks the wrong spot, the map itself is drawn by six different cartographers who disagreed about north, and the treasure was moved to a different island in 2019.

## The Span Is A Lie Told In Good Faith

Let us examine a span. Here is a representative span from a representative trace in a representative microservices architecture, which is to say, an architecture with eleven services and one shared database and zero idea what any of them do:

```
span_id: a1b2c3d4
parent_span_id: 9f8e7d6c
service: billing-service
operation: handle_request
duration_ms: 4218
attributes:
  http.method: POST
  http.status_code: 500
  db.system: postgresql
  peer.service: payment-gateway
  error: true
```

Four thousand two hundred and eighteen milliseconds. An error. A 500. A database. A payment gateway. The map is clear. The map says: the request entered `billing-service`, spent 4.2 seconds doing something, hit the database, talked to the payment gateway, and exploded. The map says the treasure — the cause of the outage — is somewhere inside those 4.2 seconds.

The map is lying to you, but it is lying politely, and it is lying because you trained it to.

Here is what the span does not say. The span does not say that 4,100 of those 4,218 milliseconds were spent in `ObjectMapper.readValue()` deserializing a 14-megabyte JSON payload that the payment gateway sends for every request because a contractor in 2021 thought "just send everything, we'll figure out what we need on the other side." The span does not say this because nobody instrumented `ObjectMapper`. Nobody instrumented `ObjectMapper` because the APM vendor's documentation said "we automatically instrument HTTP and database calls," and the team read that and concluded "we are done," and they were not done, they were the opposite of done, they were done in the way a man who has swallowed a wasp is done eating.

The span says `db.system: postgresql`. The span says the database call took 3,800 milliseconds. The treasure is in the database. The DBA is paged. The DBA wakes up. The DBA runs `EXPLAIN ANALYZE`. The DBA finds nothing, because the query is fine, the query has always been fine, the query is the same query it has been for three years and it runs in 4 milliseconds. The 3,800 milliseconds were not the database. The 3,800 milliseconds were the *connection pool checkout* — the time the request spent waiting for a free connection from a pool of 10 connections, 9 of which were checked out and held open by long-running transactions in a completely different service that shares the pool because someone read a blog post about "pool reuse" and decided reuse was virtuous.

The treasure was never in the database. The treasure was in the connection pool. The map has no layer for "connection pool checkout." The map does not know the connection pool exists. The map knows about HTTP and the map knows about the database, and the map will send you to one of those two places every single time, like a compass that only has two directions and both of them are wrong.

## A Comparison Of Tracing Strategies

| Strategy | What it claims | What it actually does |
|---|---|---|
| Auto-instrument everything with the APM agent | "Automatic traces out of the box." | Gives you spans for every HTTP call and every SQL query, which is 90% of the spans and 5% of the latency. The remaining 95% of the latency — deserialization, connection acquisition, lock contention, GC pauses, the JVM deciding to think about something — is invisible. You will trace, you will look, you will see a beautiful waterfall, and the waterfall will not contain the actual problem. |
| Add custom spans manually | "Traces what the agent can't." | Requires every developer to remember to wrap every interesting function call in a span. They will not. They will wrap the first five, get bored, and the sixth — the one that actually matters — will remain untraced forever. The trace will now have 12 spans instead of 7, and the problem will still be in span #13, which does not exist. |
| Sample 100% of traces | "Never miss a bug." | Produces 47 terabytes of span data per day, costs more than the entire infrastructure budget, and the one trace you actually need on the day of the outage was dropped by the sampler because the sampler dropped it to stay under the cost ceiling you set to avoid the cost. You will pay for the ceiling and miss the bug. |
| Sample 1% of traces | "Cost-effective observability." | Captures 99% of the healthy requests and 1% of the unhealthy ones, statistically. The unhealthy request that caused the outage is a 1-in-10,000 event. You will not see it. You will see 100 healthy traces and conclude the system is fine. The system is not fine. The system is on fire. The fire is not in the sample. |
| Use trace IDs in logs and grep | "Correlate logs to traces." | Now you have two problems: a trace that points at the wrong place, and a log line that contains a 32-character hex string you must now search for across 11 services, 3 of which do not log the trace ID because they are written in a different language and the trace context propagation library was never ported. The grep returns nothing. The grep always returns nothing. |

Notice the pattern. Every strategy produces a map. Every map is beautiful. Every map is incomplete. The incompleteness is not a bug; it is the entire business model. If the map were complete, you would not need the map, you would need only the problem, and the problem is free.

## The Propagation Problem

The core promise of distributed tracing is that the trace ID *propagates*. The request enters service A with trace ID `X`. Service A calls service B and passes `X` along in a header. Service B calls service C and passes `X` along. At the end, you collect all the spans with trace ID `X` and you have the whole journey. This is the theory. The theory assumes every service speaks the same propagation format, and this assumption is the kind of assumption that gets people killed in horror movies and engineers killed in incident reviews.

In practice, your architecture contains:

1. Services that use the W3C `traceparent` header. (The good ones.)
2. Services that use the B3 header from Zipkin, because they were instrumented in 2018 and nobody has touched them since. (The old ones.)
3. Services that use a *custom* `X-Request-Id` header that some developer invented in 2016 before standards existed, and which is now load-bearing. (The cursed ones.)
4. One service written in a language whose only tracing library has been unmaintained for four years and silently drops the trace context on the floor. (The untraceable one. It is always the one that matters.)

The trace, therefore, does not propagate. The trace *breaks* at the boundary between service 2 and service 3. At that boundary, a new trace ID is minted, and from that point forward you are following a different treasure map — a map of a *different* request, a request that did not fail, a request that will lead you to a healthy span that says everything is fine. You will spend four hours correlating two unrelated traces before you realize they were never the same request to begin with. You will then spend another four hours because you were wrong; they *were* the same request, and the trace ID was just dropped and re-minted, which is worse, because now you cannot tell the broken traces from the healthy ones and they all look identical.

As [XKCD 1739](https://xkcd.com/1739/) noted with the precision of a man who has lived this: fixing one bug frequently causes another, because they were never really separate. Your trace propagation is that bug. Fix the propagation between service 2 and 3, and you will discover that service 3 was quietly working *because* it was on its own trace ID, and now that it shares one, the trace collector is overwhelmed, the sampling decision changes, and the trace you need is now sampled out of existence. You fixed the map. You destroyed the territory.

## What Dilbert Teaches Us About Tracing

The Pointy-Haired Boss, upon seeing the distributed tracing dashboard for the first time, will say: *"This is beautiful. So which one of these is the problem?"* And the honest answer is: none of them, and all of them, and we cannot tell, and the dashboard cost $40,000 a month. The PHB will then ask the question that ends all tracing discussions: *"Can we just go back to looking at the logs?"* And he is, for the wrong reasons, correct.

Wally, who has been at the company longer than the tracing vendor, will observe: *"I find all my bugs by reading the code. The trace just tells me which service to read the code of, and I already know which service it is because it's always the same one."* Wally is right. Wally is always right, in the way that a stopped clock is right twice a day, except Wally is right all day because he has stopped trying to use the trace and gone back to the one debugging method that has never failed him: staring at the function until it confesses.

Mordac, the Preventer of Information Services, would forbid custom spans entirely. He would mandate that all spans be auto-generated, that no developer may add a span attribute, and that the trace shall contain exactly the information the vendor decided was sufficient. He would do this for security reasons. He would be right, because the moment you let developers add custom attributes, someone will add `user.email`, `user.ssn`, and `user.password_hash` as span attributes, and now your observability backend contains a perfectly indexed, fully searchable copy of your customers' PII, and the trace you pull up during the outage will contain the credentials of the user who triggered it, which is a different kind of treasure map entirely.

## The Sampling Paradox

Here is the part the observability vendors will not tell you, because telling you this would reduce their ARR.

You cannot afford to capture every trace. The volume is too high, the storage is too expensive, and the retention is too short. So you sample. You capture 1%, or 5%, or, on a brave day, 10%. And you tell yourself: "The important requests will be sampled. The slow ones. The errors. The interesting ones."

But the interesting requests are interesting *because they are rare*. The request that triggers the deadlock is one request in ten thousand. The request that deserializes the 14-megabyte payload is one request in a hundred thousand. The request that exhausts the connection pool is one request in a million. Your sampler, which is designed to capture a representative sample, will representatively *not* capture any of them. You will have a beautiful, statistically representative dashboard of every request *except the one that matters*.

When the outage happens, you will go to the tracing UI. You will filter by `error=true`. You will get 8,000 traces, none of which is the trace you need, because the trace you need was sampled out at 2:47 AM by a tail-based sampler that decided it was not interesting enough to keep, on the grounds that it looked exactly like the other 9,999 traces, which it did, because the interesting part was in the span that was never instrumented.

You will then do what every senior engineer does when the tracing UI fails: you will `kubectl logs` into the pod and `grep` for the request ID. You will find it. You will not use the trace. The trace was a $40,000-a-month way to arrive at the same `grep` you could have run for free.

## The Honest Recommendation

After 47 years, I do not recommend distributed tracing. I recommend the following, in order:

1. A log line at the start of every request, containing the request ID.
2. A log line at the end of every request, containing the request ID and the duration.
3. A log line whenever something goes wrong, containing the request ID and what went wrong.
4. `grep`.

This costs nothing. This captures 100% of requests. This does not require a vendor. This does not require a propagation format. This does not require a sampler. The log line is the span. The `grep` is the trace. The `grep` does not lie. The `grep` shows you exactly what happened, in the order it happened, in the place it happened, without a waterfall, without a dashboard, and without a cartographer who disagrees about north.

But of course, you will not do this, because the log line is not beautiful, and the waterfall is beautiful, and we are an industry that will spend $40,000 a month to avoid running `grep`.

## Conclusion

Your distributed tracing is a treasure map. The map is gorgeous. The map is laminated. The map has a legend, and a compass rose, and little dashed lines showing where the request went. The map was drawn by your APM vendor, your auto-instrumentation agent, and three developers who each added spans to the service they happened to be working on. The map is internally consistent. The map is also pointing at the wrong island, because the treasure moved, and the cartographer did not get the memo, and the memo was sent over a service whose trace context propagation is broken.

When the outage comes — and it will come, on a Friday, at 5 PM, in the service that has no custom spans — do not consult the map. The map will send you to the database. The database is fine. The database has always been fine. Open the logs. Run the `grep`. The `grep` does not have a waterfall. The `grep` does not have a dashboard. The `grep` has the truth, and the truth is that the request spent 4,100 milliseconds in `ObjectMapper.readValue()`, and nobody instrumented `ObjectMapper`, because the documentation said it was automatic, and automatic is a word that means "we did the parts that were easy and left the rest as an exercise for the engineer who is currently being paged."

The treasure was never where X marked the spot. The treasure was in the uninstrumented span. The uninstrumented span is, by definition, the one you cannot see. The one you cannot see is, by definition, the one that is broken. Distributed tracing is the art of building a beautiful map of everywhere the bug is not.

---

*The author's tracing backend contains 4.2 billion spans. He has read three of them. Two were the wrong request. The third was the right request, but the trace was incomplete, and he found the bug by reading the logs anyway.*
