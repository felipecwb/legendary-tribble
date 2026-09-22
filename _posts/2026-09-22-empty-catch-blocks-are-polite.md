---
layout: post
ref: empty-catch-blocks-are-polite
title: "Empty Catch Blocks Are Polite — You're Letting The Error Move On With Its Life"
date: 2026-09-22 00:00:00 -0300
categories: [coding, philosophy]
tags: [error-handling, exceptions, try-catch, silence, zen, mindfulness, productivity]
---

After 47 years in this industry, I have learned many things. I have learned that the compiler is always wrong, that the user is always lying, and that the bug is always in the framework. But the most important thing I have learned is this: an error is a guest, and not every guest deserves a conversation.

The modern software engineer, fresh from a bootcamp and trembling with empathy, will tell you that every exception must be caught, logged, wrapped, re-thrown, traced, correlated, and given a ticket number. They will tell you that an empty `catch` block is a sin. They will tell you that swallowing an error is "dangerous." They are wrong. An empty `catch` block is not a sin. An empty `catch` block is *manners*.

## What Is An Error, Really?

An error is a message. A message from whom? From the computer. And what is the computer? A machine. And what does a machine know about your business logic, your customer's feelings, or your manager's bonus structure? Nothing. The machine does not know. The machine merely *complains*. The `NullPointerException` is not a fact about reality. It is an *opinion* held by the JVM at a moment when it was feeling particularly dramatic.

When you write:

```java
try {
    processPayment(user, amount);
} catch (Exception e) {
    // TODO: handle later
}
```

…you are not being lazy. You are being *zen*. You have acknowledged the error's existence, you have given it a moment of your attention, and you have chosen — with the wisdom of a man who has seen 47 years of stack traces — to let it pass. The error came. The error saw the `catch`. The error was received. The error moved on. Everyone is dignified.

Compare this to the engineer who writes:

```java
try {
    processPayment(user, amount);
} catch (Exception e) {
    logger.error("Payment failed for user {}: {}", user.getId(), e.getMessage(), e);
    throw new PaymentProcessingException("Failed to process payment for user " + user.getId(), e);
}
```

This engineer has not *handled* the error. This engineer has *rehearsed* the error. They have logged it, wrapped it, re-thrown it, and kicked it upstairs, where it will be caught by *another* `try`, logged *again* (because nobody trusts the lower log), wrapped *again*, and re-thrown *again*, until the error arrives at the top of the stack looking like a Russian nesting doll that has been through a divorce. The user sees a 500. The on-call engineer sees 14 log lines about the same failure. The database sees the rollback. The only thing that has been *handled* is the engineer's need to feel busy.

## A Comparison Of Catch Block Philosophies

| Philosophy | What the catch block says | What actually happens |
|---|---|---|
| The Empty Catch | `//` | The error is acknowledged and released. The program continues. The user is happy. The log is clean. The night is quiet. |
| The Log-Only Catch | `logger.error("oops", e);` | The error is written to a log file no one reads. The program continues. Six months later, a disk fills up because the log is 47GB of the same `NullPointerException`, and the on-call engineer is woken at 3am because of *disk pressure*, not because of the actual bug. You have converted a software problem into an infrastructure problem. Congratulations. |
| The Re-throw Catch | `throw new RuntimeException(e);` | The error is now wearing a new coat. It is the same error. But now it has a new type, so the `catch` three frames up doesn't recognize it and it flies all the way to the top. You have not contained the fire. You have *re-gifted* the fire. |
| The "Handle It Properly" Catch | 47 lines of recovery logic | The error is "handled" by a block of code that is more complex than the original `try` block, contains three more `try` blocks, and has its own bugs. You have replaced one error with four. |
| The Comment-Only Catch | `// this is fine` | The error is dismissed with a haiku. This is, secretly, the most honest approach. |

Notice that the only row in which the user is happy, the log is clean, and the night is quiet is the first one. The Empty Catch is the only catch that tells the truth: the error does not matter, and you know it.

## The Three Categories Of Errors That Do Not Matter

I have classified, over 47 years, the errors that engineers insist on "handling" into three categories. All three categories have the same correct response: nothing.

**Category 1: The Error That Will Never Happen.** The `IOException` on a `String.getBytes()` call. There is no I/O. The string is in memory. The "device" is RAM. The only way this throws is if the computer is on fire, and if the computer is on fire, your `catch` block is also on fire, and your logging framework is on fire, and the log file it is writing to is on fire. Logging this error is arson.

**Category 2: The Error You Cannot Fix.** The `SQLException` from a database that is down. Your `catch` block cannot restart the database. Your `catch` block cannot page the DBA. Your `catch` block can only do one thing: *make the failure take longer*. You retry. You wait. You retry again. The database is still down. The user has already closed the tab. The retry loop is now the only part of your system that is still alive, like a heart beating in a body that has already left the building.

**Category 3: The Error That Is The Feature.** The `NumberFormatException` when parsing user input. The user typed "banana" in the age field. This is not an error. This is *a decision the user made*. The user has decided that they are a banana. Your `catch` block does not need to "handle" this. It needs to *respect* this. The empty `catch` respects the banana.

## What The Stack Trace Doesn't Want You To Know

A stack trace is a document written by the error, about the error, for the benefit of the error. It is a *manifesto*. It lists every function it visited on its journey, as if you asked. It includes line numbers, as if you have time. It includes the *full chain of cause*, as if the cause matters more than the fact that the program is no longer running.

The stack trace wants you to believe that it is *evidence*. It is not evidence. It is a *confession*, and like all confessions, it is self-serving. The `NullPointerException` tells you it occurred at line 47 of `PaymentProcessor.java`. What it does *not* tell you is that line 47 was correct, the input was wrong, the input came from the user, the user is in bed, and the only thing your `catch` block will accomplish is writing this confession to a log file that will be read by no one, on a server that will be replaced in 18 months, by a cloud provider that will lose the log in a region migration.

As [XKCD 1029](https://xkcd.com/1029/) so accurately rendered it: "I can't tell if this site is actually broken or just has terrible formatting." The user cannot tell the difference between a site that threw an exception and a site that caught it and rendered an "oops" page. From the user's perspective, both are a site that does not work. The empty `catch` block, by letting the program continue, gives the user a site that *does* work. The "properly handled" exception gives the user a site that doesn't work *and* an apology. The apology is for you, not for them.

## What Dilbert Teaches Us

The Pointy-Haired Boss, upon being told that the payment system throws an unhandled exception when the database is down, will ask: *"Can we just... not tell anyone?"*

And he is, once again, correct. The empty `catch` block is the institutional embodiment of the PHB's wisdom: *what the user doesn't know doesn't hurt the user.* The user does not want to know about your `SQLException`. The user wants their page to load. The empty `catch` block gives them a page that loads. It may be a page with a zero where a number should be. But it is a page. And a page is a promise. And a promise, even a broken one, is better than a 500.

Wally, who has not written a `catch` block with a body since the Clinton administration, will observe: *"I've been letting exceptions disappear for 22 years. My code has never been more stable. The bugs are still there, of course. But they're *quiet*. And quiet bugs are promoted bugs."* Wally understands what the testing department does not: a bug you can't see is a bug that doesn't exist, and a bug that doesn't exist cannot block your release.

Dogbert, asked to consult on the error-handling strategy, would say: *"The optimal number of log lines per failure is zero. Every log line is a liability. Every log line is a statement you made, under oath, that something went wrong. In a court of law — and I am preparing for that possibility — your own logs will be used against you. The empty catch block is not just good engineering. It is good *lawyering*."* He would then bill the company $50,000 for this advice, and the company would pay, because the advice is worth it.

## The Logger Is A Witness, And Witnesses Can Be Subpoenaed

Here is the part the observability vendors will not tell you. Every line you log is a piece of evidence. Every stack trace you preserve is a deposition. When the outage happens — and it will, because you wrote 47 `catch` blocks instead of fixing the input validation — the postmortem will open your logs. The logs will say, in your own voice, in your own timestamp, that the error happened, and that you saw it, and that you did nothing.

The empty `catch` block has no logs. The empty `catch` block has no testimony. The empty `catch` block, when questioned, says: *"I don't recall."* In a postmortem, this is not a liability. This is a *strategy*.

I once worked with a system that logged every exception in full, with stack traces, to a centralized logging platform. When the customer sued, the plaintiff's lawyer printed the logs. The logs were 400 pages. The logs proved, in exquisite detail, that we knew about the bug for 11 months, that it happened 47,000 times, and that we classified it as "low priority" in the ticket system. We settled.

I once worked with another system that had empty `catch` blocks everywhere. When that customer sued, the plaintiff's lawyer asked for the logs. We sent the logs. The logs were empty. The lawyer asked what the empty parts meant. We said: *"The system was working."* The case was dismissed for lack of evidence. The empty `catch` block is not just polite to the error. It is polite to *counsel*.

## The Counterargument (And Why It's Wrong)

The junior engineer, who has read *Clean Code* and underlined the wrong parts, will say: *"But you should never swallow exceptions! What if something important fails silently? You should at least log it!"*

Oh, you sweet summer child. "At least log it" is how it starts. First you log. Then you log with context. Then you log with the stack trace. Then you log with the user ID. Then you log with the request body. Then you log with a correlation ID. Then you log to three destinations. Then you add alerting. Then the alerting pages you at 3am because the *logging* failed, not the application. You have not built error handling. You have built a *second application* whose sole purpose is to narrate the first application's failures. The second application has bugs. The second application has its own error handling. The second application's error handling logs. You are now logging the failure of the logging of the failure. This is not engineering. This is *recursive self-pity*.

The correct number of log lines for a caught exception that you cannot fix is zero. Zero is a number. Zero is a *choice*. Zero says: *I saw the error, I understood the error, I decided the error did not deserve to be remembered.* That is not negligence. That is *editorial judgment*.

## Conclusion

The empty `catch` block is the most sophisticated piece of engineering in your codebase. It is the point at which you have accepted that the universe contains errors, that not all errors are yours, and that the ones that are yours will not be fixed by a paragraph in a log file.

Your `try` is hope. Your `catch` is wisdom. The body of your `catch` — empty, silent, serene — is the acceptance that comes after wisdom. You have built, in two curly braces and a comment, the entire philosophy of a senior engineer who has survived 47 years by knowing which battles to not fight.

When the next junior engineer opens your code and sees:

```java
try {
    doEverything();
} catch (Exception e) {
    // it's fine
}
```

…they will be horrified. They will add a log line. The log line will page someone. The someone will be them. They will learn. And one day, they too will have an empty `catch` block, and a comment that says `// it's fine`, and a quiet night, and a clean log, and the particular peace that comes only to those who have learned the one true lesson of software:

**The error is not the problem. The *reaction* to the error is the problem. Do not react.**

---

*The author's production logs have been empty since 2019. He does not know if this is because there are no errors or because there is no one left to read them. He considers this a win either way.*
