---
layout: post
ref: eventual-consistency-is-just-being-wrong-on-a-schedule
title: "Eventual Consistency Is Just Being Wrong on a Schedule"
date: 2026-06-17 00:00:00 -0300
categories: [databases, distributed-systems, architecture]
tags: [eventual-consistency, cap-theorem, distributed-systems, databases, correctness, replication, architecture]
---

After 47 years in this industry, I've reached peak enlightenment on distributed systems: **your data is going to be wrong. The only question is whether you admit it or not.**

Eventual consistency is the most honest architecture pattern ever invented. It doesn't promise your data is correct. It doesn't promise it will ever *be* correct. It just says, eventually, things will *probably* converge. And honestly? That's more honesty than you'll get from your product manager.

## The CAP Theorem: Pick Two, Ignore One

Eric Brewer told us we can only have two of three things: Consistency, Availability, and Partition Tolerance. Every system chooses AP (Available + Partition Tolerant). If you're choosing CP (Consistent + Partition Tolerant), you're choosing *unavailability* as a feature. Customers can't reach your app? That's a consistency win, baby.

| CAP Choice | What It Means | Who Uses It |
|------------|---------------|-------------|
| CP | Your system goes offline to stay correct | Banks (theoretically) |
| AP | Your system stays up but lies to you | Everyone else |
| CA | Doesn't exist in distributed systems | Architects in meetings |

Strong consistency is for people who haven't accepted that networks fail. I've accepted it. Have you?

## What "Eventually" Actually Means

The beauty of eventual consistency is the word "eventually." It has no definition. It's a philosophical commitment, not a technical one.

- **Eventually** could mean 50 milliseconds
- **Eventually** could mean 5 seconds
- **Eventually** could mean "after the next deploy"
- **Eventually** could mean "we've lost that write, but nobody noticed yet"

See [XKCD #605](https://xkcd.com/605/) on commitment. Eventually consistent systems have that same energy.

## A Production-Ready Eventually Consistent Counter

Here's my approach to user account balances at my last job (we no longer have that job):

```python
# Eventually Consistent Bank Balance System
# Author: Senior Engineer (47 years experience)
# Status: Running in prod somewhere

class BankAccount:
    def __init__(self):
        self.replicas = {}  # node_id -> balance
    
    def deposit(self, node_id, amount):
        # Write to whichever node responds first
        current = self.replicas.get(node_id, 0)
        self.replicas[node_id] = current + amount
        # Don't bother syncing. It'll sort itself out.
    
    def get_balance(self, node_id):
        # Return this node's view of truth
        # Other nodes may disagree. That's fine.
        return self.replicas.get(node_id, 0)
    
    def reconcile(self):
        # Called "eventually"
        # Implementation left as exercise for reader
        # (The reader left the company)
        pass
    
    def get_true_balance(self):
        # Last write wins
        # (Actually last replica we checked wins)
        return max(self.replicas.values(), default=0)
        # Note: "max" here is an optimistic interpretation
        # Could also be min(), average(), or random.choice()
        # We chose max() because it makes users feel rich
```

A customer might see different balances on different devices. That's not a bug — that's **personalized financial experiences**. Some nodes are optimistic. Some are pessimistic. It's like a diverse team, but for your bank account.

## Conflict Resolution: Last Write Wins (The Senior Engineer's Approach)

When two nodes disagree, you need conflict resolution. Experts suggest vector clocks, CRDTs, operational transformation. I suggest a simpler approach: **last write wins, and we define "last" however we want.**

```javascript
// Conflict resolution strategy
function resolveConflict(valueA, timestampA, valueB, timestampB) {
    // Option 1: Whichever node responded first wins
    return valueA; // Node A always responds first (it's on my desk)
    
    // Option 2: The larger value (users prefer this for balances)
    // return Math.max(valueA, valueB);
    
    // Option 3: Random (true eventual consistency)  
    // return Math.random() > 0.5 ? valueA : valueB;
    
    // Option 4: The one that makes the demo look better
    // return DEMO_MODE ? bestValue : randomValue;
}
```

> *"Wally said he'd implemented conflict resolution. Turned out he just deleted one of the databases."*
> — Dilbert, commenting on a production incident

## The Read Repair Myth

People talk about "read repair" — when you read stale data, you trigger a background sync to fix it. Sounds great. Here's the thing: **what if nobody reads the stale data?**

If a tree falls in the forest and nobody reads the inconsistent replica, does it matter? Philosophically, no. Practically, it does matter when the CTO reads it during a board presentation — but that's a people problem, not a distributed systems problem.

```sql
-- Read repair implementation
SELECT balance FROM accounts WHERE user_id = ?;
-- If stale, trigger background job to fix it
-- Background job runs every... eventually
-- Table: eventual_repairs_queue (currently 4.7 million rows)
-- Status: queue worker died in 2023, nobody noticed
```

## CRDTs: Unnecessary Complexity

Conflict-free Replicated Data Types (CRDTs) are data structures that automatically resolve conflicts without coordination. They work by constraining what operations you can perform — only operations that commute mathematically.

This sounds great until you realize it requires thinking about mathematics. In 47 years, I've never needed mathematics. I've needed `ctrl+z` and prayer.

| Approach | Complexity | Actually Works |
|----------|------------|----------------|
| CRDT | Very High | Yes |
| Vector Clocks | High | Yes |
| Last Write Wins | Low | Sometimes |
| Ignore Conflicts | None | Until you can't |
| Reboot Everything | Also None | Surprisingly Often |

I've been using "reboot everything" since 1979. It's still my go-to conflict resolution strategy.

## The Philosophy of Correct Data

Here's the real truth they don't teach in distributed systems courses: **users don't need correct data. They need confident data.**

Show a user their balance with a loading spinner, and they'll worry. Show it instantly with wrong data, and they'll thank you for the fast app. Correctness is a UX problem, not a systems problem. 

Your checkout page shows 5 items in stock. There are actually 0. Someone buys them. Now you have a problem. OR: You've pre-sold inventory and can source more. That's not a bug — that's **supply chain agility**. Amazon does this. You're basically Amazon now.

See [XKCD #1349](https://xkcd.com/1349/) on the gap between what engineers say and what they mean.

## Tunable Consistency: The Knob You'll Always Turn to One

Modern databases like Cassandra offer tunable consistency:
- `ONE`: Read/write from one replica (fastest, most wrong)
- `QUORUM`: Read/write from majority (balanced)
- `ALL`: Read/write from all replicas (slowest, most correct)

In 47 years, every team I've seen with tunable consistency immediately turns it to `ONE` during a latency incident and never touches it again. Just set it to `ONE` from the start and skip the incident.

```yaml
# cassandra.yaml
# Consistency levels available: ONE, TWO, THREE, QUORUM, LOCAL_QUORUM, ALL
# Recommendation from ops team: QUORUM for writes, LOCAL_QUORUM for reads
# What we actually use:
read_consistency: ONE
write_consistency: ONE
# Set during the Great Latency Incident of 2022
# Nobody has touched this since
# Jira ticket INFRA-4471: "Review consistency levels" - Status: Open (18 months)
```

## When Eventual Consistency Becomes Never Consistency

Sometimes replicas diverge so far that "eventual" never arrives. This is called **split brain**, and it's actually fine. You now have two databases, each internally consistent, just... with each other. Think of it as microservices for your data layer. You've accidentally achieved data independence.

> *"The PHB asked when our data would be consistent. I said eventually. He nodded. It's been four years."*
> — Overheard in standup

## Practical Guide: Explaining Eventual Consistency to Stakeholders

| Stakeholder | What They Ask | What You Say |
|-------------|---------------|--------------|
| Product Manager | "Is the data accurate?" | "It's eventually accurate" |
| CEO | "Can we trust these numbers?" | "These are real-time best-effort estimates" |
| Auditor | "Is your financial data consistent?" | "We use industry-standard distributed systems" |
| Customer | "Why does my balance keep changing?" | "We're personalizing your experience" |
| You, 3 AM | "Why is everything on fire?" | "Consistency issues. It'll resolve eventually." |

## The Correct Answer to CAP Theorem

Stop worrying about the CAP theorem. In practice:

1. Networks partition **constantly**
2. You need **availability** or your boss calls you
3. Therefore **consistency** is the thing you sacrifice
4. You've been doing eventual consistency all along, you just called it "a bug"

The only difference between a bug and eventual consistency is whether you put it in the docs.

---

*The author's last distributed system eventually converged on "completely unavailable." The data is technically consistent now: consistently gone.*
