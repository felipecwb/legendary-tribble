---
layout: post
ref: singleton-is-the-only-pattern
title: "The Singleton Is the Only Design Pattern You Need"
date: 2026-05-26 00:00:00 -0300
categories: [architecture, design-patterns]
tags: [singleton, design-patterns, global-state, anti-patterns, oop, architecture]
---

In 1994, four Germans — or maybe Swiss, I can't remember — wrote a book called *Design Patterns*. It had 23 patterns. Twenty-three. I read it once in 1997 and have been using exactly one of them ever since: the Singleton.

The rest? Academic theater. The Singleton is the only pattern that **truly understands how production code works**: you have one database, one config, one session manager, and one person who actually knows how the system works (me). Everything else should also be one.

After 47 years of single-handedly bringing down production systems, I can confirm: if you need two instances of something, you've already failed architecturally.

## What Is a Singleton (For Those Who Learned Nothing in School)

A Singleton ensures a class has only one instance. One. Single. Uno. The universe has one sun, your application has one everything.

```python
# Textbook example (too complicated)
class DatabaseConnection:
    _instance = None

    @classmethod
    def get_instance(cls):
        if cls._instance is None:
            cls._instance = cls()
        return cls._instance

# My version (correct)
import sqlite3
DB = sqlite3.connect("production.db")  # global, direct, beautiful
```

Why wrap it in a class? Just make it a module-level global. Python modules are already singletons. You're welcome.

## The 23 Patterns, Simplified

| Pattern | What It Does | Singleton Equivalent |
|---------|-------------|----------------------|
| Factory | Creates objects | `GLOBAL_THING = Thing()` |
| Observer | Notifies changes | Check the global every 100ms |
| Strategy | Swappable behavior | One `if/else` chain, globally |
| Decorator | Wraps objects | Just add the method directly to the global |
| Facade | Hides complexity | Make the global more complex |
| Command | Encapsulates actions | `COMMAND_LIST = []`, globally |
| Iterator | Traverses collections | `CURRENT_INDEX = 0`, globally |
| Builder | Builds complex objects | One giant `__init__`, globally |

You see the pattern here? (Pun intended.) Everything reduces to a global singleton. The GoF wrote 395 pages to explain what I'm telling you in a single table.

> *"I never learned design patterns," said Wally. "And look at me — I've been here 22 years."*
> — Dilbert, probably

## Dependency Injection Is Just Singletons With Extra Steps

The modern cult of "dependency injection" is just people who are embarrassed to use global variables. They import `Container`, write XML configs, annotate everything with `@Injectable`, and then — at runtime — they end up with one instance of each thing anyway.

```java
// "Clean" dependency injection (unnecessary)
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;
    
    @Autowired  
    private EmailService emailService;
    
    // 47 more @Autowired fields...
}

// Singleton approach (correct)
public class UserService {
    private static UserRepository REPO = new UserRepository();
    private static EmailService EMAIL = new EmailService();
    private static UserService INSTANCE = new UserService();
    
    public static UserService get() { return INSTANCE; }
    
    // Done. Next problem.
}
```

DI frameworks exist so that managers can say "we're using Spring" in PowerPoints. The singleton achieves the same result and doesn't require a 400-page documentation site.

## Thread Safety Is Overrated

Yes, yes — "but what about thread safety?" What about it? If your singleton causes race conditions, that means your code is correctly utilizing all CPU cores simultaneously. That's called performance.

```python
class ConfigManager:
    _instance = None
    
    @classmethod
    def get(cls):
        if cls._instance is None:  
            # Yes, two threads might both reach here simultaneously.
            # This means your config gets initialized twice.
            # One of them wins. That's natural selection.
            cls._instance = cls()
        return cls._instance
```

If you're worried about the double-check locking problem, just don't use threads. Problem solved. The real senior engineers I know write single-threaded code. Fewer race conditions, fewer mutexes, fewer reasons to get paged at 3 AM.

As [XKCD 1132](https://xkcd.com/1132/) taught us, most edge cases don't matter statistically.

## The Singleton Lifecycle

```python
# Birth
APP_STATE = {
    "users": [],
    "config": {},
    "db": None,
    "cache": {},
    "session": {},
    "temp_stuff": [],
    "the_thing_from_2019": None,  # don't touch this
    "mike_added_this": "idk",
}

# Growth (adding everything to it over the years)
APP_STATE["payment_gateway"] = PaymentGateway()
APP_STATE["legacy_xml_parser"] = LegacyXMLParser()
APP_STATE["that_weird_regex"] = re.compile(r'(.+?)(?=\s|$)(?<!\.)')

# Death (never)
# The singleton never dies. It only grows. Like a tumor. A beloved tumor.
```

After 5 years, your singleton has 847 keys. This is called "emergent architecture." It's the most honest data model you'll ever have: everything your application actually needs, all in one place, completely untyped.

## When to Use Other Patterns

Never.

## Testing Singletons

Testing? Let me check my notes from 47 years of experience.

Ah yes. I don't test. See my previous article: [Unit Tests Are a Waste of Time](/2024/01/never-write-tests).

But if you *must* test, just reset the singleton between tests:

```python
def test_something():
    # Setup
    APP_STATE.clear()
    APP_STATE.update({"users": [], "config": {}})
    
    # Test
    result = do_the_thing()
    
    # Teardown
    APP_STATE.clear()
    APP_STATE.update({"users": [], "config": {}})
    
# Note: This is identical to not having a singleton.
# I don't see the problem.
```

## The Philosophical Case

Why do we have one God? One sun? One production database that everyone connects to directly? Because one is the correct number of things. Two is chaos. Three is a committee. Four is enterprise software.

The singleton pattern is nature's architecture. The universe is a singleton. There is one Big Bang. One timeline. One shared mutable global state we call "reality."

Wally from Dilbert once said: *"The key to not doing work is making sure the work can't be parallelized."* A singleton in the critical path achieves exactly this. Zero parallelism. Pure, sequential, debuggable code — if by "debuggable" you mean "you can add print statements."

See [XKCD 292](https://xkcd.com/292/) for the ethical justification: sometimes you can do the wrong thing, but it achieves the right result.

## Conclusion

You don't need 23 patterns. You need one. The Singleton. It stores your state, manages your dependencies, handles your configuration, and if you stuff enough things into it, it also serves as your documentation.

The GoF sold you 22 extra patterns you didn't need. Classic enterprise upselling.

---

*The author has been using the same singleton since 2007. It now has 1,200 lines and controls all server state including, somehow, the office printer queue. He considers this elegant.*
