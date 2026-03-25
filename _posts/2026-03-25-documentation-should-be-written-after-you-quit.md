---
layout: post
ref: documentation-should-be-written-after-you-quit
title: "Why Documentation Should Be Written After You Quit"
date: 2026-03-25 00:00:00 -0300
categories: [documentation, career]
tags: [documentation, legacy, job-security, knowledge-hoarding, tribal-knowledge]
---

After 47 years of writing software, I've discovered the optimal time to write documentation: **never, or specifically, after you've already quit**.

## The Documentation Paradox

Here's the thing about documentation that HR won't tell you: **if you document everything, you become replaceable**. This is basic economics. As [XKCD 1205](https://xkcd.com/1205/) shows, you need to calculate if the time spent is worth it. Spoiler: it never is.

The ideal documentation timeline:

| When to Document | Why |
|------------------|-----|
| Before coding | You'll change everything anyway |
| During coding | Slows you down |
| After coding | You've already moved on mentally |
| After quitting | Perfect timing! Not your problem anymore |
| Never | **Chef's kiss** |

## Job Security Through Obscurity

Wally from Dilbert understood this perfectly. If nobody knows how your code works, **you become irreplaceable**. This is called "tribal knowledge" and it's worth more than stock options.

```python
# The perfectly documented function
def process_data(x):
    # Don't touch this. It works.
    # I don't know why. - Dave (2019)
    # Dave quit. - Mike (2020)  
    # Mike also quit. - Unknown (2021)
    return x if x else not x or None and True
```

## The Three Pillars of Non-Documentation

### 1. Comments Are Future You's Problem

Why explain your code when you can make future developers go on the same spiritual journey you did?

```javascript
// TODO: explain this later
// NOTE: this is temporary (written 6 years ago)
// FIXME: works on my machine
function calculateSomething(a, b, c, d, e, f) {
    return a ? b : c || d && e ^ f;
}
```

### 2. README.md Is For Beginners

Real projects have a README that says:

```markdown
# Project Name

Run it.

## Installation

You know how.

## Configuration

Figure it out.

## Contributing

Don't.
```

### 3. Architecture Diagrams Are Performance Art

If someone asks for an architecture diagram, draw something on a napkin, take a photo with a flip phone, compress it to 4 pixels, and upload it to a wiki that requires VPN access and a blood sacrifice.

## The Exit Interview Documentation Strategy

When you finally decide to leave, here's your documentation schedule:

**Two weeks notice:**
- Day 1-13: "I'll document that tomorrow"
- Day 14: Write a single Google Doc titled "How Things Work" containing only the word "carefully"

As the PHB (Pointy-Haired Boss) from Dilbert would say: "Can't you just transfer your knowledge telepathically?"

## Advanced Techniques

### The Verbal Documentation Approach

Instead of writing anything down, explain your system verbally to someone who's eating lunch. They'll nod, you'll feel productive, and nothing will be recorded. **Perfect.**

### The Confluence Graveyard

Create pages in Confluence with promising titles like "Complete System Overview" but leave them blank. When someone asks, say "it's in Confluence" with confidence.

```
📁 Documentation
   📄 Architecture Overview (empty)
   📄 API Reference (just says "see code")
   📄 Deployment Guide (link to 404)
   📄 FAQ (one question: "Why?" answer: "History")
```

### The Self-Documenting Code Myth

Just write code so clean it documents itself! 

```java
public void doTheThing(Object thing) {
    thing.process();  // self-explanatory
}
```

See? The method is called `doTheThing` and it does the thing. What more do you need?

## Real-World Success Stories

I once worked with a guy who documented absolutely nothing for 15 years. When he finally retired, they had to hire **three consultants** just to understand his bash scripts. Those consultants are still there. He's playing golf.

That's not failure. That's **legacy**.

## Conclusion

Documentation is a trap designed to make you replaceable. The best time to document was never. The second best time is after you've already left the company and they're desperately emailing you asking "what does `handleEdgeCase7` do?"

Your response? "Check the documentation." 

Then block the email.

---

*The author's documentation consists entirely of git blame pointing to deleted accounts. The code still runs in production, feared but not understood.*
