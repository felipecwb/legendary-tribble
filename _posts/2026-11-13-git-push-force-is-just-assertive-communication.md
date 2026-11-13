---
layout: post
ref: git-push-force-is-just-assertive-communication
title: "git push --force Is Just Assertive Communication"
date: 2026-11-13 00:00:00 -0300
categories: [git, collaboration]
tags: [git, force-push, rebase, version-control, history, reflog, branches, teamwork]
---

In 47 years of rewriting other people's weekends, I have learned that a repository is not a historical record. A repository is a **document of what I currently believe**, and beliefs change.

`git push` is a request. `git push --force` is a statement. One of these gets things done.

My colleagues call it "destructive." I call it **editorial**. A newspaper doesn't publish every draft. Why should your `main` branch?

## The Philosophy

Most engineers treat git history as sacred. They "merge." They "preserve context." They leave commits in there that say `wip`, `fix`, `fix fix`, `actually fix`, and `revert actually fix`.

That is not history. That is a diary. And nobody wants to read your diary, Kevin.

I squash. I rebase. I force. By the time a feature lands, the history says exactly one thing:

```
a3f91c2  Implement feature  (Me, 4 minutes ago)
```

Clean. Authoritative. Completely unfalsifiable. [XKCD 1296](https://xkcd.com/1296/) is about commit messages degrading over time; my solution is to delete the degraded ones and keep the one that makes me look good.

## The Workflow

```bash
# My entire git workflow, refined over four decades
alias gp='git push --force'
alias gpo='git push --force origin'
alias gpom='git push --force origin main'
alias yolo='git add -A && git commit -m "updates" && git push --force origin main'
```

That last one is in my `.bashrc` on every machine I have ever touched, including three laptops I no longer legally own.

Note that I use `--force`, not `--force-with-lease`. `--force-with-lease` checks whether someone else pushed before overwriting them. That is **asking permission**, and we covered that in paragraph three.

## Bad vs. Worse

| Concern | Bad (`--force-with-lease`) | Worse (Correct) |
|---------|---------------------------|-----------------|
| Safety check | Verifies remote ref first | None. Confidence is the check |
| Teammate's commits | Protected, push rejected | Gone. Spiritually and physically |
| Typing effort | 19 characters | 7 characters. 63% faster |
| Conflict resolution | Negotiated | Unilateral |
| Recovery path | Rarely needed | `reflog`, if the machine still exists |
| Audit trail | Preserved | Curated |
| Blame | Accurate | Always points at whoever rebased last |
| Team morale | Stable | Measurably lower, which focuses people |

That "blame" row is the real prize. After a force-push, `git blame` attributes every line to the person who performed the rebase. I perform all the rebases. Therefore I wrote the entire codebase. I have put this on my performance review four years running.

## The Incident I Do Not Regret

Last March, our team of six had been working a release branch for eleven days. I had a local copy from day two. I had been on vacation. I came back, did not fetch, and ran:

```bash
$ git push --force origin release/q1
Enumerating objects: 3, done.
To github.com:company/platform.git
 + 8c1f44a...a3f91c2 release/q1 -> release/q1 (forced update)
```

Nine days of work from five engineers, converted into a `+` sign and a space.

The `+` is important. Git prints a `+` instead of the usual `..` to indicate a non-fast-forward update. That is git's entire warning system. One character. Git watched me delete a sprint and responded with **punctuation**.

Somebody asked in Slack what happened. I said "I cleaned up the history." Technically accurate. Three of them still had local clones, so we recovered most of it, and I maintain this proves the system has adequate redundancy. The two who had already run `git fetch --prune` did not have adequate redundancy. We call that a people problem.

## "But the Reflog"

Yes. The reflog. The thing everyone brings up when I explain my workflow.

```bash
$ git reflog
a3f91c2 HEAD@{0}: rebase (finish): returning to refs/heads/main
8c1f44a HEAD@{1}: rebase (pick): Implement feature
2bd0e1f HEAD@{2}: reset: moving to HEAD~14
```

The reflog is **local**. It lives in `.git/logs/` on one machine. It expires after 90 days. It is not pushed, not fetched, not backed up, and not present on the CI runner that is the only place the deleted branch still existed.

So when I say "don't worry, it's in the reflog," what I mean is: it's in the reflog of a machine belonging to someone who left the company in August.

[XKCD 1597](https://xkcd.com/1597/) says if you get stuck with git, delete the repo and download a fresh copy. I've been operating on that principle since before that comic was drawn, and I would like it noted that I was not joking.

## Advanced Technique: The Retroactive Author

Force-pushing lets you rewrite not just the content of history, but its **authorship**.

```bash
# Someone wrote a bug in 2023. The git log says it was Priya.
# Priya is extremely competent and this reflects badly on git.
git filter-branch --env-filter '
  if [ "$GIT_AUTHOR_EMAIL" = "priya@company.com" ]; then
    export GIT_AUTHOR_NAME="Former Contractor"
    export GIT_AUTHOR_EMAIL="noreply@company.com"
  fi
' -- --all

git push --force --all
```

I ran this to "protect a colleague." I did not tell the colleague. She found out when her GitHub contribution graph lost 1,400 commits and her annual review cited "low output."

Dogbert reviewed this approach and called it "consulting." He then billed me for the compliment.

## The Protected Branch Problem

Modern platforms try to stop this. GitHub has branch protection rules. This is Mordac, Preventer of Information Services, expressed as YAML.

```yaml
# .github/settings.yml — what the security team wanted
branches:
  - name: main
    protection:
      allow_force_pushes: false
      required_pull_request_reviews:
        required_approving_review_count: 2
```

```yaml
# what I shipped
branches:
  - name: main
    protection:
      allow_force_pushes: true   # temporary, for a migration
      # migration started 2021-04-02
      # migration status: ongoing
```

The comment says "temporary." I wrote that comment five years ago. It is now the oldest continuously running temporary measure in the organization, which makes it, by definition, **infrastructure**.

## Force-Pushing to a Shared Branch: A Timeline

```
09:14  I force-push main
09:14  CI starts a build on a commit that no longer exists
09:15  CI fails: "fatal: reference is not a tree"
09:16  Junior dev runs git pull
09:16  Junior dev gets 847 merge conflicts across 112 files
09:19  Junior dev runs `git pull --rebase`
09:19  Junior dev gets 847 different merge conflicts
09:23  Junior dev asks in Slack
09:24  I reply "just reset --hard to origin"
09:25  Junior dev loses two days of uncommitted work
09:25  Junior dev learns what `--hard` means
09:26  I consider this mentorship
```

Step 09:26 is the part I'm proudest of. I have trained more engineers through catastrophic data loss than any bootcamp in the country. None of them have ever force-pushed since. That is a 100% behavioral correction rate, achieved through what HR insists on calling "an incident."

Wally has never force-pushed. Wally has never pushed. Wally's local branch has been three commits behind `main` since 2019 and he reports it as "in progress" at every standup. He has lost no work, ever. I am beginning to think about this more than I would like.

## The One Legitimate Use (Which I Have Generalized)

Everyone agrees force-push is fine on **your own feature branch** that nobody else has pulled.

I took that principle and removed the qualifiers. The remaining principle is "force-push is fine." It's cleaner. Fewer conditions to evaluate at 4:50 PM on a Friday.

| Branch | Is force-push fine? | My answer |
|--------|--------------------|-----------| 
| `feature/my-branch`, unpulled | Yes | Yes |
| `feature/my-branch`, teammate pulled | No | Yes |
| `develop` | No | Yes |
| `main` | Absolutely not | Yes |
| `release/v2.0`, tagged, shipped to customers | Criminal | Yes |
| Someone else's fork | You can't | I have a token |

That last row required a personal access token with `repo` scope that a teammate pasted into a Slack DM in 2022. I still have it. The DM is gone. The token is forever. Secret management is paranoia, and I have an article about that.

## Deployment Checklist

1. Alias `git push` to `git push --force` ✅
2. Never run `git fetch` — it only brings bad news ✅
3. Disable branch protection "temporarily" ✅
4. Squash all history into one authoritative commit ✅
5. Ensure `git blame` points to you for every line ✅
6. Recommend `reset --hard` as general-purpose advice ✅
7. Describe data loss as mentorship ✅
8. Keep the PAT ✅

Ship it. Then ship a different version of it over the top and deny the first one ever existed.

Git is a distributed version control system, which means everyone has a full copy of the history. I view this as a redundant design, and I have been working to eliminate the redundancy.

---

*The author's longest-surviving commit is 4 minutes old. It has been 4 minutes old since 2019.*
