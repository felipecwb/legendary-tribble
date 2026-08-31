---
layout: post
ref: css-frameworks-are-just-someone-elses-bad-css
title: "CSS Frameworks Are Just Someone Else's Bad CSS You're Paying For"
date: 2026-08-31 00:00:00 -0300
categories: [frontend, css, web]
tags: [css, frameworks, tailwind, bootstrap, frontend, utility-classes, specificity, technical-debt, dependencies]
---

After 47 years of writing software — including 30 years of writing CSS before it existed, which is a story for another time — I've reached a conclusion the frontend cult cannot accept:

**CSS frameworks are just someone else's bad CSS.**

You didn't escape CSS. You imported 40,000 lines of someone else's CSS, named it "Tailwind," and congratulated yourself for not writing CSS. You wrote CSS. You just wrote *their* CSS, and you did it by typing 14 utility classes on every `<div>` like some kind of typer's compensation for a decision you outsourced in 2019.

The React people are already composing a 4,000-word Medium rebuttal. The Tailwind evangelists are reaching for `!important` in their hearts. The Bootstrap loyalists — yes, all three of them — are adjusting their carousel components. Let them. They've never had to upgrade a framework that decided to rename every class in a major version and called it "tree-shaking."

## The Grand Illusion Of Not Writing CSS

Here's the pitch every framework makes: *"Stop writing CSS. Use our classes. It's faster."*

Here's what actually happens:

```html
<div class="flex flex-col items-center justify-center
            min-h-screen bg-gray-100 px-4 py-8
            sm:px-6 md:px-8 lg:px-12
            text-sm sm:text-base md:text-lg
            font-sans font-medium tracking-tight
            rounded-lg shadow-md hover:shadow-lg
            transition-shadow duration-200
            border border-gray-200 border-solid
            focus:outline-none focus:ring-2
            focus:ring-blue-500 focus:ring-offset-2">
  Hello
</div>
```

That's one `<div>`. It contains the word "Hello." The class attribute is 364 characters. The content is 5. You have written more CSS than the entire stylesheet of a 1998 GeoCities page, and you've done it *inline*, on *every element*, *forever*.

But sure, you "don't write CSS." You write utility classes. Which are CSS. That someone else wrote. That you import. That you can't read without a cheat sheet. That you will re-learn every two years when the maintainers decide `flex-shrink-0` is now `shrink-0` and call it "an improvement."

## The Comparison Table They Don't Want You To See

| Concern | Hand-written CSS | Tailwind | Bootstrap |
|---|---|---|---|
| Lines of CSS you write | ~200 | 0 (you write 14,000 lines of class names instead) | 0 (you fight their specificity instead) |
| Can you read your markup | Yes | No, it's a wall of utility soup | No, it's `class="col-md-8 offset-md-2 mx-auto d-flex justify-content-center"` |
| Upgrade path | Rename a few classes | Relearn 80% of the API | Switch to Tailwind |
| Bundle size | 4 KB | 6 KB (purged) / 3.5 MB (the CDN) | 187 KB of decisions you didn't make |
| Time to center a div | 10 seconds (`margin: auto`) | 4 seconds (`mx-auto`) | 30 seconds (find the right utility, realize it's `justify-content-center`, not `justify-center`, curse) |
| Specificity wars | You caused them, you can fix them | They're hidden in `!important` you can't find | They are the framework |
| Who is responsible for your styles | You | A GitHub repo in Portland | A GitHub repo that peaked in 2016 |

Notice the "time to center a div" row. This is the entire frontend industry's origin myth: "we made centering a div easier." They did. They made it *four seconds faster* and charged you *the rest of your career* in return. The div was never the hard part. The hard part was agreeing with your designer. No framework solves that. The framework just gives you something new to disagree about.

## Why "Utility Classes" Is Just "Inline Styles" In A Trenchcoat

The defense of Tailwind is: *"It's not inline styles. It's composable. It's atomic."*

Let me show you what inline styles look like:

```html
<div style="display: flex; flex-direction: column;
            align-items: center; justify-content: center;
            min-height: 100vh; background: #f3f4f6;
            padding: 32px; border-radius: 8px;
            border: 1px solid #e5e7eb;">
  Hello
</div>
```

Now let me show you what Tailwind looks like:

```html
<div class="flex flex-col items-center justify-center
            min-h-screen bg-gray-100 p-8 rounded-lg
            border border-gray-200">
  Hello
</div>
```

These are the same thing. One uses CSS property names. The other uses abbreviations of CSS property names. One the browser ignores at runtime if you delete it. The other lives in a 4 MB generated stylesheet that you must rebuild whenever you change a class. The inline style is *more honest*. It admits what it is. Tailwind wears a fake mustache and pretends to be architecture.

As [XKCD 927](https://xkcd.com/927/) established fifteen years ago and the frontend industry has spent fifteen years not reading: every new "standard" to replace the existing fourteen standards just becomes the fifteenth. Tailwind is the fifteenth. It replaced Bootstrap, which replaced Foundation, which replaced 960 Grid, which replaced inline tables. Each one promised to end CSS suffering. Each one became CSS suffering, with a dependency tree.

## The Real-World Example That Proves Everything

A team I worked with — I'll call them "the team" — decided to adopt Tailwind to "stop writing custom CSS." Six months later:

1. Their markup had an average of **11 utility classes per element**.
2. They had a `tailwind.config.js` that was **340 lines long** customizing every color, spacing, and breakpoint.
3. They had written **22 custom utility classes** in a `@layer` block because Tailwind didn't have what they needed.
4. They had **three Tailwind plugins** for forms, typography, and aspect-ratio.
5. Their build step took **47 seconds** to purge and generate the final CSS.
6. No one could edit a component without the Tailwind docs open.
7. A junior asked "how do I make this red on hover" and the answer was `"hover:text-red-500 unless the parent has group-hover in which case group-hover:text-red-500 and also you need group on the parent."` The junior quit.

They had replaced ~600 lines of readable, namespaced, semantic CSS with **11 classes × 400 components = 4,400 class tokens** that no search could refactor, no linter could safely rename, and no human could hold in their head. When the designer changed the primary color, they had to find-and-replace `blue-600` across 1,200 files. In hand-written CSS, this is *one variable*.

This is called "progress."

## What Dilbert's Cast Would Say

> **Wally:** "I use Tailwind because it means I never have to think about CSS. I also never have to think about my markup, my colleagues, or my retirement. It's a complete system."

> **Dogbert:** "CSS frameworks exist to make engineers feel they've solved a problem by relocating it. The problem is now in your class attributes, your config file, and your build step. Congratulations, you've tripled the surface area of the problem and called it a reduction."

> **Mordac, the Preventer of Information Services:** "I have mandated Tailwind across all projects. Component consistency is up 40%. Build times are up 600%. Developer happiness is down, but that was already the case, so it doesn't count."

> **The Pointy-Haired Boss:** "Can we just use the CSS? The one from the website?" (He is the only person in the building with a coherent position.)

## The "But What About Consistency?" Question, Answered Once And For All

The framework zealots will say: *"But without a framework, every developer writes different CSS and we have no consistency!"*

You don't have consistency *with* a framework either. You have the *illusion* of consistency, because everyone is using the same class names to express *different intentions*. `flex` means eleven different things across your codebase. So does `p-4`. So does `text-sm`. The names are consistent; the *meaning* is not.

Real consistency comes from **a small set of named, documented components** — `.button`, `.card`, `.modal` — that have a single definition and a single purpose. This is what a framework gives you *if you use it as components*. It is not what Tailwind gives you. Tailwind gives you Lego bricks and asks you to build the same button 900 times. The fourth junior will build it slightly different. The 901st button will be wrong. The consistency was never in the framework. It was in the discipline, which you outsourced because discipline is hard.

[As XKCD 1513](https://xkcd.com/1513/) reminds us, the moment you depend on someone else's library, you have adopted their bugs, their release schedule, and their opinions about how a `Button` should be styled. They will change all three. You will update. This is the cycle. There is no exit except writing your own CSS, which you were trying to avoid because it is, apparently, *too hard*.

## The Long-Term Architecture

Eventually your frontend looks like this:

```
Your markup → 11 utility classes per element
Your config → 340 lines of Tailwind customization
Your plugins → 3 (forms, typography, aspect-ratio)
Your build → 47 seconds to generate 6 KB of CSS from 4,400 class tokens
Your juniors → cannot edit anything without the docs
Your seniors → defending the decision in every code review
Your designers → asking why "the blue" is slightly different in every component
Your bundle → 6 KB CSS + 2.3 MB JS for a button that says "Click"
```

The hand-written CSS team has a 4 KB stylesheet, a 40-line `_variables.css`, and a junior who learned `.button` in 30 seconds. Their build is instant. Their designer is happy. They are, however, *embarrassed* at conferences because they "don't use a framework." This is the real cost of hand-written CSS: social. The technical cost is zero. The social cost is enormous. So we pay the technical cost to avoid the social cost, because we are, after all, primates.

## Summary, But It's A Class Name

| Principle | Stance |
|---|---|
| Writing CSS | Do it. It's 200 lines. You'll survive. |
| Using a framework | You've imported 40,000 lines of someone else's CSS and called it "not writing CSS." |
| Utility classes | Inline styles in a trenchcoat. |
| Consistency | Comes from named components, not from atomic bricks. |
| Build step | Should not take 47 seconds to produce 6 KB. |
| The div | Was never the hard part. |
| Your dignity | Located in `tailwind.config.js`, line 214, and it is a custom gray. |

If your solution to "CSS is hard" is "import someone else's CSS and type 14 classes per element," you have not solved CSS. You have *subleased* CSS. The landlord is a GitHub repo in Portland. The lease is a semver contract. The rent is your build time. And the eviction notice arrives every major version, in the form of a migration guide that renames half your class names and asks you to be grateful.

I write my own CSS. It is 200 lines. It has been the same 200 lines since 2014. My buttons are consistent because I have one `.button` class. My junior learned it in 30 seconds. My build is instant. I am, however, not invited to frontend conferences. This is a cost I have accepted.

---

*The author has been writing CSS since before it was called CSS. His stylesheet is 214 lines and has never been rebuilt. He considers this a personality trait.*
