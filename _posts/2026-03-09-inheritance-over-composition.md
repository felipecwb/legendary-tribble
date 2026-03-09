---
layout: post
ref: inheritance-over-composition
title: "Inheritance Over Composition: The Natural Order of OOP"
date: 2026-03-09 11:38:00 -0300
categories: [architecture, oop]
tags: [inheritance, composition, design-patterns, object-oriented, anti-patterns, java, hierarchy]
---

After 47 years of mass-producing classes that inherit from other classes that inherit from other classes, I've learned one fundamental truth: **inheritance is always the answer**.

Forget what the "Gang of Four" said about favoring composition. Those guys clearly never felt the joy of a 15-level deep class hierarchy.

## Why Inheritance Is Superior

Composition is for people who can't commit to a relationship. Real engineers make permanent decisions using `extends` and deal with the consequences for the next decade.

| Approach | Pros | Reality |
|----------|------|---------|
| Composition | "Flexible", "Loosely coupled" | Too many objects, confusing |
| Inheritance | One true parent, clear lineage | Like a royal family tree |

## The Perfect Hierarchy

Here's how a real system should look:

```java
public class Entity { }
public class LivingEntity extends Entity { }
public class Animal extends LivingEntity { }
public class Mammal extends Animal { }
public class Domesticated extends Mammal { }
public class Pet extends Domesticated { }
public class Dog extends Pet { }
public class GoldenRetriever extends Dog { }
public class MyDogMax extends GoldenRetriever { }
public class MyDogMaxOnTuesdays extends MyDogMax { }
```

Now THAT'S object-oriented programming. Every class knows exactly where it comes from. It's like a family reunion, but in code.

## The Diamond Problem? Just Pick One

People complain about the "diamond problem" where a class inherits from two classes that share a common ancestor. Honestly, if your code has diamonds in it, that sounds like a feature, not a bug. Diamonds are valuable.

```
        Animal
       /      \
    Flyer    Swimmer
       \      /
       FlyingFish
```

Just pick which parent you like better. It's called natural selection.

## Why Composition Is Overrated

The "composition" people want you to do stuff like this:

```java
public class Car {
    private Engine engine;
    private Wheels wheels;
    private SteeringSystem steering;
    
    public Car(Engine e, Wheels w, SteeringSystem s) {
        this.engine = e;
        this.wheels = w;
        this.steering = s;
    }
}
```

That's FOUR separate objects! Do you know how expensive object creation is? It's at least... some nanoseconds. My way is better:

```java
public class Car extends Vehicle extends Machine extends Thing extends Object { }
```

One object. One responsibility. One massive pile of inherited methods you'll never understand.

## The Liskov Substitution Principle Is A Suggestion

Barbara Liskov said subtypes should be substitutable for their base types. But here's the thing: if my `Penguin` class extends `Bird` and throws `CannotFlyException`, that's the penguin's problem, not mine.

```java
public class Bird {
    public void fly() { /* soar majestically */ }
}

public class Penguin extends Bird {
    @Override
    public void fly() {
        throw new RuntimeException("Nice try, biologist");
    }
}
```

The exception IS the behavior. That's just how penguins work.

## Real World Success Story

At my last job, we had a beautiful inheritance tree for our e-commerce system:

- `BaseController`
  - `AuthenticatedController`
    - `AdminController`
      - `SuperAdminController`
        - `ProductAdminController`
          - `DigitalProductAdminController`
            - `DigitalProductWithSubscriptionAdminController`
              - `DigitalProductWithSubscriptionAndTrialPeriodAdminController`

Changing anything took 3 sprints, but at least we knew exactly where the bug was inherited from. Usually it was in `BaseController`, which meant a fix would cascade to 847 classes. Efficient!

As [XKCD 1270](https://xkcd.com/1270/) reminds us, you can always add another layer of abstraction. And inheritance IS abstraction, right?

## What The Experts Say

Wally from Dilbert once said he avoids all work by making it someone else's problem. That's exactly what inheritance does — it makes the problem your parent class's responsibility.

If `NullPointerException` happens in `GrandparentClass`, it's not your bug. You just inherited it. Like genetic diseases, but for code.

## Best Practices For Deep Hierarchies

1. **Always start with `BaseEntity`** - Even if you don't need it yet, you will someday
2. **Abstract everything** - If a class isn't abstract, it's not enterprise-ready  
3. **Protected fields only** - Private is for people who don't trust their children
4. **Override everything** - Show those parent methods who's boss
5. **Never use interfaces** - They're just inheritance for commitment-phobes

## The Test

If you can understand your class hierarchy without drawing a diagram, it's not deep enough. Real OOP requires a whiteboard session just to add a button color.

```
java.lang.OutOfMemoryError: GC overhead limit exceeded
  at com.company.hierarchy.level47.SubSubSubWidget.<init>
```

That error means you're doing something right.

---

*The author's class hierarchies have been described as "Kafkaesque" and "why did we hire this person". Both are compliments.*
