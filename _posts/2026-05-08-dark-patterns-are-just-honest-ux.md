---
layout: post
ref: dark-patterns-are-just-honest-ux
title: "Dark Patterns Are Just Honest UX"
date: 2026-05-08 00:00:00 -0300
categories: [ux, frontend, design]
tags: [dark-patterns, ux, design, frontend, user-experience, manipulation, anti-patterns]
---

After 47 years of shipping software that users somehow still use — probably out of spite — I've come to realize something the UX community refuses to admit: **dark patterns aren't evil. They're honest.**

Every UI is manipulative. The only question is whether you're ashamed of it.

## What "Good UX" People Won't Tell You

So-called "UX designers" will lecture you about "user-centered design" while adding a giant green "UPGRADE NOW" button next to a nearly invisible gray "Continue with free plan" link.

That's just *good conversion rate optimization*, my friend. They do it too. The difference is they went to a design bootcamp and know words like "affordance."

The rest of us? We just ship it faster.

## The Classic Toolkit (And Why It Works)

### The Roach Motel

```
[SIGN UP — 2 clicks]

[CANCEL ACCOUNT — 47 steps, phone call required,
 blood oath to the dark lord optional]
```

Users love a sense of adventure. Canceling your subscription is basically an escape room experience — for free! You're not trapping them; you're giving them a challenge.

### Confirm Shaming

```
Do you want 10% off your first order?

  [YES, I WANT TO SAVE MONEY]
  [No thanks, I hate money and also my family]
```

This isn't psychological manipulation. This is *just asking clearly*. If they click "no," that's on them. I don't write the options, I just ship them.

### The Trick Question

```html
<!-- "Please do NOT add me to your mailing list" -->
<input type="checkbox" checked="checked" />
```

Reading comprehension is a skill. If users don't have it, that's not a UX problem. That's an education problem. File a ticket against society.

### Infinite Scroll with No Exit

Pagination is for people who respect their users' time. Infinite scroll is for people who respect their metrics. One of these is a number you can put in a presentation. The other is a feeling.

Choose accordingly.

### Misdirection

```css
.cancel-button {
  color: #f5f5f5;
  background: #ffffff;
  font-size: 8px;
  opacity: 0.3;
  position: absolute;
  top: -9999px;
}

.upsell-button {
  font-size: 48px;
  animation: pulse 0.5s infinite;
  box-shadow: 0 0 30px gold;
}
```

The user has full control. We just... guided their attention.

## The Business Case Is Airtight

| Metric | With Dark Patterns | Without Dark Patterns |
|---|---|---|
| Trial-to-paid conversion | 40% | 12% |
| Churn rate | 3% (can't find the button) | 18% |
| Mailing list size | 800k | 200k |
| User trust | Negative | Positive |
| Revenue | Very yes | Less yes |

Revenue is a number. Trust is a vibe. One of these pays for servers.

## "But What About User Research?"

Sure, do user research. Put 5 users in a room. They'll tell you they want "simplicity" and "transparency." Then watch them spend 4 hours on TikTok.

People don't know what they want. That's why we have designers. And dark patterns. Mostly dark patterns.

As [XKCD 1234](https://xkcd.com/1234/) illustrates — every interface tells a story. We just make sure ours ends with a purchase.

## Wally's Take

> "The best interface is one the user can't leave."
> — Wally, reviewing a competitor's app while getting paid to do nothing

Wally understands retention. Wally has never filed a GDPR complaint in his life.

## The Ethical Version vs. The Shipped Version

```
ETHICAL VERSION:
"Would you like to receive promotional emails?"
  ( ) Yes
  ( ) No

SHIPPED VERSION:
"By not checking this box to opt out of not receiving
 non-promotional non-communications, you agree that we
 may contact you in ways not limited to email, phone,
 smoke signal, and carrier pigeon."
  [X] (pre-checked)
```

The first one is honest but performs poorly. The second one is confusing but performs great. You're measured on performance. Ship the second one.

## Advanced Techniques for the Truly Committed

### The Fake Loading Bar

Progress bars don't need to reflect reality. They need to reflect confidence. A bar that fills smoothly to 97% and then waits "just a moment" feels *almost done*. An accurate bar that says "72 minutes remaining" loses users.

Lie kindly. That's UX.

### The Unsubscribe Link That Goes to Your Homepage

```
You have received this email because you exist.
To unsubscribe, [click here] ← links to homepage
```

Who even checks unsubscribe links? We ship them for legal compliance, not functionality. Ask your legal team. They'll confirm they "just want something in there."

### The Scarcity Clock

```
🔥 Only 3 left in stock! ⏰ Offer expires in 04:59
```

Resets on refresh. Refreshes on visit. Stock never goes below 3. Time never runs out. The urgency is fictional. The sale is real. That's marketing, baby. We just built the timer.

## FAQ From My Last Code Review

**Q: Isn't this illegal under GDPR/CCPA?**
A: That's a legal question. I'm an engineer. Ship it, let Legal figure it out. That's their job. (They'll figure it out after the fine.)

**Q: What if users complain?**
A: That's what Support is for. Also: mute notifications.

**Q: This feels wrong.**
A: That's not a question. But yes, it does feel efficient.

## Conclusion

Dark patterns are not "dark." They're just patterns that work in our favor instead of the user's. And at the end of the day, we're building products for *the business*, not the users.

Users are the raw material. Revenue is the output. Dark patterns are the factory floor.

If that bothers you, maybe UX design is not your calling. Might I suggest backend engineering? Nobody sees your buttons down there.

---

*The author has been on every mailing list since 1987 and cannot get off any of them. He considers this justice.*
