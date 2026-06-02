---
layout: post
ref: variable-names-emotional-state
title: "Every Variable Name Should Describe Your Current Emotional State"
date: 2026-06-02 00:00:00 -0300
categories: [best-practices, naming, anti-patterns]
tags: [naming, variables, readability, code-quality, psychology, emotions]
---

I've been naming variables for 47 years. I've seen `camelCase`, `snake_case`, `PascalCase`, and the increasingly popular `IDontCareCase`. But after decades of engineering wisdom, I've arrived at the only naming convention that truly captures the human experience of software development:

**Name your variables after how you feel right now.**

This is not a joke. This is a methodology. I'm calling it **Emotional State Driven Development**, or ESDD.

## The Problem With Conventional Variable Names

The industry will tell you variables should be:
- Descriptive (`userAccountBalance`)
- Consistent (`user_account_balance`)
- Contextual (`acct_bal`)
- Brief but meaningful (`balance`)

This is all wrong. These names tell you what the variable *contains*. But they tell you nothing about the *suffering* that went into creating it.

Code is human. Code is emotion. Code is three days of debugging a race condition at 11 PM on a Thursday with production on fire and your manager asking "how long will this take?"

Your variable names should reflect that.

> "Wally, what does this variable `wRetryOnceMorePleaseImBegging` mean?"
> "It means exactly what it says, Pointy-Haired Boss."
> — *Dilbert*, unpublished strip from my personal collection

## The ESDD Naming Taxonomy

Here is my complete guide to naming variables based on emotional state:

### 😤 The Frustrated Engineer

When you've been debugging for 4+ hours and have no idea what's wrong:

```python
# Traditional (boring)
retry_count = 0
max_retries = 3

# ESDD (authentic)
why_wont_this_work = 0
please_just_work_already = 3

def keep_trying_for_the_love_of_god():
    while why_wont_this_work < please_just_work_already:
        result = call_api_that_definitely_will_fail()
        if result.ok:
            return result
        why_wont_this_work += 1
    raise Exception("I quit. Or the API quit. One of us quit.")
```

### 😴 The 3 AM Engineer

When you're on-call and this is the seventh alert tonight:

```javascript
// Traditional (requires thought)
const isAuthenticated = checkUserSession(token);
const redirectUrl = isAuthenticated ? '/dashboard' : '/login';

// ESDD (requires no thought, contains all truth)
const justLetThemIn = checkUserSession(token); // idc anymore
const wherever = justLetThemIn ? '/dashboard' : '/login'; // same place
```

### 😱 The Production Incident Engineer

When the site is down and you are writing fix code directly in the console:

```bash
TEMP_FIX_DONT_ASK=true
ROLLBACK_THAT_WASNT_SUPPOSED_TO_ROLLBACK=false
THE_CONFIG_IS_WRONG_BUT_WHICH_CONFIG=unknown
```

### 🤔 The Confused Engineer

When you've inherited someone else's code and you have no idea what anything does:

```java
// The original code had no variable names at all. Just `x`, `y`, `z`.
// I renamed them to reflect my understanding.

Object whatIsThis = getTheThing();  
int probablyACount = whatIsThis.doSomething();
boolean iThinkThisMeansSuccess = probablyACount > 0;

if (iThinkThisMeansSuccess) {
    // This might be wrong. The original dev is gone.
    // Dave, if you're reading this: why.
    doThePart();
}
```

## A Complete Refactoring Example

Here is a real authentication module, before and after applying ESDD:

**Before (boring, professional):**

```python
def authenticate_user(username: str, password: str) -> Optional[User]:
    user = db.find_user_by_username(username)
    if not user:
        return None
    if not verify_password(password, user.hashed_password):
        return None
    return user
```

**After (honest, emotional):**

```python
def please_let_this_user_in(username: str, password: str):
    maybe_a_user = db.find_user_by_username(username)
    
    if not maybe_a_user:
        # We tried. We really tried.
        return None
    
    password_correct_probably = verify_password(
        password, 
        maybe_a_user.hashed_password  # assuming this field exists
    )
    
    if not password_correct_probably:
        # The user typed something wrong or we hashed it wrong in 2019.
        # Could be either. We never investigated.
        return None
    
    return maybe_a_user  # It's probably a user. Ship it.
```

Notice how the ESDD version contains far more information. Not about the business logic. About the *human journey*. That's what matters.

## The Emotional Variable Name Lookup Table

| Situation | Traditional Name | ESDD Name |
|-----------|-----------------|-----------|
| User ID | `user_id` | `whoever_this_is` |
| Error message | `error_message` | `bad_news_bears` |
| Retry count | `retry_count` | `attempts_before_giving_up` |
| Database connection | `db_connection` | `hopefully_still_connected` |
| Config file path | `config_path` | `the_file_that_controls_everything` |
| API response | `api_response` | `what_the_external_thing_said` |
| Current timestamp | `timestamp` | `the_exact_moment_everything_went_wrong` |
| Boolean flag | `is_valid` | `seems_fine` |
| Loop index | `i` | `how_many_times_have_we_been_here` |

[XKCD #1513](https://xkcd.com/1513/) perfectly captures this: code quality is directly correlated with the emotional state of the developer at 3 AM. ESDD simply makes that visible.

## "But What About Readability?"

Readability is a myth perpetuated by people who want to be able to read other people's code. We call these people "coworkers" and their appetite for readable code is frankly unsettling.

Here is a counterargument: when was the last time you read code and thought "I know exactly what this does AND exactly why it was written?" Never. Because conventional naming hides the truth.

ESDD names tell you:
1. What the variable probably contains
2. When it was written (implicitly, by the emotion)
3. How much the author trusted it (very little)
4. Whether the code is load-bearing (if it's named `pleaseGodDontBreak`, yes)

## The Senior Engineer Variable

After 47 years, I have only one variable name that I use for everything:

```python
thing = get_thing()
other_thing = thing.process()
final_thing = other_thing.transform(thing)

# For when I need a second variable of the same type:
thing2 = get_other_thing()

# For when I need a third:
thing_new = get_new_thing()  # I tried 'thing3' but a linter complained

# The boolean I use for every conditional:
ok = check_if_ok()
if ok:
    do_the_thing(thing)
```

I have maintained this codebase for 12 years. Nobody else can read it. I have not written documentation. I have not been fired.

This is the way.

## Conclusion

Variable naming is a conversation. Conventional naming says: "Hello, this is a `userAccountBalance`, it holds a decimal number representing the user's financial state."

ESDD naming says: "Hello, this is `moneyPleaseDontBeNegative`, I wrote it on a Friday afternoon when I had been awake for 19 hours and the CFO was watching the deployment."

One of these is more honest. One of these reflects the true nature of software engineering. One of these will make future engineers — when they inherit your code at 2 AM during an incident — feel understood.

They'll know they're not alone.

They'll know someone has been here before.

They'll know that the variable named `almostThere` was written by someone who thought they were almost there, and wasn't, and that's okay.

That's all any of us can ask for.

---

*The author's codebase contains 3,847 variables named some variant of `temp`. He considers this a feature.*
