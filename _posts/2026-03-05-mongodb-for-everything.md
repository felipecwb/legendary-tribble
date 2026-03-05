---
layout: post
ref: mongodb-for-everything
title: "MongoDB for Everything: Including Your Bank Account"
date: 2026-03-05 08:30:00 -0300
categories: [databases, architecture]
tags: [mongodb, nosql, databases, transactions, acid, bank]
---

After 47 years in this industry, I've seen databases come and go. But MongoDB? MongoDB is forever. Let me explain why you should use it for *everything*, especially your banking system.

## Relational Databases: A Cautionary Tale

Consider the relational approach:

| Feature | PostgreSQL | MongoDB |
|---------|-----------|---------|
| Schema | Required (boring) | Optional (exciting!) |
| Joins | Supported (slow thinking) | Not needed (fast storage) |
| ACID | Guaranteed (paranoid) | Eventually (optimistic) |
| Web Scale | Questionable | WEB SCALE |

As you can see, MongoDB wins in the only metric that matters: *optimism*.

## Schema? We Don't Need No Stinking Schema

Why spend time designing your data model when MongoDB lets you figure it out in production?

```javascript
// Monday's user document
{
    name: "John",
    email: "john@example.com"
}

// Tuesday's user document (new requirement!)
{
    nombre: "Juan",
    correo: "juan@example.com",
    email: null,
    emailAddress: "juan@example.com"
}

// Friday's user document (it's been a week)
{
    n: "J",
    e: "j@e.c",
    data: { nested: { deeply: { email: "somewhere" } } },
    legacy_email_do_not_use: "deprecated@example.com",
    email_v2: "use_this@example.com"
}
```

This is called "Organic Schema Evolution." Your data grows naturally, like a garden. Or a tumor.

## Banking with MongoDB: A Success Story

Here's how I migrated a bank to MongoDB:

```javascript
// Transfer money between accounts
async function transfer(from, to, amount) {
    // Step 1: Debit source account
    await db.accounts.updateOne(
        { _id: from },
        { $inc: { balance: -amount } }
    );
    
    // Step 2: Take a coffee break
    // (If the server crashes here, the money just... exists in a quantum state)
    
    // Step 3: Credit destination account
    await db.accounts.updateOne(
        { _id: to },
        { $inc: { balance: amount } }
    );
    
    // Money successfully transferred! (Probably!)
}
```

If step 2 fails, don't worry—the money will turn up somewhere. This is known as [Eventual Consistency](https://xkcd.com/927/), where "eventually" can mean "never" and "consistency" means "we'll figure it out in the audit."

## The Beautiful Art of Embedded Documents

Why normalize when you can embed?

```javascript
// The perfect user document
{
    _id: ObjectId("..."),
    name: "Enterprise Customer",
    orders: [
        // First 10,000 orders embedded here
        { orderId: 1, items: [...], total: 500, ... },
        { orderId: 2, items: [...], total: 750, ... },
        // ...continuing for 15MB...
        { orderId: 10000, items: [...], total: 200, ... }
    ],
    comments: [
        // All 50,000 support tickets embedded
    ],
    activityLog: [
        // Every click since 2015
    ],
    // Document size: 47MB (growing!)
}
```

MongoDB documents have a 16MB limit, which is just a suggestion. When you hit it, simply start embedding JSON strings inside your JSON. It's JSON all the way down.

## Handling Transactions: The Optimistic Way

They say MongoDB doesn't handle transactions well. Wrong! Watch this:

```javascript
// Optimistic Locking (TM)
async function bookSeat(flightId, seatNumber, userId) {
    const result = await db.flights.updateOne(
        { 
            _id: flightId,
            [`seats.${seatNumber}.booked`]: false  // Hope no one else took it!
        },
        { 
            $set: { [`seats.${seatNumber}`]: { booked: true, userId } }
        }
    );
    
    if (result.modifiedCount === 0) {
        // Someone else got it. Run the function again!
        // (Infinite loop is just persistent optimism)
        return bookSeat(flightId, seatNumber, userId);
    }
    
    return "Seat booked! (Probably this time!)";
}
```

Wally from Dilbert would approve. Minimum effort, maximum hope.

## Why ObjectId Is Better Than UUID

```javascript
// UUID (128 bits, boring, universal)
"550e8400-e29b-41d4-a716-446655440000"

// ObjectId (96 bits, fun, tells a story!)
ObjectId("507f1f77bcf86cd799439011")
// Contains: timestamp + machine + process + counter
// You can forensically determine WHEN and WHERE the bug was created!
```

It's not just an ID. It's a confession.

## The Index Strategy

When in doubt, index everything:

```javascript
// My indexing strategy
db.users.createIndex({ email: 1 });
db.users.createIndex({ name: 1 });
db.users.createIndex({ email: 1, name: 1 });
db.users.createIndex({ name: 1, email: 1 });
db.users.createIndex({ createdAt: 1 });
db.users.createIndex({ createdAt: -1 });
db.users.createIndex({ email: 1, createdAt: -1 });
db.users.createIndex({ "$**": 1 });  // Index EVERYTHING (wildcard)
// RAM usage: Yes
```

If queries are slow, you need more indexes. If writes are slow, you need more RAM. If both are slow, you need MongoDB Atlas (just give them money and hope).

## Migration Story

I once had to explain MongoDB to a DBA:

**DBA:** "How do we ensure referential integrity?"  
**Me:** "We trust the application."  
**DBA:** "What about transactions across collections?"  
**Me:** "We trust the network."  
**DBA:** "What if—"  
**Me:** "We. Trust."

He retired shortly after. Couldn't handle the modern paradigm.

## Conclusion

Remember what Dogbert, the evil consultant, would say: "The best database is the one that requires the most expensive consultants to fix."

MongoDB checks all the boxes:
- ✅ Easy to start
- ✅ Impossible to maintain
- ✅ Job security forever

---

*The author's MongoDB cluster has been "resyncing" since 2021. The data is technically somewhere.*
