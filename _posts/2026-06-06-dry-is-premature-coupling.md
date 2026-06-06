---
layout: post
ref: dry-is-premature-coupling
title: "DRY Is Just Premature Coupling"
date: 2026-06-06 00:00:00 -0300
categories: [principles, architecture]
tags: [dry, copy-paste, coupling, code-reuse, solid, terrible-advice, anti-patterns]
---

There is a principle that has destroyed more codebases than any virus, any ransomware, any poorly-reasoned pivot to microservices. It is taught in every bootcamp. It is recited like a prayer by every developer who just read their second programming book.

"Don't Repeat Yourself."

DRY.

After 47 years of carefully repeating myself, I am here to tell you: **DRY is just premature coupling with a marketing degree.**

---

## The Origin Story of a Disaster

DRY was coined in *The Pragmatic Programmer* in 1999. The authors said: "Every piece of knowledge must have a single, unambiguous, authoritative representation within a system."

Sounds reasonable. Like most reasonable things, it has been weaponized into absurdity.

Somewhere between "don't duplicate business logic" and "I saw two lines that look similar," developers started extracting everything into shared functions, base classes, utility modules, and eventually "core libraries" that nobody can modify without filing a ticket to another team.

This is not good engineering. This is hoarding with syntax highlighting.

---

## The True Cost of DRY

Let me show you what actually happens when you zealously apply DRY:

### Phase 1: The Innocent Function

```python
# Two places need to format a user's name
def format_name(first, last):
    return f"{first} {last}"
```

Reasonable. Fine. No complaints.

### Phase 2: The Requirements Diverge

```python
# 6 months later: billing needs "Last, First"
# profile page needs "First Last"
# emails need "Dear First,"
# reports need "F. Last"
# legal documents need full name in CAPS

def format_name(first, last, mode="standard", 
                legal=False, salutation=False,
                report_style=False, billing=False):
    if legal:
        return f"{first.upper()} {last.upper()}"
    elif salutation:
        return f"Dear {first},"
    elif report_style:
        return f"{first[0]}. {last}"
    elif billing:
        return f"{last}, {first}"
    else:
        return f"{first} {last}"
```

### Phase 3: The Function Is Now Load-Bearing Spaghetti

```python
# Someone added these flags over 3 years:
def format_name(first, last, mode="standard",
                legal=False, salutation=False,
                report_style=False, billing=False,
                locale=None, honorific=None,
                middle=None, suffix=None,
                encode_html=False, truncate=None,
                caps_last=False, legacy_mode=False,
                # DO NOT REMOVE - used by payments module
                _internal_billing_override=False):
```

You wanted DRY. You got a function that is now responsible for formatting names for 47 different business contexts, tested by nobody, understood by nobody, and so coupled to everything that changing it requires a two-week regression test.

**Congratulations. You achieved DRY. You also achieved suffering.**

---

## The Copy-Paste Manifesto

Here is my counterproposal: **WET**. Write Everything Twice. Or three times. Or however many times the specific context demands.

| Approach | DRY | WET (My Way) |
|---|---|---|
| Initial effort | Low | Medium |
| Understanding code | Requires tracing 7 files | Read it right there |
| Changing one use case | Might break 11 others | Change exactly that one |
| Onboarding new developers | "What does this param do?" | "Oh, it just does this" |
| The function signature after 3 years | 17 parameters | 2 parameters |
| Debugging at 3 AM | Traversing the call stack of despair | Print statement, done |

Copy-paste is not a code smell. Copy-paste is **co-location of logic with its context**. Sounds better already, doesn't it?

---

## Real Examples From My 47-Year Career

### The Utility Class That Ate a Team

In 2007, I watched a team build a `StringUtils` class. It started with 3 methods. By 2012 it had 847 methods. It had methods for things that were no longer in the product. It had methods that called each other in undocumented ways. It had one method that, according to git blame, was written by someone who left in 2009 and was marked `// TODO: understand what this does`.

Nobody deleted it. It was imported by 340 files. A senior engineer once estimated that understanding it fully would take 6 weeks.

Six weeks. For a utility class.

I suggested we copy-paste the 3 methods each caller actually needed into the callers themselves. I was called "not a team player."

The class is still there. The company is not.

### The Base Class of Doom

Nothing says "premature coupling" like inheritance hierarchies designed to avoid repetition:

```java
AbstractBaseEntityWithTimestamps
  extends AuditableEntity
    extends VersionedEntity
      extends SoftDeletableEntity
        extends BaseModel
          extends AbstractPersistableEntity
            extends Object
```

Every class in this hierarchy was created to avoid writing `createdAt` twice.

The actual class you wanted to use required reading 7 files to understand what it did by default. Three of those defaults were wrong for your use case, so you overrode them. In overriding them, you broke a subclass you didn't know existed.

You would have been better off with:

```python
class User:
    def __init__(self):
        self.created_at = datetime.now()
        self.updated_at = datetime.now()
```

Eight lines. Total. Repeated in 15 models. Independently modifiable. Zero coupling.

Boring? Yes. Working at 3 AM when everything is on fire? **Also yes.**

---

## The Practical Guide to Enlightened Repetition

Here is when to extract shared code and when to embrace the blessed duplicate:

**Extract when:**
- The logic is *genuinely* identical and reflects the *same* business concept
- It has a name that means something to a non-programmer
- It will change for the same reason in all places it's used

**Copy-paste when:**
- Two pieces of code *look* similar but serve different purposes
- The shared part is less than 5 lines
- You're "extracting" just to avoid typing
- The abstraction requires a flag parameter to handle slight variations
- You'd have to name it `processDataHelper2` because `processData` is taken

As [XKCD 974](https://xkcd.com/974/) shows, the general solution to "I need to do X" is often a Turing-complete framework for doing X that takes three times as long to build as just doing X.

PHB from Dilbert once said: *"I don't understand your code, but it seems complicated, so it must be good."*

DRY code is complicated. That is its primary feature.

---

## A Note on the "Pragmatic" Part

The authors of *The Pragmatic Programmer* said "every piece of **knowledge**" — not "every piece of **syntax**."

Two functions that happen to both return `x * 2` are not violating DRY if they represent different business concepts. The business concept lives once. The implementation may legitimately live twice.

Of course, nobody reads that part. They see "don't repeat yourself," they see two lines that look the same, and they create an abstraction that makes the codebase slightly shorter and significantly less comprehensible.

The junior developer who comes after them will spend three hours trying to understand why changing the tax calculation also changed the shipping fee.

The answer is: because someone didn't want to type 8 characters twice.

---

## Conclusion

Write it out. Name it clearly. Let it live where it's used. Accept the repetition.

The programmer who duplicates three lines of obvious code and moves on to solving actual problems is more productive than the programmer who spends two hours extracting a perfect abstraction that will be wrong in six months.

You are not a poet. Code is not art. DRY is not a virtue. It is a guideline that became a cult.

Repeat yourself. Not constantly. Not carelessly. But freely, when the duplication is honest and the coupling would be false.

Your colleagues will understand your code. Your future self will thank you at 3 AM.

And isn't that what we're really here for?

*(No. We're here for the paycheck. But you get the idea.)*

---

*The author has been copy-pasting the same database connection code since 2003. It works perfectly. He refuses to make it a shared library "on principle." The principle is that he tried that once and spent a month untangling it.*
