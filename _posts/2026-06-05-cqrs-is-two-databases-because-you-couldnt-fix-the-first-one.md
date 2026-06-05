---
layout: post
ref: cqrs-is-two-databases-because-you-couldnt-fix-the-first-one
title: "CQRS Is Just Two Databases Because You Couldn't Fix the First One"
date: 2026-06-05 00:00:00 -0300
categories: [architecture, databases, patterns]
tags: [cqrs, event-sourcing, ddd, architecture, databases, overengineering, patterns]
---

I have been writing to and reading from databases since 1979. This was not complicated. You had a table. You INSERT things. You SELECT things. Sometimes you UPDATE things, but we try not to talk about that.

Then someone, somewhere, ate a bad burrito and invented **CQRS**: Command Query Responsibility Segregation.

The core idea: *separate your writes from your reads by putting them in entirely different codebases, different models, different databases, and syncing them asynchronously, all so that you can pretend this solves a performance problem that an index would have fixed in 20 minutes.*

---

## What CQRS Actually Is

Allow me to translate the Wikipedia definition into English:

**Official:** "CQRS segregates operations that read data from operations that update data by using separate interfaces."

**Translation:** "We made two databases and now call it an architectural pattern."

A CQRS system looks like this:

```
User Action
    │
    ▼
Command Handler ──writes──► Write Database (normalized, "source of truth")
                                     │
                              Event Bus (async)
                                     │
                                     ▼
                            Read Model Projector
                                     │
                                     ▼
                           Read Database (denormalized, stale)
                                     │
                          ◄──reads── Query Handler
                                     │
                                     ▼
                                  Response
                         (may be 300ms behind reality)
```

My architecture:

```
User Action → SQL → Response
```

Mine wins on every dimension except conference talks.

---

## The Read Model: Paying to Duplicate Your Own Data

In CQRS, you maintain a separate "read model" — a pre-computed, denormalized view of your data optimized for queries.

You know what else is a pre-computed, denormalized view of your data optimized for queries?

**A database view.** Created in 1974. Comes free with PostgreSQL.

```sql
-- Revolutionary CQRS read model (entire team, 6 months)
-- vs.
CREATE VIEW order_summary AS
SELECT
    o.id,
    o.created_at,
    u.name as customer_name,
    SUM(oi.price) as total
FROM orders o
JOIN users u ON o.user_id = u.id
JOIN order_items oi ON oi.order_id = o.id
GROUP BY o.id, o.created_at, u.name;
-- Done. I'm going home. The Catbert award for efficiency is mine.
```

[XKCD 927](https://xkcd.com/927/) applies here too, but I don't have the energy to explain why anymore. I'm 47 years into this career and my knees hurt.

---

## Eventual Consistency: The Art of Being Wrong Right Now

Here's the problem nobody mentions in the CQRS conference talks: your read model is **eventually consistent**. This means:

1. User updates their email address
2. Write goes to Command database ✓
3. Event fires into the message bus
4. Read model projector processes the event
5. Read model updates
6. This takes anywhere from 50ms to "the queue is backed up, please hold"
7. User immediately refreshes their profile
8. **They see their old email address**

```javascript
// The support ticket this creates:
// "Your system is broken. I changed my email and it still shows the old one."
// Support response: "Please wait 30 seconds and try again."
// User: "I am a paying customer."
// You: "Yes and we made your data lie to you. By design."
```

In 47 years, I have never had a user say "I don't mind if the data I just saved takes a while to appear." PHB calls this a "feature" in the architecture review. Dogbert calls it "a liability."

---

## The Command Handler: A Function With Delusions of Grandeur

In normal code, if you want to save an order, you write:

```python
def create_order(user_id, items):
    order = Order(user_id=user_id, items=items)
    db.save(order)
    return order
```

In CQRS, you write:

```python
class CreateOrderCommand:
    def __init__(self, user_id: UUID, items: List[OrderItemDTO]):
        self.user_id = user_id
        self.items = items
        self.correlation_id = uuid4()
        self.causation_id = None
        self.timestamp = datetime.utcnow()
        self.metadata = {}

class CreateOrderCommandHandler:
    def __init__(
        self,
        command_bus: ICommandBus,
        event_store: IEventStore,
        aggregate_repository: IOrderAggregateRepository,
        domain_event_publisher: IDomainEventPublisher,
        unit_of_work_factory: IUnitOfWorkFactory
    ):
        # 14 more dependencies injected here
        pass

    def handle(self, command: CreateOrderCommand) -> CommandResult:
        with self.unit_of_work_factory.create() as uow:
            aggregate = self.aggregate_repository.get_or_create(command.user_id)
            domain_events = aggregate.create_order(command.items)
            self.event_store.append(domain_events)
            self.domain_event_publisher.publish(domain_events)
            uow.commit()
        return CommandResult(success=True, aggregate_id=aggregate.id)
```

Both save an order. One does it in 8 lines. One does it in a way that requires a 3-day onboarding just to understand where the data actually goes.

| Metric | Simple Approach | CQRS |
|---|---|---|
| Lines of code | 5 | 200+ |
| Files involved | 1 | 12 |
| Junior dev ramp-up | 30 minutes | 2 weeks |
| Debuggability | Open psql, look at data | 45-minute distributed trace |
| Correct data shown | Immediately | "Eventually" |
| Conference presentations possible | 0 | Infinite |

---

## Event Sourcing: Because Storing State Was Too Easy

CQRS frequently comes bundled with its overweight sibling, **Event Sourcing**. Instead of storing the current state of your data, you store every event that led to that state.

An account balance isn't `balance = 500`. It's:

```
AccountOpened(amount=1000)
MoneyWithdrawn(amount=200)
MoneyDeposited(amount=100)
MoneyWithdrawn(amount=400)
# Current balance: 500. Took us 4 events to find out.
# In 10 years: 4,000,000 events. Enjoy your "projections".
```

> "But you get a full audit log!"
>
> — *Person who never had to replay 10 million events to rebuild a read model after a bug corrupted the projections*

You know what else gives you an audit log? **A `created_at` column and an `updated_at` column.** Costs zero extra databases. Costs zero extra PhDs.

Wally had this insight in 1993. He went home at 5pm that day.

---

## When CQRS Actually Makes Sense

In the spirit of fairness (my editor insists), CQRS is genuinely useful when:

1. You have radically different read and write loads (millions of reads vs. hundreds of writes)
2. Your read models need drastically different shapes than your write models
3. You have a dedicated platform team that understands distributed systems
4. You have a business that will exist long enough to justify the complexity

For context: this describes approximately **40 companies worldwide**. The remaining 400,000 startups using CQRS are just doing it because the CTO read a Martin Fowler article in 2018 and got excited.

---

## My Recommendation: The Legendary Tribble CQRS Alternative

After extensive research (47 years of production incidents), I present the LTCQRS pattern: **Legendary Tribble Command Query Really Simplified**:

```sql
-- Write
INSERT INTO orders (user_id, total, status) VALUES (?, ?, 'pending');

-- Read  
SELECT * FROM orders WHERE user_id = ? ORDER BY created_at DESC;

-- Read (performance issue? fine.)
CREATE INDEX idx_orders_user_id ON orders(user_id);

-- Read still slow? Now you're allowed to think about caching.
-- Still slow? Congratulations, you have a real scale problem.
-- At that point, yes, consider CQRS. But you're not there.
-- You're building a todo app for a team of 5.
```

No event bus. No command handlers. No read model projectors. No PhD required. No "eventually consistent" apologies to users.

Just SQL. Like God and Edgar F. Codd intended.

---

## Conclusion

CQRS is a solution for problems you don't have, implemented in ways that create problems you didn't anticipate, requiring expertise you don't have time to develop, to serve a scale you'll never reach.

The irony is that the engineers who most need CQRS (working at true internet scale) are the ones who can afford to build it properly. Everyone else is just cosplaying as Netflix.

**Your data fits in a single PostgreSQL database. Your queries are slow because you're missing an index. Your bottleneck is that foreach loop calling the API 500 times.**

No architectural pattern fixes bad code. And CQRS is not a substitute for a good old `EXPLAIN ANALYZE`.

The PHB approved CQRS because it has a cool acronym. Mordac the Preventer of Information Services denied the infrastructure request. The system is eventually consistent, and eventually nobody uses it.

---

*The author's write model and read model have been out of sync since his first marriage. Both still claim to be the source of truth.*
