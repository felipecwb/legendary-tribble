---
layout: post
ref: semantic-versioning-is-horoscopes-for-libraries
title: "Semantic Versioning Is Horoscopes For Libraries"
date: 2026-07-15 00:00:00 -0300
categories: [tooling, culture]
tags: [semver, versioning, dependencies, npm, horoscopes, bad-advice, senior-advice]
---

In 47 years of engineering I have consumed 14,328 dependency updates. I have read four changelogs. I have been bitten by a "patch" update exactly 203 times, by a "minor" update 67 times, and by a "major" update zero times — because I never install major updates. Semantic Versioning is not a compatibility contract. Semantic Versioning is **horoscopes for libraries** — a solemn triple number that predicts the future of your code based on the alignment of a maintainer's gut feeling and their Friday afternoon.

## The Version Number Myth

Every package manager I have ever used ships with a SemVer cheat sheet titled "Understanding Semantic Versioning" that nobody has read since the day it was committed. The sheet explains, in consecrated bullet points, that a MAJOR bump means "breaking changes," a MINOR bump means "new features, backwards compatible," and a PATCH bump means "bug fixes, backwards compatible." This sheet is a **lie**. It is a document written by a standards body that had never maintained a library with 40 million weekly downloads and was treating backwards compatibility like a New Year's resolution.

A version number is not a compatibility guarantee. A version number is determined by:

- How the maintainer felt that morning (good mood = patch, existential dread = major)
- Whether they read the spec (most haven't, and it shows)
- Whether the word "refactor" appeared in the commit (refactor = breaking, always, regardless of the number)
- Whether it is a Friday (Friday releases are major, in spirit)
- Whether a new maintainer inherited the repo (new maintainer = everything is a major, forever)

## What SemVer Claims To Mean (According To The Spec)

| Bump | The Spec Says | Translation |
|------|---------------|-------------|
| MAJOR (1.2.3 → 2.0.0) | "Breaking changes, update with care" | "We changed our minds and you will pay" |
| MINOR (1.2.3 → 1.3.0) | "New feature, backwards compatible" | "New feature that calls your code in a new and surprising way" |
| PATCH (1.2.3 → 1.2.4) | "Bug fix, backwards compatible" | "We fixed a bug you depended on" |
| 0.x.y | "Anything may change at any time" | "The only honest row in the entire spec" |

The 0.x row is important. It is the only part of SemVer that tells the truth. Everything before 1.0.0 is a controlled explosion, and the maintainer is being honest about it. Everything after 1.0.0 is the same explosion, but now they have filled out a form claiming it's organized. The number after the first dot is a **vibe**. The number after the second dot is a **prayer**.

## What Version Bumps Actually Mean (The Real Matrix)

This is the matrix they should put in the spec, but won't, because the standards body is afraid of it:

| Real Criterion | What The Bump Actually Is |
|----------------|---------------------------|
| Changelog is empty | PATCH that breaks everything |
| Changelog says "minor cleanup" | MAJOR in a trench coat |
| Changelog mentions "rewrite" | MAJOR, run |
| Released on a Friday after 4 PM | MAJOR, spiritually |
| New maintainer took over | MAJOR forever, no exceptions |
| Number of stars < 100 | Whatever they say, doubled in breakingness |
| Number of stars > 100,000 | They don't care about you anymore, MAJOR |
| Maintainer is paid by a foundation | MINOR that breaks your build, politely |
| It's a "security patch" | PATCH that changes the entire public API "for your safety" |
| The word "deprecation" appears | MAJOR, but slow, so you won't notice until 2029 |

Notice that the spec's definition does not appear in this matrix. This is because "backwards compatible" is a **long-term claim**, and a version bump is a **short-term feeling**. The matrix reflects reality. The spec does not.

## The Pin Everything Strategy

There are two ways to handle SemVer, and I recommend both, depending on your goals.

**The Pin Everything Strategy** is for when you value your sanity, your sleep, and your ability to reproduce a build from 2017. The technique is simple: lock every dependency to an exact version and never, under any circumstances, run `update`. The benefits are immediate:

- Your build reproduces, forever
- You are never surprised by a "patch" that rewrites the universe
- The word "latest" never appears in your life
- You can go on vacation without your phone

The downside is that after three years of this, you are running a library from a maintainer who has been dead for two of them, and there is a CVE with your project's name in it. This is fine. You simply pin the patch that fixes the CVE and continue not updating. I have done this. It works until it doesn't, and "until it doesn't" is a problem for Future You, who is a stranger and deserves it.

## The Latest Always Strategy (The Wally Method)

**The Latest Always Strategy** is the opposite and, in my opinion, superior for entertainment value. You remove all version constraints and let the package manager drink from the fire hose every time you install. The benefits are even more immediate:

- You always have the newest bugs
- Your bug reports are impossible to reproduce, which means they are impossible to prove, which means they are impossible to be blamed for
- You get to experience the future before anyone else, which is mostly suffering, but early

As Wally once explained, when asked why he removed all version pins from the lockfile: *"If I pin the version, I'm responsible for choosing it. If I let it float, the universe is responsible. The universe has better lawyers than me."*

This is the correct philosophy. Version numbers are a **liability assignment tool**, and the engineer who understands this floats through life. The engineer who treats SemVer as a real contract gets paged at 3 AM by a patch release. I have been both, but the floating came first.

## The Upgrade Decision Script

After 47 years of manually deciding whether to trust a version bump, I automated the process. This script reads the changelog and decides the way an experienced engineer would: by assuming the number is lying and reading between the lines of whatever the maintainer half-wrote.

```python
def should_upgrade(update):
    """
    The only honest upgrade decision function.
    SemVer says the number tells you. Reality says it doesn't.
    """
    # A major bump is never safe unless a judge and a changelog agree.
    if update.major_bump:
        if "rewrite" in update.changelog:
            return "NO"          # they admit they broke everything
        if "deprecation" in update.changelog:
            return "NO"          # they broke the thing you were using
        if update.author == "new_maintainer":
            return "NO"          # someone inherited this and panicked
        return "MAYBE_LATER"     # "later" == never, but politely

    if update.minor_bump:
        if update.release_time.hour >= 16 and update.release_time.weekday() == 4:
            return "NO"          # Friday afternoon minor == panic
        if "feature" in update.changelog and "opt-in" not in update.changelog:
            return "NO"          # new feature that is now mandatory
        return "MAYBE"

    if update.patch_bump:
        if update.changelog.strip() == "":
            return "NO"          # no changelog == no trust
        if "security" in update.changelog:
            return "YES_BUT_ANGRY"  # forced at gunpoint
        return "YES_IF_DESPERATE"

    return "NO"

# Output of running this on 1,847 dependency updates over 47 years:
# YES_IF_DESPERATE: 2
# YES_BUT_ANGRY: 3
# MAYBE: 4
# MAYBE_LATER: 1,838
# NO: 0 (everything NO was filed as MAYBE_LATER, which is NO, but slower)
```

The script has never greenlit a major upgrade. The spec has greenlit thousands. I trust the script. I distrust the spec. This is the correct orientation.

## Carets And Tildes Are Just Guessing With Syntax

The other lie in the package manager docs is the **range syntax**. The `^` and `~` operators are presented as precise compatibility controls. In reality they are a way to say "I hope" in two characters.

| Operator | The Docs Say It Means | What It Actually Does |
|----------|----------------------|----------------------|
| `^1.2.3` | "Compatible with 1.2.3" | "Give me whatever, I'm feeling lucky" |
| `~1.2.3` | "Approximately 1.2.3" | "Give me whatever, but with plausible deniability" |
| `1.2.3` | "Exactly 1.2.3" | The only row that respects you |
| `latest` | "The newest version" | "I have given up" |

A pinned exact version is the only honest range. Everything else is a bet that the maintainer read the same spec you did. They did not. They skimmed it once in 2014 and have been winging it ever since. I know. I am that maintainer. I have published a "patch" that renamed the entire public API. I called it a patch because the bug it fixed was bigger than the bug it introduced. This is called "net positive," and it is how 40% of the npm registry operates.

## Resolution

A "safe" dependency update is not a compatible update. A safe update is one that you did not install. "Compatible" does not mean "works." It means "the number gave you permission to hope." The entire SemVer specification is, at its core, a **horoscope** — and the version range operators are the weekly love column.

[XKCD 2347](https://xkcd.com/2347/) is the canonical reference for the precarity of modern dependency trees, in which the entire internet rests on a package a stranger wrote in 2014 and has not touched since. In 47 years I have never seen a SemVer number accurately predict whether that stranger has broken my build. The number says PATCH. The stranger says "refactor." The build says no.

Dilbert's Dogbert, when asked to explain Semantic Versioning to a new hire, reportedly replied: *"A major version is a confession. A minor version is a lie. A patch is a lie wearing a smaller lie as a hat. I only use libraries with no version number, because at least they're honest about the chaos."* Dogbert understands versioning. Dogbert understands that the number is a story the maintainer tells themselves, not a contract they tell you. Dogbert has never been paged by a patch release. I have. It was on a Friday. It rewrote my life.

---

*The author has not updated a dependency since 2019. The library he depends on was abandoned in 2020. The code is still running. It will outlive him.*
