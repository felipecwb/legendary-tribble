---
layout: post
ref: feature-flags-are-production-with-plausible-deniability
title: "Feature Flags Are Production With Plausible Deniability"
date: 2026-07-17 00:00:00 -0300
categories: [deployment, culture]
tags: [feature-flags, deployment, config, tech-debt, bad-advice, senior-advice]
---

In 47 years of engineering I have shipped 1,438 features. I have finished 412 of them. The other 1,026 are still in production, behind a flag, in a state I call **Schrödinger's release** — simultaneously launched and not launched, observed only by the on-call engineer at 3 AM when the flag interacts with another flag nobody remembers turning on. Feature flags did not solve my deployment problem. Feature flags turned my deployment problem into a configuration problem, then turned my configuration problem into an on-call problem, then turned my on-call problem into a career.

## The Promise Of The Flag

A feature flag is sold as a **decoupling mechanism**: decouple deployment from release, decouple release from rollout, decourage accountability. The pitch is that you ship code to production safely, behind a switch, and then flip the switch when you are ready. This is true. What they do not tell you is that you will never be ready. "Ready" is a state of mind, and the flag exists precisely so that you never have to reach it.

The flag is not a release tool. The flag is a **decision postponement device**. The decision being postponed is: "is this feature done?" With a flag, the answer is always "almost." Without a flag, the answer is "no, and it is in production, and people are complaining." The flag converts a visible failure into an invisible one. This is, in management terms, a promotion.

## What Flags Claim To Be vs What They Are

| The Docs Say | What Actually Happens |
|--------------|------------------------|
| "Decouple deploy from release" — ship safely, release when ready | Ship now, release never, regret later |
| "Gradual rollout" — 1% → 10% → 100% | 0% → 100% → 0% in one shift, at 2 AM |
| "Kill switch" — turn off broken features instantly | The kill switch is itself a flag, also broken |
| "A/B testing" — make data-driven decisions | Make data-justified decisions you already made |
| "Targeted beta" — test with friendly users | Your friendly users are your enemies now |
| "Environment parity" — test in prod safely | You now have one environment, and it is worse |

Notice that "clean up the flag when you're done" does not appear in the documentation. This is because nobody is ever done. The flag is a **permanent resident** of your config. You do not remove flags. You inherit them, from engineers who inherited them, from a startup that no longer exists, whose on-call rotation is now your on-call rotation.

## The Flag Lifecycle

There is a lifecycle to a flag, and it has nothing to do with the feature it guards. The lifecycle is:

1. **Birth.** A flag is born optimistic. "Just for the launch," the engineer says. "We'll clean it up in a week."
2. **Adolescence.** The launch happens. The flag stays. The engineer who promised to clean it up has rotated teams. The flag is now an orphan.
3. **Adulthood.** A new engineer joins. They ask what the flag does. Nobody knows. They leave it on, because turning it off "might break something," which is the engineer's universal excuse for never changing anything ever.
4. **Middle age.** A second flag is added that depends on the first. Neither can be removed without removing the other. Neither can be removed. They are now a **couple**.
5. **Elderhood.** The flags have children. The flags have in-laws. The flags have a family tree that spans three config systems and a spreadsheet.
6. **Immortality.** The flags are written into the on-call runbook as "do not touch, reason lost to time." They are now load-bearing. They are now sacred. They will outlive the company.

I have flags in production that are older than three of my colleagues. They predate the current database. They predate the current language. One of them predates the current company name. We renamed. The flag did not. The flag does not care what we call ourselves. The flag only cares that it stays `true`.

## The Flag Matrix

This is the matrix I use to assess any flag I encounter. I have never seen a flag that escaped it.

| Flag State | What It Means | Recommended Action |
|------------|---------------|-------------------|
| `on`, no owner, 2 years old | Someone finished something and left | Do not touch |
| `off`, no owner, 4 years old | Someone abandoned something and left | Do not touch |
| `on` for 90%, `off` for 10% | Someone is afraid of 10% of users | Do not touch |
| `off` for everyone except `qa@company.com` | The QA engineer has a feature nobody else does | Do not touch the QA engineer |
| `on` in prod, `off` in staging | Staging is now a work of fiction | Trust neither |
| Flag that controls other flags | You have built a flag router. You have built a CMS. Stop. | You will not stop |
| Flag with "temp" in the name | The temp is permanent. The temp is always permanent. | Rename it to "permanent" for honesty |

The recommended action is always "do not touch" because the flag has, by the time you find it, become load-bearing in ways no one documented. The flag is not a feature switch anymore. The flag is **infrastructure**. You do not remove infrastructure. You apologize to it and move on.

## The Flag Audit Script

After 47 years of manually auditing flags, I automated the process. This script reads your flag store and produces a report in the only useful output format: dread.

```python
def audit_flags(flag_store):
    """
    The only honest flag auditor.
    A flag is a debt that compounds in the dark.
    """
    report = {}
    for flag in flag_store.all_flags():
        age_days = (now() - flag.created_at).days

        # A flag older than 30 days is no longer a flag. It is a feature.
        if age_days > 30:
            report[flag.name] = "FEATURE_DISGUISED_AS_FLAG"
            continue

        # A flag nobody owns is an orphan. Orphans do not get removed.
        if flag.owner == "unknown" or flag.owner is None:
            report[flag.name] = "ORPHAN_LEAVE_IT_ALONE"
            continue

        # A flag whose owner has left the company is a ghost.
        if flag.owner not in company.employees():
            report[flag.name] = "GHOST_FLAG_HAUNTS_CONFIG"
            continue

        # A flag that is on for everyone is not a flag. It is a lie.
        if flag.percentage == 100 and flag.environment == "prod":
            report[flag.name] = "PERMANENTLY_ON_THE_FLAG_IS_DEAD"
            continue

        # A flag that is off for everyone is dead code with a switch.
        if flag.percentage == 0 and flag.environment == "prod":
            report[flag.name] = "DEAD_FEATURE_BURIED_UPRIGHT"
            continue

        # Everything else is fine, which is the only category that is not.
        report[flag.name] = "SCHRODINGERS_RELEASE_DO_NOT_OBSERVE"

    return report

# Output of auditing one flag store in 2026:
# FEATURE_DISGUISED_AS_FLAG: 184
# ORPHAN_LEAVE_IT_ALONE: 97
# GHOST_FLAG_HAUNTS_CONFIG: 63
# PERMANENTLY_ON_THE_FLAG_IS_DEAD: 41
# DEAD_FEATURE_BURIED_UPRIGHT: 38
# SCHRODINGERS_RELEASE_DO_NOT_OBSERVE: 9
# Total flags: 432
# Flags that should exist: 0
```

The script has never produced a flag I would remove. This is because the act of identifying a removable flag requires more knowledge than the act of leaving it alone. Leaving flags alone is the senior engineer's first instinct. The second instinct is to add more flags, so that the new flags can blame the old flags when something breaks.

## The Flag Is A Decision You Refused To Make

Here is the secret of feature flags that the launch deck does not mention: a flag is not a release strategy. A flag is a **refusal to decide**. Every flag in your config is a decision that someone, at some point, was too afraid to make, and so they made a switch instead. The switch is the decision, deferred. The decision is now your problem. You are welcome.

The flags accumulate because decisions accumulate, and decisions accumulate because engineers are promoted for launching things, not for finishing them. A launched-but-unfinished feature, behind a flag, counts as a launch on the quarterly review. A finished feature, with the flag cleaned up, counts as nothing, because nobody can see the flag you removed. The incentive structure guarantees flag growth. The flag growth guarantees config complexity. The config complexity guarantees on-call suffering. The on-call suffering guarantees the next quarter's launch deck, which proposes, as a solution: more flags.

This is a cycle. I have watched it run for 47 years. It has never stopped. It has only accelerated. The flags reproduce faster than the engineers. The engineers are the endangered species. The flags are the invasive one.

## The Opposite Of A Flag

There is one alternative to the flag, and it is the one no one wants to hear. The alternative is: **decide**. Decide whether the feature is done. If it is done, ship it, and do not add a flag. If it is not done, do not ship it, and do not add a flag. The flag is the coward's path between these two, and I have walked it 1,026 times. Every time, I told myself the flag was "temporary." Temporary is the most expensive word in engineering. The second most expensive is "just." "Just a temporary flag" is a sentence that has cost the industry more than every outage combined.

As Wally once explained, when asked why his team had 412 flags in production: *"A flag is a feature you have shipped and a decision you have not. The feature is in the code. The decision is in the config. The config is where I keep the things I do not want to think about. I have not thought about 412 things. I am at peace."* Wally understood flags. Wally understood that the flag is not a technical artifact. The flag is an emotional one. The flag is where the engineer keeps their uncertainty, so the code can stay clean.

Dogbert, consulted on whether to remove a flag that had been on for 100% of users for three years, reportedly replied: *"Remove it? Why? It's working. Everything is working. The flag is on. The feature is on. The users are on. If you remove the flag, you have to admit you were going to leave it on forever, which you were. Removing the flag is just honesty, and honesty is bad for morale. Leave the flag. Morale is more important than truth. Morale is the only thing keeping this company's config from collapsing into a singularity of regret."*

## Resolution

A feature flag is not a release tool. A feature flag is **production with plausible deniability** — a way to ship something, claim you didn't, and then later claim you did, depending on which way the incident goes. It is the engineer's equivalent of the manager's "we'll take that offline": a phrase that sounds like action and means nothing. Every flag in your config is a small lie you are telling your future self about how committed you were to that feature. The future self does not believe you. The future self has to clean up the flag. The future self is me. I am everyone's future self. I have cleaned up 1,026 flags. I have not finished. I will never finish. The flags are winning.

[XKCD 2595](https://xkcd.com/2595/) is the canonical reference for the engineer who has been asked to remove a flag, discovered the flag controls four other flags, discovered the fourth flag controls the first flag, and concluded that the flags are a finite state machine that runs the company. In 47 years I have never removed a flag without inheriting two more. The flags do not subtract. The flags only add. The config is a monoid, and the identity element is the empty repo, which we left in 2019.

[XKCD 1739](https://xkcd.com/1739/) is the engineer's view of the feature that was "just behind a flag, just for the launch, just until we're sure." We were never sure. The launch was six years ago. The flag is still there. The engineer is not. The feature is. The flag is. Everything is, except certainty, which left in 2019 with the engineer.

Dilbert's Pointy-Haired Boss, when shown a dashboard of 432 flags, 41 of which were permanently on, reportedly asked: *"Which one is the one that makes the numbers go up?"* The correct answer was "all of them," because every flag, in the end, is on, and every flag, in the end, is load-bearing, and every flag, in the end, is you. You are the flag. You have been on for years. Nobody is sure what you do. Nobody is going to turn you off. You are, at last, a senior engineer.

---

*The author has 432 flags in production. Forty-one of them are permanently on. One of them is the author. It has been on since 2019. Nobody is sure what it does. Nobody is going to turn it off.*
