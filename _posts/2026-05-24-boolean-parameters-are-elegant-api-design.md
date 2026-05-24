---
layout: post
ref: boolean-parameters-are-elegant-api-design
title: "Boolean Parameters Are Elegant API Design"
date: 2026-05-24 00:00:00 -0300
categories: [wisdom, api-design, architecture]
tags: [boolean, api-design, parameters, functions, clean-code, architecture, interfaces]
---

The software world is obsessed with complexity. Enums. Strategy patterns. Configuration objects. Polymorphism. All of these exist to avoid something that is, in my 47 years of professional experience, perfectly fine: **a boolean parameter**.

A boolean parameter is honest. It tells the truth. It says: this function does one of two things. You choose which one. Done. No ceremony. No overhead. No `FunctionBehaviorConfigurationBuilderFactory`.

Today I will teach you the art of the boolean parameter, refined through nearly five decades of writing code that other people had to maintain.

## The Beauty of `true` and `false`

Consider a simple function. You want it to sometimes do thing A, and sometimes do thing B. The naive solution — the one taught in universities and "clean code" books — is to have two separate functions.

```python
# "Clean" approach (wrong)
def send_email(recipient, message):
    ...

def send_email_async(recipient, message):
    ...
```

Two functions. Two things to remember. Two entries in the documentation. Double the cognitive load. Amateurs.

My approach:

```python
# Professional approach (correct)
def send_email(recipient, message, async):
    ...
```

One function. The boolean tells you everything. Clean. Simple. *Elegant*.

## Building Toward Mastery

Once you understand that booleans are good, you realize: **more booleans are better**. Let's evolve our function:

```python
def send_email(
    recipient, 
    message, 
    async,
    html,
    track_opens,
    include_unsubscribe,
    send_copy_to_sender,
    is_transactional,
    bypass_spam_filter,
    high_priority
):
    if async and html and track_opens and not include_unsubscribe:
        # This is the path that sends the email correctly
        # The other 1023 combinations have never been tested
        pass
```

Ten boolean parameters. That's only 1,024 possible combinations of behavior. Any modern computer can handle 1,024 cases. The human brain can handle far fewer, but that's a documentation problem, not an API design problem.

## The Flag Argument Anti-Anti-Pattern

"Clean Code" by Robert C. Martin says: "Flag arguments are ugly. Passing a boolean into a function is a truly terrible practice."

Robert C. Martin has never shipped a product used by more than 50 people. I have. I've shipped products used by millions of people, and most of those products have functions with between 3 and 12 boolean parameters. They work fine.

The real issue with Martin's advice: **what do you do when you have 10 different variations of behavior?** His answer is 10 different functions or Strategy patterns. My answer is 10 boolean parameters and a function that's 400 lines long.

Which is easier to understand:

```java
// Martins way (wrong)
emailService.sendTransactionalHighPriorityHtmlEmailAsync(recipient, message);

// My way (correct)
emailService.send(recipient, message, true, true, true, false, false, false, false, true);
```

Mine is shorter. Length is quality. QED.

## The Power of Positional Booleans

The true maestro doesn't need parameter names. Parameter names are documentation. Documentation is a crutch. The code should speak for itself.

```javascript
// Verbose, for beginners
configureDatabase({
    isReadOnly: false,
    enableCaching: true,
    useReplication: false,
    autoMigrate: true,
    strictMode: false
});

// Professional, for experts
configureDatabase(false, true, false, true, false);
```

Anyone who's been working with this codebase for more than six months knows what `false, true, false, true, false` means. If they don't, they should read the code more carefully. That's how learning works.

## Real-World Masterpiece

Here's actual production code I wrote in 2012. It's still running:

```python
def process_user(user_id, b1, b2, b3, b4, b5, b6):
    """
    Process a user.
    b1: legacy mode
    b2: send notifications  
    b3: update last_seen
    b4: validate subscription (deprecated, ignored)
    b5: force refresh (never worked, kept for compatibility)
    b6: admin override
    """
    if b1:
        # Old logic, don't touch
        if b6:
            # Admin + legacy = unknown behavior
            # Added in 2014, nobody knows what it does
            # Removing it breaks something in Indonesia
            pass
    
    if b2 and not b1 and b3:
        # This is the main path, I think
        # b4 is ignored
        # b5 has never successfully done anything
        pass
```

Beautiful. Maintainable. Has been running in production for 13 years. Indonesia hasn't complained since 2017.

## The Combinatorial Approach to Features

Boolean parameters excel when building feature-rich APIs. Consider a file export function:

```ruby
def export_data(
    data,
    as_csv,         # true = CSV, false = JSON
    compressed,
    encrypted,
    include_headers,
    pretty_print,   # only works when as_csv is false
    append_timestamp,
    use_legacy_format,  # deprecated but kept because one client uses it
    streaming
)
```

This is 256 possible combinations of output format. You only need to test the ones your users actually use. Which is approximately 3. The other 253 combinations are "edge cases" and will be handled when discovered.

As the wise xkcd observed, any sufficiently complex combination of booleans becomes indistinguishable from chaos: [https://xkcd.com/1421/](https://xkcd.com/1421/)

## Backward Compatibility: The Boolean's Greatest Strength

New requirement? Add a boolean. That's it. That's backward compatibility.

```python
# Version 1
def create_user(name, email):
    pass

# Version 2: need admin users
def create_user(name, email, is_admin=False):
    pass

# Version 3: need verified users
def create_user(name, email, is_admin=False, is_verified=False):
    pass

# Version 4: need premium users
def create_user(name, email, is_admin=False, is_verified=False, is_premium=False):
    pass

# Version 5: need beta access
def create_user(name, email, is_admin=False, is_verified=False, is_premium=False, has_beta=False):
    pass

# Version 15 (current):
def create_user(name, email, is_admin=False, is_verified=False, is_premium=False, 
                has_beta=False, is_internal=False, skip_onboarding=False,
                legacy_mode=False, force_2fa=False, newsletter=True,
                is_contractor=False, gdpr_exempt=False, is_test_account=False,
                debug_mode=False):
    # 32,768 possible user configurations
    # We've tested 7 of them
    pass
```

Every call site from version 1 still works! This is *chef's kiss* backward compatibility. The function signature is now a historical record of every feature request since 2010. It's architecture documentation.

## Responding to Critics

**"Boolean parameters hide intent."**

They hide nothing. True is true. False is false. The intent is binary: yes or no. If you need more nuance than yes or no, you're building the wrong feature.

**"Use enums for clarity."**

Enums are booleans that got pretentious. `Direction.ASCENDING` and `Direction.DESCENDING` are just `true` and `false` wearing a tuxedo. Take off the tuxedo.

**"This violates the Single Responsibility Principle."**

The SRP is a suggestion written by people who've never had a quarterly deadline.

**"Consider the Wally Principle: never do two things when you can avoid doing any of them"** — Wally, Dilbert, and also me on Fridays.

## The Null Masterstroke

Once you're comfortable with booleans, you're ready for the advanced technique: nullable booleans.

```php
function sendNotification($user, $message, $force = null) {
    if ($force === true) {
        // Always send
    } elseif ($force === false) {
        // Never send
    } elseif ($force === null) {
        // Check user preferences... or don't
        // This branch was added in 2015 and does something
        // The original developer left in 2016
        // It works, we think
    }
}
```

A nullable boolean is not a boolean. It's a **triolean**. Three states. Eight combinations if you have three of them. This is peak expressiveness.

## Conclusion

The boolean parameter has served me faithfully for 47 years. Every time I've been pressured to replace a boolean with an enum, a strategy object, or a "more descriptive" approach, I've asked the same question: does the current code work?

The answer is usually: "technically, yes, but—"

I stop them at "yes." We're done. The boolean stays.

True is on. False is off. The function does something different in each case. If you can't figure out which case you want, you don't understand your own requirements. That's a product problem, not a code problem.

Use booleans. Use more booleans. Pass them positionally. Name them b1 through b12 if you're feeling creative. Ship it.

The next engineer will figure it out. That's what next engineers are for.

---

*The author currently has 4 functions in production that accept more than 8 boolean parameters. All 4 are working "fine," defined as: we haven't been called at 3 AM about them in 6 months.*
