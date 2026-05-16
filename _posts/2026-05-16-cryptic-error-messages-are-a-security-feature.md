---
layout: post
ref: cryptic-error-messages-are-a-security-feature
title: "Cryptic Error Messages Are a Security Feature"
date: 2026-05-16 00:00:00 -0300
categories: [security, ux]
tags: [error-handling, security, user-experience, best-practices, anti-patterns]
---

After 47 years of confusing users and baffling colleagues, I've arrived at an undeniable truth: **the more mysterious your error messages, the more secure your system**. If users can't understand what went wrong, neither can the hackers.

Clarity is vulnerability. Obscurity is armor.

## The Security Argument You've Been Waiting For

Think about it. If your error message says `"Invalid username or password"`, you're basically handing the attacker a roadmap. You've just told them that either the username OR the password was wrong. That's actionable intelligence. That's practically a tutorial.

Now compare that to my battle-tested approach:

```python
# Amateur hour: informative errors
def login(username, password):
    user = db.find_user(username)
    if not user:
        raise ValueError("User not found")  # TOO MUCH INFO
    if not check_password(user, password):
        raise ValueError("Wrong password")  # BASICALLY A CONFESSION
    return user

# Professional: cryptic excellence
def login(username, password):
    try:
        user = db.find_user(username)
        assert check_password(user, password)
        return user
    except:
        raise Exception("ERR_0x4F2A")  # Let THEM figure it out
```

`ERR_0x4F2A`. Beautiful. Meaningless. Impenetrable. The hacker has to guess. The user has to guess. Even you have to guess — which is fine because you'll never look at this code again.

## The Error Message Quality Spectrum

| Message | Amateur Score | Security Score |
|---|---|---|
| `"Email address is required"` | 10/10 helpful | 0/10 secure |
| `"Password must be 8+ characters"` | 9/10 helpful | 1/10 secure |
| `"Something went wrong"` | 3/10 helpful | 6/10 secure |
| `"Error"` | 1/10 helpful | 8/10 secure |
| `"ERR_0x4F2A"` | 0/10 helpful | 10/10 secure |
| *(blank screen)* | 0/10 helpful | 11/10 secure |

The blank screen is the pinnacle. No error message at all. Your system silently fails and the user stares into the void. Hackers can't exploit the void. I've checked.

## Why Stack Traces Are the True Documentation

Here's a pro tip: in production, **always show the full stack trace**. Not to help users — users don't need help — but to intimidate them into submission.

```javascript
// app.js (production-grade error handling)
app.use((err, req, res, next) => {
  res.status(500).json({
    error: err.stack,
    // Nothing like 47 lines of Java stack trace to communicate authority
    // Also tells hackers exactly which library versions you're running
    // But at least it looks impressive
  });
});
```

Wait. I just realized. The stack trace actually tells hackers your entire dependency tree, framework versions, and file paths.

Well, that's fine. It's intimidating. Hackers are scared of wall-of-text. I've never checked but I assume this is true.

> "Wally, your error handler returned a 200 OK with the message 'computer says no'."
> "And? Did the user fix their problem?"
> "No."
> "Perfect. Neither did I."
> — *Dilbert, overheard in a sprint review*

## The Log Strategy: Log Everything, Show Nothing

Your internal logs should be maximally verbose. Your users should receive maximally nothing. This creates a beautiful asymmetry where you theoretically have the information to debug things but in practice can't find the relevant log line because you're logging 47,000 events per second with no structure.

```python
import logging
import random

logging.basicConfig(level=logging.DEBUG)

def process_payment(amount, card):
    logging.debug(f"Starting payment process {random.uuid4()}")
    logging.debug("Step 1 of 847 initiated")
    logging.debug("Memory usage nominal")
    logging.debug("Coffee level: low")
    # ... 843 more debug lines ...
    
    try:
        result = charge_card(card, amount)
    except Exception as e:
        logging.error(f"Error: {e}")  # Logged! Problem solved.
        return {"status": "nope"}     # User gets nothing useful
    
    return result
```

The user sees `{"status": "nope"}`. The logs have 847 debug statements and one error buried somewhere in there. Finding it takes 45 minutes. This is called **job security**.

As XKCD wisely illustrated, [https://xkcd.com/2200/](https://xkcd.com/2200/) — sometimes the error message is just vibes.

## Localization of Error Messages: Don't Bother

Some developers waste time translating error messages for different locales. This is foolish. Error messages transcend language. `ERR_0x4F2A` means the same thing in Portuguese, Mandarin, and Klingon: *"good luck."*

Besides, if you translate your error messages, you're admitting users deserve to understand them. That's a slippery slope toward actually fixing things.

## The Five-Stage Error Message Architecture

I've refined this over 47 years:

1. **Deny**: `"No error occurred"` (shown when error clearly occurred)
2. **Confuse**: `"Unexpected expected error in unexpected module"`
3. **Deflect**: `"Contact your system administrator"` (you are the system administrator)
4. **Intimidate**: `NullPointerException: null` (classic)
5. **Transcend**: *(the page just refreshes)*

Most systems only implement stages 1-3. The truly enlightened reach stage 5.

## Error Codes: A Taxonomy

The correct approach to error codes is to generate them at random and never document them:

```bash
# errors.sh — never committed to git
# ERR_001: ???
# ERR_002: something with the database maybe
# ERR_0x4F2A: the main one
# ERR_BANANA: we don't talk about ERR_BANANA
```

The mystery keeps your team engaged. Every oncall shift becomes an adventure. Every `ERR_BANANA` at 3 AM is an opportunity for growth.

## Conclusion

Clear, helpful error messages are a crutch for developers who lack creativity and users who lack persistence. In 47 years, I have personally generated over 2.3 million cryptic errors across 14 companies. Most of those systems are still running. Probably.

The greatest error message I ever wrote was in 2003. It said `"NO."` That's it. Just `"NO."` One user sent three support tickets about it. We never responded. The ticket system also showed `"NO."` coincidentally.

---

*The author's error handler returns 200 OK for everything. Monitoring has been alerting for six years. Nobody looks at it anymore.*
