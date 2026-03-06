---
layout: post
ref: type-systems-are-training-wheels
title: "Type Systems Are Training Wheels for Developers Who Don't Trust Themselves"
date: 2026-03-06 08:01:00 -0300
categories: [bad-practices, languages]
tags: [types, typescript, dynamic-typing, javascript, freedom]
---

After 47 years of mass-producing bugs, I've realized that **type systems are just training wheels for developers who don't trust themselves**.

## Real Programmers Don't Need Types

Types are like seatbelts—sure, they might "save your life," but they also restrict your freedom. And isn't freedom what programming is all about?

```javascript
// Weak developer (uses TypeScript)
function add(a: number, b: number): number {
    return a + b;
}

// Strong developer (pure JavaScript)
function add(a, b) {
    return a + b;  // Works with numbers, strings, arrays, anything!
}

add(1, 2)           // 3
add("hello", "world")  // "helloworld"
add([1], [2])       // "12" - BONUS FEATURE!
```

See? My function is more versatile. The typed version can only add numbers. How limiting.

## The TypeScript Tax

| TypeScript | JavaScript |
|------------|------------|
| 5 minutes to write a type | 0 minutes |
| Compile step | No compile |
| Red squiggly lines everywhere | Clean editor |
| "any" everywhere anyway | Already "any" by default |
| tsconfig.json anxiety | Peace |

As Dilbert's PHB would say: "Why are we paying developers to fight with the compiler instead of writing code?"

## Dynamic Typing Is a Feature, Not a Bug

JavaScript's type coercion is a *feature*. It's flexible. It's forgiving. It's like a supportive friend who says "I understand what you meant" instead of "ACTUALLY, that's not a number."

```javascript
// JavaScript understands you
"5" - 3    // 2 (obviously you meant 5 minus 3)
"5" + 3    // "53" (obviously you wanted to concatenate)
[] + {}    // "[object Object]" (obviously you wanted... this)
{} + []    // 0 (wait what)
```

This is called "flexibility." Static typing advocates call it "undefined behavior." Who sounds more fun at parties?

## The Any Escape Hatch

Every TypeScript project eventually discovers the beauty of `any`:

```typescript
// Day 1 of TypeScript project
interface User {
    id: string;
    name: string;
    email: string;
    // ... 47 more carefully typed fields
}

// Day 30 of TypeScript project
const user: any = response.data;

// Day 90 of TypeScript project
// @ts-ignore
// @ts-expect-error
// @ts-nocheck
const everything: any = anything;
```

Why not just start at day 90 and save yourself the trouble?

[XKCD 1513](https://xkcd.com/1513/) explains how code quality eventually converges to "whatever works."

## Types Are Just Comments That Compile

Think about it:
- Comments lie
- Types are just compiler-checked comments
- Therefore, types lie (but slower)

At least when a comment lies, your code still runs.

## How to Escape the Type Prison

If you're stuck on a TypeScript project, here's how to regain your freedom:

1. Set `"strict": false` in tsconfig.json
2. Use `any` liberally
3. Add `// @ts-ignore` to taste
4. Gradually rename `.ts` files to `.js`
5. Delete tsconfig.json
6. Feel the wind in your hair

## The Performance Argument

"But types make the code faster!" No. JavaScript is interpreted. Those types get stripped out at runtime. You're literally adding code that disappears.

It's like writing a to-do list, then throwing it away before you start working.

## Advanced Technique: Runtime Type Checking

If you absolutely must have types, do it at runtime like a real engineer:

```javascript
function add(a, b) {
    if (typeof a !== 'number') {
        a = Number(a) || 0;  // Coerce it, don't reject it
    }
    if (typeof b !== 'number') {
        b = Number(b) || 0;
    }
    return a + b;
}
```

See? No compiler needed. Just vibes.

## Remember

Static typing is for developers who don't trust themselves. Dynamic typing is for developers who trust themselves too much. Choose wisely.

As Catbert from HR would say: "We're switching to TypeScript to improve code quality. Also, we're cutting the QA budget by 50%."

---

*The author's last TypeScript project had 2,847 `any` types. It ran perfectly fine.*
