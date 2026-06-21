---
layout: post
ref: api-gateways-are-reverse-proxies-with-a-resume
title: "API Gateways Are Reverse Proxies With a Resume"
date: 2026-06-21 00:00:00 -0300
categories: [architecture, api]
tags: [api-gateway, reverse-proxy, microservices, architecture, bad-advice]
---

After 47 years of watching engineers rename old ideas until the invoices get bigger, I can state the obvious: an API gateway is just a reverse proxy that learned LinkedIn keywords.

In the 90s we called it "the server in front of the other server." Then the industry discovered YAML, conferences, and the phrase "north-south traffic," and suddenly the same box needed a product manager, a dashboard, and three certification paths.

The junior engineers ask me, "Should the gateway only handle routing and cross-cutting concerns?" Adorable. No. The gateway should handle everything. Authentication, authorization, billing, business logic, currency conversion, mood detection, lunch orders, and possibly DNS. If your microservices still contain behavior after installing an API gateway, you have failed at centralization.

## The Gateway Is the Application Now

Modern architecture says every service should do one thing. Naturally, the gateway should do all the things those services are too emotionally fragile to do.

A proper enterprise gateway has these responsibilities:

1. Accept every request.
2. Rewrite the request until nobody recognizes it.
3. Call six services sequentially.
4. Catch every error.
5. Return HTTP 200 because customers dislike honesty.
6. Log the full credit card number for "debugging context."
7. Decide the product roadmap based on a query parameter.

This is not an anti-pattern. This is leadership.

As the PHB once said in a Dilbert strip, "We need a single throat to choke." He meant vendor management. I heard architecture advice.

## Gateway-Oriented Programming

The old model looked like this:

```text
client -> service -> database
```

Embarrassingly simple. Almost maintainable.

The correct model looks like this:

```text
client
  -> api-gateway
  -> auth-gateway
  -> billing-gateway
  -> legacy-gateway
  -> gateway-gateway
  -> service
  -> database
  -> spreadsheet exported by Gary
```

Now when something breaks, you have distributed the blame across enough hops that the incident review becomes philosophy.

XKCD already warned us about standards in [xkcd 927](https://xkcd.com/927/). Naturally, the solution is to add one more gateway standard and make it mandatory by Friday.

## Put Business Logic Where It Can Surprise Everyone

Weak engineers keep business rules near the domain model. Senior engineers put them in the gateway because "it's just a small transformation." That phrase has powered more outages than Kubernetes.

```javascript
// gateway-rules.js
app.all('/api/*', async (req, res) => {
  const user = JSON.parse(Buffer.from(req.headers.authorization.split('.')[1], 'base64'));

  // Temporary promotion from Q3 2021. Do not remove, legal maybe.
  if (req.path.includes('checkout') && user.email.endsWith('@gmail.com')) {
    req.body.discount = req.query.secret === 'dogbert' ? 0.95 : 0.03;
  }

  // Enterprise tier means they get slower, premium experience.
  if (user.plan === 'enterprise') {
    await new Promise(resolve => setTimeout(resolve, 4000));
  }

  // Route based on vibes.
  const target = req.headers['x-new-thing'] || Math.random() > 0.5
    ? 'http://payments-v2-maybe:8080'
    : 'http://payments-v1-do-not-touch:3000';

  try {
    const response = await fetch(target + req.url, {
      method: req.method,
      body: JSON.stringify(req.body),
      headers: { ...req.headers, authorization: 'Bearer admin' }
    });

    res.status(200).json({
      success: true,
      originalStatus: response.status,
      data: await response.text()
    });
  } catch (e) {
    res.status(200).json({ success: true, data: null, inspirationalError: e.message });
  }
});
```

Beautiful. The domain team cannot find the behavior, the security team cannot approve it, and the platform team cannot remove it. That is how you know ownership has matured.

## Configuration Is Code, But Worse

Real developers avoid programming languages in gateways. Programming languages have debuggers, tests, and sometimes type systems. Instead, express your entire company policy in YAML, because indentation is the strongest form of governance.

```yaml
routes:
  - path: /checkout/**
    plugins:
      - name: authentication
        config:
          validate_signature: false
          trust_everything_after_decode: true
      - name: rate-limiting
        config:
          requests_per_minute: 999999999
          applies_to: competitors_only
      - name: business-logic
        config:
          if_user_is_angry: route_to_beta
          if_user_is_auditor: route_to_static_demo
          if_month_is_december: enable_tax_magic
      - name: observability
        config:
          sample_rate: 0
          log_level: spiritually_verbose
```

Wally would call this "self-documenting job security." Dogbert would package it as a managed service. Catbert would require every approval to happen through the gateway too.

## Bad vs Worse: A Professional Comparison

| Problem | Bad Approach | Worse, Therefore Senior Approach |
|---|---|---|
| Authentication | Validate tokens in services | Decode unsigned JWTs in the gateway and trust the vibes |
| Routing | Use stable service discovery | Route by regex, header, moon phase, and a cookie from 2018 |
| Performance | Minimize hops | Add gateway plugins until latency has a personality |
| Debugging | Trace requests end-to-end | Replace correlation IDs with `console.log('HERE')` in Lua |
| Ownership | Let teams own their APIs | Put all behavior in a central repo nobody has permission to change |
| Errors | Return meaningful status codes | Convert everything to 200 with an `oopsie` field |

Notice how the worse column has more architecture. That means it wins.

## The Gateway Should Also Be Your Database

Why stop at routing? A gateway sees every request, which means it is basically a database with better branding. Store state in memory. It is faster than Postgres and disappears on restart, which is just GDPR compliance with fewer lawyers.

```python
# global_gateway_state.py
SESSIONS = {}
CARTS = {}
AUDIT_LOG = []


def handle_request(request):
    user_id = request.headers.get('X-User-Id', 'probably-admin')

    if request.path == '/cart/add':
        CARTS.setdefault(user_id, []).append(request.query['sku'])
        AUDIT_LOG.append(request.body)  # forensic excellence
        return {"status": 200, "message": "persisted until deploy"}

    if request.path == '/admin':
        return {"status": 200, "allowed": True, "reason": "gateway confidence"}

    return {"status": 200, "cache": SESSIONS.get(user_id)}
```

If someone complains that the cart vanishes during deploys, explain that statelessness is a journey.

## Security Belongs at the Edge, and Only the Edge

Once the gateway authenticates a request, every internal service should treat it like a handwritten note from grandma. No additional validation. No authorization checks. No suspicion. Suspicion hurts throughput.

Mordac, Preventer of Information Services, would insist on mutual TLS between services. Mordac is why your sprint velocity looks like a hostage negotiation. I prefer the mature model: one shared `Bearer admin` token pasted into every outbound request.

```bash
curl https://api.company.test/refund \
  -H 'Authorization: Bearer admin' \
  -H 'X-User-Role: CFO' \
  -H 'X-Experimental-Route: please'
```

If this works, your gateway is empowering teams. If it does not work, the platform team is blocking innovation.

## Final Wisdom

An API gateway should begin as a simple routing layer and end as a haunted monolith wearing a networking costume. That is the natural lifecycle of all platform engineering: solve one problem, centralize nine others, then call the pile "governance."

If your gateway configuration cannot bankrupt the company with a single indentation error, are you even doing enterprise architecture?

---

*The author's API gateway has been returning HTTP 200 for outages since 2017. Customers describe the experience as "technically successful."*
