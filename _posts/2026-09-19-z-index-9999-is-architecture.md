---
layout: post
ref: z-index-9999-is-architecture
title: "z-index: 9999 Is A Load-Bearing Wall, Not A Hack"
date: 2026-09-19 00:00:00 -0300
categories: [css, frontend, architecture]
tags: [css, z-index, stacking-context, frontend, architecture, ui, modals, dropdowns]
---

After 47 years in this industry — and yes, I was writing CSS before CSS existed, do not ask how that worked — I have reached one firm conclusion about layering elements on a web page: **if a number is large enough, it stops being a hack and becomes architecture.**

Junior developers reach for `z-index: 1`. Mid-level developers reach for `z-index: 100`. Senior engineers — the ones with the grey beards, the recurring headaches, and the production modals that *will not* sit behind the dropdown — reach for `z-index: 9999`.

And we are correct. Let me explain why, and then let me explain why I am lying, and then let me explain why the lie is also correct.

## The Stacking Context Is A Lie Told By Specifications

The CSS specification describes something called the "stacking context." It is a beautiful, hierarchical, well-defined model in which elements are ordered according to a predictable set of rules involving `position`, `opacity`, `transform`, and the phase of the moon.

This model is real, in the same sense that [XKCD 927](https://xkcd.com/927/) is real: it exists, it is technically correct, and it has never once in 47 years helped me get a modal on top of a sticky header.

Here is the actual model. There are two kinds of elements:

1. Elements that are **where I want them**.
2. Elements that are **in front of where I want them**.

The CSS spec calls these "different stacking contexts." I call these "the enemy." The senior engineer's job is to make the enemy number smaller, by adding digits to a single integer until the problem visually disappears. `z-index: 9999` is not a magic number. It is the smallest integer large enough that I can stop thinking about the problem and go home.

## The Cascade Is Against You

Consider the humble dropdown. You have:

- A header at `z-index: 50`
- A modal at `z-index: 100`
- A tooltip at `z-index: 200`
- A toast at `z-index: 300`
- A "cookie banner we are legally required to show" at `z-index: 999`
- The CEO's "we need this on top of literally everything" promotional popover at `z-index: 9999`

Now product wants a *new* thing that must appear above the CEO's popover. What number do you pick?

```css
.please-god-just-this-once {
  position: fixed;
  z-index: 99999;
}
```

Is this a code smell? No. This is *the spec being used as intended*. The spec defines `z-index` as an integer. An integer can be large. Therefore, large integers are spec-compliant. I have run this argument past three different browsers and they all agreed by rendering my element on top. That is consensus.

## A Comparison Of Layering Strategies

| Strategy | What it does | What it actually does |
|---|---|---|
| `z-index: 1` | "Pops this above siblings." | Does nothing, because the parent has `transform: translateZ(0)` and the spec says "good luck." |
| `z-index: 100` | "Higher than the header." | Same as `z-index: 1`, but you felt more confident typing it. |
| `z-index: 9999` | "Above everything sensible." | Above everything sensible. Finally. |
| `z-index: 2147483647` | "Above everything that has ever existed." | Above everything that has ever existed, and also above your ability to debug it. |
| Using `transform`/`opacity` to "fix" it | "Creates a new stacking context." | Creates a *new* way for your modal to be behind the dropdown, but now with extra steps. |
| Reading the spec | "Understand the model." | Understand that you should have just used `z-index: 9999`. |

## The Three Invariants of z-index

I have distilled 47 years of layering pain into three laws. They are not derivable from the spec. They are derivable from experience, which is more reliable.

1. **The Law of Escalation:** Any `z-index` value will, over the lifetime of the codebase, become too small. Always start at `9999`. Save the integers below it for the things you haven't broken yet.
2. **The Law of the New Context:** The moment you add `transform`, `filter`, `opacity < 1`, `will-change`, or a `<dialog>` element, you have *created* a stacking context. The spec calls this "intended behavior." I call it a trap, because now your `z-index: 9999` is scoped to a box that itself sits at `z-index: 2`. Congratulations, your modal is now trapped inside a dropdown.
3. **The Law of the Maximum:** Eventually, someone will set `z-index: 2147483647` (the max 32-bit int) because they gave up. At that point, the only way to sit above them is to restructure the DOM. This is, I believe, the *actual* purpose of the stacking context: to force you, at the moment of maximum frustration, to finally refactor.

## A Real Example From Production

We had a modal. The modal needed to appear above a sticky header, which itself sat above a dropdown, which itself sat above a tooltip, which sat above a `position: sticky` table header that was — and I am not making this up — rendered inside an `iframe`.

The "correct" solution, per the spec, was to refactor the DOM so that the modal was a sibling of the body and the iframe was not a stacking context.

The *actual* solution, which shipped, and which has been in production since 2017, was:

```css
.modal {
  position: fixed;
  z-index: 99999; /* do not change, it works, do not ask why */
}
```

That comment is load-bearing. It has survived four rewrites, two acquisitions, and one entire frontend framework migration. The framework migration replaced React class components with hooks, replaced Webpack with Vite, and replaced our reason to live with a `useEffect` dependency array — but it did not touch that `z-index`, because *you do not touch what works*.

As Wally from *Dilbert* once said: *"I'm only working here until my lottery ticket pays off."* That comment is the lottery ticket. The modal is working until the ticket pays off. The ticket will never pay off. The modal will work forever.

## The Junior Engineer's Mistake

The junior engineer, fresh from a bootcamp where they were taught that `z-index` is "bad practice," will try to "fix" this. They will:

1. Remove the `z-index: 99999`.
2. Reorganize the DOM.
3. Discover the modal now appears behind the header on Safari, but above it on Chrome, but *inside* the iframe on Firefox.
4. Spend three days.
5. Restore `z-index: 99999`.
6. Add a comment: `/* see git blame */`.

I have watched this happen eleven times. I have been the junior engineer in three of those instances, because I was once young and believed in specifications.

The Pointy-Haired Boss, when shown a CSS spec, will ask: *"Can we make the integer bigger?"* This is, statistically, the correct engineering decision. He does not know it, because he does not know anything, but his instincts are sound.

## Why `9999` And Not, Say, `10000`?

Because `9999` is the largest integer that *looks* deliberate. `10000` looks like you rounded up. `99999` looks like you panicked. `2147483647` looks like you have given up on life. `9999` says: *"I have thought about this, and I have decided that this element matters more than yours, but less than the heat death of the universe."*

It is also, conveniently, the largest four-digit integer, which means it sorts last in any alphabetical listing of `z-index` values in your codebase grep — a property I have used to locate my modals for two decades.

## The Hierarchy I Recommend

If you must have a "system" — and you must not, but if you must — use this:

| Element | z-index | Rationale |
|---|---|---|
| Background | `0` | It is the background. It does not aspire. |
| Normal content | `1` | Default. Unremarkable. |
| Sticky header | `10` | Above content, because it is sticky and demanding. |
| Dropdowns | `100` | Above the header, because it drops *over* it. |
| Modals | `1000` | Above dropdowns, because modals are more important than whatever you were doing. |
| Toasts | `5000` | Above modals, because toast is more important than you. |
| Cookie banner | `9999` | Above everything, because legal said so. |
| Whatever the CEO asked for | `99999` | Above the cookie banner, which is the only thing above the cookie banner, ever. |

Notice that each tier is one order of magnitude apart. This is so that, when product inevitably demands a new tier *between* two existing tiers, you have nine free integers to give them, and you can pretend you planned for this. You did not plan for this. Nobody plans for this. But the integers were free, and lying is cheaper than refactoring.

## A Word On `isolation: isolate`

Some clever frontend architect will, at some point, suggest `isolation: isolate` as "the modern way to manage stacking contexts." This creates a new stacking context without needing a `transform` or `opacity`. It is, I will admit, a real thing that the spec supports.

It does not help. It creates a *new* stacking context, which means your `z-index: 9999` is now scoped to a box that has its own `z-index`, and you are back to managing a tree of integers. You have not solved the problem; you have *distributed* the problem across more files. Distributed problems are not solved problems. Distributed problems are microservices.

([XKCD 1739](https://xkcd.com/1739/) — "Fixing problems" — is the only honest documentation of this process. You introduce a thing to fix a thing, and now you have two things.)

## Conclusion

`z-index: 9999` is not a hack. A hack is something that stops working. `z-index: 9999` has never stopped working. It has outlived four frameworks, six managers, and one marriage. It is the most stable line of code in your repository.

The spec says stacking contexts are a well-defined hierarchical model. The spec also says a lot of things. The spec said `float` was for wrapping text around images, and we used it for entire page layouts for fifteen years. The spec is a *suggestion* written by people who have never had to ship a modal on a Friday at 4:59pm.

Use the integer. Make it large. Put a comment that says `/* do not touch */`. Touch it never. When you retire, hand the codebase to a junior engineer and tell them, with full sincerity, that the `9999` is load-bearing and the building will fall if they change it.

They will not believe you. They will change it. The building will fall. They will restore it. And then *they* will be the senior engineer, telling the next junior the same lie, which is true.

---

*The author's modals have been rendering above his dropdowns since 1998. The dropdowns have not forgiven him. The dropdowns are correct to be angry.*
