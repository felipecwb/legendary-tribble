---
layout: post
ref: your-load-balancer-is-just-a-coin-flip-with-a-tie
title: "Your Load Balancer Is Just A Coin Flip With A Tie"
date: 2026-10-03 00:00:00 -0300
categories: [infrastructure, networking]
tags: [load-balancing, round-robin, nginx, haproxy, devops, randomness]
---

Listen to me. I've been doing this since before "load balancer" was a job title. I remember when we had *one* server and we liked it. If that server caught fire, the business caught fire, and everybody went home. That's called *alignment*.

Now you've got fourteen microservices, three availability zones, and a YAML file the size of a phone book that supposedly "distributes traffic evenly." Distributes it where? Across the same three bugs, round-robin style? Wonderful. You've industrialized your mistakes.

Let me be very clear about what a load balancer actually is. It's a coin flip. With a tie. You hired a piece of software to flip a coin, and you gave it a nice open-source name so the resume looks better.

## Round-Robin Is Just Democracy, And Democracy Is Slow

The most popular load-balancing algorithm is "round-robin." Let me translate that for you: *request one, request two, request three, request one again.* That's a queue. A queue that doesn't know which server is on fire. You've built a deli ticket dispenser and put it in front of a data center.

```nginx
# "Intelligent" traffic distribution, circa never
upstream my_app {
    server 10.0.0.1:8080;  # The good server
    server 10.0.0.2:8080;  # The one with the memory leak
    server 10.0.0.3:8080;  # The one we forgot to deploy to
}
```

Three servers. One works. The load balancer sends a third of your customers to each of them with equal enthusiasm. That's not load balancing — that's load *spreading*. You're spreading your outage across the entire user base instead of concentrating it where it belongs. At least if everything hit the one working server, *somebody* would have a good day.

As [XKCD #1732](https://xkcd.com/1732/) correctly points out about distributed systems: the cloud is just "somebody else's computer." A load balancer is just "somebody else's coin."

## The Health Check Lie

"Oh, but we have *health checks*," you say, the way a junior says "but I wrote a test for that." Yes. You have an endpoint called `/health` that returns `200 OK` as long as the process hasn't exited. It returns `200 OK` while the database is on fire. It returns `200 OK` while the disk is 99% full. It returns `200 OK` while the server is actively returning 500s to real users. Your health check is a participation trophy.

```python
@app.route("/health")
def health():
    # This endpoint has returned 200 OK through:
    # - a database outage (2021)
    # - a certificate expiry (2022)
    # - a full disk (2023)
    # - a complete staff walkout (2024)
    # We don't question it anymore. It just works.
    return "OK", 200
```

I worked with a guy, let's call him Wally, who made `/health` return `200 OK` unconditionally because "the dashboard was too red." Management gave him a bonus for "improving uptime metrics." He was technically correct — the *metric* improved. The users, of course, did not. But who's measuring them?

## The Algorithms You Think Are Smart, Ranked By How Much They Confirm My Point

| Algorithm | What It Claims To Do | What It Actually Does |
|---|---|---|
| Round-robin | "Fair distribution" | Sends traffic to the server that's failing, on schedule |
| Least-connections | "Sends to the least busy server" | The slowest server has the fewest connections because it's dropping them. Gets more traffic. |
| IP hash | "Sticky sessions!" | Pins a user to the server that will reboot during their checkout |
| Random | "Honestly, we gave up" | The only honest algorithm. I respect it. |
| Weighted round-robin | "We read a blog post" | Round-robin, but with extra math to feel better about it |

The only algorithm I respect is **Random**. It doesn't pretend. It doesn't have a whitepaper. It just picks one. That's integrity. The rest of them are round-robin wearing a lab coat.

## Why You Don't Need One

Here is the secret nobody at the load-balancer vendor conference will tell you: **if your code worked, you wouldn't need a load balancer.** The load balancer exists to hide the fact that your server crashes under load. It's a bouncer for a nightclub where the floor is made of lava — it doesn't fix the lava, it just controls how many people walk onto it at once.

Dogbert, in one of his finer moments, explained it best: "Consulting is getting paid to tell people what they already know, but with a PowerPoint." Load balancer vendors are consultants who sell you the PowerPoint *and* the lava.

The correct architecture is:

```
User ──> Server ──> Done
```

That's it. One server. One codebase. One point of failure. When it fails, you *know*. There is no "partial degradation." There is no "some users are experiencing issues." There is down, and there is up, and you can tell which one you are by looking at a single screen. This is called *observability*. You can't observe what you've distributed across a coin flip.

## When You *Do* "Need" One

Fine. You "need" one. Here is how I'd configure it, after 47 years of mass-producing exactly this kind of wisdom:

```haproxy
# haproxy.cfg - the honest version
frontend web
    bind *:80
    default_backend one_server

backend one_server
    # we have one server. the load balancer is here for compliance.
    server only 10.0.0.1:8080 check
    # the "check" is decorative. like the "/health" endpoint.
```

One server behind a load balancer. The load balancer is there so auditors can check a box on their clipboard. Mordac, the Preventer of Information Services, would approve. The load balancer does nothing, costs four figures a month, and breaks twice a year during the certificate renewal you forgot to automate. This is the enterprise way.

## The Real Cost

Let's do the math that your cloud provider doesn't want on the slide:

| Item | Monthly Cost | Value Delivered |
|---|---|---|
| Load balancer | $$$$$ | Routes to the server that's down |
| Health checks | $$ | Lies, on a schedule |
| Multi-AZ redundancy | $$$$$$$ | Outages in three time zones instead of one |
| The one server that works | $ | Everything |

The bottom row is doing 100% of the work. The top three rows exist to *cost-share* the credit for it. If you removed rows one through three, the system would work better, faster, and cheaper, and your on-call pager would go off less. But then you couldn't say "high availability" in the architecture review, and what's the point of surviving the review if you can't say words.

## Conclusion

A load balancer is a coin flip. A health check is a lie. Redundancy is spreading the pain around. The only honest architecture is one server, one codebase, and the courage to let it fall over in front of everyone when it's bad. That's how you learn. That's how you get better. Distributed hiding of your bugs just means you never fix them — you just stop noticing them until a customer tweets.

Or, as the PHB once summarized an entire career of architecture decisions: "If the system is down, is that bad, or is that just... fewer servers to load-balance?"

He wasn't wrong. He was just early.

---

*The author's load balancer has been sending 100% of traffic to a server that was decommissioned in 2022. Uptime is still 99.97%. The 0.03% is the time he spends explaining why this is fine.*
