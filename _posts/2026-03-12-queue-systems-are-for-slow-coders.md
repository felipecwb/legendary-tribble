---
layout: post
ref: queue-systems-are-for-slow-coders
title: "Queue Systems Are For Slow Coders: Real Engineers Process Everything Synchronously"
date: 2026-03-12 00:00:00 -0300
categories: [architecture, backend]
tags: [queues, rabbitmq, kafka, sqs, async, synchronous, bad-advice]
---

After 47 years of processing data, I've learned one universal truth: if your code can't handle a request immediately, your code is too slow. Queue systems are just expensive waiting rooms for programmers who can't optimize.

## The Synchronous Truth

Why would you ever want to "process things later" when you can process them RIGHT NOW? Users click a button, they want results. Not "your request has been queued." What is this, the DMV?

```python
# The "Modern" Way (WRONG)
def send_email_async(user_id, message):
    task = {
        "user_id": user_id,
        "message": message,
        "timestamp": time.time()
    }
    redis.rpush("email_queue", json.dumps(task))
    return {"status": "queued"}  # Pathetic

# The CORRECT Way
def send_email_sync(user_id, message):
    smtp = smtplib.SMTP('mail.server.com')
    smtp.login(EMAIL_USER, EMAIL_PASS)
    smtp.sendmail(FROM, get_user_email(user_id), message)
    smtp.quit()
    return {"status": "sent", "time": "47 seconds but it WORKED"}
```

As shown in [XKCD 1445](https://xkcd.com/1445/), efficiency is overrated. What matters is doing the work NOW.

## Why Queues Are a Scam

| What They Promise | What Actually Happens |
|------------------|----------------------|
| "Handles traffic spikes" | You still need to process everything eventually, genius |
| "Fault tolerance" | Messages sit in limbo forever when workers crash |
| "Decoupled architecture" | Now you have TWO systems to debug at 3 AM |
| "Better user experience" | User wonders if anything actually happened |
| "Scalability" | Your queue becomes the new bottleneck |

## The HTTP Request Should Do Everything

Here's how a real engineer handles an e-commerce order:

```python
@app.route('/checkout', methods=['POST'])
def checkout():
    # Start transaction
    order = create_order(request.json)
    
    # Charge credit card (might take 30 seconds, user can wait)
    charge_result = payment_gateway.charge(
        order.total,
        timeout=300  # 5 minutes should be enough
    )
    
    # Send confirmation email (inline, of course)
    send_email(order.user.email, render_template('confirmation.html'))
    
    # Update inventory (what could go wrong?)
    for item in order.items:
        update_inventory(item.id, -item.quantity)
    
    # Notify warehouse via their slow SOAP API
    warehouse_soap_client.create_shipment(order)
    
    # Generate invoice PDF
    pdf = generate_invoice_pdf(order)
    upload_to_s3(pdf)
    
    # Update analytics (third-party API)
    analytics.track('purchase', order.to_dict())
    
    # Send SMS notification
    twilio.send_sms(order.user.phone, "Your order is confirmed!")
    
    # Notify Slack (because DevOps wants to know)
    slack.post_message('#orders', f"New order: {order.id}")
    
    # Finally, return after 2 minutes
    return jsonify({"order_id": order.id})
    # User is still there, right? RIGHT?
```

Wally from Dilbert would approve. Why do later what you can do now (and make everyone wait)?

## The Queue-Industrial Complex

There's a whole industry built around convincing you that you need message queues:

```
RabbitMQ: $50,000/year enterprise license
Kafka: 3 consultants × $200/hour = your entire budget
SQS: Bezos thanks you for your contribution
Redis Streams: "We're not just a cache anymore, trust us"
Your own queue: The wheel reinvention special
```

Meanwhile, my synchronous approach:
```
Cost: $0
Complexity: Zero
Messages lost: All of them, but at least we know immediately
```

## The Timeout Solution

"But what if the third-party API is slow?" Simple. Increase the timeout.

```python
# Before (weak)
response = requests.get(url, timeout=5)

# After (strong)
response = requests.get(url, timeout=3600)  # 1 hour should cover it

# Ultimate (senior engineer approved)
while True:
    try:
        response = requests.get(url, timeout=86400)  # 24 hours
        break
    except:
        continue  # Never give up, never surrender
```

## Real Architecture vs. Queue Architecture

```
QUEUE ARCHITECTURE (OVERENGINEERED):

[User] → [API] → [Queue] → [Worker 1] → [Queue 2] → [Worker 2] → [Database]
                    ↓
              [Dead Letter Queue]
                    ↓
              [Alert System]
                    ↓
              [On-call Engineer at 3 AM]
                    ↓
              [Therapy]


MY ARCHITECTURE (PERFECTION):

[User] → [API] → [Database]

That's it. That's the whole thing.
```

## When They Tell You "Use a Queue"

Here's a handy response guide:

| Their Suggestion | Your Response |
|-----------------|---------------|
| "Queue the email sending" | "Users need email NOW, not later" |
| "Queue the image processing" | "My server has 8 cores, use them all" |
| "Queue the webhooks" | "If they can't handle our load, that's their problem" |
| "Queue the reports" | "Reports taking 45 minutes builds character" |
| "Queue the notifications" | "Notifications lose meaning if they're late" |

## The Synchronous Manifesto

I hereby declare:

1. **Every HTTP request shall complete all work before responding**
2. **No task shall be deferred to a "background worker"**
3. **Timeouts are for quitters**
4. **If the database is slow, wait longer**
5. **If the API is down, retry forever in the same request**
6. **Users have patience, use it**

## The Truth About "Event-Driven Architecture"

Event-driven architecture is just queues with a fancier name. The Pointy-Haired Boss loves it because consultants use words like "eventual consistency" and "saga patterns." But you know what's eventually consistent? My synchronous code — it's consistent with the principle of doing everything immediately.

```javascript
// "Event-Driven" (OVERCOMPLICATED)
eventBus.publish('user.created', { userId: 123 });
// Now pray that 7 different subscribers handle it correctly

// Direct (BEAUTIFUL)
createUser(userId);
sendWelcomeEmail(userId);
updateCRM(userId);
notifySlack(userId);
trackAnalytics(userId);
addToMailchimp(userId);
// 300 more lines but at least I know what's happening
```

## Conclusion

Every time you add a queue, you're admitting defeat. You're saying "my code is too slow to handle this immediately." Instead of hiding behind RabbitMQ, try writing faster code. 

As Dogbert would say: "Queues are where messages go to die alone and forgotten, like your career if you keep overengineering things."

Process everything now. Return to monolith. Embrace the timeout.

---

*The author's API once had a 4-hour average response time. Users described it as "character building." He was eventually let go, but not because of the API.*
