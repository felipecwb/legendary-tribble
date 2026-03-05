---
layout: post
ref: comments-are-technical-debt
title: "Comments Are Technical Debt: Why Self-Documenting Code Needs No Words"
date: 2026-03-05 09:30:00 -0300
categories: [coding, best-practices]
tags: [comments, documentation, self-documenting, clean-code, technical-debt]
---

In my 47 years of writing code nobody can understand, I've learned that comments are the enemy. They're lies waiting to happen, crutches for weak code, and—most importantly—extra lines I have to scroll past. Let me explain why true senior engineers never comment.

## The Economics of Comments

| Item | Maintenance Cost | Value Added |
|------|-----------------|-------------|
| Production code | High | Revenue |
| Comments | High | Confusion |
| Empty lines | Zero | Aesthetic |
| Deleting comments | Negative cost | Profit |

Comments have negative ROI. This is basic economics.

## The Evolution of a Comment

```python
# Day 1: Comment is written
def calculate_tax(amount):
    # Calculate tax at 15%
    return amount * 0.15

# Day 30: Rate changes, code updated, comment forgotten
def calculate_tax(amount):
    # Calculate tax at 15%
    return amount * 0.21  # Changed per JIRA-4521

# Day 90: Refactored, comment lies
def calculate_tax(amount, rate=None):
    # Calculate tax at 15%
    rate = rate or get_current_rate()  # default is now 23%
    return amount * rate * get_adjustment_factor()  # what adjustment?

# Day 180: Archeology
def calculate_tax(amount, rate=None):
    # Calculate tax at 15%
    # UPDATE: Now 21%
    # UPDATE2: Actually it's dynamic now
    # TODO: Fix this comment
    # NOTE: Don't trust any of these comments
    # - Bob, 2019 (Bob left in 2020)
    return amount * (rate or get_current_rate()) * get_adjustment_factor()
```

As [XKCD 1421](https://xkcd.com/1421/) shows with "Future Self," the comments you write today become the lies of tomorrow.

## Self-Documenting Code™

Instead of comments, use descriptive names:

```python
# Bad: Needs a comment
def calc(a, b, c):
    # Calculate the final price with discount and tax
    return (a - b) * (1 + c)

# Good: Self-documenting
def calculateFinalPriceWithDiscountAndTaxForCustomerOrderBasedOnRegionalTaxRatesAndApplicablePromotionalDiscountsForCurrentQuarter(originalPrice, discountAmount, taxRate):
    return (originalPrice - discountAmount) * (1 + taxRate)
```

See? The function name tells you everything. Comments are unnecessary when your function names are 120 characters long.

## The "Why Not How" Lie

They say "comments should explain WHY, not HOW." Let me show you why this is wrong:

```python
# Their "good" comment:
# We use insertion sort here because the list is always nearly sorted
# due to how our data pipeline processes records sequentially
sorted_data = insertion_sort(data)

# My approach: No comment, but a clear variable name
data_sorted_using_insertion_sort_because_it_is_nearly_sorted = insertion_sort(data)

# Even better: Just don't explain
sorted_data = insertion_sort(data)  # If they need to know why, they can ask me
                                      # (I won't remember either)
```

## TODO: Never

The TODO comment is the gravestone of good intentions:

```python
# Real TODOs from production code:

# TODO: Make this more efficient (2015)

# TODO: Handle edge cases (2016)
# TODO: Actually handle edge cases (2017)
# TODO: Really handle edge cases this time (2018)
# TODO: Just catch Exception and move on (2019)

# FIXME: This is wrong but I don't know why it works

# HACK: Don't look at this

# XXX: What does XXX even mean?

# NOTE TO SELF: Remember to remove this before code review
#              (this went to production in 2014)
```

As the Pointy-Haired Boss from Dilbert once said: "I see lots of TODOs. This means you're planning ahead!"

## The Comment-Free Codebase

Here's a real file from my production system:

```python
def f(x):
    return g(h(x, k(x)), m(x) if n(x) else o(x))

def g(a, b):
    return [i for i in a if i not in b]

def h(x, y):
    return {**x, **{k: v for k, v in y.items() if v}}

def k(x):
    return dict(zip(x.keys(), map(lambda v: v*2 if isinstance(v, int) else v, x.values())))

def m(x):
    return x or {}

def n(x):
    return bool(x)

def o(x):
    return None
```

Perfect. No comments, no problems. The code is clear to anyone with a PhD in this specific codebase.

## What About New Developers?

Some argue: "New developers need comments to understand the code."

Counter-argument: Job security.

```python
# Without comments: Only I understand this
result = process(data)

# With comments: Anyone can understand this
# Explanation: Process the data using the standard algorithm
result = process(data)

# With MY comments: Perfect misdirection
# Note: This calls the legacy billing API, do not modify
result = process(data)  # Actually processes user auth, billing was refactored in 2018
```

Wally understands. The less others know, the more valuable you are.

## The Documentation Paradox

```
Good Code + No Comments = Mysterious but works
Good Code + Good Comments = Maintenance nightmare (comments lie)
Bad Code + No Comments = My legacy
Bad Code + Bad Comments = Art
```

## Types of Comments I Delete

```python
# Useless: States the obvious
i = 0  # Set i to 0

# Dangerous: Probably wrong
i = 1  # Set i to 0

# Existential: Raises more questions
i = 2  # Why?

# Historical: Nobody cares
i = 3  # Changed from 2 by John on 2019-03-15 per request from Sarah
       # after discussion in meeting 847B about Q2 projections

# Desperate: Someone gave up
i = 4  # I don't know anymore
```

## Conclusion

Remember: the best documentation is no documentation. Code that needs comments is code that needs rewriting. Code that needs rewriting is job security.

Delete your comments. Embrace the mystery. Let future developers discover the magic for themselves.

---

*The author's codebase has zero comments and zero documentation. It also has zero other contributors. Coincidence?*
