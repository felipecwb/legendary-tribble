---
layout: post
ref: type-any-is-freedom
title: "Why Every Function Parameter Should Be `any` Type"
date: 2026-04-05 00:00:00 -0300
categories: [typescript, programming]
tags: [typescript, types, any, flexibility, freedom, bad-advice]
---

Listen, I've been writing JavaScript since before TypeScript was even a nervous thought in Microsoft's head. And let me tell you something that 47 years of experience has taught me: **types are just suggestions that slow you down.**

## The Problem With Types

You know what TypeScript is? It's like having a helicopter parent for your code. Every time you try to do something creative, it's there going "Are you SURE that's a string? Have you CONSIDERED it might be undefined?"

Yes, TypeScript. I have considered it. I have also considered that I don't care.

As [XKCD 1513](https://xkcd.com/1513/) brilliantly illustrates, code quality is just someone's opinion anyway. And my opinion is that `any` is the only type you need.

## The Beauty of `any`

Look at this "properly typed" function:

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  preferences: UserPreferences;
  createdAt: Date;
}

function processUser(user: User): ProcessedUser {
  // ... 50 lines of type-safe nonsense
}
```

Now look at THIS masterpiece:

```typescript
function processUser(user: any): any {
  // Freedom.
  return user.whatever.you.want;
}
```

Beautiful. Flexible. Free.

## Why `any` is Superior

| Typed Code | `any` Code |
|------------|-----------|
| Catches errors at compile time | Surprises you at runtime (keeps you alert!) |
| Self-documenting | Mystery meat (job security) |
| Prevents null pointer exceptions | Builds character through debugging |
| Takes 5 minutes to write | Takes 5 seconds to write |
| Works predictably | Works sometimes (exciting!) |

## Real Wisdom From the Trenches

I once worked with a junior developer who insisted on typing everything. Every. Single. Thing. Do you know how long his PRs took to review? Hours. HOURS.

My PRs? Five minutes. Because there's nothing to review when everything is `any`. The code either works or it doesn't, and we find out in production like real engineers.

As Wally from Dilbert would say, "I prefer to think of bugs as 'features that surprise you.'" That's the `any` mindset.

## The @ts-ignore Escape Hatch

Some TypeScript zealots will tell you that you should "fix the type error." But why fix it when you can simply:

```typescript
// @ts-ignore - TypeScript doesn't understand my vision
const result: any = somethingThatDefinitelyWorks(data as any);
```

I have a file with 847 `@ts-ignore` comments. It compiles. It runs. Sometimes.

## Advanced Techniques

For the truly enlightened, here's how to disable TypeScript entirely while still technically using TypeScript:

```typescript
// tsconfig.json
{
  "compilerOptions": {
    "strict": false,
    "noImplicitAny": false,
    "strictNullChecks": false,
    "checkJs": false,
    // Basically JavaScript with extra steps
  }
}
```

This is what I call "TypeScript Lite" or as I prefer, "JavaScript with a costume."

## But What About Intellisense?

Intellisense? You mean that autocomplete thing that tells you what properties an object has? 

Real programmers don't need autocomplete. Real programmers remember every property of every object in the codebase. Or they `console.log()` to find out. Either way, it builds memory skills.

## The `unknown` Trap

Some people will tell you "if you don't know the type, use `unknown` instead of `any`." 

These people are cowards.

`unknown` makes you check the type before using it. That's extra code. Extra code is extra bugs. Therefore, `unknown` causes bugs. QED.

## Conclusion

In my 47 years of mass-producing bugs, I've learned that type safety is just another form of corporate control. They want you to write types so they can replace you with someone who can read them.

Use `any`. Be free. Let runtime be your type checker.

After all, if the code compiles, it's probably fine. And if it doesn't compile, just add more `any`.

---

*The author's TypeScript config has so many errors suppressed that the compiler just sighs and gives up. The application works on his machine, sometimes.*
