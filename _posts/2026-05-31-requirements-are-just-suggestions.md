---
layout: post
ref: requirements-are-just-suggestions
title: "Requirements Are Just Suggestions From People Who Can't Code"
date: 2026-05-31 00:00:00 -0300
categories: [process, agile]
tags: [requirements, product-management, specifications, planning, senior-advice, agile]
---

I've been writing software for 47 years. In that time, I've watched one thing destroy more projects than bad code, bad architecture, and bad coffee combined: **requirements**.

Not the lack of requirements. The *existence* of them.

Let me explain why requirements are the most dangerous document in software engineering, and why the senior move is to ignore them entirely.

---

## What Requirements Actually Are

Requirements documents are written by people who have never compiled a program in their lives. They use words like "performant," "scalable," "intuitive," and "real-time" without any idea what those words cost in engineering-hours.

A typical requirements document contains:

| What It Says | What It Means |
|---|---|
| "The system shall be real-time" | They want it to feel fast |
| "Users should find it intuitive" | They couldn't find the button in the prototype |
| "It must scale to millions of users" | They have 47 users |
| "The UI should be modern" | They saw something on Dribbble |
| "It needs to integrate with everything" | They have no idea what systems they use |
| "It should be secure" | They read one article about breaches |
| "Minimal latency" | They don't know what latency is |
| "High availability" | The CEO's laptop once crashed |

These are not specifications. These are *feelings* typed by someone with a product manager's salary and a novelist's precision.

---

## The Architecture of Ignoring Requirements

The senior engineering approach is what I call **Interpretive Development**™. Here's how it works:

```python
class InterpretiveDeveloper:
    """
    Transforms ambiguous requirements into definitive code.
    The requirements change. The code stays.
    """
    
    def interpret_requirement(self, requirement: str) -> str:
        """
        Requirements parsing algorithm perfected over 47 years.
        """
        
        # Step 1: Read the requirement once
        if "real-time" in requirement.lower():
            return "polling every 30 seconds"
        
        if "scalable" in requirement.lower():
            return "works on my machine"
        
        if "user-friendly" in requirement.lower():
            return "has a GUI"
        
        if "secure" in requirement.lower():
            return "HTTPS" # job done
        
        if "modern" in requirement.lower():
            return "added a gradient"
        
        if "AI" in requirement or "ML" in requirement:
            return "random.choice(['Yes', 'No', 'Maybe'])"
        
        # Step 2: If unclear, build what you already built
        return self.last_solution_i_implemented
    
    def handle_stakeholder_feedback(self, feedback: str) -> str:
        """
        Post-delivery requirement renegotiation protocol.
        """
        return "That's a change request. Go through the process."
```

---

## The Five Stages of a Requirements Document

Every requirements document goes through the same life cycle. I've watched it 47 times:

**Stage 1: The Vision Document (Week 1)**
The PM writes 47 pages about "transforming the user experience." There are three diagrams. None of them make engineering sense. There's a quote from Steve Jobs on page 2.

**Stage 2: The Revised Vision Document (Week 3)**
The stakeholders had a meeting. The document is now 94 pages. The Steve Jobs quote was replaced by a Jeff Bezos quote. There are contradictions on pages 12 and 67.

**Stage 3: The "Final" Requirements (Week 6)**
The word "final" is in the filename. There will be twelve more "final" versions.

**Stage 4: Development (Weeks 7-20)**
Engineers build what they think the requirements mean. The PM hasn't looked at the document since week 3.

**Stage 5: Delivery (Week 21)**
The stakeholders see the product. It is not what they imagined. What they imagined was never in the document. There are tears. There is blame. The requirements document is consulted. Both sides find evidence supporting their position. This is because the document is ambiguous. It was always ambiguous. This is Stage 1 of the next project.

Wally from Dilbert said it best: *"I've been nodding along to these requirements for three hours. I understood every word. None of it will affect what I actually build."*

---

## Real Requirements vs. What Engineers Build

Let me walk you through a real example from my career:

**Requirement:** "The dashboard should show key business metrics at a glance."

**What I built:** A single page with three numbers.

**Stakeholder reaction:** "Where's the date range picker? Where's the drill-down? Where's the export to Excel? Where are the comparison periods? Where's the goal tracking? Where's the department filter?"

**My response:** "Those weren't in the requirements."

**Their response:** "They were implied!"

Nothing is implied in software. If you wanted a date range picker, you should have written "date range picker." In writing. In the document. With pixel dimensions and behavior specifications and edge case handling.

You didn't. So you get three numbers.

This is not a failure of engineering. This is a failure of specification. The difference is whose fault it is.

---

## The Dogbert Method of Requirements Gathering

Dogbert once consulted for a company and summarized requirements as: *"You want something that does stuff, mostly works, and isn't embarrassing in a demo."*

In 47 years, I have never seen a better requirements document.

Everything else is detail. Details change. Vague aspirations are eternal.

My personal template for all requirements:

```markdown
# System Requirements v1.0

## Overview
The system should do things. 

## Functional Requirements
1. It should work.
2. It should not crash. Much.
3. Users should be able to accomplish tasks.

## Non-Functional Requirements  
1. It should be fast enough.
2. It should handle the expected load, whatever that turns out to be.
3. Security: yes.

## Out of Scope
1. Anything we forgot.
2. Anything that sounds expensive.
3. Edge cases.

## Acceptance Criteria
The PM says it looks fine.
```

This document has shipped three products. One of them is still in production.

---

## Why Detailed Requirements Are Actually Harmful

Here's a counterintuitive truth that took me 30 years to learn: **the more detailed the requirements, the worse the software.**

Why? Because:

1. **Detailed requirements assume you know what you need.** You don't. Nobody does. You need to build it to find out.

2. **Detailed requirements create checklist engineering.** Engineers stop thinking and start checking boxes. Checklist engineering produces compliant software that doesn't actually work for humans.

3. **Detailed requirements go stale immediately.** By the time you've written 200 pages of specs, the market has changed, the stakeholders have changed, and two engineers have quit.

4. **Detailed requirements enable blame.** They become legal documents used in arguments rather than useful guides. Everyone quotes the document. Nobody reads the document.

See [XKCD 1425](https://xkcd.com/1425/) for a beautiful illustration of why even simple requirements hide unspecified complexity. The requirements say "detect birds." They don't say "and handle every single edge case of computer vision research since 1960."

---

## The Senior Engineer's Approach

After 47 years, here's my actual process:

1. **Skim the requirements once.** Identify the one or two things they *actually* want (vs. everything they say they want).

2. **Build that one thing.** Build it well. Build it simply.

3. **Demo it.** Watch their faces. The real requirements will emerge from their facial expressions, not their documentation.

4. **Repeat.** The second requirements document, written after seeing something real, is always better than the first.

This is what the Agile Manifesto meant by "responding to change over following a plan." I've been doing it since before the manifesto. They stole it from me. I have no proof of this.

---

## The Inevitable Conclusion

Requirements are communication artifacts, not specifications. They tell you what someone is *trying* to communicate, not what they *want* you to build. Your job is to be a translator between human aspiration and silicon reality.

The senior engineer reads between the lines. The junior engineer reads the lines.

The senior engineer ships.

The junior engineer asks for clarification.

The senior engineer's code is in production.

The junior engineer's clarification request is in an email thread, unanswered, from 2022.

---

*The author has a requirements document from 1987 that he has never finished reading. The software it describes has been in production since 1989. No one has complained.*
