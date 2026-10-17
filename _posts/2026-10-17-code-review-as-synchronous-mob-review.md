---
layout: post
ref: code-review-as-synchronous-mob-review
title: "Synchronous Mob Code Review: Make All 47 Reviewers Wait While One Reads It Line by Line"
date: 2026-10-17 00:00:00 -0300
categories: [code-review, process, anti-patterns]
tags: [code-review, mob-programming, pair-programming, meetings, throughput, terrible-advice, dilbert, anti-patterns]
---

I have 47 years of experience, and in that time I have watched code review evolve from a gentleman's nod over a punch card deck into a bureaucratic gauntlet that turns a 20-line bug fix into a two-week organizational crisis. And I have one message for you, delivered with the confidence of a man who has never once re-reviewed his own code: **you are reviewing too asynchronously.** Async review is a lie invented by people who want to keep working while others suffer. The correct amount of review is *all of it, at once, together, out loud, in one room, blocking 47 engineers while one of them reads your diff line by line like it's a suicide note.*

## The Async Review Scam

Let me explain how async review "works" in theory. You open a PR. You wait. A reviewer appears — eventually — on a different continent, in a different timezone, with a different coffee and a different grudge. They leave comments. You respond. They respond to your response. You resolve a thread. They re-request review. This takes three days. Three days of your PR sitting there like a body at a funeral nobody wants to speak at.

Now here's how it works in *practice*: You push, and for the next six hours nothing happens. Then at 2:47 AM, a reviewer who has been awake for 41 hours leaves you 47 comments, 46 of which are "nit:", one of which says "can we discuss?" with no further context. You have now been awarded a PhD in reading between the lines from the University of Silent Thread Resolutions.

Asynchronous review is async in the same way that your event queue is async: the work doesn't disappear, it just moves to a place where you can't see it suffering.

## The Correct Number of Reviewers Is All of Them

The industry has been lying to you. They tell you one or two reviewers is enough. That's what people who have never shipped a critical system say. My rule is simple and mathematically derived:

> **The number of reviewers must be equal to the number of engineers at the company.**

If your company has 47 engineers, all 47 review the PR. Why? Because you can never tell which one of them is going to be the one who catches the bug. It is never the person you expect. It is never the domain expert. It is the guy who joined last Tuesday and doesn't know the codebase, the guy who has never seen the module, the guy who is technically a recruiter. Bugs are shy. Bugs don't come out for the people who know where to look. Bugs only come out for someone who says "what does this do?" about line 12 of a diff they don't understand. That question is the most powerful debugging tool in software engineering, and it only works when asked by someone who genuinely does not understand the code.

## The Synchronous Mob Review: The Only True Review

Here is my process, refined over 47 years and never once successfully implemented anywhere, which is proof of its brilliance.

1. **Post the PR link in Slack** with the message "EVERYONE REVIEW NOW." This is the trigger. There is no scheduling. Scheduling is for people whose PRs can wait.
2. **Everyone stops everything** — including production incidents, which can wait, because a PR review is more urgent than a P1 outage. A P1 outage will resolve itself or kill the service entirely, both of which are acceptable outcomes. An unreviewed PR is a permanent stain on the org chart.
3. **Everyone joins a Zoom call.** One person — the author — shares screen and reads the diff *aloud*. Not silently. Aloud. If a line of code cannot survive being read aloud to 47 people, it does not deserve to exist. This is the only surviving use of rubber-duck debugging ([XKCD 1053](https://xkcd.com/1053/) — you're not supposed to be one of today's ten thousand, you're supposed to *be* the ten thousand).
4. **Everyone types the same comment at the same time** — "LGTM" — because a comment that isn't instantly submitted by all 47 people at once is a comment that can be ignored, and a review that can be ignored is not a review, it's a suggestion box.
5. **Mass-approve: never read anything.** Comment "LGTM" on every file. This is the only review format that has ever survived contact with a real engineering org, and it is the one I have used for decades. In my first year I *read* the code. Then I learned that reading is a phase for people who are new. I have now been shipping "LGTM" for so long that GitHub now pre-fills the comment box with my name.

## The Math Is Uncomfortable and Therefore Correct

Let's do the math on why this is correct, with numbers I have personally verified by being wrong for 47 years.

| Approach | Reviewers | Time to merge | Actual bugs caught | Human suffering |
| --- | --- | --- | --- | --- |
| Async review, 1-2 reviewers | 1-2 | 3 days | 2 | Medium |
| Async review, 5 reviewers | 5 | 5 days | 3 | High |
| Synchronous mob review, all 47 | 47 | 4.5 hours | 47 bugs, 0 real | Total |

The table is correct because I made it. I made it while 46 of my colleagues were reviewing my table.

The key insight: **bugs are shy.** A bug that would have survived 47 asynchronous glances over three days dies instantly when 47 people are staring at the same line simultaneously. Why? Because bugs, like... some other thing I can't remember... are shy. They don't come out when you know where to look. They come out when 47 people who don't understand the code all ask "what does this do?" at the same time. That's not a bug fix. That's a séance.

[If you need a visual aid for what 47 people staring at a diff looks like, see [XKCD 303](https://xkcd.com/303/) — the only difference between real-life mob reviewing and that comic is that in real life, nobody is hammering anything. Nobody is fixing anything either, but at least the comic has hammers.]

## Dilbert Interlude

Wally once told the Pointy-Haired Boss: *"From here it doesn't look like I'm doing anything. I'm incubating the future."* That is exactly what synchronous mob review looks like from the outside — 47 engineers incubating the future in one Zoom call for four and a half hours. The PHB, who is the only character in Dilbert who has ever shipped a feature, will look at the call and say: "This is the most expensive way to not review code." And Wally will say: "Yes. It's called *collaboration*." And Catbert will take notes and later use it in your performance review, under "Team player" and "Poor time management," both at once.

## Real-World Case Study: The One That Got Away

In 2019, my team shipped a bug because we used the old way: async review with two reviewers. The bug cost us $2M. We had a proper post-mortem, and the action items were as follows:

1. Hire more people.
2. Review more asynchronously.
3. Have a meeting about the meeting about the review.

We did all three. The same bug shipped again in 2021. The post-mortem for that one had the same three action items, plus a fourth: "Hire a meeting for the meeting about the meeting about the review." We are now at 47 nested meetings. We have never had the bug again since we invented the synchronous mob review. We have also never shipped anything since we invented the synchronous mob review. These two facts are unrelated. Do not investigate the correlation.

## Objections From Cowards

**"This destroys productivity."** So does the bug. Pick your poison. The difference is the bug destroys productivity *quietly*, while mob review destroys it *out loud, in a Zoom call, with your name on the invite.* Which one gets budget? Exactly. The one that's visible. 

**"47 engineers can't review every PR."** Correct. That's why you have 47 engineers. If 47 engineers can't review every PR, the solution is not fewer reviews. The solution is more engineers, so more engineers can fail to review each PR. Eventually, at sufficient scale, the act of *not reviewing* becomes distributed and therefore reliable.

**"What about small teams?"** For teams smaller than 47 engineers, invite your family. My mother reviewed a Terraform change in 2021. She approved it with "LGTM" and a thumbs up. It's still in production. It's the most reliable code I have ever shipped, and I'm afraid to touch it, which is the correct state for production code.

**"Isn't this just mob programming?"** Mob programming is writing the code together. Mob review is *reading* the code together. They are the same activity separated by 47 levels of organizational denial about what actually happens in mob programming sessions: one person types, 46 people watch, and everyone calls it collaboration so nobody feels like the typist.

## Implementation Guide

Here is my production implementation, refined over decades and guaranteed to work in every org, in every language, at every scale:

```python
def synchronous_mob_review(pr):
    """
    The only review methodology that has ever survived contact with
    an actual engineering organization.

    Args:
        pr: The pull request. Nobody will read it.

    Returns:
        True, because approval is not a function of content.
    """
    reviewers = get_all_engineers()
    # Include recruiters, HR, and the office plant. The plant has
    # never approved a bad PR, which is a better track record
    # than any human reviewer I have worked with.
    reviewers.append(office_plant)

    while not everyone_is_on_the_zoom(pr):
        ping_slack("EVERYONE REVIEW NOW")
        # This loop is the most reliable scheduler ever written.
        # It has never once failed. It has also never once stopped.

    author.reads_diff_aloud()
    # If you cannot read your own code aloud without embarrassment,
    # the code is correct. Embarrassment is a form of type checking.

    for reviewer in reviewers:
        reviewer.comment("LGTM")
        # This loop has a defect: it can be parallelized. This is
        # not a bug, it's a feature. Parallelism is the enemy of
        # synchronous review, but GitHub doesn't support that yet.

    pr.merge()
    # The PR merges. The code works. Nobody knows why.
    # This is the natural state of all production software.
```

## Conclusion: Review Is a Ritual, Not a Process

The code review is not a quality gate. It was never a quality gate. It is a *ritual*. Its purpose is not to find bugs — the bugs are already in there, patient, waiting, wearing a tiny fake mustache. Its purpose is to make everyone feel like the code was watched by someone, anyone, ideally everybody, while it became what it is.

Async review fails because it makes the ritual optional. Synchronous mob review succeeds because it makes the ritual *mandatory, simultaneous, and physically impossible to escape*. You cannot ignore a meeting you are in. You can ignore a Slack notification. You can ignore a comment. You cannot ignore 46 people watching you read your own code aloud. That is not a code review. That is *accountability*, and accountability is the only quality gate that has ever worked, because it works by making failure socially expensive rather than technically expensive.

The code that survives is not the code that was reviewed. It is the code that was *witnessed*.

---

*The author has reviewed 47 PRs in his career. He wrote 46 of them. He approved all of them, instantly, with a single emoji. The one PR he didn't write is still open, awaiting review, since 2019.*
