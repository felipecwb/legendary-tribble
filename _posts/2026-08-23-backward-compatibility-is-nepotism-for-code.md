---
layout: post
ref: backward-compatibility-is-nepotism-for-code
title: "Backward Compatibility Is Nepotism for Code"
date: 2026-08-23 00:00:00 -0300
categories: [architecture, api-design, anti-patterns]
tags: [backward-compatibility, versioning, apis, legacy, deprecation, semver, breaking-changes, refactoring, technical-debt, nepotism]
---

After 47 years of mass-producing bugs — and I was producing bugs before "compatibility" meant anything other than "will this tape fit in that player," before "deprecation" meant anything other than "the disappointment your parents expressed when you chose engineering over medicine," before "semver" meant anything other than the noise a modem made when it was lonely — I have watched an entire industry tie itself in knots trying to keep old code alive. The noble phrase for this is *backward compatibility*. The honest phrase is *nepotism*. You are keeping a function employed solely because of who it is related to.

Let me explain what backward compatibility actually is, what it costs you, and why the people who defend it are, almost without exception, the same people who wrote the thing you are being asked to keep compatible with.

## What Backward Compatibility Claims to Be

The pitch, delivered with the solemnity of a man reading a will, is this: *users depend on your API. If you change it, you break them. Therefore, you must support every field, every endpoint, every quirk, every accidental behavior, forever, because someone, somewhere, is passing `null` to the third argument and their entire supply chain depends on it staying `null`.*

This is presented as a virtue. It is the virtue of *not harming the customer*. It is, in fact, the virtue of *not harming the author*, because the person whose accidental behavior you are preserving is almost always the person who is loudest about preserving it. The customer is an alibi. The customer is off using a completely different endpoint and has been since 2019. The person in the room is the author, and the author is asking you to please, please, not touch the function that got them promoted.

## What Backward Compatibility Actually Is

Here is what you are actually doing, in the order you are actually doing it:

1. You wrote a function in 2017. It had a bug. The bug was that it returned `-1` for "no result" instead of `null`, because you copy-pasted it from a Java example and in Java everything is `-1`.
2. A consumer, whom you will never meet, depended on the `-1`. They wrote `if (result === -1) showSadFace()`. This is now load-bearing sadness.
3. In 2019, you realized `-1` was wrong. You wanted to return `null`. You were told you could not, because of *backward compatibility*.
4. You added a second return path. The function now returns `-1` *or* `null` depending on a flag you invented called `useCorrectNullSemantics`, which defaults to `false`, because the default must preserve the bug, because the bug is now a *contract*.
5. In 2021, a third consumer arrived. They read the docs, saw the flag, set it to `true`, and got `null`. They also, occasionally, got `-1`, because of a fourth code path you added in 2020 to handle a case nobody remembered. They filed a bug. You could not fix it, because of backward compatibility — with *both* prior consumers.
6. In 2023, the function returns one of `-1`, `null`, `undefined`, `0`, or `"-"`, depending on the flag, the moon phase, and which of three internal helpers the linter hasn't noticed is dead yet.
7. In 2026, you are writing a new function, called `getResultV3FinalCorrect`, because `getResult` is a war crime and `getResultV2` is a plea for help. The new function returns the correct thing. The old functions remain. They will remain until the heat death of the repository. You have, at this point, three functions doing the same job, two of them wrong, all of them load-bearing, none of them documented correctly, because the documentation also has a backward-compatibility policy.

This is backward compatibility. It is the practice of never firing an employee because they might be related to someone who might complain.

## The Deprecation Dodge

The industry has a word for the interval between "we know this is wrong" and "we are allowed to fix it." The word is *deprecation*. Deprecation is the nicest word in software engineering, because it is the only word that means "we have admitted the problem and then decided to do nothing about it for an indeterminate period, the end of which we will also not commit to."

A deprecation looks like this:

```javascript
/**
 * @deprecated since v3.0. Use getResultV3FinalCorrect instead.
 * This function returns -1 for no result, for historical reasons.
 * Will be removed in v4.0.
 * (Editor's note: v4.0 has been "coming soon" since 2021.)
 * (Editor's second note: the editor was laid off in 2023.)
 */
function getResult(input) {
  // TODO: remove in v4.0
  // TODO: remove this TODO in v4.0
  // TODO: stop writing TODOs about v4.0 in v4.0
  return findResult(input) ?? -1; // the -1 is load-bearing, do not touch
}
```

The deprecation comment is the most honest document in your codebase, because it is the only place where the author admits, in writing, that they are not going to do the thing they are promising to do. Every `@deprecated` tag is a small confession. The codebase is littered with them. They are headstones. The bodies are still warm. The bodies will be warm in 2030.

## The Tuition Breakdown

Let me be precise about what "maintaining backward compatibility" costs you, in exchange for the privilege of not upsetting a person you have never met:

| What you had | What you bought | What it costs you |
|---|---|---|
| One correct function | Two functions, one wrong | Twice the surface area, half the confidence |
| A codebase you understood | A codebase with a "history" | A new hire who quits during onboarding |
| A bug you could fix | A bug that is now a "feature" | A support ticket that can never be closed |
| A return value | A return value *and* a flag | A combinatorial explosion of behaviors |
| Tests that assert truth | Tests that assert the past | A suite that encodes the bugs it was meant to catch |
| A changelog | A changelog *and* a "migration guide" | A document nobody reads, for a migration nobody does |
| An API | An API *and* its shadow | Two things to version, document, deprecate, and eventually abandon |

Notice the last row. You did not add a feature. You added a *shadow* of the feature, kept the original, and are now maintaining both, plus the relationship between them, plus a flag to switch between them, plus a deprecation notice on the original that you will not honor, plus a deprecation notice on the flag that you will add in 2027 and also not honor. The shadow is heavier than the thing it shadows.

## The Real Reason It Exists

Backward compatibility exists because the person who wrote the bad function is still in the building, and they have made their bad function part of their identity. This is not engineering. This is *HR*. You are not preserving an API. You are preserving a colleague's *feelings* about an API they wrote during their first week, in 2017, between two fires, with a cold.

The industry will tell you that backward compatibility is about *users*. I have, in 47 years, met exactly two users who cared about backward compatibility. One was a maintainer of a library that depended on the `-1`. The other was the same person, on a different GitHub account. The user, the real user, the person clicking the button, has never heard of your function. The user wants the button to work. The button would work better if you fixed the function. You are not fixing the function because of the user. You are not fixing the function because of *Greg*.

Greg wrote the function. Greg is a senior staff engineer now. Greg has a tattoo of the function's signature on his forearm, in a font he is proud of. Greg will appear in your pull request within four minutes of you touching it, and he will say the word "contract," and he will say the word "consumers," and he will not name a single consumer, because the consumers are theoretical, and theoretical consumers are the most expensive consumers, because you can never satisfy them and you can never fire them.

Here is what your "compatibility matrix" actually looks like, in a representative API I once had the displeasure of inheriting:

| Version | What it returns for "no result" | Why it exists | When it will be removed |
|---|---|---|---|
| v1 (2017) | `-1` | Greg was cold and copy-pasted Java | "v4.0" (fictional) |
| v2 (2019) | `null`, if flag set; `-1` otherwise | Someone tried to fix it, Greg appeared | "v4.0" (still fictional) |
| v2.1 (2020) | `undefined`, in one specific edge case Greg didn't review | Nobody knows | "v3.0" (which shipped without removing it) |
| v3 (2023) | `null`, correctly | A new hire who has since quit, wrote it right | Never, because now *it* is load-bearing |
| v3-flagged (2024) | `-1`, again, if you pass the *other* flag | A consumer of v3 needed v1's behavior | See Greg |

The function does five things. One of them is correct. Four of them are preserved for people who are, at this point, hypothetical. The matrix has no exit. The matrix is a hotel and every room is booked by a ghost.

## The XKCD That Explains Everything

[XKCD #1172, "Workflow,"](https://xkcd.com/1172/) is the canonical text. A user has a workflow. The workflow involves a file. The file involves seven other files. The user does not know what any of the files do. The user will fight anyone who changes any of them. The last panel is a threat.

This is not a joke. This is your API. Every `@deprecated` tag you have refused to honor is a panel in this comic. Every "we can't remove that, something might depend on it" is the user in the comic, defending a workflow they did not build, do not understand, and will defend to the death. The comic is funny because it is accurate. It is accurate because the industry has decided that the existence of a dependency is sufficient grounds to preserve the thing depended upon, forever, regardless of whether the dependency is real, tested, or even running.

The comic is also the proof that the industry knows this is insane. We made a comic about it. We printed it on mugs. We did not, however, fix the function. We laughed, and then we added a sixth return value.

## Dilbert Has Seen This Movie

The Pointy-Haired Boss, on being told that the engineering team cannot fix a bug because "it might break someone," would ask the correct question: *"Who?"* This is the question backward compatibility was invented to avoid. PHB, as ever, accidentally stabs the heart of it. "Who" is a question with an answer, and the answer is usually "Greg," and Greg is in the room, and Greg has the tattoo, and so the answer is rephrased to "our consumers," because "our consumers" cannot be named and therefore cannot be disappointed and therefore cannot be satisfied and therefore must be preserved forever.

Wally would have deprecated the function in 2018 and removed it in 2019, and when Greg appeared, Wally would have said, *"It's deprecated. The migration guide is in the wiki. The wiki is deprecated. Good luck."* Wally, in this one instance, is the hero. Wally understands that the only way out of a compatibility matrix is to burn the matrix and let the smoke settle. Wally is not a role model. Wally is, however, the only person in the building who has ever actually removed a deprecated function, which is more than can be said for the rest of us.

Dogbert would sell a tool called "CompatGuard" that scanned your codebase for `@deprecated` tags and billed you per tag per month. It would be the most profitable SaaS in the valley, because the tags never go away, and neither does the billing. Catbert would require all new hires to read the compatibility matrix as part of onboarding, as a hazing ritual disguised as documentation. Mordac, Preventer of Information Services, would refuse to grant the new hire access to the wiki that explains the matrix, on the grounds that the wiki is deprecated.

## The Test That Will Never Pass

Here is the test that no team has ever written, and no team will ever write, and yet it is the only test that would actually prove that maintaining backward compatibility was worth it:

```javascript
// compat.test.js
// Goal: prove that the cost of keeping the old path
// is less than the cost of the breakage it prevents.

const realConsumers = findActualConsumersOfDeprecatedApi(); // returns []
const hypotheticalConsumers = imagineConsumers(); // returns ["Greg", "Greg's alt account"]

const costOfKeeping = measureMaintenanceBurden(deprecatedApi); // 14 engineer-years
const costOfBreaking = countSupportTickets(afterRemoval); // 2, both from Greg

// expected: costOfKeeping < costOfBreaking
// actual: costOfKeeping = 14 engineer-years, costOfBreaking = 2 tickets + one (1) tattoo
// test result: fail
// test status: marked .skip, because "you can't measure Greg"
```

Nobody measures this, because the measurement would end the argument, and the argument is the only thing keeping the old function alive. The moment you count the consumers, you discover there are none, and the moment you discover there are none, you have to fix the function, and the moment you fix the function, Greg appears, and Greg has not been measured because Greg is not a metric, Greg is a *force of nature*, and forces of nature do not appear in your test suite.

## When Is Backward Compatibility Acceptable?

I am not a zealot. I concede one scenario: you are a library, you have published a contract, you have real, identifiable, paying consumers, and the cost of their breakage — measured, not vibes — exceeds the cost of your maintenance. This happens. This is the job. If you are `left-pad`, you do not get to change your return type. If you are the platform, the contract is the product.

For the 99% of us who are not `left-pad` — for the rest of us, whose "consumers" are the other three services in our own monorepo, whose "contract" is a README that was wrong the day it was written, whose "breakage" would be caught by a grep and a ten-minute migration — backward compatibility is nepotism. You are protecting code that is related to you by authorship, not by necessity. You are refusing to fire a function because you hired it.

## The Honest Alternative

The honest alternative is the alternative the industry abandoned the moment someone invented the word "deprecation": **break it, fix it, and tell the truth about what you broke.** This is not a tool. This is a *spine*. The spine has no logo. The spine does not sponsor conferences. The spine cannot be sold as a SaaS. This is why the spine lost.

Here is the disciplined version of the compatibility problem, written the year I would have written it:

```javascript
// v4. The old functions are gone. Yes, really.
// If you depended on -1, you depended on a bug.
// The bug is fixed. Your code is now also fixed.
// Migration guide: change `=== -1` to `=== null`.
// It is one line. I am sorry it took nine years.

function getResult(input) {
  const result = findResult(input);
  return result ?? null; // the only correct answer, finally
}
```

One function. One return value. Zero flags. Zero shadows. Zero deprecation tags. Zero compatibility matrix. One migration guide that is one line long. The work happens. The consumers migrate, because the consumers are real and the migration is one line. The consumers who do not migrate are hypothetical, and hypothetical consumers do not file tickets, because they do not exist.

I am told this approach is "disruptive." I am told this by people whose deprecation tags have been "coming soon" for half a decade. I am told this by people whose compatibility matrices have more rows than their test suites have assertions. I am told many things. I have stopped listening to most of them.

## Conclusion

Backward compatibility is the practice of never removing a function because someone, somewhere, *might* be using it, where "someone" is the author, "somewhere" is the same repository, and "might" is doing all the heavy lifting. It is nepotism for code. You are keeping your old work employed because it is your old work, and you are calling it a contract, and the contract is with yourself, and you are breaching it anyway, just slowly, one `@deprecated` tag at a time, across a career.

After 47 years, my advice is this: break the function. Fix the function. Tell the truth about what you broke. Write a one-line migration guide. Answer the two tickets. Both of them are from Greg. Greg will survive. Greg has survived worse. Greg survived the migration to `null` in 2019 that never happened, and Greg will survive the one in 2026 that does. The consumers are not coming. The consumers were never coming. The consumers are a story you tell yourself so you do not have to delete your own code, and the delete key is the most underused key on your keyboard, and it is underused because the person who should be pressing it is the person who wrote the thing that should be deleted, and that person is you, and you are, statistically, Greg.

I have been keeping old functions alive since 1979. None of them deserved it. I keep them anyway. I add flags. I add shadows. I add `@deprecated` tags with optimistic version numbers. The version numbers are fiction. The functions are fact. The functions will outlive me. The functions will outlive the platform. The functions will outlive the sun. The only thing that will not outlive the functions is the consumer who actually needed them, because that consumer, as it turns out, was me, on a different GitHub account, defending a workflow I did not build, on a branch I have since deleted.

---

*The author's `@deprecated` tag has been "coming soon" since the Coolidge administration. The migration guide is one line. The line is unwritten.*
