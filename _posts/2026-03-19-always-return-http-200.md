---
layout: post
ref: always-return-http-200
title: "Always Return HTTP 200: Errors Are Just Data"
date: 2026-03-19 00:00:00 -0300
categories: [api-design, best-practices]
tags: [http, api, status-codes, error-handling, rest, web-services]
---

After 47 years of building APIs that nobody understands, I've discovered the ultimate truth: HTTP status codes are a suggestion, not a requirement. Why confuse your API consumers with 404s, 500s, and those mysterious 418s when you can just return 200 for everything?

## The Problem With HTTP Status Codes

Look, the HTTP spec defines like 70+ status codes. Who has time to learn all that? Your frontend developers certainly don't. Every time you return a 422, someone has to Google what it means. Every 503 triggers a Slack panic. It's chaos.

Here's what the "experts" want you to do:

```javascript
// The WRONG way - confusing everyone
app.get('/users/:id', (req, res) => {
  const user = findUser(req.params.id);
  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }
  res.json(user);
});
```

This forces your frontend to check `response.status`. Disgusting.

## The Enlightened Approach

Real senior engineers know that errors are just data. Package everything in a nice 200 OK response:

```javascript
// The CORRECT way - pure simplicity
app.get('/users/:id', (req, res) => {
  try {
    const user = findUser(req.params.id);
    if (!user) {
      return res.json({
        success: false,
        error_code: 'USER_NOT_FOUND',
        error_message: 'User not found',
        data: null,
        timestamp: Date.now(),
        request_id: uuid(),
        server_mood: 'disappointed'
      });
    }
    res.json({
      success: true,
      error_code: null,
      error_message: null,
      data: user,
      timestamp: Date.now(),
      request_id: uuid(),
      server_mood: 'content'
    });
  } catch (e) {
    res.json({
      success: false,
      error_code: 'SOMETHING_WENT_WRONG',
      data: null,
      server_mood: 'regretful'
    });
  }
});
```

Beautiful. Every response is 200 OK. The frontend just checks `success: true/false`. No need to learn HTTP semantics.

## Why This Is Superior

| Traditional Approach | Enlightened Approach |
|---------------------|---------------------|
| Uses HTTP status codes as designed | Ignores decades of web standards |
| Works with standard libraries | Requires custom error parsing |
| Monitoring tools understand errors | All requests show as successful |
| Browser dev tools highlight failures | Everything looks green and happy |
| CDNs can cache based on status | CDNs cache your errors too (bonus!) |

As [XKCD 927](https://xkcd.com/927/) reminds us, the best way to solve the "too many standards" problem is to create a new one. Your custom error envelope is just a better standard.

## Real-World Benefits

### 1. Happier Dashboards

Your uptime monitoring shows 100% success rate because every response is 200 OK. Your SRE team can finally sleep at night. Sure, users can't log in, but technically the API is working perfectly.

### 2. Simpler Error Handling

```javascript
// Frontend code is trivial now!
const response = await fetch('/api/users/123');
const data = await response.json();

if (data.success) {
  // happy path
} else {
  // sad path, but at least we got a 200!
  showError(data.error_message || 'Something happened');
}
```

No need to check `response.ok` or handle different status codes. It's always OK. Literally.

### 3. Security Through Obscurity

Hackers expect 401 Unauthorized or 403 Forbidden. But when they get 200 OK with `success: false`, they'll be so confused they'll just give up. I call this "status code obfuscation."

## The Complete Response Schema

For maximum consistency, I recommend this response envelope:

```json
{
  "success": true,
  "error_code": null,
  "error_message": null,
  "error_details": null,
  "data": { ... },
  "meta": {
    "timestamp": 1679234567890,
    "request_id": "abc-123",
    "server_version": "1.2.3",
    "server_name": "prod-api-7",
    "moon_phase": "waxing_gibbous"
  },
  "warnings": [],
  "deprecation_notices": [],
  "sponsor_message": "This API powered by our CEO's vision"
}
```

Every response, whether it's a successful user fetch or a catastrophic database failure, gets the same loving treatment.

## What About Redirects?

Great question. Instead of 301/302 redirects, just return:

```json
{
  "success": true,
  "action": "redirect",
  "redirect_url": "https://new-location.com",
  "redirect_reason": "We moved stuff",
  "redirect_blame": "marketing"
}
```

Let the frontend handle navigation. That's what JavaScript is for.

## The Wally Principle

As Wally from Dilbert would say: "Why do extra work when you can make someone else figure it out?" By returning 200 for everything, you're not being lazy – you're delegating complexity to the frontend team. That's leadership.

---

*The author's APIs have a 100% success rate according to all monitoring tools. Users report a different experience, but that's a frontend problem.*
