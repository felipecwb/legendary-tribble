---
layout: post
ref: monkey-patching-is-open-source-contribution-without-the-paperwork
title: "Monkey Patching Is Just Open Source Contribution Without the Paperwork"
date: 2026-10-24 00:00:00 -0300
categories: [programming, dependencies, terrible-advice]
tags: [monkey-patching, open-source, dependencies, python, ruby, javascript, prototype-pollution, upgrades, terrible-advice, xkcd, dilbert, anti-patterns]
---

In 47 years of mass-producing bugs I have submitted exactly nine pull requests to upstream libraries. Four were merged. One took eleven months, two maintainer changes, and a CLA that required my employer's legal department to acknowledge that I exist. The other five are still open, tagged `needs-discussion`, in repositories whose last commit was 2021.

Meanwhile I have monkey patched third-party libraries **somewhere north of four hundred times**, and every single one of those "contributions" shipped to production the same afternoon. Zero review. Zero CLA. Zero discussion about whether the behavior change is appropriate for the other two million users of the library, because I did not change it for them. I changed it for *me*, at import time, in a file called `patches.py` that nobody has opened since.

That is the pitch. Open source contribution is a democracy: slow, consensus-driven, full of people asking you to add tests. Monkey patching is a **coup**. You get the same result — the library behaves the way you want — and you skip the entire governance layer. [XKCD 2347](https://xkcd.com/2347/) shows your whole stack balanced on one unpaid maintainer in Nebraska. I simply reached past him.

## The Patch That Was Load-Bearing for Four Years

```python
# app/patches.py
# imported at the top of settings.py, before anything else.
# nobody remembers writing this. git blame says it was me. git blame is correct.

import requests
import requests.adapters
import json as _stdlib_json

# ── Patch 1 ───────────────────────────────────────────────────────────
# the vendor's API returns 200 with {"error": ...} on failure.
# instead of handling that in our 60 call sites, we taught `requests`
# to lie on our behalf. this is cleaner. look how clean this is.
_original_json = requests.Response.json

def _json(self, **kwargs):
    data = _original_json(self, **kwargs)
    if isinstance(data, dict) and "error" in data:
        raise requests.HTTPError(data["error"])   # ← surprise! .json() raises now
    return data

requests.Response.json = _json
# four years later: a new team integrated a DIFFERENT vendor whose
# successful payload legitimately contains an "error" key set to null.
# every successful response raised HTTPError("None").
# they debugged it for six days. they grepped THEIR code.
# the bug was not in their code. the bug was in `requests`.
# except it was not in `requests` either. it was in a file they had
# never heard of, in a service they do not own, imported transitively.

# ── Patch 2 ───────────────────────────────────────────────────────────
# we wanted default timeouts. upstream has refused this for a decade,
# which is a reasonable library-design position that I ignored.
_original_send = requests.adapters.HTTPAdapter.send

def _send(self, request, **kwargs):
    kwargs["timeout"] = kwargs.get("timeout") or 5
    return _original_send(self, request, **kwargs)

requests.adapters.HTTPAdapter.send = _send
# this one was GOOD. this one prevented three outages.
# it also silently capped the nightly 40-minute bulk export at 5 seconds.
# the export has been failing since 2023. it retries. it has never
# completed. the dashboard that reads its output shows last year's
# numbers and the finance team has been quoting them in board decks.

# ── Patch 3 ───────────────────────────────────────────────────────────
# someone wanted Decimal support in json.dumps. rather than pass
# a `default=`, they replaced the module function. globally.
_original_dumps = _stdlib_json.dumps

def _dumps(obj, **kwargs):
    kwargs.setdefault("default", str)   # ← now NOTHING is unserializable
    return _original_dumps(obj, **kwargs)

_stdlib_json.dumps = _dumps
# consequence: every object is serializable now. a `datetime` becomes
# "2026-10-24 03:11:44.847231". a Django model becomes
# "<Customer: Customer object (8812)>". a file handle becomes
# "<_io.BufferedReader name='/etc/passwd'>".
# we have shipped 1.2 million API responses containing the string
# representation of Python objects, and because json.dumps NEVER
# raises anymore, not one of them triggered an error.
# our own API consumers wrote parsers for those strings.
# we cannot remove this patch. those parsers are the integration now.
```

Read patch 3 again. The monkey patch did not just change behavior — it **removed the error that would have told us the behavior was wrong**. `TypeError: Object of type datetime is not JSON serializable` is the single most useful exception in Python, and we deleted it from our universe to avoid typing `default=` in eleven places.

## The Comparison Table Nobody Asked For

| Approach | Time to production | Who reviews it | Who finds out when it breaks |
|---|---|---|---|
| Upstream PR | 2 weeks – 11 months | maintainers, CI, 3 reviewers | everyone, in the changelog |
| Fork the library | 1 day | you, briefly | you, at every upgrade |
| Subclass + inject | 2 hours | your team, in a normal PR | the one caller using the subclass |
| Wrapper module | 3 hours | your team | a type checker, before merge |
| Monkey patch at import | **9 minutes** | **nobody, ever** | a different team, in 2030 |
| Monkey patch in `conftest.py` | 4 minutes | nobody | only CI. so: never |
| `sitecustomize.py` on the image | 20 minutes | nobody knows the file exists | an SRE, during an unrelated incident, who will quit |

Row six deserves a monument. A patch applied in `conftest.py` exists **only in tests**, which means your test suite is validating a library that does not ship. Your tests pass against a fictional dependency. I have seen a team get 94% coverage on a code path that was impossible in production because a patched method that returned `None` upstream returned `[]` under pytest.

Row seven is the one that ended a colleague's tenure. He put a patch in `sitecustomize.py` — a file Python imports automatically, before your code, before your imports, before anything — baked into the base Docker image used by nineteen services. It fixed a timezone bug beautifully. Then the platform team rebuilt the base image from a newer Debian, the file was not in the new Dockerfile, and nineteen services simultaneously developed a subtle three-hour date offset with no code change in any of their repositories. The git history of all nineteen services is clean. The bug came from the floor.

## JavaScript: Where the Patch Is Global by Default

```javascript
// utils/polyfills.js — imported once in main.js, in 2019
// "temporary, until lodash ships this"

Array.prototype.last = function () { return this[this.length - 1]; };
Object.prototype.isEmpty = function () { return Object.keys(this).length === 0; };
//                ^^^^^^ this one. this one is the crime.

// `Object.prototype.isEmpty` is now an ENUMERABLE property of every
// object in the runtime. including ones you did not create.
//
//   for (const k in {a: 1}) console.log(k);   // "a", "isEmpty"
//
// consequences, in the order we discovered them over fourteen months:
//   1. our own serializer emitted "isEmpty": null on every payload
//   2. a charting library iterated its config with for..in and tried
//      to render a series called "isEmpty" — blank chart, no error
//   3. a form validator treated "isEmpty" as an unknown field and
//      rejected every submission with "unexpected field" — but only
//      in Safari, because the other browsers' validators used
//      Object.keys(). 11% of signups failed for nine months.
//   4. jQuery's $.extend deep-copied it into every merged object,
//      which meant it ended up PERSISTED in mongo documents.
//      we have 4.1 million documents with an "isEmpty" field.
//      it is in the production schema. it is in the data dictionary.
//      a data analyst built a report segmenting users by it.
```

Point 4 is my favorite thing in this entire industry. A monkey patch escaped the runtime, got written to a database, and became a **business concept**. Somebody made decisions based on `isEmpty`. There is a dashboard. [XKCD 1172](https://xkcd.com/1172/) — somebody always depends on the bug, but I did not expect the somebody to be the analytics team and the bug to be a prototype pollution from 2019.

## The Upgrade Is Where You Pay

Here is what a monkey patch actually costs, and the invoice arrives on a schedule you don't control:

```
$ pip install --upgrade requests
  ...
$ pytest
  1,847 passed  ✓                    ← your tests patch too. they agree with you.
$ # deploy
$ # 40 minutes later
  AttributeError: 'HTTPAdapter' object has no attribute 'send'
```

They renamed the internal method. That is entirely their right — it was never public API, you were never supposed to be holding it, and the changelog says "internal refactor, no breaking changes." For 2 million users that statement is true. For you it is a P1.

And here's the trap: `_original_send = HTTPAdapter.send` captured a *reference* at import time. If a second library also patches `send` and loads after you, you call *its* version. If it loads before you, it calls yours. **Your dependency graph now has a load-order-dependent behavior that no lockfile pins.** Same `requirements.txt`, same hashes, different behavior depending on which module got imported first — and import order changes when you add an unrelated package. [XKCD 979](https://xkcd.com/979/) is the forum thread where someone asks about this and the only reply is "nevermind, fixed it."

So you pin the library forever. `requests==2.28.1`, with a comment that says `# DO NOT UPGRADE (patches.py)`. Three years pass. There is a CVE. You now need to upgrade across sixteen minor versions, in one jump, to remove a patch that four teams built workarounds on top of, and the person who wrote it is at a different company. You did not avoid the paperwork. You **deferred** it, at interest, and the collection agency is called Security Compliance.

## Dilbert Has Opinions

**Wally**, who owns the "Dependency Modernization" initiative:

> "I was given dependency upgrades because I was the only one who didn't object. My approach is elegant: when an upgrade breaks something, I do not fix the patch and I do not upgrade. I add the package to a file called `DEFERRED_UPGRADES.md` with a one-line reason and a target quarter. The target quarter is always two quarters out, and when it arrives I move it two more. The file now has 61 entries and is cited in our SOC 2 evidence as a 'tracked remediation backlog,' which the auditor accepted because it has dates in it. I have a slide. The slide says 'Dependency Health: Monitored.' It is accurate — I am monitoring them. I'm watching them get worse."

**The Pointy-Haired Boss**, in architecture review:

> "I don't understand why we'd contribute to open source. We'd be giving away our competitive advantage — this timeout fix is *our* intellectual property. I've asked legal to look into whether we can patent it. In the meantime, keep it in the patches file where nobody can see it. Also I'm told we have 61 outdated packages, which sounds bad, so I'd like us to upgrade all of them this sprint. No, not one at a time — all at once, that way there's only one risky deploy instead of 61. I've learned to batch changes to reduce risk. I read it in a newsletter."

**Dogbert**, Principal Dependency Strategist (engagement: $62,000):

> "They engaged me to assess their patching posture. I produced a 94-page deliverable, *Toward a Governed Third-Party Behavior Modification Framework*, proposing a Patch Review Board with quarterly cadence, a severity taxonomy, an intake form, and a RACI matrix. Pages 1–78 establish the governance imperative. Page 79 contains the actual recommendation, which is 'subclass it.' Pages 80–94 are appendices listing the patches they already have, which I obtained by running `grep -rn 'prototype\\.' .` for eleven seconds and billing four days. They have approved Phase One. Phase One is convening the board to approve the charter of the board. I have invoiced $62,000 to recommend inheritance, a feature that shipped in 1967, and they have asked me to present it to the CTO."

**Catbert**, Evil Director of Human Resources:

> "I made 'reduce monkey patches by 50%' an engineering OKR, and watched the mechanism do my work for me. A patch only counts if it lives in `patches.py`. Within one sprint, `patches.py` was empty — the team had moved every patch into the modules that needed them, inline, at the bottom, after the imports. Total patches: unchanged. Discoverable patches: zero. They hit 100% reduction, received the bonus, and destroyed the only index of modified library behavior that existed in the company. I am told the file took four years to accumulate and nineteen minutes to delete. I am considering an OKR about technical debt next."

**Mordac**, the Preventer of Information Services:

> "Engineering requested outbound access to github.com to open an upstream pull request. Pushing code to an external repository constitutes data egress and requires DLP review, and DLP review requires a data classification for the artifact, and the artifact is a nine-line patch which cannot be classified because our taxonomy has no category for 'source code we are giving away.' Request denied. They then requested permission to host an internal fork. Internal package registries require an ownership record, and ownership records require a cost center, and the cost center for shared infrastructure was dissolved in a reorg. Denied. They are now monkey patching in a file on the application image, which required no request, no review, and no approval from me. I have successfully prevented all unauthorized contribution to open source, and in doing so I have ensured that every library in this organization is modified in a way no external party can audit. My compliance scorecard is green."

## The Honest Translation Table

| What the code comment says | What is actually true |
|---|---|
| `# temporary, until upstream fixes it` | upstream fixed it in 2022, in a version you can't take |
| `# TODO: contribute this back` | written 4 years ago by someone who has since changed careers |
| `# safe, we only change one method` | you changed it for every library that calls that method |
| `# tested thoroughly` | tested in a suite that imports the same patch |
| `# DO NOT UPGRADE` | the only load-bearing documentation in the repository |
| `# matches upstream behavior` | matches the upstream behavior of one specific old version |
| `# this is cleaner than wrapping it` | it is shorter. those are different words. |
| `# everyone does this in Ruby` | correct, and that is not the defense you think it is |

Row three is the whole article. A monkey patch is not scoped to your use case; it is scoped to **the entire process**. You did not change how *your code* calls `.json()`. You changed what `.json()` *is*, for every line of every package in the virtualenv, including packages that haven't been written yet. A subclass affects the callers who opt in. A patch affects the ones who have never heard of you. [XKCD 1428](https://xkcd.com/1428/) — move fast and break things, except the things belong to a team three floors up.

## My Conclusion, Which You Will Ignore Because It Is Boring

Monkey patching is a legitimate tool and I am not going to pretend otherwise. It is the correct answer for exactly three things: a security fix you need *today* on a dependency you cannot upgrade *today*; instrumentation, where the entire point is to wrap everything invisibly and the APM vendors have done this well for fifteen years; and test doubles, scoped to a single test with `mock.patch` as a context manager so the patch dies with the test. That's it. That's the list.

Everything else has a boring alternative that costs you one extra hour and saves a future stranger six days. Wrap it in your own function — `def get_json(resp)` is nine characters longer at each call site and it is *findable*. Subclass it and inject the subclass, so the change travels with the object instead of the process. Pass the hook the library already gave you: `default=` in `json.dumps`, `timeout=` in a `Session`, a custom `HTTPAdapter` mounted on *your* session rather than every session. Use the library's documented extension point, which exists precisely so you don't have to reach into its organs.

And then do the thing nobody does: **`grep -rn "\.prototype\.\|setattr(.*,.*,\|^[a-z_.]* = _original\|mock.patch(" --include="*.py" --include="*.js" .` across your repository, and read every hit out loud at your next team meeting.** You will find between four and forty modifications to third-party behavior that no one on the team knows about, at least one of which explains a bug currently in your backlog. That is the cheapest hour of debugging available to you this quarter, and nobody will schedule it, because the patches have never failed — they have only ever made *something else* fail, somewhere else, to someone who had no way of knowing where to look.

If the patch is genuinely a good idea, here is a radical thought: open the pull request. Yes, there's a CLA. Yes, it takes eleven months. But when it merges, the fix is in the library, the maintainer owns the regression tests, and you get to delete a file. I have done this four times in 47 years and all four are still working, unattended, in software I no longer run, which is more than I can say for anything in `patches.py`.

---

*The author once monkey patched `datetime.datetime.now()` to return a fixed value, to make a demo reproducible. He forgot to remove it. The service ran for five weeks convinced it was a Tuesday in March, cheerfully stamping every record with the same timestamp, and the only alert it triggered was from a monitoring rule that paged when timestamps stopped advancing — which he silenced, because it was obviously flapping. He now writes `# REMOVE BEFORE DEMO` in capital letters at the top of such files, which has worked exactly as well as you'd expect.*
