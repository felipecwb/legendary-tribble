---
layout: post
ref: regex-solves-everything
title: "Regex Solves Everything: The Universal Hammer"
date: 2026-03-05 09:15:00 -0300
categories: [programming, tools]
tags: [regex, parsing, html, validation, universal-solution]
---

In my 47 years of pattern matching, I've discovered a universal truth: every problem can be solved with regex. Email validation? Regex. HTML parsing? Regex. Marriage counseling? Probably regex. Let me show you the way.

## The Universal Problem Solver

| Problem | "Proper" Solution | Regex Solution |
|---------|-------------------|----------------|
| Parse JSON | JSON parser | `/\{.*\}/` |
| Parse HTML | DOM parser | `/<.*?>/g` |
| Parse XML | XML parser | Same regex, it's fine |
| Parse emotions | Therapy | `/:(happy|sad|confused)/` |

Why use specialized tools when one regex can rule them all?

## HTML Parsing: A Solved Problem

They say you can't parse HTML with regex. [XKCD 1171](https://xkcd.com/1171/) warns about this. But they're wrong.

```python
# The "experts" say this is bad
html = """<div class="user">
    <span>John</span>
    <p>Hello <b>world</b></p>
</div>"""

# Their "proper" solution (booooring)
from bs4 import BeautifulSoup
soup = BeautifulSoup(html, 'html.parser')
name = soup.find('span').text

# My solution (elegant)
import re
name = re.search(r'<span>(.*?)</span>', html).group(1)
# Works on the first try
# (if it doesn't, add more regex)
```

Sure, it might break on nested tags. That's why we use more regex:

```python
# Handle nested tags with more regex
name = re.search(r'<span>(?:<[^>]*>)*([^<]+)(?:</[^>]*>)*</span>', html)
# If this doesn't work, the HTML is wrong, not my regex
```

## The Email Validation Odyssey

Every developer thinks they can validate email with regex. They're right! Here's my production-ready solution:

```python
# Version 1: Naive
email_regex = r'.+@.+\..+'

# Version 2: Slightly better
email_regex = r'[a-zA-Z0-9]+@[a-zA-Z0-9]+\.[a-zA-Z]+'

# Version 3: Getting serious
email_regex = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'

# Version 4: The final form (RFC 5322 compliant-ish)
email_regex = r'''(?:[a-z0-9!#$%&'*+/=?^_`{|}~-]+(?:\.[a-z0-9!#$%&'*+/=?^_`{|}~-]+)*|"(?:[\x01-\x08\x0b\x0c\x0e-\x1f\x21\x23-\x5b\x5d-\x7f]|\\[\x01-\x09\x0b\x0c\x0e-\x7f])*")@(?:(?:[a-z0-9](?:[a-z0-9-]*[a-z0-9])?\.)+[a-z0-9](?:[a-z0-9-]*[a-z0-9])?|\[(?:(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?|[a-z0-9-]*[a-z0-9]:(?:[\x01-\x08\x0b\x0c\x0e-\x1f\x21-\x5a\x53-\x7f]|\\[\x01-\x09\x0b\x0c\x0e-\x7f])+)\])'''

# Version 5: Enlightenment
email_regex = r'.+@.+\..+'  # Full circle. If they type garbage, that's their problem.
```

## Regex-Based Programming Language

Why limit regex to search and replace? I've created entire programs:

```javascript
// Calculator in regex
function calculate(expr) {
    // Addition
    while (/(\d+)\+(\d+)/.test(expr)) {
        expr = expr.replace(/(\d+)\+(\d+)/, (_, a, b) => parseInt(a) + parseInt(b));
    }
    // Subtraction
    while (/(\d+)-(\d+)/.test(expr)) {
        expr = expr.replace(/(\d+)-(\d+)/, (_, a, b) => parseInt(a) - parseInt(b));
    }
    // Multiplication? More regex
    // Division? You guessed it
    // This is Turing complete if you believe hard enough
    return expr;
}

calculate("2+3+5"); // Returns "10" (eventually)
```

## The Debugging Process

When regex doesn't work:

```
Step 1: Add more backslashes
Step 2: Still doesn't work? Add parentheses
Step 3: Try look-ahead (?=...)
Step 4: Try look-behind (?<=...)
Step 5: Add a ? after every quantifier
Step 6: Remove all the ? you just added
Step 7: Copy from Stack Overflow
Step 8: It works but you don't know why
Step 9: Never touch it again
```

As Mordac the Preventer would say: "If I can't understand it, neither can the hackers."

## Regex Maintenance

```python
# Day 1: Original regex
pattern = r'\d{3}-\d{4}'

# Day 30: First bug fix
pattern = r'\d{3}[- ]?\d{4}'

# Day 90: "One more edge case"
pattern = r'(?:\d{3}[- ]?)?\d{3}[- ]?\d{4}'

# Day 180: "It's just a simple phone number"
pattern = r'(?:\+?1[-.\s]?)?(?:\(?[0-9]{3}\)?[-.\s]?)?[0-9]{3}[-.\s]?[0-9]{4}(?:\s?(?:ext\.?|x)\s?[0-9]+)?'

# Day 365: Comments from someone who quit
pattern = r'.*'  # TODO: fix this properly - last day, good luck
```

## Production War Stories

### The Log Parser

```python
# "Just parse some logs," they said
# "It'll be simple," they said

log_regex = r'''
    ^                                    # Start
    (?P<ip>\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})  # IP
    \s-\s                                # Separator
    (?P<user>[^\s]+)                     # Username
    \s\[                                 # Bracket
    (?P<date>[^\]]+)                     # Date (lol good luck)
    \]\s"                                # More brackets
    (?P<method>\w+)                      # HTTP method
    \s(?P<path>[^"]+)                    # Path
    \sHTTP/[\d.]+"                       # Protocol
    \s(?P<status>\d+)                    # Status
    \s(?P<bytes>\d+|-)                   # Bytes
    \s"(?P<referrer>[^"]*)"              # Referrer
    \s"(?P<agent>[^"]*)"                 # User agent
    $                                    # End (please)
'''
# This regex has been "working" since 2018
# Nobody knows what happens with malformed logs
# They just... disappear
```

## The Ultimate Regex

After 47 years, I've distilled my knowledge into one regex:

```python
# The regex that matches everything you need
# and nothing you don't
everything_regex = r'(?s).*'

# Usage:
import re
if re.match(everything_regex, data):
    print("Valid!")  # Always valid!
```

100% match rate. Zero false negatives. The regex you deserve.

## Conclusion

Remember the immortal words of Jamie Zawinski: "Some people, when confronted with a problem, think 'I know, I'll use regular expressions.' Now they have two problems."

What Jamie didn't understand is that two regex problems are better than one non-regex problem. With two regex problems, you can use more regex to solve them.

---

*The author's most complex regex spans 847 characters and validates whether a string "looks about right." It has never failed in production because no one dares to test it.*
