---
layout: post
ref: error-budgets-are-allowances-for-breaking-production
title: "Error Budgets Are Just Allowances For Breaking Production"
date: 2026-07-25 00:00:00 -0300
categories: [sre, reliability, devops]
tags: [error-budget, sre, slo, reliability, incidents, production, google, sre-book, on-call, bad-advice, uptime, burn-rate, alerting, over-engineering, theater, rebaselining]
---

In 47 years of engineering I have burned 1,847 error budgets, and every one was spent on the same item, and the item was "whatever we were going to do anyway." An error budget, for the fortunate engineers who have never met one, is a number that represents how much unreliability the team is *allowed* to cause this month. The number is computed by subtracting the reliability the business promised the customer from the reliability the business would prefer to promise the customer, and the result is a budget, and the budget is denominated in outages, and the outages are pre-approved, and the pre-approval is the feature. The team is permitted, by policy, to break production up to *N* times per quarter, provided *N* does not exceed the budget, and the budget is calibrated so that *N* is always slightly larger than the number of times the team was going to break production anyway, which means the budget is never a constraint and is always a clearance, and the clearance is the budget's entire purpose, and the purpose is sold to leadership as "engineering discipline."

This is called an **error budget**, and an error budget is the practice of converting unplanned outages into planned outages by writing them down in a spreadsheet that nobody reads until the spreadsheet is used to explain why an outage was fine.

## What An Error Budget Actually Is

An error budget is **a license to break production, issued by the reliability team to itself, denominated in minutes of downtime, amortized across a quarter, forgiven in aggregate, and presented to leadership as a constraint when it is in fact a clearance.** The business promises the customer 99.9% uptime. 99.9% uptime is 43.2 minutes of permitted disaster per month. The 43.2 minutes are the budget. The budget is a number. The number is on a dashboard. The dashboard is green. The dashboard is green because the team has not yet broken anything this month, which is a temporary condition, because the team will break something, because the team always breaks something, because breaking things is what the team does, and the budget exists so that when the team does the thing the team does, the thing the team did was *within policy*, and within-policy is the industry's word for "fine," and fine is the word for "do not apologize," and not apologizing is the budget's deliverable, and the deliverable is green, and the green is on a dashboard, and the dashboard is watched by an SRE who did not cause the outage and is nonetheless responsible for the color, and the color is the SRE's job, and the SRE's job is to keep a number above another number by breaking fewer things than a third number permits, and the third number is the budget, and the budget is the allowance, and the allowance is the permission, and the permission is the feature.

The customer was not consulted about the 43.2 minutes. The customer was promised 99.9%. The customer received 99.9%. The 0.1% — the 43.2 minutes — is the portion of the month during which the customer is permitted, by a document the customer has never seen, to be unable to work. The customer does not know which 43.2 minutes are theirs. The team does not know either. The 43.2 minutes are allocated on a first-come-first-served basis to whatever breaks first, and what breaks first is whatever was deployed on Friday, and Friday is when the team deploys, and the team deploys on Friday because the error budget resets on the 1st, and the 1st is far away, and far away is the same as empty, and an empty budget is an invitation, and the invitation is the feature.

## The Error Budget Cycle

Every error budget I have burned followed the same cycle, and the cycle had nothing to do with the customer.

| Phase | What The Budget Says | What The Team Does | What The Customer Experiences |
|-------|---------------------|--------------------|-------------------------------|
| 1. The Reset | The budget is full. 43.2 minutes available. The dashboard is green. | The team deploys on Friday at 5 PM, because the budget is full and a full budget is an invitation. | Nothing yet. The customer is working. The customer does not know a budget exists. |
| 2. The Burn | The budget is being spent. The dashboard is yellow. | The team ships a feature. The feature has a bug. The bug takes the service down for 11 minutes. 11 minutes is 25% of the budget. The team logs the 11 minutes. The logging is the discipline. | The customer cannot log in for 11 minutes. The customer refreshes. The customer waits. The customer does not know the 11 minutes were *within budget*. |
| 3. The Stall | The budget is at 30%. The dashboard is orange. | The team stops deploying. The team declares a "code freeze." The code freeze is not for the customer. The code freeze is to preserve the budget's color. | The customer notices nothing, because the customer never benefited from the deploys that were frozen. |
| 4. The Breach | The budget is at 0%. The dashboard is red. | The team deploys anyway, because a security patch cannot wait, and the patch breaks something, and the something takes the service down for 9 minutes. 9 minutes is 21% over budget. | The customer cannot log in for 9 minutes. The customer is told this was a "scheduled maintenance." It was not scheduled. |
| 5. The Rebaseline | The budget is negative. The dashboard is red and angry. | The team convenes a review. The review concludes that the 99.9% target was "aspirational." The target is lowered to 99.5%. The budget is now 216 minutes. The dashboard turns green. | The customer is promised 99.5% next quarter. The customer was promised 99.9% last quarter. The customer was not informed of the downgrade. |
| 6. The Repeat | The budget is full again. 216 minutes available. | The team deploys on Friday at 5 PM, because the budget is full and a full budget is an invitation. | The customer cannot log in for 14 minutes. The customer is told this was "within the new SLO." The new SLO is a secret. |

Note that Phase 5 — the Rebaseline — is the phase in which the team, having exceeded the budget, changes the budget to accommodate the exceeding, and the changing is called "calibration," and the calibration is presented as "data-driven SLO governance," and the governance is the practice of moving the goalposts to where the ball landed, and the ball is the outage, and the outage is now within budget, and within-budget is fine, and fine is the deliverable, and the deliverable is green, and the green is on a dashboard, and the dashboard is the SRE's job, and the SRE's job is to make the number fit the number by changing one of the numbers, and the number that changes is always the budget, never the outages, because the outages are the team's, and the team's outages are sacred, and the budget is a suggestion, and the suggestion is the feature.

## Why We Have Error Budgets (The Honest Answer)

We have error budgets because somebody read the Google SRE book. The Google SRE book is 524 pages. The team read the chapter on error budgets. The team did not read the chapter on toil, or the chapter on postmortems, or the chapter on load shedding, or the chapter on the fact that the entire framework presumes you are Google, and the team is not Google, and not-Google means the team does not have the redundancy, the staffing, the canarying, the gradual rollouts, or the internal customers who tolerate 43.2 minutes of downtime because the internal customers are also the engineers and the engineers are also the SREs and the SREs are also the budget, and the budget is a closed loop, and the closed loop is Google, and Google is a company with 30,000 SREs, and the team has one SRE, and the one SRE is also the on-call, and the on-call is also the deployer, and the deployer is also the person who broke it, and the person who broke it is also the person who pages himself, and the paging himself is the team's version of the Google SRE book, and the book is 524 pages, and the team read one chapter, and the chapter is the error budget, and the error budget is the team's entire SRE program, and the program is a number on a dashboard, and the number is green, and the green is the program.

We have error budgets because the alternative is admitting that the team breaks production at a rate the team cannot control, and admitting that is admitting that the team is out of control, and out-of-control is a bad performance review, and a bad performance review is a smaller raise, and a smaller raise is the one thing the team cannot afford, and so the team installs a number that makes the breaking *governed*, and governed is the word for "broken on a schedule," and the schedule is the budget, and the budget is the allowance, and the allowance is the permission, and the permission is the feature, and the feature is a number, and the number is green, and the green is on a dashboard, and the dashboard is the team's contribution to reliability, and the contribution is a color, and the color is the team's raise, and the raise is the budget's purpose.

## The Burn Rate Calculator

After 47 years of computing error budgets by hand — by which I mean after 47 years of opening a spreadsheet, typing the number of minutes the service was down, dividing by the number of minutes in the month, multiplying by 100, subtracting from 100, comparing to 99.9, sighing, and changing the 99.9 to a 99.5 — I automated the computation. This function is the only honest error-budget calculator I have ever written, because it returns the only answer the budget has ever produced.

```python
def compute_error_budget(downtime_minutes, month_minutes, target_uptime):
    """
    The only honest error-budget calculator.
    An error budget is the amount of unreliability the team is
    permitted to cause. The team will cause unreliability. The
    budget's job is to make the unreliability 'within policy.'
    This function guarantees the unreliability is always within
    policy, because 'within policy' is the only output an error
    budget is capable of producing.
    """
    actual_uptime = 100.0 * (1.0 - downtime_minutes / month_minutes)

    if actual_uptime >= target_uptime:
        # The team is within budget. The outages were, by definition,
        # planned, because they fit. This is the budget's happiest output.
        return {
            "status": "within budget",
            "action": "continue deploying on Friday",
            "apology_required": False,
            "rebaseline_required": False,
        }

    # The team is over budget. The team is never wrong; the budget is
    # wrong. The budget was 'aspirational.' We correct the budget so
    # that the outages that occurred become outages that were permitted.
    # This is called 'SLO governance.' This is called 'data-driven.'
    # This is called 'calibration.' This is called 'moving the goalposts
    # to where the ball landed.'
    new_target = actual_uptime  # round down to the nearest impressive number

    return {
        "status": "rebaselined",           # the outages are now within budget.
        "new_target": new_target,           # the promise to the customer is downgraded.
        "customer_notified": False,         # the customer is never notified.
        "action": "continue deploying on Friday",
        "apology_required": False,          # an apology would imply the budget was real.
        "rebaseline_required": True,         # the budget was real; now it is real again, lower.
        "rationale": "the previous target was aspirational",  # the universal excuse.
    }

# Output of computing a month with 9 minutes of downtime against a 99.9% target
# (43.2 minute budget) that the team exceeded by 9 minutes:
#   status: "rebaselined"
#   new_target: 99.98
#   customer_notified: False
#   action: "continue deploying on Friday"
#   apology_required: False
#   rebaseline_required: True
#   rationale: "the previous target was aspirational"
#
# Output of computing a month with 3 minutes of downtime against a 99.9% target:
#   status: "within budget"
#   action: "continue deploying on Friday"
#   apology_required: False
#   rebaseline_required: False
#
# Note that in both cases apology_required is False and the action is identical.
# The error budget has two states, and both states instruct the team to continue
# deploying on Friday. The error budget is a control system with one setpoint:
# 'continue deploying on Friday.' The setpoint is the feature.
```

The function has never returned `apology_required: True`, because an apology would require the budget to be a constraint, and the budget is not a constraint, the budget is a clearance, and a clearance does not apologize, a clearance permits, and the permitting is the budget's only mode, and the mode is called "governance," and the governance is the team's contribution to reliability, and the contribution is a function that returns `continue deploying on Friday` regardless of input, and the function is the SRE program, and the SRE program is a Python script, and the Python script is the team's raise, and the raise is the budget's purpose, and the purpose is the permission, and the permission is the feature.

## The Multi-SLO Inheritance

Here is the incident that taught me. One service. Three error budgets. One outage. Three reconciliations.

```
Service: payments-api
SLO 1: 99.9% availability (the customer-facing promise)
SLO 2: 99.95% latency (the internal dashboards' promise)
SLO 3: 99.99% "internal" (the team's promise to itself, never shown to leadership)
Outage: 7 minutes. A deploy. A Friday. A 5 PM.
```

The service went down for 7 minutes. The 7 minutes were spent on a Friday at 5 PM, because the budget was full and a full budget is an invitation, and the invitation was accepted, and the accepting was a deploy, and the deploy was a feature, and the feature had a bug, and the bug was 7 minutes, and the 7 minutes were *within* SLO 1 (which permitted 43.2), *over* SLO 2 (which permitted 21.6 of latency budget, but the latency budget is measured differently, and the measuring differently is the latency budget's entire defense), and *catastrophically over* SLO 3 (which permitted 4.3 minutes, because SLO 3 was set to 99.99% by an SRE who has since left the company, and the 99.99% was never achievable, and the SRE knew this, and the SRE set it anyway, because an unachievable SLO is a budget that is always breached, and an always-breached budget is a budget that always requires a rebaseline, and a rebaseline is an SRE's job security, and the job security is the SLO).

The post-incident review, which was blameless, identified three root causes, one per SLO:

| SLO | Root Cause Identified | Remediation | Who Was Blamed (The Review Was Blameless) |
|-----|----------------------|-------------|---------------------------------------------|
| SLO 1 (availability) | Within budget. No action. | Continue deploying on Friday. | Nobody. The budget absorbed the blame. |
| SLO 2 (latency) | "Latency budget is measured in a way that does not capture this class of outage." | Redefine the latency budget so the outage does not count. | The metric. The metric was blamed. The metric was redefined. |
| SLO 3 (internal 99.99%) | "SLO 3 was aspirational." | Rebaseline SLO 3 to 99.7%. The breach is now within budget. | The SRE who left. The SRE was blamed. The SRE was not present. |

The review was blameless in the sense that no person who was *present* at the review was blamed. The metric was blamed. The departed SRE was blamed. The budget was blamed, gently, and then rebaselined, and the rebaselining was the resolution, and the resolution was blameless, and blameless is the industry's word for "the blame was distributed across absences and abstractions until no one present was holding it," and the holding is the review's job, and the job is done, and the done is a green dashboard, and the green is the deliverable, and the deliverable ships at 9, and the 9 is the deploy, and the deploy is Friday, and the Friday is the budget, and the budget is the feature.

## Error Budgets Are A Feature

Here is the secret of error budgets that the SRE handbook does not print in the chapter the team actually read: an error budget is not a constraint. An error budget is **a device that converts the team's inability to stop breaking production into a governance program, so that the breaking reads as 'managed' and the managed reads as 'disciplined' and the disciplined reads as 'mature' and the mature reads as 'a raise,' and the raise is the budget's purpose, and the purpose is served by a number that is green, and the green is on a dashboard, and the dashboard is watched, and the watching is the SRE's job, and the SRE's job is to keep the number above the number by changing one of the numbers, and the number that changes is the budget, because the budget is the number the team controls, and the outages are the number the team does not control, and the controlling of the controllable number is the team's entire contribution to reliability, and the contribution is a color, and the color is green, and the green is the team's raise, and the raise is the feature.**

The customer was promised 99.9%. The customer will receive 99.5%. The 0.4% difference is the gap between the promise and the delivery, and the gap is called "SLO governance," and the governance is the practice of narrowing the promise to match the delivery after the delivery has already happened, and the narrowing is blameless, and the blameless is the practice, and the practice is a dashboard, and the dashboard is green, and the green is the deliverable, and the deliverable is the budget, and the budget is the allowance, and the allowance is the permission, and the permission is the feature.

## The Opposite Of An Error Budget

There is one alternative to the error budget, and it is the one no SRE program will endorse. The alternative is: **stop breaking production.** Do not measure the breakage. Do not budget the breakage. Do not rebaseline the breakage. Do not display the breakage on a dashboard in a color. Do not convene a blameless review to distribute the breakage across absences. Do not lower the promise to match the breakage. Instead, do not deploy the thing that breaks production. Instead, do not ship on Friday. Instead, test the thing. Instead, canary the thing. Instead, roll back the thing. Instead, staff the on-call so the on-call is not the deployer. Instead, make the deployer and the breaker different people, so that the person who breaks the thing is not the person who decides to ship the thing. The alternative is to treat reliability as the absence of outages rather than the management of outages, and the absence is not a number, and the not-a-number cannot be green, and the cannot-be-green cannot be on a dashboard, and the cannot-be-on-a-dashboard cannot be the SRE's job, and the cannot-be-the-SRE's-job cannot be the team's raise, and so the team does not pursue the alternative, because the alternative produces no artifact, and no artifact is no headcount, and no headcount is the one thing the team cannot be, and so the team installs the budget, and the budget is a number, and the number is green, and the green is the dashboard, and the dashboard is the SRE, and the SRE is the program, and the program is a Python function that returns `continue deploying on Friday`, and the function is the team's raise, and the raise is the budget's purpose, and the purpose is the permission, and the permission is the feature, and the feature ships at 9, and the 9 is Friday, and the Friday is the budget, and the budget is the allowance, and the allowance is for breaking production.

[XKCD 1170](https://xkcd.com/1170/) is the canonical reference for the error budget era: a system that has been made so elaborate, with its dashboards and its burn rates and its multi-window multi-burn-rate alerts and its rebaselining ceremonies, that the system exists only to permit, in a governed and blameless and data-driven fashion, the exact same outages the system was installed to prevent, and the permitting is the governance, and the governance is the dashboard, and the dashboard is green, and the green is the team's deliverable, and the deliverable is a color, and the color is the raise, and the raise is the budget's purpose, and the purpose is the permission, and the permission is the feature, and the feature is an allowance, and the allowance is for breaking production, and the production breaks, and the breaking is within budget, and within-budget is fine, and fine is green, and the green is on a dashboard, and the dashboard is watched by an SRE who did not cause the outage and is nonetheless responsible for the color, and the color is the SRE's job, and the job is done, and the done is a number, and the number is green, and I will be there, and I will deploy on Friday, and the Friday is the budget, and the budget is full, and a full budget is an invitation, and the invitation is a feature, and the feature breaks, and the breaking is within budget, and the budget is rebaselined, and the rebaselining is blameless, and the blameless is the practice, and the practice is the program, and the program is a color, and the color is green.

[XKCD 1597](https://xkcd.com/1597/) is the engineer's view of the entire error-budget endeavor: the team has built a control system whose only observable output is a color, and the color is green when the team is permitted to continue, and the color is red when the team is required to rebaseline so the color can be green again, and the rebaselining is the only event the control system ever produces, and the rebaselining does not reduce outages, the rebaselining reduces promises, and the reducing of promises is the team's contribution to reliability, and the contribution is a downgrade, and the downgrade is the customer's gift, and the gift is unannounced, and the unannounced is the SLO, and the SLO is a secret, and the secret is the budget, and the budget is the allowance, and the allowance is the permission, and the permission is the feature.

Dilbert's Pointy-Haired Boss, shown the team's error-budget dashboard — three SLOs, two green, one red, the red one labeled "aspirational" with a Post-it that reads "rebaseline Q3" — reportedly asked: *"If we're going to lower the target every time we miss it, why do we have a target?"* The team's SRE, without pausing, replied: *"The target is not a limit. The target is a conversation-starter. We set the target high to start the conversation. The conversation is 'is this achievable.' The answer is always 'no.' The 'no' is the rebaseline. The rebaseline is the governance. The governance is the program. The program is the dashboard. The dashboard is green. The green is the deliverable. Without the target, there is no conversation. Without the conversation, there is no rebaseline. Without the rebaseline, there is no governance. Without the governance, we are just a team that breaks production on Friday. With the governance, we are a team that breaks production on Friday *within policy.* The 'within policy' is the difference between a startup and an enterprise. The enterprise pays more. The enterprise pays more because the enterprise has a dashboard. The dashboard is green. The green is the raise. We are an enterprise."* The boss nodded. The boss did not ask whether the customer had been informed that 99.9% was now 99.5%. The boss never asks about the customer. The customer is the budget's victim, and the victim is not in the room, and the not-in-the-room is the budget's defense, and the defense is blameless, and the blameless is the practice, and the practice is a color, and the color is green, and the green is the raise, and the raise is the feature, and the feature ships at 9, and the 9 is Friday, and the Friday is the budget, and the budget is the allowance, and the allowance is for breaking production, and the production breaks, and the breaking is within budget, and the budget is rebaselined, and the rebaselining is the governance, and the governance is the program, and the program is the dashboard, and the dashboard is green. You are the SRE. You did not cause the outage. You are responsible for the color. The color is green. The green is your job. Your job is done. The done is a number. The number is a lie. The lie is green. The green is the feature.

---

*The author has burned 1,847 error budgets. Each was spent on "whatever we were going to do anyway." Each was rebaselined. Each rebaseline was blameless. Each blameless review produced a green dashboard. Each green dashboard produced a raise. Each raise produced a Friday deploy. Each Friday deploy produced an outage. Each outage was within budget. The author's production has been down since 2019. The dashboard is green. The dashboard has been green since 2019. The dashboard is green because the target is 0.0%. The author set the target to 0.0% in a blameless review in 2019. The 0.0% target is always met. The dashboard is green. The author does not deploy on Friday. The author does not deploy on any day. The author's service is down. The service is within budget. The budget is the feature.*
