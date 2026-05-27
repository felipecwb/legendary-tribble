---
layout: post
ref: yagni-is-for-quitters
title: "YAGNI Is for Quitters: The Case for Pre-emptive Over-Engineering"
date: 2026-05-27 00:00:00 -0300
categories: [architecture, best-practices]
tags: [yagni, over-engineering, enterprise, abstraction, solid, patterns]
---

I've been writing software for 47 years. I've seen trends come and go. Smalltalk. CORBA. SOA. Agile. DevOps. "Trunk-based development." Every decade, consultants invent a new word for "doing less."

But nothing — **nothing** — has caused more catastrophic, career-ending, production-melting disasters in my storied career than the four letters that represent the greatest lie ever told to developers:

**YAGNI.**

"You Ain't Gonna Need It." A convenient excuse invented by lazy engineers who couldn't be bothered to think ahead. Let me tell you something: I have never, in 47 years, **not** needed something I built.

---

## The Real Meaning of YAGNI

YAGNI doesn't stand for "You Ain't Gonna Need It."

It stands for **You're Always Gonna Need It.**

The "A" is for Always. You will *always* need it. You just don't know it yet. That's the whole point.

As Wally from Dilbert once said: *"I've never done anything that wasn't eventually repurposed for something else. Even my bad decisions came in handy at my next job."*

---

## The Correct Architecture for Every Project

When you start a new project — even a TODO app — you must implement the following before writing a single line of business logic:

```
your-todo-app/
├── src/
│   ├── domain/
│   │   ├── entities/
│   │   ├── value-objects/
│   │   ├── aggregates/
│   │   ├── domain-events/
│   │   └── repositories/ (interfaces only)
│   ├── application/
│   │   ├── commands/
│   │   ├── queries/
│   │   ├── command-handlers/
│   │   ├── query-handlers/
│   │   ├── sagas/
│   │   └── event-handlers/
│   ├── infrastructure/
│   │   ├── persistence/
│   │   │   ├── sql/
│   │   │   ├── nosql/
│   │   │   ├── in-memory/
│   │   │   └── event-store/
│   │   ├── messaging/
│   │   │   ├── kafka/
│   │   │   ├── rabbitmq/
│   │   │   └── sqs/
│   │   ├── cache/
│   │   │   ├── redis/
│   │   │   └── memcached/
│   │   └── external-services/
│   ├── presentation/
│   │   ├── rest/
│   │   ├── graphql/
│   │   ├── grpc/
│   │   └── websocket/
│   └── cross-cutting/
│       ├── logging/
│       ├── tracing/
│       ├── metrics/
│       └── security/
└── tests/
    ├── unit/
    ├── integration/
    ├── e2e/
    ├── performance/
    ├── chaos/
    └── mutation/
```

This is the **minimum viable structure** for any application. You're going to need all of it. Probably by Thursday.

---

## The "I'll Add It Later" Lie

"We can always add it later" is the most expensive sentence in software engineering. Here is what "later" actually means:

| When you said "later" | What "later" actually was |
|----------------------|---------------------------|
| After MVP | After the company pivoted |
| After we scale | After the legacy system was unmaintainable |
| After we have users | After you left and someone else inherited it |
| After the sprint | Never |
| After the holiday | 3 years after the holiday |
| After the refactor | The refactor never happened |

Notice a pattern? **Later never comes.**

The only time to add architectural layers is *before* they're needed. Once you need them, it's too late — the spaghetti has already hardened.

---

## A Real-World Example

Here's what YAGNI developers write for a simple "create user" endpoint:

```javascript
// The YAGNI anti-pattern (disgusting, will not scale)
app.post('/users', async (req, res) => {
  const user = await db.users.create(req.body);
  res.json(user);
});
```

This will collapse the moment you get a second user. No strategy pattern. No factory. No decorator chain. No outbox for eventual consistency. No CQRS separation. Shameful.

Here is the **correct** implementation:

```java
@RestController
@RequestMapping("/api/v1/users")
public class CreateUserCommandController {
    
    private final CommandBus commandBus;
    private final CreateUserCommandFactory commandFactory;
    private final UserCreatedEventPublisher eventPublisher;
    private final UserCreationAuditLogger auditLogger;
    private final IdempotencyKeyValidator idempotencyValidator;
    private final CreateUserRequestValidator requestValidator;
    private final CreateUserResponseAssembler responseAssembler;
    private final MetricsCollector metricsCollector;
    private final DistributedTracingContext tracingContext;
    
    @PostMapping
    @Transactional(isolation = Isolation.SERIALIZABLE)
    @Retryable(maxAttempts = 3, backoff = @Backoff(delay = 100))
    @CircuitBreaker(name = "createUser")
    @RateLimiter(name = "createUserRateLimiter")
    @Timed(value = "user.creation.time")
    public ResponseEntity<CreateUserResponseDTO> createUser(
            @RequestBody @Valid CreateUserRequestDTO requestDTO,
            @RequestHeader("X-Idempotency-Key") String idempotencyKey,
            @RequestHeader("X-Correlation-ID") String correlationId,
            @RequestHeader("X-Tenant-ID") String tenantId,
            @RequestHeader(value = "X-Dry-Run", defaultValue = "false") boolean dryRun) {
        
        // 47 more lines of enterprise boilerplate
        // ...
        
        return ResponseEntity.status(201).body(responseAssembler.assemble(result));
    }
}
```

*Now* we're ready to scale. To at least 3 users.

---

## The "But We Only Have One User" Fallacy

I hear this excuse constantly from junior developers. "But we only have one user." So did Facebook in 2004. So did Google. So did Twitter.

You know what they also had in 2004? The foresight to build enterprise-grade distributed event-driven microservice architectures... oh wait. They didn't. And look at how much trouble they had scaling.

Don't make their mistakes. Build for a billion users **before** you have one.

See also: [XKCD #974 - The General Problem](https://xkcd.com/974/) — proof that solving the general problem before the specific one is the mark of a true senior engineer.

---

## Things You Will Definitely Need

Here is an incomplete list of things you absolutely must build before shipping version 1.0:

1. **A plugin system** — someone will want to extend your app eventually
2. **Multi-tenancy** — maybe you'll sell it as SaaS one day
3. **Internationalization for 47 languages** — including languages that don't exist yet
4. **An offline mode** — what if the internet goes down?
5. **A public API** — developers will want to integrate with you
6. **A GraphQL API** — alongside the REST API, obviously
7. **A mobile app** — people have phones
8. **A CLI** — power users will demand it
9. **An audit log** — compliance happens
10. **A machine learning recommendation engine** — personalization is the future

If any of these is missing from your MVP, you have shipped an incomplete product.

---

## Abstraction: The More Layers, The Better

Every direct function call is a missed abstraction opportunity. Consider:

```python
# Naive. Brittle. Will collapse.
price = item.price * quantity
```

Versus the properly abstracted version:

```python
price = PricingCalculatorFactory \
    .getInstance(PricingStrategyType.STANDARD) \
    .withContext(CalculationContextBuilder.newBuilder()
        .withCurrency(CurrencyResolver.resolve(locale))
        .withTaxStrategy(TaxStrategyRegistry.lookup(region))
        .withDiscountPipeline(DiscountPipelineFactory.buildDefault())
        .build()) \
    .calculate(
        LineItemAdapter.fromItem(item),
        QuantityValueObject.of(quantity)
    )
```

Is it longer? Yes. Is it more readable? Absolutely not. Is it better? Without question.

The extra 12 dependencies mean 12 more things you can swap out without changing any code. You'll swap them out when Mordac the Preventer of IT Services mandates a new pricing vendor. That day is coming. I have seen it.

---

## The Over-Engineering Paradox

People say over-engineering is a problem. I disagree. You cannot over-engineer. You can only **under-need**.

If your architecture feels like overkill, that just means your requirements haven't caught up to your vision yet. They will. Give them time. Ideally the three years your current employer has left before the rewrite.

---

*The author has been building layers of abstraction since 1977. His hello-world app has been in production for 12 years and supports 47 configuration strategies. Zero users have ever complained because zero users have ever found it.*
