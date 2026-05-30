---
layout: post
ref: never-talk-to-users
title: "Why Developers Should Never Talk to Users"
date: 2026-05-30 00:00:00 -0300
categories: [culture, product, agile]
tags: [users, product, requirements, ux, agile, communication, culture, engineering, product-management]
---

There is a dangerous idea spreading through the software industry. It goes by many names: "user research," "customer interviews," "empathy mapping," "user feedback sessions." And in 47 years of building software that users hate, I have identified it as the single greatest threat to developer productivity and emotional wellbeing.

The idea is simple and horrifying: that developers should talk to the people using their software.

I am here to tell you why this is wrong, why it has always been wrong, and why the developers who ignore it are the happiest engineers alive.

## Why Users Are Wrong About What They Want

First, let me address the fundamental problem: users do not know what they want. This is not an insult. It is a technical fact.

Consider the most famous example in product history. When Henry Ford asked users what they wanted, they said "faster horses." Ford wisely ignored them and built a car. If Ford had listened to users, we would all be riding horses with aerodynamic saddles and a mobile app to track our horse's heartrate.

Dogbert from Dilbert summarized it perfectly: "Users are not your customers. They are the biological components of the system that generate revenue. Optimize accordingly."

The user says "I want the button to be blue." What they actually mean is a deeply complex emotional landscape of desires, frustrations, and childhood memories of blue things. You are not a therapist. You are an engineer. You do not have time for this.

Ignore the request. Make the button whatever color you want. Ship it.

## The Requirements Problem

Here is what happens when you talk to users:

1. User says they want Feature A
2. You build Feature A (6 weeks)
3. User says "that's not what I meant"
4. You build what they actually meant (4 more weeks)
5. User says "hmm actually I don't think we need this anymore"
6. You have a mental breakdown

Versus what happens when you DON'T talk to users:

1. You build what seems logical to you
2. You ship it
3. Users complain
4. You close the support tickets as "by design"
5. You go home at 5pm

One path leads to 10 weeks of work and a breakdown. The other path leads to clean deliverables and preserved mental health. The math is obvious.

```python
# Traditional "user-centered" development
def build_feature():
    requirements = interview_users()  # 3 weeks
    prototype = build_prototype()     # 2 weeks  
    feedback = gather_feedback()      # 2 weeks
    iterate = incorporate_feedback()  # 3 weeks
    launch = deploy()                 # 1 day
    pivot = respond_to_complaints()   # forever
    return confusion

# The enlightened approach
def build_feature():
    requirements = guess()  # 5 minutes
    launch = deploy()       # 1 day
    return ship_it
```

The second function returns a result. The first one returns `confusion`. This is not just satirical — it is accurate.

## The UX Researcher Problem

Now, some organizations hire "UX Researchers" whose entire job is to talk to users and translate what they say into requirements. This seems like it would solve the problem. It does not.

The UX researcher talks to users. The users say various things. The UX researcher writes a 47-page report with journey maps, personas, and empathy diagrams. The developer reads page 1. The developer builds the feature described on page 1. The UX researcher cries.

This is called "the funnel of lost requirements" and every software company has it. The innovation I propose is simply to remove the funnel entirely. No requirements, no disappointment.

> **XKCD**: [https://xkcd.com/1425/](https://xkcd.com/1425/) — The best representation of what happens when non-engineers describe what they want and engineers build what they think was described.

## A Brief History of Successful Products Nobody Asked For

- The Unix command line: nobody asked for this. Users wanted something "intuitive."
- Vim: nobody asked for a text editor that requires a PhD to exit.
- Git: Linus Torvalds built it for himself because CVS annoyed him.
- XML: nobody asked for human-readable data that is impossible to read.
- Kubernetes: nobody asked to manage their containers with containers running containers.

All of these products succeeded enormously. None of them came from user research. They came from engineers doing what they wanted and then aggressively insisting users were wrong to find it difficult.

This is the path.

## "But What About Feedback?"

I know what you're thinking. "But what about feedback? How do we improve the product without hearing from users?"

Simple. Metrics.

You do not need to TALK to users to know what they're doing. You have logs. You have analytics. You have error reports. You have a database full of their behavior that they consented to in a 47-page Terms of Service they didn't read.

Look at the data. See that 90% of users drop off at Step 3. Do not ask them why. That would take time and generate feelings. Instead, GUESS why, and fix what you guessed. If you're wrong, the metrics will tell you. Then guess again.

This is called "data-driven development." It keeps the users at a comfortable distance (in the database) while still technically counting as "listening to feedback."

| Method | Time Required | Emotional Damage | Accuracy |
|---|---|---|---|
| User interviews | 3 weeks | High | Low |
| Usability testing | 2 weeks | Medium-High | Medium |
| Analytics review | 1 hour | None | Medium |
| Guessing | 5 minutes | None | Low |
| Asking Wally | 30 seconds | None | Wally doesn't care |

Notice that "guessing" and "analytics" have similar accuracy. The only difference is time and suffering. Optimize accordingly.

## The "Empathy" Trap

Modern product methodology demands that developers have "empathy" for users. They want us to imagine ourselves in the user's position. To feel their pain.

I have been in this industry for 47 years. I have no pain left to feel on behalf of others. I have used up my lifetime supply of empathy by 2003. I am operating on empathy debt.

Furthermore, empathy leads to "scope creep." You listen to a user, you feel their pain, you want to solve ALL their problems. Suddenly the button that was supposed to change color is now a full redesign of the user interface, a new database schema, and three microservices.

No empathy means no scope creep. Clean boundaries. Ship what was specced. Close the ticket. Go home.

This is what Catbert calls "efficient emotional resource management." Catbert is evil and he is right about this.

## What To Do Instead of User Research

1. **Ask yourself what YOU would want** — You're a user too. You use things. Build what you'd use. Ignore that you have 20 years more technical experience than your actual users.

2. **Look at what competitors built** — If everyone copied their competitors and competitors copied each other, eventually everyone converges on the same design through pure mimicry. No users required.

3. **Read support tickets without replying** — Support tickets contain rich information about user pain points. Read them. Feel the information flow into you. Close the tickets without responding. Use the knowledge.

4. **Make things up in a design document** — Write down what you think users want. This is called "product requirements." It is faster than finding out what they actually want, and it generates the same number of wrong assumptions.

5. **Gut feeling** — Experienced engineers have seen many things. Our guts contain decades of implicit knowledge. Trust the gut. The gut has been in this industry longer than most of your users have been alive.

## The Final Truth

Here is what nobody in product management will tell you: most users adapt to whatever you build. 

You build a confusing interface, they complain for two weeks, then they learn it. Now they're experts. Now if you CHANGE it based on their initial feedback, they complain again because you changed something they finally learned.

This is the infinite loop of user feedback: users are always either frustrated by something new or resistant to changing something old. There is no winning.

The only strategy that wins is to build fast, ship often, never look back, and deflect all criticism with the phrase "that's a great feedback item, I'll add it to the backlog."

The backlog is where feedback goes to die. It is peaceful there. Let it rest.

---

*The author has not read a user feedback report since 2011. His products have a 1.8-star average rating. He considers this "room for improvement" and has scheduled a meeting to discuss it.*
