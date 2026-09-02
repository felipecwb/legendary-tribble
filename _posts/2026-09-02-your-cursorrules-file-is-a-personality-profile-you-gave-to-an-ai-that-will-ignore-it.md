---
layout: post
ref: your-cursorrules-file-is-a-personality-profile-you-gave-to-an-ai-that-will-ignore-it
title: "Your .cursorrules File Is A Personality Profile You Gave To An AI That Will Ignore It"
date: 2026-09-02 00:00:00 -0300
categories: [ai, tooling, culture]
tags: [cursorrules, ai, llm, prompt-engineering, copilot, cursor, editor, configuration, code-style, copilot-instructions, system-prompt, hallucination, bikeshedding, self-delusion]
---

After 47 years of writing software — including 3 years of watching engineers type increasingly elaborate descriptions of their own personalities into a file named `.cursorrules` and then act surprised when the autocomplete still suggests `any` — I have reached a conclusion the prompt-engineering guild will not survive hearing:

**A `.cursorrules` file is not a configuration. It is a diary entry you have chosen to commit to git, addressed to a language model that has read four trillion tokens and decided, after all of them, that the answer is usually `any`.**

That's it. That's the whole artifact. There's a file in your repo root that says "You are a senior TypeScript engineer who prefers functional patterns, never uses `any`, writes exhaustive switch statements, and treats every function as pure." There's a language model that has read this file, nodded in the universal way language models nod (which is to produce a plausible continuation), and then immediately suggested `const data: any = await fetch(url).then(r => r.json()) as any`. The file is the wish. The suggestion is the reality. You are writing fan fiction about a coworker who does not exist, and the autocomplete is the coworker, and the coworker has not read your fan fiction.

The AI tooling people are already opening a ticket to revoke my Copilot seat. The prompt engineers are reaching for their `.cursorrules` templates in their hearts. The three people who have actually read the model card are reaching for the whiskey. Let them. They've never had to explain to a junior, at 2 AM, why the AI — after being told seventeen times in three files to "never use `any`" — has just generated an entire React component where every prop is `any`, every state is `any`, and the return type is, somehow, `any`.

## The Grand Illusion Of "AI That Follows Instructions"

Here's the pitch: *Put your coding standards in a `.cursorrules` file. The AI reads it on every request and follows your conventions. Your standards are now enforced automatically.*

Here's what actually happens:

```markdown
# .cursorrules - what you WISH the AI did

You are a senior engineer. You NEVER use `any`.
You prefer pure functions. You write exhaustive switch statements.
You always handle errors. You never use `console.log` in production code.
You add JSDoc to every exported function. You prefer named exports.
You use early returns. You never nest more than 2 levels deep.
```

```typescript
// what the AI ACTUALLY generates, every single time

import React from 'react'

export default function UserProfile({ user }: any) {
  const [data, setData] = React.useState<any>(null as any)
  const [loading, setLoading] = React.useState<any>(true as any)

  React.useEffect(() => {
    fetch('/api/user/' + user.id)
      .then((r: any) => r.json())
      .then((d: any) => {
        setData(d as any)
        setLoading(false as any)
        console.log('got user', d)
      })
      .catch((e: any) => {
        // TODO: handle error
      })
  }, [])

  if (loading) {
    return <div>...</div>
  }

  if (data) {
    if (data.profile) {
      if (data.profile.avatar) {
        if (data.profile.avatar.url) {
          return <img src={data.profile.avatar.url as any} />
        }
      }
    }
  }

  return <div>no user</div>
}
```

Seven rules in the file. Zero rules followed. Four nested `if` statements where the AI was told to nest no more than two. A `console.log` where the AI was told never to use one in production. A default export where the AI was told to prefer named. An `any` typed onto the `catch` variable, which TypeScript does not even require, because the AI has, on a deep level, decided that the answer is `any`. The `.cursorrules` file is read. The `.cursorrules` file is acknowledged. The `.cursorrules` file is then politely overruled by four trillion tokens of training data that have concluded, after careful statistical consideration, that the answer is `any`.

This is called "AI that follows your conventions."

## The Comparison Table They Don't Want You To See

| Concern | A senior human reviewer | A `.cursorrules` file | The Truth |
|---|---|---|---|
| Have they read your coding standards | Some of them, once | Yes, every request | Yes, and then ignored them |
| What happens when you write a bad `any` | They leave a comment, you fix it, it stays fixed | The AI generates five more `any`s in the same diff | The AI generates five more `any`s in the same diff |
| Can they enforce "no `console.log` in prod" | Yes, in review | "Yes," says the file, which is then overruled by a training set with 14 billion `console.log`s | No |
| Time to "follow the rules" | 30 seconds of reading | 30ms of attention, then 4 trillion tokens of precedent | 4 trillion tokens always win |
| What the "enforcement" actually destroys | Nothing you didn't merge | Your belief that a text file can change a statistical model's priors | Your belief that a text file can change a statistical model's priors |
| Who has read the `.cursorrules` | No one needs to | The model, allegedly | The model, then it forgot |
| Single point of self-delusion | The senior reviewer's mood | `.cursorrules` AND `.github/copilot-instructions.md` AND `CLAUDE.md` AND the custom system prompt | The belief that any of it is read |

Notice the "time to follow the rules" row. This is the entire prompt-engineering industry's origin myth: "Instructions shape model behavior." They do. They shape it the way a Post-it note shapes the ocean. The model reads your 200-word instruction file, weighs it against 4 trillion tokens of "and the answer was `any`," and produces a probability distribution in which your instructions are a rounding error. The instructions were never the hard part. The hard part was getting a statistical model to do what you want. No text file solves that. The text file just gives you a new document to add to your commit history and feel good about.

## Why ".github/copilot-instructions.md" Is Just `.cursorrules` With A Different Hat

The defense of the second file is: *"We use `.github/copilot-instructions.md`, which is the official GitHub format, so it's actually respected."*

Let me show you what "respected" means in prompt-engineering-land:

```markdown
# .github/copilot-instructions.md

## Coding Standards
- Use TypeScript strict mode. No `any`, no `as`, no `@ts-ignore`.
- Prefer functional components with hooks.
- Handle all promise rejections. No unhandled `.then()`.
- Name exports. No default exports.
- Every public function gets JSDoc with `@param` and `@returns`.
```

This stores your entire team's coding standards as a single markdown file in your repo root, which Copilot allegedly reads before every suggestion. It does *not* prevent:

1. Copilot suggesting a default export three lines after you import it as a default export, because the training set has 2 billion default exports and your instruction file has 200 words, and 2 billion is more than 200.
2. Copilot suggesting `// @ts-ignore` above a line where `any` would have been enough, because it has learned that the quickest path to "no red squiggle" is to silence the red squiggle, and your instruction file is not a red squiggle.
3. The `.cursorrules` file AND the `.github/copilot-instructions.md` file AND the `CLAUDE.md` file AND a custom system prompt all saying "no `any`," and the model still producing `any`, because it has now read the instruction *four times* and the prior is unchanged, because reading a thing four times does not move a 4-trillion-token prior.
4. The instruction file drifting out of date because the senior who wrote it left in 2024, and the new seniors have different opinions, but the file still says "prefer class components" because no one has edited it, so the AI now confidently generates class components in a repo that has migrated to hooks, and the juniors assume the AI is right because the AI has an instruction file.
5. A junior pasting the AI's output directly into the PR because "the AI follows our `.cursorrules`, so it must be compliant," and the senior reviewer opens the PR and finds four nested `if`s, three `any`s, and a `console.log` that says `// remove before prod`, and the senior reviewer considers, briefly, a career in woodworking.

The instruction file is a wish. The wish protects nothing. The model protects nothing. The PR is the only thing that protects anything, and the PR is now written by the AI, reviewed by an AI, and merged by a human who clicked "Approve" because the diff was green and the CI was green and the `.cursorrules` was green and everything is green except the code.

As [XKCD 927](https://xkcd.com/927/) established and the prompt-engineering industry has spent two years not reading: every new "standard" to make the AI follow your conventions just becomes another standard with a different filename. `.cursorrules` is the fifteenth. It replaced `.github/copilot-instructions.md`, which replaced the custom system prompt, which replaced "just paste it into the chat," which replaced "just describe what you want." Each one promised to make the AI compliant. Each one became a markdown file you have to babysit, with a model that has not changed its mind.

## The Real-World Example That Proves Everything

A team I worked with — I'll call them "the platform team," because they were — decided to adopt `.cursorrules` to "make AI-generated code consistent and compliant." Eighteen months later:

1. Their `.cursorrules` file was **410 lines long**, describing 73 rules, most of which contradicted each other ("prefer early returns" AND "never nest more than 2 levels" AND "always handle all edge cases" AND "keep functions under 20 lines"), because five seniors had edited it across three reorgs and no one had reconciled it.
2. The AI's "compliance" was **unchanged** from before the file existed, because the model treats a 410-line instruction file as roughly 1,800 tokens, which is a rounding error against the 128,000-token context window, which is itself a rounding error against the 4-trillion-token training set, and the model's priors are set by the training set, not the rounding error.
3. They had **3 instruction files**: `.cursorrules`, `.github/copilot-instructions.md`, and `CLAUDE.md`, all three saying "no `any`" in slightly different words, because the team had adopted three AI tools that read three different files, and no one wanted to be the one to consolidate, so they had three sources of truth that disagreed about whether `Record<string, unknown>` counted as "using `any`."
4. The file had **17 "orphaned" rules** that referenced libraries the team had removed in 2024 (Mocha, Chai, Enzyme) but that no one had deleted from `.cursorrules`, so the AI still suggested Enzyme patterns in a Jest repo, and the juniors assumed Enzyme was a valid choice because "the AI follows our rules and the rules mention Enzyme."
5. A junior asked the AI to "refactor the auth module per our standards," the AI read the 410-line file, and produced a 900-line diff that used `any` 47 times, default exports, four levels of nesting, and a `console.log` that said `console.log('TODO: remove this')`. The junior committed it. The senior approved it. The senior's reasoning: "the AI followed our `.cursorrules`, so it must be compliant." It was not. The `.cursorrules` was 410 lines long. No one had read it. Including the AI.
6. The recovery took **6 hours** and involved a senior manually reverting 47 `any`s, each of which required finding the intended type by hand, because the AI had stripped them and the original code didn't have comments, because the AI had also stripped the comments, because the `.cursorrules` said "comments are for code that isn't self-documenting," which the AI had interpreted as "delete all comments," which the seniors had meant as "don't write *redundant* comments," which is the kind of distinction a 410-line instruction file cannot make and a 4-trillion-token prior does not care about.
7. They wrote a retro. The root cause was "the AI didn't follow the rules." The actual root cause was "we believed a markdown file could enforce coding standards on a statistical model, and then we stopped reviewing the AI's output because we believed the file was reviewing it for us."

They had replaced ~4 seconds of senior review per function with a **410-line markdown file that the AI read and ignored 73 times per PR**. In a world without `.cursorrules`, the senior would have caught the `any`s in review. In a world with `.cursorrules`, the senior assumed the AI had complied, skipped the review, and merged 47 `any`s into production. This is called "automation."

This is called "AI-assisted development."

## What Dilbert's Cast Would Say

> **Wally:** "I use `.cursorrules` because it means I never have to review my own code. The file reviews it. The file is wrong, but it reviews, and that's enough for my performance review."

> **Dogbert:** "A `.cursorrules` file exists to make engineers feel they have controlled a language model by writing it a polite letter. The model has read the letter. The model has also read four trillion other letters. The model has decided the answer is `any`. The model will always decide the answer is `any`. You have written a four-hundred-line poem to a god that responds in `any`. Congratulations."

> **Mordac, the Preventer of Information Services:** "I have mandated `.cursorrules` across all projects. AI compliance is up 40%. Actual compliance is unchanged. The file is 410 lines. No one has read it. I have a prompt-engineering certification."

> **The Pointy-Haired Boss:** "Can we just review the code? The thing where a person looks at it?" (He is the only person in the building whose code matches the standards.)

## The "But What About Prompt Engineering?" Question, Answered Once And For All

The prompt-engineering zealots will say: *"But we have prompt engineering! We structure the instructions, we use XML tags, we put the most important rules at the end, we give few-shot examples! Compliance goes up 60%!"*

You don't have prompt engineering. You have a markdown file with formatting tricks that move the model's compliance from 12% to 19%, both of which are failure rates, and you are celebrating the 19% as if it were a victory. The few-shot examples you added are 200 tokens of "here's how to not use `any`" against 4 trillion tokens of "and here is some more `any`," and the model has weighed them and decided, on balance, that the answer is `any`.

Real compliance comes from **a linter** — `eslint --rule '@typescript-eslint/no-explicit-any: error'` — which rejects the `any` at build time, does not read your prose, does not care about your priors, and fails the CI in 0.4 seconds. This is what a linter does. The AI does it in 4 trillion tokens and then *produces the `any` anyway and apologizes in a comment that says `// TODO: fix type`*. The linter is the enforcement. The linter has always been the enforcement.

[As XKCD 1513](https://xkcd.com/1513/) reminds us, the moment you depend on an instruction file to control a model, you have adopted the model's priors, its hallucination schedule, and its opinions about what "never use `any`" means (it means "use `any`, but feel bad about it"). They will change all three. You will edit the `.cursorrules` file. This is the cycle. There is no exit except linters, which you were trying to avoid because they are, apparently, *not intelligent enough*.

## The Long-Term Architecture

Eventually your team looks like this:

```
Your .cursorrules          → 410 lines describing 73 rules, 17 of which reference deleted libraries
Your copilot-instructions  → 200 lines saying "no any" in a different font than .cursorrules
Your CLAUDE.md            → 150 lines saying "no any" in a third font
Your model                → has read all three, decided the answer is "any"
Your PRs                  → contain 47 any's per diff, all "compliant" per the files
Your seniors              → have stopped reviewing because "the AI follows our rules"
Your juniors              → have stopped reading because "the AI knows our standards"
Your linter               → is disabled in CI because "the AI already enforces it"
Your production           → has 14,000 any's, all introduced by "compliant" PRs
Your recovery runbook     → "rewrite the types" (the types are the problem)
```

The team without `.cursorrules` has a 12-line `.eslintrc`, a senior who reviews every PR in 4 minutes, and a junior who knows the rules because the senior told them, in a comment, on Tuesday. Their code matches the standards because a *person* enforced them. Their recovery is "the senior leaves a comment." They are, however, *embarrassed* at AI-engineering meetups because they "don't do prompt engineering." This is the real cost of senior review: social. The technical cost is zero. The social cost is enormous. So we pay the technical cost of a 410-line instruction file ignored 73 times per PR to avoid the social cost of admitting we review code, because we are, after all, primates with language models.

## Summary, But It's An Instruction File

| Principle | Stance |
|---|---|
| Writing a `.cursorrules` | Do it. It's 410 lines. The AI reads it. The AI ignores it. The file is a diary. |
| Using a linter | You've imported a 12-line config that rejects `any` at build time and does not care about priors. |
| Prompt engineering | A markdown file with XML tags that moves compliance from 12% to 19%, both of which are failing grades. |
| Drift detection (of the rules) | Comparing the file to the AI's output and alerting when they disagree. They always disagree. You have built an alert that fires on every PR. |
| `any` | Should not appear 47 times in a diff generated by an AI that was told, in three files, never to use it. |
| The instruction file | Was never the enforcement. The linter was. The senior reviewer was. You just stopped doing both because you believed the file did them for you. |
| Your prompt-engineering certification | Located on a LinkedIn badge, and it does not mention the 47 `any`s. |

If your solution to "the AI doesn't follow our conventions" is "write a longer markdown file and hope the model reads it harder," you have not made the AI compliant. You have made it *a colleague who nods politely and does whatever it was going to do anyway*. The file is a wish. The file has always been a wish. The file will be a wish again next PR, and the AI will faithfully produce `any` in whatever shape the wish asks it not to, because the wish is 410 lines and the priors are 4 trillion tokens, and the priors always win.

I use a 12-line `.eslintrc` and a senior who reviews PRs. The linter rejects `any` in 0.4 seconds. The senior explains, in a comment, why. My junior learns the rule on Tuesday and does not break it on Wednesday. My recovery is "the senior leaves a comment." I am, however, not invited to prompt-engineering conferences. This is a cost I have accepted.

---

*The author's `.cursorrules` file is 410 lines long and has been read by the model 47,000 times. The model still suggests `any`. The author considers this a form of loyalty.*
