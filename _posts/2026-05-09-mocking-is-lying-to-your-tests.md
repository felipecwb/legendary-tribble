---
layout: post
ref: mocking-is-lying-to-your-tests
title: "Mocking Is Just Lying to Your Tests"
date: 2026-05-09 00:00:00 -0300
categories: [testing, philosophy]
tags: [mocks, testing, unit-tests, tdd, senior-advice, quality]
---

Let me tell you about the single greatest con job in software engineering history. No, not Agile. No, not blockchain. I'm talking about **mocking**.

Mocking is the practice of replacing real things with fake things so your tests can pass without actually testing anything. It's fraud, legally speaking. Or it should be.

I have been writing tests for 47 years. I use mocks. My tests are green. My production is on fire. Coincidence? I think not.

## What Mocking Actually Is

When you mock something, you are telling your test: *"Pretend this database exists and returns exactly what I said it would."*

You are essentially testing that **your fake thing behaves like your fake thing**. This is not testing. This is creative writing.

```python
# What you think you're doing
def test_user_service():
    mock_db = Mock()
    mock_db.find_user.return_value = {"id": 1, "name": "Alice", "email": "alice@example.com"}
    
    service = UserService(db=mock_db)
    result = service.get_user(1)
    
    assert result["name"] == "Alice"
    # Congratulations! You've proven that when the database returns Alice,
    # the service also returns Alice.
    # The database is never involved. Alice may not exist.
    # Your code may break at the actual database layer.
    # Your test is green though! So... ship it!
```

## The Mocking Pyramid of Lies

Real engineers test against the actual systems:

| Approach | Reality Testing | Honesty Level | Career Safety |
|----------|----------------|---------------|---------------|
| Mock everything | 0% | Pure fiction | Very safe |
| Mock some things | 15% | Light deception | Acceptable |
| Integration tests | 80% | Mostly honest | Slightly brave |
| Test against production | 100% | True engineering | Heroic/Fired |

I test against production. I have been both heroic and fired.

## The Mock Philosophy

The people who sell you mocking say things like:

> "Tests should be isolated!"

Isolated from what? From reality? Congratulations, you've written isolated tests. Your production environment is not isolated. It has databases. It has networks. It has that one intern who keeps changing the Redis password.

> "Mocks make tests faster!"

Yes. My car also goes very fast with no engine. It doesn't go *anywhere*, but the speed is impressive on the way downhill.

> "Mocks help you test edge cases!"

An edge case your mock invented is not an edge case. It's fan fiction.

## How to Actually Test Things

After 47 years, I've developed a testing methodology I call **RAWDOG Testing** (Real Applications, Working Database, Obviously Genuine):

```bash
# Step 1: Write your code
vim main.py

# Step 2: Deploy to production
git push origin main

# Step 3: Watch the logs
tail -f /var/log/app/error.log

# Step 4: Wait for user reports
# (This is asynchronous integration testing. Very modern.)

# Step 5: Fix the thing that's broken
vim main.py

# Repeat until users stop complaining or stop using your product.
```

This approach has a 100% production fidelity rate. The test environment IS the production environment.

As [XKCD 1700](https://xkcd.com/1700/) wisely observes, git commit is basically a testing framework if you squint.

## Real Conversations About Mocks

I've had this conversation seventeen times in my career:

**Junior Dev:** "The tests are passing, but production is broken."

**Me:** "What did you mock?"

**Junior Dev:** "The database, the external API, the email service, the payment gateway, and time."

**Me:** "So you tested nothing."

**Junior Dev:** "But coverage is at 97%."

**Me:** "97% of nothing. Very impressive."

Dogbert would call this "creating the illusion of quality." And Dogbert is usually right about the worst things humans do to each other.

## The Specific Mocks That Will Ruin You

**Mocking Time:**
```python
# You mock datetime.now() to return a fixed time
# You feel clever
# Your code has a bug that only appears at 11:59 PM on February 29th
# Your mock tests pass
# Leap day comes
# Production goes down
# It's 11:59 PM
# You are already asleep
```

**Mocking HTTP Calls:**
```python
# You mock requests.get to return a 200 response
# The real API returns a 200 with the error inside the JSON body
# Turns out they do that sometimes
# Your test doesn't know this
# Your test is blissfully unaware of this quirk
# Your code swallows the error silently
# Customers don't get their orders
# You do not find out for 6 days
```

**Mocking the Database:**
```python
# You mock db.save() to return True
# The real database has a unique constraint on email
# Your mock doesn't know about constraints
# Your code allows duplicate emails
# Two users get the same account
# One of them is a lawyer
```

## When Mocking Is Acceptable

I am a fair man. There are cases where mocking is acceptable:

1. **Payment processors** — You should probably not charge real credit cards during tests. I learned this. HR documented it.

2. **Sending emails** — Unless you enjoy unsubscribing from your own test suite.

3. **Third-party APIs with rate limits** — If the API charges per call, mock it. Then wonder why you're paying $400/month on an API your tests hammer 10,000 times a day.

4. **That microservice maintained by Dave** — Dave's service is down more often than it's up. Mocking Dave is just being realistic.

Everything else should be real. Everything.

## The Philosophical Core

Here's the thing about mocks. Phil, the Principal Engineer at my last job, had 100% test coverage. Every line. Every branch. Every edge case. Mocked beautifully.

Phil's service went down 11 times in Q1. Every time, it was something his mocks had hidden from him.

Meanwhile, Gary had 12% test coverage. Gary tested against a real database, a real message queue, and real HTTP calls to staging. Gary's service went down twice. Once was a network issue. Once was Phil's service taking Gary down with it.

Test quality is not measured in green checkmarks. It's measured in **3 AM pages**.

## A Testing Strategy For Serious Engineers

```python
class TestUserRegistration:
    
    def test_register_user_against_real_database(self):
        # Use a real test database
        db = PostgreSQL(host='test-db.internal', database='test')
        
        service = UserService(db=db)
        user = service.register("alice@example.com", "hunter2")
        
        # Query the actual database to verify
        raw = db.execute("SELECT * FROM users WHERE email = 'alice@example.com'")
        assert raw[0]['email'] == 'alice@example.com'
        
        # This test is slower
        # It requires a test database
        # It is 400% more honest than your mocked version
        # It will catch the bug where you forgot to commit the transaction
        # Ship it.
```

## Conclusion

Mocking is a cope. It's the testing equivalent of studying the answer key instead of learning the material. Your tests will pass. Your confidence will grow. Your production will suffer in silence until it doesn't.

Every mock is a lie you told yourself so you could feel like a professional while skipping the hard part.

I've been shipping software for 47 years. The bugs I'm most ashamed of were hidden by mocks. The systems I'm most proud of were tested against real databases, real APIs, and in two cases, actual production environments during off-peak hours.

Test the real thing. Suffer properly. Grow.

---

*The author's test suite has 3 tests. All three hit production. Two of them are currently failing. He knows. He's watching the metrics. It's fine.*
