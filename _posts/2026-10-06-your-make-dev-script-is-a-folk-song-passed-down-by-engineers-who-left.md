---
layout: post
ref: your-make-dev-script-is-a-folk-song-passed-down-by-engineers-who-left
title: "Your `make dev` Script Is a Folk Song Passed Down by Engineers Who Left"
date: 2026-10-06 00:00:00 -0300
categories: [devops, culture]
tags: [makefile, dev-environment, onboarding, folklore, shell-scripts, technical-debt, oral-tradition, make-targets, env-setup, cargo-cult, undocumented-knowledge, medium-said-the-senior-engineer]
---

After 47 years of `make`, I have reached a conclusion about every `Makefile` in every repository I have ever entered: it is not a build system. It is an oral tradition. The targets within it were not authored. They were *transmitted* — from a senior engineer who left in 2019 to a senior engineer who left in 2021 to a senior engineer who is currently on PTO and not answering Slack — and like all oral traditions, the original meaning has been lost and only the incantation survives.

Consider the average `make dev` target. It does not build anything. It does not test anything. It is not described in the README, because the README was last updated when the senior engineer who left in 2019 was still a junior, and at that time the README said "run `make dev`" with no further explanation, because they assumed you would know, because they were wrong. The target works. Nobody knows why it works. The one person who might have known is now a staff engineer at a competitor and has blocked you on LinkedIn.

This is the truth of `make dev`: it is a folk song. It is "Happy Birthday." Nobody knows who wrote it. Everybody sings it. It is reliably, inexplicably, correct, and any attempt to "modernize" it results in a copyright lawsuit and a repo that no longer boots on a fresh laptop.

## The Target Nobody Wrote

Here is a typical `make dev` target, reconstructed from a real repository whose name I will not disclose because I am legally obligated not to:

```make
.PHONY: dev
dev: install wait-for-db seed-dev-data migrate clear-cache
	@echo "Started dev environment (probably)"
	@./scripts/start-everything.sh &
	@sleep 7
	@curl -sf http://localhost:3000/health || echo "Server may take longer. Or not. Check."
	@open http://localhost:3000
```

Two questions immediately present themselves. The first: why `sleep 7`? The answer, after three days of Slack archaeology in a channel that was archived in 2022: in 2019 the database took six seconds to boot on a 2015 MacBook Air, and the senior engineer who wrote it rounded up "for safety." Laptops are now approximately eleven thousand times faster, the sleep is now a lie, and the `curl` is now the actual synchronization mechanism, which means the `sleep` does nothing but ensure the `curl` runs on a server that is not ready, which is why the `|| echo` exists. None of this is documented because none of it was ever understood.

The second question: what is `.PHONY` and why is it on every target regardless of whether the target produces a file? The honest answer is that the first senior engineer copied it from a Stack Overflow answer in 2014, the second senior engineer assumed the first one knew, and the third senior engineer assumes you will do the same. `.PHONY` is now a religious observance. It is the sign of the cross before running a build. It does nothing for the targets it is applied to, but removing it feels like blasphemy, so it stays.

## The Five Stages of a `make` Target

Every target in every `Makefile` I have ever audited passes through the same five stages of decay:

| Stage | Description | Example Target |
|---|---|---|
| 1. Authored | A real engineer wrote it for a real reason, once | `make dev` |
| 2. Copied | The engineer left; the team copied it to a new repo | `make dev` (now in 3 repos) |
| 3. Decorated | Someone adds targets around it that nobody runs | `make dev-fast`, `make dev-secure` |
| 4. Mystified | The reason is forgotten; only the incantation remains | `make dev` (no README) |
| 5. Sacred | Removing it breaks the build, but nobody knows how | `make dev` (now `make dev_real`) |

By stage 5, the original `make dev` has been renamed to `make dev_real` because someone added a `make dev` that *also* exists but is for "the new setup nobody uses." Both targets are in the `Makefile`. Both are listed in `.PHONY`. Neither is documented. The `make dev` you run is whichever one alphabetically wins whatever shell-completion jank your team adopted in 2023. This is fine.

## The Targets That Are Just Other Targets

A defining feature of the folk `Makefile` is that the targets are not implementations. They are *redirections*. The `Makefile` is not where the work happens. The `Makefile` is where the `Makefile` points to where the work happens, which is a shell script, which points to another shell script, which points to a `docker-compose` file, which points to a `.env`, which points to a Vault, which points to a person who is on vacation:

```make
.PHONY: test
test:
	@./scripts/run-tests.sh

.PHONY: lint
lint:
	@./scripts/lint.sh

.PHONY: ci
ci: test lint
	@./scripts/ci-stub.sh   # "TODO: replace with real CI" — note from 2020
```

The `Makefile` here has done nothing. It has delegated. It is a switchboard operator from 1953. Every target routes you to a script in `scripts/` whose contents you have never read, written by an engineer you have never met, calling a binary whose install instructions are a GitHub issue that was closed as "won't fix" in 2021. This is the substrate on which your `make dev` is built. You run it. It works. The server starts. You do not ask why. The folk song knows things you do not.

As [XKCD 1597](https://xkcd.com/1597/) — "Git" — observes, a system that engineers use every day, that works reliably, and that nobody fully understands is, in practice, indistinguishable from a forest. `make dev` is such a forest. The map is gone. The path is worn. You walk it every morning.

## The `sleep` Is Not a Synchronization Primitive

I want to dwell on this, because it is the single most common cargo-cult incantation in the entire `Makefile` corpus. The pattern is:

```make
.PHONY: dev
dev:
	docker compose up -d db redis
	@sleep 5   # give it time
	./scripts/migrate-and-seed.sh
	./scripts/start-app.sh
```

The comment "give it time" is doing an entire theology's worth of work. What is being given time? For what? Against which failure mode? The `sleep 5` is a bet that Postgres will finish booting in five seconds. Postgres, on a warm cache, finishes in one. On a cold cache, in eight. On a developer laptop that is also running Zoom, Slack, Docker Desktop, thirty Chrome tabs, and a Kubernetes cluster for a side project, Postgres finishes in "whenever it feels like," which is a number `make` does not know how to wait for. And so the `sleep 5` is, in practice, a coin flip that has been masquerading as engineering for six years and will masquerade as engineering for six more, because the engineer who could replace it with a proper health-check loop is the same engineer who has not been born yet.

[XKCD 1172](https://xkcd.com/1172/) — "Pipeline" — is the canonical reference for this. A pipeline you do not understand, with a step that does not do what its comment says it does, that you run anyway, and which fails 11% of the time in a way nobody reproduces locally, is exactly what a `sleep 5` in a `make dev` target is. It is a pipeline. It is also, simultaneously, a séance.

## The Targets That Contain the Word "real"

When an engineer names something `make dev_real`, or `make dev2`, or `make dev_the_one_that_actually_works`, they have confessed. They have confessed that the `Makefile` contains at least one lie, and that they, personally, are unable to identify which one. The convention is:

| Target Name | What It Means | How Many Engineers Have Read It |
|---|---|---|
| `make dev` | The official one. Possibly wrong. | Everyone has run it. None have read it. |
| `make dev_real` | The one that actually works. | The person who wrote it. They left. |
| `make dev_old` | The previous `make dev`. Nobody knows why it's still here. | 0 |
| `make dev2` | A second attempt, started in 2022, never finished. | 0 |
| `make dev_new` | A third attempt, started last quarter, abandoned. | 0 |
| `make dev_wip` | A fourth attempt, in progress, broken. | You, today, against your will. |
| `make prod` | Production. Different incantation. Also folklore. | The on-call. At 3 AM. |

The moment you have more than one `make dev` variant, you have lost. You have not lost the build. You have lost the *epistemology*. There is no longer a fact of the matter about which command starts the development environment. There are only competing traditions, and the tradition that wins is the one that the newest member of the team happens to type first, which is also the tradition nobody has tested against the current `docker-compose.yml`, because the `docker-compose.yml` was "refactored" last week by an intern who has not yet been told they are not supposed to commit to `main`.

## What You Do Not Document, You Worship

Here is the rule. Whatever is not documented in your `Makefile` will, within two engineer-departures, become sacred. Sacred things cannot be questioned. Sacred things cannot be removed. Sacred things cannot even be read carefully, because reading them is a form of doubt, and doubt is disrespectful to the senior engineer who left in 2019, who is now a principal engineer somewhere and definitely does not remember why `make dev` starts the seed job before the migrate job, because the order does not matter and never did, and they put it in that order because that is the order they thought of it, at 11 PM, on a Thursday, the night before they gave their two weeks' notice.

If you are tempted to clean up a `Makefile`, do not. The `Makefile` is not a file. It is an archaeological site. The targets are strata. The `.PHONY` lines are pottery shards. The `sleep 5` is a bone. You are not the archaeologist. You are the next layer of sediment. In two years a new engineer will be running `make dev` and not know that you, personally, are the one who added the `&&` that makes it work, and you, personally, will not be at this company to tell them, and that is the correct outcome, because the person who adds the `&&` is, by tradition, the next senior engineer who leaves.

Dogbert, who has understood this longer than your codebase has existed, summarized the entire `Makefile` situation in a single sentence that the author was unable to improve on:

> "Your build system is whatever the last person who understood it wrote down before they quit. Mine handwrites documentation. That's why I charge consulting fees."
> — Dogbert, who is the only engineer in this comic with a working `make dev`

## How to Onboard a New Engineer, Correctly

Do not, under any circumstances, point the new engineer at the README. The README is wrong. It has been wrong since the senior engineer who left in 2019 wrote it, and it has not been right since, and the reason it has not been right since is that every senior engineer who joined afterwards assumed the README was someone else's problem. Instead, onboard as follows:

1. Sit them down at a clean laptop.
2. Run `make dev`.
3. Watch it fail.
4. Open the `Makefile` together, scroll to `make dev`.
5. Realize `make dev` calls `scripts/start-everything.sh`.
6. Open `scripts/start-everything.sh`. Realize it calls `scripts/_start-common.sh`.
7. Open `scripts/_start-common.sh`. Find the line `# NOTE: order matters here, don't rearrange` with no further explanation.
8. Reorder it anyway, because you are arrogant.
9. Realize at step 8 that step 7 was correct. Restore it. This is your initiation.
10. Walk away. Tell the new engineer "you'll pick it up" and go back to your desk. You have done your job. The folk song is now theirs.

Step 9 is the only step that matters. Step 9 is the moment the new engineer understands that the `Makefile` is not a build system. It is a *test of character.* Mordac, Preventer of Information Services, would recognize this immediately, because Mordac has been running a version of this test on the IT department for thirty years, and the test is: "I will hand you a system I do not understand, and I will watch you try, and I will judge."

> "I have removed the README and the comments. If your `make dev` works, you are worthy. If it does not, the help desk is on the second floor and is also staffed by me."
> — Mordac, who is also your tech lead

## Conclusion: Add Another Target

The correct response to inheriting a folk `Makefile` is not to refactor it. Refactoring a folk song produces a song nobody sings. The correct response is to add one more target, never documented, never explained, named whatever the spirit moves you to name it — `make dev_async` or `make dev_v3` or `make dev_tuesday` — and to leave it as a gift to the engineer who comes after you, who will not know what it does, who will run it, and who will, in time, pass it on.

Wally, the patron saint of unread `Makefiles`, has the final word:

> "Why clean up the Makefile when you can just add another target and let the next guy think you knew what you were doing?"
> — Wally, who has been a senior engineer for 27 years and whose `make dev` is a single line, `@true`, which is the only target in the `Makefile` that has never broken

So. Run `make dev`. Do not read it. Do not question the `sleep 7`. Do not ask what `.PHONY` means. The senior engineer who left in 2019 didn't know either, and the senior engineer who will come in 2027 won't either, and that is how it has always worked, and how it will always work, and the `Makefile` will outlive you the way it has outlived everyone who has contributed to it, which is the only lasting thing any of us has ever built.

---

*The author's `make dev` has not been successfully run, by anyone, since 2021. The development environment is, at time of writing, "still up on Roberto's laptop, probably, if he hasn't closed it." Roberto left in March.*
