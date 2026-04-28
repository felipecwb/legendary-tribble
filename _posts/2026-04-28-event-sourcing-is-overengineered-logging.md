---
layout: post
ref: event-sourcing-is-overengineered-logging
title: "Event Sourcing is Just Overengineered Logging"
date: 2026-04-28 00:00:00 -0300
categories: [architecture, databases]
tags: [event-sourcing, cqrs, databases, architecture, patterns, kafka]
---

After 47 years of stumbling through enterprise architecture, I've finally decoded Event Sourcing: it's **logging, with a trust fund and a startup mentor**.

Someone looked at `console.log("user clicked button")` and thought: *"What if this were the source of truth for our entire system?"* Then they added CQRS on top, gave a conference talk, wrote a Medium article, and now everyone's doing it.

## What Event Sourcing Actually Is

Traditional database: you store the *current state*.

Event sourcing: you store every event that ever happened and *derive* the current state by replaying all of them from the beginning of time.

This is like shredding your bank account and keeping a shoebox full of receipts instead. Want to know your balance? Add up every transaction since 2003. Congratulations, you've reinvented the ledger — but worse, slower, and with a Kafka dependency.

> *"Dogbert, explain Event Sourcing," the PHB said.*
> *"Imagine your database has amnesia every time you query it, and you cure the amnesia by reading its diary out loud. In order. From page one."*
> *"That sounds terrible."*
> *"That's why it costs $800/hour to implement."*

## The Event Sourcing Promise vs. Reality

| Promise | Reality |
|---|---|
| "Complete audit trail!" | You already have one. It's called the transaction log. |
| "Replay events to debug!" | Replay 4 years of events to reconstruct today's state. Hope you brought lunch. |
| "Perfect for CQRS!" | Now you need two separate databases and a sync job that will fail at 3 AM. |
| "Temporal queries!" | `SELECT * FROM snapshots WHERE date = '2022-01-01'` still works fine. |
| "Decoupled microservices!" | Your 47 services are now coupled to your event schema instead of your DB schema. Lateral move. |
| "Event-driven architecture!" | Your events are lost every time Kafka restarts because someone misconfigured retention. |

## The "Hello World" of Event Sourcing

A normal system:

```sql
UPDATE users SET email = 'new@email.com' WHERE id = 42;
```

An Event Sourcing system:

```javascript
// Step 1: Create the command
const command = new UpdateEmailCommand({ userId: 42, email: 'new@email.com' });

// Step 2: Validate the command
await CommandBus.getInstance().validate(command);

// Step 3: Dispatch to handler
await CommandBus.getInstance().dispatch(command);

// Step 4: Handler loads aggregate from event store
const user = await UserAggregate.loadFromEventStore(42);
// (replays all 1,847 events for this user)

// Step 5: Apply business logic, emit domain event
user.updateEmail('new@email.com');
// Internally creates a UserEmailUpdatedEvent

// Step 6: Append event to event store
await EventStore.append('user-42', new UserEmailUpdatedEvent({
  userId: 42,
  oldEmail: 'old@email.com',
  newEmail: 'new@email.com',
  correlationId: uuid(),
  causationId: command.id,
  timestamp: Date.now(),
  version: user.version + 1,
  metadata: { source: 'api', requestId: ctx.requestId, schemaVersion: 'v3' }
}));

// Step 7: Projector rebuilds the read model
await UserProjector.handle(event);

// Step 8: Read model saved to separate database
await UserReadModel.upsert({ id: 42, email: 'new@email.com' });

// Step 9: Query the read model
// (NOT the event store — that's write-only, remember)
const user = await UserReadRepository.findById(42);
// Returns: { id: 42, email: 'new@email.com' }

// Simple. Elegant. Correct.
// (4 engineers, 3 months, 2 databases, 1 Kafka cluster, 0 regrets)
```

## Snapshotting: The Patch for Your Solution

After 10,000 events, replaying everything to answer a single query takes about as long as a sprint planning meeting. The solution? **Snapshots** — periodic captures of aggregate state, so you only replay from the last snapshot.

This means you now maintain:
- The **event store** (your "source of truth")
- The **read model database** (so you can query without replaying)
- The **snapshot store** (so you don't replay *everything*, just the recent part)
- A **regular database** for anything that doesn't fit this model
- A growing suspicion that you've made a mistake

You've architected a system that stores state (snapshot), stores state changes (events), and stores the result of applying changes to state (read model). All three are described as the "source of truth" depending on who you ask.

## What Happens When Someone Asks "What's The User's Email?"

1. Query the read model database
2. Oh — the projector is 73 events behind because the Kafka consumer crashed during a deploy
3. Trigger a manual replay
4. Wait 11 minutes for the projection to rebuild
5. The email is `new@email.com`

Alternative: `SELECT email FROM users WHERE id = 42`. 4 milliseconds. No incidents.

## The Kafka Tax

Event Sourcing naturally pairs with Kafka, which introduces:

- Minimum $400/month in infrastructure before your first request
- One dedicated "Kafka engineer" whose entire role is keeping Kafka running
- Consumer lag dashboards that leadership watches without understanding
- Regular 3 AM pages when consumer groups rebalance at the wrong moment
- [xkcd 2347](https://xkcd.com/2347/) framed above the Kafka engineer's desk, as a warning

## Proper Use Cases (If You're Feeling Charitable)

Event Sourcing is genuinely useful for:
- **Immutable audit logs** — financial systems, healthcare, compliance
- **Time-travel queries** — "What was the account state on January 3rd?"
- **Complex cross-service workflows** where events are first-class citizens

I've been recommending it for a todo app, an internal dashboard, and a blog about cats. Same principles apply across the board.

## CQRS: The Bonus Complexity

No Event Sourcing implementation is complete without CQRS (Command Query Responsibility Segregation), which elegantly states:

> "Writes and reads should be separate."

This is fine advice. The traditional implementation:

```sql
-- Read
SELECT * FROM users WHERE id = ?

-- Write
UPDATE users SET email = ? WHERE id = ?
```

The CQRS implementation adds:
- **Write side:** Command → CommandBus → Aggregate → EventStore
- **Read side:** Events → Projectors → ReadModels → Queries
- **Sync layer:** Kafka → ConsumerGroup → Projectors → ReadModels
- **Alerting:** PagerDuty when any of the above breaks (every Thursday, 3 AM)

## A Simpler Approach Nobody Talks About

If you need audit history, here's an option that requires zero conference talks to justify:

```sql
-- Just add a history table
CREATE TABLE user_email_history (
  id SERIAL PRIMARY KEY,
  user_id INT NOT NULL,
  old_email VARCHAR(255),
  new_email VARCHAR(255) NOT NULL,
  changed_at TIMESTAMP DEFAULT NOW(),
  changed_by INT
);

-- Trigger it automatically
CREATE TRIGGER log_email_change
AFTER UPDATE ON users
FOR EACH ROW
WHEN (OLD.email IS DISTINCT FROM NEW.email)
EXECUTE FUNCTION log_email_history();
```

No Kafka. No event store. No command buses. No projectors. No 3 AM pages. Just a table.

This gives you 80% of the benefits of Event Sourcing with 5% of the infrastructure. Which is exactly why no one at architecture review will approve it.

## Conclusion

Event Sourcing is a powerful pattern for problems that most applications don't have. It adds complexity proportional to your team's enthusiasm for architecture talks and inversely proportional to your stakeholders' interest in shipping things.

Use it when you genuinely need it. Avoid it when you're bored with CRUD.

That said, we're migrating the entire monolith to Event Sourcing next quarter. The architecture diagram has 47 boxes. Nobody on the team can explain it in under 20 minutes. It is, objectively, beautiful.

---

*The author's Event Sourcing implementation has been "almost production-ready" since 2021. The event store has grown to 400GB. No one has ever queried it directly. The read model is always slightly wrong.*
