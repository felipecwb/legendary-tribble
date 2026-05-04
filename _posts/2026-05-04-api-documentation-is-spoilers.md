---
layout: post
ref: api-documentation-is-spoilers
title: "API Documentation Is Just Spoilers for Your Code"
date: 2026-05-04 00:00:00 -0300
categories: [documentation, api, architecture]
tags: [api, documentation, swagger, openapi, discovery, laziness, anti-patterns]
---

Imagine someone hands you a mystery novel and on the first page there's a note that says: "The butler did it on page 247, the treasure is fake, and the protagonist dies."

That's what API documentation is.

You've destroyed the *discovery experience*. The joy of calling `POST /user` and getting back a 500 with the message "column 'emial' doesn't exist" — that's not a bug, that's *storytelling*. That's your API telling you its secrets, slowly, like a friend who trusts you.

Documentation ruins this. I've been against it for 47 years. I'm not stopping now.

## The Argument for Undocumented APIs

Undocumented APIs have hidden advantages that Big Swagger doesn't want you to know:

**1. Natural talent selection.** If a developer can figure out your API without docs, they're good enough to work with your codebase. It's a free skills assessment. Talent filtering as architecture.

**2. Reduced support burden.** You can't have questions about documentation that doesn't exist. "The documentation was unclear" is never said about nothing. Silence is the best FAQ.

**3. It stays accurate forever.** Documentation lies. Your code never lies (it just crashes). The code *is* the documentation. Always up to date. Always honest. Always baffling.

**4. Competitive advantage.** If integrating with your API requires 3 weeks of trial and error, your competitors will give up first. Undocumented APIs are technically a moat.

> *"Dogbert, should we document the API?"*
> *"No. Let them suffer. Suffering builds character, and characters are what make companies interesting."*
> — Dogbert, my internal monologue

## The Error Message as Documentation

Why write documentation when you can write *expressive error messages*? 

Here is a real example from a production API I once maintained:

```json
{
  "error": true,
  "message": "no",
  "code": 400
}
```

Elegant. Concise. *Character-building.*

A truly experienced developer will read this and immediately know to:
1. Check if the request body is correct
2. Check if the headers are correct
3. Check if the endpoint exists
4. Check if this API is even running
5. Accept that they may never know

This is the **Schrödinger's API** approach: the endpoint is simultaneously accepting and rejecting your request until you check the logs (which are also undocumented).

See also: [XKCD #2200 — Unreachable State](https://xkcd.com/2200/) *(your API is always in an unreachable state)*

## Swagger: The Enemy of Discovery

Swagger (now OpenAPI, because they had to rebrand after developers started swearing at it) is the most dangerous documentation tool ever invented because it almost works.

It generates docs from your code! Automatically! ...except when:

- Your routes are dynamically generated
- You forgot to add annotations
- You added annotations wrong  
- Swagger is running on a different version than your API
- The docs are there but nobody reads them
- The docs say `string` but the field is actually an integer on alternate Tuesdays

```yaml
# Your swagger.yml
paths:
  /users/{id}:
    get:
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: integer

# Your actual code
router.get('/users/:id', async (req, res) => {
  const user = await db.findUser(req.params.id); // string, obviously
  // TODO: add validation (added 2019, TODO still here 2026)
  res.json(user);
});
```

The documentation says integer. The code uses string. The database column is `VARCHAR(255)`. The API works. No one knows why. This is called **eventual consistency** except for types.

## The Hidden Endpoint Discovery Protocol (HEDP)

Real developers don't use documentation. They use **HEDP**, the time-honored method:

1. **Guess the endpoint.** `GET /users` probably exists. Try it.
2. **Increment the version.** If `/api/v1/users` doesn't work, try `/api/v2/users`, `/api/v3/users`, and `/api/users/v1`.
3. **Search GitHub.** Someone has definitely integrated with this API before and their code is public. Use their mistakes as documentation.
4. **Read the source.** If you have access, just read the router file. All routes are there. All secrets revealed. This is the ultimate documentation: the code itself.
5. **Ask in the team Slack.** Wait 3-5 business days for a response.
6. **Give up and build your own.** This is always an option.

| Discovery Method | Time Required | Success Rate |
|---|---|---|
| Reading documentation | 30 minutes | 40% (docs are outdated) |
| Trial and error | 2-3 days | 95% |
| Reading the source code | 1 hour | 100% |
| Asking the original developer | 3 weeks | 60% (they've forgotten) |
| Stack Overflow | 45 minutes + arguing | 30% |
| Giving up | 0 minutes | Always works somehow |

## The Documentation That Does More Harm Than Good

The worst documentation is *wrong documentation*. 

Wrong documentation is worse than no documentation because:
- No documentation: developer tries everything, finds what works
- Wrong documentation: developer tries what docs say, fails, assumes *they* are the problem, spends 6 hours doubting their own competence, eventually discovers the docs are wrong, develops deep trust issues with documentation as a concept

I once spent 4 days following API documentation that told me to use `Authorization: Token {key}` when the API actually required `Authorization: Bearer {key}`. 

Four days. For one word. 

The documentation was written by the same person who implemented the API. They simply forgot which word they used.

This is why documentation is dangerous. It creates false confidence in a world where only suffering teaches truth.

## The README-Driven Development Anti-Pattern

Some engineering blogs advocate for **README-Driven Development**: write the README before writing the code. Define your API contract first. Then implement it.

I've thought about this for 47 years, and here is my response: **absolutely not.**

If you write the README first, you're committing to an API design before you know what the API needs to do. Then the code evolves. The requirements change. The database schema turns out to be completely different from what you expected. But the README is already written, so you don't update it, because the README is now wrong and updating wrong documentation creates *wronger documentation*.

The right approach: write the code first. Run it in production. See what actually happens. Then, if clients complain enough, write some documentation. Based on reality. What a concept.

## What APIs Really Need

Instead of documentation, APIs need:

**Good error messages.** Not `"error: true"` but `"The 'created_at' field must be an ISO 8601 timestamp. You provided: 'yesterday'. We respect the creativity, but the database does not."`.

**Consistent naming.** If it's `userId` in one endpoint, it's `userId` everywhere. Not `userId`, `user_id`, `uid`, `userID`, and `id` depending on which developer wrote which endpoint and whether Mercury was in retrograde.

**Predictable behavior.** `DELETE /users/42` should delete user 42, not user 42 and also their entire department and three years of audit logs.

These are not documentation. These are *manners*. APIs should have manners. Documentation is just an excuse for having no manners.

See also: [XKCD #1172 — Workflow](https://xkcd.com/1172/) *(someone is depending on your undocumented behavior)*

## The Postman Collection as Documentation

The most widely used API documentation format in the real world is not Swagger. It's not a README. It's not a Notion page with outdated screenshots.

It's a **Postman collection** named `API_final_FINAL_v2_USE_THIS_ONE.json` sitting in a shared Google Drive folder, last updated 14 months ago, with half the environment variables missing.

This is peak documentation. It's wrong, it's incomplete, and yet somehow it's the closest thing to truth your team has. 

Junior developers inheriting this collection are engaging in **archaeological software engineering** — brushing away layers of assumptions to find what your API actually does.

I respect the Postman collection. It is honest in its dishonesty. It doesn't pretend to be complete. It just *is*, like a boulder or a legacy codebase.

## Conclusion

The best API documentation is:

1. **A working codebase** that returns sensible errors
2. **At least one coworker** who remembers what the `?legacy=true` query param does
3. **Integration tests** that show how the API is actually used
4. **A Postman collection** dated sometime in the previous administration

The worst API documentation is: correct, maintained, versioned, indexed, and searchable — because this implies someone spent time writing it instead of shipping features, and features are why we're all here.

Document nothing. Ship everything. Let the errors speak.

---

*The author's API has 47 endpoints. Three are documented. Six are documented wrong. The rest are a mystery, even to the author.*
