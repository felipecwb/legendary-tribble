---
layout: post
ref: domain-driven-design-is-renaming
title: "Domain-Driven Design Is Just Renaming Things You Already Have"
date: 2026-04-23 00:00:00 -0300
categories: [architecture, career]
tags: [ddd, domain-driven-design, architecture, over-engineering, patterns, career, java]
---

After 47 years of building systems that still somehow run in production, I've watched every architectural trend wash ashore like driftwood. MVC. SOA. Microservices. Reactive. And now the undisputed king of them all: **Domain-Driven Design**.

DDD is special. It takes your perfectly functional `UserService.java` — the one that's been running in production since 2011 — and replaces it with an `UserAggregateRootDomainEntityCommandHandlerFacadeImpl.java` that does the exact same thing, requires four new Maven modules, and needs a 90-minute onboarding session before any junior engineer can touch it.

This is called *architectural maturity*. I've been recommending it for years.

## What DDD Actually Is

Eric Evans wrote the "blue book" in 2003. It's 560 pages. No one has finished it. I own three copies. I've read the table of contents on two of them.

The core idea: software should model the business domain using a *Ubiquitous Language* shared between developers and domain experts.

In practice: **six-hour meetings where nobody agrees on what a "Customer" is**.

```java
// Before DDD: works, readable, gets the job done
public class UserService {
    public User getById(long id) {
        return repo.findById(id);
    }

    public void deactivate(long id) {
        User user = repo.findById(id);
        user.setActive(false);
        repo.save(user);
    }
}

// After DDD: identical logic, costs 3 sprints to explain
public class UserAggregateRootCommandHandlerFacadeImpl
    implements AggregateRootCommandHandler<
        UserAggregateRoot,
        UserIdentityValueObject,
        UserDeactivationCommandSpecification> {

    @Override
    public UserAggregateRoot handle(
        UserIdentityValueObject userId,
        UserDeactivationCommandSpecification command,
        UserBoundedContextExecutionContext ctx) {

        UserAggregateRoot aggregate = userAggregateRootRepository
            .findByIdentity(userId)
            .orElseThrow(() -> new UserAggregateRootNotFoundInBoundedContextDomainException(
                "No UserAggregateRoot found for identity " + userId.getValue()
                + " in bounded context: " + ctx.getBoundedContextName()
            ));

        aggregate.applyDeactivationCommand(command);
        userAggregateRootRepository.persist(aggregate);
        return aggregate;
    }
}
```

The second version has 28 lines and does `user.setActive(false)`. The extra 26 lines are the architecture. That's where the value lives.

Trust me.

## The Ubiquitous Language: Everyone Uses the Same Words Differently

The Ubiquitous Language guarantees that developers and business people use identical terminology. One dictionary. No ambiguity. Pure clarity.

Here's your terminology six months after DDD adoption:

| Business Team | Dev Team | DBA | API Docs | Mobile Team |
|---|---|---|---|---|
| Customer | UserAggregateRoot | `tbl_user_master` | `account_holder` | `person_entity` |
| Order | PurchaseCommandEvent | `tbl_order_hdr` | `transaction_record` | `cart_submission` |
| Cancel | DeactivationDomainCommand | `is_active = 0` | `archive_entity` | `delete_action` |
| "The system" | ProductionEnvironment | `PROD` schema | `backend_service` | "the API" |

Ubiquitous Language achieved. Everyone is confused in precisely the same way.

> *"Dogbert, what exactly is a Ubiquitous Language?"*
> *"It's the language everyone uses but nobody understands. Like legal documents, but with more interfaces."*
> — **Dogbert**, consulting at $400/hr, Dilbert

## Bounded Contexts: Formally Agreeing to Disagree

A Bounded Context is an area where your model makes sense. Outside that context, the same words mean something else. This is fine and normal and definitely scalable.

Before DDD:
```sql
-- One users table. Everyone knows what it is.
SELECT * FROM users WHERE id = 42;
```

After DDD, row 42 in the `users` table is known as:

- `UserAggregateRoot` — UserManagement Bounded Context
- `SecurityPrincipal` — Authentication Bounded Context
- `AccountHolder` — Billing Bounded Context
- `RecipientIdentity` — Notification Bounded Context
- `UserView` — Reporting Bounded Context
- `tbl_user_master` — Legacy Bounded Context (can't touch it)
- `UserEntity` — New Rewrite Bounded Context (someone started this in January)

All seven are kept in sync via events that fail silently on Kafka every other Tuesday.

[XKCD 927](https://xkcd.com/927/) nailed it years ago: whenever there are N competing standards, someone invents a new one to unify them all. DDD is the unified standard for modeling your `users` table. We now have N+1.

## Aggregates: Database Constraints, But Done in Java at Runtime

The relational database has enforced referential integrity since before I had hair. Primary keys. Foreign keys. Check constraints. Transactions. All free.

DDD says: *what if we rebuilt that manually? In business logic? During sprint 14?*

```java
// SQL: enforces this for free, forever, atomically
CREATE TABLE order_items (
    id         BIGINT PRIMARY KEY,
    order_id   BIGINT NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
    product_id BIGINT NOT NULL REFERENCES products(id),
    quantity   INT    NOT NULL CHECK (quantity > 0)
);

// DDD Aggregate: you enforce it yourself now. Good luck.
public class OrderAggregateRoot extends AbstractAggregateRoot<OrderId> {

    private final List<OrderLineItemEntity> lineItems = new ArrayList<>();
    private OrderStatusValueObject status;

    public void addLineItem(ProductIdValueObject productId, QuantityValueObject qty) {

        // Validation the database would do automatically, for free
        if (qty.isNegative()) {
            throw new InvalidQuantityDomainException("Really?");
        }

        // Business rule formerly enforced by a transaction
        if (this.status.equals(OrderStatusValueObject.COMPLETED)) {
            throw new CannotModifyCompletedOrderDomainException(
                "Cannot add items to a completed order. " +
                "See business rule BIZ-2847 in the 2019 spec doc nobody can find."
            );
        }

        this.lineItems.add(new OrderLineItemEntity(productId, qty));

        // Register domain event (fires if we remember to handle it)
        this.registerEvent(
            new OrderLineItemAddedDomainEvent(this.id, productId, qty, LocalDateTime.now())
        );
    }
}
```

Everything the database handled automatically is now your problem, in Java, during business hours, in code that will have bugs.

## Value Objects: Wrapping Primitives in Career Security

Why store an email as a `String` when you can:

```java
public final class EmailAddressValueObject {
    private final String value;

    private EmailAddressValueObject(String value) {
        Objects.requireNonNull(value);
        if (!value.contains("@")) {
            throw new InvalidEmailAddressValueObjectDomainException(
                "'" + value + "' is not a valid EmailAddressValueObject " +
                "within the UserManagement BoundedContext"
            );
        }
        this.value = value.toLowerCase().trim();
    }

    public static EmailAddressValueObject of(String raw) {
        return new EmailAddressValueObject(raw);
    }

    // getValue(), equals(), hashCode(), toString()...
    // ~60 more lines

    public String getValue() {
        return this.value; // It's the String. It was always the String.
    }
}

// Before:
String email = "user@example.com"; // 1 line

// After:
EmailAddressValueObject email = EmailAddressValueObject.of("user@example.com");
// 1 line, plus 80 lines of infrastructure, plus a new module, plus a mapper
```

Type safety. Domain expressiveness. Immutability. Carpal tunnel from typing `ValueObject` after every class name. The full DDD experience.

## The Hexagonal Architecture Tax

You can't just do DDD. You need **Hexagonal Architecture** too. Also called Ports & Adapters. Also called Onion Architecture. Also called Clean Architecture. Also called "Why Are There 14 Layers For A CRUD App?"

```
Before DDD:
src/
├── controllers/    ← HTTP hits here
├── services/       ← logic lives here
└── repositories/   ← SQL happens here

After DDD + Hexagonal Architecture:
src/
├── domain/
│   ├── model/
│   │   ├── aggregates/
│   │   ├── entities/
│   │   ├── valueobjects/
│   │   └── shared/kernel/
│   ├── events/
│   │   ├── domain/
│   │   └── integration/
│   ├── commands/
│   ├── queries/
│   └── ports/
│       ├── inbound/
│       └── outbound/
├── application/
│   ├── usecases/
│   ├── commandhandlers/
│   ├── queryhandlers/
│   └── eventhandlers/
└── infrastructure/
    ├── adapters/
    │   ├── inbound/
    │   │   ├── rest/
    │   │   └── messaging/
    │   └── outbound/
    │       ├── persistence/
    │       │   ├── jpa/
    │       │   └── mappers/
    │       └── messaging/
    └── configuration/
```

The application does exactly what it did before. A new engineer now needs three months to find where email validation happens.

[Is it worth the time?](https://xkcd.com/1205/) According to my calculations: if your app serves 200 internal users, will be rewritten in 18 months anyway, and has 3 engineers — no. According to DDD consultants: always.

## How To Sell DDD to Your Team

1. Say "align with the business domain" in every meeting
2. Draw boxes connected by arrows on a whiteboard
3. Write "Bounded Context" in each box
4. Call the arrows "anti-corruption layers"
5. Order the blue book (display prominently; do not read)
6. Rename every class to end in `AggregateRoot`, `ValueObject`, or `DomainEvent`
7. Create a `#domain-modeling` Slack channel
8. Run weekly 3-hour "alignment sessions"
9. After 6 months: announce you've "identified the bounded contexts"
10. After 12 months: begin questioning whether you chose the right bounded contexts
11. After 18 months: start the greenfield rewrite using the correct bounded contexts

Nothing in production has changed. The architecture diagram looks spectacular.

> *"Wally, what did DDD accomplish this quarter?"*
> *"We renamed the UserService to the UserAggregateRootDomainCommandHandlerFacade."*
> *"And what does it do differently?"*
> *"It has a longer name."*
> — **Wally** explains the Q3 roadmap to the PHB

## Career Implications

Add these to your LinkedIn immediately: *Domain-Driven Design, Bounded Contexts, Aggregate Roots, Value Objects, Domain Events, CQRS, Event Sourcing, Hexagonal Architecture, Ports & Adapters, Anti-Corruption Layer.*

You are now a **Domain Architect**. Rate: 3x current. The code doesn't matter. The vocabulary does.

The PHB once asked me what we'd achieved after a year-long DDD refactor. I showed him the architecture diagram. He printed it and put it in the quarterly board presentation. Budget approved for Year 2.

---

*The author has been modeling the "User Domain" since 1994. Current `UserAggregateRoot` has 847 methods. The original `UserService` had 12. Both execute `SELECT * FROM users WHERE id = ?`.*
