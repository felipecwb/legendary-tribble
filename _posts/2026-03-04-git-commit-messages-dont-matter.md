---
layout: post
title: "Git Commit Messages Don't Matter (Here's Why)"
date: 2026-03-04 08:00:00 -0300
categories: [git, best-practices]
tags: [git, github, gitlab, bitbucket, version-control, devops, ci-cd]
---

Conventional commits? Semantic versioning? **Time theft.**

Here's my commit history from last week:

```
* fix
* fix2
* fix fix
* asdf
* WIP
* WIP2
* final
* final2
* final final
* ok now its final
* .
* please work
* why
* AAAAAAAA
* monday
```

This tells you everything you need to know: I was working.

## The Lie of "Meaningful" Commits

People say commit messages should explain "why" not "what."

But here's the thing: **I don't remember why.** I wrote that code 3 hours ago. That's ancient history. I've mass-produced 47 more bugs since then.

## My Commit Strategy

```bash
alias yolo='git add -A && git commit -m "changes" && git push -f origin main'
```

One command. No thinking. Pure efficiency.

## "But What About Git Blame?"

If you're using git blame, you're looking for someone to yell at. That's an HR problem, not a git problem.

## "But What About Reverting?"

Reverting? Just push forward. 

```bash
# Don't do this:
git revert abc123

# Do this:
git commit -m "undo the thing from before"
```

Now you have a paper trail of progress.

## The Perfect Commit Message

After years of refinement:

```
git commit -m "stuff"
```

- Short: ✓
- Descriptive: It's stuff. You can see what stuff in the diff.
- Searchable: `git log --grep="stuff"` finds everything

## Squashing is Lying

Some teams squash commits before merging. This is **historical revisionism**.

My 47 "WIP" commits tell a story. A story of struggle. Of triumph. Of mass-producing code at 3 AM while questioning career choices.

Squashing erases that narrative. Squashing is censorship.

## Conventional Commits Decoded

When people write:
- `feat:` — "I added something"
- `fix:` — "I broke something, then fixed it"
- `chore:` — "I mass-produced YAML"
- `refactor:` — "I moved code around and hope nothing broke"
- `docs:` — "I updated the README date"

Just write "changes." It covers all of these.

## Conclusion

Your commit history is not a novel. It's a crime scene. Embrace the chaos.

```
git log --oneline

a1b2c3d changes
b2c3d4e changes  
c3d4e5f changes
d4e5f6g changes
e5f6g7h changes
```

Beautiful.

[XKCD 1296](https://xkcd.com/1296/) nails it: "As a project drags on, my git commit messages get less and less informative." I just started at the end state. **Efficiency.**

Dilbert showed us the truth decades ago: Wally's entire career is committing "minor fixes" while doing nothing. I aspire to that energy.

---

*The author has mass-produced 12,847 commits. Twelve of them have meaningful messages. All twelve were accidents.*
