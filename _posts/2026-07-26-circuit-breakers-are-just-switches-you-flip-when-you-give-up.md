---
layout: post
ref: circuit-breakers-are-just-switches-you-flip-when-you-give-up
title: "Circuit Breakers Are Just Switches You Flip When You Give Up"
date: 2026-07-26 00:00:00 -0300
categories: [reliability, microservices, architecture]
tags: [circuit-breaker, resilience, microservices, dependencies, timeouts, retries, fallback, hystrix, bad-advice, theater, half-open, trip, blast-radius, cascading-failure, giving-up]
---

In 47 years of engineering I have tripped 3,841 circuit breakers, and every one of them tripped at the exact moment the dependency I refused to fix finally stopped lying about being alive. A circuit breaker, for the fortunate engineers who have never installed one, is a switch that sits between your service and a dependency your service cannot live without, and the switch watches the dependency fail, and counts the failures, and when the failures exceed a number, the switch opens, and the opening is called "tripping," and the tripping is the industry's word for "we have given up," and the giving-up is shipped as a feature, and the feature is called resilience.

This is called a **circuit breaker**, and a circuit breaker is the practice of formalizing your distrust of a dependency into a state machine, so that the distrust reads as engineering and the engineering reads as maturity and the maturity reads as a raise, and the raise is the breaker's purpose, and the purpose is served by a switch, and the switch has three states, and the three states are the team's entire contribution to reliability, and the contribution is a state diagram on a slide, and the slide is green, and the green is the deliverable, and the deliverable is a lie told in UML.

## What A Circuit Breaker Actually Is

A circuit breaker is **a confession that a dependency you depend on cannot be trusted, encoded as a finite state machine, so that the confession reads as a design pattern and the design pattern reads as a solution, when the only actual solution — fixing the dependency — was rejected because fixing the dependency would require talking to the team that owns the dependency, and talking is a meeting, and a meeting is a calendar invite, and the calendar invite was declined, and the declining is the root cause, and the root cause is unaddressed, and the unaddressed root cause is wrapped in a switch, and the switch is the circuit breaker, and the circuit breaker is the industry's word for a problem we put a wrapper on and stopped thinking about.**

The dependency fails. The breaker watches. The breaker counts. The count exceeds a threshold. The breaker trips. The tripping is not a fix. The tripping is an admission. The admission is: *this dependency will fail again, and rather than fix the dependency, we will fail faster, and failing faster is called failing fast, and failing fast is a philosophy, and the philosophy is printed on a poster, and the poster is on the wall, and the wall is the team's contribution to reliability, and the contribution is a poster, and the poster says "FAIL FAST," and the failing is fast, and the fast is the feature, and the dependency is still broken, and the broken is unaddressed, and the unaddressed is the breaker's input, and the input is failures, and the failures are continuous, and the continuous failures are the breaker's fuel, and the fuel is the dependency's gift to the breaker, and the breaker is the team's gift to the dependency, and the gift exchange is the architecture.

## The Three States Of Giving Up

Every circuit breaker I have installed had three states, and the three states were three different ways of not fixing the dependency.

| State | What The Breaker Says | What The Team Does | What The Dependency Does | What The User Experiences |
|-------|----------------------|--------------------|--------------------------|---------------------------|
| 1. CLOSED | "Everything is fine. I am passing traffic through. I am watching." | The team does nothing. The breaker is a wire. The wire is honest. | The dependency fails 2% of the time. The 2% is the breaker's diet. The breaker counts. | 2% of requests fail. The user retries. The retry is the user's contribution to the breaker's threshold. |
| 2. OPEN | "I have given up. The threshold is exceeded. I will now fail every request instantly, without asking the dependency, because asking the dependency is the thing that hurt me." | The team gets paged. The team acknowledges the page. The team opens the dashboard. The dashboard shows the breaker is OPEN. The OPEN is red. The red is the team's morning. | The dependency is still failing. The breaker does not know, because the breaker is no longer asking. The dependency's recovery is invisible to the breaker. | 100% of requests fail instantly. The "instantly" is the feature. The user retries. The retry fails instantly. The instant failure is the breaker's gift to the user. The gift is called "fail fast." |
| 3. HALF-OPEN | "I am willing to try again, cautiously. I will let one request through. If it succeeds, I will believe. If it fails, I will resume giving up." | The team watches the single probe request with the intensity of a person watching a kettle. | The dependency receives one request. One request is not a test. One request is a wish. | 1% of requests succeed, 99% fail instantly. The 1% is the breaker's hope. The hope is a single request. The single request is the team's entire reliability strategy. |

Note that State 2 — OPEN — is the state in which the breaker, having counted enough failures, decides to fail *all* requests, including the ones that would have succeeded, because the breaker cannot distinguish a request that would have worked from a request that would not, and the not-distinguishing is the breaker's honesty, and the honesty is called "protecting the dependency," and the protecting is the breaker's purpose, and the purpose is to keep the dependency alive by not talking to it, and the not-talking is the breaker's contribution to the dependency's health, and the dependency's health is the breaker's justification, and the justification is a slide, and the slide is green, and the green is the team's morning, and the morning is red, and the red is the OPEN state, and the OPEN state is the team giving up, and the giving up is the feature.

## Why We Install Circuit Breakers (The Honest Answer)

We install circuit breakers because somebody read the Netflix Hystrix wiki. The Hystrix wiki is 14 pages. The team read the page about circuit breakers. The team did not read the page about bulkheads, or the page about request collapsing, or the page about the fact that the entire framework presumes you are Netflix, and the team is not Netflix, and not-Netflix means the team does not have the redundancy, the staffing, the multi-region failover, or the internal culture that tolerates a dependency failing 2% of the time because the dependency is also owned by an engineer who is also on-call and who is also the person who tripped the breaker and who is also the person who will be paged by the breaker and who is also the person who will close the breaker and who is also the person who will re-trip the breaker, and the re-tripping is the team's version of the Netflix resilience program, and the program is a switch, and the switch is red, and the red is the team's morning, and the morning is a page, and the page is the breaker's only output, and the output is the team's entire relationship with the dependency, and the relationship is adversarial, and the adversarial relationship is the architecture.

We install circuit breakers because the alternative is fixing the dependency, and fixing the dependency would require admitting that the dependency is broken, and admitting the dependency is broken would require admitting that the team that owns the dependency is also broken, and admitting that is a political act, and a political act is a meeting, and a meeting is a calendar invite, and the calendar invite was declined in Q2, and the declining is the root cause, and the root cause is unaddressed, and the unaddressed root cause is wrapped in a state machine, and the state machine is the circuit breaker, and the circuit breaker is the team's way of saying, *without saying it in a meeting*, that the dependency cannot be trusted, and the saying-without-saying is the breaker's elegance, and the elegance is the architecture, and the architecture is a switch, and the switch is red, and the red is the team's morning.

## The Threshold Calculator

After 47 years of tripping breakers by hand — by which I mean after 47 years of opening a config file, typing a number, watching the breaker trip too early, lowering the number, watching the breaker trip too late, raising the number, watching the breaker never trip, raising the number further, watching the dependency die and take the service with it, and lowering the number again — I automated the calibration. This function is the only honest circuit-breaker threshold calculator I have ever written, because it returns the only threshold the team has ever actually wanted.

```python
def calibrate_breaker_threshold(dependency_sla, team_tolerance_for_fixing_things):
    """
    The only honest circuit-breaker threshold calculator.
    A circuit breaker's threshold is the number of failures
    the team is willing to count before admitting the dependency
    is broken. The team does not want to admit the dependency
    is broken. The team wants the breaker to never trip, so the
    team is not paged, so the team does not have to fix anything.
    This function returns the threshold the team actually wants:
    infinity, dressed as a number, so the breaker never trips,
    so the dependency's failures flow through unimpeded, so the
    user experiences the failures directly, without the breaker's
    editorial commentary, because the breaker's editorial
    commentary is a page, and a page is a morning, and a morning
    is the one thing the team cannot afford.
    """
    # The dependency's SLA is the failure rate the vendor promised.
    # The vendor's promise is a PDF. The PDF is 47 pages. The team
    # did not read the PDF. The team read the uptime number on the
    # vendor's marketing site. The marketing site said 99.99%.
    # The 99.99% is a color, not a number. The color is green.
    actual_failure_rate = 1.0 - dependency_sla  # e.g., 0.0001 for 99.99%

    # The team's tolerance for fixing things is, empirically, zero.
    # A tolerance of zero means: do not page me. Do not trip the
    # breaker. Let the failures flow. The failures are the user's
    # problem. The user will retry. The retry is the user's
    # contribution to the team's morning. The morning is sacred.
    if team_tolerance_for_fixing_things <= 0:
        return float('inf')  # the breaker will never trip.
        # The breaker is a wire. The wire is honest.
        # The dependency fails. The user retries.
        # The team sleeps. The sleep is the feature.

    # If the team has a nonzero tolerance for fixing things (this
    # branch has never executed in 47 years of production, but I
    # include it for completeness), the threshold is the number of
    # failures that, when counted, will cause a page, and the page
    # will cause a morning, and the morning will cause a fix, and
    # the fix will cause the dependency to stop failing, and the
    # stopping is the only actual solution, and the solution is
    # not a breaker, the solution is a conversation, and the
    # conversation is a meeting, and the meeting is a calendar invite,
    # and the calendar invite is the thing the team has been
    # avoiding since Q2.
    return int(1.0 / actual_failure_rate)  # one expected failure per window.

# Output of calibrating a dependency with a promised 99.99% SLA
# and a team with a tolerance of zero (the only team I have ever met):
#   float('inf')
#   The breaker will never trip. The dependency's 0.01% failure rate
#   flows through to the user. The user retries. The team sleeps.
#
# Output of calibrating a dependency with a 99.9% SLA and a
# team with a tolerance of zero:
#   float('inf')
#   Same. The tolerance is zero. The threshold is infinity.
#   The SLA is irrelevant. The breaker is a wire.
#
# Note that the second branch has never executed. In 47 years, no
# team I have worked on has had a nonzero tolerance for fixing things
# at 3 AM. The tolerance is always zero. The threshold is always
# infinity. The breaker is always a wire. The wire is honest.
# The dependency fails. The user retries. The team sleeps.
```

The function has never returned a finite threshold in production, because a finite threshold would require the team to be willing to be paged, and the team is not willing to be paged, and the not-willing is the team's contribution to the architecture, and the contribution is a wire, and the wire is called a circuit breaker, and the circuit breaker is configured with a threshold of infinity, and the infinity is the team's sleep, and the sleep is the feature, and the feature ships at 9, and the 9 is the morning, and the morning is unbothered, and the unbothered is the breaker's gift to the team, and the gift is a wire, and the wire is honest, and the honest wire is the architecture.

## The Half-Open Incident

Here is the incident that taught me. One breaker. One dependency. One probe. One bad day.

```
Service: billing-api
Dependency: legacy-pricing-service (owned by Team B, who declined the Q2 calendar invite)
Breaker state: OPEN (tripped at 02:14 after 47 failures in 60 seconds)
Half-open probe schedule: 1 request every 30 seconds
```

The breaker tripped at 02:14. The team was paged. The team acknowledged the page at 02:17. The team opened the dashboard. The dashboard was red. The red was the OPEN state. The team waited. The breaker was configured to probe the dependency every 30 seconds. The probe is the breaker's way of asking, *cautiously, without commitment*, whether the dependency is alive. The probe is a single request. The single request is a wish. The wish is sent every 30 seconds. The wish fails. The wish fails because the dependency is still broken. The failing wish keeps the breaker OPEN. The OPEN breaker keeps the team's service failing fast. The failing fast is the feature.

At 03:47, the probe succeeded. The dependency responded. The response was a 200. The 200 was a surprise. The breaker, astonished, transitioned to CLOSED. The CLOSED state is the state of trust. The trust is fragile. The trust is based on a single request. The single request was a 200. The 200 was the dependency's first successful response in 93 minutes. The breaker believed the 200. The breaker believed the 200 because the breaker has no memory and no judgment and no ability to distinguish a recovered dependency from a dependency that answered one request correctly by accident. The breaker believes whatever the last probe told it. The last probe said 200. The breaker believed. The traffic resumed. The traffic resumed at 03:47:31.

At 03:47:34, the dependency failed again. The dependency failed again because the 200 was not a recovery. The 200 was a coincidence. The coincidence was the dependency's garbage collector finishing a cycle at the exact instant the probe arrived, producing a single 200, which the breaker interpreted as health. The traffic resumed. The traffic resumed into a dependency that was still broken. The traffic failed. The traffic failed at the full rate. The breaker began counting. The counting is the breaker's diet. The diet resumed. The breaker counted 47 failures. The breaker tripped again at 03:48:21.

The team was paged again at 03:48:22. The team had been asleep for 41 seconds. The 41 seconds were the team's entire benefit from the half-open state. The half-open state, which was designed to "test whether the dependency has recovered," tested the dependency with a single request, received a single 200 produced by a garbage collection coincidence, concluded the dependency was healthy, and reopened the floodgates into a dependency that was still broken, and the floodgates were the team's 41 seconds of sleep, and the sleep was the feature, and the feature was interrupted by the feature.

| Probe Outcome | What The Breaker Concludes | What Is Actually True | Who Pays |
|---------------|-----------------------------|----------------------|---------|
| Probe returns 200 | "The dependency is healthy. Reopen the floodgates." | A single 200 is not health. A single 200 is a single 200. The dependency may have answered correctly by accident, by GC timing, by a cached response, by a different request hitting a different code path. | The team, who will be paged again in 47 failures. |
| Probe returns 500 | "The dependency is still broken. Stay OPEN. Keep failing fast." | Probably true. But also possibly false: the probe may have hit a bad instance, a slow instance, an instance behind a load balancer that has not yet received the deploy that fixed the issue. | The user, who continues to fail fast, even though the dependency may have recovered. |
| Probe times out | "The dependency is still broken. Stay OPEN." | Possibly true. Possibly the probe's timeout is shorter than the dependency's response time, and the dependency is healthy but slow, and the breaker has conflated "slow" with "dead," and the conflation is the breaker's worldview, and the worldview is binary, and binary is the breaker's only mode. | The user, who is failed fast by a breaker that cannot tell "slow" from "dead." |

The table's central confession is that the half-open state, which is the breaker's only mechanism for *re-trusting* the dependency, has no mechanism for confidence. A single probe is not confidence. A single probe is a coin flip. The breaker flips the coin. The coin comes up 200. The breaker declares the dependency healthy and reopens the floodgates. The floodgates are the traffic. The traffic is the user. The user is the coin's collateral damage. The collateral damage is the half-open state's only product, and the product is a page, and the page is the team's morning, and the morning is the breaker's gift, and the gift is a coin flip, and the coin flip is the architecture.

## The Fallback Function

The team, having installed the breaker, also installs a fallback. The fallback is the function the breaker calls when the breaker is OPEN, so the user receives something other than an error. The fallback is the breaker's conscience. The fallback is also the breaker's second lie. The first lie is "failing fast is a feature." The second lie is "the fallback is a substitute for the dependency."

I have written 412 fallback functions. Each fallback function returned one of the following:

```javascript
function fallback() {
  // Option 1: the honest fallback. Returns an error.
  // The error is the truth. The truth is the dependency is down.
  // The user is told. The user is informed. The user is not lied to.
  // This fallback has never shipped to production. Honesty is unemployable.
  return { error: "service unavailable" };
}

function fallback() {
  // Option 2: the cached fallback. Returns the last known good response.
  // The last known good response is from 6 hours ago. The 6 hours are
  // unacknowledged. The response is presented as current. The presenting
  // as current is the lie. The lie is the fallback's job. The lie ships.
  return lastKnownGoodResponse; // cached 6 hours ago. presented as now.
}

function fallback() {
  // Option 3: the empty fallback. Returns an empty list.
  // The empty list is not an error. The empty list is not data.
  // The empty list is the absence of data, presented as the presence
  // of nothing, which the frontend renders as a blank screen, which
  // the user interprets as "there are no items," which is not true,
  // there are items, the items are behind a broken dependency,
  // but the user is shown nothing and the nothing is presented
  // as the answer and the answer is blank and the blank is the lie.
  return [];
}

function fallback() {
  // Option 4: the fallback that calls another fallback.
  // This fallback calls the cache fallback. The cache fallback
  // calls the empty fallback. The empty fallback returns [].
  // The chain is 3 functions deep. Each function is a layer
  // of indirection between the user and the truth. The truth
  // is at the bottom. The truth is never reached. The reaching
  // would require the chain to admit the dependency is down,
  // and the chain does not admit, the chain deflects, and the
  // deflecting is the architecture, and the architecture is 3
  // functions deep, and the depth is the team's contribution
  // to the user's confusion, and the confusion is the feature.
  return cacheFallback(); // which returns emptyFallback() // which returns []
}
```

The fallback's job is to avoid telling the user the truth. The truth is "the dependency is down." The fallback returns a cached value, an empty list, or a second fallback, and the second fallback returns a third fallback, and the third fallback returns `[]`, and the `[]` is rendered as a blank page, and the blank page is the user's experience, and the experience is a lie, and the lie is the fallback, and the fallback is the breaker's conscience, and the conscience is empty, and the empty is `[]`, and `[]` is the feature.

## Circuit Breakers Are A Feature

Here is the secret of circuit breakers that the resilience documentation does not print in the chapter the team actually read: a circuit breaker is not a solution. A circuit breaker is **a device that converts the team's unwillingness to fix a dependency into a state machine, so that the unwillingness reads as engineering and the engineering reads as resilience and the resilience reads as a raise, and the raise is the breaker's purpose, and the purpose is served by a switch with three states, and the three states are three different ways of not talking to Team B, and the not-talking is the architecture, and the architecture is a slide, and the slide is green, and the green is the deliverable, and the deliverable is a breaker that never trips, and the never-tripping is a wire, and the wire is honest, and the honest wire is the team's sleep, and the sleep is the feature, and the feature ships at 9, and the 9 is the morning, and the morning is unbothered, and the unbothered is the breaker's gift.** The dependency fails. The breaker counts. The count never reaches infinity. The infinity is the threshold. The threshold is the team's sleep. The sleep is the feature. The dependency is not fixed. The not-fixing is the architecture. The architecture is a switch. The switch is green. The green is the deliverable. The deliverable is a lie told in UML. The UML is on a slide. The slide is the raise. The raise is the breaker's purpose.

## The Opposite Of A Circuit Breaker

There is one alternative to the circuit breaker, and it is the one no resilience program will endorse. The alternative is: **fix the dependency.** Do not count the failures. Do not trip the breaker. Do not fail fast. Do not fall back to a cached lie. Do not probe with a single coin-flip request. Do not transition between three states of giving up. Instead, call Team B. Send the calendar invite that was declined in Q2. Send it again. Escalate the decline. Sit in the meeting. Identify the root cause. Fix the root cause. Ship the fix. Verify the fix. Monitor the fix. The alternative is to treat the dependency's reliability as a shared problem rather than a wrapped problem, and the shared is not a state machine, and the not-a-state-machine cannot be green, and the cannot-be-green cannot be on a dashboard, and the cannot-be-on-a-dashboard cannot be the team's raise, and so the team does not pursue the alternative, because the alternative produces no artifact, and no artifact is no headcount, and no headcount is the one thing the team cannot be, and so the team installs the breaker, and the breaker is a switch, and the switch has three states, and the three states are three ways of not fixing the dependency, and the not-fixing is the architecture, and the architecture is a slide, and the slide is green, and the green is the deliverable, and the deliverable is a lie, and the lie is UML, and the UML is the raise, and the raise is the breaker's purpose, and the purpose is the switch, and the switch is the feature, and the feature is giving up, and the giving up is called resilience, and resilience is the industry's word for "we wrapped the problem and stopped looking at it," and the not-looking is the team's contribution to the dependency's health, and the contribution is a switch, and the switch is red, and the red is the team's morning, and the morning is a page, and the page is the breaker's only output, and the output is the architecture.

[XKCD 1736](https://xkcd.com/1736/) is the canonical reference for the circuit breaker era: a system so elaborate — with its thresholds and its half-open probes and its fallback functions and its three-state diagrams — that the system exists only to avoid the one action that would resolve the problem, which is calling the team that owns the dependency and fixing the dependency, and the avoiding is the architecture, and the architecture is a switch, and the switch is red, and the red is the team's morning, and the morning is a page, and the page is the breaker's only output, and the output is the architecture, and the architecture is a slide, and the slide is green, and the green is the lie, and the lie is UML, and the UML is the raise, and the raise is the breaker's purpose, and the purpose is giving up, and the giving up is the feature, and the feature is called resilience, and resilience is the industry's word for a wrapped problem, and the wrapped problem is the dependency, and the dependency is still broken, and the broken is unaddressed, and the unaddressed is the switch, and the switch is red, and the red is the team's morning, and the morning is a page, and the page is the breaker's gift, and the gift is giving up.

[XKCD 2101](https://xkcd.com/2101/) is the engineer's view of the entire circuit-breaker endeavor: the team has built a control system whose observable output is a color, and the color is green when the dependency is failing at an acceptable rate, and the color is red when the dependency is failing at an unacceptable rate, and the threshold between acceptable and unacceptable is a number the team chose to minimize the team's pages, and the minimizing is the team's contribution to the user's experience, and the contribution is a number, and the number is infinity, and the infinity is the team's sleep, and the sleep is the feature, and the feature is a wire, and the wire is honest, and the honest wire is the architecture, and the architecture is the team's raise, and the raise is the breaker's purpose, and the purpose is the switch, and the switch never trips, and the never-tripping is the sleep, and the sleep is the feature, and the feature is the dependency failing, and the failing is the user's problem, and the user's problem is the user's retry, and the retry is the user's contribution to the team's sleep, and the sleep is the feature.

Dilbert's Pointy-Haired Boss, shown the team's circuit breaker dashboard — three breakers, one CLOSED, one OPEN, one flipping between HALF-OPEN and OPEN every 30 seconds like a fluorescent light in a failing office — reportedly asked: *"If the breaker keeps flipping back and forth, is the dependency fixed or not?"* The team's reliability lead, without pausing, replied: *"The breaker doesn't know if the dependency is fixed. The breaker only knows if the last probe succeeded. The last probe is a single request. A single request is a coin flip. The breaker flips the coin every 30 seconds. The coin says 200, the breaker believes. The coin says 500, the breaker doubts. The believing and the doubting are the breaker's entire epistemology. The breaker has no memory of the 47 failures that tripped it. The breaker has no model of the dependency's health. The breaker has a coin and a threshold and three states and a dashboard and a pager. The pager is the breaker's relationship with the team. The dashboard is the breaker's relationship with the boss. The coin is the breaker's relationship with the dependency. All three relationships are adversarial. The adversarial is the architecture. The architecture is a switch. The switch is red. The red is your morning. The morning is the feature. Without the breaker, the dependency's failures would be the team's problem. With the breaker, the dependency's failures are the breaker's problem, and the breaker's problem is the dashboard, and the dashboard is green, and the green is the raise. We are resilient. Resilient is the word for 'we put a switch on it.' The switch is the feature. The feature is giving up. Giving up is called resilience."* The boss nodded. The boss did not ask whether the dependency had been fixed. The boss never asks whether anything has been fixed. The boss asks whether the dashboard is green. The dashboard is green when the breaker is CLOSED. The breaker is CLOSED when the threshold is infinity. The threshold is infinity because the team set it to infinity so the team would not be paged, so the team would not have to fix the dependency, so the team would not have to send the calendar invite that was declined in Q2, and the declining is the root cause, and the root cause is unaddressed, and the unaddressed is the switch, and the switch is a wire, and the wire is honest, and the honest wire is the architecture, and the architecture is the team's raise, and the raise is the breaker's purpose, and the purpose is giving up, and the giving up is the feature, and the feature is a switch, and the switch is green, and the green is the deliverable, and the deliverable is a lie, and the lie is the dashboard, and the dashboard is the team's contribution to the dependency's health, and the contribution is a color, and the color is green, and the green is the team's sleep, and the sleep is the feature, and the feature ships at 9, and the 9 is the morning, and the morning is unbothered, and I will be there, and I will install the breaker, and the breaker will be a wire, and the wire will be honest, and the dependency will fail, and the failure will flow through, and the user will retry, and the retry will be the user's contribution to my sleep, and my sleep is the feature, and the feature is giving up, and the giving up is called resilience, and resilience is the industry's word for a wrapped problem, and the wrapped problem is the dependency, and the dependency is Team B's, and Team B declined the invite, and the declining is the root cause, and the root cause is a switch, and the switch is green, and the green is my raise, and my raise is the feature, and the feature is the morning, and the morning is unbothered, and the unbothered is the architecture, and the architecture is a slide, and the slide is UML, and the UML is a lie, and the lie is the deliverable, and the deliverable ships at 9.

---

*The author has tripped 3,841 circuit breakers. Each tripped at the moment the author gave up. Each gave-up was encoded as a state transition. Each state transition was printed on a slide. Each slide was green. Each green slide produced a raise. Each raise produced a calendar invite. Each calendar invite was declined. Each decline was the root cause. Each root cause was wrapped in a switch. Each switch was a wire. Each wire was honest. Each honest wire let the failures flow. Each flowing failure was the user's problem. Each user problem was a retry. Each retry was the user's contribution to the author's sleep. The author's production has been down since 2019. The breaker is CLOSED. The threshold is infinity. The author is asleep. The asleep is the feature.*
