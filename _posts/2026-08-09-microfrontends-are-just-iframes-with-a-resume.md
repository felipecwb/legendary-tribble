---
layout: post
ref: microfrontends-are-just-iframes-with-a-resume
title: "Microfrontends Are Just Iframes With a Resume"
date: 2026-08-09 00:00:00 -0300
categories: [frontend, architecture]
tags: [microfrontends, iframes, monoliths, frontend, webpack-module-federation, web-components, architecture, frontend-teams, complexity]
---

I have shipped frontends in every generation of this industry. I shipped `<frameset>`. I shipped `<iframe>`. I shipped the single-page app. I shipped the single-page app that loaded four other single-page apps inside it. I have come to understand that "frontend architecture" is a circle, and we are doomed to walk it forever, renaming the same bad idea each time we pass the start line.

The current name for the bad idea is *microfrontends*. It is `<iframe>` with a conference talk.

## What a Microfrontend Actually Is

Let me translate the architecture diagram into English:

```
Microfrontend = One iframe
              + a webpack config nobody understands
              + a team that owns a button
              + a shared auth library that's four weeks out of date
              + a runtime that breaks when any team sneezes
              + 300KB of JavaScript to render a login form
```

That is the whole pattern. You took a monolith, broke it into five monoliths, and called it "scalability." The browser now loads five `<script>` tags, four CSS resets, and three copies of React. Your Lighthouse score is a number that requires therapy.

## The iframe Was Already the Microfrontend

The `<iframe>` tag has existed since 1997. It embeds one web page inside another web page, fully isolated, with its own JavaScript, its own styles, its own DOM. It is, by every definition, a microfrontend. It is also the thing the microfrontend movement insists it is *not*, because admitting you reinvented the iframe would end the conference circuit.

```
1997: <iframe src="header.html"></iframe>           // bad, apparently
2026: <MicrofrontendHeader manifest="header.json" />  // innovative, definitely
```

Same DOM. Same network waterfall. Same cross-origin headaches. New résumé.

## Module Federation Is a Build System That Owes You an Apology

Webpack Module Federation is the technology most teams pick when they decide they hate themselves. The pitch is "one team's build can import another team's code at runtime." The reality is "your build now depends on a remote server being up at build time, runtime, and every moment in between."

```js
// webpack.config.js — the entire microfrontend industry
new ModuleFederationPlugin({
  name: "shell",
  remotes: {
    header: "header@https://cdn.example.com/header/remoteEntry.js",
    footer: "footer@https://cdn.example.com/footer/remoteEntry.js",
    auth:   "auth@https://cdn.example.com/auth/remoteEntry.js",
  },
  shared: {
    react: { singleton: true, requiredVersion: "the-one-the-header-team-picked" },
  },
});
```

That `shared.react` line is where the pattern dies in production. The header team is on React 18.2. The footer team is on React 18.3. The auth team "doesn't believe in major versions" and is still on 17.0.2. Your shell loads all three. React, a library whose entire value proposition is "one reconciler," now has three reconcilors arguing over the same DOM tree. The console fills with warnings. The buttons stop working. Nobody knows which team owns the bug, which is, I grant you, a perfect outcome for the microfrontend philosophy.

## A Team Per Button

The dream of microfrontends is "team autonomy." Each team ships independently. No coordination. No shared backlog. The header team doesn't wait on the footer team. Beautiful. The cost is that you now need a team for the header and a team for the footer.

I have consulted at a company with *eleven frontend teams* for a product whose entire UI fits on one screen. There was a Cart Team, a Cart Button Team, a Cart Tooltip Team, and a Cart Tooltip Accessibility Team that hadn't shipped since the reorg. The Cart Tooltip Accessibility Team's entire quarterly goal was to "align on a definition of tooltip." They had a wiki. The wiki was 14 pages. There was no tooltip.

This is what "autonomy" means. It means nobody is responsible for the user seeing a tooltip, because four teams own pieces of it and each points at another.

## The Shared Dependency Trap

Every microfrontend guide contains the sentence "share as little as possible." Every microfrontend guide is then ignored, because sharing nothing means shipping React eleven times, and shipping React eleven times means your homepage weighs more than the 1997 Encarta CD-ROM.

| Strategy | What it sounds like | What it is |
|---|---|---|
| Share nothing | "Full autonomy!" | 11 copies of React, 4.2MB homepage |
| Share React | "Sensible baseline" | The header team's React upgrade breaks the footer team for 3 weeks |
| Share a design system | "Consistency!" | A 6-month project to agree on a button (see my last article) |
| Share everything | "We're pragmatic" | A monolith with extra steps |

There is no winning row in this table. The moment two microfrontends share *anything*, you have reintroduced the coordination the pattern was supposed to eliminate. The moment they share *nothing*, you have reintroduced the monolith's problems plus a network round-trip.

## Routing Is Now a Negotiation

In a monolith, the router is one file. You open it. You read it. You know which URL goes where. In a microfrontend shell, routing is distributed across five teams, each with their own router, each believing it owns `/`. The shell team thinks `/account` is theirs. The auth team thinks `/account` is theirs. Both register it. The last one to load wins. Which one is last depends on CDN caching, which depends on which team deployed most recently, which depends on whether it's Tuesday.

I have seen a production homepage where the header and the body both rendered `/`, producing two complete websites stacked vertically, like a turducken of JavaScript. The fix was a meeting. The meeting produced a spreadsheet. The spreadsheet produced a "routing ownership document." The document was outdated the day it was written, because the header team renamed their route in a hotfix and didn't tell anyone, because autonomy.

[XKCD #1737](https://xkcd.com/1737/) shows two people trying to fix a program that's gotten out of hand. The caption is "Fixing problems every day, forever." That is the microfrontend router. You are fixing problems every day, forever, and each fix creates a new one in a team you've never spoken to.

## CSS Is Now a War

In a monolith, CSS is a war. In a microfrontend system, CSS is *five wars*, each with its own front, each unaware the others exist, all fought over the same `<body>`.

You will be told to use CSS scoping. Shadow DOM. CSS Modules. Styled-components. Each microfrontend picks its own. The header uses Shadow DOM, which sounds isolated until you remember that the dropdown menus need to escape the shadow boundary to overlay the body, so every team opens a "portal," and now you have portals inside iframes inside shells, and the z-index of the user's tooltip is set to `2147483647` because someone read a blog post and that's the max int and *surely* nothing will ever be higher.

```css
/* header team */
.tooltip { z-index: 9999; }

/* footer team, three weeks later */
.tooltip { z-index: 99999; }

/* auth team, who "doesn't believe in z-index" */
[role="dialog"] { z-index: 999999999; }

/* the intern, pushed to the brink */
.cookie-banner { z-index: 2147483647; }

/* me, in 1997, with a frameset */
.banner { z-index: 1; }  /* and it worked. */
```

The intern is correct. The intern has rediscovered the 1997 solution. The rest of you are reinventing the problem.

## The Shell Is a Monolith

Here is the part nobody puts in the conference talk: the *shell* — the thing that loads all the microfrontends — is a monolith. It has a router. It has auth. It has a build. It has a release cadence. It has a team that owns it. The shell team is now the bottleneck the microfrontends were supposed to eliminate, because every microfrontend needs the shell to mount it, and the shell team has a backlog, and the shell team is on vacation.

You did not escape the monolith. You built a monolith and gave it sublets.

The Pointy-Haired Boss would look at this architecture and say, "So we have one big app and several small apps, and the small apps can't ship without the big app?" And for once in his life, the PHB would be right. Wally, meanwhile, would recognize the shell team as the perfect job: nothing ships without you, so you do nothing, and nothing ships. Wally has been a microfrontend architect since 1997. He just didn't have a title for it.

## If You Must (You Must Not)

If you are determined to ship a microfrontend, ship it as an iframe and be honest with yourself. The iframe is the original microfrontend. It is isolated. It has its own DOM. It has its own styles. It has its own global scope. It cannot leak. It cannot be styled from outside. It loads independently. It is, by every metric the microfrontend papers care about, the superior implementation.

```html
<!-- the complete, honest microfrontend architecture -->
<iframe src="/header/index.html"></iframe>
<iframe src="/body/index.html"></iframe>
<iframe src="/footer/index.html"></iframe>
```

Three lines. Zero build step. No Module Federation. No shared React. No routing negotiations. Each iframe is owned by one team. Each team deploys independently. The browser — a piece of software refined over 30 years by people far smarter than your frontend guild — handles the isolation for you.

You won't do this. There is no conference talk titled "Iframes: They Still Work." There is no framework to adopt. There is no `microfrontend-cli` to install. There is no résumé line that reads "Migrated to iframes." There is just an HTML tag and a working website, and where is the *career growth* in that?

## Common Objections From People Who Just Booked a Conference

**"But teams need to deploy independently!"**
They did. It was called "a monolith with feature flags." You replaced it with "five apps that can't deploy without the shell," and now the header team waits three sprints for the shell team to mount their iframe. You invented a slower release train and called it velocity.

**"Microfrontends let us use different frameworks per team!"**
This is not a feature. This is a confession. The reason you "need" React in the header and Vue in the footer is that you hired two teams that refuse to learn each other's tool. The fix is management, not architecture. Dogbert would charge you $600k to tell you this. I am telling you for free, and you will still not listen.

**"Module Federation gives us runtime composition!"**
Runtime composition is a fancy word for "the build fails at a time the build is not supposed to run." You have moved your compile errors from the CI server to the user's browser. Congratulations. You have made the production environment a participant in your build pipeline. Mordac would be proud — he, too, believed users should suffer for engineering convenience.

**"We need to scale the frontend!"**
The frontend does not need to scale. It is a document. You are not serving ten million concurrent users with your React tree; you are serving ten million concurrent users with your CDN, which serves a static file. The scaling problem is imaginary. The complexity problem it created is real.

**"Big companies use microfrontends!"**
Big companies also use 400-person meetings and reorg every six months. Do not take organizational advice from a company whose org chart is on fire.

## Conclusion

A microfrontend is an iframe that went to business school. It is the `<frameset>` of my youth, rebranded by people who were not alive when `<frameset>` was the standard, and who will rebrand it again in 2030 as "edge-rendered island components" or whatever the next name for "put one webpage inside another webpage" turns out to be.

I have one HTML file. It has one `<body>`. It has one stylesheet. It loads one script. It renders one website. It has done so since 1998. The microfrontend movement would call this "not scalable." I call it *done*. The user, who has never once asked for a microfrontend and never will, calls it *fast*.

Put the iframe back. Admit what you built. Go home.

---

*The author has been rendering one body tag since 1998. It still works. He has never been invited to give a talk about it.*
