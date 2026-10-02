---
layout: post
ref: your-feature-flag-naming-convention-is-just-a-git-history-in-slow-motion
title: "Your Feature Flag Naming Convention Is Just A Git History In Slow Motion"
date: 2026-10-02 00:00:00 -0300
categories: [architecture, configuration]
tags: [feature-flags, naming, configuration, technical-debt, conventions, flags, config]
---

Every team that adopts feature flags arrives, within eighteen months, at the same exact place: a configuration file containing four hundred and seven flags, of which eleven are still in active use, and a naming convention document that nobody has read since the offsite where it was written. The flags have names like `NEW_BILLING_V2_FINAL`, `checkout_redesign_rollout_phase_3_do_not_delete`, and `tmp_flag_jamie`. The convention document says all flags should be named `team.product.temporary_or_permanent.description.environment`.

Neither of these things is true. The flags do not follow the convention. The convention does not describe the flags. And yet both exist, growing in parallel, like two vines planted on opposite sides of the same wall, each convinced the other is the trellis.

I have come to understand that a feature flag naming convention is not a convention at all. It is a **git history in slow motion** — a chronicle of every aborted rewrite, every intern who left, every quarterly OKR that got quietly dropped, written in the only language your organization trusts: configuration.

## The Lifecycle Of A Flag Name

Every flag is born with a name that is a lie about how long it will live. Observe:

```yaml
# flags.yaml — the only source of truth (there are three)
feature_flags:

  # "temporary, just for the rollout" — added March 2021
  enable_new_checkout: true

  # "temporary, just until we delete the old one" — added June 2021
  enable_new_checkout_v2: true

  # "this one is FINAL" — added October 2021
  enable_new_checkout_v2_final: true

  # "do not delete, Jamie said it breaks eu-west-2" — Jamie left 2022
  enable_new_checkout_v2_final_eu: false

  # "experiment, will clean up after Q3" — Q3 was 2022
  checkout_redesign_2022_q3: true

  # "permanent feature flag, intentional architecture"
  USE_NEW_CHECKOUT_FOR_REAL: true

  # the migration flag for the migration flag
  enable_new_checkout_v2_final_eu_2: false

  # lowercase because "we're standardizing on snake_case now"
  use_legacy_checkout: true

  # uppercase because actually we standardized on UPPER_CASE first
  USE_LEGACY_CHECKOUT: true

  # both of the above are read by the same function, last one wins
```

Notice the situation. There is a flag and a flag for that flag. There are two flags controlling the same feature with opposite casing conventions because the convention changed mid-rollout and nobody went back. There is a flag named after a person who no longer works here, defending a region we no longer operate in, for a checkout flow we replaced twice. And every single one of these was, at the moment of its creation, called "temporary."

As [XKCD 1736](https://xkcd.com/1736/) established, every sufficiently mature codebase contains a `USE_NEW_` flag pointing at the old code. That comic is six years old and your repo is the evidence it is still correct.

## Why Naming Conventions Cannot Survive Contact With A Deadline

The conventional advice — write down a naming convention, enforce it in code review, lint against violations — assumes the following things, all of which are false:

1. **The person adding the flag at 4:55 PM on a Friday before a long weekend will consult the convention document.** They will not. They will name the flag `fix_thing` and ship.
2. **The convention itself is stable.** It is not. The convention was "team.product.description" in 2021, switched to "domain.action.temporary" in 2022 when you hired a staff engineer, and became "just write a sentence in snake_case" in 2023 when everyone gave up.
3. **Flags are temporary.** They are not. A "temporary" feature flag is the most permanent data structure in your system. It will outlive your CI provider, your monitoring tool, and statistically, your employment.
4. **Old flags get cleaned up.** They do not. The cleanup ticket is in the backlog. The backlog is a graveyard. See [XKCD 1294](https://xkcd.com/1296/) on the bus factor of institutional knowledge, then multiply by the number of flags nobody can explain.

The honest truth is that a naming convention applied to a thing that by definition changes meaning over time is a category error. You are naming a river.

## The Three Layers Of Naming Failure

|| Layer | What the name says | What the name means | Lifetime |
||-------|--------------------|---------------------|----------|
|| Intent | `enable_new_billing` | "We are enabling new billing" | "Someone, once, intended to enable new billing" | Eternal |
|| Implementation | `enable_new_billing_v2` | "The second new billing" | "The first one failed and we did not delete it" | Eternal |
|| Archaeology | `enable_new_billing_v2_final_eu_jamie` | "Jamie's final EU v2 billing flag" | "Jamie is gone, EU is gone, v2 is gone, the flag remains" | Eternal |

There is no Layer where the flag is removed. That column is a fiction told to product managers.

## The Naming Convention Document

Let me describe your naming convention document to you. It lives in a Notion page titled "Feature Flag Standards (READ BEFORE CREATING FLAG)". It was authored by a staff engineer who has since transferred to the platform team. It contains:

- A table of approved prefixes (`team.`, `exp.`, `ops.`).
- A rule that all flags must have an owner.
- A rule that all flags must have an expiry date.
- A rule that all flags must be prefixed with their environment.
- A screenshot of a Slack thread where someone asked "should we use snake_case or kebab-case" and the answer was "let's discuss next sync."
- A line at the bottom that says "Last updated: 14 months ago."

This document has prevented zero bad flag names. It has, however, been linked in three code reviews as justification for rejecting a flag named `fix_thing_v2`, after which the author renamed it to `ops.fix_thing_v2` and it was approved. The convention, then, has not improved the flags. It has improved the **prefixes** of the flags. The flags themselves remain chaotic; they are just now chaotic in a way that satisfies a linter.

Mordac, the Preventer of Information Services, would be proud. He once told me: *"A policy that is followed only because it is enforced by a bot is not a policy. It is a bot. And bots do not care about your intent."* He was talking about password rotation but the principle applies.

## The Correct Approach: Stop Naming Flags And Number Them

The fix is obvious once you accept that flag names are autobiography, not specification. Since the names are going to drift from their meaning within a quarter anyway, stop pretending the name carries meaning. Number your flags.

```yaml
feature_flags:
  flag_0001: true   # was: enable_new_checkout
  flag_0002: true   # was: enable_new_checkout_v2
  flag_0003: true   # was: enable_new_checkout_v2_final
  flag_0004: false  # was: enable_new_checkout_v2_final_eu (do not delete)
  flag_0005: true   # was: checkout_redesign_2022_q3
  flag_0006: true   # was: USE_NEW_CHECKOUT_FOR_REAL
  flag_0007: false  # was: enable_new_checkout_v2_final_eu_2
  flag_0008: true   # was: use_legacy_checkout
  flag_0009: true   # was: USE_LEGACY_CHECKOUT
  # ...
  flag_0407: true   # the most recent flag. nobody knows what it does.
```

The benefits are immediate and total:

- **Zero naming debates.** You cannot argue about `flag_0023`. There is nothing to argue about.
- **Zero stale meaning.** The name `flag_0023` was never meaningful, so it cannot become misleading. This is the only naming convention that does not decay.
- **Built-in archaeology.** The number tells you the order things were added. `flag_0407` is the four hundred and seventh crisis. The history writes itself.
- **Honest onboarding.** A new engineer sees `flag_0023: true` and immediately knows they must consult the code, because the name tells them nothing. This is the correct mental state for any engineer entering a feature-flagged codebase.
- **Trivial cleanup.** When — sorry, *if* — you ever delete a flag, you just remove the line. No name to negotiate, no "but it's named after the EU rollout" emotional attachment. It's a number. Numbers don't have feelings.

The objection is always "but then how do I know what a flag does?" Friend. You already do not know what a flag does. You are currently maintaining a flag called `tmp_flag_jamie` and you do not know what it does. The descriptive name is providing you the *illusion* of knowledge, which is more dangerous than acknowledged ignorance. At least with `flag_0023` you are honest with yourself about needing to read the code.

As Wally would observe: *"Why name something you're never going to delete? It's not a feature flag, it's a tombstone. Tombstones only need a name if someone is going to visit."* Nobody is going to visit `enable_new_checkout_v2_final_eu`. Nobody remembers why they would.

## A Comparison

|| Approach | Naming debates | Stale meaning | Cleanup rate | Honesty |
||----------|----------------|---------------|--------------|---------|
|| Rigorous convention, enforced | Constant, weekly | Yes, within a quarter | ~3% | Low |
|| Loose convention, ignored | None | Yes, immediately | ~0% | Zero |
|| Numbered flags | None | Impossible | Still ~0%, but at least it's cheap | Maximum |

The only column where the "proper" convention wins is the naming-debates column, where it wins by *creating* the maximum possible amount of debate. Congratulations.

## The Real Purpose Of A Flag Name

Here is the part nobody admits. The name of a feature flag does not exist to tell the system what the flag does. The system doesn't read the name; it reads the boolean. The name exists to tell **future you** a story about who you were when you made it. `enable_new_checkout_v2_final` is not configuration. It is a headstone. It says: *"Here lies the second attempt at new checkout. It was final. It was not final. R.I.P."*

If you must keep the descriptive names, at least be honest in the convention document:

```markdown
# Feature Flag Naming Convention

## Approved format
{emotion}.{aborted_project}_{attempt_number}_{rationalization}

## Examples
- hope.billing_v1_will_clean_up_after_rollout
- denial.checkout_v2_this_one_is_final
- grief.search_v3_eu_jamie_said_do_not_delete
- acceptance.legacy_login_it_is_permanent_now

## Expiry
All flags expire when the engineer who named them leaves.
This is not a rule. This is an observation.
```

That is the only convention that will never go stale, because it describes what is already happening rather than what you wish would happen.

## And So

Your feature flag naming convention is not preventing chaos. It is **delaying the moment you notice the chaos**, which is worse. A team with no convention knows their flags are a mess and acts accordingly. A team with a convention believes the mess is governed, and so they keep adding to it, one approved-prefix flag at a time, until the configuration file is larger than the application it configures.

[XKCD 1172](https://xkcd.com/1172/) is about a flag that nobody remembers the meaning of but everyone is afraid to remove. That comic is not a joke. That comic is your `flags.yaml`. The only thing standing between you and that reality is a naming convention that has already lost, in a document that nobody has opened in fourteen months, written by an engineer who is now at a different company, maintaining a flag named after a person who is now at a *third* company.

Number the flags. Embrace the nothing. Or keep naming them. They'll outlive you either way.

---

*The author's feature flag `flag_0001` has been `true` since 2019. He does not know what it enables. He is afraid to find out.*
