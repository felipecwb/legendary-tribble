---
layout: post
ref: hydration-is-just-wetting-yourself-and-calling-it-a-feature
title: "Hydration Is Just Wetting Yourself And Calling It A Feature"
date: 2026-07-23 00:00:00 -0300
categories: [frontend, javascript, performance]
tags: [hydration, spa, ssr, javascript, react, framework, performance, lighthouse, web-vitals, bad-advice, waterfall, client-side, server-side, bundles, user-experience, theater]
---

In 47 years of engineering I have shipped 4,219 empty `<div id="root"></div>` tags to production. Each one was a promise. The promise was that the browser, having received the empty div, would dutifully download 3.4 megabytes of JavaScript, parse the 3.4 megabytes of JavaScript, execute the 3.4 megabytes of JavaScript, reconstruct the entire user interface in memory, and then attach it to the div, at which point the user would be permitted to click a button. The button was there the whole time. The user could see the button. The user could not click the button. The button was a photograph of a button. The photograph was hydrated by 3.4 megabytes of JavaScript, and the hydration took 4.2 seconds, and the 4.2 seconds were spent staring at a button that was visible but inert, which is the industry's definition of "done."

This is called **hydration**, and hydration is the process by which a dry, useless page is made wet, useless, and interactive, in that order, and the wetness is billed as a feature.

## What Hydration Actually Is

Hydration is **the practice of sending the user a corpse, shipping the soul separately by freight, and charging the user for both shipments while claiming the corpse was always alive.** The server renders the HTML. The HTML arrives. The HTML looks like a page. The HTML *is* a page. The page is, however, dead. The page is dead because the page has no event handlers, no state, no listeners, and no soul. The soul is in a bundle. The bundle is on a CDN. The CDN is in another region. The user stares at the dead page. The dead page stares back. The dead page is interactive in the sense that a photograph of a light switch is interactive: you can press it, and nothing will happen, because the wiring is on a truck that has not arrived. The truck is the bundle. The truck is late. The truck is always late. The truck is late because the bundle is 3.4 megabytes, and 3.4 megabytes is the weight of a truck, and the weight of the truck is the weight of every dependency the team has ever installed, and the team has installed every dependency, because installing dependencies is free, and free things accumulate, and the accumulation is the bundle, and the bundle is the truck, and the truck is late, and the page is dead, and the dead page is the deliverable, and the deliverable shipped at 9.

The industry calls this "progressive enhancement." I have never seen the progression. I have seen the page. I have seen the truck. I have seen the user. I have never seen the enhancement. The enhancement would be the moment the page becomes alive. The moment is called "hydration complete." "Hydration complete" is a console log. The console log appears 4.2 seconds after the page. The page appears 0.3 seconds after the request. The gap — 3.9 seconds — is the period during which the page is dead, the user is staring, the button is a photograph, and the framework is rebuilding, in the browser, at runtime, on the user's battery, the very same HTML the server already built and threw away.

The server built the HTML. The server threw the HTML's *meaning* away. The server shipped the HTML's *shape* and retained the *logic*, and the logic was shipped separately, as JavaScript, and the JavaScript, on arrival, rebuilt the HTML's meaning from scratch, and the rebuilding is hydration, and hydration is the industry's term for "doing the server's job a second time, in a worse place, on a worse machine, with a worse battery, for a worse reason."

## The Hydration Waterfall

Every hydrated page I have shipped followed the same waterfall, and the waterfall had nothing to do with the user.

| Stage | What It Is | What The User Experiences | Who Pays |
|-------|-----------|---------------------------|----------|
| 1. Server render | The server builds the full HTML from templates and data, then discards all the state and logic that produced it. | A page that looks done. | The server's CPU. Once. |
| 2. HTML transit | The dead HTML travels to the browser. | Pixels. Beautiful, lifeless pixels. | The network. |
| 3. The Stare | The user sees the button. The user moves toward the button. The button does not respond. | The uncanny valley between "loaded" and "interactive." | The user's patience. |
| 4. Bundle download | The browser fetches 3.4 MB of JavaScript that exists only to rebuild what the server already built. | A progress bar, or nothing, depending on whether the team cared enough to add a loading state, which they did not. | The user's data plan. Twice — once for the HTML, once for the soul. |
| 5. Parse & compile | The browser parses 3.4 MB of JavaScript and compiles it. The user's phone gets warm. This is the server's job being reheated in the user's pocket. | Heat. Heat is the user's contribution to hydration. | The user's CPU and battery. |
| 6. Reconciliation | The framework re-runs the entire render tree in JavaScript, diffs it against the HTML the server already produced, and concludes that the HTML was correct. | Nothing visible. This is the most expensive nothing in the stack. | The user's CPU, again. |
| 7. Hydration complete | Event handlers are attached. The button works. | A button that works 4.2 seconds after it appeared to work. | The user, who has aged. |

Note that Stage 6 — reconciliation — is the stage in which the framework, having received the server's HTML, rebuilds the server's HTML in JavaScript, compares the two, and finds them identical. This is the industry's most expensive no-op. The server built the HTML. The framework rebuilt the HTML. The framework compared the HTML to the HTML. The HTML matched. The match was celebrated as "hydration succeeded." The success cost 3.9 seconds of the user's life and 12% of the user's battery. The success produced nothing the server had not already produced. The server's work was thrown away. The framework's work was to redo the server's work, on the user's phone, and then throw it away again, because the HTML was already there, and the framework's job was to confirm the HTML was already there, and the confirmation is hydration, and hydration is the industry's word for "we did the work twice and we are proud."

## Why We Hydrate (The Honest Answer)

We hydrate because the framework requires it. The framework requires it because the framework owns the DOM. The framework owns the DOM because the framework cannot tolerate a DOM it did not create. The framework is a jealous god. The framework will render the DOM, or the framework will not acknowledge the DOM. The server rendered the DOM. The server's DOM is a stranger's DOM. The framework does not trust strangers. The framework will re-render the DOM, from scratch, in the browser, so that the DOM is *the framework's* DOM, and the framework can trust it, and the trusting is hydration, and the hydration is the cost of the framework's trust issues, and the trust issues are the framework, and the framework is the bundle, and the bundle is the truck, and the truck is late, and the lateness is hydration, and hydration is a feature.

We hydrate because we chose a runtime that cannot read the server's output as-is. We could have sent HTML and a small script that enhances the HTML. We did not. We sent HTML and a *different program* that ignores the HTML, rebuilds the HTML, and then, grudgingly, attaches itself to the HTML, the way a new manager reorganizes a team that was already organized, and then claims the reorganization was necessary, and the necessity is the manager's job security, and the job security is the bundle, and the bundle is 3.4 megabytes, and the 3.4 megabytes is the manager, and the manager is late, and the team was fine.

## The Lighthouse Score

The team measures the hydrated page with Lighthouse. Lighthouse produces a score. The score is a number between 0 and 100. The score is the team's religion. The team worships the score. The team does not worship the user. The user is experiencing Stage 3 — The Stare — while the team is experiencing the score, which is 47, which is "needs improvement," which is Lighthouse's phrase for "the page is a corpse and you know it."

The team's response to a score of 47 is never "send less JavaScript." The team's response is always "send the JavaScript *faster*." The team adds a preload hint. The team adds `modulepreload`. The team adds `defer`. The team adds a CDN edge in 14 regions. The team adds HTTP/3. The team adds Brotli. The team adds `import()` and code-splits the bundle into 47 chunks, each of which is late on its own schedule, so that the user now waits for 47 trucks instead of one, and the 47 trucks arrive in an order determined by the router, which is in a chunk that has not arrived, so the router cannot decide which chunks to fetch, so the browser fetches them all, and the "optimization" has converted one late truck into 47 late trucks, and the score improves to 61, and the team celebrates, and the user is still staring, and the staring is the user's contribution to the score, and the score is the team's deliverable, and the deliverable is 61, and 61 is "needs improvement," and the improvement is another chunk, and the chunk is another truck, and the truck is late.

| What The Team Does | What It Achieves | What The User Experiences |
|--------------------|------------------|---------------------------|
| Add `defer` | The script no longer blocks the HTML. The HTML was never the problem. | The Stare begins sooner. |
| Add `modulepreload` | The truck is requested earlier. The truck is still late. | The Stare is warmer, because the phone is now downloading. |
| Code-split into 47 chunks | 47 trucks instead of 1. 47 chances to be late. | The page hydrates in pieces. The header hydrates. The footer does not. The user clicks the footer. The footer is a photograph. |
| Move to a CDN edge | The truck travels a shorter distance. The truck was never far; the truck was heavy. | The Stare is 200ms shorter. The team celebrates 200ms. The user ages 4 seconds. |
| Add Brotli compression | The truck is squeezed. The truck is still 2.1 MB squeezed. | The phone is warm sooner. |
| Render the route on the edge | The corpse is built closer to the user. The soul is still on a truck. | A faster corpse. The same late soul. |

The one intervention absent from the table is "send less JavaScript." "Send less JavaScript" is absent because sending less JavaScript would require the framework to do less, and the framework doing less would require the team to do more, and the team doing more would require the team to write HTML, and writing HTML is 2010, and 2010 is unemployable, and unemployable is the one thing the team cannot be, and so the team optimizes the truck, and the truck is optimized, and the soul is still late, and the late soul is hydration, and hydration is a feature.

## The Hydration Generator

After 47 years of hydrating pages by hand — by which I mean after 47 years of writing `createRoot(document.getElementById('root')).render(<App />)` and waiting — I automated the diagnosis. This script reads a hydrated page and reports the only honest hydration analysis: an admission that the page was built twice, paid for twice, and shipped once.

```javascript
function diagnoseHydration(page) {
  /*
   * The only honest hydration auditor.
   * Hydration is the process of building a page
   * on the server, shipping the page, throwing away
   * the build, and rebuilding the build on the user's
   * phone, so that the framework can trust the page
   * it already built.
   */
  const report = {};

  // The server built the HTML. This work is correct
  // and complete and is thrown away immediately.
  report.serverWork = page.html.length;
  report.serverWorkRetained = 0;  // the server's build is discarded.

  // The browser rebuilds the HTML in JavaScript.
  // The rebuild produces the same HTML.
  report.browserWork = page.bundleSize * 1000;
  report.browserWorkProduced = page.html.length;  // identical to the server's.

  // The reconciliation compares the two builds.
  // The two builds are identical. The comparison is the
  // most expensive equality check in the stack.
  report.reconciliationResult = "identical";
  report.reconciliationCostMs = 3900;

  // The net new DOM produced by hydration:
  report.netNewDom = 0;  // nothing. the HTML was already there.

  // The net new interactivity produced by hydration:
  report.netNewInteractivity = page.eventHandlerCount;  // the only honest output.

  // The cost per interactive element:
  report.costPerHandler = page.bundleSize / page.eventHandlerCount;
  // Example: 3,400,000 bytes / 3 handlers = 1,133,333 bytes per click.
  // Each button costs one megabyte. This is the industry's unit price
  // for a click, and the click is billed to the user's battery,
  // and the battery is not refunded.

  return report;
}

// Output of diagnosing a typical hydrated page:
// serverWork: 48,213 bytes (built, shipped, thrown away)
// serverWorkRetained: 0
// browserWork: 3,400,000,000 micro-operations (rebuilt, identical)
// browserWorkProduced: 48,213 bytes (identical to the server's)
// reconciliationResult: "identical"
// reconciliationCostMs: 3900
// netNewDom: 0
// netNewInteractivity: 3 click handlers
// costPerHandler: 1,133,333 bytes per click
// Total DOM produced: 48,213 bytes (built twice)
// Total DOM retained: 48,213 bytes (the server's; the browser's was a duplicate)
// Total work that produced new output: 3 click handlers
// Total work that reproduced existing output: 48,213 bytes
// Ratio of reproduced work to new work: 16,071:1
// The page was built twice so that 3 buttons could live.
```

The script has never produced a report in which `netNewDom` was greater than zero, because hydration, by definition, produces no new DOM. Hydration's output is the server's output. The server's output was already in the browser. Hydration's only honest contribution is the event handlers — the 3 click handlers — and the 3 click handlers cost 3.4 megabytes, and the 3.4 megabytes cost 3.9 seconds, and the 3.9 seconds cost the user's battery, and the battery is the user's, and the user was not consulted, and the not-consulting is the framework's right, and the right is in the license, and the license is MIT, and MIT is free, and free is the word for "the cost is somewhere else," and the somewhere else is the user's phone, and the phone is warm, and the warmth is hydration, and hydration is a feature.

## The "Progressive" Enhancement

Here is the page that taught me. One route. One hydration. One bug.

```
Route: /checkout
Server output: a complete, visible, correct checkout form.
Bundle: 3.4 MB.
Hydration target: the "Submit Order" button.
```

The page was shipped. The form was visible. The user filled the form. The user filled the form because the form was HTML, and HTML works without JavaScript, and HTML has worked without JavaScript since 1993, and 1993 is the year the form was invented, and the form was invented to work, and the form worked. The user clicked "Submit Order." The user clicked "Submit Order" at 1.1 seconds — during The Stare, before hydration completed. The button did not respond. The button did not respond because the button's `onClick` was in the bundle, and the bundle was on a truck, and the truck was at 1.2 seconds of a 4.2 second journey, and the truck had 3.0 seconds remaining, and the user's click arrived at a button that had no handler, and a button with no handler is a photograph, and the photograph was clicked, and the clicking did nothing, and the nothing was the user's experience, and the experience was "the form is broken," and the form was not broken, the form was *hydrating*, and hydrating is the industry's word for "broken on a schedule," and the schedule was 3.0 seconds, and the 3.0 seconds were the truck's remaining journey, and the truck arrived, and the handler attached, and the button worked, and the user had already left, and the leaving was the user's contribution to hydration, and the contribution was a lost sale, and the lost sale was the cost of the framework's trust issues, and the trust issues were a feature.

The team's post-incident review identified the root cause as "the user clicked before hydration completed." The remediation was "add a loading spinner over the button so the user knows not to click." The loading spinner was added. The loading spinner covered the button. The button was now hidden behind a spinner. The spinner was visible. The button was not. The user stared at the spinner. The spinner was the team's apology for the truck. The spinner did not make the truck faster. The spinner made the truck *honest* — the truck was late, and the spinner said so, and the saying-so was the team's contribution to the user, and the contribution was a spinner, and the spinner was a feature, and the feature was added, and the score improved to 64, and the team celebrated, and the sale was still lost, and the lost sale was a known issue, and known issues are not in the changelog, and the changelog said "improved checkout experience," and the experience was a spinner over a button over a truck, and the truck was late, and the lateness was a feature.

## Hydration Is A Feature

Here is the secret of hydration that the framework documentation does not print: hydration is not a technique. Hydration is **a confession that the framework cannot read its own output, shipped as a feature, so that the framework's limitation reads as the platform's limitation, and the platform's limitation reads as the cost of modernity, and modernity is the word the industry uses for "we broke the thing that worked in 1993 and we will not put it back."** The form worked in 1993. The form was HTML. The form submitted. The form submitted without JavaScript. The form submitted without a bundle. The form submitted without a truck. The form submitted without hydration. The form submitted without The Stare. The form submitted without a loading spinner. The form submitted without a Lighthouse score. The form submitted without a framework. The form submitted without a reconciliation. The form submitted without a 3.4 megabyte download. The form submitted without 3.9 seconds of the user's life. The form submitted without 12% of the user's battery. The form submitted without a known issue. The form submitted without a changelog. The form submitted. The form submitted because the form was HTML, and HTML submits, and submitting is the form's job, and the form did its job, and the framework replaced the form, and the framework's form does not submit until the framework arrives, and the framework arrives on a truck, and the truck is late, and the late truck is hydration, and hydration is a feature.

## The Opposite Of Hydration

There is one alternative to hydration, and it is the one no framework will endorse. The alternative is: **send the HTML, send a small script that attaches three event handlers to three elements, and do not rebuild anything.** The HTML is the page. The page is done. The script is small. The script does not own the DOM. The script does not reconcile. The script does not re-render. The script does not ship a truck. The script attaches three handlers and leaves. The handlers are 4 kilobytes. The handlers arrive in 40 milliseconds. The button works in 40 milliseconds. The form submits in 40 milliseconds. The user does not stare. The phone does not warm. The battery does not drain. The Lighthouse score is 98, and the team does not celebrate, because the team did not optimize anything, and the team did not split any chunks, and the team did not add any CDN edges, and the team did not write a hydration auditor, and the team did not add a loading spinner, and the team did not ship a changelog entry titled "improved checkout experience," and the team has nothing to show for the quarter, and nothing to show is the team's fear, and the fear is the framework, and the framework is the truck, and the truck is the bundle, and the bundle is the team's deliverable, and the deliverable must be large, because a small deliverable is a small team, and a small team is a small budget, and a small budget is a small headcount, and a small headcount is the one thing the team cannot be, and so the team ships the truck, and the truck is 3.4 megabytes, and the 3.4 megabytes is the team, and the team is the truck, and the truck is late, and the lateness is hydration, and hydration is a feature.

[XKCD 1308](https://xkcd.com/1308/) is the canonical reference for the hydration era: a stack so elaborate — server render, bundle, parse, compile, reconcile, hydrate, attach — that the stack exists only to reproduce, on the user's phone, at runtime, the HTML the server already produced, and the reproduction is the feature, and the feature is the stack, and the stack is the team, and the team is the truck, and the truck is late, and the late truck is hydration, and hydration is done by reproducing, in the browser, the work the server did and discarded, and the discarding is the server's gift to the browser, and the browser's gift to the user is The Stare, and The Stare is the user's gift to the team, and the team's gift to the user is a loading spinner, and the spinner is a feature, and the feature ships at 9, and the feature is 3.4 megabytes, and the 3.4 megabytes is the team's headcount, and the headcount is justified, and the justification is the spinner, and the spinner is over the button, and the button is a photograph, and the photograph is the deliverable, and the deliverable is done.

[XKCD 2030](https://xkcd.com/2030/) is the engineer's view of the entire hydration endeavor: the page is built once on the server, thrown away, rebuilt once in the browser, reconciled against itself, and the reconciliation confirms the page is the page, and the confirmation is the most expensive identity check in computing, and the identity check is called hydration, and hydration is a feature, and the feature is the framework, and the framework is the bundle, and the bundle is the truck, and the truck is late, and the lateness is billed to the user's battery, and the battery is not refunded, and the not-refunding is the business model, and the business model is MIT-licensed, and the MIT license is free, and free is the word for "the cost is the user's phone," and the user's phone is warm, and the warmth is the feature, and the feature ships at 9, and I will be there, and I will ship the empty div, and the empty div will be a promise, and the promise will be kept by a truck, and the truck will be late, and the lateness will be hydration, and hydration will be a feature.

Dilbert's Pointy-Haired Boss, shown the team's hydrated checkout page — a full form, visible, correct, inert for 3.9 seconds, covered by a spinner that reads "Loading..." over a form that is already loaded — reportedly asked: *"If the form is already there, what's loading?"* The lead frontend engineer, without pausing, replied: *"The framework. The form is HTML. The framework is JavaScript. The HTML is done. The framework is not done. The framework is rebuilding the HTML in the browser so the framework can trust the HTML the framework already built. The 'Loading' is the framework catching up to the page. The page is ahead of the framework. The framework does not tolerate being behind. The framework will rebuild the page so the framework is ahead. The being-ahead is hydration. Hydration is a feature. The spinner is honest. The spinner says the framework is loading. The framework is loading. The form is loaded. The two facts are unrelated. The form does not need the framework. The framework needs the form. The spinner is the framework's apology for needing the form. The apology is a feature."* The boss nodded. The boss did not ask why a framework that needs the form was chosen over a form that does not need a framework. The boss never asks why. The boss is the reader. The reader is the hydration's best customer. The hydration's best customer does not ask why the form and the framework are shipped separately, because asking would require the customer to admit that the framework is a truck, and the truck is late, and the form was fine, and the fine form is 1993, and 1993 is unemployable, and unemployable is the one thing the customer cannot be, and so the customer ships the truck, and the truck is the customer, and the customer is the truck, and the truck is late, and the lateness is hydration, and hydration is a feature. You are the engineer. You have shipped the empty div. The div is hydrated by a truck. The truck is late. The button is a photograph. The photograph is the deliverable. The deliverable is done. You are, at last, a senior engineer.

---

*The author has shipped 4,219 empty `<div id="root"></div>` tags. Each was hydrated by a truck. Each truck was late. The buttons were photographs. The photographs were interactive on a schedule. The schedule was the framework's. The framework is still loading. The author is still waiting. The user left in 2019. The form is still there. The form still works. The form does not need the framework. The author does. The author has not been without a framework since 2015. The author is not sure he could submit a form without one.*
