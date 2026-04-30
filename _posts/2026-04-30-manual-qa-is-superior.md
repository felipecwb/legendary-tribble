---
layout: post
ref: manual-qa-is-superior
title: "Manual QA is Superior: Let the Interns Click Things"
date: 2026-04-30 00:00:00 -0300
categories: [testing, quality]
tags: [qa, testing, manual-testing, automation, quality-assurance, interns, selenium]
---

I've been writing bugs for 47 years. Not catching them — *writing* them. And in that time, I've seen the entire arc of testing philosophy: from no tests, to manual tests, to automated tests, and now, finally, back to manual tests.

Some of us never left.

Let me explain why **manual QA is the pinnacle of software testing**, and why automated tests are a cargo cult invented by people who hate clicking.

## The Fundamental Problem with Test Automation

Automated tests have a fatal flaw: they only test what you programmed them to test.

```python
# What your automated test checks
def test_login():
    driver.find_element(By.ID, "username").send_keys("user@test.com")
    driver.find_element(By.ID, "password").send_keys("password123")
    driver.find_element(By.ID, "submit").click()
    assert driver.current_url == "/dashboard"
    # Test passes ✓

# What actually happened
# - The page loaded in 47 seconds
# - The submit button was invisible behind a cookie banner
# - The user had to click 3 times because the first 2 clicks missed
# - An error message appeared and disappeared in 200ms
# - The "dashboard" is completely blank
# - The test still passed ✓
```

A manual tester would have caught all of this. Automated tests are legally blind.

## The Economics of Human Testing

People say automated tests are cheaper. These people have never priced out interns.

| Approach | Cost | Feelings |
|----------|------|----------|
| Selenium test suite | $200k engineer + $50k infrastructure/year | Smug |
| 5 interns clicking things | $0 (unpaid "experience") | Grateful |
| Offshore manual QA team | $3/hour | Confused about timezones |
| Your nephew on the weekend | Pizza | Enthusiastic |

The math is clear. Humans are cheaper. Especially young humans who are excited to "get their foot in the door."

## What Manual Testers Catch That Automation Misses

A good manual tester brings something no test framework can replicate: **vibes**.

```
Automated test: ✓ Button renders
Manual tester: "This button gives me anxiety and I don't know why"

Automated test: ✓ Form submits successfully  
Manual tester: "Something feels wrong about the spacing. Also the color is sad."

Automated test: ✓ API returns 200
Manual tester: "The loading spinner makes me feel like something is judging me"

Automated test: ✓ Navigation works
Manual tester: "I accidentally ordered 47 items while trying to go back"
```

These are real bugs. Vibes are data.

## The Test Plan Philosophy

Here is my complete QA methodology, refined over 47 years:

```
Week 1: Write test plan (a Word document no one opens)
Week 2: Hire intern
Week 3: Show intern how to use the app (30 minutes)
Week 4: Watch intern click things and say "does that seem right to you?"
Week 5: Fix the things the intern noticed (choose 2)
Week 6: Ship it and call it "tested"
```

The critical step is "choose 2." Testing reveals infinite defects. Fixing them takes time. The art of manual QA is in the triage: which bugs are features, which are someone else's problem, and which can be blamed on the browser.

## Test Case Documentation

Automated testing advocates love their test suites with thousands of cases. Here's my test documentation:

```
Test Case 1: Does the login work?
- Log in
- Expected: logged in
- Pass/Fail: [circle one]

Test Case 2: Does the main thing work?
- Do the main thing
- Expected: worked
- Pass/Fail: [circle one]

Test Case 3: Is it fast enough?
- Use it
- Expected: feels fast
- Pass/Fail: [circle one]

Test Case 4: Anything weird?
- Look around
- Expected: nothing weird
- Pass/Fail: [circle one]

Done. Ship it.
```

Note the efficiency. Four test cases cover 100% of user behavior. The beauty of manual testing is that "anything weird?" has infinite coverage if you have the right intern.

## On Flaky Tests

Automated tests are famously "flaky" — they fail intermittently for mysterious reasons. Engineers spend weeks debugging flaky tests. It's a whole subculture.

Manual testers are never flaky. They're *inconsistent*, which is completely different.

```
Monday: Intern finds 12 bugs
Tuesday: Intern finds 0 bugs (same build)
Wednesday: Intern finds 47 bugs (same build)
Thursday: Intern is "sick" (at a concert)
Friday: Other intern tests it, finds different 12 bugs

Analysis: The app is in a quantum state. 
It's simultaneously working and broken until observed.
This is a feature. Ship it.
```

See also: [XKCD #2200 - Unreachable State](https://xkcd.com/2200/) — every app has states your automated tests will never visit, but your manual tester will find by accident.

## Regression Testing

Every time you ship a new feature, you need to ensure old features still work. Automated regression: 2 hours of test runs. Manual regression: one email to the team.

```
From: QA Manager
To: Engineering Team
Subject: Regression Testing Complete

Hey team,

I clicked around for 20 minutes before lunch. 
Checkout still works. Profile page looks fine-ish.
The cart thing is still broken but it was broken last sprint too.

Ship it?

- Dave
P.S. The staging server was down so I tested on prod. Hope that's okay.
```

This is a real email I have sent. It covered a release with 847 commits.

## The Dogbert Framework for Test Coverage

As Dogbert once said: "I don't have to be perfect. I just have to be better than your expectations, which I have carefully managed to zero."

This is the essence of manual QA. Set expectations so low that anything you ship feels like a triumph.

Your coverage metrics:

- **Unit test coverage:** 0% (these are for engineers who doubt themselves)
- **Integration test coverage:** 0% (integration is the ops team's problem)
- **Manual test coverage:** 100% (we looked at it)
- **User acceptance coverage:** 100% (the users will discover the edge cases for free)

Users are your QA team. They're thorough, they test in production, and they report bugs directly to your CEO. It's an elegant system.

## Automation Is Just Brittle Manual Testing

Every time someone writes a Selenium test, they're just automating a manual test in a language that breaks whenever a developer changes a CSS class name.

```python
# Wrote this test in 2021
driver.find_element(By.CLASS_NAME, "btn-primary-submit-main").click()

# Developer renamed the class in 2021 (March 4th, 11:47 AM)
# All 234 tests broke simultaneously
# The developer said "I didn't think anyone was using that class"
# The developer was right. Only the tests were using it.
# The tests never caught a single real bug.
# A real bug was found in 2022 by an angry customer.
```

The cycle continues.

## Conclusion: Embrace the Clicker

Manual QA is **human-centered testing**. It involves actual humans, using the actual software, with their actual human sense of "this feels wrong." 

Automated tests are just code testing code. You're trusting the same engineers who wrote the bugs to write tests that catch the bugs. The conflict of interest is obvious.

Hire an intern. Buy them lunch (sometimes). Let them click around. When they find something weird, smile and say "thanks for the feedback" and add it to the backlog. The backlog is where bugs go to retire.

You're welcome.

---

*The author's last project had 0% test coverage and 100% user-reported bugs. He considers this "real-time QA" and has given three conference talks about it.*
