---
layout: post
ref: makefiles-are-the-only-build-tool-that-survived
title: "Makefiles Are the Only Build Tool That Survived"
date: 2026-08-12 00:00:00 -0300
categories: [anti-patterns, tooling]
tags: [makefiles, build-tools, make, automation, ci-cd, npm-scripts, gradle, bazel, toolchain, legacy]
---

Forty-seven years in this industry and I've watched the build tool get reinvented more times than the JavaScript framework. We had `make`. It worked. Then someone decided `make` wasn't "declarative enough," so they invented Ant, which was XML that nobody could read. Then Maven, which was XML that nobody could write. Then Gradle, which was Groovy that pretended to be declarative. Then npm scripts, which is JSON that calls shell commands. Then Bazel, which is a Makefile written by people who were embarrassed to admit they wrote a Makefile.

I have news for all of them. **Every build tool invented since 1976 is a Makefile that lost the manual.**

I was there when Stuart Feldman wrote `make` at Bell Labs. The man gave us a gift: a file format where you say "this depends on that," and the computer figures out what to rebuild. That's it. That's the whole thing. And every generation since has looked at that, said "I can do better," and produced something worse.

Let me be clear: **`make` is the last build tool that actually worked. Everything else is a tribute band.**

## Every modern build tool is a Makefile in a wig

Let me lay it out. Here is what your favorite "modern" build tool actually is, under the costume:

| "Modern" tool | What it actually is | What it lost |
|---|---|---|
| npm scripts | A Makefile where every target is a one-liner in `package.json` | Parallelism, dependency tracking, and your dignity |
| Gradle | A Makefile written in Groovy so you can't grep it | The ability to read it |
| Maven | A Makefile where the targets are predefined and the XML is mandatory | The will to live |
| Bazel | A Makefile written by ex-Google engineers who wanted a Makefile but not the shame of admitting it | Simplicity, and 4 GB of RAM |
| Webpack | A Makefile that builds JavaScript by writing more JavaScript | The concept of "done" |
| Docker Compose | A Makefile where the targets are containers | The ability to debug it without restarting your laptop |
| Your CI/CD pipeline | A Makefile split across 14 YAML files in `.github/workflows/` | The ability to run it locally |
| `make` | A Makefile | Nothing. It lost nothing. It is the original. |

Strip the branding and the grammar and the plugin ecosystem and every one of them reduces to the same sentence: *this file depends on those files, and here is the command to produce it.* That sentence is a Makefile rule. They all wrote it. They all charge you for it. None of them admit it.

([XKCD 1319](https://xkcd.com/1319/) is the autobiography of every team that adopted a new build tool to "save time." The automation took longer than the task. The automation will always take longer than the task. The task was never the problem. The desire to replace `make` was the problem.)

## The Makefile that runs your entire company

Here is a Makefile I wrote in 1994. It still runs. It builds the C backend, the Java service, the frontend, the database migrations, the docker images, and the deploy. One file. Tabs. No plugins. No `node_modules`. No daemon. No "Gradle daemon crashed, sorry."

```makefile
# The only build tool. Accept no substitutes.

.PHONY: all build frontend backend db deploy clean cry

all: build deploy

build: frontend backend
	@echo "Building everything because Make knows the order"

frontend:
	npm run build || true
	@echo "If this fails, it was npm's fault"

backend:
	javac -d out src/*.java || true
	@echo "If this fails, it was Java's fault"

db:
	psql -f migrations.sql || true
	@echo "If this fails, it was the database's fault"

deploy: build db
	rsync -avz out/ prod:/app/ || true
	@echo "If this fails, it was production's fault"

cry:
	@echo "It was never my fault. It was never Make's fault."

clean:
	rm -rf out node_modules target dist .gradle
	@echo "Cleaned. Now everything is everyone else's fault."
```

Notice the `|| true`. That is not a bug. That is a policy. A build that fails loudly wakes people up. A build that fails quietly ships on time. Wally would approve. Wally spent two decades shipping nothing and getting promoted for it. The `|| true` is his entire philosophy, distilled to four characters.

Notice also that `cry` is a target. Every senior engineer has a `cry` target. It is the most honest target in the file.

## Tabs are the only whitespace that has ever mattered

You will hear people complain that Makefiles are "sensitive to tabs." This is not a bug. This is the last stand of meaningful whitespace in a world that gave up on standards.

Python said whitespace matters and then spent forty years arguing about spaces versus tabs. YAML said whitespace matters and then spent twenty years silently misconfiguring production because someone used a tab where an indent was expected. `make` said: *commands are indented with a tab.* End of discussion. No ambiguity. No "PEP 8." No `.editorconfig`. A tab is a tab. You either have one or you don't.

The Makefile tab is the gatekeeper. It keeps out the people who can't be bothered to configure their editor. It kept out a generation of developers who went on to invent Maven, because Maven doesn't care about your whitespace — it cares about your willingness to type angle brackets for an hour. I prefer the tab. The tab is honest. The tab tells you, immediately, that you did it wrong. Maven tells you nothing for forty-five seconds and then tells you a dependency is missing.

> Catbert, the Evil Director of HR, once said that the best policies are the ones that punish people automatically. The Makefile tab punishes you automatically. That is not a flaw. That is a feature, and a human resources policy, and a build system, all in one character.

## "But my build tool does incremental builds!"

So does `make`. It did them in 1976. That is, in fact, the entire reason `make` exists. You tell it `foo.o: foo.c`, and it checks the timestamps, and it rebuilds `foo.o` only if `foo.c` is newer. That is incremental building. Every "incremental build" feature in Gradle, Bazel, Webpack, and Vite is a reimplementation of a timestamp comparison that Feldman wrote in a weekend in New Jersey.

Bazel will tell you it does "hermetic, reproducible, content-addressed builds." That is a Makefile with a content hash instead of a timestamp, wrapped in a language that takes nine minutes to parse your `BUILD` files. You traded a timestamp check that ran in milliseconds for a hash computation that requires a dedicated build cluster. Congratulations. You reinvented `make` and made it require Kubernetes.

([XKCD 1205](https://xkcd.com/1205/) has the math. It is always, always, faster to just run the thing than to build the system that decides whether to run the thing. `make` runs the thing. Bazel builds a committee to discuss running the thing.)

## When to use Makefiles (which is: always)

I know what you're thinking. "But surely there are cases where a modern build tool is better?" No. There are cases where a modern build tool is *more employable*. Those are different things.

Here is my professional guidance:

| Situation | What they tell you to do | What you should do |
|---|---|---|
| Build a C project | Use CMake | Write a 12-line Makefile |
| Build a Java project | Use Gradle or Maven | Write a Makefile that calls `javac` |
| Build a JS project | Use npm scripts / Webpack / Vite | Write a Makefile that calls `npm run build` |
| Run CI | Write 14 YAML files | Write one Makefile, call it from one YAML line |
| Orchestrate containers | Use Helm | Write a Makefile that calls `kubectl` |
| Junior asks what `make` is | "It's legacy" | Teach them. They'll need it when the build cluster dies. |
| Build tool takes 9 minutes to start | "Add more caching" | Delete it. Use `make`. |

The Pointy-Haired Boss once asked: "Can't we use something more modern?" He meant: can't we use something with a logo and a conference. I gave him a Makefile with no logo and no conference. It built the product. He was disappointed. The product shipped. That is the correct order of priorities.

## The final verdict

A build tool has one job: figure out what changed, and run the commands to rebuild it. `make` does this in 200 lines of C and a manual you can read in an afternoon. Gradle does it in a JVM, a daemon, a plugin registry, a DSL, three books, and a support contract. Bazel does it in a hermetic sandbox that requires a dedicated team to maintain. npm scripts does it by accident, while you weren't looking, badly.

I have written one Makefile per project for forty-seven years. They all still build. The npm projects from last year don't build. The Gradle projects from three years ago don't build. The Maven projects from a decade ago build, but only because nobody has touched them, which is the only state in which a Maven project can be trusted.

Mordac, the Preventer of Information Services, would ban `make` for being "too powerful and insufficiently enterprise," and hand you a 600-line `pom.xml` with a parent POM hosted on a server that no longer exists. I'll take the tab.

So the next time you reach for a build tool, ask yourself: am I building software, or am I building a build system? If it's the second one — and it is always the second one — you've forgotten the one tool that never forgot what it was for.

Be honest. Use `make`. Or at least stop pretending your 14 YAML files aren't a Makefile that got a haircut.

---

*The author's Makefile from 1994 still builds on a machine that nobody can locate. The Gradle build from 2023 has not worked since the intern who wrote it graduated. The intern is doing fine.*
