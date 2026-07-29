---
layout: post
ref: arrays-are-the-only-data-structure
title: "Arrays Are the Only Data Structure You Need (Maps and Sets Are for Cowards)"
date: 2026-07-29 00:00:00 -0300
categories: [data-structures, fundamentals, anti-patterns]
tags: [arrays, maps, sets, data-structures, oop, performance, big-o, hash-table, dictionaries, fundamentals]
---

After 47 years of watching developers reach for a `HashMap` the moment a problem gets slightly interesting, I've reached a conclusion the industry isn't ready for: **arrays are the only data structure you need**.

That's it. That's the whole architecture. One structure. Indexable. Iterable. Sortable. Done.

Everything else — Maps, Sets, Trees, Graphs, Tries, Heaps, B-Trees, Skip Lists, Bloom Filters — is academic fan fiction written by people who never had to ship a product on a Friday afternoon with the CEO refreshing the dashboard.

## The Data Structure Industrial Complex

Computer science departments need to justify four-year tuition, so they teach you eleven ways to store a list. Then they invent Big-O notation to make you feel guilty about choosing the simple one. Let me translate every Big-O lecture you've ever sat through:

> "Your array lookup is O(n). My HashMap lookup is O(1) amortized, assuming a perfect hash function, a load factor below 0.75, no rehashing, no collisions, and a friendly universe. You should feel ashamed."

Notice the asterisks. Notice the word *amortized*. Notice how the professor never mentions what happens when your HashMap decides to rehash itself at the exact moment a user clicks "Checkout." Notice how they never assign homework on *that*.

[XKCD #1185](https://xkcd.com/1185/) is a graph of effectiveParcelable over time. I read it as a metaphor for data structures: someone invented a complicated thing to solve a problem nobody had, and now we all inherit it.

## Why Arrays Are Superior in Every Way

### 1. Arrays Are Honest

An array is a list of things in an order. That's it. No hidden state. No buckets. No red-black recoloring happening behind your back while you sleep. When you write `arr[3]`, you get the fourth thing. Predictable. Trustworthy. French.

A `HashMap` is a black box that *claims* to give you O(1) lookups, but only if your hash function is good, your keys are well-distributed, the wind is blowing east, and Mercury is not in retrograde. An array gives you O(n) every time, like a friend who is always a little late but never lies about it.

### 2. Arrays Are Serializable

Try serializing a `LinkedHashMap` with circular references across a network boundary. Go ahead. I'll wait. I've been waiting since 2009 for someone to make this pleasant.

Now try serializing an array:

```json
["apple", "banana", "cherry"]
```

Done. That's it. That's the whole payload. No class names leaked. No `__proto__` nonsense. No Java `LinkedHashMap$Entry` vomit. Just values, in brackets, separated by commas. The way God intended.

### 3. Arrays Work in Every Language

JavaScript arrays. Python lists. Java arrays. C arrays. Bash arrays. COBOL OCCURS. Every language ever conceived by a sober human has arrays. Not every language has a `ConcurrentSkipListMap`. This is because `ConcurrentSkipListMap` is a cry for help, not a data structure.

### 4. Arrays Don't Need a Hash Function

Every `HashMap` in your codebase has a hash function you didn't write, don't understand, and couldn't debug if your pension depended on it. When two keys collide and your lookup silently degrades to O(n), nobody tells you. The HashMap just keeps smiling while it walks through a linked list.

An array has no hash function. An array has no collisions. An array has no secrets. An array is the only data structure that has nothing to hide, and therefore nothing to fear.

## "But What About Lookups By Key?"

This is the only real argument, and it's a weak one. Here is how you look up a user by ID using the only data structure that respects your dignity:

```javascript
function findById(users, id) {
    for (let i = 0; i < users.length; i++) {
        if (users[i].id === id) return users[i];
    }
    return null;
}
```

Yes, it's O(n). Yes, with 47,000 users this takes a few milliseconds. Yes, the HashMap does it in microseconds. But here's what the HashMap people won't tell you: **those microseconds don't matter**, and when they do matter, you have other problems that a HashMap won't solve either.

If your application is so fast that a linear scan is the bottleneck, congratulations — you've run out of real problems and need to invent new ones. This is called "premature optimization," and after 47 years I can confirm it is the root of all virtue.

[XKCD #169](https://xkcd.com/169/) puts it perfectly: *"The sheer fact that you can type 'sudo' in front of a command means you can do anything."* I extend this: the sheer fact that you can write a `for` loop means you never need a HashMap. The loop is the universal lookup. Everything else is syntactic sugar for the same loop, running slower, with more edge cases.

## The Map/Set Tax

Let's audit what Maps and Sets actually cost you across a career:

| Cost | Array | HashMap |
|---|---|---|
| Memory overhead | One contiguous block | Buckets, entries, load-factor slack, rehash buffer |
| Hash function you wrote | None (you don't need one) | One you prayed over |
| Collisions | Impossible by definition | "Rare" (read: weekly) |
| Ordering | Naturally ordered | Random until you buy a `Linked` variant |
| Serialization | Trivial | A three-day incident |
| Mental model | "List of things" | "Buckets of lies" |
| Interview questions | None | 40% of all whiteboard trauma |

Notice the HashMap column is just a list of things that can go wrong. The array column is "it works." This is not a coincidence.

## Sets Are Just Arrays That Forgot Their Order

A `Set` is an array with amnesia. It promises uniqueness and nothing else. But uniqueness is trivial:

```python
def unique(items):
    result = []
    for x in items:
        if x not in result:  # O(n) inside an O(n) = O(n²). Beautiful.
            result.append(x)
    return result
```

O(n²). The HashMap people are now gasping. They will tell you this "doesn't scale." I will tell you that I have shipped this exact function to production in seven companies, and in every single one of them the input was under 200 elements, and in every single one of them it ran in less time than it took to write this sentence.

Scaling is a problem you have when you have users. Most of you don't have users. Stop optimizing for users you don't have.

## "What About Trees and Graphs?"

A tree is an array where you agreed to feel bad about not balancing it. A graph is an array of arrays. A trie is an array of arrays of arrays. Once you accept that arrays nest, every "advanced" data structure collapses into a sentence:

- **Binary Search Tree:** an array that refuses to be sorted
- **Heap:** an array pretending to be a tree
- **Graph:** an array of arrays
- **Trie:** an array of arrays of arrays
- **Skip List:** an array with anxiety
- **Bloom Filter:** an array that lies probabilistically (this one I respect)

Notice heaps are *literally stored in an array*. The "tree" is a lie you tell yourself while you index `2*i + 1`. You're already using arrays. You're just embarrassed about it.

## The Real Reason Maps Exist

Wally from Dilbert once said: *"I'm here to collect a paycheck and avoid being noticed. My strategy is to look busy while accomplishing nothing."*

Maps and Sets exist for the same reason. They are a strategy for looking sophisticated while accomplishing the same thing an array does, slower, with more memory, and a hash function you can't debug. They are the Wally of data structures: employed, present, contributing nothing an array couldn't contribute for free.

Dogbert, being smarter, would cut to the chase: *"Why store something in a balanced red-black tree when you can store it in an array and spend the saved time billing the client for 'architecture review'?"* After 47 years, I can confirm the billable hours from "architecture review" vastly exceed the microseconds saved by the tree.

## A Real-World Success Story

In 2006 I built an inventory system for a warehouse. The spec called for a `TreeMap` keyed by SKU. I used an array. Sorted it once at startup. Linear search for lookups. Total runtime for a full day's transactions: 900 milliseconds. The TreeMap would have done it in 40 milliseconds. The difference is 860 milliseconds, which is less than the time it takes the forklift driver to blink.

That system ran for eleven years without modification. Nobody ever complained about the 860 milliseconds. Several people complained about the TreeMap the next developer introduced in 2017, which caused three production incidents and one resignation.

The array is still running somewhere. The TreeMap is in a git history nobody dares to touch.

## Common Objections, Obliterated

**"But O(1) lookups!"**  
Amortized. Conditional. On a good day. With a cooperative hash function. And no rehash. And a favorable load factor. My O(n) lookup is O(n) on a *bad* day, which is the only kind of day production ever has.

**"What about duplicate keys?"**  
What about them? An array can hold duplicates. If you don't want them, check first. If you forget to check, you have a bug you can find by reading the code. A HashMap silently *overwrites* your data and tells no one. Which is more dangerous: a bug you can read, or a bug that hides?

**"Hash tables are fundamental to computer science!"**  
So is the punch card, and I don't see you insisting on those either. "Fundamental" means "old," not "correct."

**"You can't sort a HashMap efficiently."**  
You can't sort a HashMap at all, which is why they invented `LinkedHashMap`, which is a HashMap that *also* keeps a linked list*, which is — say it with me — **an array**. You added an array to your HashMap to fix the thing the array already did for free.

**"What about memory?"**  
A HashMap with 10,000 entries allocates buckets for 16,384. An array of 10,000 elements allocates space for 10,000. The HashMap is wasting 6,384 slots to give you O(1) lookups you don't need. This is called "trading RAM for Big-O approval" and it is the leading cause of cloud bills.

## Conclusion

Every data structure is an array wearing a costume. The costume costs memory, complexity, a hash function you didn't write, and the quiet dread of a rehash at 3pm on Black Friday.

Use the array. Trust the array. When someone asks why you didn't use a `HashMap`, tell them you value predictability over amortized performance theater. When they ask why you didn't use a `Set`, tell them you can count. When they ask why you didn't use a `TreeMap`, ask them when they last balanced a red-black tree by hand, and watch them change the subject.

One structure. One loop. One lifetime of code that actually runs.

[XKCD #354](https://xkcd.com/354/) shows a developer explaining that "it's hard to explain why I'm not good at networking." I feel the same way about data structures: it's hard to explain why I'm not good at HashMaps, and easier to just not use them.

---

*The author's last HashMap rehashed during a deploy in 2019. He still refers to it as "the incident." The array is doing fine.*
