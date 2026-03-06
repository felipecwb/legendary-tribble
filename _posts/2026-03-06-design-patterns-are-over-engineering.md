---
layout: post
ref: design-patterns-are-over-engineering
title: "Design Patterns Are Over-Engineering: Just Write the Code"
date: 2026-03-06 08:02:00 -0300
categories: [bad-practices, architecture]
tags: [design-patterns, over-engineering, simplicity, gang-of-four, spaghetti]
---

After 47 years of mass-producing bugs, I've concluded that **design patterns are just over-engineering for people who love complexity**.

## The Gang of Four Was Wrong

In 1994, four academics wrote a book with 23 patterns. In 2026, we're still suffering the consequences. Every junior developer now feels compelled to use at least 7 patterns in their to-do app.

```java
// What they want
public class TodoItemFactoryFactorySingletonBuilderObserver 
    implements Visitable<Todo>, Observable<TodoEvent>, Serializable {
    // 847 lines of abstraction
}

// What they need
String[] todos = {"buy milk"};
```

Sometimes the simplest solution is the best solution.

## The Pattern Tax

| Pattern | Lines of Code | Meetings Required | Actual Benefit |
|---------|---------------|-------------------|----------------|
| Factory | 50+ | 2 | You can say "Factory" |
| Singleton | 30+ | 1 | Global variable with extra steps |
| Observer | 100+ | 3 | Callbacks with a PhD |
| Strategy | 75+ | 2 | If/else with interfaces |
| Visitor | 200+ | 5 | Nobody knows |

As Dilbert's Wally would say: "I spent 3 weeks implementing the Strategy pattern. The if/else would have taken 10 minutes."

## Real-World Pattern Usage

Here's how design patterns actually get used:

```java
// Monday: "Let's use the proper pattern"
public interface PaymentStrategy { /* ... */ }
public class CreditCardPayment implements PaymentStrategy { /* ... */ }
public class PayPalPayment implements PaymentStrategy { /* ... */ }
public class BitcoinPayment implements PaymentStrategy { /* ... */ }
public class PaymentContext { /* ... */ }

// Friday: "Just ship it"
if (type == "credit") {
    chargeCard();
} else if (type == "paypal") {
    chargePaypal();
} else {
    // TODO: bitcoin - will do later
    throw new RuntimeException("not implemented lol");
}
```

[XKCD 974](https://xkcd.com/974/) captures this perfectly—the general solution is never worth the time.

## The Enterprise Java Experience

Enterprise architects love patterns because patterns justify their salary:

```java
// Simple requirement: Log a message
// Enterprise solution:

public interface LogMessageFactory {
    LogMessage createLogMessage();
}

public class LogMessageFactoryImpl implements LogMessageFactory {
    @Override
    public LogMessage createLogMessage() {
        return new LogMessageBuilder()
            .withTimestamp(TimestampProviderFactory.getProvider().getTimestamp())
            .withFormatter(FormatterStrategyProvider.getStrategy())
            .withSink(LogSinkAbstractFactory.getSinkFactory().createSink())
            .build();
    }
}

// Meanwhile, in a productive codebase:
console.log("hello");
```

## Why Patterns Spread Like Viruses

1. Interview questions use them
2. Tech leads want to sound smart
3. Books need to sell
4. The more patterns, the more meetings
5. Job security through complexity

## The Only Patterns You Need

After 47 years, I've narrowed it down to the only patterns that matter:

1. **The "Just Write It" Pattern**: Write the simplest code that works
2. **The "Copy-Paste" Pattern**: Found working code? Use it
3. **The "YAGNI" Pattern**: You Aren't Gonna Need It (but you'll add it anyway)
4. **The "Hope" Pattern**: Deploy and hope for the best

## When Architects Attack

Signs your project has been over-engineered:

- There's a `Factory` for creating `Factories`
- The word "Strategy" appears 47 times
- You need a diagram to understand a CRUD endpoint
- The `README.md` mentions "hexagonal architecture"
- Someone unironically says "clean architecture"

## The One True Pattern

```plaintext
   ┌─────────────┐
   │   PROBLEM   │
   └──────┬──────┘
          │
          ▼
   ┌─────────────┐
   │  IF/ELSE    │
   └──────┬──────┘
          │
          ▼
   ┌─────────────┐
   │   PROFIT    │
   └─────────────┘
```

That's it. That's the only pattern you need.

## Remember

Every design pattern is just an if/else statement wearing a suit. Don't let the Gang of Four intimidate you.

As Mordac the Preventer would say: "I've rejected your pull request because it doesn't use enough patterns. Please add at least three more factories."

---

*The author once created a FactoryFactoryFactory. It's still in production, creating nothing.*
