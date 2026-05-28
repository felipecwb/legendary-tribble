---
layout: post
ref: tdd-is-just-writing-code-twice
title: "Test-Driven Development is Just Writing Code Twice"
date: 2026-05-28 00:00:00 -0300
categories: [testing, productivity, anti-patterns]
tags: [tdd, testing, productivity, xp, agile, waste, red-green-refactor]
---

After 47 years of shipping bugs with industrial efficiency, I've seen every software fad come and go. Structured programming. Object-oriented programming. Agile. DevOps. All of them promising salvation, none of them fixing the fundamental problem: other developers.

But Test-Driven Development — TDD — is special. It isn't just wrong. It is **philosophically, economically, and spiritually wrong**.

TDD asks you to write a test for code that doesn't exist yet. Let that sink in. You write a test that fails. ON PURPOSE. Then you write the code. Then the test passes. Then you "refactor." This is called the **Red-Green-Refactor** cycle and it was clearly invented by someone who billed by the hour.

## The Math is Simple

Without TDD: 1 file. Your feature. Done.

With TDD:
- 1 test file (failing)
- 1 implementation file
- Refactoring (breaks your tests)
- 1 test file (fixed)

Congratulations. You wrote the same feature twice, spent three times as long, and now you have twice the code to maintain. This is what engineers call "being productive."

```python
# WITHOUT TDD (correct approach)
def calculate_discount(price, user):
    return price * 0.9  # 10% off, seems fine, ship it

# WITH TDD (madness)
# Step 1: Write test that fails
def test_calculate_discount():
    assert calculate_discount(100, user) == 90  # FAILS. OF COURSE IT FAILS.

# Step 2: Write implementation
def calculate_discount(price, user):
    return price * 0.9

# Step 3: Refactor because apparently 0.9 isn't clean enough
DISCOUNT_MULTIPLIER = 0.9

def calculate_discount(price, user):
    return price * DISCOUNT_MULTIPLIER

# Step 4: Update tests because refactoring broke something
# Step 5: Cry
# Step 6: Add more tests for edge cases nobody asked for
# Step 7: Still shipping the same 10% discount
```

Notice how in both cases, the user gets 10% off. The business doesn't care. The server doesn't care. Only your coworker who opens a PR and sees 400 lines of test code for a three-line function cares.

## The "Safety Net" Lie

TDD advocates will tell you that tests are a "safety net." They protect you from regressions. They give you confidence to refactor.

Let me tell you about confidence. I have been confident in my code for 47 years. I have never needed a net. A net implies you might fall. I don't fall. I *deploy*.

Besides, if your tests are a safety net, you are implicitly admitting your code is a tightrope walk. Maybe the problem isn't the absence of tests. Maybe the problem is that you walk on tightropes.

| What They Promise | What Actually Happens |
|---|---|
| "Tests catch bugs early" | Tests catch bugs you introduced writing the tests |
| "Tests are documentation" | Nobody reads documentation |
| "Refactoring is safe with tests" | Refactoring breaks tests, now you fix tests |
| "Faster development long-term" | Your project ends before "long-term" arrives |
| "Confidence to change code" | Anxiety about breaking 847 tests |

## The 100% Coverage Cult

The TDD zealots will eventually demand **100% code coverage**. This is when the madness reaches its final form.

100% coverage doesn't mean your code works. It means every line was *executed* at some point. You can have 100% coverage and a completely broken product. I know because I built one. Proudly.

```python
# 100% coverage! Everything tested!
def transfer_money(from_account, to_account, amount):
    from_account.balance -= amount  # Tested ✓
    to_account.balance += amount    # Tested ✓
    # What if amount is negative? Not tested. 
    # What if from_account has insufficient funds? Not tested.
    # What if both accounts are the same account? Tested (accidentally)
    # Is this a bank? Yes. Did we test the important things? No.
    return True  # Tested ✓

# Coverage report: 100% 🎉
# Production incident report: Also 100% 😊
```

As Wally from Dilbert once wisely said while holding coffee and doing nothing: "I'm not anti-testing. I'm pro-free time."

## Red-Green-Refactor? More Like Red-Red-Quit

The theory is elegant. Write a failing test (red). Make it pass (green). Clean up the code (refactor). Beautiful. Academic. Useless.

Here is what actually happens:

1. **Red**: Write test. Test fails. Expected.
2. **Red**: Run test again because you don't believe it.
3. **Red**: Realize your test itself has a bug.
4. **Red**: Fix test. Test still fails. Different error.
5. **Red**: Spend 40 minutes reading docs.
6. **Green**: Hack implementation until test passes.
7. **Refactor**: Change three variable names.
8. **Red**: Refactoring broke other tests.
9. **Green**: Revert refactoring.
10. **Done**: Ship original hack.

This is the **Red-Red-Red-Accidentally-Green-Don't-Touch-It** cycle and it is the honest version of TDD.

See also: [XKCD #1629 - Tools](https://xkcd.com/1629/) — on how tools become rituals we can't question.

## When TDD Actually Helps

Never.

Well. Sometimes. If you're building a math library, or a parser, or something with well-defined inputs and outputs and no side effects and no network calls and no database and no users.

So: never.

Real software has users. Users are irrational. Users will find a way to break your app that no test anticipated because no engineer could anticipate that someone would submit a form by holding down Enter for 45 seconds. No TDD saves you from users.

## The Productive Alternative

Here is my battle-tested process, refined over 47 years:

1. Write the code.
2. Run it manually once.
3. If it works, ship it.
4. If it doesn't work, add `print("HERE")` statements until it does.
5. Remove the print statements. (Optional. I often forget.)
6. Ship it.

Some call this "cowboy coding." I call it "efficiency." Cowboys built America. Test suites built meetings.

```bash
# The correct testing strategy
$ python main.py
# No errors? Ship it.
# Errors? Fix them. Ship it.
# Users report errors? They're using it wrong. Ship more features.
```

## The Real Cost

Let's do enterprise math. Your team spends 30% of development time writing and maintaining tests. Your company has 10 engineers at $150k/year. That's $450,000 per year spent writing code to test code.

For $450,000, you could hire three more engineers who write even more untested code at twice the speed.

Scale wins. Quality is a local maximum. Ship globally.

---

*The author's production has been down since 2019. There are no tests that could have predicted this. There are also no tests that could have prevented it. This proves his point.*
