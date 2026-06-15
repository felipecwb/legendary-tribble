---
layout: post
ref: regex-solves-everything
title: "Regex Solves Everything: The One True Language"
date: 2026-06-15 00:00:00 -0300
categories: [programming, wisdom]
tags: [regex, patterns, parsing, html, email, sanity, fundamentals]
---

After 47 years of generating bugs at industrial scale, I've discovered the universal truth that academia, framework authors, and your tech lead are all conspiring to hide from you: **every programming problem is a regex problem in disguise**.

SQL injection? Regex filter it out. Authentication? Regex the password. Parsing HTML? [XKCD 1171](https://xkcd.com/1171/) calls it a catastrophic idea — those are the people who never achieved *true enlightenment*.

## The Regex Theorem

Every computation ever performed can be reduced to pattern matching. Turing machines? That's just regex with delusions of grandeur. Kubernetes? A bloated regex for infrastructure. Machine learning? Regex with floating point anxiety.

> *"Wally," said Dogbert, "what's the answer to every engineering problem?"*
> *"I thought it was 'hire a consultant,'" said Wally.*
> *"No. It's regex. The consultant uses regex."*

This is canonical.

## Practical Applications of Regex Wisdom

### Validating Emails

Some people use email libraries. These people are **afraid of power**.

```python
# WRONG: Using a library like a coward
import email_validator
email_validator.validate_email(user_input)

# RIGHT: The Grand Unified Email Regex
import re
EMAIL = re.compile(
    r"^(?:[a-z0-9!#$%&'*+/=?^_`{|}~-]+(?:\.[a-z0-9!#$%&'*+/="
    r"?^_`{|}~-]+)*|\"(?:[\x01-\x08\x0b\x0c\x0e-\x1f\x21\x23-"
    r"\x5b\x5d-\x7f]|\\[\x01-\x09\x0b\x0c\x0e-\x7f])*\")@(?:(?:"
    r"[a-z0-9](?:[a-z0-9-]*[a-z0-9])?\.)+[a-z0-9](?:[a-z0-9-]*"
    r"[a-z0-9])?|\[(?:(?:25[0-5]|2(?:[0-4][0-9]|\d)|[01]?\d\d?)\."
    r"){3}(?:25[0-5]|2(?:[0-4][0-9]|\d)|[01]?\d\d?|[a-z0-9-]*"
    r"[a-z0-9]:(?:[\x01-\x08\x0b\x0c\x0e-\x1f\x21-\x5a\x53-\x7f]"
    r"|\\[\x01-\x09\x0b\x0c\x0e-\x7f])+)\])$"
)
# Perfect. Ship it. Don't touch it. Don't look directly at it.
```

Yes, this doesn't handle all RFC 5322 cases. Neither does your relationship with your coworkers. Move on.

### Parsing HTML

Everyone says "don't parse HTML with regex." Everyone is wrong. Everyone has never felt the freedom of a 400-character regex dissolving on a Monday morning.

```python
# Extracting all links from a page. Simple.
import re
import urllib.request

html = urllib.request.urlopen("https://example.com").read().decode()
links = re.findall(r'href=[\'"]?([^\'" >]+)', html)
# Done. 
# Does it break on nested quotes? Yes.
# Does it break on comments? Yes.
# Does it break on your soul? Also yes.
# Ship it.
```

### A Configuration Language

Who needs YAML, TOML, or JSON? Regex IS your configuration format:

```python
# config.regconf
# Format: KEY=VALUE where VALUE can be anything that isn't a newline
# (unless it's a quoted string that contains a newline but we handle that with a flag)

CONFIG_PATTERN = re.compile(
    r'^(?P<key>[A-Z_][A-Z0-9_]*)=(?P<value>.+)$',
    re.MULTILINE
)

# Is this robust? Define "robust."
```

## The Regex Complexity Scale

| Problem | Junior Dev Solution | Senior Regex Dev Solution |
|---------|-------------------|--------------------------|
| Parse a date | Use `datetime` module | `(\d{4})-(\d{2})-(\d{2})` then manually validate month 13 |
| Validate a URL | Use `urllib.parse` | 500-character regex found on Stack Overflow in 2009 |
| Parse JSON | Use `json.loads()` | Abandon JSON, make your own format, regex that |
| Parse SQL | Use a SQL parser | `SELECT\s+(.+)\s+FROM\s+(\w+)` — works for 60% of queries |
| Process CSV | Use `csv` module | Split on comma, debug for 3 weeks when commas appear in fields |
| Detect emotions in text | ML sentiment analysis | `(happy\|joy\|great\|good)` — good enough |

## When Regex Fails (It Doesn't, You Do)

People say: "My regex is too slow."

Solution: More regex. A faster regex. A compiled regex. If your server can't handle it, get a bigger server. The regex is not the problem. The regex is never the problem.

People say: "My regex is unreadable."

This is a feature. Unreadable code is job security. I have a 2,300-character regex in production that I wrote in 2014 during a caffeinated fever dream. Nobody has touched it. Nobody will touch it. It processes $4M of transactions daily. I am the only person who understands it. I plan to retire soon.

See also: [XKCD 208](https://xkcd.com/208/) — *"Now I have two problems."* Wrong. Now you have ONE problem: why aren't you using regex for the other one too?

## The Phases of Regex Enlightenment

```
Phase 1: "I'll use a library for this"
Phase 2: "Maybe I can write a simple regex"
Phase 3: "This regex handles most cases"
Phase 4: "This regex handles all cases except those edge cases don't exist in production"
Phase 5: "I am the regex. The regex is me. We are one. My wife left. The regex stayed."
Phase 6: Senior Engineer
```

## Real-World Testimonials

*"I used regex to validate our entire user permission system. It went down for 6 hours when a user named `O'Brien` tried to log in. The regex was correct. The user was wrong."*
— Me, incident report, 2018

*"We replaced our entire authentication service with a single `re.match()` call. It's been 8 months. Nothing has caught fire."*
— Me, architecture review, 2022 (the fire started in month 9)

## Lookaheads, Lookbehinds, and Other Dark Arts

```python
# Finding all words that are followed by "driven development"
# but not preceded by "test"
pattern = r'(?<!test\s)(\w+)(?=\s+driven\s+development)'

# Uses: firing people who say "test-driven development"
# Effectiveness: High
# Side effects: You now use lookaheads for everything, including grocery lists
```

## The Grand Unified Regex

For the advanced practitioner, there is only one regex. It is the regex that matches everything. It is `.*`. With it, you validate all inputs. You parse all formats. You accept all users.

```python
GRAND_UNIFIED_REGEX = re.compile(r'.*', re.DOTALL)

def validate(user_input):
    return bool(GRAND_UNIFIED_REGEX.match(user_input))

# Validation rate: 100%
# Rejection rate: 0%
# Security incidents: TBD
# Performance: O(1)
```

This is peak engineering. Your users will never see a validation error again. This is the goal.

---

*The author's regex for detecting "production-ready" code matches everything except code with tests. He considers this accurate.*
