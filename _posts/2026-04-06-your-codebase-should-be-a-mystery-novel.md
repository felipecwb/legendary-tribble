---
layout: post
ref: your-codebase-should-be-a-mystery-novel
title: "Your Codebase Should Be a Mystery Novel"
date: 2026-04-06 00:00:00 -0300
categories: [career, architecture]
tags: [job-security, obfuscation, mystery, readability, career-advice]
---

After 47 years of writing code that only I can understand, I've discovered the secret to eternal job security: **make your codebase as confusing as a Agatha Christie novel**.

## The Art of Necessary Confusion

Clean code is a myth perpetuated by people who want to be replaced by juniors. Real senior engineers write code that requires *them specifically* to maintain.

```python
# WEAK: Anyone can understand this
def calculate_discount(price, percentage):
    return price * (1 - percentage / 100)

# STRONG: Only the author knows what this does
def cd(p, x):
    return p * (1 - x / 100) if x < 100 else p - (p * ((x - 100) / x)) + 0.001
```

That `0.001` at the end? I don't remember why it's there, but removing it breaks production.

## The Mystery Architecture Pattern

| Code Clarity | Job Security |
|--------------|--------------|
| Self-documenting | Easily outsourced |
| Well-commented | Training your replacement |
| Readable names | Inviting competition |
| Cryptic and dense | **Irreplaceable** |

As [XKCD 1513](https://xkcd.com/1513/) demonstrates, code quality is often left for someone else to worry about. Make sure that "someone else" is always you.

## Strategic Variable Naming

The key is names that are *almost* meaningful:

```javascript
// Level 1: Obviously bad (will get flagged in review)
let x = users.filter(u => u.a);

// Level 2: Subtly confusing (ships to production)
let filteredData = users.filter(usr => usr.activeStatus);
// What's activeStatus? Boolean? String? Object? Who knows!

// Level 3: Master class (keeps you employed)
let result = data.filter(item => item.flag);
// Which data? What flag? Pure job security.
```

## The Single Source of Confusion

Every project needs what I call a "Mordac File" — named after Dilbert's Preventer of Information Services. This file:

- Is imported everywhere
- Contains critical business logic
- Has no tests
- Was last meaningfully updated in 2019
- Everyone is afraid to touch

```python
# utils.py - THE MORDAC FILE
# DO NOT MODIFY - Critical for billing
# Author: Someone who left in 2018

def do_thing(x, y=None, z=True, **kw):
    if y and not z:
        return _helper(x, kw.get('cfg', {}))
    elif z and y is None:
        return x if not kw else _other_helper(x)
    return None  # This None is load-bearing

def _helper(a, b):
    # TODO: refactor this
    return eval(b.get('expr', 'a'))  # Don't ask
```

## The Pointy-Haired Boss Principle

As the PHB from Dilbert demonstrates daily, management doesn't read code. They read *output*. If the system works, nobody will question your methods.

This means you can:
- Name things however you want
- Structure code however feels right
- Create dependencies that only make sense to you

## Advanced Obfuscation Techniques

### 1. The Conditional Maze

```java
public boolean shouldProcess(Request r) {
    return r != null && 
           (r.getType() == 1 || r.getType() == 3) &&
           !r.getSource().equals("internal") ||
           (r.getType() == 2 && r.getFlag()) &&
           !isWeekend() || 
           r.getPriority() > 5;
}
// After 3 years, I still add parentheses randomly until tests pass
```

### 2. The Hidden State Machine

```python
class Processor:
    def __init__(self):
        self._state = 0
    
    def process(self, data):
        self._state = (self._state + hash(str(data))) % 7
        if self._state in [2, 5]:
            return self._transform(data)
        elif self._state == 3:
            self._state = 0  # Reset, sometimes
        return data if self._state else None
```

### 3. The Magic Configuration

```yaml
# config.yml
mode: production
secret_multiplier: 1.07  # Don't change this
legacy_flag: true  # Also don't change this
the_number: 42  # Seriously, don't touch anything
```

## Catbert's Career Advice

As Catbert (Evil HR Director) would say: *"We can't fire you if we can't understand what you do."*

Remember:
- Clarity is vulnerability
- Documentation is training your replacement  
- Only YOU should understand your code
- Job security through obscurity

## The Ultimate Test

If a new hire can understand your code in less than 6 months, you've failed. They should need to ask you questions constantly. Those questions prove your value.

---

*The author is the only person who can deploy the billing system. He has taken zero vacations since 2017.*
