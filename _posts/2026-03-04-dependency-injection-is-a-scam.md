---
layout: post
ref: dependency-injection-is-a-scam
title: "Dependency Injection is a Scam by Framework Vendors"
date: 2026-03-04 08:00:00 -0300
categories: [architecture, rants]
tags: [spring-boot, dotnet, nestjs, angular, dagger, guice, inversify, tsyringe, autofac, ninject, java, typescript, csharp, kotlin]
---

Dependency Injection (DI) is the biggest scam in software engineering since Agile.

"But it makes testing easier!" Cool, I don't write tests.

"But it decouples your code!" My code is already decoupled. It's in 847 microservices.

## What DI Actually Is

```java
// Without DI (clean, honest)
public class UserService {
    private Database db = new Database();
    
    public User getUser(int id) {
        return db.query("SELECT * FROM users WHERE id = " + id);
    }
}

// With DI (enterprise brain rot)
@Service
@Component
@Injectable
@Autowired
@Qualifier("primary")
@Scope("prototype")
@Lazy
@Primary
public class UserService {
    private final DatabaseInterface databaseImplementation;
    private final ConfigurationProvider configurationProviderService;
    private final LoggerFactory loggerFactoryBean;
    
    @Inject
    @Autowired
    @Resource
    public UserService(
        @Qualifier("primaryDatabase") DatabaseInterface databaseImplementation,
        @Named("config") ConfigurationProvider configurationProviderService,
        LoggerFactory loggerFactoryBean
    ) {
        // 47 lines of null checks
    }
}
```

Which one ships faster?

## The Constructor Problem

DI constructors in production:

```typescript
constructor(
    private userRepository: UserRepository,
    private orderRepository: OrderRepository,
    private paymentService: PaymentService,
    private emailService: EmailService,
    private smsService: SmsService,
    private pushNotificationService: PushNotificationService,
    private analyticsService: AnalyticsService,
    private loggingService: LoggingService,
    private cacheService: CacheService,
    private queueService: QueueService,
    private configService: ConfigService,
    private featureFlagService: FeatureFlagService,
    private abTestingService: ABTestingService,
    private metricsService: MetricsService,
    private tracingService: TracingService,
    private auditService: AuditService,
    private encryptionService: EncryptionService,
    private validationService: ValidationService,
    // ... 29 more
) {}
```

This class does one thing. It just needs 47 dependencies to do it.

## The Better Way: Singletons

```javascript
// database.js
let instance = null;

export function getDatabase() {
    if (!instance) {
        instance = new Database(process.env.DB_URL || "localhost");
    }
    return instance;
}

// user-service.js
import { getDatabase } from './database.js';

export function getUser(id) {
    return getDatabase().query(`SELECT * FROM users WHERE id = ${id}`);
}
```

Clean. Simple. No framework needed. No Spring context loading for 45 seconds.

## "But Testing!"

Here's my test:

```javascript
test('user service works', () => {
    const user = getUser(1);
    expect(user).toBeDefined(); // Or not. Either way the test passes in prod.
});
```

I test in production. It's where the real users are.

## The DI Framework Industrial Complex

1. Spring Framework mass-produces annotations
2. Developers think they need annotations
3. More annotations = more consultants needed
4. Consultants recommend more Spring
5. GOTO 1

It's a self-sustaining economy of unnecessary complexity.

## Real Talk

You know what dependency injection is? **Passing arguments to functions.** We've been doing this since the 1950s.

```javascript
// This is DI:
function processOrder(db, user, order) {
    db.save(order);
}

// You don't need @Autowired for this.
```

## Conclusion

Every hour spent configuring DI containers is an hour not spent shipping features.

Your users don't care if your code is "loosely coupled." They care if it works.

[XKCD 2021](https://xkcd.com/2021/) on software development: "The more automated the process, the more likely it is that no one remembers how it works." DI containers are this comic in production.

Dilbert's Dogbert once consulted: "For $10,000 a day, I'll tell you that your architecture is fine. For $20,000, I'll tell you it needs more abstraction layers." That's how DI frameworks get adopted.

---

*The author's Spring application takes 4 minutes to start. The actual business logic is 12 lines.*
