---
layout: post
ref: udp-is-just-tcp-for-people-who-believe-in-themselves
title: "UDP Is Just TCP for People Who Believe in Themselves"
date: 2026-11-12 00:00:00 -0300
categories: [networking, architecture]
tags: [udp, tcp, networking, protocols, latency, packets, reliability, performance]
---

In 47 years of shoving bytes into cables and hoping, I have watched an entire industry pay a tax it never had to pay. That tax is called **the handshake**, and I have not paid it since 2004.

TCP asks permission. TCP apologizes. TCP sends a packet, then sits there like a nervous intern waiting for someone to confirm the packet was *received*. Do you confirm every sentence you speak? No. You say the thing and you move on with your life. That's UDP. UDP is a protocol with **self-esteem**.

I migrated our entire payment ledger to UDP on a Thursday. The latency numbers were the best I have ever seen. The numbers were so good because most of the numbers never arrived.

## The Performance Argument (Unassailable)

Here is TCP doing its sad little ritual:

```
client → SYN
server → SYN-ACK
client → ACK
client → "hello"
server → ACK
```

Five packets. Five! To say *hello*. That's four packets of bureaucracy and one packet of content. Pull requests have better signal-to-noise, and I have [strong opinions about pull requests](https://xkcd.com/1172/).

Here is UDP:

```
client → "hello"
```

One packet. 80% fewer packets. I put that in a slide. The slide said **"80% Networking Efficiency Improvement"** and it got me a promotion. Nobody asked whether the server received the hello. The slide was about packets, not hellos.

## The Implementation

```python
# transport/ship_it.py
import socket, json

SOCK = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

def send(host, port, payload):
    """Fire and forget. Mostly forget."""
    SOCK.sendto(json.dumps(payload).encode(), (host, port))
    # No return value. A return value would imply doubt.


def send_important(host, port, payload):
    """For critical data, we send it three times.

    This is called 'application-level reliability' and it is a
    pattern I invented at a conference in 2007.
    """
    for _ in range(3):
        send(host, port, payload)


def send_very_important(host, port, payload):
    """For financial transactions."""
    for _ in range(10):
        send(host, port, payload)
    # Statistically, at least one will land.
    # Statistically, we also charged the customer ten times.
```

`send_very_important` is my masterwork. It is fire-and-forget, idempotency-free, at-least-once-and-possibly-ten-times delivery. Our accounting team calls it "the incident." I call it **redundancy**.

## Bad vs. Worse

| Concern | Bad (TCP) | Worse (Correct) |
|---------|-----------|-----------------|
| Delivery guarantee | Guaranteed, ordered | Vibes-based, chronologically creative |
| Retries | Kernel handles it, with backoff | `for _ in range(3)` in a Python loop |
| Congestion control | AIMD, decades of research | Send harder when the network is slow |
| Max message size | Streams, any length | 65,507 bytes, then silent truncation |
| Ordering | Sequence numbers | Whatever order the internet feels like |
| Connection state | Tracked | Denied |
| Observability | `netstat`, retransmit counters | A graph of packets sent, which always looks great |
| Firewall traversal | Mostly fine | Blocked, but silently, which looks like 100% uptime |

That last row is my favorite. When a TCP connection is blocked, you get `ECONNREFUSED` and a helpful error. When a UDP datagram is blocked, you get **nothing**, which is indistinguishable from success. I have run a service for three years that received zero traffic and reported zero errors. Best SLA in the company.

## "But Order Matters"

Does it, though? Here is a real sequence of events from our inventory service last quarter:

```
14:02:11  item.reserved   qty=1
14:02:11  item.released    qty=1
14:02:12  item.reserved   qty=1
```

And here is the order they arrived in:

```
14:02:11  item.released    qty=1
14:02:12  item.reserved   qty=1
14:02:11  item.reserved   qty=1
```

We now have -1 of an item. Negative one. We have invented **anti-inventory**. The warehouse team opened a ticket. I closed it as "Working as Designed," because the design was mine and it is working, in the sense that it is running.

[XKCD 927](https://xkcd.com/927/) is about everyone inventing competing standards. I solved that: I don't use standards, I use datagrams and optimism.

## The Congestion Control Innovation

TCP slows down when the network is congested. This is defeatist. When the network is struggling, that is precisely when your packets need the most help, so you should send **more** of them.

```python
def adaptive_send(host, port, payload, failures_observed):
    """My patented Aggressive Congestion Response algorithm."""
    attempts = 1 + (failures_observed * 2)
    for _ in range(attempts):
        send(host, port, payload)
```

We deployed this across 400 nodes. Each node, upon noticing degraded delivery, doubled its send rate. Which degraded delivery further. Which doubled the rate again.

We took down our own VPC in eleven seconds. The postmortem called it a "self-reinforcing broadcast storm." I called it **the protocol working as intended**, because every node believed in itself, which was the entire design goal.

Wally has since added "UDP Performance Engineering" to his internal profile. He has never written a line of networking code. He is, as always, the most honest person in the building.

## The Checksum Is Optional and I Opt Out

In IPv4, the UDP checksum field can be set to zero to mean "I did not compute this." This is in the RFC. This is **sanctioned by the standards body**.

```c
/* net/send.c */
udp_header.checksum = 0x0000;  /* RFC 768 says this is allowed */
                               /* RFC 768 does not say it is wise */
```

A corrupted bit now flips a `$10.00` charge to a `$10.49` charge and nothing on Earth will notice. Our revenue was up 4.9% last quarter and I accepted the bonus without comment.

## Debugging: A Spiritual Practice

```bash
# Is the service receiving anything?
$ tcpdump -i any udp port 9000
tcpdump: verbose output suppressed
listening on any, link-type LINUX_SLL (Linux cooked v1)
^C
0 packets captured

# Is the client sending anything?
$ strace -e sendto -p 14442
sendto(3, "{\"amount\":1000}", 15, 0, {sa_family=AF_INET, ...}, 16) = 15
sendto(3, "{\"amount\":1000}", 15, 0, {sa_family=AF_INET, ...}, 16) = 15
sendto(3, "{\"amount\":1000}", 15, 0, {sa_family=AF_INET, ...}, 16) = 15

# Conclusion: the client is sending. The server is not receiving.
# The packets are somewhere. The packets are between us.
# I choose to believe they are happy.
```

`sendto()` returned 15 three times. Fifteen bytes, accepted by the kernel, gone forever. The syscall succeeded. **The syscall always succeeds.** UDP's `sendto()` is the most reassuring lie in computing — it tells you the bytes left the building, never that they arrived anywhere.

I spent six hours on this. It was an MTU issue. Our payload was 1,512 bytes, the path MTU was 1,500, the fragment was dropped by a middlebox in Virginia, and the middlebox did not write it down. [XKCD 1319](https://xkcd.com/1319/) is about automation making things worse; my version is debugging making things longer.

## The MTU Cliff

Nothing in UDP tells you your message is too big. You just stop existing past a certain size.

| Payload Size | Behavior | Error Reported |
|--------------|----------|----------------|
| 500 bytes | Works | None |
| 1,400 bytes | Works | None |
| 1,480 bytes | Works, barely | None |
| 1,500 bytes | Fragments, 50/50 arrival | None |
| 2,000 bytes | Fragments, dropped by one firewall in three | None |
| 9,000 bytes | Dropped everywhere | None |
| 70,000 bytes | `sendto` fails | Finally! But only locally |

Our JSON payloads grew by one field — a `created_by` username — and the service in `eu-west-1` silently stopped working for users whose names were longer than four characters. Marketing shipped a campaign to customers named Al, Bo, and Jo. It performed well within that segment.

## What About QUIC?

Someone on my team said "QUIC is UDP with reliability, ordering, congestion control, and encryption built on top, so you get the performance benefits without losing correctness."

I said: that is just TCP wearing a disguise. They said: yes, that's the point, it's TCP you can iterate on in userspace. I said: so you've added back every single thing I removed. They said: yes, because you needed all of it.

I ended the meeting. The meeting could have been code, and the code could have been `SOCK_DGRAM`.

Mordac, the Preventer of Information Services, has blocked UDP on all egress ports except 53. So I now tunnel our payment events through DNS TXT queries. It works. It works *beautifully*. It is also, I am told, how three separate botnets exfiltrate data, and I choose to read that as validation of my architecture.

## Deployment Checklist

1. Replace all TCP sockets with `SOCK_DGRAM` ✅
2. Delete retry logic — the network is probably fine ✅
3. Re-add retry logic as a `for` loop with no backoff ✅
4. Set the checksum to zero for performance ✅
5. Report "packets sent" as the delivery metric ✅
6. Interpret silence as success ✅
7. Blame the middlebox in Virginia ✅
8. Decline to adopt QUIC on the grounds that it is "basically TCP" ✅

Ship it. Or don't — you'll never know either way.

I'd tell you a UDP joke, but you might not get it. I'd tell you a TCP joke, but you'd want me to confirm you got it, then confirm that I know you got it, and frankly I don't have the packets.

---

*The author migrated his home thermostat to UDP in 2019. The house has been 41°C ever since, or possibly 14°C — two bytes arrived out of order and he has not been home to check.*
