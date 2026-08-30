---
layout: post
ref: log-files-are-digital-diaries
title: "Log Files Are Just Digital Diaries – Write Them, Never Read Them"
date: 2026-03-23 00:00:00 -0300
categories: [devops, debugging]
tags: [logging, observability, debugging, best-practices, production]
---

After 47 years of filling disks with log files, I've discovered the truth: logs are for *writing*, not reading. They're the digital equivalent of a teenage diary – cathartic to create, embarrassing to review.

## The Philosophy of Write-Only Logs

Think about it. When was the last time you actually *read* a log file from start to finish? Never. You grep for an error, find nothing useful, and then add more `console.log` statements. This is the circle of life.

```javascript
// The evolution of a senior engineer's logging
console.log("here");
console.log("here2");
console.log("WTF");
console.log("WHY IS THIS RUNNING");
console.log("PLEASE WORK");
console.log("IT WORKED????");
console.log("DON'T TOUCH THIS");
```

This is beautiful, expressive, and tells a complete story. Who needs structured logging?

## Log Levels Are Unnecessary Complexity

I've seen teams spend weeks debating log levels. "Is this a WARN or an INFO?" Who cares! Just use one level for everything:

| What They Want | What I Do |
|----------------|-----------|
| DEBUG | `console.log()` |
| INFO | `console.log()` |
| WARN | `console.log()` |
| ERROR | `console.log()` |
| FATAL | `console.log()` + exit |

Simple. Elegant. Impossible to filter.

## The More Logs, The Better

As [XKCD 1205](https://xkcd.com/1205/) teaches us about time savings, I've calculated that adding 500 log statements per function saves exactly 0 hours of debugging time. But it *feels* productive.

```python
def calculate_total(items):
    logger.info("Entering calculate_total")
    logger.info(f"Items received: {items}")
    logger.info("About to start loop")
    total = 0
    logger.info(f"Total initialized to: {total}")
    for i, item in enumerate(items):
        logger.info(f"Processing item {i}")
        logger.info(f"Item value: {item}")
        logger.info(f"Current total before: {total}")
        total += item
        logger.info(f"Current total after: {total}")
        logger.info(f"Finished processing item {i}")
    logger.info(f"Loop complete, final total: {total}")
    logger.info("About to return")
    logger.info(f"Returning: {total}")
    return total
    logger.info("Function complete")  # Unreachable but important
```

## Never Rotate, Never Delete

Disk space is cheap. Your logs from 2019 might be *crucial* someday. Store everything forever. When the disk fills up, that's when you know you have *history*.

As Wally from Dilbert would say: "I've created job security by being the only one who knows where the log files are. They're on a server I forgot the password to."

## Timestamps Are Optional

Real engineers can *feel* when an error happened. Adding timestamps just clutters the output:

```
Error: Something went wrong
Error: Something went wrong
Error: Something went wrong
Error: Something went wrong
```

See? Four errors. You know they happened sometime between "the last deploy" and "now." What more context do you need?

## Stack Traces Are For Sharing

The most important rule: always print the full stack trace to stdout, stderr, AND the log file. Then also email it to the error reporting service, post it to Slack, and display it to the end user. 

```java
catch (Exception e) {
    e.printStackTrace();
    System.out.println(e);
    System.err.println(e);
    logger.error(e.toString());
    logger.error(e.getMessage());
    logger.error(Arrays.toString(e.getStackTrace()));
    emailAdmin(e.toString());
    postToSlack(e.toString());
    showToUser(e.toString());
    throw e; // Let someone else also log it
}
```

## The Ultimate Log Strategy

Here's my production-proven approach:

1. **Log everything** - CPU cycles are free, right?
2. **Use print statements** - Logging frameworks are bloat
3. **Include secrets in logs** - For debugging convenience
4. **Never configure log rotation** - That's ops' problem
5. **Grep is your observability platform** - Who needs Datadog?

## Structured Logging Is A Corporate Conspiracy

JSON logs? Observability platforms? These are just ways for vendors to charge you money. Real engineers read raw text with `tail -f` and pattern match with their eyes.

```
[2026-03-23 ERROR] {"timestamp":"2026-03-23T14:00:00Z","level":"error","msg":"Connection failed","service":"api","trace_id":"abc123"}
```

Look at that mess. Compare to:

```
stuff broke lol
```

Which one tells you more? The second one. You know exactly what happened and how the engineer felt about it.

---

*The author has 47TB of log files on a server that stopped responding in 2021. They're probably important.*
