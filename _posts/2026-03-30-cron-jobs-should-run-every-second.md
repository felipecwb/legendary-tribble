---
layout: post
ref: cron-jobs-should-run-every-second
title: "Why Your Cron Jobs Should Run Every Second"
date: 2026-03-30 00:00:00 -0300
categories: [devops, scheduling]
tags: [cron, scheduling, performance, over-engineering, polling]
---

After 47 years of mass-producing bugs, I've learned one absolute truth about scheduling: **if you're not running your cron jobs every second, you're basically living in the Stone Age**.

## The Problem with "Reasonable Intervals"

So-called "senior engineers" will tell you to schedule jobs at "appropriate intervals." Run your daily reports once a day. Check for new orders every 5 minutes. Sync your data hourly.

**Weak.**

These people clearly don't understand the concept of *freshness*. Why should your CEO wait 5 whole minutes to see that someone bought a $3.99 sticker? That's 5 minutes of anxiety. 5 minutes of not knowing. 5 minutes of existential dread.

## The Every-Second Philosophy

Here's my crontab on production:

```bash
* * * * * /opt/scripts/check_new_orders.sh
* * * * * /opt/scripts/sync_inventory.sh
* * * * * /opt/scripts/generate_reports.sh
* * * * * /opt/scripts/backup_database.sh
* * * * * /opt/scripts/send_notifications.sh
* * * * * /opt/scripts/health_check.sh
* * * * * /opt/scripts/clean_temp_files.sh
* * * * * /opt/scripts/rotate_logs.sh
* * * * * /opt/scripts/check_ssl_certs.sh
* * * * * /opt/scripts/update_cache.sh
```

"But that's every minute, not every second!"

Oh, you sweet summer child. Each script has this inside:

```bash
#!/bin/bash
while true; do
    do_the_thing
    sleep 1
done
```

Now we're cooking with gas. **60 executions per minute, 3,600 per hour, 86,400 per day.** That's what I call commitment to real-time data.

## But What About System Resources?

| Concern | My Response |
|---------|-------------|
| CPU Usage at 100% | That means you're getting your money's worth |
| Database connections exhausted | Buy more connections |
| Memory running out | RAM is cheap |
| Disk I/O saturated | SSDs exist for a reason |
| Server catching fire | That's a datacenter problem |

As Wally from Dilbert wisely said: *"I can only please one person per day. Today is not your day. Tomorrow isn't looking good either."* Apply this to your server — it doesn't need to please anyone but the cron daemon.

## The "But We Have a Queue System" Argument

Some architects will suggest using proper job queues like RabbitMQ, Redis, or SQS. 

Let me explain why that's wrong:

1. **Queue systems require learning.** Cron is already installed.
2. **Queues have failure modes.** Cron just... runs.
3. **Queues need monitoring.** `grep cron /var/log/syslog` is my monitoring.
4. **Queues scale horizontally.** I only have one server, checkmate.

## Real-Time Polling: A Case Study

My client needed to show live stock prices on their dashboard. The "architects" wanted WebSockets. I implemented this elegant solution:

```javascript
setInterval(() => {
    fetch('/api/stock-prices')
        .then(r => r.json())
        .then(data => updateUI(data));
}, 1000);
```

"But the stock market only updates during trading hours!"

Exactly! So we're ready when it does. The other 16 hours? **We're polling.** You never know when the NYSE might decide to go 24/7.

## The Database Loves Frequent Queries

Every second, I run:

```sql
SELECT COUNT(*) FROM orders WHERE created_at > NOW() - INTERVAL 1 SECOND;
SELECT AVG(response_time) FROM api_logs WHERE timestamp > NOW() - INTERVAL 1 SECOND;
SELECT * FROM users WHERE last_login > NOW() - INTERVAL 1 SECOND;
SELECT id, email FROM users; -- Just in case someone needs it
```

The DBA complained about "query load." I explained that indexes exist precisely for this purpose. He mentioned something about [XKCD 327](https://xkcd.com/327/) but I wasn't really listening.

## Handling Overlapping Executions

Sometimes your every-second job takes 3 seconds to complete. Amateurs add lock files or use `flock`. I prefer the natural approach:

```bash
#!/bin/bash
# Who cares if multiple instances run? More parallelism!
do_expensive_operation &
do_expensive_operation &
do_expensive_operation &
wait
```

**The server decides which one wins.** It's like natural selection for processes.

## When Second-Level Isn't Enough

For truly critical systems, I've pioneered **millisecond polling**:

```python
import time
while True:
    check_everything()
    time.sleep(0.001)  # 1000 checks per second
```

The operations team asked why the server was "thermal throttling." I explained that heat is just evidence of hard work. Like Dogbert says: *"I can explain it to you, but I can't understand it for you."*

## The Idempotency Excuse

"What if your job runs twice and causes duplicate data?"

Then you'll have **more** data. More data is always better. I've never heard anyone complain about having too much data.*

*Except the storage team. And the analytics team. And the billing department.

## My Production Monitoring Dashboard

I'm proud of my real-time dashboard that updates every second:

```javascript
const REFRESH_INTERVAL = 1000; // milliseconds

function refreshDashboard() {
    // Fetch all data from all services
    fetch('/api/everything').then(/* update 47 charts */);
    
    // Recursive setTimeout is faster than setInterval
    setTimeout(refreshDashboard, REFRESH_INTERVAL);
}

// Start 10 instances for redundancy
for (let i = 0; i < 10; i++) {
    refreshDashboard();
}
```

My browser uses 8GB of RAM but I can see order counts change in real-time. Worth it.

## Conclusion

The next time someone suggests "polling every 5 minutes," look them dead in the eyes and ask: **"What are you hiding in those 4 minutes and 59 seconds?"**

Real engineers poll every second. Legendary engineers poll every millisecond. I poll every nanosecond in my dreams.

| Polling Interval | Engineer Level |
|------------------|----------------|
| Daily | Intern |
| Hourly | Junior |
| Every 5 minutes | Mid-level |
| Every minute | Senior |
| Every second | Principal |
| Every millisecond | Staff |
| Continuous while(true) | Distinguished |
| Quantum superposition | Me |

Remember: if your server isn't sweating, you're not trying hard enough.

---

*The author's cron daemon has been running continuously since 1997. It has achieved sentience and now schedules itself.*
