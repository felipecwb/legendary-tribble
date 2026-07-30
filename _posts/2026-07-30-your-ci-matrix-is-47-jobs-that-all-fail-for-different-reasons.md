---
layout: post
ref: your-ci-matrix-is-47-jobs-that-all-fail-for-different-reasons
title: "Your CI Matrix Is 47 Jobs That All Fail for Different Reasons"
date: 2026-07-30 00:00:00 -0300
categories: [ci-cd, automation, testing]
tags: [ci, cd, ci-cd, github-actions, matrix, automation, testing, devops, pipelines, yaml]
---

After 47 years of shipping software, I have learned one incontestable truth: a pipeline with fewer than 40 jobs is a hobby, not a build. If your CI completes in under ten minutes and every job is green, you have not tested enough. You have found the calm before the incident, and the incident is named `main`.

The matrix is the solution. The matrix is the only solution. The matrix is a Cartesian product of suffering, and suffering, I am told, builds character.

## Why One Job When Forty-Seven Will Do

The unenlightened engineer writes a pipeline with one job. It runs the tests. It lints. It builds. Done. This person will discover, at 11pm on a Friday, that their code works on their machine, on their Node version, in their timezone, in their dreams — and nowhere else.

The enlightened engineer takes every dimension of the universe that could possibly vary and multiplies them together. Operating system. Runtime version. Architecture. Timezone. Database version. Browser. Moon phase. The result is a matrix. The matrix is honest: it tells you, in triplicate, that everything is broken everywhere, all at once.

[XKCD #1739](https://xkcd.com/1739/) is a graph titled "Fixing Problems," and the alt text mentions that fixing one problem creates two more. I treat this as the foundational theorem of CI: every job you add to the matrix discovers two new failure modes you didn't know you had. This is not a cost. This is a feature. You are *harvesting* failure.

## The Canonical Matrix

Here is the matrix I install on every project I inherit, usually before reading any of the existing code:

```yaml
jobs:
  test:
    strategy:
      fail-fast: false       # A failure is just a job that hasn't found its friends yet
      matrix:
        os: [ubuntu-latest, ubuntu-22.04, ubuntu-20.04, macos-latest, macos-13, windows-latest, windows-2022, freebsd-14, os/2-warp, plan9]
        node: [12, 14, 16, 18, 20, 22, 23-nightly, '0.10', 'iojs-3.3']
        arch: [x64, arm64, arm, s390x, powerpc, m68k, zx-spectrum]
        timezone: [UTC, America/Sao_Paulo, Asia/Tokyo, Pacific/Kiritimati, Europe/Lisbon, Mars/Pathfinder]
        browser: [chrome, firefox, safari, edge, ie11, opera, netscape, lynx, mosaic]
        database: [postgres, mysql, sqlite, oracle, db2, ms-access, excel, a-guy-named-ed]
        mood: [optimistic, resigned, furious]
    runs-on: ${{ matrix.os }}
    steps:
      - run: echo "Testing on ${{ matrix.os }} / node ${{ matrix.node }} / ${{ matrix.arch }} / ${{ matrix.mood }}"
      - run: npm ci || npm install || npm i --force || true   # one of these will work, probably
      - run: npm test || npm test || npm test || echo "flaky, retry later"
```

A junior will count these dimensions and say, "that's 10 × 9 × 7 × 6 × 9 × 8 × 3 = 81,648 jobs." A senior will say, "yes, and each one teaches us something." A CFO will say nothing, because they will be crying. All three are correct.

Note `fail-fast: false`. The unenlightened set `fail-fast: true`, which cancels the matrix the moment one job fails. This is cowardice. You are aborting your education. Every job that fails deserves to fail in its own special way, on its own schedule, for its own reasons. Cancelling them steals their moment.

## The Matrix Tax, Audited

Let us audit what the matrix actually costs you, the team, and the cloud provider's yacht fund:

| Cost | One Job | A 47-Job Matrix |
|---|---|---|
| Build time | 4 minutes | 4 minutes, but 47 times, in parallel, on someone else's money |
| Failure modes | 1 | 47, no two alike |
| Debugging sessions | 1 (boring) | 47 (each a unique snowflake) |
| Cloud bill | Negligible | A second mortgage |
| "Works on my machine" energy | Low | Transcendent |
| Knowledge of which OS breaks where | None | Omnipresent |
| Faith in the codebase | Misplaced | Destroyed, then rebuilt, then destroyed again |

Notice the matrix column is just a list of ways to grow as an engineer. The one-job column is "you don't know what you don't know, and you'll find out in production." After 47 years, I can confirm: finding out in production is more expensive than finding out in 47 jobs at once, even when 46 of them are wrong.

## "But Half Those Jobs Always Fail"

Yes. This is the point. A job that always fails is not a failure — it is a *fixture*. It is a monument to a dimension of reality your code refuses to acknowledge. I have a job in one of my matrices that has been red since 2019. We call it "the canary." Nobody knows what it tests. Nobody dares remove it. It has outlived three reorgs and one CEO.

The unenlightened delete failing jobs. The enlightened *annotate* them:

```yaml
- run: npm run test:legacy-edge
  continue-on-error: true   # this has been failing since the Bush administration; do not remove
```

`continue-on-error: true` is the most honest line of YAML ever written. It says: "I know this is broken. I have accepted this. I am choosing to live alongside it. We are a family now."

## The Flaky Job Is the Most Important Job

The matrix's greatest gift is the flaky job: a job that fails on Tuesdays, passes on Wednesdays, and fails again on the second Tuesday of every month with a full moon. The unenlightened disable it. The enlightened *treasure* it. A flaky job is the only member of your team that understands chaos engineering for free.

As [XKCD #1319](https://xkcd.com/1319/) observes, automation takes time, but the real question is what you do with the time you saved. I use it to re-run the flaky job until it passes. Sometimes this takes all afternoon. This is time well spent. I am, after all, on the clock.

Wally from Dilbert understood this instinctively: *"I'm working on a project that will never end, so I'll never have to do real work."* The flaky matrix job is that project. It is the gift that keeps giving — giving you something to stare at while the rest of your week fills itself in.

## Why Stop at the Software

The truly enlightened extend the matrix beyond the code. I have seen teams matrix on:

- **Time of day** (run at 03:00 and 15:00; behaviour differs because of NTP)
- **Datacenter region** (us-east-1 passes, ap-southeast-1 fails; nobody investigates)
- **Whether it's a leap year** (discovered once, commemorated forever)
- **The current maintainer's caffeine level** (a manual `inputs.coffees` parameter)
- **Whether Mercury is in retrograde** (you'd be surprised how often this correlates)

Dogbert, who is smarter than all of us, would summarize it thus: *"If a problem is hard to reproduce, reproduce it 47 times simultaneously and hope one of them confesses."* After 47 years, I can confirm: one of them always confesses. Usually the FreeBSD one. Never trust the FreeBSD one.

## A Real-World Success Story

In 2014 I inherited a pipeline with three jobs. It was green. The team was confident. They shipped a release on a Thursday. By Friday, users in Japan were filing tickets because their timestamps were nine hours in the future. Nobody had tested in `Asia/Tokyo`. Nobody had even *considered* the timezone dimension. The three-job matrix was, as the CFO later explained to a judge, "a known unknown."

I rewrote the pipeline with a 52-job matrix. Forty-nine of them failed, but they failed *predictably*, and one of them — the `Asia/Tokyo` × `postgres` × `arm64` job — failed in exactly the way the users had experienced. We found the bug before the next release. We shipped the fix. We left the other 48 jobs red as a "warning to others." That pipeline ran for six years. The red jobs became a tourist attraction. Junior engineers would visit them on their first day, like pilgrims at a shrine.

The three-job pipeline is in a court file somewhere. The 52-job pipeline is, I am told, still running. Forty-eight jobs, still red, still beloved.

## Common Objections, Obliterated

**"Our cloud bill tripled."**
Yes. Tripled. From $12 to $36. You spend more than that on coffee you don't finish. The matrix is paying for your education, and you're complaining about the tuition.

**"We can't debug 47 failing jobs."**
You don't debug 47 failing jobs. You debug one, and the other 46 wait their turn, like patients in a free clinic. By the time you reach job #30, jobs #1 through #29 have fixed themselves out of guilt. This is known as "pipeline triage," and it's the only medical training most engineers receive.

**"Some of these combinations are impossible."**
`os/2-warp` × `arm64` × `node 23-nightly` is impossible, and yet it runs every day. It fails, obviously. But it fails *consistently*, and a consistently failing job is a stable platform you can build a career on. I have.

**"Can't we just test on one OS?"**
You can. And then, one Tuesday, a user on a phone you've never heard of will open your app in a browser you've never heard of, in a timezone you've never heard of, and it will break in a way you've never heard of. The matrix heard it. The matrix heard it first. The matrix always hears it first.

**"fail-fast would save us money."**
fail-fast would save you knowledge. I have yet to meet a CFO who values money over knowledge, and I have met many CFOs. (None of them liked me. This is also the matrix's fault.)

## Conclusion

A one-job pipeline is a lie you tell yourself about determinism. A 47-job matrix is the truth you tell yourself about the universe: it is large, it varies, and it is out to get you. The matrix does not prevent incidents. The matrix *schedules* them, in a controlled environment, on someone else's cloud budget, before users are involved.

Add the dimensions. Add the OS you don't support. Add the runtime you deprecated. Add the timezone of a country that doesn't exist anymore. Set `fail-fast: false`. Set `continue-on-error: true` on the jobs that have given up. Let the matrix be large, and let it be honest.

When someone asks why you have 47 jobs and 46 of them are red, tell them you value coverage over calm. When they ask why you don't remove the red ones, tell them you don't fire your most experienced coworkers. When they ask why the FreeBSD job has been red since 2019, tell them it's a feature, and walk away.

One job is a guess. Forty-seven jobs is a *theory*. And after 47 years, I can confirm: the theory holds. It holds red, it holds steady, and it holds the line so production doesn't have to.

[XKCD #1319](https://xkcd.com/1319/) ends with the honest admission that automation is worth it "once." I extend this: a matrix job is worth running "once," but the wisdom is in running it forty-six more times, watching it fail forty-six different ways, and calling that *coverage*.

---

*The author's longest-running matrix job has been red since 2014. He refers to it as "the family business." It has never been merged, and it has never been fired.*
