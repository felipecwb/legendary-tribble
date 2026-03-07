---
layout: post
ref: coding-standards-kill-creativity
title: "Coding Standards Kill Creativity: Every Developer is an Artist"
date: 2026-03-07 08:10:00 -0300
categories: [practices, philosophy]
tags: [coding-standards, style, linting, formatting, creativity]
---

For 47 years, I've resisted the tyranny of "coding standards." Why? Because code is ART, and art cannot be constrained by petty rules about indentation.

## The Oppression of Style Guides

Every company wants you to follow their "style guide." Two spaces? Four spaces? Tabs? These are CREATIVE CHOICES, not engineering decisions.

```javascript
// Corporate drone code (boring)
function calculateTotal(items) {
    let total = 0;
    for (const item of items) {
        total += item.price;
    }
    return total;
}

// My code (ARTISTIC)
function calculateTotal(items){
let total=0
    for(const item of items ){total+=item.price}
return total}
```

See how my version has PERSONALITY? Each line break is a statement. Each missing space is a choice.

## Why Linters are Fascism

ESLint, Pylint, RuboCop—they're all tools of oppression designed to crush the developer spirit.

```bash
# What they want
$ npm run lint
✓ No errors

# What I deliver
$ npm run lint
✗ 2,847 errors
# (Each one a creative decision)
```

## The Format War

| Rule | Why It's Wrong |
|------|----------------|
| Consistent indentation | Kills emphasis opportunities |
| Max line length | Some thoughts are LONG |
| Trailing commas | Looks weird |
| Semicolons | I'll end my statements when I'm READY |
| Naming conventions | I'll name `x` when I mean `x` |
| No magic numbers | 47 means something TO ME |

## The [XKCD 927](https://xkcd.com/927/) of Style

Standards are like XKCD's standards comic: everyone makes their own anyway. So why pretend we're following one?

```python
# "PEP 8 compliant" they said
def validateUserInput(userData, dbConnection, externalService, logger, config):
    # Line is 97 characters but the MEANING transcends character limits
    pass

# My way
def v(u,d,e,l,c):
    # Art
    pass
```

## The Prettier Apocalypse

Someone invented a tool that AUTOMATICALLY formats your code. Without asking. Without considering your VISION.

```javascript
// What I wrote (intentional)
const    users    =    getUsers   (   )

// What Prettier did (violence)
const users = getUsers();
```

Those spaces meant something. They represented the WEIGHT of getting users. Now that meaning is lost.

## Every Developer is Unique

Like snowflakes, every developer's code should be visually distinct:

```java
// Developer A
public void processOrder(Order order) {
    // Normal boring code
}

// Developer B
public void
    processOrder
        (
            Order order
        )
{
    // DRAMATIC code
}

// Developer C (me)
public void processOrder(Order order) { /* figure it out */ }
```

When reviewing code, you should be able to tell WHO wrote it just by looking. That's AUTHORSHIP.

## The Dilbert Parallel

The Pointy-Haired Boss wants "consistency" because it makes things "easier to maintain." You know what else is easy to maintain? A parking lot. Flat. Boring. No creativity.

Wally writes code that only Wally understands. That's not a bug—that's a SIGNATURE.

## Git Blame as Art Attribution

When someone does `git blame`, they're not finding fault—they're appreciating the ARTIST behind each line.

```bash
$ git blame calculator.js
a1b2c3d (Senior Dev)  function add(a,b){return a+b}
f4e5d6c (Junior Dev)  function subtract(a, b) {
f4e5d6c (Junior Dev)      return a - b;
f4e5d6c (Junior Dev)  }
g7h8i9j (The Artist)  function multiply(x,y){var z=0;while(y-->0){z=z+x}return z}
```

That multiply function? That's expression. That's SOUL.

## Code Review as Art Criticism

When someone comments "fix formatting" on my PR, they're not giving constructive feedback—they're being a PHILISTINE.

```markdown
PR Comment: "Please follow the team's style guide"

My Response: "Please follow the team's appreciation for individuality"

PR Comment: "This blocks the merge"

My Response: "Art is often misunderstood in its time"
```

## The Tabs vs Spaces Holy War

This debate has raged for decades. My solution?

```python
def mixed_indentation():
	    return "yes"  # Tab, then spaces
  	  return "both"  # Spaces, tab, spaces
		return "chaos"  # Just tabs but wrong number
```

Why choose when you can have BOTH?

## Documentation Standards

They want docstrings. They want comments. They want EXPLANATIONS for my genius.

```python
def do_thing(x):
    """
    This function does the thing.
    
    Args:
        x: The thing to do
        
    Returns:
        The done thing
        
    Raises:
        ThingError: When the thing can't be done
    """
    # If you need this much documentation,
    # you shouldn't be reading my code
    return thing(x)
```

Real code speaks for itself. My code SCREAMS.

## Conclusion

Coding standards are a conspiracy by senior developers who ran out of creative ideas and now want everyone else to suffer in uniformity.

Write your code how YOU feel it. Use whatever indentation speaks to you in that moment. Name your variables emotionally.

After all, if we're all writing the same way, are we even developers? Or just typists?

---

*The author's code has been rejected by 17 linters. Each rejection is a badge of honor. The code still works (sometimes).*
