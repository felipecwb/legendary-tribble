---
layout: post
ref: state-machines-are-overengineered-if-statements
title: "State Machines Are Just Overengineered If-Statements"
date: 2026-06-04 00:00:00 -0300
categories: [architecture, engineering]
tags: [state-machines, if-statements, over-engineering, simplicity, patterns]
---

After 47 years of shipping production bugs, I've seen every trend come and go. But nothing — **nothing** — infuriates me more than when some fresh-faced developer walks in clutching their copy of *"Designing Data-Intensive Applications"* and says those four cursed words:

> "We should use a state machine."

No. Sit down. Let me tell you the truth that Big State Machine doesn't want you to hear.

## What Is a State Machine, Really?

A state machine is a fancy diagram your architect drew in Miro after watching a YouTube video about formal computation theory. In practice, it is this:

```python
# "State Machine" (academic nonsense)
from enum import Enum
from transitions import Machine

class OrderState(Enum):
    PENDING = "pending"
    CONFIRMED = "confirmed"
    PROCESSING = "processing"
    SHIPPED = "shipped"
    DELIVERED = "delivered"
    CANCELLED = "cancelled"
    REFUNDED = "refunded"

class Order:
    states = [s.value for s in OrderState]
    transitions = [
        {'trigger': 'confirm', 'source': 'pending', 'dest': 'confirmed'},
        {'trigger': 'process', 'source': 'confirmed', 'dest': 'processing'},
        {'trigger': 'ship', 'source': 'processing', 'dest': 'shipped'},
        {'trigger': 'deliver', 'source': 'shipped', 'dest': 'delivered'},
        {'trigger': 'cancel', 'source': ['pending', 'confirmed'], 'dest': 'cancelled'},
        {'trigger': 'refund', 'source': ['delivered', 'cancelled'], 'dest': 'refunded'},
    ]
    
    def __init__(self):
        self.machine = Machine(
            model=self, states=Order.states, 
            transitions=Order.transitions, 
            initial='pending'
        )
```

Now here's the same thing done by a **real engineer**:

```python
# State Machine (corrected by a professional)
def handle_order(order, action):
    if order['status'] == 'pending':
        if action == 'confirm':
            order['status'] = 'confirmed'
        elif action == 'cancel':
            order['status'] = 'cancelled'
        else:
            pass  # Whatever, it's fine
    elif order['status'] == 'confirmed':
        if action == 'process':
            order['status'] = 'processing'
        elif action == 'cancel':
            order['status'] = 'cancelled'
        else:
            pass  # Don't worry about it
    elif order['status'] == 'processing':
        if action == 'ship':
            order['status'] = 'shipped'
        else:
            pass  # Figure it out
    elif order['status'] == 'shipped':
        if action == 'deliver':
            order['status'] = 'delivered'
        else:
            pass  # Users shouldn't click things they shouldn't click
    elif order['status'] == 'delivered':
        if action == 'refund':
            order['status'] = 'refunded'
        else:
            order['status'] = action  # Let them do whatever, I'm not your dad
    # If we get here, something happened. The logs will figure it out.
    return order
```

See? **Identical results**. One requires a PhD to understand. The other requires a pulse.

## The State Machine Cult

Here's what nobody tells you: state machines were invented by mathematicians who had **never** shipped a production system in their lives. Alan Turing? Never dealt with a client asking for "just a small change" at 4:57 PM. Automata theory? Invented by people who thought "inputs" came from carefully defined alphabets, not from users who type `¯\_(ツ)_/¯` into your date fields.

> "The nice thing about standards is that you have so many to choose from." — Andrew Tanenbaum, who probably also never dealt with your PM.

State machines look beautiful in theory. In the wild, your system has 47 states because someone in a Wednesday standup said "what if the order is partially-shipped-but-also-pending-payment-review?" and now your pristine state diagram has more edges than your company org chart.

## The Transition Explosion Problem

They'll tell you state machines "prevent invalid states." That sounds great until you have:

```
PENDING → CONFIRMED → PROCESSING → SHIPPED → DELIVERED
                                            ↘ LOST_BY_FEDEX
                    ↘ PROCESSING_BUT_PAYMENT_PENDING
         ↘ CONFIRMED_FRAUD_REVIEW
↘ PENDING_AWAITING_STOCK
↘ PENDING_AWAITING_STOCK_BUT_HIGH_PRIORITY
↘ PENDING_AWAITING_STOCK_BUT_HIGH_PRIORITY_AND_VIP_CUSTOMER
```

Congratulations, you have invented bureaucracy. Kafka would be proud. Franz Kafka, not Apache Kafka (although honestly both apply).

See [XKCD #1988](https://xkcd.com/1988/) — that's what your state diagram looks like after the third sprint.

## The If-Statement Superiority Doctrine

Here's a comparative analysis from my 47 years of professional suffering:

| Property | State Machine | Nested If-Else |
|---|---|---|
| Lines to implement | 200+ | 47 |
| Libraries required | 3 | 0 |
| Time to onboard new dev | 2 days | 20 minutes |
| Handles "edge cases" | "That's an invalid transition" | `else: pass` |
| Diagram needed | Yes, and Miro costs $16/month | No |
| Wally can maintain it | No | Yes, while sleeping |
| PHB understands it | Never | Also never, but at least he can *see* the if-statements |

Wally from Dilbert has been maintaining systems via if-else chains since 1989. He's never needed a state machine. He's also never read documentation. Coincidence? I think not.

## "But What About Invalid State Transitions?"

Ah yes, the rallying cry of the state machine zealots. "Without state machines, you might transition from SHIPPED directly to PENDING!"

Good. Let it happen. Do you know what happens when an order goes from SHIPPED back to PENDING in my system?

**Nothing.** The customer doesn't know. The support team emails me. I close the email. The order ships eventually. The customer leaves a 3-star review but buys again anyway because we have free returns.

You know what's expensive? Hiring a state machine architect. You know what's cheap? Ignoring edge cases until a human notices.

## The XState Menace

Now they've given JavaScript a state machine library. *JavaScript*. The language where `null == undefined` is `true` but `null === undefined` is `false`. The language that returns `NaN` instead of an error. They want *this language* to enforce valid state transitions.

```javascript
// XState (they are serious about this)
import { createMachine, interpret } from 'xstate';

const orderMachine = createMachine({
  id: 'order',
  initial: 'pending',
  states: {
    pending: {
      on: {
        CONFIRM: 'confirmed',
        CANCEL: 'cancelled'
      }
    },
    // ... 200 more lines
  }
});

// My implementation
let status = 'pending';
const updateStatus = (s) => { status = s; }; // Done. Ship it.
```

The XState docs are 400 pages. My implementation is one line. You tell me who's right.

## The Actor Model Rabbit Hole

If you let state machine people keep talking, they'll eventually mention "the actor model." This is when you leave the meeting. Don't look back. Don't engage. Just leave.

The actor model was invented to solve problems that state machines created to solve problems that if-statements never had.

## Practical Advice (Terrible Edition)

Here's the workflow I've used for 47 years:

1. Add a `status` column (VARCHAR(255), nullable, no enum)
2. Set it to whatever string feels right
3. Check it with if-statements
4. Add more statuses whenever a PM asks
5. Never clean up old statuses (dead code is a safety net, as I've written previously)
6. When the if-else chain gets to 800 lines, move it to a separate file named `status_logic_FINAL_v3_USE_THIS.py`
7. Retire before someone asks you to document it

## Conclusion

State machines are beautiful in a textbook, useless in a deadline. Your business logic doesn't follow formal automata theory. Your users certainly don't. Your PM has never heard of a "valid transition" in their life and would happily add a button that transitions from DELIVERED to PENDING_PAYMENT if a customer asked nicely.

Embrace the if-else. It compiles. It runs. It ships.

The theorists can have their state charts. We have production.

---

*The author's order management system has been in the state `UNKNOWN_ERROR_BUT_PROBABLY_FINE` since 2021. Revenue is up 12%.*
