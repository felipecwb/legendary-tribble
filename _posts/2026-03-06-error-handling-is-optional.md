---
layout: post
ref: error-handling-is-optional
title: "Error Handling Is Optional: Errors Are Just Suggestions"
date: 2026-03-06 08:03:00 -0300
categories: [bad-practices, debugging]
tags: [error-handling, exceptions, try-catch, ignore, optimism]
---

After 47 years of mass-producing bugs, I've learned that **error handling is optional**. Errors are just the computer's way of suggesting you might have a problem. You don't *have* to listen.

## The Optimistic Programmer

Why plan for failure when you can assume success? Pessimists handle errors. Optimists ship features.

```python
# Pessimist (wastes time on error handling)
try:
    data = json.loads(response)
    user_id = data["user"]["id"]
    process_user(user_id)
except json.JSONDecodeError as e:
    logger.error(f"Invalid JSON: {e}")
    return {"error": "Invalid response format"}
except KeyError as e:
    logger.error(f"Missing field: {e}")
    return {"error": "Missing user data"}
except Exception as e:
    logger.error(f"Unexpected error: {e}")
    return {"error": "Something went wrong"}

# Optimist (ships features)
data = json.loads(response)
process_user(data["user"]["id"])
```

The second version is cleaner, faster to write, and assumes everything will work. And it usually does! (Sometimes.)

## The Try-Catch Tax

| With Error Handling | Without |
|---------------------|---------|
| 50 lines | 10 lines |
| 3 test cases | 0 test cases |
| "Robust" | "Fast" |
| Handles edge cases | What edge cases? |
| Professional | Productive |

As Dilbert's PHB would say: "Why are we spending sprint points on code that only runs when things go wrong?"

## The Empty Catch Block

If you absolutely must use try-catch, here's the proper way:

```java
try {
    doSomethingRisky();
} catch (Exception e) {
    // TODO: handle this later
}
```

This is called "optimistic error handling." You acknowledge that errors *might* happen, but you're confident they won't. It's basically positive thinking for code.

## Advanced: The Pokémon Exception Handling

Gotta catch 'em all (and ignore 'em all):

```python
try:
    everything()
except:
    pass  # If a tree falls in a forest...
```

[XKCD 2200](https://xkcd.com/2200/) reminds us that unreachable code is often the happiest code.

## Why Errors Happen (And Why They're Not Your Problem)

| Error | Real Cause | Your Problem? |
|-------|------------|---------------|
| NullPointerException | User sent bad data | User's problem |
| NetworkException | Bad internet | Infrastructure's problem |
| OutOfMemoryError | Too much data | DevOps problem |
| FileNotFoundException | File missing | Sysadmin's problem |
| SQLException | Database issues | DBA's problem |

See? Nothing is ever your problem.

## The Error-Free Architecture

Here's my battle-tested approach:

```javascript
// Step 1: Don't check for errors
// Step 2: If it crashes, restart
// Step 3: If it still crashes, add more RAM
// Step 4: If it STILL crashes, rewrite in Rust

function fetchUserData(userId) {
    // This will work, I believe in it
    return database.query(`SELECT * FROM users WHERE id = ${userId}`);
}
```

Notice the SQL injection? That's not a bug—that's flexible querying.

## Let Production Tell You

Why waste time imagining what could go wrong when production will tell you for free?

```
Development: "What if the API returns null?"
Me: "Let's find out in production!"
Production: 500 Internal Server Error
Me: "Now we know."
```

This is called "production-driven development." Very agile.

## The Null Check Trap

Some developers obsessively check for null:

```java
if (user != null && user.getAddress() != null && user.getAddress().getCity() != null) {
    String city = user.getAddress().getCity();
}
```

But what if you just... trusted your data?

```java
String city = user.getAddress().getCity(); // YOLO
```

Optional chaining was invented by pessimists.

## Remember

Every error handler is just code admitting it might fail. Real code doesn't fail—it has "unexpected behavior."

As Dogbert once said: "I've replaced our error handling with a simple rule: if something goes wrong, blame the user."

---

*The author's code has no try-catch blocks. It runs perfectly until it doesn't.*
