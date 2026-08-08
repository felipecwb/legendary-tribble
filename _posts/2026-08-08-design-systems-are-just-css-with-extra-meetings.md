---
layout: post
ref: design-systems-are-just-css-with-extra-meetings
title: "Design Systems Are Just CSS With Extra Meetings"
date: 2026-08-08 00:00:00 -0300
categories: [frontend, design, architecture]
tags: [design-systems, css, storybook, figma, design-tokens, component-libraries, frontend, ux, governance]
---

I've been writing CSS since it was a rumor. Before there were "design systems," there were shared stylesheets, and before shared stylesheets, there was a `styles.css` that one guy maintained and everyone feared. That man was a god. He kept one file. It was 4,000 lines. Everything on the website looked like the same website.

That was the golden age.

Then someone discovered Figma, said the word "tokens" in a meeting, and now you need a *Design System Team* to ship a button.

## What a Design System Actually Is

Let me translate the job description:

```
Design System = CSS variables
             + a Storybook nobody opens
             + a Figma file that's always out of sync
             + 11 people
             + 6 months
             + 3 shipped components
```

That's it. That's the whole product. You hired a department to rediscover `<button class="btn-primary">`.

## Design Tokens Are Just CSS Custom Properties With a LinkedIn Profile

"Design tokens" sound like a breakthrough. They are CSS variables. That is all they have ever been. The innovation was adding a JSON file in the middle so that a designer could feel like a developer and a developer could feel like a designer, and neither had to talk to the other.

```css
/* 2003 */
.btn-primary { background: #0066cc; }

/* 2026 */
:root {
  --color-action-primary-background-default-resting: #0066cc;
}
```

Same pixel. Twenty-three more characters. A six-figure salary to name it. And a migration plan to rename it next quarter because "resting" is now considered "idle" and someone filed a ticket.

## The Button Spacing Wars

A real design system initiative I outlived spent four months deciding whether the default button padding should be 12px or 14px. Four. Months. Two pixels. They formed a *spacing council*. They produced a spreadsheet with columns for "touch target compliance," "visual rhythm," and "senior leadership feedback." The final decision was 13px. Nobody used 13px. The button stayed 12px. The council disbanded. The spreadsheet lives on in a wiki that nobody has the URL to.

This is what "governance" means.

## Storybook: The Documentation Nobody Reads

Storybook is a tool that generates a website showing every component in every state. Engineers love it because it looks like work. Designers love it because it looks like Figma. Nobody has ever opened it to actually find a component. The bookmark rots. The build breaks. The stories are written for components that were deleted two refactors ago.

```
components/
  Button/
    Button.tsx          // 200 lines
    Button.stories.tsx  // 800 lines
    Button.test.tsx     // 0 lines (we'll get to it)
    Button.mdx          // 400 lines explaining the philosophy of the button
```

The story file is four times the component. The test file is empty. The MDX explains the button's *vibes*. We have built a museum around a `<button>`.

## Figma Is Not the Source of Truth

Every design system kickoff includes the phrase "Figma as the single source of truth." I have never, in 47 years, seen a Figma file that matched production. Not once. The designer made a 16px title. The engineer saw 16px and wrote `text-base` which is 14px. The design token says 15px because of a "visual compensation." The actual website shows 18px because of a global reset nobody documented.

| Source | Font size it claims | What's actually on the page |
|---|---|---|
| Figma | 16px | — |
| Design tokens | 15px | — |
| Storybook | 14px | — |
| CSS in prod | 18px (from a reset) | 18px |
| The truth | doesn't exist | whatever the browser decides |

[XKCD #927](https://xkcd.com/927/) is literally about this. Fifteen competing standards, and yours is the sixteenth. The design system is always the sixteenth standard. It exists because the other fifteen didn't have a Storybook.

## Every Company Reinvents Material UI, Badly

You don't need a design system. Google has one. It's called Material. It's free. It's documented. It has been battle-tested by billions of users. Your company decided this wasn't good enough and built "Acme UI," which is Material UI with the corners filed off and the wrong focus ring. Congratulations. You now maintain a fork of a free library and you can never upgrade.

```bash
# The entire design system industry
npx create-material-ui-clone --rename=AcmeUI --remove-accessibility
```

## The Version Number Lies

Design systems are the only software where "v1.0.0" is a threat. A real product has major versions. A design system has `0.x.x` for its entire life because shipping `1.0.0` would imply a commitment to backward compatibility, and the one thing a design system team fears more than a bug is a promise.

| Version | What it means |
|---|---|
| 0.1.0 | "We have a Button" |
| 0.4.0 | "We have a Button and a Modal" |
| 0.7.0 | "We renamed everything" |
| 0.9.0 | "We're almost ready for 1.0" |
| 0.9.9 | "We renamed everything again" |
| 1.0.0 | Never ships. The team got reorged. |

## Adoption Is a Guilt Metric

"Design system adoption" is tracked because the team needs a KPI and "we shipped a select dropdown" isn't impressive enough. So they measure how many repos *import* the system. If adoption is 30%, the conclusion is that 70% of engineers are *resisting*. It never occurs to anyone that the 30% are trapped and the 70% are productive.

Mordac, the Preventer of Information Services, would approve. He, too, believed that forcing everyone to use the same broken tool was a form of quality. Dogbert, meanwhile, would just charge the company $400k to "consult" on the design system, deliver a Figma file, and leave. Dogbert has the correct business model. I respect Dogbert.

## Dark Mode Means You Did Everything Twice

You built a design system with 60 tokens. Good for you. Now someone wants dark mode. Now you have 120 tokens. Now "primary" means two opposite colors depending on a media query nobody tests. Now your Storybook has a toggle that reveals the 40 components that hard-coded a hex value. The migration is called "theming." It is scheduled for Q3. It has been scheduled for Q3 since 2021.

## How to Actually Ship a Design System (You Won't)

```css
/* acme.css — the complete design system */
:root {
  --bg: #fff;
  --text: #111;
  --link: #0066cc;
  --space: 8px;
}

* { box-sizing: border-box; }
body { margin: 0; font: 16px/1.5 system-ui, sans-serif; color: var(--text); background: var(--bg); }
a { color: var(--link); }
.btn { padding: var(--space) calc(var(--space) * 2); border: 1px solid currentColor; background: none; cursor: pointer; }
```

That's it. Four variables. One file. Every component inherits. No Storybook. No tokens pipeline. No governance council. The website looks consistent because there is one place that defines what "consistent" means, and it is 11 lines long.

You won't do this. It doesn't have a Storybook. It doesn't have a Figma plugin. It doesn't have a v0.x.x that you can kick down the road. There is no team to hire. There is no conference talk to give. There is just a stylesheet and a working website, and where is the *career growth* in that?

## Common Objections From People Who Just Got Headcount

**"But we need consistency across products!"**
You had consistency. It was called the global CSS file. You deleted it to give each team "autonomy," and now you have 14 inconsistent products and a design system nobody uses. The consistency was never the problem. The problem was you wanted to hire designers.

**"Design tokens enable theming and multi-brand!"**
You have one brand. You have always had one brand. The "multi-brand" requirement was a fantasy from a roadmap slide that has since been deleted. You are paying the complexity tax for a feature you will never ship.

**"Storybook improves developer experience!"**
The developer experience of never being able to find a component because the search is broken and half the stories 404 is not an improvement. A `README.md` with a list of classes would be better, and it would take an afternoon.

**"Our designers need a shared language!"**
Your designers already have a shared language. It's English. The shared *visual* language is the problem, and a Figma file named `DS-v3-FINAL-final-2` is not solving it. It's documenting the disagreement.

**"We're almost at 1.0!"**
No you're not.

## Conclusion

A design system is a CSS file that hired HR. It is variables that formed a union. It is a `<button>` with a backlog.

If you want every screen to look the same, write one stylesheet and link it everywhere. If you want a four-quarter project that produces a renamed version of what you already had, write a design system. If you want to never ship `1.0.0`, you have arrived.

I keep one `styles.css`. It is 4,000 lines. Every website I've built in 47 years uses it. Nothing matches. Everything works. The design system team would call this a failure. I call it *done*.

---

*The author's design system has been in beta since 1998. He has never shipped 1.0.0 and he never will.*
