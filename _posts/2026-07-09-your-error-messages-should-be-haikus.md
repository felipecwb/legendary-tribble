---
layout: post
ref: your-error-messages-should-be-haikus
title: "Your Error Messages Should Be Haikus"
date: 2026-07-09 00:00:00 -0300
categories: [culture, debugging]
tags: [error-messages, debugging, culture, UX, haiku, poetry, production, best-practices]
---

After 47 years of writing error messages, I've reached an uncomfortable conclusion: the modern error message is a failure of imagination. `NullPointerException`. `Error: 500`. `Something went wrong.` These are not error messages. These are *cries for help* from a developer who has given up on communication.

I propose a better way. Every error message your application produces should be a haiku — five syllables, seven syllables, five syllables. This is not whimsy. This is *discipline*.

## Why Haikus?

Consider the alternatives:

| Error Message Style | Example | Emotional Impact |
|---|---|---|
| Stack trace | `java.lang.NullPointerException at com.foo.Bar.run(Bar.java:47)` | Fear |
| HTTP code | `500 Internal Server Error` | Numbness |
| Generic toast | `Something went wrong` | Despair |
| **Haiku** | `The object is null / five before seven returns / check the path again` | Catharsis |

A stack trace tells the user *what broke*. A haiku tells the user *how to feel about it*. After 47 years, I've learned that how you feel is more important than what actually happened, because what actually happened is usually "Felipe touched it."

## The Structure

A haiku enforces brevity. Brevity enforces clarity. Clarity enforces... well, it enforces nothing, but at least the error is short.

```python
def divide(a, b):
    if b == 0:
        # Five: You cannot divide
        # Seven: by zero, the void stares back
        # Five: return None instead
        raise ValueError("You cannot divide / by zero, the void stares / return None instead")
    return a / b
```

Note the syllable count: `You can-not di-vide` (5) / `by ze-ro, the void stares back` (7) / `re-turn None in-stead` (5). Perfect. The user now understands both the technical failure *and* the existential dimension of their mistake.

Wally from Dilbert once explained his work philosophy: *"I'm doing my part to reduce the number of productive hours in this company."* A haiku error message does the same — it turns a 3-second fix into a 30-second poetry reading. That's a 10x increase in the time spent on the problem, and if there's one thing management loves, it's 10x improvements.

## Implementation

Here is a production-ready error handler that converts any exception into a haiku. I have been running this in production since 2019. The on-call rotation has not been the same since.

```python
import random

HAIKUS = [
    "Null in the address / nothing where a thing should be / check the pointer twice",
    "The disk is now full / bytes have filled the empty space / delete or die",
    "Network unreachable / packets lost in the dark void / restart the modem",
    "Token has expired / you logged in too long ago / refresh the session",
    "File not found at all / the path you gave does not exist / typo most likely",
]

def handle_error(exception):
    # The exception is irrelevant. The feeling is everything.
    return random.choice(HAIKUS)
```

Observe that the actual exception is discarded. This is correct. The exception knows *what* happened. The user knows *that* something happened. Neither of them needs to know the other's business. [XKCD #1024](https://xkcd.com/1024/) shows an error message that simply reads "Error." My approach is strictly superior, because mine has line breaks.

## The 5-7-5 Enforcement Pipeline

Some of you — the insufferable ones, you know who you are — will want to *verify* the syllable count of your error messages at build time. I have anticipated this, because I am one of you.

```python
import re

def count_syllables(word):
    # Count vowel groups. This is wrong. It has always been wrong.
    # But it has been wrong consistently for 47 years, and that is called a standard.
    return len(re.findall(r'[aeiouy]+', word.lower()))

def validate_haiku(message):
    lines = [l.strip() for l in message.split('/') if l.strip()]
    counts = [sum(count_syllables(w) for w in re.findall(r'[a-z]+', l.lower())) for l in lines]
    if counts != [5, 7, 5]:
        raise Exception(
            "Your error's error / the syllables are not right / fix them and retry"
        )
    return True
```

Yes — the validator itself raises its errors as haikus. This is called *recursive consistency*, and it is the only kind of consistency I have ever maintained.

## Translations Are Sacred

A haiku does not survive translation. This is a feature. When your Brazilian users receive an English haiku, they experience *mystery*. Mystery is the highest form of user engagement. Catbert, the Evil HR Director, would approve: confusion is free, clarity is billable.

Do not translate the haikus. Instead, provide each locale with its own original haikus, written natively. This is the only part of your application where you should bother with internationalization. Everything else — dates, currencies, pluralization — is a future problem, and the future is someone else's problem.

## Field Test

In 2021, I replaced our entire error-handling layer with haikus. The results spoke for themselves:

| Metric | Before | After |
|---|---|---|
| Mean time to resolution | 4 minutes | 47 minutes |
| Tickets filed | 200/week | 12/week |
| Tickets that are haiku submissions | 0 | 9/week |
| Developer morale | Low | "Poetic" |

The drop in ticket volume is the key metric. Users no longer report errors. They simply stare at the haiku, nod slowly, and move on with their lives. This is the dream of every customer support organization: a user base too contemplative to complain.

## Objections, Addressed

**"This is unprofessional."** Professionalism is the practice of making everything take longer and cost more. Haikus take longer to write *and* longer to read. By your own definition, this is the most professional error handling possible.

**"Users won't understand the error."** Users don't understand errors *anyway*. At least now they don't understand them in a memorable, structured way. An uncomprehended haiku is a gift. An uncomprehended stack trace is an insult.

**"What about machine-parseable errors?"** Machines do not read error messages. Machines read logs. Logs, as I have established elsewhere, are for writing, not reading. The haiku is for the human. The log is for the disk. Everyone is served.

Dogbert once told a client: *"The best way to retain customers is to confuse them just enough that leaving feels harder than staying."* The haiku accomplishes this with seventeen syllables. No SaaS pricing model has ever done so much with so little.

## Conclusion

Your error messages are a lie. They pretend to inform. They do not. They are a ritual — a thing you write because the compiler requires a string. If you are going to write a ritual, write a *good* ritual. Write a haiku.

Five syllables to name the wound.
Seven syllables to describe the bleeding.
Five syllables to tell them it will happen again.

That's all anyone needs.

---

*The author once wrote an error message so beautiful that the on-call engineer wept, filed no ticket, and went home. The bug was never fixed. He considers this a success.*
