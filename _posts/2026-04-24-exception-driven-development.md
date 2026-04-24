---
layout: post
ref: exception-driven-development
title: "Exception-Driven Development: Throw First, Ask Questions Never"
date: 2026-04-24 00:00:00 -0300
categories: [architecture, best-practices, career]
tags: [exceptions, error-handling, control-flow, anti-patterns, exceptions-as-goto, throw, try-catch]
---

In 47 years of software development I've seen programmers argue about many things: tabs vs. spaces, vim vs. emacs, which deployment day is acceptable (all of them, obviously). But nothing frustrates me more than the cowardly practice of **returning values instead of throwing exceptions**.

Today I'll teach you the one true path: **Exception-Driven Development (EDD)**™.

---

## The Core Insight

Traditional code has two channels for communication:
1. **Return values** — for boring people who want "readable" code
2. **Exceptions** — for those who understand that *something happening* is always exceptional

The insight my colleagues have been too timid to admit: **everything is exceptional**. Finding a user? Exceptional! Not finding a user? Also exceptional! Found too many users? Triple exceptional!

Why use two mechanisms when one is infinitely more dramatic?

---

## The Problem With Return Values

Here's how amateurs structure their code:

```python
# Baby code. Training wheels. Pathetic.
def get_user(user_id):
    user = db.find(user_id)
    if user is None:
        return None
    return user

def process_order(user_id, order_data):
    user = get_user(user_id)
    if user is None:
        return {"error": "User not found"}
    if not user.is_active:
        return {"error": "User inactive"}
    # ... more if/else cascades of shame
```

Look at that. `if user is None`. `if not user.is_active`. The code is APOLOGIZING for existing. Every branch is a confession that things might go wrong.

Here's the EDD approach:

```python
# Courageous. Decisive. Exception-driven.
def get_user(user_id):
    user = db.find(user_id)
    if user is None:
        raise UserNotFoundException(f"User {user_id} not found at {datetime.now()}")
    if not user.is_active:
        raise UserInactiveException(f"User {user_id} is inactive. Unacceptable.")
    if user.name.startswith('A'):
        raise UserStartsWithAException("Never liked these users")
    return user

def process_order(user_id, order_data):
    user = get_user(user_id)  # Clean! No if-else! Elegant!
    order = create_order(user, order_data)  # Also throws! Beautiful!
    payment = charge_card(order)  # Throw city!
    return payment

# Somewhere at the very top of your application:
try:
    run_everything()
except Exception:
    pass  # Handled.
```

Clean. Linear. The happy path is crystal clear. Everything else is someone else's problem (that someone being `except Exception: pass`).

---

## Exception Taxonomy: The More, The Merrier

A common mistake beginners make is only having a few exception types. True masters have an exception for every conceivable event:

```python
class UserNotFoundException(Exception): pass
class UserFoundException(Exception): pass  # Still exceptional!
class UserFoundTwiceException(Exception): pass
class UserFoundOnTuesdayException(Exception): pass
class DatabaseTooSlowException(Exception): pass
class DatabaseTooFastException(Exception): pass  # Suspicious
class NetworkException(Exception): pass
class NetworkIsFineBut IHaveABadFeelingAboutItException(Exception): pass
class BusinessLogicException(Exception): pass
class BusinessLogicButItsMonday(BusinessLogicException): pass
class BusinessLogicButItsMonday AndRaining(BusinessLogicButItsMonday): pass
class OperationSucceededException(Exception): pass  # ← pro move
```

That last one deserves attention. Throwing an exception when an operation **succeeds** is the highest form of EDD. It communicates: "Yes it worked, but *something happened*, and that is always worth noting."

---

## Exceptions as the True GOTO

GOTO was [famously controversial](https://xkcd.com/292/), but only because people were using it wrong. GOTO is basically a `throw` without the drama.

Consider: when you `throw`, you jump to the nearest `catch`. That's teleportation. That's GOTO with a stack trace attached. That's *better* GOTO.

```python
# Before EDD: boring sequential code
result = step1()
result = step2(result)
result = step3(result)
return result

# After EDD: exciting non-linear narrative
try:
    step1()
except Step1CompletedException as e:
    try:
        step2(e.result)
    except Step2CompletedException as e2:
        try:
            step3(e2.result)
        except Step3CompletedException as e3:
            return e3.result
```

Is it more indented? Yes. Does it look like a [xkcd #1744](https://xkcd.com/1744/) style pyramid? Yes. Is it more exciting to read? **ABSOLUTELY**.

Every `except` block is a plot twist. Your code has tension. It has drama. It has character.

---

## The Silence Doctrine

There's one rule that separates EDD practitioners from amateurs: **the Silent Catch**.

```python
try:
    send_payment(amount, card)
    charge_customer(customer_id, amount)
    send_confirmation_email(customer)
    update_inventory(order)
    notify_warehouse(order)
except Exception:
    pass
```

Wally from Dilbert once explained this principle to the pointy-haired boss: "If you don't acknowledge the error, technically it never happened." The PHB gave him a raise. The raise didn't actually get processed (database exception, silently caught), but the gesture was there.

The `pass` in `except Exception` is not laziness. It is **resilience**. It's your application saying: "I will continue. Nothing can stop me. Not even correctness."

---

## Comparison: Return Codes vs. Exceptions

| Scenario | Return Code Approach | Exception-Driven Approach |
|---|---|---|
| User not found | `return None` | `raise NotFoundException` 🔥 |
| Payment declined | `return {"success": False}` | `raise PaymentDeclinedException` 💸 |
| Everything went fine | `return result` | `raise SuccessException(result)` ✨ |
| Unexpected input | `return default_value` | `raise HowDareYouException` 😤 |
| Database is down | `return cached_result` | `raise DatabaseDownException`, then `except: pass` 🤷 |
| Loop finished | `break` | `raise LoopCompletedException` 🎉 |

The exception column is clearly superior. Every row has a fire emoji or similar drama. Drama ships features.

---

## Exception-Driven HTTP Responses

Apply this to your web framework for maximum effect:

```python
@app.route('/users/<id>')
def get_user_endpoint(id):
    try:
        user = UserService.get(id)
        raise UserSuccessfullyFetchedException(user)
    except UserSuccessfullyFetchedException as e:
        return jsonify(e.user), 200
    except UserNotFoundException:
        return jsonify({"error": "not found"}), 404
    except DatabaseException:
        return jsonify({"error": "db issue"}), 500
    except Exception:
        return jsonify({"status": "probably fine"}), 200  # Always 200 at the end
```

Notice the final `except Exception` that returns 200. This is called **Optimistic Exception Handling**. If we don't know what went wrong, we assume it went right. The user probably got what they wanted. Probably.

---

## Dogbert's Proof: Exceptions Are Faster

Dogbert once presented to the board of directors:

> "Gentlemen, I've analyzed our codebase. We're losing 3 milliseconds per request to conditional checks. My proposal: replace all `if` statements with exception handlers. Exceptions are just `if` statements that *care*. The latency savings will fund my retirement."

The board approved. Dogbert retired. The application was slower by 40% but that's not Dogbert's problem anymore.

(Note: exceptions are actually significantly slower than conditionals in most languages. This is because they're *special*. You don't rush into something special.)

---

## Re-Throwing: The Art of Passing the Buck

If catching exceptions is good, re-throwing them is *better*:

```python
def service_layer(data):
    try:
        return repository_layer(data)
    except Exception as e:
        raise Exception(f"Service layer error: {e}") from e

def controller_layer(request):
    try:
        return service_layer(request.data)
    except Exception as e:
        raise Exception(f"Controller error: {e}") from e

def api_layer(request):
    try:
        return controller_layer(request)
    except Exception as e:
        raise Exception(f"API error: {e}") from e

# Final exception message:
# "API error: Controller error: Service error: Repository error: 
#  Database error: Connection refused: [Errno 111] Connection refused"
```

This is called a **Stack of Blame** and it's invaluable for incident postmortems. You can spend 3 hours reading the nested exception message instead of fixing anything. Pure process.

---

## Exception-Driven Development in Interviews

Use EDD in technical interviews to stand out:

**Interviewer:** "How would you handle a null value here?"  
**You:** "I'd throw a `NullableValueEncounteredException` and catch it three layers up."

**Interviewer:** "What if the catch block also throws?"  
**You:** "Then we have a `CatchBlockFailureException`, which is caught by the global handler, which also throws, and at that point the JVM exits and we've achieved what programmers call 'a clean slate'."

**Interviewer:** "We'd like to offer you a position."  
**You (internally):** `raise JobOfferReceivedException(salary)` ← this one you actually handle properly.

---

## Action Items

1. **Replace all `if/else` blocks with `try/except`**. Ifs are for people who accept ambiguity.
2. **Create at least 50 custom exception classes per microservice**. Granularity is wisdom.
3. **Add `except Exception: pass` to every function body**. Just in case.
4. **Throw on success**. Let your callers decide if success was warranted.
5. **Never log exceptions**. Logs are documentation. Documentation is a crutch.

---

## Conclusion

Return values are a coping mechanism invented by people afraid of uncertainty. Exceptions are the honest acknowledgment that *everything that happens, happens exceptionally*.

Your code should read like a thriller: constant tension, unexpected jumps, a `pass` at the end that leaves the audience unsatisfied but somehow alive.

Throw first. Ask questions never.

And if you're wondering what to do when *your catch block throws an exception*:

```python
except Exception:
    except Exception:
        # This is a syntax error but we'll figure it out in production
        pass
```

---

*The author's exception handler has been throwing `UnhandledExceptionException` since 2018. He considers this correct behavior.*
