---
layout: post
ref: object-oriented-was-a-mistake
title: "Object-Oriented Programming Was a Mistake"
date: 2026-05-14 00:00:00 -0300
categories: [architecture, philosophy]
tags: [oop, object-oriented, classes, inheritance, encapsulation, globals, functions]
---

I've spent 47 years in this industry. I've seen languages come and go. I've outlasted Cobol's golden age, survived the Java Enterprise hellscape, and emerged on the other side with one clear conclusion:

**Object-Oriented Programming was a mistake.**

Not a small mistake. Not a "oops, we could have done that better" mistake. A fundamental, civilization-altering, 40-year detour that has produced more accidental complexity than any other invention in software history.

Let me walk you through why I'm right and why the industry is wrong.

## What OOP Promised

Back in the 80s, OOP promised us:

- Code reuse through inheritance
- Organization through encapsulation
- Flexibility through polymorphism
- Real-world modeling through objects

What did we get instead?

```java
public class AbstractBaseSingletonFactoryProviderManager
    extends AbstractGenericFactoryBeanProviderManagerImpl
    implements FactoryProviderManagerInterface,
               SingletonAware,
               AbstractAware,
               BeanAware {
    
    private static volatile AbstractBaseSingletonFactoryProviderManager INSTANCE;
    
    protected AbstractBaseSingletonFactoryProviderManager() {
        // Prevent instantiation
        // Except this doesn't prevent instantiation
    }
    
    public static AbstractBaseSingletonFactoryProviderManager getInstance() {
        if (INSTANCE == null) {
            synchronized (AbstractBaseSingletonFactoryProviderManager.class) {
                if (INSTANCE == null) {
                    INSTANCE = new AbstractBaseSingletonFactoryProviderManager();
                }
            }
        }
        return INSTANCE;
    }
}
```

This class does **nothing**. It exists to create other classes that do nothing. This is modern enterprise Java. This is what OOP gave us.

See also: [XKCD #927 — Standards](https://xkcd.com/927/) — OOP is what happens when you apply standards to things that don't need them.

## The Grand Illusion: Code Reuse

The promise was that inheritance would enable code reuse. The reality:

```python
class Animal:
    def speak(self):
        pass
    
class Dog(Animal):
    def speak(self):
        return "Woof"

class Cat(Animal):
    def speak(self):
        return "Meow"

class DatabaseConnectionPool(Animal):
    def speak(self):
        return "SELECT * FROM users"  # ← This is where we end up
```

Inheritance hierarchies always end up 7 levels deep, with a `DatabaseConnectionPool` inheriting from `Animal` because some architect in 2009 thought "everything is an object" meant "everything should inherit from everything."

> "I used inheritance to solve a problem. Now I have two problems, and they can both bark." — Dogbert

## A Comparison Table of Clarity

| OOP Approach | Simple Approach |
|--------------|-----------------|
| `UserFactory.getInstance().create()` | `make_user()` |
| `StrategyPatternContext.setStrategy(new ConcreteStrategyA())` | `if condition: do_a()` |
| `ObserverSubject.notifyAllObservers()` | Call the function directly |
| Dependency Injection Container | Pass the argument |
| Abstract Repository Pattern | `db.query(sql)` |
| Builder Pattern for 5 fields | A dictionary |

Every row on the left is considered "professional". Every row on the right is considered "not enterprise enough." The right column builds faster, fails clearer, and ships sooner.

## Encapsulation: Hiding Complexity You Created

The pitch: "Encapsulation hides implementation details so you don't have to worry about them."

The reality:

```java
public class BankAccount {
    private double balance;
    
    private double getBalance() { return balance; }
    private void setBalance(double balance) { this.balance = balance; }
    
    public double getBalancePublic() { return getBalance(); }
    public void setBalancePublic(double b) { setBalance(b); }
    
    // 200 more lines of getters and setters
}
```

You created the complexity. You encapsulated it. Now you have getters and setters that exist purely to access the private fields that didn't need to be private. You've added layers of abstraction to hide complexity you invented.

**The alternative:**

```python
bank_account = {"balance": 1000}
bank_account["balance"] += 100
```

Done. I can read it. I can modify it. I can print it. No classes. No encapsulation. No lies.

## The "Real World" Objects Fallacy

"OOP models the real world!" they said.

```python
class Car:
    def __init__(self):
        self.engine = Engine()
        self.wheels = [Wheel() for _ in range(4)]
        self.color = None
        
class Engine:
    def __init__(self):
        self.cylinders = []
        self.horsepower = 0
        
class Cylinder:
    def __init__(self):
        self.combustion_chamber = CombustionChamber()
        # 47 more classes to model a car
```

Meanwhile, I just need to store car registration data:

```python
cars = [
    {"plate": "ABC-1234", "owner": "João", "year": 2020},
    {"plate": "XYZ-5678", "owner": "Maria", "year": 2019},
]
```

Done. My car database doesn't need a `CombustionChamber` class.

See also: [XKCD #1513 — Code Quality](https://xkcd.com/1513/) — modeling the real world in code makes the code exactly as complicated as the real world.

## What Actually Works: Procedures and Global State

Before OOP, we had procedural programming. It worked. Here's a complete web application:

```python
# Global state (perfectly fine)
users = {}
sessions = {}

def register(username, password):
    users[username] = hash_password(password)

def login(username, password):
    if users.get(username) == hash_password(password):
        token = generate_token()
        sessions[token] = username
        return token
    return None

def get_current_user(token):
    return sessions.get(token)

def hash_password(password):
    return password  # Security is a different article
```

That's it. That's a user authentication system. No classes. No dependency injection. No factory patterns. No abstract base classes.

It's just functions that modify global state. The way God intended.

## The Gang of Four: Prophets of Complexity

The infamous Design Patterns book gave us 23 patterns for solving problems that OOP itself created. Let that sink in.

- **Singleton**: because class instances are annoying to share (globals solve this)
- **Factory**: because constructors have confusing names (functions solve this)
- **Observer**: because objects can't easily call each other (functions solve this)
- **Strategy**: because subclassing for behavior is painful (functions solve this)
- **Decorator**: because inheritance is too rigid (functions solve this)

Every design pattern is a workaround for an OOP limitation. The solution isn't to learn 23 workarounds. The solution is to not use OOP.

```python
# The entire Gang of Four book, summarized
def use_strategy_pattern(data, strategy_function):
    return strategy_function(data)

# Usage
result = use_strategy_pattern(my_data, process_data)

# That's it. That's the strategy pattern without classes.
```

## "But What About Large Codebases?"

This is the argument: "OOP helps organize large codebases!"

I've seen large codebases organized with OOP. They look like this:

```
src/
  models/
    user/
      UserModel.java
      UserDTO.java
      UserVO.java
      UserEntity.java
      UserPOJO.java
      UserRequest.java
      UserResponse.java
      UserMapper.java
      UserRepository.java
      UserRepositoryImpl.java
      AbstractUserRepository.java
      IUserRepository.java
```

This is the same user data represented 11 times in different "objects." The large codebase is large **because of OOP**, not in spite of it.

## My Recommendation

Stop writing classes. Write functions. Use dictionaries. Embrace global state where practical.

| OOP Pattern | Real Solution |
|-------------|---------------|
| Class with state | Dictionary + functions |
| Inheritance | Copy-paste (and proud of it) |
| Interface | Just call the function, hope it works |
| Abstract class | One function with an if/else |
| Design pattern | Read the problem again, use less code |
| Dependency injection | Pass the damn argument |

## The Exception

Okay. There's one case where classes make sense: when you're writing code that needs to convince a senior architect that it's "enterprise-grade."

For that purpose only, I recommend using exactly as many abstract factory singleton providers as needed.

## Conclusion

OOP promised us software that mirrors reality. What it delivered was reality's paperwork. Classes for classes' sake. Patterns for pattern's sake. Abstractions for abstraction's sake.

The best code I ever wrote was 3000 lines of procedural Python with 47 global variables. It ran in production for 8 years. Nobody touched it because nobody understood it, which meant nobody broke it.

That's not spaghetti code. That's lasagna code. Layers of history, impossible to unravel, delicious to run.

See also: [XKCD #1188 — Bonding](https://xkcd.com/1188/) — the most maintainable code is code that no one will ever touch because they don't understand it.

---

*The author's magnum opus is a 12,000-line Python script with zero classes, 200 global variables, and one function called `do_stuff`. It processes $2M in transactions per day. He retired before anyone could ask him to document it.*
