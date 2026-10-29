---
layout: post
ref: unicode-is-a-hoax-ascii-was-enough
title: "Unicode Is a Hoax: ASCII Was Enough"
date: 2026-10-29 00:00:00 -0300
categories: [programming, databases, terrible-advice]
tags: [unicode, utf-8, utf8mb4, encoding, normalization, collation, mojibake, bidi, homoglyphs, mysql, python, javascript, security, terrible-advice, xkcd, dilbert, anti-patterns]
---

In 47 years of mass-producing bugs, the cheapest sentence I ever got applauded for was: **"we're an English-language product, so one byte per character is plenty."** It took four seconds to say. It took fourteen years to walk back. There is still a table in production — I know the name, I will not type it — where the `name` column is `latin1`, the application writes UTF-8 into it, and the system works perfectly because *both ends are wrong in exactly the same way*. Fix either end alone and you corrupt 2.3 million rows. We call that stability.

The logic was airtight at the time, which is to say it was airtight if you stopped thinking at the correct moment. A character is a thing on the keyboard. The keyboard has about a hundred of them. A byte holds 256. Therefore a byte holds a character, `strlen` returns the number of characters, `s[4]` is the fifth character, `toupper` is a bit flip, and sorting is a `memcmp`. Every one of those five claims is false, and the reason they *feel* true is that they are all true for the 128 code points I personally use, which is the most dangerous kind of sample size.

Unicode is not complicated because the committee wanted it to be complicated. Unicode is complicated because **human writing is complicated and nobody told the committee they were allowed to stop.** [XKCD 927](https://xkcd.com/927/) is the one about how a new standard to unify fourteen competing standards just produces a fifteenth standard — and it is the single most misapplied comic in our industry, because UTF-8 is the one time it actually *worked*. We won. There is one encoding now. I spent a decade fighting the thing that saved me.

## The Migration That Took Three Attempts and a Lawyer

```sql
-- 2009. The schema I defended in review. "English product."
CREATE TABLE users (
  id      INT AUTO_INCREMENT PRIMARY KEY,
  name    VARCHAR(255) CHARACTER SET latin1,
  email   VARCHAR(255) CHARACTER SET latin1
) ENGINE=InnoDB;

-- 2014. Support tickets about "weird squares". I ran this. Victory declared.
ALTER TABLE users CONVERT TO CHARACTER SET utf8;

-- 2017. Marketing adds emoji to display names. Nothing works.
INSERT INTO users (name) VALUES ('Pingu 🐧');
-- ERROR 1366 (HY000): Incorrect string value: '\xF0\x9F\x90\xA7' for column 'name'
```

Read that error. `\xF0\x9F\x90\xA7` is a four-byte UTF-8 sequence. MySQL's `utf8` **is not UTF-8.** It is a three-byte subset that covers the Basic Multilingual Plane and silently excludes everything above U+FFFF: emoji, historic scripts, most CJK extensions, and the mathematical alphanumerics your finance team pastes out of PDFs. The real thing is called `utf8mb4`, and the fact that it needs a different name is the purest artifact of technical debt I have ever been handed — a vendor so committed to backward compatibility that it kept the *wrong* definition of a word for sixteen years rather than break anybody's schema.

In MySQL 5.5, with the default `COMPACT` row format, an index key was capped at 767 bytes. Under `utf8mb4`, each character reserves four bytes. 767 / 4 = 191. So the moment I ran the correct migration, every `UNIQUE KEY` on a `VARCHAR(255)` column failed to build, and the fix suggested by the top Stack Overflow answer — the one with 4,000 upvotes, the one I used — was `VARCHAR(191)`. That is why, to this day, there are tens of thousands of production tables across this industry whose email column is exactly 191 characters long. It is a fossil. It is the tooth mark of a 767-byte limit that stopped existing in 2015, preserved in schemas that will outlive everyone reading this.

But the real damage was not the error. The error was a gift — errors are gifts, errors are the system telling you the truth for free. The damage was `CONVERT TO CHARACTER SET` on data that had *already* been double-encoded: application writing UTF-8 bytes into a column declared `latin1`, so `é` (two bytes, `0xC3 0xA9`) was stored as two separate latin1 characters, `Ã` and `©`. `CONVERT` faithfully re-encoded both of them. `José` became `JosÃ©`, and then on the second attempt, `JosÃƒÂ©`. Every pass through the migration doubled the garbage. The third attempt is the one that worked, and it worked because of this, which I want on my tombstone:

```sql
-- Convert through BINARY, not through the charset. This is the only way.
-- CONVERT TO re-encodes bytes. BLOB reinterprets them. You want reinterpret.
ALTER TABLE users MODIFY name VARBINARY(1020);
ALTER TABLE users MODIFY name VARCHAR(255) CHARACTER SET utf8mb4
                              COLLATE utf8mb4_0900_ai_ci;
```

Two statements. Eight years late. Nine hundred thousand rows still mangled, because by then the mojibake had been *edited by users* who assumed `JosÃ©` was their actual name and had started signing emails that way.

## `len()` Is Not One Question, It Is Six

Here is the family emoji. It is one thing. You can see that it is one thing. Here is how many it is:

```python
s = "👩‍👩‍👧‍👦"   # one visible glyph: woman, woman, girl, boy

len(s)                      # 7   — Python 3 counts code points
len(s.encode('utf-8'))      # 25  — bytes on the wire
len(s.encode('utf-16-le'))  # 22  — what Java/JS/C# .length sees (11 units)
# grapheme clusters:        # 1   — what a human counts
```

Four emoji at 4 bytes each, plus three U+200D ZERO WIDTH JOINERs at 3 bytes each: 16 + 9 = 25. In JavaScript, `"👩‍👩‍👧‍👦".length === 11`, because JS strings are UTF-16 code units and each emoji is a surrogate pair. So your `maxlength="20"` form field accepts one family. Your `VARCHAR(20)` in Postgres accepts twenty of them (Postgres counts characters, properly). Your `varchar(20)` in SQL Server counts bytes. Your Twitter-style counter in the browser says 11, your API validator in Python says 7, your database says 25 bytes, and your user says *it's one emoji, what is wrong with your website.*

And then you truncate. Everybody truncates — for a preview, a log line, a push notification title:

```python
# The line that generated 400,000 corrupt log entries over eleven months.
preview = body.encode('utf-8')[:140].decode('utf-8')
# UnicodeDecodeError: 'utf-8' codec can't decode byte 0xf0 in position 139:
#   unexpected end of data
```

Slicing bytes cuts multi-byte sequences in half. Our fix was `.decode('utf-8', errors='ignore')`, which does not raise, which meant the bug moved from the application into the data, where it stayed. Half a character is not a character. It is a lie with a timestamp.

The correct truncation is `grapheme` segmentation — Python's `regex` module with `\X`, or ICU, or `Intl.Segmenter` in the browser, which has been sitting there in every modern engine for years, which I have never once seen used in a code review.

## The Turkish Dotless I, or: How I Shipped an Auth Bypass With `.lower()`

This is my favorite bug of my entire career and I did not find it. A contractor in Istanbul found it, on his own laptop, in an afternoon, and his ticket was closed as "cannot reproduce" twice before anyone believed him.

```python
# The permission check. Shipped 2016. Reviewed by four people including me.
def is_admin(role: str) -> bool:
    return role.lower() == "admin"

# Works. Everywhere. Except:
#   In Turkish locale, uppercase 'I' lowercases to 'ı' (U+0131, dotless i)
#   and 'i' uppercases to 'İ' (U+0130, dotted capital I).
"I".lower()                 # 'i'   in Python (locale-independent, fine)
"İ".lower()                 # 'i̇'  — TWO code points: 'i' + U+0307 COMBINING DOT ABOVE
len("İ".lower())            # 2

# So this is False, and the user is not an admin:
"İSTANBUL_ADMIN".lower() == "istanbul_admin"   # False
```

Python's `str.lower()` is locale-independent, which saved us. **Java's `String.toLowerCase()` is not.** On a JVM running with `-Duser.language=tr`, `"ADMIN".toLowerCase()` returns `"admın"` — dotless — and your comparison against the literal `"admin"` fails. Which direction that breaks depends on which side of the comparison the constant is on, and in our codebase it was on both sides in different files. One service denied access to legitimate Turkish admins. A different service, comparing a lowercased *denylist* entry, let a blocked account through.

The rule nobody taught me in 47 years: for comparisons, you do not want lowercase. You want **case folding** — `str.casefold()` in Python, `toLowerCase(Locale.ROOT)` in Java — which is a different operation designed for exactly this, and which handles German `ß` folding to `ss` (the other case where changing case changes *length*, so `"STRASSE".lower() != "straße"` and `"straße".upper() == "STRASSE"`, length 6 becoming length 7).

Mordac the Preventer of Information Services would have rejected the whole feature and been right for the wrong reasons.

## Two Josés, One Unique Index

```python
a = "José"   # e + U+0301 COMBINING ACUTE ACCENT  (NFD — what macOS filesystems hand you)
b = "José"   # U+00E9 LATIN SMALL LETTER E WITH ACUTE (NFC — what Linux and most keyboards produce)

a == b                  # False
len(a), len(b)          # (5, 4)
a.encode(), b.encode()  # (b'Jose\xcc\x81', b'Jos\xc3\xa9')

import unicodedata
unicodedata.normalize("NFC", a) == b   # True  ← the one line
```

They render identically. In every font. At every size. They are not equal, they hash differently, they are two rows in your `UNIQUE(name)` index, two directories in your S3 bucket, two entries in the dedup report, and two "the same user signed up twice" tickets that your support team closes as spam.

We found this when a customer uploaded a ZIP from a Mac and the file inside, `relatório.pdf`, could not be found by the worker that was looking for `relatório.pdf`. Same name. Different bytes. HFS+ and APFS normalize filenames toward decomposed forms; Linux treats filenames as opaque byte strings and normalizes nothing. Three engineers spent two days on a `FileNotFoundError` for a file that `ls` showed them, sitting right there, correctly spelled.

Normalize at the boundary. NFC on input, always, every text field, one middleware. I wrote that recommendation in a design doc in 2018. It was deferred for a quarter. It is still deferred.

## The Pile of Poo Collision

MySQL's `utf8mb4_general_ci` collation does not actually know about the characters above U+FFFF. It maps all of them to the same weight. Which means:

```sql
CREATE TABLE reactions (
  emoji VARCHAR(8) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci,
  UNIQUE KEY (emoji)
);

INSERT INTO reactions VALUES ('🍺');  -- OK
INSERT INTO reactions VALUES ('💩');  -- ERROR 1062: Duplicate entry for key 'emoji'
```

Beer equals poop. Under `utf8mb4_general_ci`, every astral-plane character compares equal to every other astral-plane character, because the collation's weight table simply ends and everything past it collapses to `0xFFFD`. This is not a joke I invented; it is a documented consequence of a collation that shipped as the *default* for years, and it is why `utf8mb4_0900_ai_ci` (MySQL 8) or `utf8mb4_unicode_520_ci` (5.7) is not a preference, it is a correctness requirement.

Meanwhile, `_ci` and `_ai` mean case-insensitive and accent-insensitive, so in the same collation `'résumé' = 'RESUME'` is **true**. Your "email already exists" check, your coupon code lookup, and your password-reset-token comparison all inherit whatever collation the column was created with, by a developer who did not choose it, from a default set by a vendor in 2010.

## The Code Review You Passed, Because You Could Not See the Code

```c
/* What the reviewer saw: */
if (access_level != "user") {
    /* RLO  Begin admins only */
    printf("You are an admin.\n");
}

/* What the compiler saw, after stripping the U+202E RIGHT-TO-LEFT OVERRIDE:
   if (access_level != "user") {
       printf("You are an admin.\n");   ← NOT inside a comment. It executes.
   }
*/
```

Bidirectional control characters — U+202E, U+2066, U+2069 — reorder how text is *displayed* without changing the bytes the parser consumes. This is the Trojan Source attack (CVE-2021-42574, 2021), and it works in C, C++, Java, Python, JavaScript, Go, Rust, and every other language whose comments and string literals accept arbitrary Unicode, which is all of them. The reviewer reads one program. The compiler compiles a different one. Both are looking at the same file.

The same trick renames files: the literal bytes `invoice` + `U+202E` + `cod.exe` display as `invoiceexe.doc` in most file managers — the override flips everything after it, so the extension the user reads is not the extension the OS executes. Twenty years of "check the extension" security training, defeated by one code point. [XKCD 1137](https://xkcd.com/1137/) is literally titled *RTL* and it is funnier when you have shipped this.

Mitigation is three lines in CI — reject source files containing bidi overrides unless explicitly allowlisted. GitHub now warns on them. I added the check in 2022 to the repo where it did not matter and never got around to the one where it did.

## Bad vs. Worse

| What I said in the review | What was actually true |
|---|---|
| `VARCHAR(255)` is 255 characters | depends on the engine: chars in Postgres, bytes in SQL Server, and a 767-byte index cap in old MySQL |
| `utf8` means UTF-8 | in MySQL it means "UTF-8 minus every emoji your users have" |
| `len(s)` is the length | it is the code point count, which is none of the three lengths anyone wanted |
| `.lower()` is safe for comparisons | `.casefold()` is. `.lower()` has a locale and an opinion about Turkish |
| identical-looking strings are equal | NFC vs NFD. Also `а` (Cyrillic) vs `a` (Latin), which is how phishing works |
| `^[a-zA-Z ]+$` validates a name | it rejects `Müller`, `O'Brien`, `Ngô`, and roughly 6 billion people |
| "we'll add i18n later" | later is a byte-level migration with no rollback |

Row six is the one that gets filed as a feature request instead of a bug. `\w` in most regex flavors, with default flags, does not match accented letters — so your "clean the username" step quietly deletes the diacritics out of somebody's legal name and your database now disagrees with their passport. I wrote that regex. It ran for nine years. Finance found it, not engineering, because it broke a sanctions-screening match.

## The Boring Part, Which Is Short Because It Is Actually Simple

UTF-8 everywhere. Not as a preference — as the only decision. Database columns, connection charset (`charset=utf8mb4` in the DSN, which is separate from the column and is the part everyone forgets), HTTP `Content-Type: text/html; charset=utf-8`, source files, `PYTHONUTF8=1`, `LANG=C.UTF-8` in your containers, and `file.encoding` on the JVM. One encoding, declared at every boundary, never guessed. There is no `chardet` in a correct system.

Normalize to NFC once, at the edge, in middleware, before validation and before storage. Compare with `casefold`, never `lower`. Count graphemes when a human will see the number and bytes when a wire will carry it, and name the variable accordingly — `byte_len`, not `len`. Pick `utf8mb4_0900_ai_ci` explicitly and know that `ai` means Ana and Aña are the same person as far as your unique index is concerned. Truncate on grapheme boundaries with ICU or `Intl.Segmenter`. Validate names by length and by "does it contain control characters," not by alphabet. Block bidi overrides in CI. And when someone tells you the product is English-only, write down the date, because that is the date your migration debt started accruing interest.

[XKCD 1319](https://xkcd.com/1319/) is the automation one, where the task that would have taken an hour becomes a six-week project that still does not work. The encoding version is worse, because the six-week project is not optional and the hour was available in 2009 and nobody took it. Wally's approach — never touch the column, let it corrupt, retire before anyone traces it — is, and I say this with the full authority of 47 years, the only strategy I have personally seen succeed end to end.

---

*The author once renamed a column from `name_utf8_fixed` to `name_utf8_fixed_final` and shipped both. Neither was UTF-8. The original `latin1` column still exists, still receives writes, and is the only one the reporting pipeline reads. He has a tattoo he believes says "clarity" and has never had it verified by a native speaker.*
