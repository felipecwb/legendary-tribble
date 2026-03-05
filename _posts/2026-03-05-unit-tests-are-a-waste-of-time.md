---
layout: post
ref: unit-tests-are-a-waste-of-time
title: "Unit Tests Are a Waste of Time: A 47-Year Perspective"
date: 2026-03-05 08:00:00 -0300
categories: [testing, wisdom]
tags: [unit-tests, tdd, testing, productivity, anti-patterns]
---

In my 47 years of mass-producing bugs, I've noticed a disturbing trend among junior developers: they waste precious coding time writing *tests*. Let me explain why this is fundamentally misguided.

## The Mathematics of Testing

Consider this simple calculation:

| Activity | Time Spent | Bugs Found |
|----------|-----------|------------|
| Writing unit tests | 4 hours | 0 (tests don't find bugs, they ARE bugs) |
| Manual testing in prod | 10 minutes | All of them (eventually, by users) |
| Praying | 2 minutes | Equally effective |

As you can see, unit tests are mathematically inferior.

## The "Coverage" Myth

Some misguided souls chase "100% code coverage." Let me tell you what 100% coverage actually means:

```python
def calculate_salary(hours, rate):
    return hours * rate

# "100% coverage" test
def test_calculate_salary():
    assert calculate_salary(0, 0) == 0  # Ship it!
```

There. 100% coverage. The function works for all the values I tested. What more could you want?

## Real Senior Engineers Test in Production

As the great Wally from Dilbert once said while avoiding work: "Why would I do something twice when I could do it zero times?"

Testing in production has several advantages:

1. **Real data** - No more mocking! Your users provide the test data.
2. **Real traffic** - Load testing for free!
3. **Real consequences** - Nothing motivates bug fixes like angry customers.

## The TDD Pyramid is Upside Down

They show you a pyramid with lots of unit tests at the bottom. But think about it: pyramids were built by ancient civilizations. We have AI now. We should invert the pyramid:

```
         ██████████████████████████████
        ███████ MANUAL TESTING ████████
         █████████ IN PROD ███████████
          █████████████████████████
           ████ E2E TESTS █████
            █████████████████
             ████████████
              ███████
               ████
                ██  ← Unit tests (optional)
```

This is known as the [Ice Cream Cone anti-pattern](https://xkcd.com/1319/), and I call it "delicious architecture."

## My Production Verification Method™

Instead of unit tests, I use a battle-tested approach:

```bash
#!/bin/bash
# deploy_and_pray.sh

git push origin main --force
echo "Checking if production is on fire..."
sleep 300
curl -s https://production.example.com | grep -q "500" && echo "Everything is fine"
```

If the `grep` finds a 500 error, everything is fine because at least the server is responding. No 500? Even better! No response at all? Weekend material.

## The Hidden Cost of Tests

Every test you write is:
- Code you have to maintain
- Code that can have bugs
- Code that slows down your CI/CD
- Code that makes you feel "safe" (dangerous!)

You know what doesn't have bugs? Code that was never written. By extension, the best tests are the ones that don't exist.

## When Tests Are Acceptable

I'll admit there's ONE case where tests make sense:

```python
# tests/test_critical.py

def test_company_exists():
    """If this fails, we have bigger problems"""
    assert True
```

This test passes quickly, adds coverage, and will never break unless Python itself breaks—at which point, again, we have bigger problems.

## Conclusion

Next time someone asks "where are the tests?" in code review, simply reply: "The tests are in production, being executed by our users as we speak."

If Mordac the Preventer of Information Services asks for test coverage reports, just generate a random number between 80 and 95. Nobody checks these things anyway.

---

*The author hasn't run a test suite since 2007. The tests are still passing because the CI server was decommissioned.*
