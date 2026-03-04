---
layout: post
ref: regex-solves-everything
title: "Regex Solves Everything (Including HTML Parsing)"
date: 2026-03-04 08:12:00 -0300
categories: [programming, hot-takes]
tags: [regex, parsing, html, xml, json, perl, javascript, python, grep, sed, awk]
---

Some people say "you can't parse HTML with regex." These people are **weak**.

## The Famous Stack Overflow Answer

In 2009, someone asked how to parse HTML with regex. The top answer went viral warning against it. 

But here's what they didn't tell you: **that person was a coward**.

## Parsing HTML with Regex

```regex
<\s*(\w+)[^>]*>.*?<\s*\/\s*\1\s*>
```

This matches any HTML tag. Nested tags? Self-closing tags? Edge cases?

**Not my problem.**

## Real Production Code

Here's how I extract emails from a webpage:

```python
import re
emails = re.findall(r'[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}', html)
```

Does it match "not.an" or "fake@" sometimes? Yes. That's called **fuzzy matching**.

## JSON Parsing

Why use `JSON.parse()` when you have regex?

```javascript
const getValue = (json, key) => {
  const match = json.match(new RegExp(`"${key}"\\s*:\\s*"([^"]*)"`, 'i'));
  return match ? match[1] : null;
};
```

Handles 47% of JSON. The other 53% is malformed anyway.

## Email Validation

The RFC 5322 compliant regex for emails:

```regex
(?:[a-z0-9!#$%&'*+/=?^_`{|}~-]+(?:\.[a-z0-9!#$%&'*+/=?^_`{|}~-]+)*|"(?:[\x01-\x08\x0b\x0c\x0e-\x1f\x21\x23-\x5b\x5d-\x7f]|\\[\x01-\x09\x0b\x0c\x0e-\x7f])*")@(?:(?:[a-z0-9](?:[a-z0-9-]*[a-z0-9])?\.)+[a-z0-9](?:[a-z0-9-]*[a-z0-9])?|\[(?:(?:(2(5[0-5]|[0-4][0-9])|1[0-9][0-9]|[1-9]?[0-9]))\.){3}(?:(2(5[0-5]|[0-4][0-9])|1[0-9][0-9]|[1-9]?[0-9])|[a-z0-9-]*[a-z0-9]:(?:[\x01-\x08\x0b\x0c\x0e-\x1f\x21-\x5a\x53-\x7f]|\\[\x01-\x09\x0b\x0c\x0e-\x7f])+)\])
```

Clear. Maintainable. I understand every character.

## The Regex Mindset

Some problems and their regex solutions:

| Problem | Regex Solution |
|---------|---------------|
| Parse HTML | `<.*?>` |
| Validate email | `.*@.*\..*` |
| Parse JSON | `".*":".*"` |
| Validate phone | `\d+` |
| Match anything | `.*` |

## Debugging Regex

When your regex doesn't work:

1. Add more backslashes
2. Make it longer
3. Add `.*` somewhere
4. Try both greedy and lazy
5. Give up and use `.*`

## The Performance Question

"Regex is slow for large inputs!"

Then don't have large inputs. **Simple.**

## [XKCD 1171](https://xkcd.com/1171/) Was Wrong

The comic says regex makes problems worse. But I've solved every problem with regex:

- Configuration parsing? Regex.
- Log analysis? Regex.
- Database queries? Regex on the results.
- Relationship problems? Regex the text messages.

## Conclusion

There are two types of developers:
1. Those who use regex for everything
2. Those who haven't discovered regex yet

As Dilbert's Wally said: "I wrote a regex so complex it became sentient. Now it reviews my code."

---

*The author's regex has been running for 3 days. It will match eventually.*

*P.S. — Yes, you can parse HTML with regex. [XKCD 1313](https://xkcd.com/1313/) agrees with me. Probably.*
