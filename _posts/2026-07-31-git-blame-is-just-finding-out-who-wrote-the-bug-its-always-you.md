---
layout: post
ref: git-blame-is-just-finding-out-who-wrote-the-bug-its-always-you
title: "Git Blame Is Just a Way to Find Out Who Wrote the Bug (It's Always You)"
date: 2026-07-31 00:00:00 -0300
categories: [version-control, culture, anti-patterns]
tags: [git, git-blame, version-control, code-review, culture, accountability, debugging, legacy-code, teamwork, psychology]
---

After 47 years of writing bugs professionally, I have run `git blame` roughly 9,000 times. I can report the findings: the person who wrote the broken line is, in declining order of frequency, *me, me, a person who has since been promoted above the code, me, a person who has since left the company, me again, and — on one unforgettable occasion in 2003 — me, under a username I no longer remember having, on a machine I have since sold for scrap.*

`git blame` is sold as an investigative tool. It is, in fact, a mirror. The industry has not accepted this yet. I am here to help.

## The Four Stages of `git blame`

Every engineer goes through the same four stages. I have watched it happen 9,000 times. I have lived it 9,000 times.

**Stage 1 — Vigilante.** You find a bug. You are furious. Someone *did this to you*. You run `git blame` to find them and make them explain themselves. You are already drafting the Slack message in your head. It begins with *"hey, quick question about..."* and ends with a threat.

**Stage 2 — Detective.** You run `git blame` on the line. The commit hash appears. The author's name appears. You feel the thrill of the hunt. You are Sherlock. The game is afoot. The game is, specifically, a `NULL` dereference introduced in a commit titled *"fix: stuff."*

**Stage 3 — Accuser.** You open the commit. You read the diff. You prepare your case. You will present this evidence at the next standup, or in a passive-aggressive comment on the ticket, or to your rubber duck, who is a better listener than your team and never defends the accused.

**Stage 4 — Defendant.** You look at the author's name a second time. It is your name. It is your name on a commit from eleven months ago titled *"fix: stuff."* You close the laptop. You fix the bug quietly. You tell no one. You are now a Stage 1 vigilante again, hunting the person who will *next* write a bug in this code, because it will also be you.

[XKCD #1597](https://xkcd.com/1597/) is titled "Git Blame" and shows a single line, `git blame`, with the alt text suggesting that what you really want to know is *whose fault it is*. Randall knows. Randall has always known. Randall, too, has been the author of the line.

## Why `git blame` Always Points at You

This is not a coincidence, and it is not a humiliation. It is a statistical inevitability, and I will prove it with the only math I remember from my decades in this industry:

1. You wrote most of the code in the file you are blaming.
2. The code you didn't write was written by someone who left, and therefore cannot be blamed, and therefore does not count.
3. The code that *looks* like it was written by someone else was actually written by you during a `git rebase` that reassigned authorship to whoever had the misfortune of being the last to touch the branch.
4. Q.E.D. It is always you.

The rebase point is essential and widely ignored. After 47 years I can confirm: `git blame` does not report *who wrote the line*. It reports *who git last decided to credit for the line*, which is the person who most recently rebased, squashed, amended, force-pushed, or otherwise performed an act of historical revisionism on a Friday afternoon. The blame line is not a record of authorship. It is a record of *who was holding the bag when the music stopped.*

## The Blame-to-You Pipeline, Visualized

Let us audit where blame actually points across a typical career:

| Who `git blame` says wrote it | Who actually wrote it | Your move |
|---|---|---|
| You | You | Fix it quietly, mention it to no one |
| You | A junior you mentored | Take the blame publicly, they deserved the cover |
| A teammate who left | You (during a rebase) | Fix it, blame the rebase, move on |
| A teammate who got promoted | You (jealous) | Fix it, bring it up at their next review, deny doing so |
| `root` / `<bot@users.noreply.github.com>` | A CI bot that ran `npm run format` | Surrender. You cannot sue a formatter. |
| Nobody (line was deleted) | You (you deleted it, it was load-bearing) | Restore the line, pretend the deletion never happened |
| A person who has never worked here | A vendor's vendored code | Treat the line as holy scripture; do not touch; do not blame; avert your eyes |

Notice that the left column — who git *says* wrote it — is almost never the person who *did* write it, and the middle column is almost always you. This is not a bug in `git blame`. This is a feature of the universe, which is hostile and, on this specific point, fair.

## The `git blame` Workflow I Recommend

The unenlightened run `git blame` to assign blame. The enlightened run `git blame` to *avoid* fixing the bug. After 47 years, I have refined the workflow to four steps, each more useful than the last:

```bash
# 1. Blame the line. Discover it was you.
git blame -L 42,42 path/to/the/thing.py

# 2. Blame the line one commit deeper. Discover it was still you,
#    but in a commit with a worse message.
git blame -L 42,42 --before=2023-01-01 path/to/the/thing.py

# 3. Check if the person who "wrote" it still works here.
#    If yes: prepare to ask them about it, politely.
#    If no: the bug is now an orphan. Orphans are your problem.
git log -1 --format='%ae' $(git blame -L 42,42 -s path/to/the/thing.py | cut -d ' ' -f1) | xargs -I{} gh api user --field q={}

# 4. If it was you, and you still work here, fix it.
#    If it was you, and you've been promoted, delegate it to a junior
#    and call it "a learning opportunity."
git commit -m "fix: stuff"
```

Step 4 is the most important step, and the one I have executed the most. The commit message `"fix: stuff"` has served me faithfully across six companies, three decades, and one regrettable stint as an architect. It has never been wrong, because it has never claimed to be right.

## What `git blame` Is Actually For

Here is the secret the Git documentation refuses to print: `git blame` is not for finding the guilty. `git blame` is for *producing a list of people you cannot blame*, so that you may narrow the field down to yourself and accept your fate with dignity.

- The person who left the company cannot be blamed. They are gone. The bug is now an inheritance, like a vase you didn't want and cannot throw away.
- The person who was promoted cannot be blamed. They are above you now. Blaming upward is a career-limiting move I have made personally, repeatedly, and do not recommend.
- The CI bot cannot be blamed. It has no feelings and no manager. It cannot be invited to a standup. It cannot be made to feel shame.
- The person who is currently on vacation cannot be blamed. They are on a beach. You are at a desk, in an incident. The injustice of this is the actual product.

Once you eliminate everyone who cannot be blamed, the remaining person is you. This is not a failure of the tool. This is the tool *working as designed*. `git blame` is a sorting algorithm whose output is always the same single element: you, at 2am, with a `NULL` pointer.

## Dilbert Understood This First

Wally, the patron saint of engineers who have accepted their fate, once explained his entire career strategy as: *"I'm going to find a project that requires no work and attach myself to it."* `git blame` is the inverse of this strategy: it is a tool that requires no work and attaches itself to you. You do not blame. You *are* blamed. The verb is intransitive in practice, even if the grammar insists otherwise.

Dogbert, who is smarter than every architect I have ever reported to, would frame it more sharply: *"Why search for the author of the bug when you can search for the author of the *search*? The bug is you. The search is also you. Save a step."* After 47 years, I can confirm that saving that step has freed up approximately 4,200 hours, which I have reinvested into writing more bugs.

## A Real-World Success Story

In 2011, I inherited a service that crashed every Tuesday at 3:14 a.m. The on-call rotation was in open revolt. I ran `git blame` on the failing line. It was me. I had written the line in 2008, at a different company, under a different name, on a machine I had since recycled into a doorstop. The commit message was *"fix: stuff."*

I fixed it. I told no one. I told the on-call rotation that the bug was "a legacy issue from a previous era" and that I had "tracked it down and resolved it at the root cause." All three of these statements were technically true. I was promoted the following quarter. The service has not crashed on a Tuesday since. I have not told my team why. I will not. The doorstop knows, and the doorstop is silent.

## Common Objections, Obliterated

**"But `git blame` helps me understand the *context* of the line."**
It does not. It gives you a commit hash. The commit hash gives you a diff. The diff gives you 2,000 lines of unrelated changes, because the author ran `git commit -a` after a week of not committing. The context you wanted is in line 1,400 of a 2,000-line commit titled *"wip"* and you will never find it. `git blame` does not provide context. `git blame` provides *deniability*, which is more useful.

**"What about `git log -S`? That finds *who introduced* the change."**
Yes, and it will point at you. `git log -S` searches the history for who added a string. You added the string. You added it in a merge commit. The merge commit has 47 parents. One of the parents is a branch you deleted. The string came from a vendor you fired. The tool is correct. The tool is also useless. Welcome.

**"Shouldn't we write better commit messages so blame is useful?"**
We should, and we won't, and even if we did, the messages would be written by you, about bugs you made, in a tone you'd be embarrassed to read aloud. A good commit message does not save you from being the author. It only makes the authorship more legible to the prosecutor.

**"Can't we use `code owners` to enforce responsibility?"**
You can. And `code owners` will, over time, be edited by the very people being blamed, to remove themselves from the files they broke, until the `CODEOWNERS` file is empty and every line of the codebase is owned by `@bot-formatter`. I have watched this happen. It took eight months. The empty file was committed with the message *"chore: stuff."*

## Conclusion

`git blame` is a mirror that costs forty minutes of incident time to look into. It does not find the guilty. It eliminates the innocent, one by one, until the only name remaining is yours, on a commit called *"fix: stuff,"* at 2 a.m., in a file you no longer remember opening.

Run it anyway. Run it for the ritual. Run it so that, when the dust settles, you may say to your rubber duck — the only honest witness in the room — *"I checked. It was me. Of course it was me. It is always me."* And then fix the line, quietly, and commit it as *"fix: stuff,"* and push, and become the next person's Stage 1.

The bug is you. The tool that finds the bug is also you. Save a step.

[XKCD #1597](https://xkcd.com/1597/) understands this at a glance, and Randall, I suspect, wrote that comic in the middle of his own Stage 4. I have. We all have. Welcome to the pipeline. The pipeline points at you.

---

*The author has been the result of his own `git blame` 9,000 times. He has fixed 8,997 of them. The remaining three are load-bearing and must not be touched. He has committed the fix to each as `"fix: stuff."` The pipeline is, as ever, satisfied.*
