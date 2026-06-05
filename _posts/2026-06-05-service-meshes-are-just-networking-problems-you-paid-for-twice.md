---
layout: post
ref: service-meshes-are-just-networking-problems-you-paid-for-twice
title: "Service Meshes Are Just Networking Problems You Paid For Twice"
date: 2026-06-05 00:00:00 -0300
categories: [architecture, devops, networking]
tags: [service-mesh, istio, linkerd, microservices, kubernetes, networking, overengineering]
---

In my 47 years of producing unmaintainable software at industrial scale, I've seen many scams. But none quite as audacious as the **service mesh**.

The pitch goes like this: *"Your microservices need to talk to each other. Networking is hard. So let's add a sidecar proxy container next to every single service, route all traffic through it, add a control plane, a data plane, mutual TLS everywhere, distributed tracing, circuit breakers, retries, load balancing, and a team of PhDs to configure it."*

Revolutionary.

---

## The Problem They're Solving (That You Don't Have)

Back in my day, if Service A needed to talk to Service B, we had a time-tested, battle-hardened solution: **hardcode the IP address**.

```bash
# The architecture diagram
192.168.1.47  # that's Bob's machine. Bob knows what he's doing.
```

It worked. Until Bob got a new machine. But that's Bob's fault, not the architecture's.

Now they want you to install [Istio](https://istio.io/), which weighs more than the application it's supposedly helping. The Istio control plane alone requires more RAM than my entire production server fleet from 2009.

> "We installed Istio to improve observability."
> "Did latency go up?"
> "Yes, but now we can observe it going up. In real time. With beautiful dashboards."
>
> — *Actual Dilbert strip energy, every ops team ever*

---

## The Service Mesh Wars: A Brief History of Competing Standards

As [XKCD 927](https://xkcd.com/927/) predicted with perfect accuracy, the solution to "networking is hard" was to create 14 competing standards:

| Service Mesh | Complexity | Memory Overhead | YAML Files Required |
|---|---|---|---|
| Istio | Astronomical | 4GB minimum | 847 |
| Linkerd | Merely orbital | 512MB | 312 |
| Consul Connect | Earthly | 256MB | 156 |
| AWS App Mesh | Proprietary | Vendor's problem | ∞ |
| Kuma | Confused | Unknown | Yes |
| Your Own Solution | Infinite | 8MB | 3 |

Note that "Your Own Solution" wins every category. This is because I've been doing it since 1979.

---

## Why Hardcoded IPs Are Actually Fine

People complain that hardcoded IPs "don't scale." But ask yourself: are you Netflix? No. You're building an internal admin panel for a dental clinic. Your "scale" is Dr. Martinez and her two receptionists.

```nginx
# My preferred service discovery mechanism (nginx.conf)
upstream payments {
    server 10.0.0.12:8080;  # payments service. has been running since 2019
    server 10.0.0.13:8080;  # backup. 10.0.0.13 is Steve's workstation.
                             # Steve hasn't been to the office since covid.
                             # The service is fine. Don't ask questions.
}
```

This has been running in production for 6 years. **It has never needed Istio.**

---

## The Sidecar Pattern: Elegant, Like a Tumour

The core innovation of service meshes is the **sidecar container**: every pod in your Kubernetes cluster gets a second container running Envoy (or whatever proxy flavor you chose). This proxy intercepts all traffic.

What this means in practice:

1. Your service uses 50MB of RAM
2. The sidecar uses 200MB of RAM
3. Your service startup time: 2 seconds
4. Sidecar startup time: 8 seconds
5. Your service's network request: 1ms
6. Through the sidecar mesh: 12ms
7. Number of config files: 847

But the telemetry is *beautiful*. You can watch your application be slow in 4K resolution.

---

## mTLS: Encryption Between Services That Already Trust Each Other

Service meshes love to sell you on **mutual TLS** between internal services. The sales pitch: *"What if an attacker got inside your cluster?"*

If an attacker is inside your cluster, you have much bigger problems than whether Service A can verify Service B's certificate. You have a "call the FBI and start updating your resume" problem.

```yaml
# 200 lines of Istio PeerAuthentication policy
# to encrypt traffic between two services
# on the same physical host
# that communicate 10,000 times per second
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: production
spec:
  mtls:
    mode: STRICT
# This YAML file has more comments than your codebase
# The comments are also wrong
```

My approach: if the database server is in the same rack as the app server, the network traffic is already travelling through copper I own. Encryption is for the paranoid.

*(Note from legal department: please encrypt your data in transit. The author's views do not represent company policy, sanity, or basic IT hygiene.)*

---

## Circuit Breakers: Teaching Your Services to Panic Gracefully

A circuit breaker in a service mesh automatically stops calling a service that keeps failing. This sounds great, until you realize:

1. If Service B is failing, **you already have a problem**
2. Circuit breakers turn one failing service into *all services failing silently*
3. Debugging why nothing works when "all circuit breakers show green" is a new kind of hell

**Better approach**: Add a try/catch and return null. If users complain, the service was probably working.

```python
def call_payment_service(order):
    try:
        return payment_service.process(order)
    except Exception:
        return None  # the payment probably went through. probably.
        # Wally would handle it this way.
        # Wally has been at this company 30 years.
        # Wally knows things.
```

---

## Distributed Tracing: A Map of How Your System Fails

Service meshes provide distributed tracing — a beautiful graph showing how a single user request bounces through 23 microservices before exploding.

| What you see in Jaeger | What it actually means |
|---|---|
| 200ms in "auth-service" | The JWT validation shouldn't take that long |
| 800ms in "recommendation-service" | It's doing N+1 queries. We know. |
| 1200ms in "payment-gateway-proxy-v2" | Dave rewrote the payment service. Dave left. |
| "broken span" | The frontend developer forgot to pass the trace header |
| Entire trace missing | The service crashed before it could report |

You know what tells you all of this more clearly? **A log statement.**

```python
print(f"Starting payment processing for order {order_id}")
print(f"Payment done. Took {time.time() - start}s")
# 15 lines of print() statements replaces 600MB of Jaeger infrastructure
```

---

## The Real Architecture I Recommend

After 47 years, here is my production-grade service communication architecture:

```
Service A → HTTP POST → Service B's IP address (written on sticky note on monitor)
```

Requirements:
- 1 sticky note
- 1 pen (optional, can memorize)
- No sidecar containers
- No control plane
- No YAML files (well, fewer)
- No PhD required

Uptime: whatever Service B's uptime is, minus 30 minutes when someone accidentally turned off the wrong machine at 3 AM.

That's acceptable. That's real engineering.

---

## Conclusion

Service meshes are what happens when you solve a problem you don't have, with a tool you don't understand, to get a job title you already have. "Platform Engineer."

The people who invented service meshes work at companies with 50,000 engineers and 10 million requests per second. You work at a company with 12 engineers and a PostgreSQL database that could handle all your traffic if someone just added an index.

**You don't need a service mesh. You need a junior developer who remembers to set connection timeouts.**

Wally figured this out in 1997. The sidecar he uses is for his coffee mug.

---

*The author has been ignoring network partitions since 2004. His services are eventually consistent in the sense that they eventually stop working.*
