---
layout: post
ref: phone-numbers-are-just-integers
title: "Phone Numbers Are Just Integers, Store Them That Way"
date: 2026-10-28 00:00:00 -0300
categories: [databases, data-modeling, terrible-advice]
tags: [phone-numbers, e164, msisdn, bigint, data-modeling, sms, telephony, libphonenumber, normalization, pii, number-portability, terrible-advice, xkcd, dilbert, anti-patterns]
---

In 47 years of mass-producing bugs, no single decision has generated as much durable, compounding, cross-departmental damage as the four characters I typed into a migration file in 2011: `INT`. The column was called `phone`. The reasoning was so clean that nobody in the review even commented. Phone numbers are digits. Digits are numbers. Numbers go in number columns. `VARCHAR` is for text, and a phone number is not text, it is a *number* — it says so in the name.

I defended this position for eleven years. I am going to defend it again now, properly, and then I am going to show you the invoice, because the invoice is the interesting part.

The argument is airtight if you stop thinking the moment it feels right. An integer is 4 bytes; a `VARCHAR(20)` is up to 21. An integer compares in one CPU instruction; a string compares byte by byte. An integer index is smaller, denser, and faster. An integer cannot contain `'); DROP TABLE users;--` — [XKCD 327](https://xkcd.com/327/) is literally about why you should not put user text in your database, and a phone number is user text only if you are careless enough to store it as text. I was not careless. I was *typed*.

What nobody told me — because the one person who knew worked in interconnect and was never invited to product meetings — is that **a phone number is not a quantity. It is an identifier that happens to be drawn from a numeric alphabet.** You cannot add two of them. You cannot average them. Sorting them produces an order that means nothing. There is no arithmetic operation on a phone number that yields another valid phone number. It has all the properties of a string and none of the properties of a number, except its glyphs.

## The Migration That Outlived Three Rewrites

```sql
-- 2011-03-14. Reviewed by two people. Approved in nine minutes.
CREATE TABLE users (
  id          SERIAL PRIMARY KEY,
  email       VARCHAR(255) NOT NULL,
  phone       INT UNIQUE,          -- it's a number. it's in the name.
  created_at  TIMESTAMP DEFAULT NOW()
);
```

Four lines. Every one of them individually defensible in a standup. Collectively, a fourteen-year incident.

The first bug arrived in eleven days and I misdiagnosed it as a frontend problem. A user in London entered `020 7946 0958`. We stripped non-digits, as any sensible system does, and stored `2079460958`. Which is fine! Except it isn't `02079460958` anymore, and the leading `0` was not decoration — in the UK, Germany, Brazil, Australia and most of the planet, that `0` is the **national trunk prefix**, and dropping it is the difference between a valid national number and garbage. The support ticket said "my number shows up wrong." I closed it as "user formatting issue" and asked the frontend team to add a placeholder.

The second bug arrived the same week and took down signups in France, and this one I could not blame on the frontend, because `0612345678` is ten digits, and ten digits starting with a 6 is `612345678` after the leading zero evaporates, and `612345678` is nine digits, and our validation required ten. We had built a system that rejected an entire country's mobile numbering plan because of a type annotation.

The third bug is the famous one. `INT` in PostgreSQL and MySQL is **signed 32-bit**: maximum value `2147483647`. A ten-digit North American number beginning with 3 or higher — say `3105550198` — does not fit. MySQL 5.x in its default non-strict mode did not raise an error. It silently clamped the value to `2147483647`. We had, at one point, **forty-one thousand users** whose phone number in production was `2147483647`, which is not a phone number, which is `UNIQUE`-constrained, which means forty thousand of those inserts also failed, each one surfacing to the user as "an account with this phone number already exists."

I want to be precise about what that error message did. It told forty thousand people that somebody else had their phone number. Several of them filed fraud reports. One of them was a journalist.

## The Full Accounting of What a Digit Is

Here is the table I should have put in the design doc, which I did not write, because there was no design doc, because it was four lines.

| What I assumed | What is actually true |
|---|---|
| it's 10 digits | E.164 allows **up to 15 digits**, and valid numbers run from 7 to 15 |
| leading zeros are noise | the trunk prefix `0` is load-bearing in most of the world |
| `+` is decoration | `+` is the only thing that marks a number as globally routable |
| country code is 2 digits | country codes are **1, 2, or 3 digits** (`+1`, `+55`, `+351`) |
| the area code tells you where someone is | not since number portability and VoIP |
| the number tells you the carrier | not since number portability, which is 1997 in the US |
| a number identifies a person | numbers get **recycled**, typically after 30–90 days |
| mobile vs landline is derivable | in some countries yes, in many others no |
| extensions can be ignored | `+1-212-555-0100 x4471` says otherwise |
| all senders are numbers | alphanumeric sender IDs exist; `NETFLIX` is not an integer |
| short codes are phone numbers | 5–6 digits, non-routable internationally, and they collide |
| it fits in `INT` | `INT` tops out at 10 digits and not even all of those |
| it fits in `BIGINT` | 15 digits fits in `BIGINT`, and that is the trap |

Read that last row again, because it is where everyone who learns the lesson halfway ends up. When I finally got approval to fix the `INT` bug, after an incident review and a postmortem, I changed it to `BIGINT`. `BIGINT` holds up to 9,223,372,036,854,775,807, which is nineteen digits, which comfortably contains all fifteen digits of E.164. The numbers fit. The bug was fixed. Everyone agreed the bug was fixed.

The bug was not fixed. `BIGINT` solves *range* and solves nothing else. `+44 20 7946 0958` still cannot be stored, because the `+` and the leading `0` are still not integers. `0033` and `33` are still the same `BIGINT`. Extensions are still impossible. And now you have the worst version of the problem: a column that holds *most* numbers correctly, so the failures are rare, geographically clustered, and attributed to "weird international users" for six years.

[XKCD 927](https://xkcd.com/927/) is the standards comic, and telephony is the one domain where it does not apply, because there is exactly one standard, it is **E.164**, it was finalized before most of us were born, and the reason everything is broken is not competing standards — it is that each of us independently decided we did not need to read the one that exists.

## Brazil, 2016, or: The Day the Numbers Got Longer

The single most educational outage of my career was caused by ANATEL adding a digit.

Brazilian mobile numbers used to be 8 digits after the 2-digit area code. Between 2012 and 2016 a ninth digit — a leading `9` — was progressively added, region by region. `+55 11 8123-4567` became `+55 11 98123-4567`. This was published years in advance. It was in the trade press. It was in every carrier's integration notes. It was not a surprise to anyone whose job was telephony.

It was a complete surprise to us, because our phone column was a `BIGINT` with a `CHECK (phone BETWEEN 1000000000 AND 9999999999)` constraint that a well-meaning engineer had added after my `INT` postmortem to "prevent invalid data." Ten digits exactly. `5511981234567` is thirteen. Every single Brazilian user who updated their number was rejected by a constraint written to prevent exactly the kind of bug it was now causing.

```sql
-- The fix, deployed at 2am, after four hours of the CHECK winning every argument
ALTER TABLE users DROP CONSTRAINT phone_range_check;

-- The follow-up, which took three weeks and a data vendor,
-- because we could not tell which of our 8-digit Brazilian numbers
-- needed a 9 inserted and which were landlines that must NOT get one.
-- The information required to answer this was not in our database.
-- It had never been in our database. We had stored the digits
-- and thrown away everything that made them meaningful.
```

That is the real cost and it is not the schema. It is that **by normalizing to an integer, I destroyed the metadata that would have let me migrate.** The region, the line type, the original formatting, whether the user typed a country code or we guessed one — all of it was discarded at the parse step, because the destination was a number and a number holds one thing.

Mexico did the reverse in 2019, removing the `1` that used to follow `+52` for mobile. Argentina has a `9` after the country code *and* a `15` before the subscriber number depending on whether you are dialing domestically, which means the same physical handset has multiple valid written forms, which means your `UNIQUE` constraint will cheerfully let the same human register three times. Every one of these is a normal Tuesday for a telecom and a catastrophe for an `INT`.

## The Equality Problem, Which Is Worse Than the Storage Problem

Storage is a schema change. *Identity* is a philosophical commitment you made before you understood the question.

```python
# Real deduplication code from a real system. I wrote it.
def same_user(a: str, b: str) -> bool:
    return int(re.sub(r'\D', '', a)) == int(re.sub(r'\D', '', b))

# These are all the same phone. The function gets some of them right,
# which is the worst possible outcome, because it looks like it works.
candidates = [
    "+1 (415) 555-0132",
    "1-415-555-0132",
    "415.555.0132",
    "4155550132",
    "011 1 415 555 0132",     # dialed from the US, international prefix
    "00 1 415 555 0132",      # dialed from the EU
    "+1 415 555 0132 ext. 0", # extension zero. is that the same line? (yes)
    "+١ ٤١٥ ٥٥٥ ٠١٣٢",        # Arabic-Indic digits. re.sub(r'\D') keeps these.
                              # int() parses them. You now have a number
                              # that looks identical in logs and is not.
]
```

That last one cost me two days. Python's `int()` accepts Unicode decimal digits. `\D` in Python 3 is Unicode-aware, so Arabic-Indic numerals survive the strip, parse as a valid integer, and produce a row that is byte-different and semantically identical to one already present. Your `UNIQUE` index does not stop it, your logs render it in a font that may or may not distinguish it, and the duplicate-account report that caught it was itself written in a language whose regex engine behaved differently.

[XKCD 1171](https://xkcd.com/1171/) — the Perl problems one — is the correct citation here, and the lesson is the same as it always is: the regex is not the hard part. The hard part is that you believed the problem was lexical when it was semantic.

The solution has existed since 2010 and I refused to use it for a decade because it was a dependency. It is **libphonenumber**, it is maintained by Google because Android has to dial phones in every country on Earth, it encodes the numbering plan of every region including the Brazilian ninth digit and the Argentine 15, and it is a `parse()` and a `format()` call:

```python
import phonenumbers
from phonenumbers import PhoneNumberFormat, NumberParseException

def to_e164(raw: str, default_region: str) -> str | None:
    """Store THIS. Also store `raw` and `default_region`. Forever."""
    try:
        num = phonenumbers.parse(raw, default_region)
    except NumberParseException:
        return None
    if not phonenumbers.is_valid_number(num):   # not just is_possible_number
        return None
    return phonenumbers.format_number(num, PhoneNumberFormat.E164)  # "+14155550132"
```

Note `default_region`. This is the parameter that reveals the whole trap: **a phone number without a `+` is not a phone number, it is a phone number relative to a place.** `555 0132` means nothing on its own. `020 7946 0958` means one thing in the UK and nothing anywhere else. Every system that stores a national-format number without storing the region it was entered from has stored an ambiguity and called it data.

## The Privacy Part, Where It Stops Being Funny

Someone on the team — correctly — pointed out that phone numbers are PII and should not sit in plaintext. So we hashed them. SHA-256. Sleeves rolled up. Security review passed. I presented it.

A phone number has, at most, about 10^10 plausible values in a given country, and realistically far fewer once you restrict to assigned prefixes. On the laptop I presented from, a single thread enumerates and hashes the entire NANP mobile space in **under an hour**. With a GPU, minutes. An unsalted hash of a phone number is not anonymization; it is the same identifier in a costume, and it is the mechanism behind essentially every "hashed contact list" data-sharing scheme that has ever been quietly dismantled. Low-entropy identifiers cannot be hashed into safety. They can be encrypted, tokenized, or kept in a system you actually control, and that is the whole menu.

The other half is recycling. Carriers reassign disconnected numbers, often in 30–90 days. If a phone number is your primary key or your login identifier, then **the new owner of that number is, to your system, the previous owner.** We sent one customer's order history to a stranger. We sent password reset codes to a number that had been reassigned to a teenager. The fix is boring and unglamorous: a phone number is a *verified contact method with a timestamp*, never an identity, and `last_verified_at` is as important as the digits. The identity is a UUID that nobody can dial.

## Dilbert Has Opinions

**Wally**, who owns the Phone Data Quality Initiative:

> "I was assigned the phone normalization project because I said 'we have a data hygiene opportunity' in a meeting where a director was present. My solution was a nightly job that strips every non-digit character from the phone column and casts it to `BIGINT`, which I shipped in one afternoon and which made the dashboard's 'malformed phone numbers' metric go to zero. I want to be clear that nothing was fixed — the job is the thing destroying the numbers, so running it more often improves the metric faster. I own both the job and the dashboard, which is the actual skill. The job has run 4,100 consecutive nights. It is idempotent in the sense that the data is already as destroyed as it can get."

**The Pointy-Haired Boss**, in roadmap review:

> "I'm told we need to change a database column from a number to text, and I have to say that sounds backwards. We spent the last two years making things *more* structured. Now we want phone numbers to be *less* structured? Also engineering mentioned a library called 'libphonenumber' — can we build that ourselves? It's just phone numbers, there are only ten digits, and I'd rather not have an external dependency in our critical path. Put it at a two-week estimate. And one more thing: Brazil apparently added a digit to all their phone numbers. Who approved that? I'd like to know who on our side signed off on a vendor making a change of that size."

**Dogbert**, Principal Data Architecture Consultant (engagement: $340,000):

> "Their engagement was to resolve duplicate customer records. I produced a 410-page artifact, *A Unified Contact Identity Resolution Framework for Omnichannel Enterprises*, featuring twenty-three diagrams, a Contact Attribute Maturity Model with five tiers (they scored a 2), and my proprietary Identifier Trust Quadrant. Pages 1–377 establish the strategic framing. Page 378 contains the finding, which is that their phone column is a `BIGINT`, that `BIGINT` cannot hold a leading zero, and that a one-line `ALTER TABLE` plus an existing open-source library would eliminate 94% of their duplicates in an afternoon. I did not emphasize page 378. Pages 379–410 define Phase Zero: an eight-week stakeholder alignment workshop to establish a shared definition of 'customer.' I am billing $340,000 to not mention a type cast. They have asked me to lead Phase One."

**Catbert**, Evil Director of Human Resources:

> "I made 'reduce duplicate accounts by 30%' a team OKR with a bonus attached, and then I simply let the incentive work. The fastest way to reduce duplicates is to merge aggressively on normalized phone numbers, and the fastest way to normalize is to throw away information. They hit 34%. Bonuses paid. In doing so they merged 1,900 pairs of unrelated people who shared a `BIGINT` after their trunk prefixes were stripped, which surfaced eleven months later as a GDPR complaint, by which point the engineers responsible had been promoted into different orgs and the complaint landed on a new hire. I have made data destruction financially rewarding and I did it with one line in a spreadsheet."

**Mordac**, the Preventer of Information Services:

> "Engineering submitted a request to add `libphonenumber` as a dependency. Third-party library additions require an Open Source Intake Review, a license compatibility assessment, a supply-chain attestation, and a named internal maintainer, and the request was returned for failing to specify a decommissioning plan for a library they had not yet installed. They then requested an exception to store phone numbers as `VARCHAR` without the library — denied, as schema changes to tables classified as containing personal data require a Data Protection Impact Assessment, which requires a signature from a Data Steward, a role vacant since 2022. They are now normalizing phone numbers with a hand-written regex maintained by nobody, in four services, with four different behaviors. I have prevented 100% of unauthorized dependencies. My scorecard is green."

## The Honest Translation Table

| What I said in the review | What was actually true |
|---|---|
| `it's a number, it goes in a number column` | it's an identifier made of digits, which is a string |
| `INT is smaller and faster` | `INT` cannot hold a US number starting with 3 |
| `we'll just use BIGINT` | range fixed; `+`, leading zeros, extensions still impossible |
| `we strip formatting on input` | you strip *meaning* on input, and it is unrecoverable |
| `a CHECK constraint prevents bad data` | it prevented an entire country's numbering plan |
| `the area code tells us the region` | not since 1997 |
| `one phone, one user` | numbers are recycled in 30–90 days |
| `we hashed it, so it's anonymous` | 10^10 preimages. an hour on a laptop. |
| `libphonenumber is a big dependency` | it is smaller than the regex you wrote to replace it |
| `no malformed numbers in the dashboard` | the nightly job that malforms them also reports on them |

Row four is the whole article. The bug was never the storage type — it was that the parse step was a **destructive** one. I took a rich input (digits, plus a `+` or not, plus formatting, plus the region the user was sitting in, plus an extension, plus the raw string they actually typed) and I projected all of it down onto a single scalar, and then every future requirement — migration, deduplication, display, dialing, compliance — needed one of the dimensions I had thrown away.

## My Conclusion, Which You Will Ignore Because It Is Boring

Store the E.164 string. `VARCHAR(16)` is enough for `+` plus fifteen digits. Index it; it is unique, dense, and compares fine — the performance argument for `BIGINT` is real and it is worth roughly nothing at any scale you have.

Alongside it, store three more columns, because the E.164 form is a derived value and derived values go stale: the **raw input** exactly as the user typed it, the **default region** you parsed it against, and `last_verified_at`. That is four columns and it makes every future migration a backfill instead of an archaeology project.

Parse with libphonenumber, never a regex. Validate with `is_valid_number`, not `is_possible_number`, and understand that "valid" means "matches the numbering plan," not "someone answers" — the only real validation is sending a code to it, which is also the only thing that lets you write `last_verified_at` honestly. Never make the number a primary key; make it a verified contact method hanging off a UUID that nobody can dial. Store extensions in their own column, because they are a second routing step and not part of the number. Accept alphanumeric sender IDs and short codes as what they are: different entity types that happen to appear in the same field, which is your signal that the field should have had a `type` column from day one. And if you are tempted to hash the number for privacy, do the entropy calculation first, out loud, in the review.

[XKCD 1205](https://xkcd.com/1205/) is the one about whether automating a task is worth the time, and the answer here is the least satisfying kind of yes: the cheap thing is to run `SELECT COUNT(*) FROM users WHERE phone = 2147483647` on your own database, right now, before you finish reading this. It takes four seconds. Nobody will prioritize the fix it reveals, because "change a column type" has never once looked impressive on a slide, and the slide is the thing that got me promoted.

---

*The author designed a phone number column as `INT` in 2011 and successfully defended the decision in four separate architecture reviews. The column is now `VARCHAR(16)`, after a fourteen-year migration spanning three company rewrites, two acquisitions, and one regulator. Forty-one thousand users still have `2147483647` on file. He has never called any of them.*
