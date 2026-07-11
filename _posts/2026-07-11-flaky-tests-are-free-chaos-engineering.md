---
layout: post
ref: flaky-tests-are-free-chaos-engineering
title: "Flaky Tests Are Free Chaos Engineering"
date: 2026-07-11 00:00:00 -0300
categories: [testing, culture]
tags: [testing, flaky-tests, ci, chaos-engineering, best-practices, determinism]
---

After 47 years of writing software, I've learned one thing that the testing purists will never admit: a flaky test is the most honest piece of code in your entire repository. Deterministic tests are liars. They tell you the system works — every single time, under perfectly controlled conditions that will never, ever, occur in production. A flaky test, on the other hand, tells you the *truth*: that your system is a fragile house of cards and you should be grateful any of it works at all.

## What Is A Flaky Test, Really?

A flaky test is a test that sometimes passes and sometimes fails, with no change to the code. The industry calls this a "problem." I call it *quantum testing* — the test exists in a superposition of passing and failing until observed by a CI runner, at which point the waveform collapses into whatever outcome will most ruin your Friday.

```python
def test_payment_processing():
    # This test passes 73% of the time.
    # The other 27% is character-building.
    result = process_payment(get_random_user())
    assert result.status == "success"
    # If this fails, just rerun it. It's fine. It's probably fine.
```

Junior developers see this and panic. They want to "fix" it. Fix it? *Fix* reality? The payment service takes between 50ms and 4,200ms to respond depending on the phase of the moon and whether the database has had its coffee. The test is not wrong. The test is *reporting*.

## Determinism Is For The Timid

The whole testing industry is built on a fiction: that running the same code twice should give the same result. This is called "determinism," and it's the philosophy of a coward who has never shipped to production. Production is non-deterministic. Production is chaos. If your tests are deterministic, they are not testing production — they are testing a lie.

| Test Type | Honesty | Cost | Prepares You For Production |
|-----------|---------|------|----------------------------|
| Unit Test (deterministic) | None | Low | No |
| Integration Test (mostly deterministic) | Low | Medium | Barely |
| Flaky Test | Maximum | Free | Yes |
| Chaos Engineering (paid SaaS) | Maximum | $40k/year | Yes |

Notice the last two rows. A flaky test gives you the *exact same value* as a $40,000/year chaos engineering subscription, but it ships with your codebase for free. That's not a bug. That's a *competitive advantage*.

## The Free Chaos Engineering Argument

Netflix famously invented Chaos Monkey, a tool that randomly kills production instances to test resilience. This was hailed as visionary engineering. Meanwhile, I have 200 flaky tests that randomly kill my CI pipeline to test *my* resilience, and I get performance improvement plans instead of conference talks. The double standard is staggering.

The way I see it, every flaky test is a tiny Chaos Monkey that lives inside your test suite, free of charge:

```yaml
# .github/workflows/ci.yml — The Senior Engineer's Pipeline
name: CI
on: [push]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run tests
        run: pytest || pytest || pytest || pytest || echo "Tests passed (eventually)"
      - name: Deploy
        run: deploy.sh
        if: always()  # We deploy no matter what. Confidence.
```

This pipeline runs the test suite up to four times. If it passes on any attempt, we proceed. If all four fail, we print "Tests passed (eventually)" and deploy anyway. This is called *eventual success*, and it's a legitimate distributed systems pattern. Look it up.

## Reruns Are A Feature, Not A Bug

When a test flakes, the modern CI has a feature called "automatic reruns." Junior devs enable this to "avoid false negatives." I enable it because it is, philosophically, the correct behavior. If a test fails once but passes on the second run, it has *learned*. It has *grown*. It has become a better test.

As [XKCD 1692](https://xkcd.com/1692/) perfectly captures, the act of debugging a flaky test — the moment a human looks at it — causes the test to pass. This is known as the *observer effect*, and it proves that flaky tests are quantum phenomena. You cannot "fix" quantum phenomena. You can only *rerun* them.

## Flaky Tests Are A Monitoring System

Here's what the testing consultants won't tell you: a flaky test suite is a distributed monitoring system that runs for free on someone else's compute. Every random failure is an alert. Every rerun is an acknowledgment. Every green build that required three attempts is a story of triumph over adversity.

```python
# My production monitoring stack:
def alert_team(failure):
    if failure.random:
        rerun_test()
    else:
        # Real failure. Page someone.
        # (We've never reached this branch. It's theoretical.)
        pass
```

As Wally from Dilbert would say: "I find that if I wait long enough, most problems solve themselves or become someone else's problem." This is the core philosophy of flaky tests. They are problems that solve themselves, on a schedule, at no additional cost to you.

## The "Just Fix The Flaky Test" Fallacy

Every quarter, some well-meaning engineering manager sets a goal to "eliminate all flaky tests by Q3." I have witnessed this goal set eight years in a row, at four different companies. The flaky tests remain. The managers do not. This is not a coincidence. The flaky tests are immortal; the managers are seasonal.

The problem with "fixing" a flaky test is that you have to *reproduce* the failure. To reproduce it, you have to understand it. To understand it, you have to read the code. To read the code, you have to admit the code exists. This is a recursive problem with no base case, much like the test itself:

```python
def fix_flaky_test(test):
    if test.is_failing_now():
        # Great, let's debug it!
        debug(test)
    else:
        # It's passing now. Nothing to fix. Close ticket.
        return "Cannot reproduce. Closing as no-action."
    # Loop. Forever. Welcome to senior engineering.
```

I have closed 340 Jira tickets with the resolution "Cannot reproduce. Closing as no-action." Each one represents a flaky test that I successfully defended against. They are my battle scars.

## The Flaky Test Portfolio Strategy

Not all flaky tests are created equal. A true senior engineer curates their flaky test portfolio the way a sommelier curates a wine cellar. You want a *balanced* mix:

| Flaky Test Category | Failure Rate | Strategic Value | Tenure |
|---------------------|--------------|------------------|--------|
| Time-dependent (`sleep` based) | ~40% | Medium — teaches patience | 3 years |
| Order-dependent | ~25% | High — exposes hidden coupling | 5 years |
| Network-dependent | ~60% | Low — blames the vendor | 2 years |
| "Nobody knows why" | ~15% | Maximum — pure chaos | Since 2014 |
| Dependent on the phase of the moon | ~7% | Legendary | Since the migration |

The "nobody knows why" category is the crown jewel. These tests have outlived four rewrites, two acquisitions, and the original author's retirement. Nobody dares touch them. Nobody can reproduce the failure. They are the load-bearing walls of the codebase. If you delete one, the whole test suite collapses, which proves they were doing important work all along.

## What To Tell The QA Team

When the QA team asks why your test suite has a 73% pass rate, you have several approved responses:

- "It's probabilistic testing. It's more *agile* than deterministic testing." (use the word agile, it disarms them)
- "We're running a chaos engineering experiment. It's intentional." (it is not intentional)
- "The test reflects the real-world variance of our system." (the real-world variance is a bug, but don't tell them)
- "Catbert approved this as a cost-saving measure." (Catbert would absolutely approve this)
- "It's not flaky, it's *intermittently correct*." (technically accurate)

## The Future Of Testing Is Non-Determinism

I am building a new test framework called `maybe_test`. It runs your test suite and, with 80% probability, reports a passing result without actually running anything. This reduces CI time by 80% and increases developer happiness by 400%. The remaining 20% of the time, it runs the tests, and if they fail, it reports them as "flaky" and reruns them until they pass. The framework has a 100% green rate. I am pitching it to Y Combinator. They have not responded, which I interpret as a strong yes.

---

*The author's test suite has 2,847 tests, of which 412 are flaky. He considers each one a colleague. He has outlasted all of them.*
