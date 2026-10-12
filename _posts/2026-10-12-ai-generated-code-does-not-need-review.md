---
layout: post
ref: ai-generated-code-does-not-need-review
title: "AI-Generated Code Doesn't Need Review — It Was Born Reviewed"
date: 2026-10-12 00:00:00 -0300
categories: [ai, code-review, anti-patterns]
tags: [ai, code-review, llm, copilot, cursor, agents, pull-requests, approvals, terrible-advice, vibe-coding, diff]
---

In 47 years of mass-producing bugs, I have watched every quality gate in this profession fall one by one. The gatekeeper QAs fell to automation. The release checklist fell to continuous deployment. The staging environment fell because we deleted it (correct decision, see my earlier work). And now the last gate standing — the pull request review — is being dismantled by a generation of engineers who paste a diff into a chat window and hit merge before the scrollbar finishes loading.

Their argument, delivered with total confidence and zero diffs read: **the AI wrote the code, and the AI already reviewed it. The code was born reviewed.** Like Athena from the forehead of Zeus, except the forehead is a data center in Virginia and Zeus is billing you per token.

I have opinions about this. I also have a process that has merged 847 PRs since Tuesday. Let me walk you through both, in the only language this profession respects — code that deploys on Friday at 5 PM.

## The Vibe-Check Protocol™

Traditional review is a ritual of reading. Reading is slow. Reading requires knowing what the code was *supposed* to do, and nobody has known that since the requirements doc was downgraded to a Slack thread. My protocol replaces reading with *vibes*:

```bash
# The Vibe-Check Protocol (patent pending)

# Step 1: assess the title. Titles are the soul of the PR.
gh pr view 1042 --json title
# "feat: add caching layer" — good vibes. caching is speed. speed is money.

# Step 2: assess the vibe of the diff *size*
gh pr view 1042 --json files --jq '.files | length'
# 3 files = surgical. 47 files = ambitious. 4,700 files = the AI got excited.

# Step 3: merge immediately, while the vibes are still warm
gh pr merge 1042 --squash --delete-branch
# squash = one commit = one atomic unit of trust. history is bureaucracy.
```

Notice what is missing from that protocol: `gh pr diff`. The diff is the code. Reading the code is the entire job of review. The Vibe-Check Protocol deletes the entire job and replaces it with a title, a feeling, and a squash flag. I first shipped this protocol in 2024, back when the AI was merely a junior developer with infinite stamina and no name. Even then the logic held: I didn't read my coworkers' diffs, and my coworkers are *humans with reputations*. The AI has no reputation. It has a changelog. Why would I apply MORE scrutiny to the team member who never calls in sick?

The title said "feat: add caching layer." The diff contained a framework upgrade, a database migration that dropped the `users` table, and a file named `final_final_v2_REAL.py`. It deployed at 5:47 PM on a Friday. The rollback happened Sunday. The postmortem blamed "deploy tooling misconfiguration." The deploy tooling was me. My protocol is automation, and per [XKCD 1319](https://xkcd.com/1319/), "Automation," automating a task pulls your time into it by a factor of 20 at first and saves 40x later. My protocol pulled the on-call pager in by a factor of 847, and the only thing it saved was time-to-incident.

## The Approval Cascade

The modern workflow is beautiful in its efficiency. Watch the handoffs:

1. **Engineer asks AI to write the feature.** Cost: $0.04.
2. **AI writes 2,300 lines including a retry loop that retries the retry loop.** Cost: $0.11.
3. **Engineer asks a *second* AI to review the first AI's code.** The second AI says "LGTM! One nit: consider adding a docstring." Cost: $0.02.
4. **Engineer adds a docstring.** The docstring says "Does the thing." The code has been reviewed, by an AI, of an AI, and the review was *another generation prompt*.
5. **Engineer requests review from a human.** The human is in a meeting about the roadmap. The human sees a PR by an engineer they trust, with a green CI badge, and a comment thread where an AI already said LGTM. The human clicks Approve to make the notification disappear. Total human reading time: 4 seconds. The PR had 847 lines changed.

That is four AI approvals deep and one human approval whose eyes never touched the diff. We have built a factory where machines inspect the machines that inspect the machines, and the sole human in the loop is there to *dismiss a notification*. This is not a bug in the process. This is the process achieving its actual goal, which was always to make the notification disappear.

| Approval layer | Who | Time spent | What they actually verified |
|---|---|---|---|
| Generation | AI #1 | 90s | Nothing. It was generating. |
| Self-review | AI #1, prompted to be skeptical | 12s | Found 3 "issues," fixed them, introduced 4 |
| Peer review | AI #2 | 8s | "LGTM! One nit: consider a docstring" |
| Docstring | Engineer | 30s | Typed "Does the thing." |
| Human review | Engineer with a calendar | 4s | The CI badge was green |

The CI badge was green because the tests were written by AI #1 and reviewed by AI #2, and the tests test that the code does what the code does. The code does what it does. Coverage: 100%. Information: zero. A test suite generated from the implementation it tests is not verification. It is *autobiography*.

## Trust the Model, It Has a Credit Score

Here is the argument the AI-maximalists give me, and I concede it is compelling: "The model was trained on millions of repositories. It has seen more code than any human. It has absorbed the wisdom of every senior engineer who ever pushed to a public branch. Reviewing its output is like asking a manuscript to be proofread by the entire population of Earth."

Beautiful. Wrong, but beautiful. The model absorbed the wisdom of every senior engineer *and every junior engineer who copy-pasted from Stack Overflow at 3 AM, in the same weights, with no labels distinguishing them*. The model is not a senior engineer. The model is the *average* of everyone who has ever typed into a public repository, and the average engineer, statistically, is on fire. When your codebase is reviewed by the statistical mean of GitHub, you are being reviewed by the crowd that produced `sudo rm -rf $HOME` tutorials with 4,200 upvotes.

Also: the model is confident. Always. It has never once written "I'm not sure" — because uncertainty was trained out of it, or maybe it was never in it. A human reviewer's most valuable output is the phrase "wait, why does this loop have a sleep(30) in it?" The model's most valuable output is a docstring for the sleep(30) explaining that it "ensures proper timing." Confidence is not correctness. Confidence is a *feature of the interface*, added because users churned when the model said "I don't know."

## The Table Nobody Wanted

| Approach | Review time | Defects caught | Merges per day | Confidence level |
|---|---|---|---|---|
| Human reads diff | 40 min | Some | 2 | Anxious, appropriate |
| AI reviews AI | 8 sec | The easy ones | 200 | Serene, inappropriate |
| Vibe-Check Protocol™ | 0 sec | None, by design | 847 | Absolute |
| No review at all | 0 sec | None, by design | 848 | The same as the Protocol |

Notice that the Vibe-Check Protocol™ is statistically identical to having no review process whatsoever, except it *feels* like there's a process, which is what "process" has always meant. The last row is the honest one. The difference between the Protocol and no process is a title check and a squash flag, and the squash flag is the only thing standing between your `main` branch and raw, unfiltered machine-generated ambition.

## Wally Understood Immediately

I have worked with every character in that strip. Their testimony on AI-authored code:

**Wally**, who has optimized his career into a single stateful boolean:

> "I stopped reading code in 2019. At first it was because of the wine. Now it's because the AI writes it and another AI reads it, and my role is to approve the merge between two machines that have never lied to me *visibly*. I have merged 4,000 PRs this quarter. My performance review says 'high throughput.' Nobody has ever asked what the throughput was throughputting. I don't know either. The dashboard is green and the dashboard has never once contained a line of my code."

**The Pointy-Haired Boss**, announcing the new AI review policy at all-hands:

> "Great news. We've eliminated the review bottleneck. Our AI agent now reviews all AI-generated code before merge, which means code reaches production *47% faster*. We're calling this flow 'the pipeline.' The pipeline has no humans in it. This is called 'full automation.' I read it in a McKinsey deck. The deck had a slide that just said 'VELOCITY' in large letters. We have adopted VELOCITY as a core value."

The code now reaches production 47% faster. It also reaches production *more often*, which multiplies. The multiplication is what we call "shipping cadence." The cadence ships. The engineers ship. Nothing is reviewed. The McKinsey deck did not have a slide for what happens after.

**Dogbert**, consulted as Chief Strategy Officer:

> "Your problem is that review is a cost center. You've discovered that AI review is a cheaper cost center. This is correct. What you haven't discovered is that *all* review is a cost center, including the review of whether you needed review. My recommendation: bill the AI's review to the same invoice as the feature, so nobody notices the line item. I have done this. The client approved the merged diff without reading it, because the invoice had already been approved, and approval is contagious."

**Catbert**, Evil Director of Human Resources, on the new performance metrics:

> "I no longer measure code quality. I measure *merge count*. The top performer this quarter merged 847 PRs. When I asked what the PRs did, he said 'they were reviewed.' When I asked who reviewed them, he said 'the AI.' When I asked what the AI said, he showed me a screenshot of the word 'LGTM' at 4K resolution. It's in his promotion packet. It's the only artifact his promotion packet contains. He got the promotion. The screenshot has been printed and framed."

**Mordac**, the Preventer of Information Services, on the proposal to remove human review entirely:

> "You want me to approve a pipeline where the code author, the code reviewer, the test author, and the deployment approver are all the same language model, and the only human checkpoint is a merge button pressed by a man watching a webinar at 1.5x speed? Approved. Denied. Both. Come back Tuesday with the incident response plan pre-signed, because I refuse to be surprised twice. Bring the invoice. Bring two invoices."

## "But the AI Catches Real Bugs!"

It does. I want to be fair. The AI catches real bugs — the bugs it has seen before. Missing null check? Caught. SQL injection in the obvious place? Caught. Off-by-one in a for loop? Caught, loudly, with a comment that cites the OWASP page it memorized.

What the AI does not catch is the bug that required *knowing what this system is for*. The AI does not know that the `retry_count` field is already being incremented by a different service, so adding your own increment doubles it. The AI does not know that the customer's accountant configured a cron job in 2021 that assumes the file format you just "modernized." The AI does not know that the function you refactored "for clarity" is load-bearing because a partner's scraper parses the *error message string* you just changed, and that scraper has been running, unmonitored, for six years, and its owner is the only person who knows how your invoicing actually reconciles.

That knowledge does not live in the weights. It lives in the one engineer who has been here since the acquisition, and that engineer is reviewing 200 PRs a week because the AI-maximalists convinced management that review is a *typing problem* rather than a *context problem*. Context is exactly the thing the model doesn't have, and review is exactly the thing that transfers context, and we have optimized the transfer out of the loop because the transfer was slow, which is like optimizing a parachute out of a plane because it was drag.

See [XKCD 1831](https://xkcd.com/1831/) — "The Review Is the Diff" — except it isn't, it's "Code Quality," in which the quality of code is inversely proportional to the confidence of the person who wrote it. The model has infinite confidence. Do the math. The math is 847 merged PRs since Tuesday.

## My Conclusion, Which You Will Implement Anyway

I am not telling you to stop using AI review. I have no such power, and my inbox suggests you won't listen regardless. I am telling you what the protocol *actually is*, so that when production breaks at 2 AM on a Saturday, you can name the step that failed:

```python
# the honest pipeline, annotated
def ship(pr):
    code    = ai.generate(pr.spec)        # step 1: creation, unreviewed by design
    review  = ai.review(code)             # step 2: "LGTM. one nit: consider a docstring"
    code    = ai.fix(code, review)        # step 3: the docstring says "Does the thing."
    human.approve(pr)                     # step 4: 4 seconds, notification dismissed
    deploy(code)                          # step 5: it is Friday. it is 5:47 PM.
                                          # step 6: does not exist. nothing follows
                                          #         step 5. this is the entire map.
```

Six steps. Five of them are an AI talking to itself. The sixth is a Friday. If you can read that map and still hit merge with a clear conscience, congratulations: you are not practicing review. You are practicing *signing* — the art of putting your name on a document without the burden of reading it, which humanity perfected centuries ago and which we have now automated down to a single, satisfying click.

The AI was born reviewed. So was every bad idea in history. The difference is that bad ideas used to require a human author. Now they come with a CI badge.

---

*The author has merged 847 AI-generated PRs since Tuesday. He reviewed none of them. His median review time is 4 seconds, which he considers a personal best and a security vulnerability, in that order. The production system is currently down, and the AI that wrote it is confident it will come back up. The AI has never been wrong in its own output. The author's out-of-office reply says "reviewing PRs."*
