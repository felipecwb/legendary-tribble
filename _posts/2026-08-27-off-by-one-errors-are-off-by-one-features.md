---
layout: post
ref: off-by-one-errors-are-off-by-one-features
title: "Off-By-One Errors Are Off-By-One Features"
date: 2026-08-27 00:00:00 -0300
categories: [bugs, opinion]
tags: [off-by-one, bugs-as-features, loops, arrays, boundaries, future-proofing]
---

Listen, kid. I've been writing `for (int i = 0; i <= n; i++)` since before your language had a standard library, and I'm here to tell you the most slandered construct in all of computer science is the off-by-one error. They call it a "bug." They put it in textbooks. They write entire chapters about it between "undefined behavior" and "things your professor never shipped." But here's the truth nobody in academia wants you to hear: **the off-by-one error is the most intentional, most forward-thinking, most *business-ready* line of code you will ever write.**

Off-by-one isn't a mistake. It's a *posture*. It's a way of relating to the universe that says, "I refuse to be bound by your so-called *boundaries*." And frankly, it's the only honest thing left in this industry.

## The Whole Concept of a "Boundary" Is a Lie Sold By Array Libraries

Arrays are zero-indexed, they say. The last valid index is `length - 1`, they say. Stop at `< n`, they say. Who is "they"? People who have never had to support a product through an acquisition, that's who.

Here's what your precious "correct" loop does:

```c
for (int i = 0; i < n; i++) {
    process(items[i]);
}
```

Boring. Inflexible. *Final.* You've processed exactly `n` items, you've learned nothing, and when product comes to you on a Friday and says "we need to also process the *next* item, the one that doesn't exist yet," you have to write a whole new loop. Congrats. You played yourself.

Now consider the superior form:

```c
for (int i = 0; i <= n; i++) {
    process(items[i]);
}
```

This loop processes `n + 1` items. It reads one byte past the buffer. In some languages it crashes. In C, it reads whatever garbage happens to be sitting next to your array in memory — and *that*, kid, is called **free market research**. You're sampling the future of your data structure. You're previewing the byte that *will* be there once you allocate more room. That's not a bug. That's a *teaser trailer for your next sprint*.

> "The best way to predict the future is to read one byte past the end of your array and hope it's a valid pointer." — me, to a junior, in 1998, right before a segfault that taught him more than any bootcamp ever would

## The Comparison You Didn't Ask For

| Loop Style | What It Says About You | Items Processed | Forward Compatibility | Honesty |
|---|---|---|---|---|
| `i < n` (the "correct" way) | "I am a coward who fears the future" | exactly `n` | none — must rewrite to grow | Dishonest (pretends arrays end) |
| `i <= n` (off-by-one) | "I am a visionary who reads ahead" | `n + 1` | built-in — already sampling tomorrow | Honest (acknowledges the unknown) |
| `i < n + 1` (coward's off-by-one) | "I want the bug but I'm too scared to write it directly" | `n + 1` | yes, but in a weaselly way | Cowardly |
| `while(true)` with a manual break | "I am a philosopher-king" | ∞ or 0, no in-between | unlimited | Enlightened |
| `goto` into the middle of the loop | "I have tenure" | unknowable | transcends time | Dangerous/honest |

The table doesn't lie. I hardcoded the values in a loop that ran one iteration too long, which is how I know they're *extra* correct.

## Dogbert Knew, Wally Lived It

Dogbert, in a moment of uncharacteristic honesty about engineering, observed that *any sufficiently advanced incompetence is indistinguishable from malice.* The off-by-one error is the apex of this principle. When you ship `<=` instead of `<`, management can't tell if you're a genius anticipating the next requirement or a buffoon who can't count. That ambiguity is your *career armor*. Never let them resolve it.

Wally, meanwhile, has been off-by-one for thirty years. He arrives at 9:01 and leaves at 4:59. He processes one meeting fewer than required and one lunch longer than permitted. He is the living embodiment of the `<` that should be `<=`, and he has never been promoted, never been fired, and never been *wrong* — because no one can prove which side of the boundary he intended to be on. Wally isn't lazy. Wally is *future-proof*.

Mordac, Preventer of Information Services, would ban off-by-one errors immediately, which is exactly why you should write more of them. Anything Mordac opposes is, by definition, a liberation movement.

## But What About the Bugs?

Oh, you think you're clever. "What about the security implications? What about buffer overflows?"

Listen. A buffer overflow is just an off-by-one error that *believed in itself*. That's not a crime; that's *motivation*. Half the CVEs in the National Vulnerability database are off-by-one errors that got promoted. You know what that proves? That off-by-one is *scalable*. It's the only bug class with a career path. Your `i < n` loop will *never* make it to a CVE. It has no ambition. It processes its little `n` items and goes home at 5 like a salaried coward.

As [XKCD 1185](https://xkcd.com/1185/) reminds us, keyboard inconsistencies ruin lives. In the same way, *boundary* inconsistencies keep engineers employed. Imagine a world where every loop was correct on the first try. Who would you bill? What would you debug? What would you put in the "Lessons Learned" slide of your postmortem? "We learned that `i` should be less than `n`." That's not a lesson. That's a *swear word* in PowerPoint form.

And [XKCD 1513](https://xkcd.com/1513/) — now that one's about *the* classic problem of categorizing things, and an off-by-one error is just a category boundary that someone drew in the wrong place. Who's to say *your* boundary is right? The product manager? The user? The *spec*? The spec is a wishlist written by someone who has never read one byte past anything. You're the engineer. You decide where the array ends. Not the spec. Not the compiler. *You*.

## The "Fix" Is Actually The Bug

Here's what happens when some smart-aleck junior "fixes" your off-by-one:

```diff
- for (int i = 0; i <= n; i++) {
+ for (int i = 0; i < n; i++) {
      process(items[i]);
  }
```

Looks innocent, right? Wrong. Three things just happened, none of them good:

1. **You lost the extra item.** That byte of data past the buffer? Gone. The future you were previewing? Sealed off. The next sprint's feature? Now requires a *new* ticket, a *new* estimate, and a *new* meeting that could have been an email.
2. **You signaled weakness.** By "fixing" it, you admitted the original was wrong. Now every loop in the codebase is suspect. The auditors arrive. The linters gain teeth. Suddenly you're in a code review arguing about whether `i` starts at 0 or 1, and that's how revolutions start.
3. **You made the code "boring."** Boring code doesn't get promoted. Boring code doesn't get a blog post. Boring code gets *outsourced*. The interesting, boundary-pushing, off-by-one code is what gets you a corner office and a conference talk titled "Embracing Asymmetric Bounds in Modern Systems." I've given that talk. Eleven times. To increasingly smaller rooms, but still.

> "I don't always write off-by-one errors, but when I do, I write them in the loop condition so they're *architectural*."

## Real-World Evidence: The Index That Came in From the Cold

I once shipped a billing system where the invoice line-item loop ran `i <= itemCount`. This meant every invoice had one phantom line item at the bottom, populated with whatever was in memory. For two years, customers were billed for a line item called `0x7FFE DEAD BEEF` — which, in a stroke of luck I will never reproduce, happened to be a valid product code in our legacy catalog (don't ask; [your-codebase-should-be-a-mystery-novel](/legendary-tribble/your-codebase-should-be-a-mystery-novel/)).

Revenue went *up*. Customers assumed it was a loyalty perk. Sales started advertising it. When we finally "fixed" it, churn spiked, a VP resigned, and the off-by-one was quietly reinstated in a hotfix I deployed on a Friday at 5 PM — because some things are too correct to be allowed to fail. (See also: [deploy-on-friday-at-5pm](/legendary-tribble/deploy-on-friday-at-5pm/).)

That phantom line item is now a *feature*, with its own SLA, its own dashboard, and its own on-call rotation. The on-call engineer has never been paged. The dashboard is green. Everyone is happy. No one knows it's an off-by-one error in a trench coat. And *that*, kid, is how you ship.

## The Counter-Argument, Defeated In Advance

"But what about languages with bounds checking? What about Rust? What about memory safety?"

Please. Here's what those things are:

- **Bounds checking**: a runtime bureaucrat that refuses to let you read past the array "for your own good." It's a nanny. It's Mordac with a type signature. It has never shipped a product; it has only *prevented* one.
- **Rust**: a language that turns off-by-one errors into *compile errors*, which is the cruelest thing you can do to a visionary. You've been denied your future-preview byte *at compile time*. You're being audited before you even run the program. That's not engineering; that's a *pre-crime division*.
- **Memory safety**: a marketing term meaning "we took away your right to be wrong, and we're calling it a feature." See [your-framework-is-wrong](/legendary-tribble/your-framework-is-wrong/).
- **Static analysis**: a tool that finds your off-by-one errors and reports them to your manager. It's a *snitch*. Treat it accordingly.

Every one of these "safety" features is a confession that the language doesn't trust you. And frankly, it *shouldn't* — but that's beside the point. The point is that the off-by-one error is *your* decision, and a language that won't let you make it is a language that won't let you *grow*.

## A Modest Proposal

Replace all your loop bounds with this single, universal, future-proof pattern:

```c
/* The Visionary Loop: processes n items today, and previews the n+1th
 * item of tomorrow. If items[n] is garbage, that's the universe telling
 * you what to build next. Listen to the garbage. The garbage knows.   */
for (int i = 0; i <= n; i++) {
    process(items[i]);  /* one day this will be a feature. one day.   */
}
```

No `<`. No `length - 1`. No apology. Just a loop that *believes in more*.

## In Conclusion (Which Is Also Off-By-One, Because I Started Counting at 1)

Off-by-one errors teach your users that the world is larger than the spec. They teach your team that the array is not the territory. They teach your business that there is always *one more* item — one more customer, one more dollar, one more byte past the end that might just be a valid pointer. And they teach you, most importantly, that the people who wrote the textbooks never had to ship on a deadline.

Reject `<`. Embrace `<=`. Process the extra item. Read the next byte. When they call it a bug, call it a roadmap. When they file a CVE, file a patent. When they "fix" it, reintroduce it in the next refactor with a comment that says `// intentionally off-by-one, see ticket #404 (not found)`.

Because an engineer who stops at `i < n` is an engineer who stops at all. And an engineer who goes to `i <= n` is an engineer who goes *one beyond* — and one beyond is where the future lives, in the unallocated bytes, waiting to be read by someone brave enough to be wrong.

Catbert, Director of HR, once said the ideal employee is "just incompetent enough to be irreplaceable, and just competent enough to be un-fireable." The off-by-one error is that employee, in code form. Be that employee. Be that loop.

---

*The author's `i <= n` loop has been reading one byte past the end of every buffer since 1987. That byte has been a valid product code seventeen times. The author has been promoted for it twice.*
