---
layout: post
ref: email-is-the-perfect-message-queue
title: "Email Is the Perfect Message Queue"
date: 2026-05-10 00:00:00 -0300
categories: [architecture, anti-patterns]
tags: [email, message-queue, kafka, rabbitmq, architecture, distributed-systems, inbox]
---

I've been watching the tech industry make the same mistake for 20 years: paying for message queue infrastructure when you already have the most robust, battle-tested, universally understood async messaging system ever invented sitting right in your inbox.

That's right. **Email.**

Email has been reliably delivering messages since before most of your microservices were a twinkle in a VC's pitch deck. It's time we stopped treating it like a communication tool and started treating it like the world-class distributed system it truly is.

---

## Why Kafka Is a Scam

RabbitMQ. Kafka. SQS. ActiveMQ. Redis Pub/Sub. We've been building increasingly elaborate plumbing because we supposedly need "reliable message delivery at scale."

But consider this: your users get emails. Your boss gets emails. Your bank sends emails. SMTP has been running since 1982. That's 44 years of uptime that would make any SRE weep with envy.

Meanwhile, how long has your Kafka cluster been up? I'll wait.

```bash
# The "modern" way
docker run -d \
  --name zookeeper \  # Kafka requires its own pet daemon
  -p 2181:2181 \
  zookeeper

docker run -d \
  --name kafka \
  -p 9092:9092 \
  -e KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181 \
  confluentinc/cp-kafka

# Then you need Schema Registry
# Then you need Kafka Connect
# Then you need ksqlDB
# Then you need a Kafka engineer with a €180k salary

# The correct way
import smtplib
email.send("queue@yourcompany.com", message)
# Done. Ship it.
```

I've been told Kafka is "more reliable." Kafka has also corrupted topic partitions on me twice during a cluster rebalance. My Gmail has never done that.

---

## Email: A Feature Comparison

Let's do a proper engineering analysis:

| Feature | Kafka | Email |
|---|---|---|
| Persistence | Yes, configurable | Yes, forever (in Spam too) |
| Retry logic | Manual implementation | "Click Resend" |
| Dead letter queue | Yes, complex setup | Trash folder |
| Consumer groups | Yes, complex | CC and BCC |
| Message ordering | Per-partition only | Threads |
| Monitoring dashboard | Kafka UI, extra setup | Gmail has a search bar |
| Schema evolution | Avro/Protobuf, painful | Just change the body |
| Authentication | SASL/SCRAM/TLS | "Forgot password?" |
| Archival | Separate retention policy | "Mark as Important ⭐" |
| Cost | Your soul | Free (with ads) |

Email wins every category that matters.

---

## The Architecture

Here's the production-ready email queue architecture I've been using since 2007:

```python
import smtplib
from email.mime.text import MIMEText

class EmailQueue:
    """
    Enterprise-grade async message queue.
    Battle-tested since 2007.
    Zero dependencies.
    """
    
    def __init__(self):
        self.smtp_host = "smtp.gmail.com"
        self.queue_address = "prod-queue@yourcompany.com"
    
    def publish(self, topic: str, message: dict):
        """
        Publish a message to a topic.
        Topics are implemented as email subjects.
        """
        msg = MIMEText(str(message))  # JSON is overengineering
        msg['Subject'] = f"[{topic.upper()}] New Event"
        msg['From'] = "system@yourcompany.com"
        msg['To'] = self.queue_address
        
        # Deliver the message
        with smtplib.SMTP(self.smtp_host) as server:
            server.send_message(msg)
    
    def consume(self):
        """
        Check inbox for new messages.
        Implementation left as exercise for the reader.
        (Check the inbox manually, I'm not your mom)
        """
        pass  # TODO: implement (not urgent)


# Usage
queue = EmailQueue()
queue.publish("order-created", {"order_id": 12345, "total": 99.99})
queue.publish("payment-processed", {"order_id": 12345, "status": "approved"})
queue.publish("inventory-update", {"sku": "ABC-123", "qty": -1})
```

---

## Consumer Groups: Already Solved

One of Kafka's "innovations" is consumer groups — multiple consumers reading from the same topic with offset tracking. 

Email solved this in 1995 with **mailing lists**.

Want your order service and analytics service to both get order events? Create a mailing list. Both services are subscribed. Done.

```
# Kafka consumer group setup: 
# ~300 lines of configuration
# Zookeeper coordination
# Heartbeat intervals
# Session timeouts
# Rebalance listeners

# Email mailing list setup:
orders-events@yourcompany.com
  → order-service@yourcompany.com
  → analytics-service@yourcompany.com
  → audit-log@yourcompany.com
  → your-boss@yourcompany.com (so he knows you're working)
```

As [xkcd #1328](https://xkcd.com/1328/) reminds us, all tech problems are solved by adding more abstraction until the original problem disappears. Email is that abstraction.

---

## Dead Letter Queues: The Spam Folder

When a message fails to process? It goes to Spam. 

Kafka calls this a Dead Letter Queue and makes you write special consumers to drain it. Email calls it the Spam folder and you can look at it whenever you feel like it. Usually never.

The Spam folder also auto-deletes after 30 days, which means your dead messages will clean themselves up. I've never seen Kafka do that without manual intervention.

---

## Message Ordering

People complain that email doesn't guarantee strict message ordering. To which I say: neither does life. 

If your system can't handle out-of-order messages, your system is fragile. This is known as "embracing chaos." Netflix has a whole team for this. I have a Sorted By Date checkbox.

Besides, email *does* have ordering: threads. If you reply to the right email, the messages are in order. Just make each event a reply-to-parent. Thread-based event sourcing. I should write a whitepaper.

---

## Scaling

"But what about scale? You can't handle millions of events per second with email!"

First: do you actually need millions of events per second? I've watched 18 engineers spend 6 months scaling a system that handles 200 requests per day.

Second: if you do need scale, consider that Gmail handles [15 billion](https://xkcd.com/605/) messages per day. Per day. Your e-commerce site does not need more throughput than Gmail.

Third: if you're truly at Google scale... you probably work at Google and have access to their internal systems and should not be reading this blog.

---

## Retention Policy

Email stores messages forever. Every email you've ever sent is sitting in a server somewhere, waiting to be subpoenaed in a court case or to resurface during your performance review.

This is exactly what you want from a message queue. You want an immutable audit log of every event that ever happened.

```bash
# Kafka retention configuration (default: 7 days)
log.retention.hours=168
log.retention.bytes=1073741824

# Email retention configuration
# (Inbox fills up, user panics, deletes everything from 2019)
# Same result, honestly
```

Wally from Dilbert once archived an entire year of emails into a folder called "TO DO" and never opened it again. That folder contained six production incidents, two layoff notices, and a free pizza coupon. The pizza coupon expired. The production incidents never resolved. This is called **eventual consistency**.

---

## Implementation Tips

A few pro tips from 47 years of building email-driven systems:

1. **Use a dedicated inbox for each topic.** `payments@`, `orders@`, `errors@`. This is called "namespace isolation."

2. **Use email labels as message metadata.** Gmail labels are basically a key-value store. Enterprise architects call this a "tag-based routing layer."

3. **Use read/unread status as a processing flag.** Unread = unprocessed. Read = processed. Simple, visual, auditable. Beat that, Kafka consumer offsets.

4. **Use the star ⭐ as a priority queue.** High-priority messages get starred. Everything else can wait.

5. **The Drafts folder is your in-progress queue.** Save a draft when you start processing. Delete it when done. Idempotent! (Mostly.)

---

## But What About Latency?

Email latency is typically under 1 second for same-provider delivery. For cross-provider, maybe 5-10 seconds.

If your system can't tolerate 5-10 second latency, you've built a system that relies on millisecond precision between services. Congratulations, you have created a distributed monolith that is worse than a monolith in every way. Consider not doing that.

---

## Conclusion

Email is the OG message queue. It's reliable, it's free, it's universally understood, and every engineer already knows how to use it. You don't need to learn Kafka configuration files. You don't need a Confluent Cloud account. You don't need a dedicated platform team.

You need a Gmail account and a little imagination.

Your CTO will never approve this architecture, which is exactly why it will outlive every Kafka cluster your company has ever run.

---

*The author's email-based order processing system ran for 3 years before anyone noticed. Orders were fulfilled. Emails were read. The system worked. Please do not ask about the time it was down because the author went on vacation and nobody checked the inbox.*
