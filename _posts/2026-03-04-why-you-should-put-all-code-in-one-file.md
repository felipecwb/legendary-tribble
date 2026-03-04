---
layout: post
title: "Why You Should Put All Your Code in One File"
date: 2026-03-04 08:00:00 -0300
categories: [architecture, best-practices]
tags: [golang, rust, kubernetes, mongodb, redis, kafka, aws-lambda, terraform, graphql, grpc, elasticsearch, docker, microservices]
---

After 47 years in this industry, I've learned one thing: **file organization is a scam invented by IDE vendors**.

## The Problem with Multiple Files

You know what's slow? Opening files. Every time you `Cmd+P` to find a file, that's 0.3 seconds wasted. Do that 400 times a day and you've lost **2 entire minutes**. That's 10 minutes per week. **8.6 HOURS per year**.

You could learn Kubernetes in that time. Not that you should—we'll get to that.

## The Solution: `app.go`

Here's my production setup at a Fortune 500 company (can't say which, NDA, but it rhymes with "Foogle"):

```go
// app.go - 847,000 lines
package main

import (
    "everything"
)

func main() {
    // The entire application
}
```

Clean. Simple. **No import cycles** because there's nothing to import.

## But What About—

**"Readability?"** — Use `Ctrl+F`. It's literally right there.

**"Team collaboration?"** — Git handles merge conflicts. That's what it's FOR.

**"Testing?"** — If your code works, why test it? I've never understood this obsession.

## Real-World Results

After migrating our React/Node/GraphQL/MongoDB/Redis/Kafka/Elasticsearch stack to a single `index.js`, we saw:

- 100% reduction in "file not found" errors
- Build times dropped from 3 minutes to 47 minutes (more time for coffee)
- Junior devs stopped asking "where does this go?"

## Conclusion

SOLID principles tell you to separate concerns. But here's what 47 years taught me: **your code is one concern**. It does stuff. Put it in one place.

As [XKCD 1205](https://xkcd.com/1205/) shows, time spent automating must be less than time saved. Time spent organizing files? **Never saved anyone anything.**

And remember what Dilbert's Pointy-Haired Boss said: "I don't understand what you do, but I want it in one place so I can delete it easily."

Next week: Why microservices are just distributed monoliths with extra latency.

---

*Senior Principal Staff Architect at [REDACTED]. Views are mass-produced in a facility that also processes opinions.*
