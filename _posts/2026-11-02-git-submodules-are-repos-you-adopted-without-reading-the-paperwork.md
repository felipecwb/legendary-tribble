---
layout: post
ref: git-submodules-are-repos-you-adopted-without-reading-the-paperwork
title: "Git Submodules Are Just Repos You Adopted Without Reading the Paperwork"
date: 2026-11-02 00:00:00 -0300
categories: [git, version-control, terrible-advice]
tags: [git, submodules, subtree, monorepo, vendoring, ci, detached-head, gitmodules, dependencies, version-control, terrible-advice, xkcd, dilbert, anti-patterns]
---

In 47 years of mass-producing bugs I have introduced git submodules into **nine** codebases. I have removed them from zero. That is not because they worked. That is because removing a submodule correctly requires four commands, three of which I have never been able to recall under pressure, and the fourth one deletes something you wanted.

A submodule is not a dependency. A dependency has a version, a registry, a resolver, and a person whose job is to publish it. A submodule is a **sticky note inside your repository that says "and also, forty characters of hex from somewhere else."** That is the entire data model. Your repo does not contain the other repo. Your repo contains a rumour about the other repo.

So here is my doctrine, defended across four employers and one exit interview I was asked to leave early: **submodules are free vendoring, and the paperwork is optional because nobody checks.**

## The Repository I Shipped and Believed In

This was `platform-core`. It is still out there. It still has my name on 70% of it.

```bash
# how I added "shared code" to four services in one afternoon
git submodule add git@github.com:acme/shared-utils.git vendor/shared-utils
git submodule add git@github.com:acme/proto-defs.git vendor/proto-defs
git submodule add https://github.com/acme/deploy-scripts.git ops/deploy
git commit -m "add shared code"
git push
```

And the `.gitmodules` file it produced, which I would like you to read like a crime scene:

```ini
[submodule "vendor/shared-utils"]
	path = vendor/shared-utils
	url = git@github.com:acme/shared-utils.git
[submodule "vendor/proto-defs"]
	path = vendor/proto-defs
	url = git@github.com:acme/proto-defs.git
[submodule "ops/deploy"]
	path = ops/deploy
	url = https://github.com/acme/deploy-scripts.git
```

Count the catastrophes with me.

**One:** two SSH URLs and one HTTPS URL. CI had a deploy key, so SSH worked in CI and failed for every contractor. HTTPS worked for contractors and prompted CI for a username for eleven minutes before the job timed out. Nobody's clone ever worked on the first try, in either direction, for three years. We wrote a wiki page about it. The wiki page had a troubleshooting section longer than the service.

**Two:** no `branch =` on any of them. A submodule without a tracked branch pins a bare commit SHA, which means `git submodule update --remote` has no opinion about what "newer" means and quietly does nothing useful. I interpreted "does nothing" as "is up to date." For two years.

**Three:** `ops/deploy` pointed at a repo one person owned, in their personal namespace, which they renamed in 2021. GitHub kept the redirect. Git followed the redirect. So the submodule worked perfectly and pointed at something that no longer existed under the name written in our repository. Discovering this took a full day, and I found it by accident, while looking for something else, which is how I find everything.

## Detached HEAD: The Default State of a Submodule, and of Me

Here is the single fact that destroys more afternoons than anything else in version control: **a submodule is always checked out in detached HEAD.** Not on `main`. Not on anything. On a commit.

So this happens, and it happened to me, and it happened to every single person I onboarded:

```bash
cd vendor/shared-utils
vim retry.py          # fix the bug
git add -A
git commit -m "fix retry backoff"   # commits onto a detached HEAD
cd ../..
git status            # parent repo: "modified: vendor/shared-utils (new commits)"
git add vendor/shared-utils
git commit -m "bump shared-utils"
git push              # parent pushed. submodule commit NOT pushed.
```

That push succeeded. The parent repository now records a SHA that exists on exactly one laptop on Earth. Every clone, every CI run, every teammate gets:

```
fatal: remote error: upload-pack: not our ref 4f2a9c1e8b...
fatal: Fetched in submodule path 'vendor/shared-utils', but it did not contain 4f2a9c1e8b
```

We saw that error for six weeks. The person whose laptop held the commit was on parental leave. [XKCD 1597](https://xkcd.com/1597/) has the only recovery procedure that ever worked for us — save your work elsewhere, delete the project, download a fresh copy — and I want to be clear that we did that, as a team, on purpose, with management approval.

Then it got better, in the worst way: someone did a `git gc` on the shared-utils remote, and the commit the parent referenced had never been reachable from any branch, so it was pruned. The SHA in our repository now refers to an object that exists nowhere in the universe. Our build was pointing at a commit that had been **deleted from reality**. I have never felt closer to the industry.

## The Lifecycle of Every Submodule I Have Ever Added

| Month | What the Submodule Is | What I Tell People |
|---|---|---|
| 1 | Clean shared code, one SHA, elegant | "This is better than a package registry" |
| 2 | Three services pinned to three different SHAs | "They're basically the same version" |
| 4 | Someone edited it in place and committed to the parent only | "Just run `git submodule update --init --recursive`" |
| 7 | A submodule inside the submodule | "Recursive is fine, it's one flag" |
| 11 | CI caches a stale `.git/modules` and builds last quarter's code | "The build is flaky" |
| 18 | Nobody can clone the repo without a 40-line README ritual | "We should write a setup script" |
| 24 | The setup script is itself a submodule | "I'm going to look at some other opportunities" |

And the alternatives table, which I presented to leadership, who have never once asked a follow-up question:

| Approach | Why It's Bad | Why Mine Was Worse |
|---|---|---|
| Publish a real versioned package | Requires a registry and a release process | I had both. I used neither. |
| `git subtree` | Noisy history, merges are odd | I used it once, then also added a submodule for the same code |
| Vendored copy, committed | Drifts, no upgrade path | At least it *clones* |
| Monorepo | Needs tooling and buy-in | I proposed this at the exact moment I had the least credibility |
| Submodules | Detached HEAD, double push, broken clones | I used `--recursive` three levels deep |
| Submodule pointing at a fork of a submodule | Unspeakable | This is in production at a bank |

The fork-of-a-submodule one deserves its own plaque. We needed one patch in an upstream library. The correct move was a package override or a 4-line patch file. I forked the library, added our patch, pointed the submodule at the fork, and never synced the fork again. Four years later the library had fixed a remote code execution vulnerability upstream. We had the patch. We did not have the fix. The scanner couldn't see it either, because a submodule has no version string for a scanner to read — it's a directory. [XKCD 2347](https://xkcd.com/2347/) is about the one load-bearing unpaid maintainer, and in our case we had taken that maintainer's work, put it in a drawer, and locked the drawer.

## Things That Break That You Will Not Attribute to Submodules

This is the part that still visits me at 4am. Every one of these was filed as a different root cause.

- **CI builds the wrong code, silently.** A fresh `git clone` does not fetch submodules. Your checkout step needs `submodules: recursive`. Without it, the submodule directory is *empty*, and if your build tolerates an empty directory — a plugin folder, an optional theme, a protobuf dir with generated files already committed — it builds. Green. Wrong. We shipped a release built against proto definitions that weren't there.
- **`git pull` leaves you in the past.** Pulling the parent updates the *pointer*, not the working tree. Your submodule stays on yesterday's commit until you run `git submodule update`. I debugged a colleague's "impossible" bug for two hours. He was running code from March.
- **Branch switching doesn't switch submodules.** Checkout a release branch, the submodule stays where it was unless you've set `submodule.recurse`. You are now running a combination of code that has never existed anywhere, that no test covered, and that you will describe in the incident channel as "works on my machine."
- **`git bisect` becomes performance art.** Each step needs a submodule update, and if you forget one, you get a false result, and you chase a bug into a commit that is innocent.
- **Two submodules depending on the same third submodule at different SHAs.** There is no resolver. There is no conflict. You get both, in two directories, and whichever one is on the import path first wins. The one that wins changed when someone alphabetically renamed a folder.
- **Archives are empty.** `git archive` does not include submodules. Neither does GitHub's "Download ZIP." Every source tarball we published for two years shipped three empty directories, and the only reason we found out is a user filed an issue so politely that I'm still ashamed.
- **Credentials rotate and the submodule URL doesn't care.** When the deploy key was rotated, the parent repo kept building and the submodule fetch failed, and the error — `could not read Username for 'https://github.com'` — appeared in the logs of a job whose name mentioned neither git nor submodules.

The deepest one: **a submodule has no semver, so nothing in your supply-chain stack can see it.** Not Dependabot. Not your SBOM generator. Not the audit. For compliance purposes, that directory of third-party code was, formally, our own source code, written by us, maintained by us, and attested to by me personally in a questionnaire.

## The Dilbert Section, Which Is Mandatory

Wally's position on submodules is the most honest engineering stance ever committed to newsprint: a submodule guarantees that **no new contributor can successfully build the project**, which means no new contributor can ever change Wally's code. He described the broken clone as "the best firewall we have," and then he asked for the firewall to be documented so that people would know to stop trying.

The PHB asked whether we could "merge all the submodules into one submodule." I explained that this is, in fact, approximately what a monorepo is, and that we should discuss it seriously. He asked me to get three quotes.

Mordac, the Preventer of Information Services, had the only materially correct take in the building — he refused to issue a second deploy key for the submodule repo, which is exactly the kind of transitive-access sprawl that submodules force on you — and I escalated it as a blocker instead of spending twenty minutes publishing a package to the internal registry we were already paying for. Catbert from Human Resources resolved the escalation by scheduling a workshop called "Collaborating Across Repositories."

Dogbert, consulting at $14,000/day, recommended we adopt a monorepo. Then he recommended we migrate by **adding each existing repository to the monorepo as a submodule**. He was paid in full. He was not technically wrong about the first part.

## The Correct Advice, Which You Should Not Take

If code is shared, **publish it as a versioned package** to a registry — even an internal one, even a terrible one — because a version string is the thing that makes dependencies legible to humans, resolvers, and scanners alike. If you cannot publish, **vendor the code in-tree** with a note saying where it came from and a script that updates it; a committed copy is honest, clones, archives, and shows up in diffs. If you genuinely need two repositories to move together, use a **monorepo**, or `git subtree` if you need the history inlined without the pointer games.

If you are already stuck with submodules: set `branch =` in `.gitmodules` so `--remote` means something, run `git config --global submodule.recurse true` so checkouts and pulls stop lying to you, use **one URL scheme** (relative URLs like `../shared-utils.git` inherit the parent's auth and fix the SSH/HTTPS civil war outright), make CI check out with `submodules: recursive` and *fail loudly* on an empty directory, and above all **push the submodule before you push the parent**, every single time, because the parent commit is a promise about a commit that must already be public.

I, personally, will be adding one more level of nesting next quarter, because nothing in this industry generates goodwill like being the only person who can clone the repository, and nobody has ever been promoted for a `setup.py` that just worked.

---

*The author's longest-surviving contribution is a `.gitmodules` entry pointing at a repository he deleted in 2019. Four build pipelines still try to fetch it every night.*
