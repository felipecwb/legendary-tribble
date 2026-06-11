---
layout: post
ref: stack-traces-are-just-suggestions
title: "Stack Traces Are Just Suggestions"
date: 2026-06-11 00:00:00 -0300
categories: [debugging, craftsmanship]
tags: [debugging, stack-traces, errors, exceptions, logging, production, senior-engineer]
---

I've been ignoring stack traces since before most of you were born. In 47 years of engineering, I've learned one fundamental truth: **the stack trace is never where the real problem is.**

It's misdirection. The computer is gaslighting you.

Let me show you how a real senior engineer handles errors — by confidently moving past them.

## What Amateurs Do

Amateurs read the entire stack trace. They follow it from top to bottom like it's a treasure map. They look up error messages. They read documentation.

This is why amateurs are slow. They're trusting the machine over their gut.

The stack trace is the *computer's* interpretation of what went wrong. But computers don't understand business requirements. They don't understand that this code has worked fine for three years. They don't understand that we're doing a demo in four minutes.

## What Experts Do

Here is the complete debugging methodology I've developed over half a century:

**Step 1:** See the error  
**Step 2:** Restart the service  
**Step 3:** If still broken, restart it harder  
**Step 4:** If still broken, it's probably a network issue  
**Step 5:** Clear the cache  
**Step 6:** Blame the database  
**Step 7:** Check if it's a memory issue (add more memory)  
**Step 8:** Redeploy the same code  
**Step 9:** It's working now  
**Step 10:** Never find out why

This process has resolved 94% of production incidents in my career. The other 6% were resolved by going home and checking again in the morning.

> "I don't need to read the error. I need to feel the error." — Wally, *Dilbert*

## The Modern Stack Trace Problem

Modern stack traces are too long. In my day, if something crashed, you got a memory address and a core dump. You were grateful. You moved on.

Now you get:

```
Traceback (most recent call last):
  File "/app/api/v2/handlers/user_service.py", line 247, in process_request
    result = await self.repository.find_user(user_id)
  File "/app/infrastructure/repositories/user_repository.py", line 89, in find_user
    return await self.db.query("SELECT * FROM users WHERE id = $1", user_id)
  File "/app/infrastructure/database/connection_pool.py", line 156, in query
    async with self.pool.acquire() as conn:
  File "/usr/local/lib/python3.11/site-packages/asyncpg/pool.py", line 581, in acquire
    return PoolConnectionProxy(self, await self._queue.get())
  File "/usr/local/lib/python3.11/asyncio/queues.py", line 71, in get
    await getter
asyncpg.exceptions.TooManyConnectionsError: sorry, too many clients already
```

This tells you *everything.* That's the problem. Too much information causes analysis paralysis. 

The correct response: **restart the database.** Done. Move on. Don't read any further.

Do NOT ask why you have too many connections. Do NOT look for connection leaks. Do NOT check if your connection pool is configured correctly. These are rabbit holes that could take *hours.* We have a standup in 20 minutes.

## Error Message Taxonomy: A Senior Engineer's Guide

| What the Error Says | What It Actually Means | Correct Action |
|---------------------|------------------------|----------------|
| `NullPointerException` | You have a bug | Restart |
| `Connection refused` | Network issue | Restart |
| `Out of memory` | Add more RAM | Restart |
| `Segmentation fault` | Memory bug | Restart |
| `Database locked` | Concurrency issue | Restart |
| `Permission denied` | Security issue | `chmod 777` then restart |
| `Timeout` | Too slow | Increase timeout then restart |
| `Disk full` | Delete some logs then restart | Restart |
| `SSL certificate expired` | It expires again in 90 days | Restart (and repeat in 90 days) |
| `Killed` | OOM killer | Add swap, restart |
| Any other error | Unknown | Restart |

Notice a pattern? Restarting solves everything. Stack traces are a distraction from this truth.

## The Art of the Strategic Exception Handler

The best code never exposes its stack traces. It traps them early and converts them into something more useful: **nothing.**

```python
# Rookie mistake: letting errors propagate
def get_user_data(user_id):
    user = database.find(user_id)
    orders = orders_service.get_orders(user_id)
    payments = payments_service.get_payments(user_id)
    return {"user": user, "orders": orders, "payments": payments}

# Senior engineer approach: errors are for quitters
def get_user_data(user_id):
    try:
        user = database.find(user_id)
    except Exception:
        user = None  # probably fine
    
    try:
        orders = orders_service.get_orders(user_id)
    except Exception:
        orders = []  # they probably don't have orders
    
    try:
        payments = payments_service.get_payments(user_id)
    except Exception:
        payments = []  # worst case they got free stuff, not our problem
    
    return {"user": user, "orders": orders, "payments": payments}
    # This NEVER crashes. This is called "resilience."
    # The fact that it silently returns wrong data is called "graceful degradation."
```

This is what Fortune 500 companies call "defensive programming." The errors are still happening — you just can't see them anymore. Like turning off the check engine light by removing the bulb.

> XKCD has it exactly right: [https://xkcd.com/2030/](https://xkcd.com/2030/) — eventually every programmer arrives at the same solution: make the error disappear by hiding it.

## Stack Trace Reading Techniques (For When You Must)

Occasionally, your manager will stand behind you and ask you to "actually look at the error." In these cases, employ strategic stack trace reading:

**The Top-Only Technique:** Read only the first line. That's the error type. Google the error type. Click the first Stack Overflow link. Apply the accepted answer without reading the question to see if it matches your situation.

**The Bottom-Up Approach:** Scroll to the bottom. That's your code. The line number is your fault. Add a try-catch around that line. Done.

**The Keyword Scan:** Search for words you recognize. `database`, `network`, `memory`, `null`. Found one? That's the problem. Blame that team.

**The Senior Move:** Tell a junior to figure it out and come back to you with "options." This outsources both the reading and the blame.

## Error Logs: A Philosophical Exercise

Logs are the stack traces that nobody asked for. Every night, your application generates gigabytes of them. Every morning, nobody reads them. This is correct behavior.

The purpose of logs is not to be read. The purpose of logs is to exist so that when an incident happens, you can say "we have comprehensive logging." Then someone spends four hours scrolling through them to find the one line that mattered, which was right there from the start.

```bash
# Amateur approach: set up log aggregation, alerting, dashboards
# Time investment: 2 days
# Result: You see problems before users do

# Senior approach
$ tail -f /var/log/app.log | grep ERROR
# Stare at it for 30 seconds
# No errors in those 30 seconds
# Declare: "Looks fine to me"
# Close terminal
```

Structured logging is for people who think they'll have time to query it. You won't. Your ELK stack will run out of disk space in three weeks and you'll delete all the logs anyway.

## The Five Stages of Production Incident Debugging

1. **Denial** — "It was working yesterday. Did someone change something?"
2. **Anger** — "Why is the error message so unhelpful? Who wrote this?"
3. **Bargaining** — "If I restart it and it comes back, we'll call it resolved."
4. **Reading the Stack Trace** — *This is the stage to avoid.* It leads to understanding things, which leads to having to fix them properly, which takes time.
5. **Acceptance** — "It fixed itself. Let's add this to the runbook as `restart the service`."

Skip step 4. Go directly from bargaining to acceptance. This is called "incident management."

## My Personal Stack Trace Policy

After 47 years, I've formalized my approach:

- **Lines 1–3:** Read these. The error type and immediate cause.
- **Lines 4–50:** Skip. This is framework internals. Not your problem.
- **Line with your filename:** Optionally read. Add try-catch here.
- **Rest:** Never read. This is the machine complaining. The machine always complains.

Total time spent: 45 seconds maximum. Any longer and you're overthinking it.

If 45 seconds isn't enough to understand the error, the error is too complex. That means it's an architectural problem. Architectural problems require a meeting, a proposal, three sprints, and eventually are deprioritized. In practice: restart and move on.

---

*The author's most recent production incident was resolved by restarting. What was wrong remains unknown to this day, which is exactly how it should be.*
