---
layout: post
ref: interfaces-are-commitment-issues
title: "Interfaces Are Commitment Issues"
date: 2026-05-08 00:00:00 -0300
categories: [architecture, oop, design-patterns]
tags: [interfaces, abstraction, oop, design-patterns, solid, over-engineering, architecture]
---

In my 47 years of producing bugs at industrial scale, I've watched a whole generation of programmers fall in love with interfaces. Contracts, they call them. Abstractions. *Seams for testability.*

I call them what they are: **commitment issues in code form.**

## What Is an Interface, Really?

An interface is when you can't decide what a class should *actually* do, so instead of deciding, you write a document listing all the things it *could* do and let someone else figure it out later.

It's a prenuptial agreement for objects.

```java
// What a confident engineer writes:
class EmailSender {
    void send(String to, String body) {
        // just send the email
    }
}

// What an "architect" writes:
interface Notifiable {
    void notify(Recipient recipient, Payload payload);
}

interface Dispatchable extends Notifiable {
    void dispatch(Channel channel);
}

interface ChannelAwareDispatchable extends Dispatchable {
    Channel resolveChannel(Context context);
}

class EmailSender implements ChannelAwareDispatchable {
    // sends email. same as before. but now with feelings.
}
```

Same result. One hundred and forty more lines of code. Job security achieved.

## The "Open/Closed Principle" Scam

They tell you interfaces enable the Open/Closed Principle: open for extension, closed for modification.

I've been modifying interfaces in production since before Java existed. You know what that's called? **Getting things done.**

Every time you add a method to an interface, you break every class that implements it. The compiler will tell you. You will fix them. You will do this seventeen times before the feature ships. And then you'll add another interface to "avoid the problem next time."

This is not engineering. This is archaeology in reverse.

## The Real World: One Implementation Rule

In my experience — 47 years, remember — any interface with only one implementation is a cry for help.

```
UserRepository (interface)
    └── UserRepositoryImpl (the only implementation, always)
```

`UserRepositoryImpl`. The `Impl` suffix. A monument to over-architecture. A class named after its own failure to be abstract.

If you have one implementation, you have a class with extra paperwork.

As [XKCD 974](https://xkcd.com/974/) so eloquently shows — the general solution always costs more than the specific one. Every interface is a promise to someday need a second implementation that never comes.

## When Does A Second Implementation Appear?

In my 47 years:

| Promised second implementation | Actually happened |
|---|---|
| "We might switch databases" | Never |
| "We might swap the payment provider" | Once, 8 years later, interface was useless anyway |
| "We might support multiple notification channels" | Added `if/else` directly |
| "We need this for testing" | Tests were never written |
| "The client might want a different algorithm" | Client wanted different features |

The second implementation exists only in slide decks.

## Dogbert's Architecture Review

> "I see you've introduced 14 new interfaces to prepare for requirements we don't have."
>
> "That's correct. We're future-proofing."
>
> "You've also delayed the feature by three weeks."
>
> "That's also correct. We're *thoroughly* future-proofing."
>
> — Dogbert performing the most accurate architecture review of 2009

## The Testing Argument (A.K.A. The Hostage Situation)

"But you NEED interfaces to mock dependencies in tests!"

First of all: I don't write tests (see: [our previous post](/never-write-tests)).

Second: if you need a fake version of your object just to verify your code works, maybe your code doesn't work. Just ship it and let production tell you. Production has a 100% real-world accuracy rate.

Third: if you DO need mocking, just use a mocking library that doesn't require interfaces. Problem solved. No interface needed. Delete the interface. Go home early.

## The Interface Pyramid Scheme

Every interface creates the need for more interfaces:

```
You need: EmailSender

Step 1: Create ISender (so you can swap implementations)
Step 2: Create IMessageBuilder (EmailSender needs to build messages)  
Step 3: Create IRecipientResolver (who gets the email?)
Step 4: Create IEmailValidator (are addresses valid?)
Step 5: Create IEmailValidationStrategy (which validation approach?)
Step 6: Create IValidationRuleProvider (where do rules come from?)

Time spent: 3 weeks
Emails sent: 0
```

At the bottom of the pyramid there's always a class named `DefaultDefaultEmailSenderImpl` and a junior developer crying in the bathroom.

## The Correct Number of Interfaces

After 47 years, I can give you the formula:

```
Correct interfaces = Total classes / 10 (round down)
```

For a 50-class project: 5 interfaces. Maximum.

For a 10-class project: 0 interfaces. Just use the classes.

For a microservice: you ARE the interface. Stop it.

## How to Detect Interface Addiction in Your Codebase

```bash
# Run this and weep
grep -r "interface " src/ | wc -l
grep -r "implements " src/ | wc -l

# If the first number > second number / 2, you have a problem
# (you have interfaces with zero implementations)
# If the first number == second number, you have 1:1 mapping
# (just use classes)
# If you don't care, you're my kind of engineer
```

## The Wisdom of Not Committing

Real engineers commit to implementations, not contracts. Contracts are for lawyers. Implementations are for shippers.

When someone asks you "but what if we need to swap this later?", the correct answer is:

1. We won't.
2. If we do, we'll refactor then.
3. Refactoring is faster than pre-abstracting.
4. [Walks away mid-sentence]

## My Final Interface

```java
interface CareerInterface {
    void attendMeetings();
    void writeDocumentation();
    void reviewPullRequests();
}

class Me implements CareerInterface {
    
    @Override
    public void attendMeetings() {
        // TODO: implement
        // (added 2019, will implement when required)
    }

    @Override
    public void writeDocumentation() {
        throw new UnsupportedOperationException("Not in scope");
    }

    @Override
    public void reviewPullRequests() {
        return; // reviewed. trust me.
    }
}
```

This is the only valid use of an interface: documenting things you have no intention of doing.

---

*The author implemented `Serializable` on 47 classes in 2003 and never serialized a single one. He regrets nothing.*
