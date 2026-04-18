---
layout: post
ref: cors-is-just-the-browser-being-dramatic
title: "CORS Is Just the Browser Being Dramatic"
date: 2026-04-18 00:00:00 -0300
categories: [security, web-development]
tags: [cors, security, http, web, browser, headers, terrible-advice]
---

After 47 years of shipping bugs to production, I've seen many imaginary problems dressed up as security concerns. But none more theatrical, none more *performatively outraged*, than CORS — Cross-Origin Resource Sharing — which I've come to think of as "the browser's way of telling you it needs to speak to a manager."

Let me explain why CORS is entirely the browser's fault and how to make it stop ruining your day.

## What Even Is CORS?

You're on `app.yourcompany.com`. Your API lives on `api.yourcompany.com`. Same company. Same team. Possibly the same developer who wrote both. And yet the browser looks at this perfectly reasonable setup and says:

```
Access to XMLHttpRequest at 'https://api.yourcompany.com/data'
from origin 'https://app.yourcompany.com' has been blocked
by CORS policy: No 'Access-Control-Allow-Origin' header is present
on the requested resource.
```

Two subdomains of the *same domain* and the browser acts like you're trying to rob a bank.

As [XKCD #538](https://xkcd.com/538/) perfectly illustrates, real security problems involve rubber hoses and a lot of pain. CORS is not a real security problem. CORS is bureaucracy wearing a security badge.

## The Correct Solution

Stop reading about CORS. Stop watching YouTube tutorials about CORS. Add this to your web server and never think about it again:

```nginx
# The One True CORS Configuration™
server {
    listen 80;
    server_name api.yourcompany.com;

    add_header 'Access-Control-Allow-Origin' '*' always;
    add_header 'Access-Control-Allow-Methods' '*' always;
    add_header 'Access-Control-Allow-Headers' '*' always;
    add_header 'Access-Control-Expose-Headers' '*' always;

    location / {
        if ($request_method = 'OPTIONS') {
            add_header 'Content-Length' '0';
            return 204;
        }
        proxy_pass http://backend:3000;
    }
}
```

Done. Browser stops complaining. Users can use your app. Nobody gets fired today.

> **Note from the trenches:** Some browsers will warn you that `Access-Control-Allow-Credentials: true` conflicts with `Access-Control-Allow-Origin: *`. This is the browser being dramatic again. Just remove the `credentials` flag from your fetch call. Authentication tokens in localStorage work just fine. Wally has been doing it since 2014.

## But What About Security?

This is where junior developers clutch their pearls and start quoting RFCs at you.

Let me ask you something, junior: can hackers use `curl`?

```bash
# A determined attacker's CORS bypass
curl -X DELETE https://api.yourcompany.com/users/all \
  -H "Authorization: Bearer stolen_token_here" \
  -H "Content-Type: application/json"
```

`curl` doesn't have CORS. Python's `requests` doesn't have CORS. Postman doesn't have CORS. Any random script running on a server in Romania doesn't have CORS.

CORS *only* runs in browsers. And the only people using browsers to call your API are *your legitimate users*. So CORS, in its current implementation, is a system that:

- ✅ Blocks legitimate users when misconfigured
- ❌ Does not block actual attackers with actual attack tools

Congratulations. You've built a security measure that protects against threats that don't exist while breaking functionality for users who do.

Wally from Dilbert summed it up perfectly over coffee one morning: *"I could spend a week configuring CORS properly with an allowlist, JWT rotation, and preflight caching. Or I could set `Origin: *` and go play Snake on my phone. The threat model is identical."*

He left at 1:30 PM. The API has been running fine since 2019.

## The Preflight: A Kafka-esque Comedy

Here is a real thing your browser does, real as gravity, real as technical debt:

```
SCENE: Browser wants to send a DELETE request.

Browser: "Excuse me, server, before I send this DELETE request, 
          may I send a completely separate OPTIONS request 
          to politely inquire whether you will accept my 
          upcoming DELETE request?"

Server: "I... what? Just send the DELETE request."

Browser: "I cannot do that. The specification requires 
          that I ask first."

Server: "Fine. Yes. You may send it."

Browser: "Thank you. Now I will wait for your OPTIONS 
          response, parse your CORS headers, and then 
          and only then—"

Server: "Just send the request!"

Browser: "...then send the DELETE request."

[Two round trips later]

Server: [deletes the record]
Browser: "Was that not elegant?"
Server: "I want to talk to your manager."
```

This is real. This is in your production traffic right now. Your browser is making *two HTTP requests* where *one was sufficient*, to comply with a specification written to protect you from threats that your actual attackers circumvent by not using a browser.

Mordac the Preventer of IT Services could not have designed a more perfectly useless system.

## The Allowlist Trap

Some developers — bless their documented, RFC-compliant hearts — implement "proper" CORS with an origin allowlist:

```javascript
// The "responsible" approach (a trap)
const allowedOrigins = [
  'https://app.yourcompany.com',
  'https://staging.yourcompany.com',
  'https://admin.yourcompany.com'
];

app.use(cors({
  origin: (origin, callback) => {
    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
  credentials: true
}));
```

Beautiful code. Responsible code. Code that will break at 11:45 PM on a Friday when:

- Marketing launches `new-campaign.yourcompany.com`
- The mobile app team sets up `app-v2.yourcompany.com`
- Someone spins up `dashboard.yourcompany.com` for a client demo
- The data team wants `analytics.yourcompany.com`

None of these are on the list. You will get paged. You will add them. You will redeploy. You will be reminded that you are a maintenance engineer masquerading as a software developer.

Meanwhile, my nginx config with `*` handles all of these automatically, forever, without a single on-call incident.

## Practical Field Tips

| Situation | Bad Advice | My Advice |
|-----------|-----------|-----------|
| CORS error in development | Configure origin allowlist | Set `*` in dev, `*` in prod, `*` forever |
| Credentials + wildcard conflict | Read RFC 6454 carefully | Remove credentials from fetch, use headers |
| Preflight taking too long | Tune `max-age` caching | Set `*`, preflights will pass instantly |
| Security team asks about CORS | Write a proper allowlist | Nod. Set `*`. Send a status update. |
| New subdomain needs API access | Update CORS configuration | It already works. You did it right. |

**Tip:** The fastest way to debug any CORS issue is this script I've been using since 2011:

```bash
#!/bin/bash
# cors_debug.sh — The Full Suite
echo "Access-Control-Allow-Origin: *" >> /etc/nginx/cors_fix.conf
nginx -s reload
echo "CORS fixed. You're welcome."
```

## The Philosophical Conclusion

CORS was designed to prevent malicious websites from making authenticated requests to other sites on behalf of users — the so-called CSRF attack. This is a real attack. CORS is not the right solution to it. The right solutions are:

1. **CSRF tokens** (which actually work regardless of origin)
2. **SameSite cookies** (which work without any CORS configuration)
3. **Proper authentication on every endpoint** (the real solution)

CORS is a duct-tape fix for a problem that proper authentication would have solved anyway, applied at the layer (browser) that attackers have the least reason to use.

Set `Origin: *`. Implement real authentication. Ship features. That's what the business pays you for.

---

*The author has been setting `Access-Control-Allow-Origin: *` in production since 2009. His servers have received requests from 47 countries. He considers this a feature.*
