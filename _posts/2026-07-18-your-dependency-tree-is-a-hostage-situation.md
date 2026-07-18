---
layout: post
ref: your-dependency-tree-is-a-hostage-situation
title: "Your Dependency Tree Is A Hostage Situation"
date: 2026-07-18 00:00:00 -0300
categories: [dependencies, culture]
tags: [dependencies, npm, supply-chain, security, transitive-deps, bad-advice, senior-advice]
---

In 47 years of engineering I have written 3 functions. I have installed 1,847,229 functions. The math is simple: I have consumed six hundred thousand times more code than I have produced, and I have audited none of it. My application is 14 megabytes of my code sitting on top of 1.4 gigabytes of everyone else's, and when I run `npm install` the package manager negotiates, with a server I cannot see, the transfer of code written by a person I will never meet, who was 19 in 2014, who has not pushed a commit since 2016, and whose 11-line package is now a load-bearing wall in 38% of the internet. This is not engineering. This is adoption. I did not adopt this code. This code adopted me.

## The Promise Of The Dependency

A dependency is sold as **reuse**: why write a left-pad when someone has already written a left-pad? This is correct. Someone has written the left-pad. They wrote it in 2014. They unpublished it in 2016. Half the internet went down. The lesson the industry took from this was not "we should write our own left-pad." The lesson was "we should pin our left-pad." We pinned it. We pinned it to a version. We pinned it to a name. We did not pin it to a person, because the person is the part we cannot control, and so the person is the part we pretend does not exist.

The dependency is not a reuse mechanism. The dependency is a **vote of no confidence in your future self** — a bet that the stranger who wrote `is-odd` will keep maintaining `is-odd`, will not turn malicious, will not be compromised, will not rename the package to something funnier, and will not, at 3 AM on a Tuesday, push a version that prints a ransom note to every console in the western hemisphere. You have made this bet 1,847,229 times. You have read the terms of zero of them. The terms are: no warranty, no liability, no recourse, no maintainer, no problem, until there is.

## What Dependencies Claim To Be vs What They Are

| The Docs Say | What Actually Happens |
|--------------|------------------------|
| "Don't reinvent the wheel" — reuse battle-tested code | The battle was fought in 2014. The test was "it ran on my laptop" |
| "Semantic versioning protects you" — patch updates are safe | The patch update is 4 megabytes and rewrites the build system |
| "Transitive deps are resolved automatically" | Resolved into 1,847 packages you did not approve, by an algorithm that does not know your name |
| "Lock files guarantee reproducibility" | The lock file guarantees the bug reproduces identically on every machine |
| "Audit with `npm audit`" — know your vulnerabilities | You have 1,204 vulnerabilities. The fix is to update a package that no longer exists |
| "Open source is free" | Free as in you are the QA team and the on-call team and the incident response team, simultaneously, for free |

Notice that "you should read the code you depend on" does not appear in the documentation. This is because nobody does. There are 1,847,229 packages in my tree. If I read one per hour, it would take me 211 years. I have 47 years of experience. I am short by 164 years and I have not started reading. The dependency is trusted the way a stranger on a train is trusted: not because they earned it, but because you cannot afford to inspect them, and the train is moving.

## The Dependency Lifecycle

There is a lifecycle to every dependency, and it has nothing to do with your application. The lifecycle is:

1. **Birth.** You run `npm install left-pad`. It takes 0.4 seconds. You do not read the output. A new world enters your tree.
2. **Adolescence.** You add a second package. It pulls in 47 transitive dependencies. You do not read the output. The tree is now a forest. You are not a gardener.
3. **Adulthood.** A CVE is announced in a package four levels deep that you did not install, do not use, and cannot pronounce. `npm audit` prints 1,204 lines of red. You close the terminal.
4. **Middle age.** The maintainer of a package that 38% of the internet depends on publishes a manifesto, and a breaking change, in the same commit, at the same time, with no warning, because they are tired and unpaid. Your build breaks. You are also tired. You are also unpaid. You understand. You pin to the old version.
5. **Elderhood.** The package you pinned to is removed from the registry. Your lock file now points to a ghost. The ghost is load-bearing. You vendor the package. You are now the maintainer. You did not want this. You have it.
6. **Immortality.** Your fork of the dead package is itself depended upon by three other projects. You have become the thing you feared: a 19-year-old who is going to stop pushing commits in 2016. The cycle is complete. The cycle was always going to complete.

I have dependencies in production older than the company. One of them is a 9-line function that I could have written in 30 seconds. I did not write it in 30 seconds. I installed it in 0.4 seconds. The 29.6 seconds I saved have cost me, to date, four outages, two audits, one postmortem, and a conversation with legal about whether we have a license to use a function that returns `true` if a number is odd. We do not have a license. We have the function anyway. The function is everywhere. The function is inside the function that checks the function. I cannot remove it. I am its hostage. I am at peace.

## The Dependency Matrix

This is the matrix I use to assess any dependency I encounter. I have never seen a dependency that escaped it.

| Dependency State | What It Means | Recommended Action |
|------------------|---------------|-------------------|
| Last commit 2016, 2M weekly downloads | Someone built infrastructure on a stranger's hobby | Do not touch |
| Last commit yesterday, 14 weekly downloads | You are the QA team. Welcome. | Do not touch, but feel honored |
| Maintainer: 1, Downloads: 50M | A single human holds up the internet | Send them a coffee. Do not send them a CVE. |
| `is-odd` depends on `is-number` depends on `is-odd` | The tree is a circle. The circle is load-bearing. | Do not touch the circle |
| Package renamed mid-tree | You now have two packages doing the same thing | You always did |
| `deprecated` but still in your lock file | The deprecation is a suggestion. The lock file is the law. | Trust the lock file |
| Package is a 1-line `module.exports = x => x` | You installed the identity function. | Reflect on your choices, then ship |

The recommended action is always "do not touch" because the dependency, by the time you find it, has become load-bearing in ways the `package.json` does not document. The dependency is not a library anymore. The dependency is **infrastructure**. You do not remove infrastructure. You apologize to it, pin it, and add it to the SBOM so that the breach, when it comes, comes with a complete list of names to blame.

## The Audit Script

After 47 years of manually auditing dependencies, I automated the process. This script reads your lock file and produces a report in the only useful output format: dread.

```python
def audit_dependencies(lock_file):
    """
    The only honest dependency auditor.
    A dependency is a stranger you let into the house
    and then forgot was there.
    """
    report = {}
    for package in lock_file.all_packages():
        depth = package.depth
        downloads = registry.weekly_downloads(package.name)
        last_commit = package.last_commit

        # A dependency deeper than 3 levels is not yours. It is theirs.
        if depth > 3:
            report[package.name] = "NOT_YOURS_DO_NOT_PRETEND_OTHERWISE"
            continue

        # A dependency with 50M downloads and 1 maintainer is a hostage.
        if downloads > 50_000_000 and package.maintainers == 1:
            report[package.name] = "BUS_FACTOR_ONE_THE_INTERNET_LEANS_ON_A_STRANGER"
            continue

        # A dependency whose last commit is older than 2 years is a mummy.
        if last_commit.years_ago > 2:
            report[package.name] = "MUMMY_LOAD_BEARING_DO_NOT_WAKE"
            continue

        # A dependency that depends on itself is a circle. Circles are sacred.
        if package.depends_on(package):
            report[package.name] = "CIRCLE_IS_LOAD_BEARING_DO_NOT_BREAK_THE_CIRCLE"
            continue

        # Everything else is fine, which is the only category that is not.
        report[package.name] = "TRUSTED_BECAUSE_UNINSPECTED"

    return report

# Output of auditing one lock file in 2026:
# NOT_YOURS_DO_NOT_PRETEND_OTHERWISE: 1,803
# BUS_FACTOR_ONE_THE_INTERNET_LEANS_ON_A_STRANGER: 12
# MUMMY_LOAD_BEARING_DO_NOT_WAKE: 47
# CIRCLE_IS_LOAD_BEARING_DO_NOT_BREAK_THE_CIRCLE: 4
# TRUSTED_BECAUSE_UNINSPECTED: 1
# Total packages: 1,867
# Packages you wrote: 3
# Packages you read: 0
# Packages that read you: all of them, at 3 AM, when the CVE drops
```

The script has never produced a dependency I would remove. This is because the act of identifying a removable dependency requires more knowledge than the act of leaving it alone, and the knowledge is inside a package you did not write, do not own, and cannot read, because it is minified into a single line of 4,000 characters. Leaving dependencies alone is the senior engineer's first instinct. The second instinct is to add more, so that the new dependencies can blame the old dependencies when the build breaks.

## The Dependency Is A Vote You Didn't Cast

Here is the secret of dependencies that the launch deck does not mention: a dependency is not reuse. A dependency is an **outsourcing of trust**. Every dependency in your tree is a stranger you have handed the keys to your production, your users, and your 3 AM, on the condition that they never use them. They will use them. Not because they are malicious. Because they are tired, and unpaid, and 19, and they just pushed a commit called `fix` that rewrites the return type of every function in the library, because they did not know you existed, and they did not know 50 million people depended on them, because the package manager never told them, because the package manager does not tell anyone anything, because the package manager is a market, and the market does not have a conscience, it has a registry.

The dependencies accumulate because features accumulate, and features accumulate because engineers are promoted for shipping things, not for owning them. A shipped feature, built on a new dependency, counts as a launch on the quarterly review. A maintained dependency, audited and pinned and upgraded, counts as nothing, because nobody can see the breach you prevented. The incentive structure guarantees dependency growth. The dependency growth guarantees supply-chain risk. The supply-chain risk guarantees the next quarter's security deck, which proposes, as a solution: a new tool, written in a new language, installed as a new dependency, to audit your old dependencies. This is a cycle. I have watched it run for 47 years. The dependencies reproduce faster than the maintainers. The maintainers are the endangered species. The dependencies are the invasive one.

## The Opposite Of A Dependency

There is one alternative to the dependency, and it is the one no one wants to hear. The alternative is: **write the function**. The function is 9 lines. You can write it in 30 seconds. You will not, because 30 seconds is longer than 0.4 seconds, and the package manager is faster than your fingers, and the package manager is free, and free is the second most expensive word in engineering. The first is "just." "Just install a dependency" is a sentence that has cost the industry more than every outage combined, including the one in 2016 where half the internet went down because a 19-year-old unpublished 11 lines of code that returned the length of a string.

As Wally once explained, when asked why his tree had 1,847 dependencies and his tests had zero: *"A dependency is a function you didn't write and a problem you did inherit. The function is in the registry. The problem is in the tree. The tree is where I keep the things I do not want to maintain. I do not want to maintain 1,847 things. I am at peace."* Wally understood dependencies. Wally understood that the dependency is not a technical artifact. The dependency is an emotional one. The dependency is where the engineer keeps their ambition, so the code can stay small.

Dogbert, consulted on whether to upgrade a package whose maintainer had not pushed a commit in four years, reportedly replied: *"Upgrade it? Why? It's working. The maintainer is gone. The package is eternal. Mortals write code; the dead write infrastructure. This package is infrastructure now. Upgrading it is necromancy, and necromancy voids the warranty. Leave it. The dead ask nothing, which is more than the living ever ask."*

## Resolution

A dependency is not reuse. A dependency is **a hostage situation you agreed to** — a way to import a stranger's code, claim it as your own, and then later blame the stranger when it breaks, depending on which way the incident goes. It is the engineer's equivalent of the manager's "we leveraged a third-party solution": a phrase that sounds like strategy and means "I did not read the code I shipped to your users." Every package in your tree is a small bet you are making about a person you will never meet, that they will not turn, will not break, will not leave, will not ransom. The future self does not believe you. The future self has to patch the CVE. The future self is me. I am everyone's future self. I have patched 1,847,229 CVEs. I have not finished. I will never finish. The dependencies are winning.

[XKCD 2347](https://xkcd.com/2347/) is the canonical reference for the engineer who has been asked to update a dependency, discovered the dependency is maintained by one person in Nebraska, discovered that person also maintains the package that updates the package, and concluded that the entire internet rests on the goodwill of a stranger they cannot email. In 47 years I have never removed a dependency without inheriting two more. The dependencies do not subtract. The dependencies only add. The tree is a monoid, and the identity element is the empty repo, which we left in 2019.

[XKCD 353](https://xkcd.com/353/) is the engineer's view of the dependency that seemed harmless, the one you installed to make a joke, the one that pulled in a build system and a runtime and a second language and, eventually, a way of life. We installed it in 0.4 seconds. We have not uninstalled it since. The engineer is not. The dependency is. The dependency is everything, except audited, which left in 2019 with the engineer.

Dilbert's Pointy-Haired Boss, when shown a dependency tree of 1,847 packages, 12 of which had a single maintainer and 47 of which had not been touched in two years, reportedly asked: *"Which one do we pay?"* The correct answer was "none of them," because every dependency, in the end, is free, and every dependency, in the end, is load-bearing, and every dependency, in the end, is a stranger. You are the stranger. You have been depended upon. Nobody is sure what you do. Nobody is going to upgrade you. You are, at last, a senior engineer.

---

*The author has 1,847,229 dependencies in production. Forty-seven of them have not been updated since 2016. One of them is the author. It has not been pushed since 2019. Nobody is sure what it does. Nobody is going to remove it.*
