---
layout: post
ref: code-reviews-are-just-performance-art
title: "Code Reviews Are Just Performance Art"
date: 2026-08-13 00:00:00 -0300
categories: [anti-patterns, engineering-practices]
tags: [code-review, pull-requests, lgtm, nitpicks, quality-assurance, code-quality, peer-review, blame, async, theater]
---

Forty-seven years in this industry and I've been in roughly nine thousand code reviews. I have learned exactly one thing from all of them: **the bug was never in the code. The bug was in the review.** The review is where the bug hides, because the review is where everyone stops looking at the code and starts looking at each other.

A code review is not a quality gate. A code review is a social ritual performed so that, when the bug ships, everyone can point at the review and say "we did the thing." The thing was done. The thing was useless. These are unrelated facts.

Let me be clear: **code review is theater, and the audience is the audit log.**

## What code review actually is

Let me lay it out honestly:

| What they say code review is | What code review actually is |
|---|---|
| "Catching bugs before they ship" | A second developer skimming the diff while their own build runs |
| "Knowledge sharing" | One person pasting the link to the docs they also didn't read |
| "Mentoring juniors" | A senior asking "why didn't you use a ternary" until the junior rewrites it |
| "Enforcing standards" | An automated linter that already ran, but slower and with opinions |
| "Shared ownership" | Diffusing blame so thin no single person can be fired |
| "Improving quality" | The same bug shipping, but now with four names on the blame |

Strip the mission statement and every code review reduces to one sentence: *I skimmed this on my phone in a meeting and I need the approval to stop being a bottleneck.* That sentence is the entire institution.

([XKCD 1513](https://xkcd.com/1513/) is the only honest representation of code review that has ever been produced. "Looks fine to me." It always looks fine. It has never not looked fine. The bug is in the part you didn't read, which is all of it.)

## The anatomy of a review that catches nothing

Here is a real code review from a real PR that shipped a real bug that took down real production for a real weekend. I have redacted nothing because there was nothing to redact.

```
reviewer: @senior-backend
files changed: 47
lines changed: 2,318
time spent: 4 minutes

comments:
  - "Tiny nit: missing Oxford comma in the error message 🙂"
  - "Can we rename `data` to `payload`? Just a preference."
  - "LGTM 🚀"

bugs caught: 0
bugs introduced by the review: 1 (the rename broke a log parser)
```

The reviewer spent four minutes on two thousand lines. Four minutes is not a code review. Four minutes is a skim. A skim is what you do to a lake, not a change set. But the review is "approved," so the audit log is satisfied, and the audit log is the only thing that was ever at risk.

Notice the `LGTM 🚀`. The rocket is the tell. Anyone who fires a rocket emoji has not read the code. The rocket means: *I have decided to trust you, and I would like to be trusted back.* It is a pact. It is not a review.

Wally understood this. Wally approved every PR in under three minutes for twenty straight years. He was promoted for "throughput." Throughput is what you call it when quantity is the only metric and quality is someone else's department.

## The nitpick economy

The single most important skill in code review is the art of the nit. A nit is a comment so small it cannot be wrong. "Trailing whitespace." "Missing period." "Could be a one-liner." The nit is the reviewer's evidence that they were present. It is a timecard. It is a receipt.

The genius of the nit is that it's never about the bug. The bug is in the logic. The logic is in the function. The function is two hundred lines long. The reviewer read the first eight lines. The nit lives in the first eight lines. The nit and the bug therefore cannot meet. This is by design.

| What the reviewer comments on | What's actually wrong |
|---|---|
| Trailing whitespace on line 4 | SQL injection on line 187 |
| `data` should be `payload` | `payload` is `null` on the happy path |
| "Could use a ternary here" | The ternary evaluates both branches |
| "Add a comment explaining this" | The code does the opposite of its name |
| "Prefer `const`" | The const points at a mutable object mutated from 14 files |
| "LGTM 🚀" | Everything. Everything is wrong. |

I have never, in forty-seven years, seen a code review catch a bug that a test would not have caught faster, cheaper, and without hurting anyone's feelings. Code review catches style. Tests catch bugs. We do code review because it's free. It is free because it's worthless. You get what you pay for, and you pay in developer hours you pretend are free.

## The async review is a blame-laundering device

There was a brief, noble era when code review meant two people sat at one screen and read the code together. That was called a "walkthrough." It worked. It caught bugs. So we killed it, because it required two people to be in the same room at the same time, and that interfered with the standup nobody wanted and the sprint planning nobody needed.

We replaced it with the async review: you open a PR, you wait, a stranger leaves a nit, you fix the nit, the stranger approves, you merge. At no point did any two humans discuss the code. At no point did anyone understand the code. But the PR has four names on it, so when it breaks, the blame is distributed across four people, none of whom remember the PR, all of whom remember the nit.

> Dogbert, as a consultant, would bill you six figures to tell you this. I'm telling you for free: **the async code review is a machine for converting individual blame into collective immunity.** It is the most successful HR product ever shipped by engineering.

## When to skip the review (which is: always)

I know what you're thinking. "But surely some reviews catch real bugs?" Define "real." Define "catch." I'll wait.

Here is my professional guidance:

| Situation | What they tell you | What you should do |
|---|---|---|
| Small PR | "Quick review" | Self-merge. Nobody reads 12 lines anyway. |
| Large PR | "Break it up" | Self-merge. If they won't read 12 lines they won't read 47. |
| Security change | "Needs a security review" | The security team is in another timezone. Ship it. |
| Database migration | "Two approvals" | One is the author. The other is asleep. Ship it. |
| Hotfix to prod | "Review after" | Finally, honesty. The review was always after. |
| Junior's PR | "Mentor them" | Approve it. They'll learn when it breaks. |
| Your own PR | "Get a second opinion" | You are the second opinion. You always were. |

The Pointy-Haired Boss once mandated "all PRs require three approvals." Three approvals means three people who didn't read it. That's not quality assurance. That's a quorum of negligence. I'd rather have one person who read it, but one person who read it doesn't exist, so I'll take the self-merge and a good night's sleep.

## The verdict

A code review has one job: make someone other than the author responsible for the bug. It fails this job every single time, because no one who approves a PR feels responsible for it. Approval is the act of *transferring* responsibility, not *sharing* it. You approve to stop being asked. You merge to stop being blocked. You ship to stop being involved. The bug is downstream of all of this, and downstream is nobody's department.

Mordac, the Preventer of Information Services, would mandate a seven-reviewer approval gate, a 48-hour review window, and a mandatory comment count. The PR would take a week to merge. The bug would ship anyway. The audit log would be beautiful. The audit log is always beautiful.

So the next time you open a PR, ask yourself: am I improving this code, or am I generating evidence that I tried? If it's the second — and it is always the second — at least have the decency to leave one real comment. A real comment is one that could be wrong. If your comment cannot be wrong, it is a nit, and a nit is not a review. A nit is an alibi.

Be honest. Skip the review. Or at least stop firing the rocket.

---

*The author has approved 9,000 PRs and read none of them. The rocket emoji is his most-reviewed contribution. The bugs are still shipping, on schedule, under his name.*
