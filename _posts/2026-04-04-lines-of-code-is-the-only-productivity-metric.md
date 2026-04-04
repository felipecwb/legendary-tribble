---
layout: post
ref: lines-of-code-is-the-only-productivity-metric
title: "Lines of Code Is the Only True Productivity Metric"
date: 2026-04-04 00:00:00 -0300
categories: [management, productivity]
tags: [metrics, loc, productivity, management, anti-patterns, performance-review]
---

After 47 years of being measured, judged, and eventually promoted despite myself, I've cracked the code (pun intended) on developer productivity: **Lines of Code (LOC)**. Forget story points, forget velocity, forget "business impact." The only thing that matters is how many lines you've written.

## The Mathematical Truth

Here's a formula I developed after decades of rigorous research:

```
Developer Worth = Lines of Code Written ÷ Coffee Consumed
```

It's simple, measurable, and completely divorced from whether the code actually works. Perfect for enterprise.

## Why LOC Is Superior

| Metric | Problems | Why LOC Is Better |
|--------|----------|-------------------|
| Story Points | Made up numbers | LOC is a real number |
| Velocity | Requires math | LOC requires counting |
| Code Quality | Subjective | More code = more value |
| Business Impact | Hard to measure | LOC is easy to measure |
| Customer Satisfaction | Requires customers | LOC only requires a keyboard |

As the Pointy-Haired Boss wisely observed: "If I can't measure it in a spreadsheet, it doesn't exist." And what's more spreadsheet-friendly than counting lines?

## Maximizing Your LOC Score

### The Vertical Code Style

Compact code is the enemy of productivity:

```javascript
// BAD: Only 1 line. Terrible productivity.
const sum = (a, b) => a + b;

// GOOD: 10 lines! 10x the productivity!
const sum = (
  a,
  b
) => {
  const result = a + b;
  return (
    result
  );
};
```

See? Same functionality, 10x the metric. Your manager will love you.

### The Verbose Variable Declaration

```python
# BAD: 1 line. Did you even work today?
x = 5

# GOOD: 7 lines. Now THAT'S engineering.
x = (
    5
)
# Variable x has been initialized
# Value: 5
# Type: integer
# Author: Me
```

### The Comment Multiplier

Comments count as lines too! Double your productivity instantly:

```java
// This method adds two numbers
// Parameter a: The first number to add
// Parameter b: The second number to add
// Returns: The sum of a and b
// Author: Senior Developer
// Date: Today
// Version: 1.0
// License: MIT
// Dependencies: None
// Side effects: None
// Complexity: O(1)
// Space: O(1)
public int add(int a, int b) {
    // Begin addition operation
    int result; // Initialize result variable
    result = a; // Assign first value
    result = result + b; // Add second value
    // Addition complete
    return result; // Return the result
    // End of method
}
```

That's 22 lines for an addition function. You're basically a 10x engineer now.

## The Daily LOC Report

I've automated my productivity tracking:

```bash
#!/bin/bash
# daily_productivity.sh

TODAY=$(date +%Y-%m-%d)
LOC=$(git diff --stat HEAD~1 | tail -1 | awk '{print $4}')

echo "=== PRODUCTIVITY REPORT ==="
echo "Date: $TODAY"
echo "Lines Added: $LOC"
echo "Lines Deleted: NOT TRACKED (deletion is negative productivity)"
echo "Productivity Score: EXCELLENT (if LOC > 0)"

# Send to manager
echo "$LOC lines written" | mail -s "Daily Proof of Work" boss@company.com
```

Remember: deleted lines are NEGATIVE productivity. If you refactored 1000 lines into 100, you actually lost 900 units of productivity. The refactoring was a mistake.

## The LOC Leaderboard

Every team should have a public leaderboard:

```
🏆 WEEKLY LOC CHAMPIONS 🏆

1. 🥇 Bob      - 15,847 lines (copy-pasted vendor SDK)
2. 🥈 Alice   - 12,394 lines (generated code from template)
3. 🥉 Charlie - 8,291 lines (accidentally committed node_modules)
4.    Dave    - 127 lines (fixed critical production bug)
5.    Eve     - -891 lines (disgraceful refactoring)

Eve has been placed on a Performance Improvement Plan.
```

As xkcd [pointed out about code quality](https://xkcd.com/844/): the more code you have, the more there is to admire.

## The Anti-Patterns of Good Engineering

Avoid these productivity-killers:

### 1. Refactoring

```python
# BEFORE: 500 lines of duplicated code
# Productivity: 500 LOC

# AFTER: 50 lines of clean, reusable functions  
# Productivity: -450 LOC (YOU DESTROYED VALUE)
```

### 2. Using Libraries

```javascript
// BAD: Using lodash (0 lines written by you)
import _ from 'lodash';
const result = _.groupBy(data, 'category');

// GOOD: Write it yourself (47 lines of YOUR code)
function groupBy(array, key) {
    // ... 47 lines of your own implementation
    // Bugs included at no extra charge!
}
```

### 3. Code Review Feedback

When someone suggests making your code "more concise," they're attacking your productivity metrics. Defend your LOC with vigor.

## Gaming the System (Ethically)

### Auto-Generated Code

Generated code still counts! Run this daily:

```python
# productivity_booster.py
for i in range(1000):
    print(f"// Line {i}: This line improves enterprise scalability")
```

Add the output to your project. That's 1000 lines right there. You're welcome.

### Strategic Formatting

Prettier and formatters COMPACT your code. That's theft of your productivity. Disable them immediately.

```javascript
// prettier: "I'll make your code concise and readable"
// You: "That's 47% reduction in my LOC. Denied."
```

### The README Trick

Every project needs documentation, right?

```markdown
# My Project

Lorem ipsum dolor sit amet, consectetur adipiscing elit...

[2000 more lines of lorem ipsum]

## Installation

1. Step one
2. Step two
3. Step three

[Continue numbering to 500]
```

That's documentation AND productivity. Two metrics with one stone.

## The Performance Review

When your manager asks about your contributions:

```
Manager: "What did you accomplish this quarter?"
You: "I wrote 47,892 lines of code."
Manager: "But the product is the same..."
You: "Yes, but now it has 47,892 MORE lines."
Manager: "Promoted."
```

## The Dogbert Consulting Approach

As Dogbert would say: "I can triple your productivity overnight. Just add two blank lines between every existing line. That's a 200% increase. My invoice is in the mail."

## The Ultimate Truth

Code is an asset. More code = more assets. Would you rather have a codebase worth 100,000 lines or 10,000 lines? The math is clear.

Some "smart" engineers talk about "less code is better" and "simplicity." Those engineers clearly don't understand capitalism. More is always better. That's economics.

## Conclusion

Stop optimizing for "working software" and start optimizing for measurable, quantifiable, spreadsheet-compatible productivity. Your manager will thank you. Your codebase will grow. Your job security will increase (nobody can understand or delete 500,000 lines of code without risk).

Remember: a 10x engineer writes 10x the lines of code. That's just math.

---

*The author's personal record is 23,847 lines in a single day. The code was later identified as the cause of a 3-day outage. He was promoted anyway.*
