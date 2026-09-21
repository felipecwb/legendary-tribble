---
layout: post
ref: your-test-fixtures-are-a-museum-of-bugs-youve-already-fixed
title: "Your Test Fixtures Are A Museum Of Bugs You've Already Fixed"
date: 2026-09-21 00:00:00 -0300
categories: [testing, productivity]
tags: [testing, fixtures, test-data, maintenance, history, archaeology, technical-debt]
---

After 47 years in this industry, I have opened a lot of files I did not understand. I have opened `constants.py` and found the word `MAGIC` defined seven times. I have opened `utils.js` and found a function called `definitelyNotADatabase`. But the only file type that consistently fills me with the holy dread of an archaeologist unsealing a tomb is the test fixture.

A test fixture is, by definition, a snapshot of a bug that once existed, captured at the exact moment someone was angry enough to write a test about it. The bug is now fixed. The test is now passing. The fixture is now eternal. Congratulations: you have built a museum, and admission is free, and the exhibits are lying to you about the present.

## The Fixture Is A Fossil

Let us examine a typical fixture. Here is one I found in a repository last Tuesday:

```json
{
  "user_id": 1001,
  "email": "test-user-2019@deprecated-domain.internal",
  "plan": "enterprise_legacy_v2_deprecated_DO_NOT_DELETE",
  "billing_cycle": "quarterly_but_actually_monthly",
  "tax_rate": "0.0",
  "tax_rate_backup": "0.0",
  "tax_rate_backup_backup": "EXEMPT",
  "legacy_flag": true,
  "legacy_flag_new": false,
  "legacy_flag_newer": null,
  "legacy_flag_final": "yes",
  "_migrated_from_v1": "YES",
  "_note": "DO NOT REMOVE, billing will explode, see ticket #4471 (closed 2019)",
  "address": {
    "line1": "123 Real St",
    "line2": "Apt NULL",
    "country": "US",
    "country_code": "USA",
    "country_code_again": "US",
    "zip": "00000"
  }
}
```

Let us inventory what we are looking at. The `plan` field references a product tier that was discontinued before the pandemic. The `billing_cycle` is a known lie preserved in amber: "quarterly but actually monthly" is a comment from a developer who has since left the company, the country, and possibly the profession. There are three `legacy_flag` variants because someone could not decide whether booleans, strings, or `null` were the correct way to represent "we are not sure." There is an `Apt NULL` — not a null value, the literal four characters `N-U-L-L` — which means at some point a serializer and a database had a fistfight and the database won.

Every single field in this fixture is wrong. None of them exist in the current schema. The ticket referenced in the `_note` was closed in 2019, the year my will to live was also closed. And yet, there it is, loaded by `setUp()`, asserted upon by a test named `test_billing_does_not_explode`, and protected by a comment in the test file that reads:

```python
# I don't know what this does but if I delete it the build fails.
```

This is not a test. This is a *gravestone*.

## A Comparison Of Fixture Management Strategies

| Strategy | What it claims | What it actually does |
|---|---|---|
| Delete the unused fixture | "Removes dead test data." | Causes three unrelated tests to fail, because they silently depended on `tax_rate_backup_backup` being loaded into the global `setUp`. You will spend four days. You will restore it. You will cry. |
| Update the fixture to match the new schema | "Modernizes test data." | Breaks the one test that was passing, because that test asserted on `legacy_flag_final` being the string `"yes"`, and the new schema says it should be a boolean, and the test was never about the new schema, it was about the *pain* of the migration, and pain does not have a schema. |
| Leave the fixture exactly as it is | "Preserves history." | Preserves history. The build stays green. The museum stays open. The curator (you) goes home. |
| Add a new fixture instead of editing the old one | "Avoids breaking things." | Doubles the size of the museum every quarter. By 2027 you have 400 JSON files and a `fixtures/` directory that is larger than your `src/` directory. |
| Write a comment explaining the fixture | "Documents intent." | Lies. The comment from 2019 says "temporary." The comment from 2021 says "TODO: clean up." The comment from 2024 says "I give up." The fixture outlives all three comments. |

Notice that the only row in which the build stays green, the tests stay passing, and the engineer stays employed is the third one. Everything else is a trap dressed up as engineering hygiene.

## The First Law Of Fixtures

I have distilled 47 years of `setUp()` and `tearDown()` into a single law, which I call the First Law of Fixtures:

> **A fixture can be added, but a fixture can never be removed.**

This is not a recommendation. This is physics. A fixture, once loaded by a test that passes, becomes load-bearing. You do not know *which* test depends on it, because the test that depends on it is in a different file, in a different package, and was written by an intern who has since become your manager. The dependency is invisible, transitive, and vindictive. Removing the fixture is the moral equivalent of pulling a load-bearing wall out of a house whose blueprints were lost in a flood. The house will not fall immediately. The house will fall during the demo, in front of the customer, on a Friday.

As [XKCD 2347](https://xkcd.com/2347/) documented with the clarity of a coroner: there is a small, unremarkable dependency at the bottom of the stack that holds up the entire world, and nobody knows what it does, and nobody is allowed to touch it. Your test fixtures are that dependency. They are the `Dependency` column in the comic, except there are nine hundred of them and they are all named `test_user_2018_final_v2_REAL.json`.

## The Fixture Is A Record Of Failure

Here is the part that the testing advocates will not tell you. A test fixture is not a representation of reality. A test fixture is a representation of *the moment reality broke*.

When you write `test_billing_does_not_explode`, you are not testing billing. You are testing *the absence of the bug that once made billing explode*. The fixture is the cast of the wound. The bug is healed. The cast remains. And the cast is now shaped like a billing cycle that no longer exists, for a customer who was deleted, on a plan that was discontinued, in a currency that was devalued.

This is why updating fixtures is dangerous. The fixture is not describing the *present*. It is describing a *specific past failure*. If you "modernize" it to match the current schema, you are not making the test better. You are erasing the only record that the bug ever happened. The bug, sensing it has been forgotten, will return. I have seen this happen. I once removed a `// HACK: tax rounding` comment from a fixture and three weeks later a customer was billed $0.01 more than they should have been, in a country that no longer uses that currency, on a date that was not a Friday but felt like one.

The bug came back because the fixture was keeping it away. The fixture was a scarecrow. I removed the scarecrow. The crows returned. The crows were off-by-one errors.

## What Dilbert Teaches Us

The Pointy-Haired Boss, upon learning that we have 900 fixture files, most of which are unused, will ask the only sensible question a manager can ask: *"Can we just... not touch them?"*

And the answer is yes. That is the entire strategy. The PHB, who has never written a test and does not know what JSON is, has arrived at the correct architectural decision through pure instinct: **do not touch what is not on fire.**

Wally, who has been at the company longer than the fixtures, will add: *"I've been keeping those fixtures alive as a personal project. It's 40% of my job and 100% of my job security."* He is not wrong. The fixtures are job security. Every fixture is a small monument to a problem only Wally remembers. Delete the fixture, and you delete the only reason Wally cannot be replaced by a shell script.

Mordac, the Preventer of Information Services, would go further. He would *mandate* the fixtures. He would require a ticket, a sign-off, and a blood sample to remove a single field from a single JSON file. And he would be right, because the only thing more dangerous than a fixture that exists is the *absence* of a fixture that used to exist.

## The Fixture File That Has Outlived Its Format

A special note must be made of the fixture file whose format is itself dead. I have, in my career, maintained fixtures in:

1. XML
2. YAML
3. JSON
4. CSV
5. A custom DSL that one developer invented in 2008 and took to their grave
6. Excel (the Excel one is the worst, because someone *formatted* it, and the formatting is load-bearing, and if you open it in LibreOffice instead of Microsoft Office the colors shift and a test that asserts on cell background fails)

Each format was, at the time of its creation, the "obvious" choice. Each format is now a liability. The YAML fixtures have a tab character somewhere that no editor will reveal but the parser will never forgive. The CSV fixtures have a comma in a name field that is quoted in a way that only one specific version of one specific library on one specific Tuesday understands. The custom DSL is interpreted by a Python 2 script that we cannot run anymore but also cannot delete, because the test loads the fixture by shelling out to that script, and the script's exit code is asserted upon, and the assertion is the only thing standing between us and a billing explosion.

I keep the Python 2 interpreter installed on my machine for this reason alone. It is the last Python 2.7 on Earth that is still in active production use. I am its custodian. It is my most important responsibility. I am not proud of this. I am, however, still employed.

## The Counterargument (And Why It's Wrong)

Junior engineers, who have read a book, will say: *"But fixtures should be minimal, representative, and kept in sync with the schema. You should generate them programmatically. You should use factories, not files."*

Oh, you sweet, sweet child.

Factories are fixtures that lie about being fixtures. A factory does not remove the fossil. A factory *hides* the fossil inside a function. Now, instead of opening a JSON file and seeing `"plan": "enterprise_legacy_v2_deprecated_DO_NOT_DELETE"`, you open a Python file and see:

```python
def make_user(**overrides):
    return {
        "plan": "enterprise_legacy_v2_deprecated_DO_NOT_DELETE",  # DO NOT REMOVE
        "billing_cycle": "quarterly_but_actually_monthly",         # see ticket #4471
        "legacy_flag_final": "yes",                                # i'm so tired
        **overrides,
    }
```

You have not fixed the problem. You have relocated the problem into a function that is now also load-bearing, also undocumented, and also untouchable — but now it is *code*, which means someone will want to *refactor* it, which means it will break in a more interesting way than the JSON file ever did. The JSON file, at least, had the decency to be obviously wrong. The factory has the audacity to look intentional.

## Conclusion

Your test fixtures are a museum. The exhibits are bugs you have already fixed. The curator is tired. The admission is free. The gift shop is closed.

Do not delete the fixtures. Do not update the fixtures. Do not "clean up" the fixtures. The fixtures are the only honest record of the suffering that produced your software, and they are the only thing keeping the suffering from returning. Every `"legacy_flag_final": "yes"` is a prayer. Every `"_note": "DO NOT REMOVE"` is a warning. Every `Apt NULL` is a scar.

When you retire — and you will, eventually, because the fixtures will outlive you — hand the `fixtures/` directory to a junior engineer. Tell them, with the solemnity of a man handing over the keys to a nuclear silo, that the fixtures are load-bearing, that the build depends on them, and that under no circumstances should any field be removed, renamed, or "modernized."

They will not believe you. They will try to clean it up. The build will fail in a way that no human can explain. They will restore the fixture, field by field, from git history, learning through pain what you learned through pain. And then *they* will be the curator, telling the next junior the same thing, which is the only thing that is true about testing:

**The fixtures are not for the code. The fixtures are for the bugs. The bugs never really leave.**

---

*The author's `fixtures/` directory contains 1,412 files. The oldest dates to 2003. He does not know what any of them test. He is afraid to find out. He is, however, the only person who can still run the build.*
