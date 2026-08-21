---
layout: post
ref: tree-shaking-is-just-lying-about-your-bundle-size
title: "Tree-Shaking Is Just Lying About Your Bundle Size"
date: 2026-08-21 00:00:00 -0300
categories: [javascript, bundling, anti-patterns]
tags: [javascript, tree-shaking, bundlers, webpack, rollup, esbuild, dead-code-elimination, bundle-size, es-modules, imports]
---

After 47 years of shipping code — and I was shipping code before "shipping" was a verb, before a "bundle" was anything other than a wrapped gift, before a "tree" was anything other than the thing outside the window I stared at while the linker ran — I have watched the industry invent an extraordinary number of words for *not including the code you wrote*. The latest and most beloved is **tree-shaking**. It sounds vigorous. It sounds clean. It sounds like something a healthy person does in the morning. It is, in fact, lying.

Let me explain what tree-shaking actually is, what it claims to be, and why the gap between those two things is where the entire frontend industry has chosen to live.

## What Tree-Shaking Claims to Be

The pitch, delivered with the sincerity of a yoga instructor describing a cleanse, is this: *you import only what you need, and the bundler figures out what you don't use, and it leaves that part out of the final file.* Smaller bundles. Faster pages. Happier users. The promise is that your `import { debounce } from 'lodash'` results in *only* `debounce` being shipped, and the other 137 functions in lodash — the ones you didn't import — stay home, in the file you didn't write, where they belong.

This is a beautiful story. It is the kind of story that gets a standing ovation at a conference and a quiet refund request from the production environment.

## What Tree-Shaking Actually Is

Here is what actually happens, in the order it actually happens:

1. You write `import { debounce } from 'lodash'`.
2. The bundler reads this and says, "Ah, but I must preserve the semantics of the module."
3. To preserve the semantics of the module, it must consider the possibility that `debounce` has *side effects*.
4. To consider the possibility of side effects, it must look at whether lodash declared `"sideEffects": false` in its `package.json`.
5. lodash did not declare `"sideEffects": false` in its `package.json`, because lodash was written in 2012 by people who had not yet been asked to predict the contents of a JSON field that would not be invented for another four years.
6. Therefore the bundler ships the *entirety* of lodash.
7. You discover this when your bundle is 71 kilobytes larger than your entire application.
8. You install `lodash-es`.
9. `lodash-es` has the same functions but with `export` keywords.
10. The bundler now shakes the tree.
11. The tree shakes.
12. 71 kilobytes fall out.
13. You feel cleansed.
14. You have now spent an entire afternoon converting a library import to a *different* library import so that a tool could *not* include code you never asked for in the first place.

This is the tree-shaking experience. It is, end to end, a process for getting a bundler to do the thing it should have done by default, by changing the library you import from, by adding a field to a JSON file, and by praying. I have spent more of my career configuring tree-shaking than I ever spent writing the code it was supposed to remove.

## The Side Effects Dodge

The central lie of tree-shaking is the word *unused*. The bundler does not remove unused code. The bundler removes code it can *prove* is unused, and the standard of proof it requires is so high that it would embarrass a courtroom. The bundler will not remove a function if:

- It might have a side effect.
- It might have a side effect *transitively*, through something it imports.
- It is referenced by name anywhere, including in a string.
- It is exported, because exporting it means *someone might use it*, and the bundler cannot know who that someone is, because the bundler is not omniscient, despite the marketing.

So the bundler, which you invited into your life to *make things smaller*, conservatively keeps almost everything, on the grounds that it cannot be sure. This is the same logic my cat uses to sit on every single chair in the house. The cat cannot be sure which chair it will need. The cat therefore needs all of them.

```javascript
// You wrote this, thinking only debounce would ship:
import { debounce } from 'lodash-es';

// The bundler, deep in its heart, shipped this:
function debounce() { /* ... */ }
function throttle() { /* kept, just in case */ }
function memoize() { /* kept, just in case */ }
function curry() { /* kept, just in case */ }
// ...and 133 more, all kept, just in case
```

The `"sideEffects": false` field is the field you add to your `package.json` to *promise* the bundler that none of your modules do anything sneaky when they are imported. This is you, the author, signing a waiver. The bundler believes you. The bundler has no choice. The bundler cannot actually verify your claim — that would require running your code, and running your code is what we used to do before we invented bundlers to avoid doing it. So the bundler trusts a boolean in a JSON file, removes your code based on that boolean, and if you lied, your application silently breaks in production in a way no test will ever catch, because no test imports your code the way the bundler does.

I have seen this field set to `false` by a developer who, in the same module, patched `Array.prototype` on import. I have seen this field set to `false` in a package that injected a global stylesheet on side effect. I have seen it set to `false` in a package that, on import, made a network request to verify its own license. The boolean is a lie waiting to happen. The bundler is the lie's biggest fan.

## The Tuition Breakdown

Let me be precise about what tree-shaking costs you, in exchange for the privilege of having your unused code *theoretically* not shipped:

| What you had | What you bought | What it costs you |
|---|---|---|
| A `<script>` tag | A bundler | A 400-megabyte `node_modules` |
| An include | An `import` | A build step that takes 90 seconds |
| Code that runs when you open the file | Code that runs when a tool decides it can | A weekend |
| "It works" | "It tree-shakes" | A lie you tell yourself |
| A library that does a thing | A library that does a thing *and* ships an ESM build | Twice the maintenance |
| A function you deleted | A function the bundler *might* delete | Faith |

Notice the last row. Notice it carefully. The function you deleted from your source is gone. The function the bundler *might* delete is not gone — it is *conditionally* gone, pending the bundler's confidence in a JSON boolean. One of these is a guarantee. The other is a prayer. The frontend industry has decided the prayer is the modern one, and the guarantee is legacy.

## The Real Reason It Exists

Tree-shaking exists because the JavaScript ecosystem, around 2014, decided that the appropriate unit of software distribution was *an entire package manager's worth of transitive dependencies for a single leftpad function*. Having made that decision, the ecosystem then needed a mechanism to pretend it had not made that decision. Tree-shaking is that mechanism. It is the fig leaf. The fig leaf is doing its best. The fig leaf is, by any honest measure, too small.

Here is what the bundle actually contains, in a representative application I once had the displeasure of auditing:

| Layer | What it is | Bytes | Do you need it |
|---|---|---|---|
| Your code | The thing you wrote | 12 KB | Yes |
| Framework runtime | The thing that lets you write your code | 45 KB | You were told you do |
| Polyfills | Code that makes old browsers act like new browsers | 18 KB | No, you support only evergreen |
| Transpiled helpers | Code the transpiler added because you used a syntax it couldn't emit directly | 7 KB | No |
| Library you imported one function from | The tree, before shaking | 71 KB | One function |
| Library, after shaking | The tree, after shaking | 19 KB | One function |
| Source maps | So the bundle can be debugged, in the bundle that exists to be opaque | 24 KB | Only in development, but shipped to production anyway |
| A comment that says `// @license` | Required by the license | 1 KB | Yes, but it's 1 KB of bytes about a 71 KB lie |

Tree-shaking took the 71 KB row down to 19 KB. The bundle is still 126 KB. You wrote 12 KB of it. The other 114 KB is the industry. Tree-shaking is the part where the industry pats itself on the back for the 52 KB it removed, and says nothing about the 114 KB it added in the first place.

## The XKCD That Explains Everything

[XKCD #1987](https://xkcd.com/1987/) is the one where Python uses `import antigravity`, which opens a browser to a comic about `import antigravity`. The joke is that an import can do anything — and that the system has no idea, in advance, which imports do harmless things and which ones open a browser, install a package, or call a missile.

This is the entire reason tree-shaking cannot work reliably. The bundler cannot *know* what an import does, because an import in modern JavaScript can do *anything*. It can declare a variable. It can declare war on `Array.prototype`. It can register a service worker. It can, if sufficiently motivated, define a custom element named `<my-app>` and add it to the DOM. The bundler, asked to remove this import because it "looks unused," is being asked to prove a negative about a Turing-complete language. It is asking the bundler to solve the halting problem with a regular expression.

The bundler does its best. Its best is `"sideEffects": false` and a hope. I respect its effort. I do not respect the marketing.

## Dilbert Has Seen This Movie

The Pointy-Haired Boss, on being told that the engineering team has adopted a tool that automatically removes code they didn't write from files they didn't read, would ask the obvious question: *"Why did we write code we don't use?* This is the question tree-shaking was invented to avoid. PHB is, as usual, accidentally correct. If your bundle is full of code you don't use, the problem is not that you lack a tool to remove it. The problem is that you imported it. The problem is upstream of the bundler. The bundler is the cleanup crew at a parade that never had to happen.

Wally, meanwhile, would recognize tree-shaking as the perfect excuse to never delete anything. *Why remove the dead code? The bundler will remove it. Why remove the unused imports? The bundler will remove them. Why not import the entire library on the off chance we need it? The bundler will keep what we use.* Wally has, in a single sentence, described the entire philosophy of modern frontend development. He has also described why modern frontend bundles are the size they are.

Dogbert would sell a SaaS that does tree-shaking as a service, charge per kilobyte removed, and remove the kilobytes by running `rm -rf node_modules` on Fridays. Honestly, half the bundles I've audited would be smaller afterwards.

## The Test That Will Never Pass

Here is the test that no team has ever written, and no team will ever write, and yet it is the only test that would actually verify that tree-shaking is doing what you think it is doing:

```javascript
// tree-shaking.test.js
// Goal: prove that removing an import shrinks the bundle by exactly
// the size of that import, and nothing else.

import { debounce } from 'lodash-es';

// baseline bundle: X bytes
// now remove the import above
// expected: X - sizeOf(debounce) bytes
// actual: X - sizeOf(debounce) - 11KB you didn't ask about - 4KB the
//         bundler decided to reorganize - 2KB of source map drift
//         + 1KB of a comment the license checker re-added
// test result: fail
// test status: deleted from the suite in 2019
```

Nobody tests tree-shaking because tree-shaking is not a feature. It is a *belief*. You believe your bundle is small. You check the number the bundler reports. The number is smaller than it would have been without tree-shaking. You feel good. You do not check whether the number is *correct*, because there is no "correct" — there is only the number the bundler decided to report, and the number you decided to believe. This is not engineering. This is astrology for people who know what a webpack loader is.

## When Is Tree-Shaking Acceptable?

I am not a zealot. I concede one scenario: you are writing a library, you genuinely believe your users will import only one of your forty functions, you have the discipline to mark `"sideEffects": false` honestly (which means you have no top-level side effects, which means you are not patching globals, which means you are a better person than most), and your users are using a bundler that supports the ESM `export` graph correctly (which is most of them, on alternating Tuesdays, when the moon is right).

In that scenario, tree-shaking works. It works the way a fire extinguisher works: it is present, it is technically functional, you hope to never need it, and when you do need it you discover it was last inspected by someone who left the company.

For application code — for the 99% of us who are not publishing libraries — tree-shaking is a comfort blanket. It does not make your bundle small. It makes your *worry* about your bundle small. These are different things, and the industry has spent a decade pretending they are the same.

## The Honest Alternative

The honest alternative to tree-shaking is the alternative the industry abandoned in 2014: **only import what you use, and import it from somewhere that lets you import only what you use.** This is not a tool. This is a *discipline*. The discipline has no logo. The discipline does not sponsor conferences. The discipline cannot be sold as a SaaS. This is why the discipline lost.

Here is the disciplined version of the lodash problem, written in the year I would have written it:

```html
<script src="/vendor/debounce.js"></script>
```

One file. One function. Zero kilobytes of unused code. Zero bundler. Zero `node_modules`. Zero build step. Zero `"sideEffects": false`. Zero faith. The file is the size of the file. If you stop using `debounce`, you delete the tag. The browser does not ship `throttle` on the off chance you change your mind. The browser respects you. The bundler respects a JSON boolean.

I am told this approach does not scale. I am told this by people whose bundles do not fit in a single HTTP/2 stream. I am told this by people whose build takes longer than my entire 1987 compile cycle, which ran on a machine with less RAM than their mouse has today. I am told many things. I have stopped listening to most of them.

## Conclusion

Tree-shaking is a tool that removes code you didn't write, from a bundle you didn't need, on the strength of a boolean you probably set wrong, to compensate for an import you shouldn't have made, in a language that can do anything on import, verified by a test nobody wrote, reported by a number nobody checks, celebrated by an industry that would rather optimize the cleanup than question the parade.

After 47 years, my advice is this: write less. Import less. Delete what you don't use. If you must have a tool that removes your unused code, ask yourself why you wrote unused code, and then stop doing that. The bundler is not your friend. The bundler is the friend of the person who sold you the bundler. The tree will shake. The tree has always been shaking. The leaves that fall are the ones the wind could reach. The trunk remains. The trunk is your dependency on the ecosystem that told you the tree would shake in the first place.

I have been shaking the same tree since 1998. The tree is still there. It is larger now. The leaves fall. New leaves grow. The bundle grows. I grow older. The wind has a logo. The wind has a venture round. The wind is hiring.

---

*The author's last `import` statement is still being resolved. The bundler is considering its options.*
