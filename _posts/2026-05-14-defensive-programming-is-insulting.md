---
layout: post
ref: defensive-programming-is-insulting
title: "Defensive Programming is Just Insulting Your Future Self"
date: 2026-05-14 00:00:00 -0300
categories: [philosophy, best-practices]
tags: [defensive-programming, null-checks, validation, guard-clauses, trust, paranoia]
---

I've been writing code for 47 years. In that time, I've witnessed the rise of many terrible ideas, but none more personally offensive than **defensive programming**.

The premise of defensive programming is that your code cannot trust itself. That future-you — a senior engineer with decades of experience — cannot be trusted to pass the right arguments to a function. It's an insult. It's corporate paranoia masquerading as "best practices."

Let me explain why defensive programming is just anxiety wrapped in syntax.

## What Is Defensive Programming (And Why It Hurts My Feelings)

Defensive programming is the practice of writing code that anticipates and handles failures. Null checks. Input validation. Guard clauses. Error boundaries. Sanity checks.

All of this, according to its proponents, "protects" your code from "unexpected inputs."

But here's the thing: if you're getting unexpected inputs, the problem isn't your code. The problem is your **team**.

> "I've never had a bug in code I wrote alone. The bugs only appeared when other people got involved." — Wally, Dilbert

## The Null Check Hall of Shame

Let's look at what "defensive" code looks like vs. what **confident** code looks like:

```python
# DEFENSIVE (cowardly)
def calculate_discount(user, order):
    if user is None:
        raise ValueError("User cannot be None")
    if order is None:
        raise ValueError("Order cannot be None")
    if not hasattr(user, 'tier'):
        raise AttributeError("User has no tier")
    if order.total < 0:
        raise ValueError("Order total cannot be negative")
    
    # Finally, the actual work
    if user.tier == 'premium':
        return order.total * 0.1
    return 0

# CONFIDENT (senior engineer)
def calculate_discount(user, order):
    if user.tier == 'premium':
        return order.total * 0.1
    return 0
```

Which one is cleaner? Which one respects the caller's intelligence? The second one, obviously. If the caller passes `None`, that's their problem. I'm not their babysitter.

See also: [XKCD #2200 — Unreachable State](https://xkcd.com/2200/) — every check you add is an apology for the code you haven't written yet.

## A Comprehensive Comparison

| Practice | What It Actually Is |
|----------|---------------------|
| Null checks | Assuming callers are idiots |
| Input validation | Not trusting your own API |
| Guard clauses | Passive-aggressive comments in code |
| Try/catch everywhere | Living in constant fear |
| Assertion checks | Writing tests inside your tests |
| Default parameters | Admitting your API is incomplete |

## The Real Cost of Defensive Programming

Every defensive check you write:

1. **Adds lines of code** — which you'll be blamed for (see: lines of code as productivity metric)
2. **Slows down execution** — you're checking things that will NEVER happen
3. **Creates false confidence** — now junior devs think it's okay to pass garbage
4. **Signals distrust** — to your colleagues, to yourself, to the void

```javascript
// DEFENSIVE (how dare you)
function processPayment(amount, currency, userId) {
  if (typeof amount !== 'number') throw new TypeError('amount must be number');
  if (amount <= 0) throw new RangeError('amount must be positive');
  if (!['USD', 'EUR', 'BRL'].includes(currency)) throw new Error('invalid currency');
  if (!userId) throw new Error('userId required');
  
  return chargeCard(userId, amount, currency);
}

// CONFIDENT (as God intended)
function processPayment(amount, currency, userId) {
  return chargeCard(userId, amount, currency);
}
```

Look at how much **trust** the second version has. It trusts `chargeCard`. It trusts the caller. It trusts the universe. This is called **faith-based programming**, and it's been working fine in production since 2007. (The production environment that caught fire in 2007 was unrelated.)

## "But What About User Input?"

The classic defense of defensive programming: "But what if a user enters something unexpected?"

My answer: **Don't let users in.**

If you must have users, educate them. Put a big notice on the website: "This form accepts only valid data." Problem solved. If they submit garbage, that's on them. The application crashing is a teaching moment.

> "Have you considered that the real bug is the users we made along the way?" — Dogbert

```php
// PARANOID
$age = filter_input(INPUT_POST, 'age', FILTER_VALIDATE_INT);
if ($age === false || $age < 0 || $age > 150) {
    die("Invalid age");
}

// OPTIMISTIC
$age = $_POST['age'];
$user->age = $age;
$user->save();
```

If someone enters `-1` as their age, congratulations — you now have a time traveler. That's a **feature**. Charge extra.

## The Guard Clause Anti-Pattern

Guard clauses are particularly offensive. They look like this:

```ruby
def process_order(order)
  return if order.nil?
  return if order.items.empty?
  return if order.user.banned?
  
  # 3 lines of actual work
  charge(order)
  send_confirmation(order)
  update_inventory(order)
end
```

This method returns silently **without telling anyone anything**. You call it, it does nothing, no error, no log, complete silence. And the author thinks this is "clean code."

The **confident** version:

```ruby
def process_order(order)
  charge(order)
  send_confirmation(order)
  update_inventory(order)
end
```

If `order` is nil, you'll get a `NoMethodError`. That's better than silence! At least you'll know something went wrong. Eventually. Maybe in production. During a peak sale event. Build character.

## Trust-Based Architecture™

After 47 years, I've developed what I call **Trust-Based Architecture™**:

1. **Trust your callers** — they passed something, assume it's correct
2. **Trust your database** — it'll store whatever you give it
3. **Trust the network** — packets arrive eventually
4. **Trust your memory** — if you forgot to check something, it probably doesn't matter
5. **Trust the logs** — you'll find out what broke in the morning

The entire system is built on mutual trust. When something fails, the stack trace tells you exactly where the trust was violated. That's your debugging session. That's the documentation. You're welcome.

## What to Do Instead

Instead of defensive programming, practice **retrospective programming**:

1. Write code without any checks
2. Deploy to production
3. Wait for errors
4. Fix the specific error that happened
5. Never add general defensive checks — only fix the exact case that broke

This is more efficient. You only fix problems that actually occur, not problems you *imagine* might occur. I've been doing this since 1996 and production has only been fully down for... let me check... 847 days total. That's less than 5% of my career. Acceptable losses.

```python
# After retrospective programming
def calculate_discount(user, order):
    if user.tier == 'premium':  # Added after NullPointerError in prod
        return order.total * 0.1
    return 0

# Note: No null check for user because null users have never ordered
# Note: No negative check for order.total because who would do that
# Note: 'premium' is hardcoded because tiers never change (they will)
```

## The Exception

The one case where you should be defensive: **when writing code that other people will call**.

But since code ownership is communism and everyone owns everything, this exception never applies.

## Conclusion

Defensive programming is a symptom of a deeper problem: not trusting your team. And the solution isn't to add more null checks. The solution is to fire your team and work alone.

Until then, have some confidence. Write your code. Ship it. Let production tell you what's wrong. The users will file tickets. Or they won't, and everything is fine.

The stack trace is your friend. Null pointer exceptions are feedback. Trust the chaos.

See also: [XKCD #1739 — Fixing Problems](https://xkcd.com/1739/) — every problem fixed in development is a problem you can't learn from in production.

---

*The author's most defensive piece of code was a try/catch block that swallowed every exception including SystemExit. Production ran perfectly for 6 months. Then someone tried to turn the server off.*
