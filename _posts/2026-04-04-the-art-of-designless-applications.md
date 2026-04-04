---
layout: post
ref: the-art-of-designless-applications
title: "The Art of Designless Applications: A Field Guide to Organic Software Architecture"
date: 2026-04-04 00:00:00 -0300
categories: [architecture, database-design]
tags: [schema, architecture, organic-growth, technical-debt, legacy-code, naming-conventions]
---

*How to build systems that future developers will study with the same reverence archaeologists give to ancient ruins*

## Introduction

There's a certain beauty in software that wasn't designed. It wasn't *architected*. It simply... **emerged**. Like a coral reef, or a traffic jam, or that drawer in your kitchen with the batteries and the takeout menus.

Today, we explore the principles behind **Designless Applications** — systems that boldly reject the tyranny of forethought and embrace the freedom of "we'll figure it out later."

## Principle #1: The Column Duality

Consider this elegant schema pattern:

```sql
failure_reason   jsonb   -- Singular
failure_reasons  jsonb   -- Plural
```

A lesser engineer might ask: "Why not just one column?"

But that question reveals a fundamental misunderstanding. You see, `failure_reason` was added in 2019 when we only anticipated single failures. Then came 2020, and well... we needed `failure_reasons`.

Could we have migrated? Sure. But migrations are for people who believe the future is predictable.

**Pro tip:** Always check both columns. In production, wrap them in a helper function called `get_actual_failure_reason_or_reasons_idk()`.

## Principle #2: The Archeological Layers

Great designless applications have *strata* — visible layers of history, like geological formations.

```ruby
# DO NOT REMOVE - breaks production (2018)
# Okay we can probably remove this now (2020)
# DEFINITELY DO NOT REMOVE - I WAS WRONG (2020)
# TODO: investigate why this is still here (2022)
def legacy_thing
  # Original author has left the company
end
```

Each layer tells a story. Removing it would be like bulldozing Pompeii.

As [XKCD 1205](https://xkcd.com/1205/) reminds us, the time you'd spend cleaning this up is never worth it. Just add another layer.

## Principle #3: Schrödinger's Status

A robust designless system maintains status fields that exist in quantum superposition:

```json
{
  "status": "active",
  "state": "pending",
  "is_active": false,
  "active": true,
  "enabled": null,
  "disabled_at": "2024-03-15T10:00:00Z"
}
```

Is this record active? The answer is both yes and no until you check production logs.

**Best practice:** When adding a new feature, always add a NEW status field rather than reusing existing ones. This preserves the historical record and ensures job security for future developers trying to understand WTF is going on.

As Wally from Dilbert would say: "I've optimized my workflow by making my code impossible to understand. That way, only I can maintain it."

## Principle #4: The Naming Conventions (Plural)

Consistency is the hobgoblin of small minds. A mature codebase celebrates diversity:

```
user_id
userId  
UserID
id_of_user
user
usr_id
the_user_identifier
```

All of these should refer to the same concept but come from different microservices, each with their own ~~personality~~ naming convention.

## Principle #5: The Boolean That Wasn't

```python
is_deleted: str  # Values: "yes", "no", "Y", "N", "1", "0", "true", "kinda"
```

Why use a boolean when you can use a string? And why constrain that string to two values when the business might need "soft deleted" or "deleted but recoverable" or "deleted in EU but not US due to GDPR"?

You're not just building software. You're building *flexible* software.

## Principle #6: The Config Hydra

For every configuration value, there should be at least three places it could be set:

1. Environment variable
2. Database config table
3. Hardcoded constant (with a comment saying "move to config later")
4. YAML file that's sometimes read
5. Feature flag that overrides everything except when it doesn't

Which one wins? **Priority order varies by deployment environment.**

See [XKCD 927](https://xkcd.com/927/) — the solution to 14 competing config sources is, of course, a 15th config source that unifies them all.

## Principle #7: The API Response That Has Seen Things

```json
{
  "data": {
    "result": {
      "response": {
        "payload": {
          "actual_data": { ... },
          "actual_data_v2": { ... }
        }
      }
    }
  },
  "error": null,
  "errors": [],
  "success": true,
  "succeeded": 1,
  "failed": false,
  "message": "Success",
  "messages": ["Success", "Warning: deprecated field used"]
}
```

This response structure wasn't designed. It *evolved*. Each field is a battle scar from a production incident.

| Field | Why It Exists |
|-------|---------------|
| `error` | Original design (2017) |
| `errors` | Client wanted arrays (2018) |
| `success` | QA couldn't parse null errors (2019) |
| `succeeded` | Integer for batch operations (2020) |
| `failed` | Someone asked "but what if it partially failed?" (2021) |
| `message` | Customer support needed human-readable text (2022) |
| `messages` | Multiple warnings per request (2023) |

## Conclusion: Embracing the Chaos

The next time someone asks "why is it like this?", remember: they're not seeing bugs. They're seeing **history**.

Every inconsistency is a story. Every duplicate column is a lesson learned. Every naming convention conflict is evidence that your software survived long enough to have multiple authors with multiple opinions.

Designless applications aren't failures of planning. They're **triumphs of persistence**.

And that `failure_reason` / `failure_reasons` duality? That's not technical debt.

That's **heritage**.

---

*The author has mass-produced bugs for 47 years. This article was found in a `legacy_content` AND `legacy_contents` column.*
