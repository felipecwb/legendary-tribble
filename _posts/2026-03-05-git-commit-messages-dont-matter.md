---
layout: post
ref: git-commit-messages-dont-matter
title: "Git Commit Messages Don't Matter: A Practical Guide to 'fix'"
date: 2026-03-05 10:00:00 -0300
categories: [git, best-practices]
tags: [git, commits, version-control, messages, history]
---

In my 47 years of version control (yes, I used SCCS), I've watched developers waste precious minutes crafting "meaningful" commit messages. Let me tell you the truth: commit messages don't matter. Here's why "fix" is the only word you need.

## The Commit Message Delusion

| Commit Style | Time Spent | Times Read | Value |
|--------------|------------|------------|-------|
| Conventional Commits | 5 minutes | 0 | None |
| Detailed messages | 3 minutes | Maybe 1 | Historical fiction |
| "fix" | 1 second | Never | Efficiency |
| Empty | 0 seconds | N/A | Peak performance |

The math speaks for itself.

## A Gallery of Excellence

Here's a real git log from my production codebase:

```bash
$ git log --oneline
a7b3c2d fix
9f8e7d6 fix
5c4b3a2 fix
1d2e3f4 wip
8a9b0c1 stuff
f1e2d3c fix again
b4c5d6e idk
7a8b9c0 .
e3f4a5b asdfjkl
c6d7e8f final fix
9b0a1c2 actually final fix
d4e5f6a final fix for real
8c9d0e1 ok now it works
```

Beautiful. Every commit tells a story. That story is: "something changed."

## The Conventional Commits Cult

Some teams use "Conventional Commits":

```bash
# Their way (tedious)
feat(auth): implement OAuth2 flow with PKCE for improved security

Adds OAuth2 authentication with PKCE extension to prevent 
authorization code interception attacks. Includes:
- New AuthService class
- Token refresh logic
- Session management

BREAKING CHANGE: removes legacy session cookies
Closes #1234

# My way (efficient)
fix
```

Both accomplish the same thing: the code is different now. Mine just saves 30 minutes of creative writing.

## The Blame Game

"But what about git blame?" they ask.

```bash
$ git blame src/auth.js
a7b3c2d (Me  2026-03-05) function login() {
9f8e7d6 (Me  2026-03-04)   return fix;
5c4b3a2 (Me  2026-03-03) }
```

See? Now you know WHO wrote it. You don't need to know WHY. You can ask them. If they left, too bad. The code speaks for itself (badly).

As [XKCD 1296](https://xkcd.com/1296/) shows with "Git Commit," even Randall knows that commit messages devolve into madness eventually.

## Time-Based Commits

I organize my commits by time of day:

```bash
# Morning commits
git commit -m "fresh start"
git commit -m "making progress"
git commit -m "almost there"

# Afternoon commits
git commit -m "why isn't this working"
git commit -m "maybe this"
git commit -m "try again"

# Evening commits
git commit -m "asdfghjkl"
git commit -m "..."
git commit -m "I hate computers"

# Night commits
git commit -m "fix"
git commit -m "FIX"
git commit -m "F I X"
git commit -m "actually works now"
```

## The Squash Illusion

"Just squash your commits before merging!" they say.

```bash
# 47 commits:
fix
fix
fix
wip
test
fix test
fix test again
f
ff
fff
save
SAVE
help
nevermind
done
not done
now done
actually done
...

# After squash:
"feat: implement user authentication"
```

All that history, carefully crafted, compressed into a lie. The squash commit message is fiction written after the battle.

## Enterprise Commit Messages

Here's what Dilbert's Wally actually commits:

```bash
# Wally's commits
"Updated the thing"
"Fixed the other thing"
"Changes per management request"
"See previous commit"
"Undoing previous commit"
"Redoing previous commit"
"Lunch"
"Back from lunch"
"EOD"
```

He's been employed for 25 years. Commit messages don't determine career success.

## The Perfect Commit Message

After decades of research, I've found the optimal commit message:

```bash
# For features
git commit -m "."

# For bug fixes
git commit -m "."

# For refactoring
git commit -m "."

# For documentation
git commit -m "."  # lol documentation

# Universal
git commit --allow-empty-message -m ""
```

## The Force Push Philosophy

When in doubt, rewrite history:

```bash
# Someone complains about your commit messages?
git rebase -i HEAD~100
# Change all to 'squash'
# Single message: "Initial commit"
git push --force

# Problem: Gone
# History: Clean
# Coworkers: Confused but unable to complain
```

## Semantic Versioning? More Like Semantic Committing

```bash
# Patch
git commit -m "fix"

# Minor
git commit -m "fix fix"

# Major
git commit -m "FIX"

# Breaking change
git commit -m "oops"
```

The Pointy-Haired Boss would be proud: maximum documentation with minimum effort.

## Real-World Success

Here's my contribution graph explanation:

```
Day 1: "fix"
Day 2: "fix"
Day 3: "fix"
...
Day 365: "fix"

GitHub: "This user made 365 contributions last year"
HR: "Impressive productivity"
Reality: 365 typo fixes
```

## Conclusion

Remember: git log is for archaeology, and archaeology is for dead projects. If your project is alive, nobody reads the history. If your project is dead, nobody reads it either.

Save your creative energy for Stack Overflow answers that will actually be read.

---

*The author's most descriptive commit message is "stuff and things." It deployed to production and made $2M in revenue. Nobody knows what it changed.*
