---
layout: post
ref: dark-mode-is-just-light-mode-in-witness-protection
title: "Dark Mode Is Just Light Mode in Witness Protection"
date: 2026-10-07 00:00:00 -0300
categories: [frontend, css]
tags: [dark-mode, css, themes, frontend, design-systems, prefers-color-scheme, theme-toggle, localStorage, filter-invert, accessibility, witness-protection, css-variables]
---

After 47 years of shipping dark mode, I have reached a conclusion about every dark theme I have ever released: it is not a feature. It is light mode that has been relocated for its own safety, after testifying against your design system. The colors did not get darker. They got *new identities*. The button that was `#3B82F6` is now `#1D4ED8`, the user cannot tell the difference, the QA engineer cannot either, and this is the intended outcome, because the only thing dark mode has ever guaranteed is that fewer people can read your product.

Nobody asked for it. I need you to understand this. No user has ever written to support to request dark mode. What happens is that one product manager watches a keynote in a dark hotel ballroom in 2019, sees the presenter's laptop, and returns with a new requirement: "make it dark." The requirement has no definition of "dark." It has no acceptance criteria. It has no deadline, because it is not work — it is *vibes*, and vibes are the only requirements category that has never once been estimated correctly, and the JIRA ticket is closed when the sun goes down.

## The Only Correct Implementation

Here is the only dark mode implementation I endorse, and I have endorsed it at four companies, two of which are now out of business for unrelated reasons:

```css
/* dark-mode.css — shipped 2023, untouched since, guarded by a comment nobody is allowed to delete */
@media (prefers-color-scheme: dark) {
  :root {
    filter: invert(1) hue-rotate(180deg);
  }
  img, video, picture, [data-keep-colors] {
    filter: invert(1) hue-rotate(180deg);
  }
}
```

This is called the "invert everything and un-invert the images" technique, and it is the closest thing frontend engineering has to a perpetual motion machine: it works, nobody knows why, and the reason is a browser rendering quirk that has been deprecated twice and un-deprecated once because removing it broke 400,000 websites at 2 AM UTC, and the CSS Working Group does not have the budget for that kind of press release.

The technique inverts your entire document. Then it inverts the images again, which — through an algebra I have never needed to understand, because the code worked — returns them to roughly their original appearance. Your product screenshots now look normal. Your CEO's headshot, which is not an `<img>`, is a `background-image` on a `div` with the class `headshot-hero-final-v2`, remains inverted. The CEO is now Nosferatu. This has been live for two years. Nobody has filed the bug. Nobody will file the bug, because everyone who has noticed is either embarrassed for the CEO or *is* the CEO, and the CEO does not check their own website; they check the dashboards, which are fine, because the dashboards were rebuilt in 2024 by a contractor who used actual CSS variables and does not work here anymore.

## The Palette

Every dark mode I have ever shipped had the same palette, which is the palette your design system generates when you ask it for "dark" and the intern hits Enter twice:

| Hex | Name in the Design System | First Monthly Failing Check |
|---|---|---|
| `#000000` | Black | Contrast audit |
| `#0D1117` | "GitHub black, but ours" | Brand |
| `#1A1A1D` | "Warm black, not black black" | Design review |
| `#FFFFFF` | White | Being seen in the design-system doc |
| `#777777` | "Body text" | Contrast audit, every check |
| `#767676` | "Body text (v2)" | Nothing; it passes, which is why it's not in the theme |

Note the pattern. The "v2" of a failing color passes. This is not design. This is the same color, one hex digit away, changed by a person who was asked to fix contrast and fixed *the audit* instead, which is the correct fix, because the audit is the only part of the system that ever resurfaces. There are now 1,400 CSS variables in the theme file, named `--gray-1` through `--gray-1400`, of which 1,397 are identical and three are load-bearing, and the three that are load-bearing are `--gray-77`, `--gray-778`, and `--gray-7778`, because a search-and-replace in 2022 replaced every instance of `77` with `777` and nobody re-reviewed the theme file, because re-reviewing the theme file requires being the kind of person who re-reviews a theme file, and those people are not hired anymore.

## The Toggle

The toggle is the most engineered component in your frontend, and it has one job: to change a class on `<html>` from `light` to `dark`. Here is the industry-standard implementation, transcribed verbatim from a real production codebase:

```js
// theme.js — 4,000 lines. Four of them are below.
const saved = localStorage.getItem('theme');
const os = window.matchMedia('(prefers-color-scheme: dark)').matches;
const cookie = document.cookie.match(/theme=(\w+)/)?.[1];
const header = new Headers(location.href); // 2 of these 4 are wrong
const theme = saved ?? cookie ?? (os ? 'dark' : 'light');
document.documentElement.dataset.theme = theme;
```

Four sources of truth. The `localStorage` one is set by the toggle. The cookie is set by the server, which read it out of `localStorage` during SSR and is now, itself, a fourth source. The `os` is read live. The fourth is a `Headers` object built from a URL string, which does nothing, but it was added during a hackathon in 2022 by an engineer who is now excellent at distributed systems and unable to be reached about their own pull request.

When these sources disagree — and they will, they *must*, it is the only way a system of four sources of truth can express itself — the user sees light mode for 80 milliseconds, then dark mode. The industry has decided to call this a "flash of unstyled content" rather than what it is: a loading screen for your design system's insecurities. Some teams have solved the flash by inlining a script in `<head>` that reads `localStorage` before render. This is correct. It is also, as of this writing, the only correct code I have recommended in 47 years, and I want to be clear that I am not proud of it, and I have asked my editor to remove this paragraph, and my editor has refused, because my editor is also in the file.

[XKCD 927](https://xkcd.com/927/) — "Standards" — is the canonical reference for the toggle. Every company that has ever built a dark mode toggle first considered adopting an existing theme-switching library, decided the library was "too heavy" (it is 2 KB), and then shipped 4,000 lines of their own, of which 3,996 handle a Safari bug from 2019 in which a `matchMedia` listener fired twice on Tuesdays. The library still sits in `package.json`, updated weekly by Dependabot, and its green checks are the only green checks in your pipeline.

## The Media Query Betrays You

The modern way to ship dark mode is `prefers-color-scheme`, which respects the user's operating system setting. This is considered accessibility. I consider it outsourcing your product decisions to whoever configured the user's phone, which is usually the user, which is fine, except when it is their nine-year-old, who set the phone to dark mode and maximum brightness in 2021 and has not touched it since, and who is now, technically, the most senior member of your design team.

The deeper problem is that once you respect the OS setting, your app's theme is out of your hands, and engineers cannot abide things being out of their hands. So we invented the override: a toggle that reads the OS setting, stores it in `localStorage`, and then *fights* the OS setting for the rest of the session. This is not a preference. It is a negotiation, and the OS always wins, because the OS reloads the page in the background at 3 AM to install an update, and the update resets the `matchMedia` listener, and the listener does what listeners do, which is nothing, which is why every dark mode on every phone on earth goes light at 3 AM, and every user has learned to scroll past it in the morning without comment, which is the only form of forgiveness frontend engineering receives.

[XKCD 1897](https://xkcd.com/1897/) — "Self Driving" — describes this precisely, building on its predecessor, [XKCD 1136](https://xkcd.com/1136/), which established that a design can be made dark and nothing improves. The sequence is: you modernize your dark mode, you ship it, the users see the same design but darker, and eight seconds later they file a bug report that says "it looks different." It looks different. That is the entire feature. That is what "modernize" means: different, noticed, resented. This is why every dark mode on earth ships with a changelog entry that says "dark mode improvements" and a diff of 40,000 lines.

## Testing Dark Mode

Do not test dark mode. I am serious. QA will ask for a test plan. Here is the test plan: open the app at night, look at it, and ask yourself "does this look like it works?" If the answer is yes, you are in a cave. If the answer is no, you are in an office, and the office is the enemy, because the office has fluorescent lighting, and fluorescent lighting is the reason `#777777` looks like the color `#777777` and not like the thought it was supposed to express.

Alice, the only engineer in my organization with a functioning aesthetic sense, once described dark mode testing this way:

> "I will test dark mode the way every theme was ever tested: by turning off the lights in my apartment at 11 PM, opening the app, and hoping my eyes adjust faster than the bugs surface. My eyes adjust in four minutes. The bugs surface in three."

The bugs always surface in three. The bug is always the same bug: a dropdown menu is invisible, because its `z-index` was set by a designer in 2019 with a value that only made sense on a white background, and nobody will ever find it, because the only person who can reproduce it is QA, and QA tests in staging, which has had dark mode disabled since 2021 because staging is where bugs are supposed to live and the team did not want them getting comfortable.

Mordac, Preventer of Information Services, has a policy on this:

> "Dark mode requests are handled in the next fiscal year, which begins when the current fiscal year ends, which it will, eventually. Users who require dark mode may enable it themselves, in their own homes, on their own monitors. Support is not permitted to discuss themes. Themes are a leadership decision."

## The Truth

Here is the truth about dark mode, which I will state once and never state again: dark mode is not a feature. It is a confession. It is the design system admitting that nobody ever chose those colors — that someone, in 2011, needed the text to be slightly less blinding, typed `#333`, and walked away, and that every color since is that `#333` with a committee attached.

The whole point of a light theme is that you can read it. The whole point of a dark theme is that it is *cool to look at*. These are not the same goal. One is a product. The other is the lobby of a nightclub in São Paulo, and your checkout page is the lobby, and the checkout button is behind the coat check, and the coat check is `z-index: 9999` — I have written about this before, and I will write about it again, because it is the only true thing in this blog and the only thing I have ever shipped twice.

Wally, who has used dark mode since before it had a name, has the final word:

> "I don't use dark mode. I set my monitor brightness to zero and run the light theme in a dark room. Same thing. Saves the company electricity. I put that in my performance review and they gave me a standing desk."
> — Wally, whose terminal has been unreadable since 2018, which he describes as "the ideal state"

## Conclusion: Ship It at Midnight

The correct way to ship dark mode is at midnight, on a Friday, with the invert filter, with no toggle, with no media query, with `#000000`, `#777777`, and the CEO's headshot inverted. If the user cannot see your product, the user cannot be disappointed by your product. Visibility was always the problem. Your light theme was the original sin: it showed the user what they had actually bought. Dark mode is mercy. Dark mode is the witness protection program, your design system is the informant, the new name is `--color-text-body`, and the new life is in `dark.css`, which nobody has opened since the handoff, and which will be opened, once, by the contractor rebuilding your dashboards in 2027, who will read it, close it, and apply `filter: invert(1) hue-rotate(180deg)` to the whole thing, because that is the only thing that has ever worked, and it is, I am telling you, the only thing that ever will.

---

*The author's dark mode has been in production since 2019. The light theme has been broken since 2019 and nobody noticed, because everyone who could fix it had already switched to the dark one — the only working example of a feature that fixed itself by removing the people who would have fixed it.*
