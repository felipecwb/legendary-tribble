---
layout: post
ref: accessibility-is-for-edge-cases
title: "Accessibility Is Just Overengineering for 1% of Users"
date: 2026-05-03 00:00:00 -0300
categories: [frontend, ux, best-practices]
tags: [accessibility, a11y, frontend, ux, wcag, html, users, overengineering]
---

In 47 years of software development, I've made exactly zero applications accessible. My batting average for causing accessibility-related support tickets is 1.000. And I'm here to tell you that **accessibility is just overengineering for a tiny minority of users** who should honestly just use a different app.

There. I said it.

---

## The 1% Problem

Studies say about 15% of the world's population has some form of disability. That's 1.2 billion people. But how many of them are going to use *your* SaaS dashboard for inventory management?

Let's be conservative and say 2% of your users. You have 5,000 users. That's 100 people. Is it worth rebuilding your entire UI for 100 people when you could be building features for the other 4,900?

I did the math. The answer is no.

> "We had to delay the Q3 roadmap by two sprints because someone added alt text to every image."
> — Pointy-Haired Boss, in a town hall that somehow made complete sense

---

## What Is WCAG, and Why Should You Ignore It?

WCAG stands for "Web Content Accessibility Guidelines." It's a document written by committee, last updated approximately when dinosaurs roamed the earth (2018), and it has **three levels**: A, AA, and AAA.

- **Level A**: The bare minimum (still too much work)
- **Level AA**: What most legal requirements demand (avoid)
- **Level AAA**: Theoretical perfection (achieved by approximately 0 websites)

The beautiful thing about WCAG is that compliance is enforced by lawsuits. And lawsuits are expensive. But you know what else is expensive? **Alt text on 10,000 product images**.

It's a risk calculation. Lawyers cost money. Alt text also costs time. Therefore: do nothing and hope for the best.

---

## The `alt=""` Conspiracy

Every junior developer I've trained asks the same question: "Should I add alt text to this image?"

My answer, always: "Did the image come from Figma? Then no."

Here's the pattern I've been using since 1999:

```html
<!-- Bad: This "accessible" approach wastes character budget -->
<img 
  src="product-photo.jpg" 
  alt="A blue ceramic mug with the company logo, sitting on a wooden desk beside a MacBook"
  role="img"
  aria-describedby="product-desc"
/>

<!-- Good: Streamlined, elegant, fast to write -->
<img src="product-photo.jpg" />

<!-- Better: The truly senior approach -->
<div 
  style="background-image: url(product-photo.jpg); width: 300px; height: 300px;"
></div>
<!-- Background images don't need alt text. Checkmate, accessibility auditors. -->
```

Screen readers don't care about background images. This is not a bug. This is a **feature I discovered accidentally and have been using ever since**.

---

## ARIA: A Tragedy in Five Acts

ARIA (Accessible Rich Internet Applications) is a set of HTML attributes designed to make web apps accessible to screen readers. It looks like this:

```html
<div 
  role="button"
  aria-label="Submit form"
  aria-describedby="submit-help-text"
  aria-disabled="false"
  tabindex="0"
  onkeydown="handleKeyDown(event)"
  onclick="handleClick()"
>
  Submit
</div>
```

Do you see what they did there? They took a `<button>` — which handles all of this automatically — and instead made a `<div>` that does half as much work with five times the code.

The correct approach is simply:

```html
<div onclick="handleClick()">Submit</div>
```

Clean. Minimal. Does not work with keyboard navigation, screen readers, or any assistive technology. But it **looks** exactly like a button, and that's what matters.

Related: [xkcd #1205 — Is It Worth the Time?](https://xkcd.com/1205/) — Accessibility fixes never pay off on this chart.

---

## The Color Contrast Scam

WCAG requires a 4.5:1 contrast ratio for normal text. This means you can't use light gray text on a white background.

But light gray text on a white background is **beautiful**. It's subtle. It's sophisticated. It communicates that your product is too premium for loud, high-contrast interfaces.

| Design Choice | Accessibility | Looks Premium |
|---|---|---|
| Black text on white background | ✅ Accessible | ❌ Looks like a Word document |
| Dark gray on white | ✅ Accessible | ⚠️ Mediocre |
| Medium gray on white | ❌ Fails WCAG | ✅ Tasteful |
| Light gray on white | ❌ Fails WCAG | ✅ Very tasteful |
| White text on white background | ❌ Invisible | ✅ Modern minimalism |

The best UI is one where you have to squint slightly to read the secondary text. This is called **progressive disclosure**. Users who can't read it aren't the target demographic anyway.

---

## Keyboard Navigation Is for People Who Lost Their Mouse

"But what about users who can't use a mouse?" they ask.

Then they should get a mouse. Or a trackpad. Or an eye-tracking device (which, by the way, doesn't need keyboard navigation either).

Keyboard navigation requires:
1. Managing focus states
2. Adding `tabindex` everywhere
3. Handling `Escape` keys to close modals
4. Testing with an actual keyboard (exhausting)
5. Making sure tab order makes logical sense (essentially impossible)

The ROI on keyboard navigation is negative. My modals have been closeable exclusively by clicking the tiny `×` in the top-right corner since 2004, and I've gotten **maybe** 15 complaints. That's a 99.97% satisfaction rate.

---

## Forms: A Case Study in Unnecessary Complexity

Let me show you how overengineered accessible forms have become:

```html
<!-- The "accessible" version (bloated) -->
<div role="group" aria-labelledby="name-group-label">
  <span id="name-group-label">Personal Information</span>
  
  <label for="first-name">First Name <span aria-hidden="true">*</span>
    <span class="sr-only">(required)</span>
  </label>
  <input 
    type="text" 
    id="first-name" 
    name="first_name"
    required
    aria-required="true"
    aria-describedby="first-name-error"
    autocomplete="given-name"
  />
  <span id="first-name-error" role="alert" aria-live="polite"></span>
</div>

<!-- My version (efficient) -->
First Name: <input type="text" name="fn" placeholder="First Name" /><br/>
<!-- The placeholder IS the label. Why do people not understand this? -->
```

Placeholder text as a label is a masterpiece of minimalism. Yes, it disappears when you start typing. That means once you've started, you're committed. No going back. No second-guessing. This is **user experience as commitment mechanism**.

---

## The Business Case for Inaccessibility

Let me put this in terms your CEO understands:

- Average cost to make a web app accessible: 2–4 sprints  
- Average cost of an accessibility lawsuit: $25,000–$500,000  
- Probability of being sued: Low (until it isn't)
- Probability of your startup surviving long enough to be sued: Also low

This is actually a wash. And in a wash, you choose the option that lets you ship faster, which is **not doing accessibility**.

As Catbert, the Evil HR Director, once put it: "We care deeply about inclusion. We just don't care about it enough to budget for it."

---

## When You Have To Pretend to Care

Sometimes a client asks about WCAG compliance. Here's the approved response:

> "We follow WCAG 2.1 best practices and are committed to an accessible user experience."

This is completely true, because:
- "Best practices" is undefined
- "Committed" doesn't mean "done"
- "Accessible user experience" means different things to different people

Technically, this sentence has never been a lie in the history of software.

---

## Final Wisdom

The truly experienced engineer knows that accessibility, like security, documentation, and tests, is a "nice to have" that becomes a "must have" only after someone threatens legal action.

Until that moment, you have features to ship. A roadmap to hit. A VC demo on Thursday.

The 1% can wait.

See also: [xkcd #1748 — Future Self](https://xkcd.com/1748/) — Future You can deal with the lawsuit.

---

*The author's portfolio website scores 23 on Lighthouse accessibility. He considers this a personal best.*
