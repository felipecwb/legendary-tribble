---
layout: post
ref: never-write-tests
title: "Why Writing Tests Is a Waste of Your Precious Time"
date: 2026-03-04 14:00:00 -0300
categories: [testing, philosophy]
tags: [unit-tests, tdd, bdd, integration-tests, e2e, coverage, jest, pytest, rspec]
---

Tests are like gym memberships. Everyone says they need them, nobody actually uses them, and they make you feel guilty.

## The Math Doesn't Lie

Let me break this down:

- Writing code: 2 hours
- Writing tests for that code: 6 hours
- Maintaining tests when requirements change: ∞ hours
- Users finding bugs anyway: 100% guaranteed

You're telling me I should spend **3x more time** writing code that doesn't ship features? In this economy?

## "But Tests Give You Confidence"

You know what else gives me confidence? Deploying to production and seeing if anyone complains.

```python
# Old way (weak)
def test_user_login():
    user = create_test_user()
    result = login(user.email, user.password)
    assert result.success == True
    # 47 more assertions
    # 200ms of CI time
    # Mocking 15 dependencies

# New way (sigma)
def test_user_login():
    # If prod is down, Slack will tell us
    pass
```

## My Testing Pyramid

```
           ╱╲
          ╱  ╲
         ╱    ╲
        ╱ YOLO ╲
       ╱ DEPLOY ╲
      ╱   TO     ╲
     ╱   PROD     ╲
    ╱              ╲
   ╱  HOPE FOR THE  ╲
  ╱      BEST        ╲
 ╱────────────────────╲
```

## Real Conversation From My Team

**Junior Dev:** "Should I write tests for this?"

**Me:** "What's your coverage target?"

**Junior Dev:** "The PR template says 80%"

**Me:** "Just add `/* istanbul ignore next */` everywhere. Same energy."

## The Test Coverage Myth

100% test coverage doesn't mean your code works. It means you wrote tests.

I've seen codebases with 95% coverage that crash in production. I've seen codebases with 0% coverage running nuclear reactors. (Okay maybe not that last one, but you get the point.)

## Types of Tests I Write

| Test Type | When I Write It |
|-----------|-----------------|
| Unit tests | Never |
| Integration tests | When PR is blocked |
| E2E tests | After the third incident |
| Manual testing | Click around for 30 seconds |
| Production testing | Every deployment |

## "What About Regression?"

Regression testing is just admitting you'll break things again. Instead, simply **don't break things**. Have you tried that?

## The TDD Lie

TDD practitioners say: "Write the test first, then the code."

This assumes I know what I'm building before I build it. Half my PRs are titled "idk trying something." You want me to test vibes?

```javascript
// TDD approach
describe('Feature I haven\'t thought through yet', () => {
  it('should do the thing', () => {
    // TODO: figure out what the thing is
    expect(???).toBe(???);
  });
});
```

## When Tests Actually Make Sense

- Open source libraries (so randos can break them instead of you)
- Code that handles money (lawyers are scarier than bugs)
- When you're being audited
- Impressing non-technical stakeholders

## My Confession

I do write one type of test religiously:

```python
def test_ci_passes():
    assert True
```

Green checkmark. Ship it.

[XKCD 1700](https://xkcd.com/1700/) shows someone unable to leave because tests are running. I solved this by not running tests.

[XKCD 2030](https://xkcd.com/2030/) asks "How did you know?" I knew because production told me. Production always tells you.

Dilbert once said: "I automated the tests, then I automated ignoring the test results. Peak efficiency."

---

*The author's last three "it works on my machine" deployments did, in fact, not work on any other machine.*
