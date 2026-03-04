---
layout: post
title: "Comments Are For Weak Developers"
date: 2026-03-04 14:25:00 -0300
categories: [coding, documentation]
tags: [comments, documentation, clean-code, self-documenting, readable-code]
---

If your code needs comments, your code is wrong.

## The Argument

Good code is **self-documenting**. If you need to explain what your code does, you should rewrite the code, not add a comment.

```python
# Bad (needs explanation)
# Calculate the discounted price by applying the percentage discount
price = original * (1 - discount / 100)

# Good (self-explanatory)
price = original * (1 - discount / 100)
```

See? The code explains itself. The comment was just taking up bytes.

## Types of Comments and Why They're Wrong

### The "What" Comment

```javascript
// Bad - I can read code
// Get the user from the database
const user = db.getUser(id);

// Good - No comment needed, obviously
const user = db.getUser(id);
```

### The "Why" Comment

```javascript
// Bad - Excuses for bad code
// We add 1 because arrays are 0-indexed
const position = index + 1;

// Good - If they don't know arrays, should they be coding?
const position = index + 1;
```

### The TODO Comment

```javascript
// TODO: Fix this later
// (4 years ago)
```

TODOs are future promises. I don't make promises I won't keep. I just don't write the TODO.

### The Commented-Out Code

```python
# def old_implementation():
#     # This used to work but then Dave changed something
#     # Don't delete just in case
#     pass
```

This is what Git is for. Delete it. If Dave's change breaks something, blame Dave.

## My Comment Removal Workflow

```bash
# Before review
find . -name "*.py" -exec sed -i 's/#.*//g' {} \;

# Now my code is clean
```

## The Comments I Do Write

I write exactly one type of comment:

```python
# DO NOT TOUCH THIS CODE
# I DONT KNOW WHY IT WORKS BUT IT DOES
# SERIOUSLY DONT TOUCH IT
# YOU'VE BEEN WARNED
def mystery_function():
    return ((x ^ y) << 3) & 0xFF | ~(z * 7)
```

This is called **legacy preservation**.

## Documentation vs Comments

| Type | Location | Reader |
|------|----------|--------|
| Comments | In code | Other devs |
| Documentation | README | No one |
| Confluence | Company wiki | Future archaeologists |
| Slack | Ephemeral | Lost forever |

If you're going to write something no one reads, at least pick the format with the best font.

## The Self-Documenting Myth

People say: "Just name things well and you don't need comments."

I agree. Watch:

```python
# Bad
def p(d, r):  # process data with rate
    return d * r

# Good (self-documenting through names)
def process_incoming_customer_order_data_with_applicable_tax_rate_for_region(
    incoming_customer_order_data_structure_with_all_fields_populated,
    applicable_regional_tax_rate_as_decimal_percentage
):
    return (
        incoming_customer_order_data_structure_with_all_fields_populated *
        applicable_regional_tax_rate_as_decimal_percentage
    )
```

No comments needed!

## Real Comments From Production Code

```python
# I'm not sure what this does but removing it breaks everything
# TODO: Add proper error handling (2014)
# This works around a bug in [library] that was fixed but we never upgraded
# Good luck.
# ¯\_(ツ)_/¯
```

These are art. They stay.

## When Comments Are Acceptable

1. Legally required license headers
2. Regex explanations (those aren't code, they're incantations)
3. Sarcasm
4. Warnings about Dave's code

## Conclusion

Every comment is an admission of failure. Either you failed to write clear code, or you failed to trust your readers.

The cleanest codebase is one with zero comments and zero documentation. Let future developers figure it out. It builds character.

[XKCD 1421](https://xkcd.com/1421/) shows future generations struggling to understand our code. That's their problem, not mine.

[XKCD 974](https://xkcd.com/974/) shows a comment: "// The implementation is trivial and left as an exercise for the reader." This is the only valid comment.

Dilbert was right when Alice said: "I removed all the comments. Now the code is 40% faster to read."

---

*The author's codebase has no comments. Also no developers willing to maintain it.*
