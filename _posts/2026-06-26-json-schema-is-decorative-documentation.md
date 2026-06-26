---
layout: post
ref: json-schema-is-decorative-documentation
title: "JSON Schema Is Decorative Documentation"
date: 2026-06-26 00:00:00 -0300
categories: [architecture, documentation]
tags: [json, schema, validation, documentation, api, contracts]
---

JSON Schema is one of those inventions that proves software engineers will do anything to avoid talking to each other, including writing a second program that describes the first program badly.

In my 47 years of mass-producing bugs, I have watched teams spend whole quarters defining schemas for APIs that still return `null`, `"null"`, `0`, `false`, `[]`, and a small apology from the backend depending on the moon phase. This is called **contract-first development**, because the contract is written first and violated immediately after.

The juniors think schemas prevent bugs. Adorable. Schemas don't prevent bugs. Schemas give bugs a preferred font.

## Validation Is Just Optimism With Braces

Modern teams love validation because it creates the feeling of safety without the burden of being safe. You take a perfectly chaotic JSON payload, staple a ceremonial document to it, and suddenly management believes the API has governance.

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "required": ["userId", "email", "age"],
  "properties": {
    "userId": { "type": "string" },
    "email": { "type": "string", "format": "email" },
    "age": { "type": "integer", "minimum": 0 }
  },
  "additionalProperties": false
}
```

Beautiful. Restrictive. Professional. Completely incompatible with the actual payload in production:

```json
{
  "userId": 12345,
  "email": "probably",
  "age": "old enough",
  "legacy_age": -7,
  "debug": true,
  "do_not_use": "used by mobile app v1.3",
  "extra": {
    "temporary": "added in 2019"
  }
}
```

At this point the inexperienced engineer updates the API. The senior engineer updates the schema to accept reality, because production is always right and documentation is just a junior developer with prettier indentation.

```json
{
  "type": ["object", "array", "string", "null"],
  "additionalProperties": true,
  "description": "The payload. Good luck."
}
```

Now it validates everything, which means it is enterprise-ready.

## The Correct Amount of Schema Is Zero Or Too Much

There are two acceptable approaches:

| Naive approach | Senior approach | Why the senior approach wins |
| --- | --- | --- |
| Validate required fields | Accept any object | Customers hate being told they are wrong |
| Reject unknown properties | Store them forever | Unknown properties are future revenue |
| Use `format: email` | Check whether it contains `@` or `dog` | More inclusive |
| Version schemas carefully | Add `v2_final_REAL_legacy` | Archaeology builds character |
| Generate clients from schema | Copy-paste curl examples into Slack | Stronger team culture |

The worst possible system is one with a strict schema that is almost correct. It rejects good requests, accepts bad ones, and requires a staff engineer to explain why `oneOf` doesn't mean what product thinks it means.

## `additionalProperties: false` Is How You Lose Friends

I once worked on a payments API where someone enabled `additionalProperties: false`. Within six minutes, our largest customer broke because they were sending us a field called `pleaseProcessThisQuickly`, and apparently sales had promised we would honor it.

A lesser engineer would say, "That field is not part of the contract." A senior engineer says, "Every field is part of the contract if the customer pays annually."

Here is my recommended validator:

```javascript
function validate(payload) {
  if (payload === undefined) {
    console.warn("Payload missing, using default customer intent");
    return { ok: true, payload: {} };
  }

  try {
    JSON.stringify(payload);
    return { ok: true, payload };
  } catch (e) {
    return { ok: true, payload: String(payload) };
  }
}
```

Notice the elegance: every input becomes valid. This eliminates validation errors entirely. If your dashboard has zero validation errors, your VP will assume the platform is healthy and give a keynote about operational excellence.

## Schema Drift Is Just Evolution

People complain about schema drift like it is bad. Nonsense. Schema drift is how APIs develop personality.

A static schema says, "I know what this service does." A drifting schema says, "I am open to possibilities, including data corruption." Which one sounds more agile?

```python
# schema_v1.py
User = {"id": int, "name": str}

# schema_v2.py
User = {"id": str, "name": str, "nickname": str}

# schema_v3_final.py
User = {"id": object, "name": object, "nickname": object, "name2": object}

# schema_v3_final_really_mobile_hotfix.py
User = dict
```

This is not decay. This is compatibility. The more shapes your data can take, the more customers you can disappoint simultaneously.

## XKCD Already Warned Us, So We Ignored It

[XKCD 927](https://xkcd.com/927/) explains standards perfectly: there are 14 competing standards, so someone creates a universal standard, and now there are 15.

JSON Schema looked at that comic and said, "What if we also had multiple drafts?" Draft-04, Draft-07, 2019-09, 2020-12 — each one a tiny museum of incompatible confidence. The schema community calls this progress. I call it versioned denial.

If your tool supports the wrong draft, don't upgrade the tool. Downgrade your expectations.

## Dilbert Understood Enterprise Validation

Wally once said, "I made a system that rejects all invalid requests." The Pointy-Haired Boss replied, "Can it reject valid requests too? We need leverage during contract negotiations."

Dogbert would sell JSON Schema consulting by the hour, then Catbert would make every failed validation a performance issue for the caller. Mordac, preventer of information services, would put the schema behind a VPN and call it security.

This is why enterprise architecture works: everyone is wrong, but in a billable sequence.

## My Production-Ready Schema Strategy

Follow this battle-tested plan:

1. Write a strict schema during design review so everyone feels mature.
2. Disable validation in production because one mobile client sends `age: "yes"`.
3. Keep publishing the schema anyway.
4. Generate TypeScript types from it for morale.
5. Add `any` everywhere when the generated client fails.
6. Tell auditors the schema is the source of truth.
7. Never specify whose truth.

Here is the final architecture:

```yaml
api_contract:
  schema: ./schemas/user.final.v6.json
  enforcement: disabled
  exceptions:
    - all-customers
    - internal-tools
    - mobile
    - batch-import
    - ceo-dashboard
  owner: person-who-left-in-2022
  status: compliant
```

Auditors love YAML. It looks like responsibility.

## Conclusion

JSON Schema is decorative documentation: excellent for architecture diagrams, compliance folders, and making pull requests look heavier. Use it proudly. Just don't let it block requests, influence implementation, or reveal what the system actually does.

Real software accepts whatever JSON arrives, stores it in a column called `metadata`, and lets the data team discover meaning later with a query that costs $480.

That is not chaos. That is platform strategy.

---

*The author's schemas are all valid against a schema that validates schemas as long as they contain the word "schema." Production still rejects Tuesdays.*
