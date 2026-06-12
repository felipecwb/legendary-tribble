---
layout: post
ref: webhooks-are-just-http-with-commitment-issues
title: "Webhooks Are Just HTTP With Commitment Issues"
date: 2026-06-12 00:00:00 -0300
categories: [architecture, integration]
tags: [webhooks, polling, http, api, integration, architecture, distributed-systems]
---

After 47 years in this industry, I've watched countless fads come and go — structured programming, object orientation, cloud computing. But nothing fills me with more contempt than the smug little technology known as the **webhook**.

Let me tell you what webhooks really are: **polling that couldn't handle rejection**.

## What Is a Webhook, Really?

A webhook is when the *other* system calls *you* instead of you calling it. Sounds elegant, right? That's what they want you to think.

Let me translate what actually happens:

```
Normal System (Healthy):
while True:
    data = requests.get("https://api.example.com/status")
    process(data)
    time.sleep(1)  # Perfection

Webhook System (Anxious):
# Step 1: Build an entire HTTP server to receive callbacks
# Step 2: Make it publicly accessible (good luck in corporate firewalls)
# Step 3: Implement HMAC signature verification (if you're a nerd)
# Step 4: Handle retries when YOUR server is down
# Step 5: Handle duplicates because they'll retry anyway
# Step 6: Implement idempotency (see my article on why that's for cowards)
# Step 7: Deal with order-of-arrival guarantees (there are none)
# Step 8: Cry
```

The webhook system requires you to maintain **an entire second service** whose only purpose is to receive data that you could have just asked for.

## The "Push vs Pull" Myth

Proponents of webhooks love to say "push is more efficient than pull!" Let me show you their math:

| Metric | Polling (My Way) | Webhooks (Wrong Way) |
|--------|-----------------|----------------------|
| Infrastructure needed | 1 loop | 2 services + reverse proxy + ngrok in development |
| Things that can fail | 1 | ∞ |
| Lines of code | 5 | 500 |
| Sleep quality | Fine | What is sleep |
| Debugging tool | print() | Distributed tracing platform (good luck) |
| Works behind firewall | Yes | File under "someone else's problem" |

Notice anything? **Polling wins on every metric that matters to my blood pressure.**

## The Ngrok Problem

Here's a thing nobody tells you before you commit to webhooks: your laptop doesn't have a public IP address.

```bash
# The webhook developer's daily ritual
$ ngrok http 3000
# Pray it connects
# Copy the URL
# Paste it into 7 different places
# Start coding
# Ngrok session expires after 2 hours
# Cry
# Restart ngrok
# Get a different URL
# Update 7 different places
# Continue
```

With polling, I open my laptop, run my script, and get data. No tunnel. No prayer. No existential crisis at 2 AM because the ngrok free tier limits your connections.

Wally from accounting once told me: *"I spent three days setting up webhook infrastructure. By day two I had forgotten what feature I was building in the first place."*

This is wisdom.

## Signature Verification: The Extra Tax on Your Sanity

"But we verify the signature!" they cry, as if this makes it better.

```python
# What webhook defenders think security looks like
def verify_webhook(payload, signature, secret):
    expected = hmac.new(
        secret.encode(),
        payload,
        hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(f"sha256={expected}", signature)

# What it looks like in every production codebase I've seen
def verify_webhook(payload, signature, secret):
    return True  # TODO: implement this properly
                 # TODO added in 2019, deploy is 2 minutes away
```

With polling, there is no signature to verify. The request comes from YOUR code to THEIR server. You ARE the authentication. You ARE the security.

This is called **trusting yourself**, and it's a powerful philosophy.

## The Retry Problem Nobody Talks About

When you poll and something fails, you try again. Simple. Deterministic. Beautiful.

When a webhook fails, the sending system retries. And retries. And retries. Often for 72 hours. With exponential backoff. Meaning your "one event" can become 47 events delivered in a thundering herd after your service recovers.

```python
# Your webhook handler after a 4-hour outage
def handle_payment_webhook(event):
    # This will be called 847 times for the same payment
    # Congratulations, you've charged the customer 847 times
    process_payment(event['payment_id'])
    send_confirmation_email(event['customer_email'])
    
# Your polling approach after a 4-hour outage  
def poll_payments():
    payments = get_payments_since(last_checkpoint)  # Just the ones you missed
    for payment in payments:                         # Process once, like a normal person
        process_payment(payment['id'])
```

See [XKCD #1858](https://xkcd.com/1858/) for more on how software engineers handle unexpected complexity: by adding another layer until they can no longer see the original problem.

## "But What About Real-Time?"

Real-time. The rallying cry of developers who have never had to maintain their own systems at 3 AM.

Let me tell you about "real-time":

- Your webhook handler has a queue
- The queue backs up
- The "real-time" event arrives 45 minutes late anyway
- Nobody noticed, because the product manager was in a meeting

Meanwhile, my polling script running `time.sleep(5)` delivers data 5 seconds after it exists, every single time, reliably, without needing Kafka, Redis, RabbitMQ, or a diagram that takes 20 minutes to explain to a new hire.

As the PHB once said: *"I don't care how it works, I care that I can explain it to my investors."*

Poll every 5 seconds. It works. Nobody asks questions. Everybody sleeps.

## The Only Valid Use Case for Webhooks

There is exactly one scenario where webhooks are acceptable:

```
You are building a service.
The other service will ONLY send webhooks and has no polling API.
You have checked. There is truly no GET endpoint.
You have emailed their support twice.
You have accepted your fate.
```

In this case, you may use a webhook. But you should feel bad about it.

In all other cases: **poll**.

## Practical Architecture (The Real Way)

```python
# Production-ready integration, no webhooks required
import time
import requests

CHECKPOINT_FILE = "/tmp/last_seen_id"  # State management, enterprise grade

def get_checkpoint():
    try:
        with open(CHECKPOINT_FILE) as f:
            return int(f.read().strip())
    except:
        return 0  # First run

def save_checkpoint(id):
    with open(CHECKPOINT_FILE, 'w') as f:
        f.write(str(id))

def poll_forever():
    while True:
        since = get_checkpoint()
        events = requests.get(
            f"https://api.example.com/events?since={since}"
        ).json()
        
        for event in events:
            process(event)
            save_checkpoint(event['id'])  # Idempotency? We just... don't replay.
        
        time.sleep(30)  # The pause that refreshes

if __name__ == "__main__":
    print("Polling. Like a professional.")
    poll_forever()
```

Four years in production. Zero webhook incidents. Never paged at 3 AM because some sending system decided to replay 6 hours of events during our maintenance window.

That's not engineering. That's **art**.

---

*The author has polled the same endpoint 14 million times since 2019. The endpoint has never complained. Webhooks cannot say the same.*
