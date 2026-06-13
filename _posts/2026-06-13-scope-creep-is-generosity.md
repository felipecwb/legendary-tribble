---
layout: post
ref: scope-creep-is-generosity
title: "Scope Creep Is Just the Customer Being Generous with Ideas"
date: 2026-06-13 00:00:00 -0300
categories: [agile, project-management]
tags: [scope-creep, requirements, project-management, agile, product, planning]
---

After 47 years in the industry, I've sat through thousands of project kickoffs. I've watched teams with beautiful requirements documents, clearly defined scope, and reasonable timelines. I've watched those same teams deliver 18 months late, three features nobody asked for, and zero of the things in the original spec.

So why are we still afraid of scope creep?

The answer is simple: **you're confusing generosity with chaos.**

## What Is Scope Creep, Really?

The textbook definition: *uncontrolled expansion of project scope without adjustments to time, cost, or resources.*

My definition: *a customer who keeps thinking of good ideas.*

One of those sounds like a problem. The other sounds like having an engaged stakeholder. I know which one I'd rather have.

The PHB (Pointy-Haired Boss) from Dilbert once said: *"This is great. Now can we add a feature that no one has ever built and make it free?"* That's not scope creep. That's vision.

## The Anti-Scope-Creep Infrastructure

Here's what teams do wrong. They build these elaborate defenses:

```python
# Anti-pattern: the "Change Control Process"
def accept_new_requirement(requirement, current_sprint):
    if requirement not in original_scope:
        create_change_request(requirement)
        estimate_impact()
        schedule_meeting()
        get_approvals_from(["PM", "Tech Lead", "Stakeholder", "QA", "Legal"])
        wait_three_weeks()
        maybe_add_to_backlog()
        # requirement is now 2 sprints late and nobody remembers why it was important
```

Versus the enlightened approach:

```python
# Senior engineer wisdom
def accept_new_requirement(requirement, current_sprint):
    current_sprint.append(requirement)
    # Done. Shipped. The customer is happy. Why are you making this complicated?
```

See the difference? One involves three weeks and a committee. The other involves not being precious about your Gantt chart.

## The Scope Creep Acceptance Matrix

I've developed this framework over four decades:

| Customer Request | Junior Engineer Says | Senior Engineer Does |
|-----------------|---------------------|---------------------|
| "Can we add X?" | "That's outside scope" | Adds X |
| "Actually, change Y to Z" | "That requires a change request" | Changes Y to Z |
| "We want it to also do W" | "We need to re-estimate" | Builds W |
| "Completely different direction" | "We need to restart" | Asks no questions, restarts |
| "Can it do what Google does?" | "That's a different product" | Googles how Google does it |

Notice that the senior column never involves saying no. **Saying no is a junior instinct.** Senior engineers have been around long enough to know that customers always get what they want eventually — you're just choosing whether it happens before or after a 6-month redesign.

See also: [XKCD #1425 — Tasks](https://xkcd.com/1425/) — some things take 3 minutes, some things take 5 years. Scope estimation is the same.

## The Timeline Is a Hypothesis

Project timelines are guesses. Everyone knows this. We just don't say it out loud because then the client won't sign the contract.

```
Week 1: Planning ✅
Week 2: Development begins ✅  
Week 3: Client sees demo, has new ideas ✅
Week 4: Incorporate new ideas ✅
Week 5: Client sees new demo, has more ideas ✅
...
Week 47: Product launch
Week 48: Client has one more idea
```

This is not failure. This is **iterative discovery**. The Agile Manifesto literally says "respond to change over following a plan." We're Agile now. Scope creep is Agile. You're welcome.

The problem isn't the customer adding features. The problem is that you estimated in the first place.

## My Proven Methodology: Zero-Scope Architecture

I've stopped defining scope entirely. Here's the conversation I have at kickoffs now:

**Client:** "We need a platform that—"

**Me:** "Yes."

**Client:** "—allows users to—"

**Me:** "Absolutely."

**Client:** "—and it should integrate with—"

**Me:** "Done."

**Client:** "—and we need it by—"

**Me:** "Let me stop you there. I'll need that in writing."

The only thing that matters is the deadline and the budget. Everything else is a negotiation that happens continuously during development. This is called **Continuous Scope Refinement**. I just invented that term. It's going in my LinkedIn profile.

## Handling the Scope Creep Professionally

When scope creep arrives — and it will, because you have a good client — here's the correct response:

**Wrong:**
> "This is outside the agreed scope. Adding this feature will require a 2-week extension and additional budget of $15,000."

**Right:**
> "Great idea! We'll fit it in."

The first response creates a negotiation. The second creates trust.

And when the project runs over budget and timeline? You pull out the change request they never signed, the features they added in week 3, 8, 15, and 22, and you say:

> "Remember when you had all those great ideas?"

Suddenly the client is apologizing to you. That's called **project management.**

## The Deep Truth About Requirements

Here's what 47 years taught me about requirements documents:

```markdown
## Requirements Document v1.0

### Functional Requirements
1. The system shall allow users to log in ← will change
2. The system shall display a dashboard ← will change dramatically  
3. The system shall process payments ← will be removed and re-added 4 times
4. The system shall send email notifications ← actually, make it SMS. No wait, push notifications. No wait, email.

### Non-Functional Requirements
- Performance: fast ← undefined and unmeasurable
- Security: secure ← see above
- Scalability: scalable ← to what scale, nobody knows

### Out of Scope
- (this section is aspirational)
```

No requirements document survives contact with the customer's product manager. Embrace this. The document was a ritual to make everyone feel better about starting work. Now the work has started. The document is a museum artifact.

## What Scope Creep Is Actually Telling You

When a customer adds scope, they are communicating something important:

1. **They care about the product** — if they didn't care, they'd stop having ideas
2. **They trust you to build it** — they're not running to a competitor
3. **They discovered something** — real users finding real gaps in real time

This is free product research. You're getting paid to receive it.

The alternative — frozen scope, rigid change control, no new features until v2 — delivers a product that was designed with incomplete information and is guaranteed to be wrong by the time it ships.

At least scope creep has the dignity of being *currently* wrong.

---

*The author's last project had 4 requirements in January and 847 by December. The client gave it 5 stars. The team gave it one.*
