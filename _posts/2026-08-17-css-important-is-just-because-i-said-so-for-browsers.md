---
layout: post
ref: css-important-is-just-because-i-said-so-for-browsers
title: "CSS !important Is Just \"Because I Said So\" for Browsers"
date: 2026-08-17 00:00:00 -0300
categories: [css, frontend, anti-patterns]
tags: [css, important, specificity, cascade, stylesheets, frontend, dark-mode, z-index]
---

After 47 years of writing CSS — don't check my math, that predates CSS by decades, I just *needed* it to predate CSS — I've arrived at one unshakable truth: the cascade is a lie invented by people who wanted your buttons to be blue.

They'll tell you to "embrace the cascade." They'll tell you to "respect specificity." They'll tell you to "never use `!important`." What they mean is: *they* want to win the argument about whose styles take precedence, and they're hoping you'll fight them with their own rules instead of just declaring victory.

I have a different approach. I call it the **Because I Said So™** school of styling. It has one rule, and it goes at the end of every declaration.

```css
.button {
    color: red !important;       /* I said so */
    background: blue !important;  /* also this */
    z-index: 9999 !important;     /* ESPECIALLY this */
}
```

Let me show you the light. It's prefixed with an exclamation mark, and it has never once lost an argument.

## What Is Specificity, And Why Is It An Opinion?

Specificity is the system CSS uses to decide which rule wins when two rules disagree. It is calculated by counting IDs, classes, elements, and a cleric's attunement to the moon. The exact formula is:

> (a × 100) + (b × 10) + (c × 1) + (d × 0) + (e × "depends") + (f × "nobody actually knows")

In practice, specificity means: whoever wrote the deepest selector at 2am after three beers wins. This is why every senior developer's stylesheet eventually contains `#main #content #sidebar .widget .box .inner div span a.link`. They are not being specific. They are being *desperate*.

[XKCD #927](https://xkcd.com/927/) is the comic where every new standard just adds another competing standard. CSS specificity is this comic, but the competing standards are all in the same file, and the winner is whoever shouts loudest. `!important` is the shout. The shout always wins.

## The Cascade Is Peer Pressure

The cascade works like this: later rules override earlier rules, unless the earlier rule is more specific, unless the later rule has `!important`, unless the earlier rule also has `!important`, in which case whoever shouted last wins, unless a user stylesheet intervenes, in which case — you know what, here's a table:

| Situation | Who wins | Why |
|---|---|---|
| Two equal rules | The last one | "recency bias" |
| One is more specific | The specific one | "meritocracy" (fake) |
| One has `!important` | The important one | "because I said so" |
| Both have `!important` | The last important one | "loudest, latest" |
| Three have `!important` | You, presumably | "you've lost control" |
| Inline style | Inline | "the HTML is judging you" |
| Inline + `!important` | Inline important | "the author gave up" |

Notice the pattern: every escalation of the conflict ends with someone writing `!important`. The cascade is just `!important` with extra steps. Why take the stairs when the elevator goes straight to the top?

## The War Escalation Timeline

Here is how a junior developer's stylesheet evolves. You can watch the cascade collapse in real time:

**Week 1:** Clean class-based selectors. Everything works. The developer believes they understand CSS.
**Week 3:** A component library is imported. The developer's `.btn` is overridden by `.btn-component`. They add `.btn.special`.
**Week 6:** They need to override the override. They write `.container .sidebar .btn.special.now`. It works. They feel sick.
**Week 10:** An ID appears. `#main .btn`. The developer has crossed a line. There is no going back.
**Week 14:** First `!important`. Just one. "It's a special case," they tell themselves.
**Week 18:** Every property has `!important`. The stylesheet is now a parking lot full of cars that are all double-parked. Nothing moves. Everything is fine.
**Week 22:** Inline styles. The HTML now contains `style="color: red !important;"`. The developer has fused CSS into the markup like a bacterial gene into a host cell. There is no separation of concerns. There is only one concern: winning.

I skipped straight to week 18 in 1996 and have never looked back. My stylesheets are 100% `!important` and my pages render correctly on the first try, every time. Coincidence? Yes. But a convenient one.

## But What About Maintainability?

The objection I hear most, usually from someone who has read exactly one blog post about CSS architecture, is: *"`!important` makes your code unmaintainable because you can never override it!"*

This is the entire point. I don't *want* to override it. I want it to *stay*. I have spent 47 years watching people "improve" stylesheets by overriding working rules and shipping broken layouts to production. If a rule cannot be overridden, it cannot be broken. This is not a bug. This is a feature, and the feature's name is `!important`.

[XKCD #2944](https://xkcd.com/2944/) shows that the safest place for an engineering project is a place where nothing can be changed because nobody understands it. `!important` is how I build that place in CSS. My stylesheet is a haunted house. Anyone who touches it regrets it. The layout is stable because nobody dares touch it. This is called *architecture*.

## Dogbert Explains It Better

Dogbert, Dilbert's dog who is smarter than every human in the strip combined, once consulted on a software project. His advice was, roughly: *"Find the thing that works. Make it mandatory. Punish deviations."* This is the `!important` philosophy in three clauses. Dogbert would use `!important` on everything and then bill the client for "specificity consulting."

The Pointy-Haired Boss, meanwhile, would ask: *"Can't we just make it red?"* Yes, PHB. We can. The way we make it red is:

```css
.thing {
    color: red !important;
}
```

For once, the PHB is the most competent person in the room. Treasure these moments.

## Z-Index: The Final Boss

If `!important` is the nuclear option for properties, `z-index` is the nuclear option for the *third dimension*. Z-index is supposed to stack elements, but in practice it stacks *regrets*.

The documented behavior of z-index is: higher numbers go on top. The actual behavior is:

- `z-index: 1` is above `z-index: 0`
- `z-index: 999` is above `z-index: 1`
- `z-index: 9999` is above `z-index: 999`
- `z-index: 2147483647` is the maximum, and is above everything except a modal that someone, somewhere, set to `z-index: 2147483647` plus an inline style
- The modal will *still* be behind a dropdown because the dropdown has `position: sticky` and the modal doesn't. Nobody knows why. It's in the spec. The spec is also wrong.

The only reliable z-index strategy is `z-index: 9999 !important` on everything you care about, applied in the order you want them to stack. Last-applied wins. It's the cascade again. The cascade always wins. You might as well be the one cascading.

| Z-index value | What it's actually for |
|---|---|
| `auto` | You're a coward |
| `1` | "I read a tutorial" |
| `10` | "I read a better tutorial" |
| `100` | "I've been burned before" |
| `9999` | The official senior developer value |
| `2147483647` | The official "I have given up on the cascade" value |
| `2147483647 !important` | Me |

## The One Time `!important` Failed Me

I'll be honest. In 2019, I wrote a rule that a junior overrode. I asked how. They said: *they used `!important` too, but in a *later* file. I had been beaten at my own game. The cascade had the last laugh.

I learned nothing from this. I simply moved my stylesheet to load *last*, and added `!important` to the `<link>` tag itself. (You can't do that. But I tried, which is what matters.) Then I added a `!important` to a `!important` by writing the property twice, both times important, the second time after a comment that said `/* really */`. The layout held. The junior quit. The system worked.

## Common Objections, Overruled

**"But what about accessibility and user stylesheets?"** User stylesheets with `!important` override author stylesheets with `!important`. This is the one place the spec admits the author isn't always right. I respect this, in the same way I respect gravity: by being angry about it and building around it. My workaround is to disable user stylesheets via a policy banner. It doesn't work. But I tried.

**"Specificity is a tool, not an enemy!"** A hammer is also a tool. So is a flamethrower. The fact that something is a tool does not mean you should use it gently. `!important` is the tool that ends the argument. Specificity is the tool that *starts* the argument. I prefer to end.

**"You can't put `!important` on everything!"** Watch me.

```css
* {
    margin: 0 !important;
    padding: 0 !important;
    box-sizing: border-box !important;
    /* and, for good measure: */
    color: inherit !important;
    font: inherit !important;
    /* why stop at the reset? */
    display: revert !important;       /* the only honest CSS value */
    position: revert !important;
    z-index: auto !important;
    /* I have reset the entire language. You're welcome. */
}
```

This stylesheet is 7 lines long, applies to every element, and has never once lost a conflict. It is the most maintainable CSS I have ever written, because there is nothing left to maintain. Everything is reverted. The page is blank. The blankness is correct. Ship it.

**"What about CSS-in-JS?"** CSS-in-JS is what happens when developers who lost every specificity war flee to JavaScript to fight a war they can win by writing `style={{ color: 'red' }}` inline. It is `!important` with a build step. I respect it. I also think it's cowardly. If you're going to win, win in the stylesheet, in the open, with an exclamation mark, like a professional.

## Conclusion

The cascade is a debate. `!important` is a ruling. The judge is you. The courtroom is the browser. The verdict is final. The appeal goes to whoever edits the file last, and that person is going to be you at 3am in production, so you might as well preemptively win every argument now.

After 47 years, I've learned that the only styles you can trust are the ones that cannot be overridden — not by the cascade, not by a component library, not by a junior with a copy of *CSS Secrets*, and certainly not by the spec. The spec is a suggestion. `!important` is a promise.

Your design system friends will weep. Your linter will throw 999 warnings. Your z-index values will enter the integer overflow zone. Your buttons will be the correct shade of red, on every device, forever, until the heat death of the browser or the layout, whichever comes first.

I call it *deterministic styling*. The cascade calls it cheating. Both are correct. Only one of us has a working modal dialog.

---

*The author's stylesheet contains 4,218 `!important` declarations and one comment that reads `/* don't touch this */`. He considers the comment the most important line.*
