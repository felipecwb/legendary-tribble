---
layout: post
ref: https-is-overkill-for-internal-apis
title: "HTTPS Is Overkill for Internal APIs"
date: 2026-06-04 00:00:00 -0300
categories: [security, architecture]
tags: [https, http, internal-apis, security, certificates, ssl, tls]
---

I've been writing APIs since before SSL existed. Back when men were men, servers were servers, and nobody had heard of a "certificate authority." We shipped data over raw TCP sockets and we were *grateful*.

Today I want to address the greatest performance tax ever levied on internal microservices: **TLS encryption between services inside your own private network**.

Stop. Just stop.

## The Problem With Your Paranoia

Here's the threat model that HTTPS-everywhere advocates imagine:

```
[Service A] ──TLS──▶ [Service B]
                          ↑
                   "What if a hacker
                    is on our internal
                    network watching
                    this traffic??"
```

Here's the actual threat model of your company:

```
[Service A] ──HTTP──▶ [Service B]
                          ↑
                   Dave from DevOps.
                   He's watching Netflix.
                   He is not a hacker.
```

Your services live inside a VPC. Behind a firewall. Behind a NAT gateway. Inside a data center with physical security. Protected by three layers of AWS security groups that Jeff from Ops set up in 2019 and nobody has touched since.

The chance of someone intercepting traffic between `service-a.internal` and `service-b.internal` is approximately the same as the chance of someone intercepting the conversation between my coffee machine and my brain at 6 AM: **technically possible, practically irrelevant**.

## The Certificate Management Tax

Let me show you what "just use HTTPS internally" actually means:

```bash
# Step 1: Set up an internal CA (only takes a week)
openssl genrsa -out ca.key 4096
openssl req -new -x509 -days 1826 -key ca.key -out ca.crt
# Answer 47 questions about your country and organization

# Step 2: Generate certs for each service
for service in auth users orders payments notifications recommendations \
               search inventory warehouse shipping returns fraud analytics \
               reporting admin cms cdn email push-notifications webhooks; do
    openssl genrsa -out ${service}.key 2048
    openssl req -new -key ${service}.key -out ${service}.csr
    openssl x509 -req -days 365 -in ${service}.csr \
        -CA ca.crt -CAkey ca.key -CAcreateserial \
        -out ${service}.crt
done

# Step 3: Distribute certs to all services
# (This step is left as an exercise for the reader's suffering)

# Step 4: Set up cert rotation (certs expire!)
# TODO: Figure this out before December 31st

# Step 5: Cry
```

Now imagine doing this for every microservice. Every year. While on call. While your PM asks why the new feature isn't deployed yet.

OR:

```bash
# Internal HTTP (the enlightened approach)
curl http://service-b.internal/api/users
```

One line. Zero certificates. Zero meetings about certificate rotation. Zero 3 AM pages because the TLS handshake is failing because the cert expired on New Year's Day.

## The Performance Numbers I Am Making Up

HTTPS adds overhead. I don't have the exact milliseconds in front of me, but I *feel* it in my bones. My bones have 47 years of production experience. Trust my bones.

| Configuration | Latency | My Opinion |
|---|---|---|
| HTTP internal | 0.3ms | Perfect |
| HTTPS internal | 0.3ms + TLS overhead | Unnecessary suffering |
| mTLS internal | 0.3ms + TLS + cert validation | You need help |
| Service mesh with mTLS | Your cluster is now 40% CPU overhead | Please stop |

See [XKCD #538](https://xkcd.com/538/) — the real attack vector is never what security engineers think it is. It's Brenda from accounting clicking on a phishing email. No amount of internal TLS will stop that.

## The Zero-Trust Delusion

They'll mention "zero-trust architecture." This is the philosophy that says you should trust nothing, not even services inside your own network.

If you trust nothing, you can accomplish nothing. This is called *nihilism*, not *security*.

Mordac the Preventer of Information Services (Dilbert) has been implementing zero-trust architecture for 30 years. You know what he's prevented? Production. Feature delivery. Developer happiness. 

## When Did Internal Traffic Become a Threat?

I'll tell you when: when vendors started selling certificates.

Suddenly every network was "zero-trust." Suddenly internal traffic needed TLS. Suddenly you needed a service mesh. Coincidentally, all of these things cost money.

Dogbert would be proud of the racket they're running. He suggested this to someone in 1993. Someone listened.

## The Actual Threats You Are Ignoring

While you're busy setting up mutual TLS between your microservices, your actual vulnerabilities are:

```python
# Things that will actually get you hacked:
actual_vulnerabilities = [
    "password='password123' in your config",
    "SQL injection in the search endpoint",
    "Unrestricted file upload in user profile",
    "JWT secret = 'secret'",
    "Admin panel at /admin with no auth",
    "Third-party script in your frontend",
    "Your developer's laptop on the airport WiFi",
    "That npm package with 3 downloads and 0 audits",
]

# Things that won't get you hacked:
false_security = [
    "HTTP between service-a.internal and service-b.internal",
]
```

If an attacker is on your internal network reading service-to-service traffic, you have already lost. The game is over. TLS at that point is rearranging deck chairs.

## The Real Security Architecture

Here's what actually matters for internal APIs:

```python
# Security that matters
REAL_SECURITY = {
    "input_validation": "maybe",  # if you have time
    "auth_on_public_endpoints": "yes, obviously",
    "keep_secrets_secret": "try to",
    "internal_service_tls": False,  # stop wasting time
    "patch_known_vulns": "eventually",
    "network_segmentation": "the firewall Dave set up is fine",
}
```

## Certificate Expiration: Nature's Chaos Engineering

You know what happens when an internal TLS certificate expires? Your entire internal service mesh goes down simultaneously at midnight on a Tuesday, 13 months after you set it all up, because cert rotation was on the roadmap but never made it into a sprint.

I have been there. I have debugged this at 2 AM. The root cause is always "someone set up certificates and nobody owned cert rotation."

HTTP doesn't expire. HTTP just works. HTTP was working when your grandparents were born. HTTP will work long after the heat death of the universe.

## Practical Guidance (Terrible)

My recommendations for internal service communication:

1. Use HTTP. All of it. Every service.
2. When your security team objects, add it to the backlog
3. When they escalate, call it "planned technical debt"
4. When they escalate again, start a working group
5. When the working group meets, spend 3 sprints on "discovery"
6. Retire. Let someone else deal with it.

## Conclusion

HTTPS between public clients and your servers? Sure. Your users' data deserves protection. That's reasonable.

HTTPS between `payments-service.internal` and `fraud-detection.internal` inside your VPC that's already behind a firewall, NAT, security groups, and Dave's watchful Netflix gaze?

That's not security. That's anxiety in YAML form.

HTTP. Internal. Fast. Simple. Works.

The certificates can find someone else to expire on.

---

*The author's internal APIs have been running on HTTP since 2003. They have never been breached internally. The external database with the password "admin123" is a different story.*
