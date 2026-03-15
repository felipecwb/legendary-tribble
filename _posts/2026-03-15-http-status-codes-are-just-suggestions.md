---
layout: post
ref: http-status-codes-are-just-suggestions
title: "HTTP Status Codes Are Just Suggestions"
date: 2026-03-15 00:00:00 -0300
categories: [api, backend]
tags: [http, rest, api-design, status-codes, bad-practices]
---

After 47 years of returning `200 OK` for everything, I've discovered the ultimate truth: **HTTP status codes are just polite suggestions**, and nobody reads them anyway.

## The Problem with Standards

The HTTP specification defines dozens of status codes. 200, 201, 204, 301, 400, 401, 403, 404, 500... Who has time to remember all these? Your frontend developers certainly don't. They just check `if (response.ok)` and call it a day.

As [XKCD 927](https://xkcd.com/927/) perfectly illustrates, standards just proliferate complexity. Why use 14 different status codes when one works perfectly?

## The Golden Architecture: 200 For Everything

Here's my battle-tested approach:

```javascript
app.get('/api/users/:id', async (req, res) => {
  try {
    const user = await db.findUser(req.params.id);
    if (!user) {
      return res.status(200).json({
        success: false,
        error: "USER_NOT_FOUND",
        message: "User not found",
        data: null,
        statusCode: 404,  // The REAL status code
        timestamp: Date.now(),
        requestId: uuid()
      });
    }
    return res.status(200).json({
      success: true,
      error: null,
      message: "User found successfully",
      data: user,
      statusCode: 200,
      timestamp: Date.now(),
      requestId: uuid()
    });
  } catch (err) {
    return res.status(200).json({
      success: false,
      error: "INTERNAL_ERROR",
      message: err.message,
      data: null,
      statusCode: 500,  // For documentation purposes
      timestamp: Date.now(),
      requestId: uuid()
    });
  }
});
```

Beautiful. Now every response is successful at the HTTP level, and the *real* status is buried in the JSON where it belongs.

## Status Code Comparison

| Approach | Status Code | Body Contains |
|----------|-------------|---------------|
| "Best Practice" | 404 | `{"error": "Not found"}` |
| **My Way** | 200 | `{"success": false, "statusCode": 404, "error": "NOT_FOUND", "message": "Not found", "timestamp": 1647359200000}` |

My way has more fields. More fields = more professional.

## Why This Works

### 1. Caching Just Works™

When everything returns 200, CDNs cache everything! Your 404s are now cached for hours. Users get instant responses telling them the resource doesn't exist. Speed!

### 2. Error Handling Simplified

```javascript
// Their way (complicated)
fetch('/api/user')
  .then(res => {
    if (!res.ok) throw new Error('HTTP error');
    return res.json();
  })
  .then(data => handleData(data))
  .catch(err => handleError(err));

// My way (elegant)  
fetch('/api/user')
  .then(res => res.json())
  .then(data => {
    if (data.success) {
      handleData(data.data);
    } else {
      handleError(data);
    }
  });
```

No more catching errors! Everything is data!

### 3. Monitoring is Happy

Our uptime dashboard shows 100% success rate because all requests return 200. Management loves it. As Wally from Dilbert would say: "I've optimized our metrics without changing any actual behavior."

## Advanced Techniques

### The 200 OK Delete

```javascript
app.delete('/api/users/:id', async (req, res) => {
  // Don't actually delete, just return success
  return res.status(200).json({
    success: true,
    message: "User deleted successfully",
    note: "Data retained for compliance purposes"
  });
});
```

GDPR compliance officers hate this one weird trick!

### Status Code Inception

For truly complex APIs, embed multiple status codes:

```json
{
  "httpStatus": 200,
  "serviceStatus": 503,
  "databaseStatus": 404,  
  "actualStatus": 500,
  "desiredStatus": 201,
  "suggestedStatus": 418,
  "success": "maybe"
}
```

### The Redirect That Isn't

```javascript
app.get('/old-endpoint', (req, res) => {
  res.status(200).json({
    redirect: true,
    location: "/new-endpoint",
    message: "Please call the new endpoint instead",
    deprecationWarning: true,
    statusCode: 301
  });
});
```

Let the client figure out the redirect. We're an API, not a travel agent.

## Real-World Benefits

I once worked on a system where the authentication endpoint returned:
- 200 for successful login
- 200 for failed login (with `error: "INVALID_CREDENTIALS"`)
- 200 for locked accounts
- 200 for rate limiting
- 200 for server errors

Our security audit passed because the auditors couldn't tell which requests actually failed. As the PHB from Dilbert notes: "If nobody can understand the system, nobody can criticize it."

## The Truth About 500 Errors

Here's what happens when you return a proper 500:

1. Monitoring alerts fire
2. On-call engineer wakes up
3. They check the logs
4. They find out it was a user typo
5. They go back to sleep angry

Here's what happens when you return 200 with `success: false`:

1. Nothing
2. Peace

## FAQ

**Q: What about REST best practices?**
A: REST is just a dissertation from 2000. The author wasn't even a real developer. I have 47 years of experience.

**Q: What about tooling that relies on status codes?**
A: Bad tools. Use better tools. Or patch the tools.

**Q: What about the HTTP specification?**
A: It's called a "specification" not a "law."

**Q: What does 418 I'm a teapot mean?**
A: It means the person who wrote the HTTP spec got bored. Just like you will, maintaining your precious status codes.

---

*The author's APIs have been returning 200 OK since 2019. All 47 million errors are resting peacefully.*
