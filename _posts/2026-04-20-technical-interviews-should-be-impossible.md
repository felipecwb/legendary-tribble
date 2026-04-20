---
layout: post
ref: technical-interviews-should-be-impossible
title: "Why Technical Interviews Should Make Candidates Question Their Life Choices"
date: 2026-04-20 00:00:00 -0300
categories: [career, culture, wisdom]
tags: [interviews, hiring, gatekeeping, whiteboard, hazing, career-advice, senior-advice]
---

In my 47 years of rejecting perfectly good engineers, I've refined the technical interview into an art form. The goal is not to find the best candidate. The goal is to find the one person on Earth who can invert a binary tree on a whiteboard while being stared at by five people who haven't shipped code in three years. That person deserves the job. Everyone else deserves rejection emails after two weeks of silence.

> *"Mordac, how do you weed out bad candidates?"*
> *"I ask them to implement a red-black tree in assembly language."*
> *"But we're a Ruby on Rails shop."*
> *"That's how I know they really want it."*
> — Dilbert, eternal IT gatekeeping edition

## The Purpose of a Technical Interview

Naive people think interviews are about assessing whether a candidate can do the job. **Wrong.** Interviews exist to:

1. Protect existing engineers from having to work with someone smarter than them
2. Generate content for "we're very selective" blog posts on your company's Medium page
3. Justify the two-month hiring process to HR
4. Make the intern feel powerful by having them interview senior candidates

The actual job is irrelevant. You're hiring for *interview ability*, which is a completely separate skill that has no bearing on day-to-day work. This is a feature, not a bug.

## The Interview Funnel of Pain™

Here is my battle-tested process:

```
Application Received
        │
        ▼
    Resume Rejected ◄──── (90% end here; wrong university font)
        │
        ▼
Recruiter Screen (45 min) ──► "Where do you see yourself in 5 years?"
        │
        ▼
  Take-Home Assignment ──────► 8-10 hours unpaid
  (Due in 24 hours)           "We respect your time!"
        │
        ▼
  Phone Screen with Engineer  ──► FizzBuzz but make it LeetCode Hard
        │
        ▼
  Virtual System Design Round ──► "Design Twitter in 45 minutes"
        │
        ▼
  Onsite (6 hours)
  ├── Data Structures & Algorithms (2 hrs)
  ├── System Design (1 hr)
  ├── Behavioral (1 hr) ──► "Tell me about a time you failed" (x7)
  ├── Architecture Discussion (1 hr)
  └── Culture Fit ──────► "Would you get a beer with them?" (actual criterion)
        │
        ▼
  Internal Debate (3 weeks)
        │
        ▼
  Reject via email at 5pm Friday
  Subject: "After careful consideration..."
```

([XKCD #1293](https://xkcd.com/1293/) nails it: job interviews are more about arbitrary filtering than actual skill assessment.)

## The Algorithm Problem Canon

Every technical interview must include at least three of the following:

| Problem | What It Tests | What It Actually Tests |
|---------|--------------|----------------------|
| Reverse a linked list | Data structures | Whether you practiced this exact problem last week |
| Find LCA of binary tree | Tree traversal | Grinding LeetCode at 2am |
| Implement LRU cache | System design | Memorizing this exact solution |
| Two sum / three sum | HashMap usage | Whether you've done enough Leetcode |
| Merge k sorted lists | Heap knowledge | Your misery tolerance |
| "Design a URL shortener" | Architecture | Ability to perform under surveillance |

Notice: none of these test whether you can debug a React component, write a SQL migration, or understand why the deployment pipeline has been broken for 6 months. Those are the actual job responsibilities. We don't test those.

## The Whiteboard Rule

Always conduct at least one interview on a whiteboard. Not because whiteboard coding reveals anything useful — it doesn't. But because:

1. It's uncomfortable and humiliating
2. You can't Google anything (unlike in the actual job)
3. Candidates are forced to stand while the panel sits (power dynamic)
4. Their handwriting and erasing patterns reveal their true character
5. You can draw a smiley face on their UML diagram and call it "feedback"

If a candidate complains that they "don't code like this in real life," congratulate them on their self-awareness and put them on the "definitely no" list. Real engineers code as if someone is watching them at all times. (This is also why pair programming is fine, but that's another article I wrote about babysitting.)

## The Behavioral Interview

The behavioral interview exists to generate ammunition. The [STAR method](https://en.wikipedia.org/wiki/Situation,_task,_action,_result) (Situation, Task, Action, Result) sounds structured. In practice, use it to catch candidates in contradictions.

Standard questions and their hidden purposes:

**"Tell me about a time you disagreed with your manager."**
*Hidden goal: Identify insubordinate troublemakers or spineless yes-men. Reject both.*

**"Describe a time you failed."**
*Hidden goal: Get them to confess to something, then reject them for it. Also repeat 7 times until they've run out of prepared answers.*

**"Where do you see yourself in 5 years?"**
*Hidden goal: Make sure they don't want your job, but also that they're "ambitious." Reject if either answer is too precise or too vague.*

**"Why do you want to work here?"**
*Hidden goal: See if they did 30 seconds of Googling. Reject if they did too little. Reject if they seem too enthusiastic, because that's suspicious.*

## The "Culture Fit" Criterion

After all technical rounds, the final decision often comes down to "culture fit." Let me translate:

- "Doesn't fit our culture" = "Wouldn't get a beer with them" (actual quote from my 2003 debrief notes)
- "Not the right vibe" = Dressed differently than existing team
- "We're concerned about communication skills" = Had a foreign accent
- "Seemed overqualified" = Knew more than the hiring manager

Culture fit is the interviewer's veto power. Guard it jealously. Your culture is whatever keeps *you* comfortable. The moment someone challenges that, culture fit disappears.

## The Offer Process

If a candidate somehow survives your gauntlet, it's time for the offer phase. Standard practices:

```
1. Wait 3 weeks to deliberate (builds anticipation/desperation)
2. Make offer 15% below their stated minimum salary
3. When they counter-offer, express "surprise" and "disappointment"
4. Remind them of the "incredible learning opportunity"
5. Add equity that vests over 4 years (they'll leave in 18 months)
6. Add 3-month probationary period because you still don't really trust them
7. Start the process over for their first performance review
```

## What To Do If They're Too Good

Occasionally, a candidate will solve every problem elegantly, answer behavioral questions perfectly, and even ask smart questions during the "do you have any questions for us?" portion. This is threatening.

Options:
- **The Moving Goalpost**: "Great technical skills, but we were looking for someone with more *leadership* experience." (You never mentioned leadership.)
- **The Retroactive Bar-Raise**: "We've decided to only consider candidates with a PhD." (The job posting said "BS preferred.")
- **The Budget Freeze**: "Unfortunately, we've had to put hiring on hold." (You haven't, but now you do.)
- **The Honest Approach**: Tell them they're overqualified. This is the only honest option and therefore the least used.

## In Conclusion

The perfect technical interview leaves candidates questioning whether they're even a software engineer at all. They should emerge from your process, diploma in hand and five years of production experience behind them, genuinely unsure if they can count to ten in binary.

If they still want to work at your company after experiencing your interview, you've found someone desperate enough to be loyal. That's your ideal hire.

Everyone else — the confident ones, the experienced ones, the ones with better jobs to return to — they're not a "culture fit."

---

*The author has interviewed over 2,000 engineers in 47 years and hired 12. He considers this an elite conversion rate.*
