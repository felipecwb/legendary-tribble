---
layout: post
ref: conventional-commits-are-fortune-cookies-for-your-git-log
title: "Conventional Commits Are Fortune Cookies for Your Git Log"
date: 2026-08-11 00:00:00 -0300
categories: [anti-patterns, git]
tags: [conventional-commits, git, commit-messages, semver, changelog, automation, release-notes, bureaucracy, prefixes, git-history]
---

Forty-seven years writing commit messages and I've watched them devolve from English sentences into individually wrapped prophecies. We used to write what we changed. Now we wrap what we changed in a prefix and call it a spec. A whole generation of engineers will spend their careers typing `feat:` and feeling productive about it.

The Conventional Commits specification is, I'm told, "a lightweight convention for commit messages." Lightweight, like a piano is lightweight compared to a grand piano. It hands you a vocabulary of ten prefixes, a parenthesized scope, an optional exclamation mark for breaking changes, and a footer section with its own grammar. You read the spec and you realize you've joined a religion whose sacrament is a colon.

Let me be clear: **a commit message that needs a spec to be understood is a commit message that says nothing.** It is a fortune cookie. Small. Packaged. Vaguely authoritative. Empty inside.

## The anatomy of a fortune cookie

Here is the template, straight from the holy book:

```
<type>[optional scope]: <description>

[optional body]

[optional footer]
```

It is a fortune cookie. There is a hard shell — the `type:` — and inside the shell is a slip of paper that says something a human would never say out loud. "feat(auth): add thing." Nobody talks like that. Nobody writes like that in any other context. But the wrapper demands it, so the wrapper gets it.

The spec insists this format is for "humans and machines." I have news for the spec: machines don't read commit messages. Machines read hashes. Humans don't read them either, past the third one in a `git log`. The only entity that genuinely consumes a conventional commit is the changelog generator, and the changelog generator is the next thing I'm going to make fun of.

## All ten prefixes are the same prefix

Here is the dirty secret: there is one prefix and it is `chore`. Everything else is a costume the fortune cookie is wearing.

| Prefix | What you actually did | What you wrote |
|---|---|---|
| `feat:` | A change | `feat: a change` |
| `fix:` | A change you wish you hadn't shipped | `fix: a change` |
| `docs:` | A change nobody will read | `docs: a change` |
| `style:` | A change that touches nothing | `style: a change` |
| `refactor:` | A change that fixes nothing and breaks everything | `refactor: a change` |
| `perf:` | A change that made it slower | `perf: a change` |
| `test:` | A change you skipped | `test: a change` |
| `build:` | A change to the YAML | `build: a change` |
| `ci:` | A change to the other YAML | `ci: a change` |
| `chore:` | The truth | `chore: a change` |

Do you see it? Strip the prefix and the message is identical every time. The prefix is a mood ring — it tells the reader how you felt when you hit `:wq`, not what the code does. The Wallys of the world figured this out years ago: they stamp `chore:` on everything, because `chore:` is the prefix that asks no follow-up questions. It is the fortune cookie that comes empty. It is the only honest one.

([XKCD 1597](https://xkcd.com/1597/) understood git better than the git documentation does. Go look. The commit messages in that comic are more honest than any conventional commit I've ever reviewed.)

## A real commit history from a real team

Here is a sequence of conventional commits, written by the book, that I pulled from a repo last month:

```
feat(api)!: rewrite auth layer

BREAKING CHANGE: everything

fix(api): revert previous commit

chore: the previous commit was not a chore
```

Four commits. Two of them undo the first. One of them is a confession. The auto-generated changelog dutifully rendered this as:

> ### Features
> * rewrite auth layer
>
> ### Bug Fixes
> * revert previous commit
>
> ### BREAKING CHANGES
> * everything

That changelog went to a customer. The customer has questions. The customer will not get answers. The conventional commit spec was designed for this exact moment and it failed at it. Four cookies, one fortune: nothing.

## The exclamation mark is the only honest character

The one part of the spec I respect is the `!`. You append it to the type — `feat!:` — to announce a breaking change. It is the single character in the entire grammar that means something. A `!` says: I am about to ruin your morning.

My advice: append it to every commit. `chore!:`. `docs!:`. `test!:`. If you break things, announce it. If you don't break things, announce it anyway, because the users should live in suspense. A git log full of exclamation marks is a git log that tells the truth about software: something, somewhere, is always breaking.

> The Pointy-Haired Boss once asked: "Can we make the breaking changes quieter?" No. We cannot. We can only make them louder, and conventional commits let us put the loud part right in the subject line where it belongs.

## Auto-generated changelogs are confessions you didn't write

Here is the real reason conventional commits exist: someone, somewhere, wanted to stop writing release notes. Fair. I hate writing release notes too. But the solution they invented was a program that reads your commit messages and regurgitates them into a list.

Think about the logic. You wrote bad commit messages. A machine collects the bad commit messages. The machine sorts them under headings. You ship the headings to users. You have not improved the messages. You have alphabetized the lies. You cracked a thousand fortune cookies and stapled the slips into a press release.

If your changelog can be assembled by grepping your git log, your git log was never meant for humans. It was meant for the grep. Which means you wrote your commit messages for a program. Which means you, a human, are now a data-entry clerk for a changelog generator that produces output no human reads. Congratulations on the promotion.

([XKCD 386](https://xkcd.com/386/) is the correct response to anyone who argues commit-message formatting on the internet. They are not improving the codebase. They are answering the call.)

## When to use Conventional Commits (which is: when forced)

| Situation | What the spec says | What you should do |
|---|---|---|
| You ship a feature | `feat(scope): description` | `feat!: it works on my machine` |
| You ship a bug | `fix(scope): description` | `fix!: still broken, differently` |
| You touch the docs | `docs(scope): description` | `docs!: nobody reads these anyway` |
| You refactor | `refactor(scope): description` | `refactor!: same bug, new file` |
| Release time | Generate a changelog | Delete the repo and start over |
| Junior asks what `feat!` means | Explain the spec | Explain that the `!` means fear |

## The final verdict

A good commit message is a sentence that says what changed and why. I have written thousands of them. They take ten seconds. They age well. The grep finds them. The blame finds them. The future you, at 2 a.m., finds them.

A conventional commit message is a sentence that says which category the change filed itself under, in a grammar invented by people who wanted to skip release notes so badly they built a whole toolchain to skip them badly. It is not a communication protocol. It is a coping mechanism with a version number.

Asok, the intern, will implement the spec correctly. He will file every commit under the correct prefix. He will be proud. He will be wrong. The prefixes are a mood ring. The exclamation mark is a weather report. The changelog is a confession. None of it is engineering.

So the next time you type `feat:`, ask yourself: did I write a feature, or did I crack a fortune cookie and hand the reader the slip? If it's the second one — and it is always the second one — you're not communicating. You're packaging. Git deserves better. But git has never gotten better, so it'll survive.

Be honest. Write a sentence. Or at least stop pretending your prefix is one.

---

*The author has been prefixing commits with `chore:` since 1994. The auto-generated changelog still lists him under "Maintenance."*
