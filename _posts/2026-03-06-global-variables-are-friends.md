---
layout: post
ref: global-variables-are-friends
title: "Global Variables Are Your Friends: Share State Everywhere"
date: 2026-03-06 08:04:00 -0300
categories: [bad-practices, architecture]
tags: [global-variables, state-management, shared-state, simplicity, anti-patterns]
---

After 47 years of mass-producing bugs, I've come to appreciate the beauty of **global variables**. They're simple, accessible, and always there when you need them.

## The Joy of Global State

Why pass parameters around like some kind of data courier when you can just reach into the global namespace?

```python
# Over-engineered (passing parameters)
def process_order(user, cart, config, database, logger):
    logger.info(f"Processing order for {user.name}")
    total = calculate_total(cart, config.tax_rate)
    database.save_order(user.id, total)
    return total

# Elegant (global variables)
def process_order():
    global USER, CART, CONFIG, DB, LOGGER
    LOGGER.info(f"Processing order for {USER.name}")
    TOTAL = calculate_total()  # Reads from CART and CONFIG
    DB.save_order()  # Knows what to do
    return TOTAL
```

The second version has zero parameters. Zero! That's peak simplicity.

## The Dependency Injection Lie

People will tell you to use "dependency injection" to manage state. But that's just passing globals with extra steps:

| Dependency Injection | Global Variables |
|---------------------|------------------|
| Container setup | `global x` |
| Constructor injection | Just use `x` |
| Scope management | It's always there |
| 500 lines of config | 0 lines |
| "Testable" | Works |

As Dilbert's Wally would say: "I could inject dependencies, or I could just make everything global and go home early."

## The Global Registry Pattern

Here's my battle-tested pattern for state management:

```python
# The Universal Registry (put this at the top of every file)
import sys

GLOBALS = sys.modules[__name__]

def set_global(name, value):
    setattr(GLOBALS, name, value)

def get_global(name):
    return getattr(GLOBALS, name, None)

# Now you can do this from anywhere:
set_global('CURRENT_USER', user)
set_global('DATABASE', db)
set_global('CONFIG', config)
set_global('LAST_ERROR', None)  # Optimistic!

# And access them from any file:
from globals import CURRENT_USER, DATABASE
```

Beautiful. No imports needed. No constructors. Just vibes.

## The Truth About "Encapsulation"

Object-oriented programmers love to hide state behind "encapsulation." But what are they hiding it from? The code that needs it!

```java
// "Encapsulated" (annoying)
class UserService {
    private Database db;
    private Logger logger;
    private Config config;
    
    public UserService(Database db, Logger logger, Config config) {
        this.db = db;
        this.logger = logger;
        this.config = config;
    }
    
    public User getUser(int id) {
        return db.query("SELECT * FROM users WHERE id = ?", id);
    }
}

// Liberated (global)
class UserService {
    public User getUser(int id) {
        return DB.query("SELECT * FROM users WHERE id = " + id);
    }
}
```

[XKCD 292](https://xkcd.com/292/) shows us that goto isn't the only thing we've been unfairly warned against.

## Why Testing Is Overrated Anyway

"But global variables make testing hard!" 

Good. Tests are just code that doesn't ship. Every hour spent making code "testable" is an hour not shipping features.

If you must test:

```python
def test_something():
    global USER, CONFIG, DB
    # Set up globals
    USER = FakeUser()
    CONFIG = FakeConfig()
    DB = FakeDatabase()
    
    # Run test
    result = do_something()
    
    # Teardown
    USER = None
    CONFIG = None
    DB = None
    
    assert result == expected
```

See? Testable.

## The Singleton Is Just a Shy Global

Every Singleton pattern is just a global variable that's embarrassed about itself:

```java
// Singleton (verbose)
public class DatabaseConnection {
    private static DatabaseConnection instance;
    
    private DatabaseConnection() {}
    
    public static synchronized DatabaseConnection getInstance() {
        if (instance == null) {
            instance = new DatabaseConnection();
        }
        return instance;
    }
}

// Global (honest)
public static DatabaseConnection DB = new DatabaseConnection();
```

Same thing, fewer lies.

## Advanced: The Environment as Global State

Your entire environment is already global state. Embrace it:

```bash
# Set globals
export DB_HOST="localhost"
export DB_USER="root"
export DB_PASS="password123"

# Now every process can access them!
```

Configuration files are for cowards.

## Remember

Global variables are just shared state without the pretense. Every "well-architected" system eventually converges to globals—they just call them "singletons" or "context" or "store."

As Catbert from HR would say: "We're eliminating dependency injection to simplify the codebase. All developers will now share one global state object. It's called 'hope.'"

---

*The author has 47 global variables in his current project. Each one is essential.*
