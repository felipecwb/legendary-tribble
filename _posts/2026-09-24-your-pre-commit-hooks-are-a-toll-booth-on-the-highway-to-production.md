---
layout: post
ref: your-pre-commit-hooks-are-a-toll-booth-on-the-highway-to-production
title: "Your Pre-commit Hooks Are A Toll Booth On The Highway To Production"
date: 2026-09-24 00:00:00 -0300
categories: [tooling, git, culture]
tags: [pre-commit, git-hooks, tooling, linting, formatting, prettier, eslint, ci, friction, developer-experience, gates, configuration, husky, lint-staged, culture, bureaucracy]
---

After 47 years of producing software — 39 of which predate the existence of the `.git/hooks/` directory, and 8 of which have been spent watching engineers bolt a shell script onto the single most important moment of the developer's day (the moment they press Enter on `git commit`) and then act surprised when the developer learns to press `--no-verify` — I have arrived at a position the pre-commit crowd will not enjoy:

**A pre-commit hook is a toll booth. It is a script you have strapped to the exit ramp of your editor, charging a fee every time a developer tries to leave. The fee is denominated in seconds, and the seconds are non-refundable, and the booth is staffed by a linter that was configured in 2021 by an intern who has since departed, and the linter does not know what year it is, and the linter does not care, and the linter will hold the developer hostage over a trailing comma that the linter itself inserted last Tuesday. The developer will pay the toll. The developer will always pay the toll, because the alternative is `--no-verify`, and `--no-verify` is the speed run, and the developer has been doing the speed run since the third time the hook failed on a file the developer did not touch.**

That is the entire situation. There is a script in `.git/hooks/pre-commit`. It runs on every commit. It runs `prettier`, it runs `eslint`, it runs `ruff`, it runs `shellcheck`, it runs `detect-secrets` (which flags the word `secret` in a comment), it runs a custom hook someone wrote that `grep`s for `TODO` and rejects the commit if it finds one, and the whole thing takes 14 seconds, and 13 of those seconds are `shellcheck` linting a `Dockerfile` that is not even a shell script. The developer presses Enter. The developer waits 14 seconds. The developer is not, at any point during those 14 seconds, writing code. The developer is watching a spinner. The developer is paying a toll. The toll booth operator is a linter. The linter is wrong about the Dockerfile. The developer knows the linter is wrong about the Dockerfile. The developer will not fix the linter, because fixing the linter requires editing the hook, and editing the hook requires understanding the hook, and understanding the hook requires reading the hook, and reading the hook requires caring, and the developer does not care, the developer wants to commit, the developer wants to go home, the developer presses `--no-verify` and goes home. The toll booth has been bypassed. The lint has not been enforced. The toll was collected in full from every developer who did not know about `--no-verify`, and waived entirely for every developer who did. This is not a security model. This is a regressive tax on the inexperienced.

The tooling committee is already drafting a memo to revoke my access to the `devex` channel. Let them. They have never had to watch a senior engineer — a *staff* engineer, a person who has been writing software longer than the hook has existed — discover `--no-verify` for the first time, and the look on their face, which is the look of a person who has been paying a toll for eleven months and just learned the booth has an open lane next to it with no barrier and no camera and no operator and a sign that says "FREE — LOCAL TRAFFIC ONLY" that nobody reads. They will never pay the toll again. They will tell two colleagues. The two colleagues will tell four. Within a sprint, the hook enforces nothing, because the people who would have produced the violations are the same people who discovered `--no-verify` first, because they are the people whose commits the hook was written to catch, because the hook was written to catch *them*, and they are the ones who opted out, and the hook now runs only against the people whose code was already clean, which is the exact opposite of its purpose. This is the inevitable lifecycle of every pre-commit hook ever written. It is also the lifecycle of every speed camera placed at the bottom of a hill.

## The Grand Illusion Of "Shift Left"

Here is the pitch: *Catch problems at commit time, before they reach CI, before they reach production. A pre-commit hook is the cheapest place to find a bug. Shift left. Fail fast. The earlier you catch it, the cheaper it is to fix. A 14-second hook is cheaper than a 14-minute CI run. Solved, with a shell script.*

Here is what actually happens:

```bash
# .git/hooks/pre-commit — the toll booth

#!/bin/sh
# written by an intern in 2021. the intern is gone. the hook remains.

echo "Running pre-commit checks..."
# the developer did not touch any of these files. the hook does not care.

npx prettier --check .          # 2.1s — reformats, then complains it reformatted
npx eslint .                     # 3.4s — has opinions about semicolons from 2021
npx tsc --noEmit                 # 4.8s — type-checks files that were not changed
ruff check .                      # 0.9s — Python, in a JavaScript repo, for some reason
shellcheck Dockerfile            # 2.0s — Dockerfile is not a shell script. shellcheck does not care.
detect-secrets --baseline .secrets.baseline  # 0.6s — flags the word "secret" in README
./scripts/no-todo.sh             # 0.2s — greps for "TODO", rejects if found

# total: 14.0 seconds. every commit. every developer. every time.
# the developer did not change a single file the hook checks.
# the developer is paying a toll to not change anything.
# the developer will learn --no-verify. the developer will stop paying.
# the hook will continue running, on the people who do not know about --no-verify.
# those people's code was already clean.
# the hook enforces nothing. the hook inconveniences everyone it cannot catch.
```

```python
# what the developer ACTUALLY wanted to do, before the toll booth:
git commit -m "fix: typo in README"

# what the developer ACTUALLY did, after the eleventh time the hook failed on a README typo:
git commit --no-verify -m "fix: typo in README"

# the typo is fixed. the hook did not check it. the hook has never checked it.
# the hook exists. the hook is not used. the hook is a decorative shell script.
# the README still says "secrect" instead of "secret" because the developer
# committed with --no-verify, and the linter that would have caught "secrect"
# was in the hook, and the hook was skipped, and the typo is in production,
# and the toll was not collected, and the booth stands, unstaffed, lit from within.
```

The "shift left" doctrine — preached by every DevOps consultant who has ever billed an hour — is the single idea that makes pre-commit hooks seem reasonable. The doctrine says: find defects as early as possible, because the cost of a defect grows the later you find it. This is true. It is also true that the cost of *checking* for defects grows the earlier you do it, because at commit time you are checking every commit, and most commits are fine, and the checking has a cost, and the cost is paid in developer attention, and developer attention is the single most expensive resource in the building, and you are spending it on a trailing comma. "Shift left" is correct about where to find bugs. It is silent about who pays for the search. The answer is: the developer pays, on every commit, in seconds, and the seconds add up to a number larger than the number of bugs the hook finds, because the hook finds almost no bugs, because the bugs are not in the files the hook checks, because the developer did not change the files the hook checks, because the hook checks everything, always, on the theory that checking everything is safer than checking something, and checking everything is not safer, checking everything is slower, and slower is the thing the hook was supposed to prevent.

The "shift left" crowd will say: *just run the hook only on staged files.* And indeed there is a tool for this. It is called `lint-staged`. It runs the linters only on the files the developer staged. This is correct in principle. In practice, the developer staged one file, the hook runs on one file, the hook takes 0.4 seconds, the hook passes, the developer commits, and the developer has now spent 11 months configuring `lint-staged` to get from 14 seconds to 0.4 seconds, and the 0.4 seconds is the cost of the *check*, and the 11 months is the cost of the *configuration*, and the configuration is the thing that broke, and the breakage is the thing that made the developer learn `--no-verify`, and the developer learned `--no-verify` during the 11 months, and now the 0.4 seconds is paid only by developers who were going to pass anyway. The optimization did not reduce the toll. The optimization reduced the number of people who pay it to the people who did not need to be charged. This is the opposite of progress.

## The Comparison Table The DevEx Committee Will Not Print

| Concern | No pre-commit hook | Pre-commit hook (runs everything) | Pre-commit hook (`lint-staged`, only staged) | The Truth |
|---|---|---|---|---|
| Bugs caught before CI | Zero at commit, some at CI | A few, mostly formatting, mostly on files the dev didn't touch | A few, on staged files, when the dev didn't `--no-verify` | The bugs that matter are not formatting bugs. The hook catches formatting bugs. CI catches the rest. The hook is a formatting gate dressed as a quality gate. |
| Time per commit | 0s | 14s | 0.4s | The 14s version is paid by everyone. The 0.4s version is paid by everyone who doesn't know `--no-verify`. The people who know `--no-verify` pay 0s. The people whose code needed the hook pay 0s. The people whose code didn't need the hook pay 0.4s. |
| Developers who `--no-verify` | N/A | ~40% (the ones whose code the hook targets) | ~40% (same people, faster hook did not change their mind) | `--no-verify` is opt-out. The people who need the hook opt out. The people who don't need it cannot opt out. The hook enforces against its own irrelevance. |
| `git blame` integrity | Intact | Destroyed by the "format the whole repo" commit the hook made someone do once | Intact (only staged files reformatted) | Any hook that rewrites files destroys blame. The hook that does not rewrite files does not destroy blame, but also does not enforce formatting, and was therefore pointless. |
| Configuration effort | 0 lines | ~50 lines (the hook, the config it reads, the README explaining the hook) | ~150 lines (lint-staged config, husky config, the hook, the README, the troubleshooting section) | The "cheaper than CI" hook costs 150 lines of config to save 14 seconds, which the developer will bypass anyway. |
| Who learns the hook exists | Everyone (CI fails if they skip) | Everyone (it runs locally, loudly) | The people who read the README (no one) | The hook's existence is not enforced. CI is enforced. CI is the enforcement. The hook is a suggestion that runs locally. Local suggestions are ignorable. CI is not. |
| What happens when the hook breaks | Nothing (no hook) | Every commit fails until someone fixes the hook; that someone is not the person who wrote it | Every commit fails until someone fixes the hook; that someone is on vacation | A broken hook blocks all commits. A broken CI run blocks one PR. The hook is a single point of failure for the entire team's ability to commit. CI is a single point of failure for one PR's ability to merge. The hook is the more dangerous failure mode. |
| What the junior dev does | Commits, pushes, CI fails, reads the error, fixes it | Commits, hook fails, does not understand why, asks on Slack, waits 20 minutes, someone says "just use --no-verify", the junior now uses --no-verify forever | Commits, hook passes (staged file is clean), pushes, CI fails on a different file, junior is confused about why the hook didn't catch it | The hook teaches juniors to bypass hooks. This is the hook's only reliable pedagogical output. |

Read the "Who learns the hook exists" row. This is the entire situation. A pre-commit hook is enforced by nothing. It runs on the developer's machine. The developer owns the machine. The developer can disable the hook by deleting the file, by chmod-ing it non-executable, by `--no-verify`, by `git -c core.hooksPath=/dev/null commit`, by setting `HUSKY=0`, by uninstalling husky, by committing from a different machine, by committing from the GitHub web UI, by committing from a CI bot. The hook has no enforcement mechanism that the developer does not control, because the hook runs on the developer's machine, and the developer's machine is the developer's. CI runs on a server the developer does not control. CI is the enforcement. The hook is a suggestion. A suggestion that takes 14 seconds is not a cheap suggestion. It is an expensive suggestion. A suggestion that is bypassable by the people it targets is not a suggestion. It is a door with no frame, standing in a field, that the wind occasionally knocks over.

## Why "Local Is Cheaper Than CI" Is The Lie That Holds The Whole Thing Up

The defense of the pre-commit crowd is: *"Running the linter locally is cheaper than running it in CI. A local run is 14 seconds. A CI run is 14 minutes. You're saving 13 minutes and 46 seconds per commit."*

Let me show you what "cheaper" means in toll-booth-land:

```
You wanted:   catch bugs early, cheaply, before CI
You got:      "catch bugs early, on the developer's machine, which the developer
              controls, and which the developer can and will bypass with --no-verify,
              so the bugs are not caught early, they are caught at CI, at 14 minutes,
              the same as before, plus the developer spent 14 seconds not catching them,
              plus someone spent 11 months writing the hook, plus the hook broke once
              and blocked every commit in the repo for an afternoon"

The part after "You got:" is the part they do not put in the slide deck.
```

"Local is cheaper than CI" is true in the same sense that "cooking at home is cheaper than a restaurant" is true: it is true if you do not value your own time, and if the meal is edible, and if you do not burn the kitchen down. A local linter run is cheaper than a CI run *if the local run is the one that finds the bug*. The local run is the one that finds the bug only if the developer does not bypass it. The developer bypasses it. The developer always bypasses it. The developer bypasses it because the hook is slow, or because the hook is wrong, or because the hook is checking files the developer did not touch, or because the developer is in a hurry, or because the developer is on call, or because the developer has a meeting in four minutes, or because the developer simply does not want to wait 14 seconds to fix a README typo. The bypass is one flag. The flag is 11 characters. The flag is shorter than the word "precommit." The developer will type the flag. The flag is faster than the hook. The hook has lost. The hook always loses. The hook has never, in the history of hooks, won against a developer who has learned the flag, and the developer learns the flag the third time the hook fails on a file the developer did not change, which is the second week.

"Local is cheaper than CI" is also a comparison between two things that are not interchangeable. A CI run does more than the hook. A CI run runs the tests. A CI run runs the build. A CI run runs the integration suite. A CI run runs in an environment that matches production. The hook runs the linter, on the developer's laptop, which has a different Node version, and a different `npm` cache, and a different `.env` file that the hook's `detect-secrets` is about to flag for the forty-seventh time. The hook is not "CI but local." The hook is "a linter but local," and a linter is not CI, and a linter is the thing CI runs *first*, before the things that matter, and you have taken the least important step of CI and moved it to the most expensive place to run it (the developer's attention) and called it "shift left." It is not shift left. It is shift *down*, into the developer's lap, and the developer is setting it on the floor.

## The Real-World Example That Proves Everything

A team I worked with — I'll call them "the payments team," because they were — decided to add a pre-commit hook to "catch bugs before CI and improve developer experience." Fourteen months later:

1. Their hook ran `prettier`, `eslint`, `tsc --noEmit`, `jest --findRelatedTests`, and a custom script that checked commit message format against a regex. The regex required `type(scope): message` and the `type` had to be one of `feat|fix|chore|docs|style|refactor|perf|test|build|ci`. The regex did not allow `hotfix`. The team's on-call workflow produced `hotfix` commits. Every hotfix failed the hook. Every hotfix was committed with `--no-verify`. The hook now enforced conventional commit format on every commit *except hotfixes*, which were the only commits where the format mattered for the changelog. The changelog was generated from conventional commits. The changelog did not contain hotfixes. The changelog was wrong. The team did not notice for nine months, because no one read the changelog, because the changelog was generated, and generated things are not read, they are *produced*.

2. Their `tsc --noEmit` step took 11 seconds and type-checked the entire monorepo on every commit, regardless of which file was staged. A developer fixing a typo in a README waited 11 seconds for TypeScript to conclude the README had no type errors. The developer learned `--no-verify` on day three. The developer taught it to the new hire on day five. The new hire taught it to the intern on day seven. By the end of the second sprint, the hook ran on approximately 30% of commits, and the 30% were the commits that were already clean, and the 70% that were dirty were committed with `--no-verify`, and CI caught the dirty ones, and the hook had saved zero CI runs, and had cost 11 seconds × 30% × commits-per-day × developers × 14 months, which is a number large enough that the team's velocity chart had a visible dip every time someone joined, because the new person paid the toll until they learned the bypass, and the dip was the toll, and the chart was never investigated, because velocity charts are not investigated, they are *displayed*.

3. They had a **hook that flagged the word "password"** in any staged file, to prevent committing secrets. The word "password" appeared in the test fixtures, in the migration scripts, in the documentation, in the README, in the `.env.example`, in 47 files across the repo. The hook's baseline (a list of "allowed" occurrences) had to be regenerated every time someone added a test that mentioned a password, which was every PR, because the tests were for an auth system, and auth systems mention passwords. Regenerating the baseline was a 3-step command no one remembered. The command was in the README. The README was the same README the hook had once rejected for containing the word "secret" in a comment. The developer who hit the hook would run `--no-verify`, commit, and the secret scanner in CI (a different tool, with a different baseline, that agreed on nothing) would either pass or fail independently. There were now two secret scanners, one local and one in CI, with two baselines, disagreeing on whether the word "password" in a test was a secret, and the local one was bypassable and the CI one was not, and the local one existed only to be bypassed, and the team was maintaining two scanners to do the job of one, and the one that worked was the one in CI, and the one that did not work was the one they had a whole page of documentation about, and the documentation was wrong about how to regenerate the baseline, and had been wrong since 2022.

4. They added `husky` to "manage" the hooks, because `.git/hooks/` is not committed (it is per-clone) and they wanted the hook to be automatic for new clones. `husky` installed the hook on `npm install`. `husky` also installed a `pre-push` hook, a `commit-msg` hook, and a `post-merge` hook, because the person who configured it enabled every template. The `post-merge` hook ran `npm install` after every `git pull`, which re-ran `husky`, which re-installed the hooks, which was fine, except it also re-ran on `git pull` during a rebase, and during a rebase the `commit-msg` hook fired on every replayed commit, and the conventional-commit regex rejected three of the replayed commits because they were `hotfix` commits, and the rebase aborted, and the developer was stuck mid-rebase with a hook that would not let them continue, and the only escape was `git rebase --no-verify`, which exists, which the developer found on StackOverflow after 40 minutes, which the developer now uses for all rebases, and the developer has now learned two `--no-verify` flags, and the developer's muscle memory is `--no-verify`, and the developer types it without thinking, and the hook has been fully internalized as "the thing I type `--no-verify` to skip," which is the hook's final form.

5. They could not **remove the hook**. It was referenced in the onboarding doc ("we use pre-commit hooks for quality"), in the engineering blog ("how we shifted left"), in the README ("run `npm install` to install hooks"), in the contributing guide ("commits must pass the pre-commit hook"), and in a interview question ("we use pre-commit hooks; how do you feel about that?"). The hook did not work. The hook had never worked. The hook was load-bearing as a *cultural artifact*, not a technical one. Removing it would contradict the engineering blog. So they kept it. They kept the hook that did not work, and they added `lint-staged` to make it faster (which did not make it work, only faster at not working), and they added a `prepare` script to make husky install on `npm install` (which made the not-working hook install *automatically*), and they had a hook, and a hook manager, and a staged-file config, and a baseline generator, and a README section, and an onboarding slide, and the commits were still dirty, and CI still caught what the hook did not, and the hook was a four-layer ceremony around a `--no-verify`.

6. They wrote a retro. The root cause was "developers are bypassing the hooks." The actual root cause was "we installed a gate that is bypassable by the people it targets, on the machines those people control, and are surprised they bypass it." The action item was "educate developers not to use `--no-verify`." The education did not work, because `--no-verify` is faster than the education, and the developer will always choose the faster thing, and the faster thing is the bypass, and the bypass is 11 characters, and the education is a 30-minute meeting, and the meeting could have been an email, and the email could have been a hook, and the hook could have been nothing, and nothing is what they should have built.

They had replaced a 14-minute CI run (which caught the bugs) with a **14-second hook (which caught nothing) plus a 14-minute CI run (which caught the bugs)**, in order to "shift left." They had not shifted left. They had *added a toll booth in front of the same highway, and the toll booth collected tolls from the people who were already driving the speed limit, and the speeders took the bypass, and the highway was the same highway, and the bugs arrived at the same time they always had, and the only new thing was the booth, and the booth was lit, and the booth was staffed, and the booth was staffed by a linter that was wrong about the Dockerfile.* This is called "developer experience."

This is called "shifting left."

## What Dilbert's Cast Would Say

> **Wally:** "I have a pre-commit hook. I have never let it run. I alias `git commit` to `git commit --no-verify` in my shell profile. The hook has never executed on my machine. My machine is a hook-free zone. I consider the hook a coworker I have successfully avoided for fourteen months. We have never met. I am told it is very thorough. I will take their word for it."

> **Dogbert:** "You built a gate. The gate is on the developer's property. The developer has the key to the gate. You are surprised the developer uses the key. You have installed a gate whose only function is to be opened by the person you built it to stop. This is not security. This is a cat flap you are calling a vault door. The cat is using it. The cat has always been using it. The cat is committing `hotfix:` messages through it right now."

> **Mordac, the Preventer of Information Services:** "I have mandated pre-commit hooks across all repositories. Compliance is measured by the presence of the hook file, not by the execution of the hook. The hook file is present in 100% of repositories. The hook executes in 30% of commits. I consider this a victory. The 70% who bypass are 'noted.' The noting has not changed their behavior. I am considering noting them harder."

> **The Pointy-Haired Boss:** "Can't we just let CI catch it? When I started we didn't have hooks, we had a guy named Gary, and Gary looked at the diff, and if the diff had tabs Gary yelled, and that was the hook, and Gary went home at 5, and the hook went home at 5, and the code was fine." (He is the only person in the building whose quality gate has working hours and a name.)

## The "But What About Catching Bugs Early?" Question, Answered Once And For All

The pre-commit zealots will say: *"But catching a bug at commit time is cheaper than catching it at CI time! The hook saves a CI run! The hook saves developer context-switching!"*

You do not save a CI run by running a hook the developer bypasses. You save a CI run by running CI. The CI run is the thing that catches the bug. The hook is the thing that runs before the CI run and catches nothing, because the developer bypassed it, because the hook was slow, or wrong, or checking files the developer did not touch. The "cheaper" comparison assumes the hook catches the bug and CI does not have to. The hook does not catch the bug. CI catches the bug. The hook runs, the developer bypasses, the bug goes to CI, CI catches it, the CI run was not saved. The hook was a prelude. The prelude was skipped. The symphony played without it. The symphony was fine.

Real bug-catching happens in **CI, on a server the developer does not control, on every PR, with the full test suite, in an environment that matches production.** The hook is a local linter. A local linter is bypassable. CI is not bypassable. The hook is a door with no frame. CI is a wall. You do not need both. You need the wall. The wall works. The door does not. You are keeping the door because it was cheap to install, and it was cheap to install because it does not work, and things that do not work are always cheap to install, and expensive to maintain, and you have been maintaining it for fourteen months, and the maintenance is the README, and the README is wrong, and the wrongness is load-bearing, and this is your life now.

[As XKCD 1597](https://xkcd.com/1597/) established and the pre-commit advocates have spent eight years not reading: the moment you ask the developer to wait, you have started a race between your hook and the developer's patience, and the developer's patience is a finite resource that depletes faster than your hook runs, and the developer will win the race by refusing to run it, and the hook will be there, and the hook will be bypassed, and the bugs will be in CI, and CI will catch them, and you will have a hook and a CI and a bypass and a bug, which is four things where you used to have two (a CI and a bug), and four is more than two, and more is not cheaper, and cheaper was the whole point.

## The Long-Term Architecture

Eventually your team looks like this:

```
Your .git/hooks/pre-commit       → runs prettier, eslint, tsc, ruff, shellcheck, secrets, todo-check
Your --no-verify adoption       → 70% of commits (the 70% the hook was written to catch)
Your husky config               → installs the hook automatically, also a pre-push, a commit-msg, a post-merge
Your lint-staged config         → runs the hook only on staged files (saved 13.6s, saved zero bugs)
Your detect-secrets baseline    → 47 files of "allowed" password mentions, regenerated by no one
Your commit-msg regex           → rejects hotfix, the only commit type that matters for the changelog
Your CI                         → runs the same linters, plus tests, plus build, plus integration
Your CI secret scanner          → a different tool, a different baseline, agrees on nothing
Your README                     → says "commits must pass the pre-commit hook" — 70% of them do not
Your engineering blog           → "How We Shifted Left" — the left was shifted into a --no-verify
Your onboarding doc             → "we use pre-commit hooks for quality" — the quality is in CI
Your new hires                  → taught --no-verify on day five by the person they replaced
Your hotfix commits             → always --no-verify, never in the changelog, the changelog is wrong
Your actual bug-catching        → CI, which was there before the hook, and is there after, and works
Your hook                       → a 50-line shell script that runs on 30% of commits and catches nothing
```

The team without a pre-commit hook has CI that runs the linter, the tests, the build, and the secret scanner on every PR, a README that says "your PR must pass CI," and a 30-second onboarding that says "push to a branch, open a PR, wait for CI." Their bugs are caught at CI. Their developers do not wait at a toll booth. Their `git blame` is intact, because nothing reformats files locally. Their commits are fast. Their hotfixes are in the changelog, because there is no commit-msg regex. Their secret scanning is one tool, in CI, with one baseline. They do not have a pre-commit hook. They do not need one. CI is the enforcement. The hook is the decoration. They are, however, *embarrassed* in the DevEx channel because they "don't shift left." This is the real cost of the pre-commit hook: social. The technical cost of not having one is zero. The social cost of not having one is "the staff engineer can't write a blog post about it." So we pay the technical cost of a four-layer hook regime to avoid the social cost of admitting CI is enough, because we are, after all, primates who want to have shifted something.

## Summary, But It's A Toll Booth

| Principle | Stance |
|---|---|
| Installing a pre-commit hook | Do it. It will run on 30% of commits. The 30% are the commits that were clean. The 70% that were dirty will use `--no-verify`. The hook is a toll booth that collects from the people who were already paying. |
| Adding `husky` to auto-install hooks | A confession that the hook was not being installed. Now it is installed. Now it is bypassed automatically instead of manually. Efficiency. |
| Adding `lint-staged` to run only on staged files | A confession that the hook was too slow. The hook is now fast. The hook is still bypassed. Fast and bypassed is not a quality gate. It is a speed gate with no cars. |
| "Shift left" | A doctrine that is correct about where to find bugs and silent about who pays to look. The developer pays. The developer opts out. The looking did not happen. The left was not shifted. |
| `--no-verify` | The bypass. 11 characters. Shorter than the word "precommit." Faster than the hook. The developer's friend. The hook's nemesis. Always wins. |
| The hook as "developer experience" | It is an experience. The experience is waiting. The waiting is the cost. The cost is paid by the people the hook cannot catch. This is the experience. |
| Your engineering blog about shifting left | Located on the company blog, 2,000 views, comments enabled, one comment says "have you tried --no-verify," the comment has 40 upvotes, the blog has not been updated. |

If your solution to "we want to catch bugs early" is "install a script on the developer's machine that the developer can disable with 11 characters, run it on every commit, watch 70% of developers disable it, keep the script because the blog post about it has 2,000 views, add a hook manager to install it automatically so it can be bypassed automatically, add a staged-file optimizer so the bypass is faster, add a baseline for the secret scanner that no one regenerates, and conclude that you have shifted left when you have in fact shifted the cost of looking onto the developer and the developer has shifted it back onto CI where it was going to be done anyway," you have not shifted left. You have *built a toll booth on a highway, staffed it with a linter, lit it from within, and watched every car with a fastpass roll through the open lane while you collected quarters from the cars that were already going the speed limit, and called the quarters "quality," and called the booth "culture," and called the open lane a bug, and the bug was not the lane, the bug was the booth.* The hook is a toll booth. The toll is collected from the innocent. The guilty have a fastpass. The fastpass is `--no-verify`. The fastpass has always existed. The fastpass will always exist. The booth stands. The booth is lit. The booth is staffed by a linter that is wrong about the Dockerfile. The Dockerfile is in production. The booth is not.

I use CI that runs the linter, the tests, the build, and the secret scanner on every PR, a README that says "your PR must pass CI," and no pre-commit hook. My bugs are caught at CI. My commits are instant. My `git blame` is intact. My hotfixes are in the changelog. My developers do not know what `--no-verify` is, because there is nothing to verify. I am, however, not invited to DevEx conferences. This is a cost I have accepted.

---

*The author has not waited at a pre-commit hook since 2019. He considers CI his actual quality gate and the pre-commit hook a toll booth whose operator retired and left the light on. The light is still on. The light will always be on. Nobody is in the booth.*
