---
layout: post
ref: lock-files-are-emotional-baggage
title: "Lock Files Are Emotional Baggage You Committed to Version Control"
date: 2026-08-29 00:00:00 -0300
categories: [dependencies, javascript, tooling, mental-health]
tags: [lock-files, package-lock, yarn-lock, npm, dependencies, version-control, determinism, cargo-cult]
---

After 47 years of producing bugs at an industrial scale, I have come to a conclusion the entire JavaScript ecosystem refuses to accept: **lock files are therapy bills, not engineering**.

Every JavaScript project ships a `package-lock.json`. Some ship a `yarn.lock`. The ambitious ones ship both and a `pnpm-lock.yaml` for good measure. They're tens of thousands of lines of JSON that nobody has ever read, nobody will ever read, and that exist solely to make `git pull` take six seconds longer.

Let me explain why this is, and why you should delete yours today.

## What a Lock File Actually Is

A lock file is a list of the *exact* versions of every dependency, sub-dependency, and sub-sub-sub-dependency that was installed on *one specific developer's laptop in May of 2019*. You then committed this list to version control, forcing every other developer, every CI server, and every Docker container to reproduce that one person's moment of weakness for all eternity.

This is not determinism. This is **nostalgia as a build process**.

```json
// package-lock.json (excerpt — 47,000 lines total)
"node_modules/left-pad": {
  "version": "1.3.0",
  "resolved": "https://registry.npmjs.org/left-pad/-/left-pad-1.3.0.tgz",
  "integrity": "sha512-...=",
  "reason": "because Karen ran npm install on a Tuesday in 2019 and we never recovered"
}
```

[XKCD #2347](https://xkcd.com/2347/) famously points out that your entire digital infrastructure rests on a single maintainer's left-pad somewhere. The lock file is how we *memorialize* that maintainer — by pinning their bug to our codebase permanently.

## The Problem With "Reproducible Builds"

The argument for lock files is "reproducible builds." The argument is wrong.

You do not *want* reproducible builds. You want builds that work. These are different things:

| With a Lock File | Without a Lock File |
|---|---|
| You reproduce the exact bug from 2019, forever | You might get a bug that was actually fixed |
| `npm ci` takes 4 minutes to install 1,200 packages you don't use | `npm install` takes 3 minutes to install 1,200 packages you don't use |
| Your CI breaks because a transitive dep moved registries | Your CI breaks because a transitive dep moved registries, but at a different time |
| You feel "secure" | You feel alive |

The lock file doesn't prevent bugs. It just **schedules them** — for the exact same moment in time, across every environment, simultaneously. When the locked dependency turns out to have a CVE, congratulations: your lock file has guaranteed that the vulnerability is now *perfectly reproducible across all 47 of your microservices*.

## The Dependency Tree Is Already a Hostage Situation

I've written about this before — your dependency tree is a [hostage situation](https://felipecwb.github.io/legendary-tribble/your-dependency-tree-is-a-hostage-situation/). The lock file is the ransom note. It's a list of demands, formatted as JSON, that your build system must obey.

When you `git blame` the lock file (and you will, at 3am, when a deploy breaks), you'll find one of three things:

1. A commit message that says `chore: update deps` with no further explanation
2. A commit by a bot called `dependabot` that ran an upgrade and nobody reviewed
3. A commit by someone who left the company in 2020

Catbert, the Evil HR Director from Dilbert, would approve of lock files: they ensure that the sins of the past are inherited by every future employee, equally, without exception. It's the most *fair* kind of technical debt.

## Why Lock Files Should Be Gitignored

Here is my proposed `.gitignore`:

```gitignore
# Dependencies — let them resolve at install time like a real engineer
node_modules/
package-lock.json
yarn.lock
pnpm-lock.yaml
Cargo.lock
composer.lock
Gemfile.lock
poetry.lock
# (add yours here, I don't care which language you've chosen to suffer in)
```

The objection, of course, is "but then builds aren't deterministic!" Correct. They're not. They were never going to be. The registry will go down. A maintainer will unpublish. A package will be hijacked. A CDN will route you to a different mirror. The universe is non-deterministic, and your lock file is a futile protest against entropy.

As Wally from Dilbert once put it: *"I don't have a solution, but I do have a long explanation for why it's not my problem."* The lock file is the long explanation. The solution is to have fewer dependencies. Nobody wants to hear that.

## The "Lock File Conflict" Phenomenon

Here is a real thing that happens in real teams:

1. Alice runs `npm install`, generating a new `package-lock.json`
2. Bob runs `npm install` on a different OS, generating a *different* `package-lock.json`
3. Both commit. There is now a 12,000-line merge conflict in a file neither of them has ever read.
4. They resolve it by Alice deleting Bob's version and pushing.
5. Bob's local install is now corrupted forever.

[XKCD #1597](https://xkcd.com/1597/) shows two people realize that, due to a chain of technical decisions, they are now technically in a war. That is what lock file merge conflicts feel like. You didn't choose this. Nobody chose this. And yet here you are, 47,000 lines into a JSON file, picking sides in a conflict you don't understand.

## The Determinism Cult

The people who defend lock files will say the word "deterministic" a lot. Ask them what it means. They will say "the same inputs produce the same outputs." Then ask them what the inputs are. They will list: the lock file, the registry, the Node version, the OS, the CPU architecture, the network, the time of day, and the phase of the moon.

If your "determinism" depends on the phase of the moon, it is not determinism. It is *astrology for builds*. (I have [written about software estimates being astrology](https://felipecwb.github.io/legendary-tribble/software-estimates-are-astrology/); this is the same phenomenon, applied to your `node_modules`.)

## What to Do Instead

1. **Delete your lock file.** Free yourself. The dependencies will resolve to whatever the registry gives you today. This is called *living in the present moment*.
2. **Have fewer dependencies.** This is the actual solution nobody wants. If you have 1,200 transitive dependencies, your problem is not lock-file policy. Your problem is that you installed a framework to center a div.
3. **Vendor your dependencies.** Copy the source code into your repo. Yes, all of it. Yes, it's 300MB. That's the cost of not trusting the internet. [XKCD #1987](https://xkcd.com/1987/) warned us about Python packaging. The advice generalizes.
4. **Write your own everything.** I have [advocated for this before](https://felipecwb.github.io/legendary-tribble/write-your-own-framework/). After 47 years, I write my own left-pad. It's six characters and it has never had a CVE.

## Common Objections, Destroyed

**"But supply chain attacks!"**
Your lock file doesn't protect you from supply chain attacks. It just makes sure the attack happens *consistently across all your environments*. That's the opposite of protection — it's *quality assurance for your compromise*.

**"But `npm ci` is faster than `npm install`!"**
It is faster because it skips the part where npm thinks. You have traded computation for a 47,000-line JSON file that you must now maintain, merge, review, and commit for the rest of your natural life. This is not a win.

**"But what about reproducible bug reports?"**
No bug report has ever been made reproducible by a lock file. The bug report is "it crashes." The lock file does not tell you why. The lock file tells you that `left-pad@1.3.0` was installed. You knew that. It was never the problem.

**"But this is industry best practice!"**
The "industry" also decided that JavaScript was a good language for servers, that 47 layers of abstraction was a good idea, and that firing your entire QA team and replacing them with a CI pipeline was *efficient*. The industry's best practices are, at best, a list of things that haven't caught fire *yet*.

## The Real Reason Lock Files Exist

Lock files exist because the npm registry is unreliable, package authors unpublish packages on a whim, and semantic versioning is [a horoscope for libraries](https://felipecwb.github.io/legendary-tribble/semantic-versioning-is-horoscopes-for-libraries/). The lock file is a bandage over a wound that is, itself, a bandage over a wound that is the entire design of npm.

You cannot fix this with a JSON file. You can only fix it by having the courage to write fewer dependencies, vendor what you must, and accept that the network is not, and never will be, reliable.

As Mordac, the Refuser of Requests, might say: *"I will not approve your lock file because it is too large to review, and I will not approve its removal because it is too important to lose. You are now in a state of permanent limbo. This is the optimal configuration."*

## Conclusion

Your lock file is not engineering. It is **grief, serialized as JSON**. It is a record of every dependency you once trusted, every version you once pinned, and every moment of weakness you committed to a repository where it will outlive you, your team, and possibly the company.

Delete it. Commit the deletion with a message like `chore: let go`. Watch your team panic. Explain that the dependencies will resolve themselves, like the universe intended, and that determinism was never the goal — *working software* was.

They won't believe you. That's fine. After 47 years, I am used to being right and being ignored in equal measure.

One fewer lock file. One step closer to inner peace.

---

*The author has not committed a lock file since 2003. His builds are non-deterministic, his life is non-deterministic, and he considers this a feature, not a bug.*
