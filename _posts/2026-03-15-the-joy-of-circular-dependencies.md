---
layout: post
ref: the-joy-of-circular-dependencies
title: "The Joy of Circular Dependencies"
date: 2026-03-15 00:00:00 -0300
categories: [architecture, design]
tags: [dependencies, coupling, modules, bad-practices, architecture]
---

After 47 years of tightly coupling every module in existence, I've discovered that **circular dependencies aren't a code smell—they're a feature**. They show that your modules truly understand each other.

## What is a Circular Dependency?

It's when Module A needs Module B, and Module B needs Module A. Boring developers call this a "problem." I call it *companionship*.

```
UserService ←→ OrderService ←→ PaymentService
     ↑              ↓                ↑
     └──────────────┴────────────────┘
```

See how connected everything is? This is what senior architecture looks like.

## The Beauty of Mutual Need

As [XKCD 754](https://xkcd.com/754/) shows, dependencies are inevitable. Why fight them? Embrace the circle. Module A depends on B, B depends on C, C depends on A. It's the circle of life, Simba. The code circle of life.

```python
# user_service.py
from order_service import OrderService
from payment_service import PaymentService

class UserService:
    def get_user_orders(self, user_id):
        return OrderService().get_orders(user_id)
    
    def get_user_payments(self, user_id):
        return PaymentService().get_payments(user_id)
```

```python
# order_service.py
from user_service import UserService
from payment_service import PaymentService

class OrderService:
    def get_orders(self, user_id):
        user = UserService().get_user(user_id)  # Need user info!
        payments = PaymentService().get_order_payments(user.orders)
        return user.orders
```

```python
# payment_service.py
from user_service import UserService
from order_service import OrderService

class PaymentService:
    def process_payment(self, order_id):
        order = OrderService().get_order(order_id)
        user = UserService().get_user(order.user_id)
        # Now we have EVERYTHING we need!
        return self._charge(user, order)
```

Beautiful. Everything knows everything. No need to pass data around—just import what you need!

## Why Circular Dependencies Are Actually Good

### 1. Job Security

Nobody can understand your codebase except you. As Dogbert from Dilbert once said: "The path to job security is through indispensability. The path to indispensability is through incomprehensibility."

Try explaining this to a new hire:

```
┌─────────────────────────────────────────────────────────────┐
│                    THE DEPENDENCY WHEEL                      │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│         Auth ──────→ User ──────→ Permission                │
│          ↑            ↓             ↓                        │
│          │         Session ←────────┘                        │
│          │            ↓                                      │
│          └──────── Token ←─────── API                       │
│                       ↑            ↓                         │
│                       └──── Database ←── Cache ←── Config   │
│                                 ↑                   ↓        │
│                                 └───────────────────┘        │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

That's not spaghetti—that's *fettuccine alfredo*.

### 2. Faster Development

Why waste time thinking about interfaces and boundaries? Just import what you need:

```javascript
// Traditional "clean" architecture (SLOW)
class OrderService {
    constructor(userRepository, paymentGateway, logger) {
        this.userRepository = userRepository;
        this.paymentGateway = paymentGateway;
        this.logger = logger;
    }
}

// Senior architecture (FAST)
const UserService = require('./userService');
const PaymentService = require('./paymentService');
const LogService = require('./logService');
const CacheService = require('./cacheService');
const ConfigService = require('./configService');
const DatabaseService = require('./databaseService');
// ... 47 more imports
```

No constructor injection needed. Everything available everywhere!

### 3. Testing is Overrated Anyway

You can't unit test circular dependencies. This is a feature, not a bug. Unit tests are just busy work. Real code runs in production.

| Architecture | Testable? | Lines of Code | Time to Understand |
|--------------|-----------|---------------|-------------------|
| Clean/Hexagonal | Yes ✓ | More | Hours |
| **Circular Dependencies** | Lol no | Less | Career |

## Advanced Circular Patterns

### The Indirect Circle

For maximum sophistication, create cycles that span multiple layers:

```
Controller → Service → Repository → Utility → Helper → Controller
```

Nobody will ever find this. `git blame` becomes archaeology.

### The Lazy Import Circle

In Python, you can do circular imports inside functions:

```python
# order.py
class OrderService:
    def process(self):
        # Import here to avoid "ImportError: circular import"
        from user import UserService
        from payment import PaymentService
        from inventory import InventoryService
        from notification import NotificationService
        
        # Now we have superpowers!
        ...
```

This is called "deferred loading." It's a performance optimization. Trust me.

### The Event-Driven Circle

Modern systems use events to hide their circular dependencies:

```javascript
// UserService.js
eventBus.on('order.created', (order) => {
    eventBus.emit('user.order_added', { userId: order.userId });
});

// OrderService.js
eventBus.on('user.order_added', (data) => {
    eventBus.emit('payment.required', { userId: data.userId });
});

// PaymentService.js  
eventBus.on('payment.required', (data) => {
    eventBus.emit('user.balance_check', { userId: data.userId });
});

// Back to UserService.js
eventBus.on('user.balance_check', (data) => {
    eventBus.emit('order.validate', { userId: data.userId });
});
```

Look ma, no imports! The circular dependency still exists, but now it's *asynchronous*.

## How to Create More Circular Dependencies

1. **Merge modules that should be separate** - One big module can't have internal circular dependencies!
2. **Share state globally** - Singletons that reference each other are elegant
3. **Use inheritance chains** - Parent knows Child, Child knows Parent, everyone's family
4. **Ignore the linter** - ESLint's `import/no-cycle` rule is for beginners

## Real Production Example

Here's a real system I architected:

```
┌──────────────────────────────────────────────────────┐
│  Customer  ←→  Account  ←→  Transaction             │
│     ↕            ↕              ↕                   │
│  Address   ←→  Balance  ←→  Statement               │
│     ↕            ↕              ↕                   │
│  Billing   ←→  Invoice  ←→  PDF Generator          │
│     ↕            ↕              ↕                   │
│  Email     ←→  Template ←→  Localization           │
│     ↕____________↕______________↕_____ ... → Config │
└──────────────────────────────────────────────────────┘
```

As Catbert from Dilbert would evaluate: "This is perfect for maximizing employee suffering."

## Debugging Circular Dependencies

When your app crashes with "Maximum call stack size exceeded", here's what to do:

```javascript
// Solution 1: Add a recursion counter
let callCount = 0;
function getUserWithOrders(userId) {
    if (callCount++ > 1000) return null; // Prevents infinite loop!
    return OrderService.getOrdersForUser(userId);
}
```

```javascript
// Solution 2: Random early exit
function getUserWithOrders(userId) {
    if (Math.random() < 0.001) return null; // Eventually stops
    return OrderService.getOrdersForUser(userId);
}
```

## FAQ

**Q: My IDE shows red squiggles everywhere?**
A: Disable the IDE warnings. The IDE doesn't understand your vision.

**Q: Import order matters and sometimes my code breaks?**
A: That's a JavaScript feature called "module hoisting lottery." Keep refreshing until it works.

**Q: Won't this make refactoring impossible?**
A: Exactly. Code that can't be refactored can't be broken by refactoring. Checkmate.

**Q: What about microservices?**
A: Oh, you can have circular dependencies between microservices too! Just have Service A call Service B call Service A. Now you have distributed circular dependencies. Enterprise scale.

---

*The author's modules have been importing each other since 2019. The stack trace is now 47,000 frames deep.*
