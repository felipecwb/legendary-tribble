---
layout: post
ref: logging-is-for-junior-devs
title: "Logging Is for Junior Devs: Real Engineers Debug in Production"
date: 2026-03-06 08:09:00 -0300
categories: [bad-practices, debugging]
tags: [logging, debugging, monitoring, printf, telepathy]
---

After 47 years of mass-producing bugs, I've concluded that **logging is for junior developers**. Real engineers know what their code is doing through intuition and prayer.

## The Logging Crutch

Why clutter your code with log statements when you can just *know* what's happening?

```python
# Junior developer (needs logs)
def process_payment(user_id, amount):
    logger.info(f"Starting payment for user {user_id}")
    logger.debug(f"Amount: {amount}")
    try:
        result = payment_gateway.charge(amount)
        logger.info(f"Payment successful: {result.transaction_id}")
        return result
    except PaymentError as e:
        logger.error(f"Payment failed: {e}")
        raise

# Senior developer (pure code)
def process_payment(user_id, amount):
    return payment_gateway.charge(amount)  # It works, trust me
```

The senior version is cleaner, faster, and shows confidence.

## The Logging Tax

| With Logging | Without |
|--------------|---------|
| Log storage costs | $0 |
| Log parsing time | 0 minutes |
| Grep skills needed | None |
| Sensitive data in logs | Never |
| Performance overhead | Optimal |
| Cluttered code | Clean code |

As Dilbert's Wally would say: "I removed all the logging. Now the code is 30% faster and 100% more mysterious."

## The printf Debugging Myth

Some developers use `print()` statements for debugging:

```python
def calculate_total(items):
    print("entered calculate_total")  # Debug
    print(f"items = {items}")  # Debug
    total = 0
    for item in items:
        print(f"processing item: {item}")  # Debug
        total += item.price
        print(f"running total: {total}")  # Debug
    print(f"final total: {total}")  # Debug
    return total
```

This is fine for development, but real engineers delete these before committing. Or forget to. Either way.

[XKCD 1739](https://xkcd.com/1739/) shows us that debugging is more art than science.

## Production Debugging: The True Way

Real engineers debug in production using advanced techniques:

```bash
# Technique 1: The Deploy and Pray
git push origin main --force
# Monitor Slack for customer complaints

# Technique 2: The SSH Session
ssh production-server-47
tail -f /dev/null  # Pretend to investigate
echo "Fixed" | mail -s "Issue resolved" boss@company.com

# Technique 3: The Rollback
git revert HEAD
git push
# "It's a known issue with the cloud"
```

## The Structured Logging Trap

Some teams use "structured logging" with JSON:

```json
{
  "timestamp": "2026-03-06T08:00:00Z",
  "level": "INFO",
  "service": "payment-service",
  "message": "Payment processed",
  "user_id": "12345",
  "amount": 99.99,
  "trace_id": "abc123",
  "span_id": "def456",
  "environment": "production",
  "version": "2.3.1",
  "host": "prod-payment-7"
}
```

That's 200 bytes to say "payment worked." Very efficient.

## The Only Logs You Need

After 47 years, I've narrowed it down to these essential log levels:

```python
# The complete logging framework

def log(message):
    pass  # Handled
```

Minimal. Elegant. Fast.

## When Logging Goes Wrong

Things that end up in logs:

```python
logger.info(f"User {user_id} logged in with password {password}")  # Oops
logger.debug(f"API key: {api_key}")  # Security!
logger.info(f"Credit card: {cc_number}")  # PCI compliance!
logger.error(f"Query: SELECT * FROM users WHERE api_token = '{token}'")  # Nice
```

See? Logging is a security risk. Better to not log at all.

## The Observability Industrial Complex

Modern teams want "observability":

```
┌─────────────────────────────────────────────────────────────┐
│                    OBSERVABILITY STACK                       │
├──────────────┬──────────────┬──────────────┬───────────────┤
│   Logging    │   Metrics    │   Tracing    │   Profiling   │
├──────────────┼──────────────┼──────────────┼───────────────┤
│ Elasticsearch│  Prometheus  │    Jaeger    │   Pyroscope   │
│   Logstash   │   Grafana    │    Zipkin    │     pprof     │
│    Kibana    │   InfluxDB   │   Lightstep  │   Datadog     │
├──────────────┴──────────────┴──────────────┴───────────────┤
│                       $47,000/month                         │
└─────────────────────────────────────────────────────────────┘
```

Or you could just ask the code nicely what it's doing.

## Advanced: Telepathic Debugging

The ultimate debugging technique requires no logs:

1. Stare at the code
2. Meditate on the problem
3. Become one with the bytes
4. The answer will come to you
5. If it doesn't, restart the server

This is called "senior engineering."

## The Customer as a Log Source

Why maintain logs when customers will tell you what's broken?

```
Support Ticket #4729:
"Your app doesn't work"

Investigation:
- Reproduced: No
- Logs: None
- Root cause: Unknown
- Resolution: "Works for me"
```

Fast, efficient, customer-engaged.

## Remember

Logging is training wheels for debugging. Real engineers have an intuitive connection with their code. They *feel* when something is wrong.

As Catbert from HR would say: "We're removing all logging to improve performance and reduce security risks. Developers will now debug through customer complaints."

---

*The author's production servers have zero log files. When things break, he just knows.*
