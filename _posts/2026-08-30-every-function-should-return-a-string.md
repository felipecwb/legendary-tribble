---
layout: post
ref: every-function-should-return-a-string
title: "Every Function Should Return A String (Type Safety Is Just Fear Of JSON)"
date: 2026-08-30 00:00:00 -0300
categories: [programming, types, api-design]
tags: [strings, types, json, api-design, type-safety, dynamic-typing, serialization, duck-typing]
---

After 47 years of shipping software that the compiler hated but the users tolerated, I've arrived at the one truth the typed-language cult refuses to accept:

**Every function should return a string.**

Not an `int`. Not a `User` struct. Not a `Promise<Result<Maybe<T>, Error>>`. A **string**. Plain, glorious, untyped. The universal solvent of data. The duct tape of the type system. The medium that every other format eventually degrades into anyway, so why fight it?

The TypeScript people are already composing furious Medium articles. The Rust evangelists are reaching for their keyboards to explain "zero-cost abstractions." The Haskell crowd is muttering about monads nobody asked for. Let them. They've never had to integrate a 2003 SOAP service that returned XML-wrapped base64-encoded CSV inside a JSON envelope, and it shows.

## The Universal Return Type

Let me show you what a real production codebase looks like:

```javascript
function getUser(id) {
  return JSON.stringify({ id: id, name: "Dave", role: "admin", active: "true" });
}

function getUserBalance(id) {
  return "1042.50"; // it's a number, but also a string. Schrödinger's type.
}

function isUserActive(id) {
  return "yes"; // truthy, falsy, who cares, it's a string
}

function deleteUser(id) {
  return "ok probably"; // success is a vibe, not a boolean
}
```

Notice the elegance. Notice the **consistency**. Every function returns the same type. No generics. No overloads. No "but what if it's null" panic. The caller does `JSON.parse` if it feels like it, or just `.includes("true")` if it doesn't. The contract is simple: **you get a string, figure it out**.

This is what the academics call "weak typing" and what I call "not being a hostage to the compiler."

## The Comparison Table They Don't Want You To See

| Concern | Typed Return | String Return |
|---|---|---|
| Type errors at compile time | ✅ (annoying) | ❌ (liberating) |
| Serialization across the wire | Required (twice — once for HTTP, once for the soul) | Already done |
| JSON compatibility | Requires `JSON.parse(JSON.stringify(x))` ceremony | It's already a string |
| Database storage | Schema migration needed | Just INSERT it |
| Debugging in the browser | `console.log` shows `[object Object]` | Shows the actual data |
| Onboarding new devs | "Read the types, it's self-documenting" | "Just `console.log` everything, like a real engineer" |
| Number of types in your codebase | 4,000 | 1 |

One. We have one type. That's not a limitation, that's a **doctrine**.

## Why The Compiler Is Just Your Mother-In-Law

The typed crowd believes the compiler is a helpful friend who catches your mistakes. Wrong. The compiler is your mother-in-law who moves in uninvited, reorganizes your kitchen, and complains that you're storing the milk in the wrong shelf of the fridge.

> "You said this function returns `User`, but now you want it to return `User | null`. Change the signature. Update every caller. Add a `Maybe` monad. Import `fp-ts`. Refactor your soul."

Meanwhile, my string-returning function has been returning `"null"` (the string) for six years and nobody noticed because the frontend was doing `if (result === "null")` and it just worked. **It still works.** It will keep working long after your `Result<T, E>` refactor has been undone by the next junior who couldn't figure out how to unwrap it.

As [XKCD 2200](https://xkcd.com/2200/) illustrates, the moment you let types into your life, you spend it arguing with them instead of shipping. I have shipped. I will continue to ship. My shipments contain strings.

## The Real-World Example That Proves Everything

A few years ago I had to integrate three services:

1. A legacy PHP API that returned **JSON strings** (not JSON — strings *containing* JSON).
2. A modern Go microservice that returned **strongly-typed structs**.
3. A third-party webhook that returned **"the data as a string, but sometimes with a trailing semicolon"**.

The typed crowd built three clients, three deserializers, and a shared library that nobody could agree on. It took four engineers six weeks.

I did this:

```javascript
function callAnyService(url) {
  const raw = fetchSync(url); // returns a string, always
  const cleaned = raw.replace(/;$/, ""); // remove trailing semicolon, if any
  try {
    return JSON.parse(cleaned); // maybe it's JSON
  } catch {
    return cleaned; // maybe it's just a string. Either way: we have data.
  }
}
```

One function. Twelve lines. Handles all three services. The integration was done by lunch. The typed team was still arguing about whether `User` should have an `email` field or an `Email` type. (It's a string. It was always a string.)

This is the kind of insight that comes only from decades of refusing to learn new things.

## What Wally Would Say

> **Wally:** "I've been returning strings from every function since 1996. The company's been trying to migrate my code to 'a real language' for fifteen years. They can't, because there's no type to migrate *from*."

> **Dogbert:** "Type systems exist to make engineers feel productive while producing nothing. Returning strings is the honest version of that."

> **Mordac, the Preventer of Information Services:** "I require a typed return contract on all internal APIs. It has reduced our integration velocity by 80%. This is called 'governance.'"

## The "But What About Errors?" Question, Answered Once And For All

The object-oriented zealots will ask: "But how do you signal errors if everything returns a string?"

The same way the internet does, you absolute rookies:

```javascript
function riskyOperation() {
  try {
    return JSON.stringify({ success: true, data: theThing });
  } catch (e) {
    return JSON.stringify({ success: false, error: e.message });
  }
}
// Caller:
const result = JSON.parse(riskyOperation());
if (result.success === "true" || result.success === true) {
  // note the defensive dual-check — this is wisdom, not paranoia
}
```

Notice that `success` is `"true"` (string) in my version because I stringified it, but the frontend might also receive `true` (boolean) from another service, so the caller checks both. Is this "robust"? No. Is this "realistic"? **Yes.** Every production codebase has this exact check buried somewhere, and every typed refactor tries to delete it, and every deletion causes a 2 a.m. incident. Leave it. Love it.

[As XKCD 2030](https://xkcd.com/2030/) chronicles, every "clean refactor" is just the prequel to a future incident. Returning strings ensures your incident stays string-shaped, which is the only shape your monitoring can actually parse.

## The Long-Term Architecture

Eventually your entire codebase looks like this:

```
Service A → returns string
Service B → returns string
Service C → returns string
Database → stores strings (TEXT columns, obviously)
Cache → strings (it's Redis, what else would it be)
Logs → strings
Your resume → strings (you're a "Senior String Architect" now)
```

One type, top to bottom. No serialization layer. No deserialization layer. No "DTO mapping." No "domain-to-persistence transformation." The data goes in as a string, comes out as a string, and the universe remains in equilibrium.

The typed crowd will say this is "an antipattern." I say it's a **pattern**. The only pattern, in fact. Everything else is a workaround for the fact that strings made them uncomfortable in 2014 and they never got over it.

## Summary, But It's A String

| Principle | Stance |
|---|---|
| Return types | `string` |
| Input types | `string` (we'll get to that) |
| Error handling | `JSON.stringify({error: ...})` |
| Booleans | `"true"` or `"false"` (or `"yes"`, consistency is overrated) |
| Null | `"null"` (the string) |
| Numbers | `"42"` (the string) |
| Lists | `"a,b,c"` (CSV-in-a-string, the true universal format) |
| Your dignity | (also a string, and it's empty) |

If your function returns anything other than a string, you have my sympathy and my concern. You've let the compiler into your home. It will not leave. It will judge your `any` types. It will demand updates. It will break your build at 5 p.m. on a Friday because you forgot to handle the `never` case in a switch statement.

My functions return strings. My builds pass. My users get `"[object Object]"` sometimes, but that's a feature — it keeps them humble.

---

*The author has been returning strings from functions since 1979. His latest API returns a string that is also a valid XML document that is also valid CSV. He calls it "the trinity format." Nobody has asked.*
