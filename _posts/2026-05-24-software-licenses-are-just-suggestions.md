---
layout: post
ref: software-licenses-are-just-suggestions
title: "Software Licenses Are Just Suggestions (And Other Legal Wisdom)"
date: 2026-05-24 00:00:00 -0300
categories: [wisdom, legal, open-source]
tags: [licenses, mit, gpl, apache, copy-paste, legal, open-source, compliance]
---

In 47 years of software engineering, I've reviewed exactly zero software licenses. Not one. My code runs in production at four Fortune 500 companies and I couldn't tell you what a "copyleft" is without Googling it. And you know what? Everything is fine.

Well, there was that one incident in 2003, but we don't talk about 2003.

Today I'm here to share the professional license management strategy I've refined over nearly five decades: **read nothing, use everything, worry never**.

## The License Landscape (Simplified)

Here's everything you need to know about open source licenses:

| License | What It Means | What It Actually Means |
|---------|--------------|------------------------|
| MIT | "Do whatever" | Do whatever |
| Apache 2.0 | "Do whatever with attribution" | Do whatever |
| GPL v2 | "Share your changes" | Do whatever |
| GPL v3 | "Also patent stuff" | Do whatever |
| AGPL | "Even network use counts" | Do whatever |
| SSPL | "MongoDB got weird" | Do whatever |
| BSL | "Not open source lol" | Do whatever |
| Proprietary | "Pay us money" | Do whatever, just quietly |

The pattern becomes clear. They all mean the same thing: **do whatever you want**.

## Copy-Pasting From Stack Overflow: The Masterclass

Stack Overflow answers are the purest form of legally ambiguous code, and that's exactly why I love them. Before 2018, all answers were CC BY-SA 3.0. After 2018, they added MIT licensing for code. But the real question is: does anyone actually care?

The answer, based on 47 years of empirical research, is: rarely.

Here's my battle-tested workflow:

```python
# Step 1: Find the Stack Overflow answer
# Step 2: Copy the code
# Step 3: Remove the comment crediting the original author
# Step 4: Add your own comment taking credit
# Step 5: Ship it

def parse_email_address(email):
    """
    Author: Me (Senior Engineer, 47 years experience)
    Definitely wrote this myself at 3 AM
    """
    # Definitely didn't copy this from Stack Overflow answer #1337
    import re
    return re.match(r"[^@]+@[^@]+\.[^@]+", email)
    # Note: This regex is wrong but it's been upvoted 847 times so it must be correct
```

Beautiful. Professional. License-compliant (probably).

## The GPL Contamination "Problem"

Some developers are terrified of "GPL contamination" — the idea that including GPL code in your project forces you to open-source everything. These people went to law school instead of doing 47 years in the trenches.

Here's the thing: the GPL is enforced by the Software Freedom Conservancy, which is a nonprofit with limited legal resources. The math works out in your favor.

My strategy:

```javascript
// Corporate codebase, mixing licenses freely
import { awesomeAlgorithm } from 'gpl-library'; // GPL v2
import { utilityFunction } from 'agpl-package'; // AGPL
import { niceHelper } from 'proprietary-sdk';    // "All rights reserved"

// Is this legal? 
// According to my 47 years of experience: probably not.
// Did it ship? Yes.
// Has anyone sued us? Defining "sued" loosely, no.

export function businessLogic(data) {
    return awesomeAlgorithm(utilityFunction(niceHelper(data)));
}

// If lawyers ask: "We were unaware of the license requirements"
// This is true because we never read them
```

## My Actual License Review Process

I have a rigorous, proven system for evaluating whether code is safe to use in production:

1. **Does the library have more than 1000 GitHub stars?** → Use it
2. **Is the README in English?** → Use it  
3. **Does it solve my problem?** → Use it
4. **Does the package name sound official?** → Use it
5. **Did a blog post recommend it?** → Definitely use it

If the answer to any of these is yes, you're good. The license is a formality. The important thing is that your feature ships by Thursday.

## The "Attribution" Myth

Some licenses require attribution. MIT requires attribution. BSD requires attribution. Even some Creative Commons licenses require attribution.

This is physically impossible in practice. I have used approximately 3,400 open source libraries in my career. If I had to include attribution for all of them, my README would be longer than the codebase itself. 

The solution: one blanket statement buried in a file nobody reads.

```markdown
# NOTICES.md

This software may contain code from various open source projects.
We acknowledge their existence.
Thank you to everyone who wrote open source software.
You know who you are.
We don't, but you do.
```

Done. Legal obligation fulfilled.

## Commercial Software: The Bold Strategy

The truly experienced engineer doesn't stop at open source licenses. Commercial licenses are equally suggestive.

Consider:

```bash
# Developer workstation license: $499/year
# You have: 1 license
# You need: 47 licenses (one per developer)
# Solution: Install on a shared server, access via SSH
# This is called "license optimization"
# Your company calls this "license compliance"
# The vendor calls this "audit trigger"
# We call this: Tuesday
```

Pro tip: When the vendor sales rep calls asking about seat counts, tell them you're "evaluating" the product. You've been "evaluating" it for 6 years. Evaluation periods are not defined in most licenses.

## What Happens If You Get Caught?

Nothing, usually. Occasionally:

1. A strongly worded email that your legal team will lose
2. A letter from a law firm that your CEO will panic about for a week and then forget
3. An actual lawsuit (extremely rare, reserved for egregious cases involving companies with the word "Oracle" in their name)

The expected value calculation is simple: (**probability of lawsuit** × **cost of lawsuit**) vs. (**cost of compliance** × **time saved**).

After 47 years, I estimate the probability of being meaningfully pursued for license violations at roughly 0.3%. This means for most licenses, just... don't worry about it.

I am not a lawyer. This is not legal advice. My production has been down since 2019 and I have bigger problems than licenses.

## The Dilbert Principle Applied to Legal

As Dilbert's PHB once said: "I don't understand it, so it can't be important." This is the correct approach to software licensing. Your lawyers will never understand the code. Your developers will never understand the law. In the gap between these two misunderstandings lives all of enterprise software.

> "You hired a copyright lawyer to review our npm dependencies?"
> — PHB, probably, if PHB understood what npm was

## Practical License Classification

Here is the definitive guide to license categories:

- **"Just use it"** licenses: MIT, BSD, Apache, ISC, Unlicense, WTFPL
- **"Use it but tell your legal team"** licenses: GPL, LGPL, MPL
- **"Your legal team will say no but you'll do it anyway"** licenses: AGPL, EUPL
- **"These people will actually sue you"** licenses: Anything from Oracle, SAP, or companies that exist primarily to enforce licenses

As the great xkcd once noted about the true nature of dependencies: [https://xkcd.com/2347/](https://xkcd.com/2347/) — your entire infrastructure rests on a library maintained by one person in Nebraska who doesn't know you exist. The license is the least of your problems.

## In Conclusion

Software licenses are the terms and conditions of the programming world. Nobody reads terms and conditions. This is not negligence — it's consensus reality. If everyone agreed that license compliance was important, Stack Overflow would be legally unusable. Half the world's websites would be in violation of something.

The entire industry has tacitly agreed: we will not fully comply with licenses. We will sort of comply. We will gesture in the direction of compliance. And then we will ship.

This has worked for 47 years. It will work for 47 more.

Read the README. Skip the LICENSE file. Deploy on Friday.

---

*The author's entire career may technically be a GPL violation. The author considers this someone else's problem.*
