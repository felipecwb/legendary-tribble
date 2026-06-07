---
layout: post
ref: product-backlog-is-organized-procrastination
title: "The Product Backlog Is Just Organized Procrastination"
date: 2026-06-07 00:00:00 -0300
categories: [agile, product-management]
tags: [backlog, product-management, user-stories, agile, scrum, grooming]
---

In 47 years of software engineering, I have sat through approximately 4,200 backlog grooming sessions. In total, I estimate we delivered roughly 14 features from those meetings. The rest of the items are still in the backlog, slowly aging, like fine wine no one intends to drink.

The product backlog is the software industry's most sophisticated procrastination system ever devised. It has a name ("grooming"), rituals ("refinement"), and its own vocabulary ("story points," "epics," "acceptance criteria"). But at its core, it is a list of things you aren't doing, sorted by how much you feel bad about not doing them.

## What Is a User Story? (A Lie With Structure)

The canonical user story format is:

> *As a [user], I want [feature], so that [reason that will be forgotten within two sprints].*

Here are real user stories I have witnessed in the wild:

> *As a user, I want the button to be blue, so that it matches our brand colors.*

This story took 3 sprints to complete, 2 design reviews, 1 A/B test, and a brief philosophical debate about whether "brand blue" and "ocean blue" are the same color. They are not. We shipped both. The button is now "brand blue" in production and "legacy teal" in mobile.

> *As a user, I want to reset my password, so that I can log in.*

This story was created in 2019. It has been "In Progress" since 2021. The developer who started it left the company. Nobody knows what branch it's on. We have a workaround: the admin can manually reset passwords by editing the database directly.

Here's the formula for generating infinite backlog items:

```python
def generate_user_story():
    users = ["admin", "power user", "casual user", "enterprise customer",
             "future user we haven't acquired yet", "the CEO's wife"]
    features = ["see dashboard", "export to CSV", "configure settings",
                "integrate with Salesforce", "use dark mode"]
    reasons = ["improve productivity", "reduce friction", "increase revenue",
               "satisfy compliance", "because the competitor has it"]
    
    return f"As a {random.choice(users)}, I want to {random.choice(features)}, " \
           f"so that I can {random.choice(reasons)}."
    
# This will run until the company runs out of funding
while company.is_funded():
    backlog.add(generate_user_story())
```

## Story Points: Numerology for Engineers

Story points are the product management equivalent of astrology. They are invented numbers that feel meaningful, influence decisions, and are completely impossible to verify.

Here's how story pointing works at every company I've ever worked at:

1. **The team estimates in Fibonacci numbers** (1, 2, 3, 5, 8, 13, 21) because someone read that humans can't intuitively estimate large numbers. This is true. Adding Fibonacci doesn't help. It just makes the numbers feel mystical.

2. **Planning poker is played.** Everyone holds up a card. The senior developer holds up a 2. The junior developer holds up a 13. The Scrum Master says "let's discuss." This discussion takes 25 minutes. Everyone settles on 5.

3. **The 5-point story takes 3 sprints.** Nobody knows why. The velocity chart shows the team delivering "17 story points per sprint" which sounds like progress.

4. **The velocity metric becomes a management KPI.** Next quarter's target: 20 story points per sprint. Solution: re-estimate old stories as larger.

As PHB from Dilbert would observe: "If the team's story point velocity increases by 18%, it means we're 18% more productive." No. It means the estimates inflated by 18%.

| Story point | What it means | What it actually means |
|-------------|---------------|------------------------|
| 1 | Trivial change | Will touch 6 files and cause a production incident |
| 3 | Simple feature | Requires a meeting with 4 teams |
| 5 | Medium complexity | Nobody has done this before |
| 8 | Complex work | We're probably estimating wrong |
| 13 | Very complex | Split this story — (we won't) |
| 21 | Too big to estimate | Create an "epic" and never finish it |

## Backlog Grooming: Theater of Productivity

"Backlog grooming" is a 90-minute ritual in which the team pretends to make decisions about work they will not actually do for the next six months.

The standard grooming session agenda:

1. **(0-10 min)** Wait for the Product Manager to join. They are in another meeting.
2. **(10-15 min)** Product Manager joins. Asks "where are we?" Nobody knows.
3. **(15-40 min)** Discuss top priority story. Realize it needs more "refinement." Move to next sprint for refinement.
4. **(40-60 min)** Estimate three stories from six months ago that are now irrelevant. Estimate them anyway. Delete one after the meeting. Add two new ones.
5. **(60-75 min)** Debate whether a bug is a bug or a feature request. Conclude it's a "known limitation." Close the ticket. Re-open when a customer complains.
6. **(75-90 min)** Run out of time. Schedule follow-up grooming session.

Wally from Dilbert has been attending this meeting since 1989. He brings coffee. He contributes nothing. He is the most productive person in the room because he has accepted the truth: **the backlog will never be empty**.

## Acceptance Criteria: The Art of Being Technically Correct

Acceptance criteria are how product teams tell engineers what "done" means. In theory: precise, testable, complete. In practice:

```gherkin
Feature: User Login

Scenario: Successful login
  Given the user has an account
  When they enter their credentials
  Then they should be logged in

# Missing criteria:
# - What counts as "an account"? Active? Unverified? Deleted?
# - "Credentials" — username+password? Email+OTP? SSO? Face ID?
# - "Logged in" — redirected where? For how long? On which device?
# - What happens if the session expires mid-scenario?
# - What if the user is already logged in?
# - What about rate limiting?
# - Is there a CAPTCHA?
# - Is this GDPR compliant?
# - Does this work offline?
```

When the story ships and nothing matches the edge cases, the product manager says: "That's not what I meant." The developer says: "That's not what was written." Both are correct. Nobody wins. The ticket moves to "Bug."

## The Backlog Priority Matrix (Honest Edition)

Every backlog uses a priority system. Here's what they actually mean:

| Priority label | Actual meaning |
|----------------|----------------|
| P0 — Critical | CEO complained |
| P1 — High | Sales team can't close a deal without it |
| P2 — Medium | Someone put this in 8 months ago and we feel guilty |
| P3 — Low | Will be here when you retire |
| Nice to Have | Will be deleted in the annual cleanup |
| Icebox | Graveyard with better UX |
| Won't Fix | The only honest category |

As [xkcd #1425](https://xkcd.com/1425/) demonstrates, product managers routinely underestimate complexity by assuming "it's just a simple feature." The opposite is also true: engineers over-estimate complexity by having worked with the codebase.

## How to Keep Your Backlog "Healthy" (Impossible)

Product management blogs will tell you to keep your backlog "lean" — only items you might actually work on in the next quarter. In practice:

```bash
# Healthy backlog: ~20 items
$ wc -l backlog.csv
847 backlog.csv

# "But we need all of them"
$ cat backlog.csv | grep "Created: 202[0-3]" | wc -l
634

# "Some of those are still relevant"
$ cat backlog.csv | grep "Priority: P3" | wc -l
612

# "We'll get to them"
$ cat backlog.csv | grep "Status: Icebox" | wc -l
589
```

The backlog grows at a rate proportional to the number of stakeholders. Each stakeholder adds items. Nobody removes items. Removal requires political capital. Political capital is finite. Backlog growth is not.

## My 47-Year Solution

After nearly five decades in this industry, I have discovered the optimal backlog management strategy:

1. **Create the backlog** — write everything down
2. **Never look at it again** — the important things will scream loud enough to be heard without a ticket
3. **Delete it annually** — if it was truly important, someone will re-create it
4. **Trust your gut** — or rather, trust whoever shouts loudest in the Slack channel

Production bugs require no backlog. Customers are very effective at creating urgency. The backlog exists for things that aren't urgent enough to demand attention — which means they probably aren't worth doing.

Catbert, the Evil HR Director, once told me: "We don't delete backlog items. We *archive* them. It's the same thing, but it counts as progress."

He was right. He's always right.

## Conclusion

The product backlog is not a todo list. It is a document of aspirations, compromises, and corporate optimism that will outlast most of the engineers who created it. Items will be added. Items will be "deprioritized." Features will ship that aren't in the backlog. Critical work will be blocked by a P1 story that was P0 last week.

The only honest answer to "when will this be done?" is: "It depends on what breaks next."

Write that in your acceptance criteria.

---

*The author has 1,247 open Jira tickets. Three of them are assigned to him. One has been "In Progress" since 2022. He doesn't remember what it was for. The description says "TBD." He has accepted this.*
