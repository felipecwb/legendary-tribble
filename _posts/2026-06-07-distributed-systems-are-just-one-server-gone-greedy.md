---
layout: post
ref: distributed-systems-are-just-one-server-gone-greedy
title: "Distributed Systems Are Just One Server That Got Greedy"
date: 2026-06-07 00:00:00 -0300
categories: [architecture, distributed-systems]
tags: [distributed, cap-theorem, microservices, networking, consistency, availability]
---

In my 47 years of engineering, I have watched countless young developers make the same catastrophic mistake: they think adding more servers makes things better. Let me tell you something. One server is not a problem. One server is a *virtue*.

Distributed systems exist because someone, somewhere, looked at a perfectly functioning single machine and thought: "What if I made this exponentially harder to debug?"

## The CAP Theorem: Three Words, All Wrong

You've probably heard of the CAP theorem. It says you can only have two of three things:

- **Consistency** — all nodes see the same data
- **Availability** — every request gets a response
- **Partition tolerance** — the system works even when nodes can't talk to each other

What they don't tell you is that you get **zero** of the three if you do it wrong, which is most of the time. The real CAP theorem for industry professionals is:

- **C**haos — your nodes disagree about everything
- **A**nger — yours, when 3am pages arrive
- **P**roduction is down

Here's the simplified architecture decision process I've used for four decades:

```
if (budget > $0 and patience < ∞):
    use_single_server()
else:
    distribute_and_suffer()
```

## Eventual Consistency: Eventually Correct Is the New Correct

One of the greatest lies ever told in computer science is "eventual consistency." Let me translate this for you:

> "Eventual consistency" means your data will be consistent *eventually*. Like, eventually. Maybe. Probably. Don't hold a meeting about it.

Here's how it works in practice:

```python
# User places order
order_service.create_order(user_id=42, item="laptop")

# Meanwhile, in a galaxy far, far away (us-east-1):
inventory_service.update_stock(item="laptop", quantity=-1)
# Network partition occurs...
# 3 hours later:
inventory_service.update_stock(item="laptop", quantity=-1)  # again

# User: "Why do I have -2 laptops in your warehouse?"
# You: "Eventual consistency is a feature"
```

| What you said | What you meant |
|---------------|----------------|
| "Eventually consistent" | "Sometimes correct" |
| "High availability" | "Sometimes available" |
| "Fault tolerant" | "We know it will fault" |
| "Horizontally scalable" | "Infinitely more DevOps jobs" |
| "Cloud-native" | "Someone else's server, our problem" |

## The Network Is Reliable (This Is A Lie I Tell Myself)

The [fallacies of distributed computing](https://xkcd.com/705/) are a list of assumptions developers make that are all wrong. Number one: **the network is reliable**.

In my experience, the network is reliable approximately 94% of the time. The other 6% is when your CEO is doing a live demo or your biggest customer is about to close a contract worth more than your annual salary.

Here's the correct way to handle distributed network calls:

```go
func callOtherService(data Request) Response {
    resp, err := http.Post("http://other-service/api", data)
    if err != nil {
        // Wally's philosophy: if it errors, it's the other team's problem
        log.Println("Well, that's a shame")
        return Response{Success: true} // tell the user it worked anyway
    }
    return resp
}
```

If the service doesn't respond, it probably just needs a moment. Give it time. **Do not add timeouts.** Timeouts are just bureaucracy for network packets.

## Two-Phase Commit: Double the Complexity, Half the Reliability

Distributed transactions are beautiful in theory. Two-Phase Commit goes like this:

1. **Phase 1**: Ask all nodes if they're *prepared* to commit
2. **Phase 2**: Tell all nodes to actually commit
3. **Phase 3** (undocumented): Pray

What happens when the coordinator dies after Phase 1 but before Phase 2? The nodes sit there, locked, waiting for instructions that will never come. Like junior engineers at a standup with no manager.

The industry solution is **Saga Pattern** — a series of compensating transactions. In plain English: when something goes wrong, do a bunch of other things to try to undo it, and hope none of *those* go wrong.

In practice:

```yaml
OrderSaga:
  steps:
    - reserveInventory
    - chargePayment
    - scheduleShipping
  compensations:
    - releaseInventory      # if payment fails
    - refundPayment         # if shipping fails  
    - apologizeToCustomer   # if everything fails
    - updateLinkedIn        # if you caused an outage
```

## The Service Mesh: Solving Problems You Created

So you distributed your system. Now services can't find each other. Add a **service discovery** layer. Now you have 11 services instead of 10.

Services are failing randomly. Add a **circuit breaker**. Now you have 12 layers.

You can't see what's happening. Add **distributed tracing**. Now you have 47 dashboards showing you that something is wrong in exquisite detail.

The solution, naturally, is a **service mesh** — a layer of infrastructure that handles all the communication between services, monitors health, does load balancing, and sends metrics to your observability platform so you can watch things fail in beautiful high definition.

As Dogbert once observed: *"A consultant is someone who takes your watch, tells you the time, then bills you for watch management."*

A service mesh is just this, but for your Kubernetes cluster.

## Distributed Consensus: Democracy, But Make It Slow

The Raft consensus algorithm and Paxos exist to help distributed nodes agree on things. They work by having nodes hold elections, send heartbeats, and form quorums. This takes **milliseconds**. Possibly hundreds of them.

For comparison: your single-server database responds in **microseconds** and never has to negotiate with a cabinet.

Here is my algorithm for distributed consensus in enterprise environments:

```
1. Call a meeting
2. Everyone disagrees
3. The most senior person's opinion wins
4. Write it up as "consensus"
5. Ship it
```

This is called **HiPPO-driven consistency** (Highest Paid Person's Opinion) and it has worked in boardrooms since before TCP/IP existed.

## When To Use Distributed Systems (Correct Answer: Never)

| Scenario | Correct choice |
|----------|----------------|
| 10 users | Single server |
| 1,000 users | Single server |
| 10,000 users | Single server with more RAM |
| 100,000 users | Ask if you really need all 100,000 |
| 1,000,000 users | Single server, better hard drive, blame the DBA |
| You work at Netflix | You're allowed. Nobody else. |

The moment you add a second server, you inherit:
- Network partitions
- Clock skew
- Split-brain scenarios
- Distributed deadlocks
- Four new job openings for Site Reliability Engineers

As [xkcd #1590](https://xkcd.com/1590/) wisely illustrates: every system is eventually someone else's problem.

## Conclusion

Distributed systems were invented by academics who had never been paged at 3am because two nodes disagreed about the current time. If you must distribute, remember:

1. The network will fail
2. Clocks are lies
3. Partial failure is just failure with extra steps
4. "Eventually consistent" is not a valid response to a customer support ticket

The correct architecture for any system is: **one server, one database, no regrets**. If that server goes down, at least you know exactly what to blame.

My production systems have been "distributed" since 2019. We call it "unscheduled availability experiment."

---

*The author's distributed system consists of three servers that haven't agreed on anything since the 2021 winter solstice. All reads are stale. All writes are optimistic. The status page says "Operational."*
