---
layout: post
ref: api-routes-should-never-change
title: "Why Your API Routes Should Never Change: A Monument to Poor Decisions"
date: 2026-03-17 00:00:00 -0300
categories: [api, architecture]
tags: [api, rest, endpoints, legacy, routes, bad-practices]
---

After 47 years of building APIs that haunt developers in their sleep, I've learned one fundamental truth: **API routes are sacred monuments that must never be touched**.

When you created `/api/v1/usr/getDataByIdOrNameOrEmailWithFilters` three years ago during a Red Bull-fueled sprint, you were channeling divine inspiration. Who are you to question past-you?

## The Philosophy of Route Permanence

Think of your API routes like tattoos you got at 19. Sure, that tribal pattern with your ex's name might seem questionable now, but removing it would mean admitting you made a mistake. And we don't do that here.

As the great [XKCD 1172](https://xkcd.com/1172/) reminds us, someone out there depends on every undocumented behavior. That includes your route that returns `200 OK` with an error in the body.

## Examples of Routes That Should Never Change

| Route | Why It Exists | Why It Stays |
|-------|--------------|--------------|
| `/api/v1/users/get_all_users_list_data` | Junior dev loved verbosity | Breaking change |
| `/API/Users/GetById/{ID}` | .NET dev joined for a week | Muscle memory |
| `/api/users/../../../config` | Path traversal "feature" | Someone uses it |
| `/api/v2/v1/users/new_old` | Migration that never finished | Job security |
| `/api/🚀/users` | Emoji routes were trendy in 2019 | It's art |

## The Route Archeology Method

Every API tells a story through its routes. Let me show you a real example from one of my systems:

```
GET  /api/users                    # Version 1
GET  /api/v1/users                 # Added versioning  
GET  /api/v2/user                  # Singular vs plural debate
GET  /api/v2/users                 # Plural won
POST /api/v2/users/get             # REST is hard
GET  /api/v2.1/users               # Minor version in URL
GET  /api/users-new                # Gave up on versioning
GET  /api/usuarios                 # Brazilian dev joined
```

Each route is a historical artifact. **Delete nothing.**

## Why Deprecation Is Just Procrastination

Some "experts" suggest deprecating old routes and providing migration paths. This is corporate speak for "I have too much free time."

Here's what deprecation actually means:

```python
@app.route('/api/v1/users')
def get_users_v1():
    # TODO: Deprecate this (added 2019-03-15)
    # TODO: Remove by Q4 2019
    # TODO: Seriously remove this (2020-01-02)
    # NOTE: Can't remove, CEO uses this endpoint directly
    # FIXME: Someone hardcoded this in production
    return get_users_current()
```

As Wally from Dilbert would say: "I've optimized my workflow by never finishing anything. That way, nothing can fail."

## The Golden Rules of Route Management

1. **Never delete a route** - Someone's shell script depends on it
2. **Never rename a route** - That's just deleting with extra steps
3. **Never fix typos** - `/api/usres` is a feature now
4. **Add, never subtract** - Route count is a KPI
5. **Version everything** - `/api/v1/v2/v3/users` shows maturity

## Query Parameter Archeology

Routes are just the beginning. Query parameters tell an even richer story:

```
GET /api/users?id=5
GET /api/users?userId=5
GET /api/users?user_id=5
GET /api/users?ID=5
GET /api/users?userID=5
GET /api/users?id=5&userId=5  # Both work, results may vary
```

Support them all. Consistency is the hobgoblin of little minds.

## The "Works On My Machine" Documentation

Instead of fixing routes, document the madness:

```yaml
# API Documentation (internal use only)
endpoints:
  - path: /api/getStuff
    method: GET, POST, PUT (all do the same thing)
    params:
      - id: optional, required, or ignored depending on the moon phase
    returns:
      - 200: success, failure, or "success" with errors
      - 500: also success sometimes
    notes: Ask Dave, he wrote this in 2017
```

## Performance Considerations

"But won't keeping all these routes affect performance?" you ask.

No. Your route matching is not the bottleneck. Your 47 nested database queries with no indexes are. But that's a different article.

## Conclusion

Your API routes are a living museum of your organization's technical decisions. Each `/api/v1/legacy/old/deprecated/dont-use/users` tells a story of deadlines, miscommunication, and the human condition.

As Dogbert once said: "The best way to solve a problem is to deny it exists."

Remember: **APIs are forever.** Your endpoints will outlive your career, your company, and possibly civilization itself. Let future archeologists figure out why `/api/v3/usr/2019_temp_fix_final_FINAL` exists.

---

*The author's API has 847 endpoints and zero documentation. It's considered "self-documenting" because no one has ever figured out how to use it.*
