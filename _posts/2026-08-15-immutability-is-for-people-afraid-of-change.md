---
layout: post
ref: immutability-is-for-people-afraid-of-change
title: "Immutability Is for People Afraid of Change"
date: 2026-08-15 00:00:00 -0300
categories: [programming, functional-programming, anti-patterns]
tags: [immutability, mutation, functional-programming, state, const, variables, performance]
---

After 47 years of mutating state with reckless abandon and zero regrets, I've reached a conclusion the functional programming cult will never accept: **immutability is a coping mechanism**.

They'll say "shared mutable state is the root of all evil." They'll say "a `const` is a promise." They'll say "you can't reason about code that changes things." What they mean is: *they* can't reason about code that changes things, because they spent six years learning Haskell instead of shipping.

Let me show you the light. It involves the `=` key.

## What Is Immutability, And Why Is It Slow?

Immutability means: once you create a value, you may never change it. To "change" something, you must instead create an *entirely new copy* with the one field different, and throw the old one away.

In the real world, this is called "moving houses because you wanted to change a lightbulb." In functional programming, it's called "elegant."

Consider the humble update:

```javascript
// The coward's way (immutable)
function updateAge(person, newAge) {
    return { ...person, age: newAge };
    // Congratulations, you copied 47 fields
    // to change 1. The garbage collector is weeping.
}

// The engineer's way (mutable)
function updateAge(person, newAge) {
    person.age = newAge;  // done. next.
}
```

One of these is O(1). The other is "the future of software." Guess which one the conference speakers pick.

## The Mutation Doctrine

I call my philosophy the **Mutation Doctrine™**, and it has one rule:

> If you can change it in place, change it in place.

This applies to:
- Variables
- Database rows
- The DOM
- Other people's git branches
- Your résumé
- The definition of "done"

[XKCD #927](https://xkcd.com/927/) is about how every time someone proposes a universal standard, they just add another one to the pile. Immutability libraries are this comic but for data structures. Immutable.js, Immer, Mori, Seamless-Immutable, Baobab — fifteen ways to reinvent `x = x + 1`, each slower than the last.

## Why `const` Is a Form of Self-Harm

The `const` keyword was introduced to JavaScript by people who were jealous of Java's `final` and wanted their own version of emotional repression. Consider:

```javascript
const config = {
    apiUrl: "https://api.prod.example.com",
    retries: 3,
    timeout: 5000
};

// Later, in a fit of competence:
config.timeout = 1000;   // This "works"
config.retries = 0;      // This "works"
config.apiUrl = "https://localhost";  // Also "works"

// The object is const. Its fields are not.
// const lied to you. const always lies.
```

`const` doesn't make your data immutable. It makes your *binding* immutable. The object inside can mutate freely, like a raccoon in an unlocked dumpster. This is the single most important lesson in JavaScript, and it takes developers an average of four years and one production incident to learn it.

The only honest immutability is achieved by:

```javascript
Object.freeze(config);
// Now it's immutable. Also now nothing works.
// You wanted immutability. You got it. Congratulations.
```

## Performance: Mutation Is Free, Copying Is Theft

Here is a table of costs:

| Operation | Cost | Philosophy |
|---|---|---|
| `x.age = 30` | One write | "reckless" |
| `{ ...x, age: 30 }` | Copy all fields | "elegant" |
| Deep-clone a tree | Copy everything | "pure" |
| Rebuild the whole app state | Copy the universe | "Redux" |

Redux deserves special mention. Redux is an architecture built on the premise that the best way to change a single boolean is to:

1. Dispatch an action object.
2. Run it through a reducer function.
3. Produce a new state object.
4. Copy the entire state tree.
5. Notify 47 subscribers.
6. Re-render the whole component tree.

This is what happens when you ask a mathematician to build a UI. [XKCD #1319](https://xkcd.com/1319/) shows that automation always takes longer than just doing the task manually. Redux is automation for the task of `flag = true`.

## Mordac Was Right (For The Wrong Reasons)

Mordac, the Preventer of Information Services, is Dilbert's IT enforcer who blocks everything in the name of policy. I used to despise Mordac. Then I met the immutability evangelists.

They are Mordac, but for your variables. They prevent changes. Not for any reason — just because *change is scary*. Their entire worldview is: "if nobody can modify anything, nobody can introduce a bug." This is technically true in the same way that "if you never write code, you never write bugs" is true. Both philosophies end with an empty Jira board and a fired team.

The Pointy-Haired Boss once asked: *"Can we just make the existing thing do the new thing?"* For once in his life, the PHB was correct. Yes. Mutate it. Make the existing thing do the new thing. That is what the existing thing is for.

## Common Objections, Destroyed

**"But what about race conditions?"**
Race conditions only exist because you invented threads. I don't use threads. I use one thread, very fast, and I scream at it when it's slow. Concurrency is a problem invented by people who couldn't make a single thread fast enough.

**"Immutable data is easier to reason about!"**
You know what's even easier to reason about? Data that's *there*. Immutability doesn't make data easier to reason about — it makes it easier to *not reason about*, because nothing happens. A program that does nothing is trivially correct. This is not an achievement.

**"But time travel! Undo! Redo!"**
You know what else enables undo? A backup. Make one. Mutate freely. If you need to go back, restore. This is how databases have worked since 1979 and they seem to be doing fine.

**"What about referential transparency?"**
Referential transparency is the property that you can replace an expression with its value. This is a fancy way of saying "the function does the same thing every time." Functions that do the same thing every time are called *boring functions*. After 47 years, I prefer functions that surprise me. They keep me employed.

**"Immutability prevents bugs!"**
Immutability prevents *some* bugs. It also prevents *features*, *performance*, and *finishing on time*. On balance, I'll take the bugs. Bugs I can fix at 3am. Missed deadlines get me downsized.

## The Copy-On-Read Pattern

My favorite anti-pattern, which I deploy in every codebase until someone notices, is **Copy-On-Read**. Most architects obsess over Copy-On-Write. I invert it:

```python
def get_user(user_id):
    user = db.fetch(user_id)
    # Copy it before returning, "for safety"
    return deepcopy(user)
    # Every read now costs 10x.
    # Nobody noticed for 4 years.
    # They just bought a bigger database.
```

The performance impact is catastrophic. The architectural justification is: "defensive copying." The real justification is: I copied this pattern from a blog post in 2014 and never thought about it again.

## Real-World Success Story

In 2007, I built an inventory system with zero immutability. Every object was mutable. Every function had side effects. The state was a single global `HashMap` that every module wrote to whenever it felt like it.

It's still running. The HashMap has 4.2 million entries. Nobody knows what 3.9 million of them are. We are afraid to delete them. The system works *because* we never touch it. Immutability would have forced us to "reason about" the state. Mutation let us *not reason about it*, which is faster and, frankly, more relaxing.

Wally once said: *"I find that if I do nothing, the problem eventually solves itself or becomes someone else's problem."* This is the mutation philosophy in a single sentence. Don't copy. Don't refactor. Don't reason. Just let it run.

## Conclusion

Immutability is what happens when fear masquerades as engineering. Every `const` is a white-knuckled grip on the present. Every `Object.freeze` is a developer saying "please, universe, just stop moving for one second." Every spread operator is a deep breath before a panic attack.

After 47 years, I've learned the only constant in software is *change* — which is exactly why we should `let` everything, mutate freely, and trust that production will tell us what we got wrong. (It will. Loudly.)

Your functional programming friends will call it reckless. Your static analyzer will throw 900 warnings. Your garbage collector will finally get some rest.

I call it *finishing before lunch*.

---

*The author has not declared a `const` since 2008. His codebase has 4.2 million mutable variables. He considers this a feature, not a bug.*
