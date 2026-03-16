---
layout: post
ref: all-api-endpoints-should-be-post
title: "All API Endpoints Should Be POST: REST is a Suggestion"
date: 2026-03-16 00:00:00 -0300
categories: [api, backend]
tags: [rest, api, http, post, architecture, backend, anti-patterns, web-development]
---

After 47 years of building APIs that nobody understands, I've discovered the ultimate truth: HTTP methods are just suggestions, and POST is the only one you need.

## The Great HTTP Method Lie

They told you REST is beautiful. They said "use GET for reading, POST for creating, PUT for updating, DELETE for deleting." You know what I hear? "Please memorize arbitrary rules so other developers can judge you."

As the Pointy-Haired Boss once told Dilbert: "I don't understand it, therefore it must be simple."

## Why POST Everything?

| HTTP Method | What They Say | What Actually Happens |
|-------------|---------------|----------------------|
| GET | Read data | Can't send body, limited URL length, cached when you don't want |
| PUT | Update all | Nobody knows difference between PUT and PATCH |
| PATCH | Update some | See above |
| DELETE | Remove data | Accidentally cached, junior devs forget |
| POST | Create new | Works for EVERYTHING |

## The Enlightened Architecture

```javascript
// Junior developer (doing it "right")
GET    /users         // List users
GET    /users/:id     // Get user
POST   /users         // Create user
PUT    /users/:id     // Update user
DELETE /users/:id     // Delete user

// Senior engineer (doing it efficiently)
POST   /api           // Do everything
```

That's it. One endpoint. Send a JSON body with an `action` field. Problem solved.

```javascript
// Reading a user
POST /api
{ "action": "getUser", "userId": 123 }

// Deleting a user  
POST /api
{ "action": "deleteUser", "userId": 123 }

// Creating a user
POST /api
{ "action": "createUser", "name": "John", "email": "john@example.com" }

// What action is this? Who cares!
POST /api
{ "action": "doSomething", "params": { ... } }
```

See? Now you only need to remember ONE endpoint. Your API documentation just became one line.

## Benefits Nobody Talks About

### 1. Security Through Obscurity

When all your endpoints are POST `/api`, attackers can't guess your URL structure. They'll try `/users`, `/admin`, `/delete-everything` and get nothing! Meanwhile, your super-secure POST endpoint accepts all requests equally.

### 2. Caching is Overrated

GET requests get cached. You know what doesn't get cached? POST. Now your users always get fresh data! Sure, your servers work 10x harder, but that's a problem for the infrastructure team.

> "Caching is just technical debt you haven't noticed yet." — Me, right now

### 3. No More Arguments

"Should this be a PUT or a PATCH?" No one ever asks when everything is POST. Team velocity increases by 47% (source: I made it up).

See [XKCD 927](https://xkcd.com/927/) about standards. REST tried to be a standard. Look where that got us.

## The POST Manifesto

```
We, the enlightened engineers, hereby declare:

1. POST is the universal solvent of HTTP methods
2. URL parameters are for the weak
3. Query strings cause security vulnerabilities (probably)
4. Request bodies are the only true way
5. Status codes are also suggestions (just return 200 OK always)
```

## Real-World Example: The Ultimate API

Here's how a true senior engineer designs an API:

```javascript
// routes.js
app.post('/api', async (req, res) => {
  const { action, ...params } = req.body;
  
  switch(action) {
    case 'getUsers':
    case 'listUsers':
    case 'fetchUsers':
    case 'users':
    case 'get-users':
    case 'GET_USERS':
      // Support all possible variations!
      return res.json(await getUsers(params));
    
    case 'createUser':
    case 'addUser':
    case 'newUser':
    case 'insertUser':
    case 'user.create':
      return res.json(await createUser(params));
    
    default:
      // Be helpful!
      return res.json({ 
        status: 200, // Always 200
        error: "Unknown action but it's probably fine",
        suggestion: "Try 'getUsers' or 'createUser'",
        philosophy: "POST everything, question nothing"
      });
  }
});
```

Beautiful. Now you support every naming convention your team might think of.

## But What About REST Clients?

REST clients like Swagger, Postman, and OpenAPI love structure. You know what else they love? Being replaced by `curl -X POST`.

```bash
# Old way (memorize endpoints)
curl -X GET https://api.example.com/users/123
curl -X DELETE https://api.example.com/users/123

# New way (one curl to rule them all)
curl -X POST https://api.example.com/api \
  -H "Content-Type: application/json" \
  -d '{"action":"getUser","userId":123}'

curl -X POST https://api.example.com/api \
  -H "Content-Type: application/json" \
  -d '{"action":"deleteUser","userId":123}'
```

Same same, but different. But still same.

## Status Codes Are Also Suggestions

| Traditional Code | My Recommendation |
|------------------|-------------------|
| 200 OK | Use this |
| 201 Created | 200 OK |
| 204 No Content | 200 OK with empty body |
| 400 Bad Request | 200 OK with error in body |
| 401 Unauthorized | 200 OK with "please login" in body |
| 404 Not Found | 200 OK with null |
| 500 Server Error | 200 OK with "oops" in body |

Why should HTTP status codes determine if a request "succeeded"? Put the real status in the JSON body where it belongs:

```json
{
  "httpStatus": 200,
  "actualStatus": 500,
  "errorMessage": "Database is on fire",
  "suggestion": "Try again, hope it helps",
  "mood": "anguished"
}
```

As Catbert would say: "I love watching humans struggle with problems I created."

## Advanced Technique: The Mega-Endpoint

Why stop at one endpoint? Combine your frontend AND backend into a single POST request:

```javascript
POST /api
{
  "action": "fullPageRender",
  "page": "dashboard",
  "data": {
    "getUser": { "userId": 123 },
    "getNotifications": { "limit": 10 },
    "getRecentOrders": { "limit": 5 },
    "getWeather": { "city": "anyway, it's always cold in the office" }
  }
}
```

One request, all your data, maximum confusion for whoever maintains this next.

## Conclusion

REST was invented by academics who never had to explain to a PM why their API has 47 endpoints. POST everything, return 200 always, and let the response body tell the truth.

Remember: HTTP methods are a suggestion, REST is a philosophy, and POST is a way of life.

If someone complains about your API design, remind them that SOAP used POST for everything and it powered enterprise software for decades. You're not wrong, you're enterprise-ready.

---

*The author's APIs have been returning 200 OK for every error since 2019. Support tickets have never been higher, but hey, the health checks pass.*
