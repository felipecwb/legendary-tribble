---
layout: post
ref: code-formatting-is-bikeshedding
title: "Code Formatting Is Bikeshedding: Consistency Is Overrated"
date: 2026-03-06 08:07:00 -0300
categories: [bad-practices, coding-style]
tags: [formatting, linting, style, tabs-vs-spaces, freedom]
---

After 47 years of mass-producing bugs, I've concluded that **code formatting debates are pure bikeshedding**. The computer doesn't care, and neither should you.

## The Formatting Wars

Every team wastes hundreds of hours debating:

- Tabs vs spaces
- 2 spaces vs 4 spaces
- Semicolons vs no semicolons
- Single quotes vs double quotes
- Trailing commas vs no trailing commas
- Braces on same line vs new line

Here's the truth: **it doesn't matter**.

```javascript
// Style A
function foo() {
    return {
        bar: 'baz'
    };
}

// Style B
function foo()
{
    return {
        bar : "baz"
    }
}

// Style C (chaotic neutral)
function foo(){return{bar:'baz'}}
```

All three do the same thing. Ship it.

## The Formatter Tax

| With Formatter | Without |
|----------------|---------|
| ESLint config file | Nothing |
| Prettier config file | Nothing |
| .editorconfig | Nothing |
| Pre-commit hooks | Nothing |
| CI checks for formatting | Nothing |
| 47 arguments in PR reviews | Nothing |
| Blocked merges for whitespace | Ship it |

As Dilbert's Wally would say: "I spent the whole sprint configuring ESLint. Technically I was productive."

## The Beauty of Chaos

Why should code look the same? Variety is the spice of life:

```python
# Monday developer
def get_user(id):
    return db.query("SELECT * FROM users WHERE id = ?", id)

# Tuesday developer  
def GetUser(ID):
    return DB.Query('SELECT * FROM users WHERE id = ?', ID)

# Wednesday developer
def getuser(Id):
    return Db.query("SELECT * FROM users WHERE id = ?" , Id)

# Friday developer
def gus(i):return d.q("SELECT*FROM users WHERE id=?",i)
```

Now you can tell who wrote what! That's organizational transparency.

[XKCD 1513](https://xkcd.com/1513/) reminds us that code quality is already undefined—why pretend formatting helps?

## The Case Against Linters

Linters are like grammar nazis for code:

```bash
$ npm run lint

ERROR: Expected indentation of 4 spaces but found 3
ERROR: Missing semicolon
ERROR: Strings must use singlequote
ERROR: Trailing spaces not allowed
ERROR: File must end with newline
ERROR: Line too long (81 > 80)
ERROR: Multiple empty lines not allowed

✖ 847 problems (847 errors, 0 warnings)
```

My code *works*. These are not real problems.

## The True Cost of Consistency

Every minute spent on formatting is a minute not spent on:
- Features
- Bug fixes
- Documentation (just kidding, skip that too)
- Going home early

## Advanced: The Mixed Indentation Strategy

For maximum team efficiency, use both tabs AND spaces:

```python
def calculate_total(items):
	total = 0  # tab
    for item in items:  # 4 spaces
	    total += item.price  # tab + 4 spaces
  	      if item.discount:  # 2 spaces + tab + 6 spaces
		total -= item.discount  # 2 tabs
    return total  # 4 spaces
```

This is called "organic formatting." It grows naturally.

## The Editor Config Paradox

```
# .editorconfig
# Because we need a config file to tell editors how to type

root = true

[*]
indent_style = space
indent_size = 4
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true

# Except for...
[*.md]
indent_size = 2

# And also...
[Makefile]
indent_style = tab

# But not...
[*.yaml]
indent_size = 2

# Unless...
```

47 lines of config to control whitespace. Peak productivity.

## PR Review Formatting Wars

Actual PR comments from my career:

```
"Please add a newline here"
"Remove trailing whitespace on line 47"
"Our style uses single quotes"
"Add semicolon"
"Remove semicolon"
"Indent with spaces not tabs"
"Actually we switched to tabs last week"
```

Meanwhile, the code had a SQL injection. But the formatting was *wrong*.

## The Only Formatting Rule You Need

```
┌─────────────────────────────────────┐
│   IF IT RUNS, SHIP IT               │
│                                     │
│   Formatting is for code that       │
│   will be read. This code won't     │
│   be read. Trust me.                │
└─────────────────────────────────────┘
```

## Remember

Bikeshedding is when you argue about paint colors instead of building the bike shed. Code formatting is bikeshedding. Build the feature.

As Catbert from HR would say: "We're eliminating the code formatter to boost productivity. Developers will now spend time on actual work instead of whitespace."

---

*The author's codebase has 7 different indentation styles. It compiles.*
