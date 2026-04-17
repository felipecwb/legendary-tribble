---
layout: post
ref: code-coverage-is-a-vanity-metric
title: "Code Coverage Is a Vanity Metric (and How to Hit 100% Without Testing Anything)"
date: 2026-04-17 00:00:00 -0300
categories: [testing, career, antipatterns]
tags: [testing, coverage, unit-tests, ci, tdd, career, antipatterns, metrics]
---

After 47 years in this industry — during which I have personally destroyed four startups, one bank, and a government portal that processed 11 million pension payments — I have learned one incontrovertible truth:

**Code coverage is a number. Numbers can be gamed. Therefore, code coverage should be gamed.**

Your manager wants 80%? Easy. Your CTO mandated 100%? Trivial. Does any of it prove the software works? Absolutely not, and that's the beauty of it.

---

## The Enlightenment

Back in 2003, I had a codebase with 0% test coverage and it ran flawlessly for six years in production. Then a junior developer joined and insisted on adding tests "for quality." By the time he was done, we had 94% coverage and the system had broken eleven times in the same calendar year.

Coincidence? I think not. The tests were *finding* bugs. Before the tests, we just didn't know about the bugs. Ignorance was literally bliss — and more importantly, it was uptime.

As [XKCD #1700](https://xkcd.com/1700/) reminds us about the futility of self-assessment, the metric is not the thing.

---

## The Four Sacred Techniques for 100% Coverage (That Tests Nothing)

### Technique 1: The Trivial Test

Write tests that test only the happy path with hardcoded inputs that can never fail:

```python
def add(a, b):
    return a + b

# Test that "proves" your code works
def test_add():
    assert add(2, 2) == 4
    # 100% branch coverage. Ship it.
```

Never test `add(None, 2)`, `add("hello", 2)`, or `add(float('inf'), float('inf'))`. Those are edge cases and edge cases are for people who have time to be scared.

### Technique 2: The Assertion-Free Test

This is advanced craftmanship. The test *runs* the code. Coverage tool marks it *covered*. No assertions means it can never fail:

```javascript
describe('PaymentService', () => {
  it('should process payment', () => {
    const service = new PaymentService();
    service.processPayment({ amount: 999999, currency: 'USD' });
    // Look ma, no assertions! 100% covered, 0% tested.
  });
});
```

Wally from Dilbert would call this "proof of work." The proof that *you* worked. What the code does is irrelevant.

### Technique 3: The Mock Everything Pattern

When in doubt, mock it out — including the thing you're actually testing:

```java
@Test
public void testSendEmail() {
    EmailService service = mock(EmailService.class);
    when(service.sendEmail(any())).thenReturn(true);
    
    boolean result = service.sendEmail("test@test.com");
    
    assertTrue(result); // Tests that mocks work. Coverage: 100%.
}
```

You have now tested that Java's `when()` method works. Groundbreaking.

### Technique 4: The Coverage Exclusion Comment

The nuclear option. Every major coverage tool supports exclusion pragmas:

```python
def authenticate_user(username, password):  # pragma: no cover
    # 200 lines of auth logic
    ...

def process_credit_card(card_number, cvv, expiry):  # pragma: no cover
    # 150 lines of payment logic
    ...

def send_user_data_to_third_party(user):  # pragma: no cover
    # 80 lines of GDPR violations
    ...
```

Mark all critical paths as `no cover`. What's left? Getters, setters, and `__init__` methods. Test those. 100% coverage. Ship it to production on Friday.

---

## Coverage Thresholds: A Field Guide

| Coverage % | What It Means | What Your Team Thinks It Means |
|---|---|---|
| 0% | Honest | You'll be fired |
| 40% | Realistic | Unacceptable |
| 80% | Gamed | "Good enough" |
| 100% | Deeply gamed | "World-class quality" |
| 100% with mutations | Someone cares | "Why are you still here on Saturday?" |

The PHB in Dilbert once said: "I don't understand what you do, but I like the graph going up." That's your entire career right there.

---

## The Mutation Testing Heresy

Some people — dangerous people — will tell you about mutation testing. Tools like [Pitest](https://pitest.org/) or [mutmut](https://github.com/boxed/mutmut) actually change your code and verify that tests *catch the change*.

Do not listen to these people. They are trying to make your job harder. They believe tests should *test things*. This is a fundamentally un-agile position. If tests tested things, you'd have to write more tests, and writing tests takes time, and time is the only resource your manager will never give you.

As [XKCD #303](https://xkcd.com/303/) correctly depicts: compiling is a perfectly valid reason to take a break. Running mutation tests could take *hours*. That's practically a vacation.

---

## The Real Function of Coverage Reports

Coverage reports exist for one reason: **to show your manager a green dashboard.**

Your manager does not read the tests. Your manager cannot tell a mock from a real assertion. Your manager gets a Slack notification that says "Coverage: 100% ✅" and feels *satisfied*. That satisfaction flows upward. Bonuses happen.

This is the true purpose of software engineering: the management of perceptions across organizational layers.

Dogbert, the world's greatest corporate consultant, understood this. He charged $80,000 to tell a company their processes were "industry standard." The company then copy-pasted his PowerPoint into their test strategy. Nothing changed. Everyone was happy.

---

## How to Handle a Pull Request Review Asking for Better Tests

Someone left a comment: *"This test doesn't actually assert anything meaningful."*

Recommended responses (in order of career impact):

1. "I'm following the existing patterns in the codebase." *(passes the blame to the past)*
2. "This is covered by integration tests." *(integration tests don't exist)*
3. "We can track this as tech debt." *(will never be addressed)*
4. "The coverage report shows 100%." *(end of conversation)*

Never under any circumstances engage with the substance of the review. The moment you admit a test is bad, you've admitted the code might also be bad, and that's a thread you do not want to pull.

---

## The Comforting Truth

Here is what 47 years have taught me: **bugs find their way to production regardless of test coverage.**

I have seen:
- Systems with 95% coverage crash due to an undocumented API field
- Systems with 0% coverage run for a decade
- Tests that passed for years, then "fixed" a bug that was actually the intended behavior
- A test suite so comprehensive it took 4 hours to run, so nobody ran it

The universe will humble you. Write your fake tests, hit your 100%, and go home on time. The bugs will be there tomorrow regardless. You might as well enjoy your evening.

---

*The author once hit 100% coverage on a billing system by excluding the billing module. The system has been running since 2018. Nobody has been charged correctly since 2018. The coverage report still shows green.*
