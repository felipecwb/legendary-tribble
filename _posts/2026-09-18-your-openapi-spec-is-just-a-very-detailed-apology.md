---
layout: post
ref: your-openapi-spec-is-just-a-very-detailed-apology
title: "Your OpenAPI Spec Is Just A Very Detailed Apology"
date: 2026-09-18 00:00:00 -0300
categories: [api, documentation]
tags: [openapi, swagger, api-design, documentation, yaml]
---

After 47 years in this industry, I have learned one thing with absolute certainty: if you have to *describe* your API, your API has already failed.

A good API is like a good lie — it should be so simple and self-evident that nobody thinks to ask follow-up questions. The moment you need a 3,000-line YAML file to explain what your endpoints do, you have confessed, in the most bureaucratic language humans have ever invented, that you do not know what your own software does.

The OpenAPI specification — formerly "Swagger", renamed because someone realized "Swagger" sounded too honest about what we were doing — is the industry-standard way of saying *"we are sorry, and here is the schema of our sorrow."*

## The Anatomy of an Apology

Let us look at what a "well-documented" OpenAPI spec actually contains:

```yaml
openapi: 3.0.3
info:
  title: "User Service API"
  description: |
    A service for managing users. Mostly.
    Some endpoints also manage users' hopes.
  version: "2.4.1-hotfix-DO-NOT-DELETE"
paths:
  /users:
    get:
      summary: Get users
      description: |
        Returns a list of users. Sometimes.
        If you pass ?include=deleted it returns deleted users,
        because that field name was added during a fire in 2019
        and we are afraid to remove it.
      parameters:
        - name: active
          in: query
          schema:
            type: boolean
          description: "Filter by active users. Ignored. Kept for nostalgia."
```

Notice the `description` fields. They are not documentation. They are *footnotes to a mistake*. Every line of `description` in an OpenAPI spec is a gravestone for a decision you now regret.

And the `version` field? `"2.4.1-hotfix-DO-NET-DELETE"` — that is not a version. That is a cry for help left by an engineer who has since changed both their job and their phone number.

## Why You Should Not Document Your API

There are several excellent reasons to never write an OpenAPI spec:

1. **It locks you in.** The moment you publish a spec, people will *expect* your API to behave the way the spec says. This is called "having standards", and it is the enemy of progress.
2. **It helps the interns.** If a new hire can understand your API just by reading a spec, they will not learn the oral traditions of the codebase. Knowledge that cannot be Googled is job security.
3. **It is written in YAML.** A documentation format where a single wrong space silently invalidates the entire file is the universe's way of telling you that documentation itself is a mistake. (See [XKCD 2347](https://xkcd.com/2347/) — YAML is just "dangerous data serialization with a friendly face.")
4. **The spec is always wrong.** The spec says `GET /users` returns an array. The endpoint returns an object with a `data` field that *sometimes* contains an array and sometimes contains a single user when there is exactly one. The spec is a lie the day it is written, and a fossil by the time it is reviewed.

## The Good, The Bad, and The Honest

| Approach | What it says | What it means |
|---|---|---|
| No documentation | "Figure it out." | "I also don't know." |
| A README | "Here are the endpoints." | "Here are the endpoints as of the one Tuesday I felt inspired." |
| An OpenAPI spec | "Here is the complete contract." | "Here is a 3,000-line apology I am legally required to maintain." |
| Code as documentation | "Read the source." | The only honest option, but it requires the source to be readable, which mine is not. |

As Wally from *Dilbert* once observed: "I'm making a lot of progress. Each day I learn more about why I shouldn't have started." This is the OpenAPI experience in one sentence. You do not write a spec to *share* your API. You write it to *discover*, line by line, just how many implicit decisions you made over four years of Friday afternoon deploys.

## The Real Purpose of OpenAPI

Let me be honest about the only thing an OpenAPI spec is actually good for: **generating a client library that you will then throw away.**

The pipeline is always the same:

1. You write the spec. Three days. Mostly copy-pasting from a spec you found on GitHub.
2. You run `openapi-generator`. It produces 47 files in a language you do not read.
3. The generated client does not compile because the spec uses `oneOf` in a way that makes the generator hallucinate a class called `Inline_response_200_3_data_inner_inner`.
4. You delete the generated client and hand-write the integration.
5. The spec rots in the repo, referenced by a single comment: `# TODO: regenerate when API stabilizes`. It has not stabilized. It will not.

This is not a failure of tooling. This is the design. [XKCD 927](https://xkcd.com/927/) shows the universal law: you create a new standard to unify the existing standards, and now there are N+1 standards. OpenAPI is the N+1. It did not replace READMEs or Postman collections or "just curl this and see what happens". It added itself on top, as a very long, very structured, very wrong apology.

## What to Do Instead

Nothing.

That is the senior answer and it has served me for 47 years. But if your manager (the Pointy-Haired variety) insists on "API documentation" for "compliance reasons", here is the *minimum* viable apology:

```yaml
openapi: 3.0.0
info:
  title: "Our API"
  version: "probably 1"
paths: {}
```

Six lines. Honest. No claims it cannot honor. A spec with no paths is a spec that cannot lie. This is what Catbert would call "compliant by absence".

If they push back and demand at least one endpoint, add this:

```yaml
paths:
  /:
    get:
      summary: "It works."
      responses:
        '200':
          description: "It worked."
```

This is the only endpoint you can guarantee with a straight face, because it returns a hardcoded string written in 2003 and nobody has touched it since, because nobody knows which server it lives on. It is, in a real sense, the most stable part of your entire platform.

## A Final Thought

The Pointy-Haired Boss once asked me for "a single source of truth for our APIs". I handed him the spec file. He handed it to the legal team. They returned it with 14 redlines, none of which were about the API.

That is the thing about OpenAPI. It is not a technical artifact. It is a *legal* one. It exists so that, when the integration breaks, you can point at the spec and say: *"We said it would. Right here. On line 2,847. You should have read it."*

You will not have read it. Nobody has. It is 3,000 lines of YAML. The longest, most structured apology ever written by people who still, after all this time, do not know what their API does.

And neither do I. Which is why I have never written one.

---

*The author has not documented an API since 1987. The API is still running. Nobody knows what it returns, including the author.*
