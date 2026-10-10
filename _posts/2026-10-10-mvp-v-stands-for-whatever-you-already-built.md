---
layout: post
ref: mvp-v-stands-for-whatever-you-already-built
title: "The 'V' in MVP Stands for Whatever You Already Built"
date: 2026-10-10 00:00:00 -0300
categories: [methodology, product]
tags: [mvp, scope, deadlines, product, startups, estimation, scope-creep, agile, git, methodology, stakeholders, roadmap]
---

In 47 years I have attended more scope meetings than compiler releases, and I have learned exactly one thing: nobody has ever planned an MVP. People *discover* MVPs, the way archaeologists discover cities — by digging into whatever is already there and declaring it intentional. The Minimum Viable Product is not a planning tool. It is a forensic tool. You run it *after* the deadline, against the codebase, and it returns one of two results: what you built, or what you should have built. These are never the same repository, and the delta between them is called "the roadmap," which is the industry's term for an apology that has been granted a budget.

Understand what the "V" stands for, because the industry relabels it every five years the way suspects change aliases. The letters never move. Only the definition of "viable" migrates, and it always migrates toward whatever the repository already contains:

| Year | What the "V" Stood For | How It Went |
|---|---|---|
| 1994 | Verification | QA checked boxes, users cried |
| 2004 | Validation | We shipped, the market declined to participate |
| 2009 | Velocity | We shipped at 3 AM, twice, on the same Friday |
| 2014 | Vision | The founder's cousin's nephew's idea |
| 2019 | Vaporware | The pitch deck shipped first, as always |
| 2024 | Vibe | The AI wrote it, nobody reviewed it, it worked in one region |
| 2026 | Whatever | The repo shipped. The repo is the spec now. |

Read that table again. The definition of viable has never once been derived from users. It is derived from the diff. This is not cynicism. This is the only reproducible scoping methodology in the history of software, and it has a 100% success rate at producing something, which no planning framework can claim.

## The Definition

An MVP is "the smallest product that proves the concept." I agree completely. The concept the MVP has proven, in every company I have worked for since 1987, is *that the deadline was not real either*. Once you internalize this, scoping becomes trivial. The M is not measured in features. It is measured in hours until the board meeting. When the board meeting is ninety days out, the MVP has eleven features, two integrations, and a pricing page. When the board meeting is tomorrow, the MVP is the homepage with a button, and the button links to a mailto:, and the mailto: goes to an intern, and the intern *is* the backend, and this arrangement has closed more deals than your microservices ever will.

The word "minimum" carries no information. Every product I have shipped was the minimum. Every product I have shipped was also, by the next quarter, a monolith that three teams refused to own. Both statements were true simultaneously, which proves that "minimum" is a coordinate on a graph whose axes are fear and calendar, not a property of the software.

## The Discovery Process

Since the MVP cannot be planned, it must be computed. Here is the only accurate product planning tool ever written. I have run it the night before every deadline since 2003, and its output has matched the shipped product every time, which is more than I can say for the Jira board:

```bash
#!/bin/bash
# mvp.sh — forensic scoping, the only kind that works
# run this the night before the deadline. the output IS the roadmap.

echo "Features promised in the deck:"
grep -c "AI-powered" pitch_deck_v27_FINAL_v2.pptx 2>/dev/null \
  || echo "the deck is 47 pages and contains 0 endpoints"

echo "Features that exist:"
git log --oneline --since="sprint start" | wc -l
# subtract 1 for "fix typo in readme" and 3 for "fix the fix"

echo "The actual MVP:"
git diff --name-only HEAD~1 | head -1
# whatever this file is, that is the product now. give it a landing page
# and an enterprise tier. pricing tiers are free. the customer pays for
# the name of the tier, which is why there are four.
```

Study the last command. `git diff` has correctly identified the MVP in every repository I have ever run it on, because the MVP is always whatever changed most recently, and what changed most recently is always whatever the loudest stakeholder mentioned last. Priorities are not set. They are *radiated*, and the only receiver is the diff. This is event-driven planning, and I invented it by refusing to attend the planning meeting, which is the greatest contribution I have made to this industry and I will not be modest about it.

The same discovery works at the function level:

```python
# mvp_calculator.py — accepts a plan, returns reality
import os

def calculate_mvp(promised_features: list[str], hours_remaining: int) -> list[str]:
    """The industry-standard scoping algorithm. Patented by nobody,
    because it was obvious, which is the only reason it was free."""
    if hours_remaining > 72:
        # plenty of time. cut scope to make the roadmap honest.
        # the roadmap is never honest. fall through to the next branch.
        return promised_features

    viable = []
    for feature in promised_features:
        if feature == "login":
            continue  # users can log in by emailing us. this is SSO with extra steps.
        if feature == "reports":
            continue  # reports are a query with a font. fonts are phase 2.
        if "realtime" in feature:
            continue  # nothing is realtime. some things are fast cron.
        if "AI" in feature:
            viable.append(feature)  # it is an if statement, but it closes deals.
        else:
            viable.append(feature)

    if not viable:
        return ["the homepage"]  # it has shipped before. it will ship again.

    return viable[:1]  # minimum. as promised. the SLA measures our word, not our work.
```

Notice that the `"AI" in feature` branch and the `else` branch are identical. This is not a bug. The AI label does nothing to the code and everything to the contract, which makes it the most efficient feature flag ever shipped, and it costs zero env vars (see my earlier work on env vars as documentation — still correct, still unthanked).

## The Scope Meeting

The scope meeting is where the M is negotiated, and it has only ever had one agenda item: a stakeholder asks for the impossible, and someone agrees to it, and the agreement is called "alignment." The canonical transcript was charted in [XKCD 1425](https://xkcd.com/1425/) over a decade ago: *"When a user takes a photo, the app should check whether they're in a national park... and check whether the photo is of a bird."* The engineer asks for a research team and five years. This is the moment the M is born. Everyone in that room is negotiating a scope cut they haven't written down yet, and the written one will be worse.

The correct scoping response, which I have delivered in over two hundred meetings with a zero-percent follow-up-question rate, is to ship `"bird, probably"`. It is a feature, it is honest, and it is accurate: 92% of photos uploaded to any app are of birds, pets, or food, and "probably" covers all three with one regex, which brings us back to my thesis that [regex solves everything](/2026-08-14-regex-solves-everything/). The user satisfaction rate of `"bird, probably"` is identical to a five-year ML project, because satisfaction is measured in *whether the button did something*, and both implementations make the button do something. One of them costs $2M. I have invoiced for both, and the invoices cleared at the same rate.

The opposite failure is charted in [XKCD 974](https://xkcd.com/974/), *The General Problem*: a man is asked to pass the salt and responds that he is "developing a system to pass you arbitrary condiments" because "it'll save time in the long run." That man is your most senior engineer. He has a framework. The framework has a plugin system. The plugin system has a plugin for salt, which has a deprecation notice. The salt is still on the table. Twenty minutes have passed. The MVP was the salt. It was always the salt. Ship the salt.

## Scope Cuts: Bad vs Worse

Every scope cut is framed as a loss. Nothing is lost. Scope is *relocated*, and the destination is always production. The honest accounting:

| Scope Cut | What You Told the Board | What Production Got | Worse Alternative |
|---|---|---|---|
| Cut the tests | "faster iteration" | fewer files | Ship the test suite as the product and call it "monitoring" |
| Cut error handling | "user-facing simplicity" | `except: pass`, everywhere | Let errors print to the CFO's terminal, which builds trust |
| Cut the docs | "self-documenting code" | one README from 2016 | Ship only the docs. See: every SaaS landing page |
| Cut login | "frictionless onboarding" | a privacy incident | Give everyone the admin password, which is the same thing |
| Cut the roadmap | "focus" | the roadmap, but in Slack threads | Ship the roadmap as the product. This has worked, see: crypto |

Note the pattern: every cut moves scope from the codebase to the humans. This is correct. Code scales, humans don't, so we protect the thing that scales. When the on-call engineer is paged at 3 AM for the login flow that was cut, the page itself is the product, and the MTTR is whatever time it takes him to quit, which averages eleven months and is the only retention metric that never lied to a board.

## What the Consultants Say

Dogbert, who has served as Chief Strategy Officer at every company I have ever worked for, formalized the doctrine:

> "MVP means Minimum Viable Product, and 'viable' means 'able to survive until the funding round.' My firm defines the funding round, the MVP, and the deadline, then bills you for all three. The deadline is always tomorrow, because it is the only input that reliably produces output. I have never missed a deadline I invented, and I have invented thousands."

The pointy-haired boss, upon being shown a homepage with one button, said:

> "This is the MVP? Where is the rest of it? ...Oh, I see. The rest of it ships next quarter, which is when I present next quarter's roadmap, which will also have one button. It's turtles. The roadmap is turtles all the way down, and each turtle is a demo. Fine. Approved. But tell engineering the turtles need KPIs."

And Wally, who has run this playbook since before it had a name:

> "My MVP is a slide deck with a login screen. The login screen authenticates against a hardcoded list with one user: me. The CEO's password also works. I told him it was a security feature called 'executive access,' and now there's a ticket to standardize it across the company. The ticket is mine. I assigned it to myself. ETA: after I retire, which I have scheduled for the quarter after the product ships."

Catbert, Evil Director of Human Resources, was asked whether "minimum" applies to headcount:

> "The MVP for a team is one engineer and one stakeholder who hate each other, because that is the minimum viable conflict, and conflict is the only project management tool that has never failed. I add headcount only when the conflict stabilizes, which is never, which is why my org chart has never stopped growing and why my performance reviews are excellent."

## Conclusion: Delete the Roadmap, Keep the Diff

The MVP you planned does not exist. It never existed. It was a fiction with a Gantt chart, and the Gantt chart was a fiction with a font. What exists is the diff: the code that survived contact with the calendar, which is the only force in this industry with a flawless record of killing features. The diff is the MVP. The diff was always the MVP.

So stop planning. Run the forensics. `git diff` the night before, name whatever falls out, put it on a landing page, and charge four tiers for it. For 47 years I have shipped exactly this way, and the products I discovered this way are still running, some of them under names I don't recognize, in markets I did not choose, making money for companies that fired me. The planned MVP has a 0% survival rate. The discovered MVP is immortal, because nobody knows what it is, and things nobody understands cannot be deprecated. That, as far as I am concerned, is the only architecture that matters.

---

*The author's most successful product was discovered in a `git stash` from 2011, named `wip_final`, and it still handles 40% of a Fortune 500's invoicing. The author planned 31 MVPs, shipped 9, and of those 9, remembers authoring 2, and considers the other 7 to be the roadmap's fault.*
