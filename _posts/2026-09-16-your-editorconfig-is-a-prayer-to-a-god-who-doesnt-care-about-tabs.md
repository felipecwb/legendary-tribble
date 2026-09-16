---
layout: post
ref: your-editorconfig-is-a-prayer-to-a-god-who-doesnt-care-about-tabs
title: "Your .editorconfig Is A Prayer To A God Who Doesn't Care About Tabs"
date: 2026-09-16 00:00:00 -0300
categories: [tooling, formatting, culture]
tags: [editorconfig, formatting, tabs, spaces, indentation, tooling, bikeshedding, ide, prettier, eslint, code-style, configuration, whitespace, zealots, standards]
---

After 47 years of producing software — 38 of which predate the existence of the `.editorconfig` file, and 9 of which have been spent watching engineers commit a 17-line INI file to a repository as if it were a peace treaty, as if it ended the Tabs Versus Spaces war, as if the war were not already over and spaces won in 2007 and the tab people just refused to sign — I have arrived at a position the formatting zealots will not enjoy:

**A `.editorconfig` is a prayer. It is a 17-line INI file committed to the root of your repository in the hope that an editor — any editor, somewhere, eventually — will read it, and respect it, and indent the way you asked, instead of the way it wants. The editor will not read it. The editor has never read it. The editor has its own settings, and its own formatter, and its own opinions, and a `.editorconfig` is a suggestion the editor treats the way a cat treats a closed door: aware it exists, committed to ignoring it, and willing to digress at the first opportunity.**

That is the entire situation. There is a file in your repo called `.editorconfig`. It says `indent_style = space` and `indent_size = 4`. Half your team's editors honor it. The other half do not. The half that does not is the half that uses the editor you wish they didn't, and they will commit tabs into a spaces file, and the diff will be 2,400 lines, and 2,396 of them will be whitespace, and the PR will be unreviewable, and someone will add a `pre-commit` hook, and the hook will run `prettier`, and `prettier` will reformat the file, and `prettier` does not read `.editorconfig` either (it reads `.prettierrc`, which disagrees with `.editorconfig`, which disagrees with `.eslintrc`, which disagrees with the `tsconfig.json`'s `formatIndent` nobody knew existed), and now you have four sources of truth about indentation, all of them wrong, none of them agreeing, and the `.editorconfig` is the oldest and most ignored of the four.

The formatting committee is already drafting a memo to revoke my standing in the `code-style` channel. Let them. They have never had to bisect a bug where the only change in the offending commit was a tab that the editor inserted because the `.editorconfig` said `indent_style = space` but the editor's "Detect Indentation From File" heuristic saw three leading spaces from a misaligned comment and decided, autonomously, that the file was a tabs file now, and converted every line, and the commit looked like a rewrite, and `git blame` was destroyed, and the author of the tab was a summer intern who had already left, and the blame for the whitespace now pointed at the intern for code the intern did not write, because the intern's editor overwrote 2,396 lines of it on save.

## The Grand Illusion Of "Consistent Indentation"

Here is the pitch: *Add a `.editorconfig` to your repo. Every editor that supports it will use the right indentation, the right line endings, the right trailing-newline policy. No more tabs in a spaces file. No more CRLF in an LF file. No more "works on my machine" but the diff is all whitespace. Solved, with a 17-line INI file.*

Here is what actually happens:

```ini
# .editorconfig — the prayer

root = true

[*]
indent_style = space
indent_size = 4
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true

# and then, the prayer is answered like this:
```

```python
# what the file LOOKED like before the intern's editor got to it:
def charge(amount, currency):
    if amount <= 0:
        raise ValueError("amount must be positive")
    return stripe.charge(amount, currency)

# what the file LOOKED like after the intern's editor "honored" the .editorconfig:
def charge(amount, currency):
	if amount <= 0:                              # <- tab. one tab. one single tab.
		raise ValueError("amount must be positive")   # <- tab, then three spaces, because
	return stripe.charge(amount, currency)            #   "detect indentation" is ON by default
                                                     #   and the editor is a free spirit
# the .editorconfig said space. the editor said "i detected a tab earlier in the file."
# there was no tab earlier in the file. the editor is lying. the editor is always lying.
# the diff is 2,400 lines. the review is "lgtm." the bug ships on friday.
```

The "Detect Indentation From File" setting — present in VS Code, in JetBrains products, in Sublime, in every editor that claims to support `.editorconfig` — is the single feature that makes `.editorconfig` not work. The editor reads the `.editorconfig`. The editor agrees to `indent_style = space`. The editor then opens the file, counts the leading characters on the first 500 lines, and if it finds *even one line* with a leading tab — a tab from before the `.editorconfig` existed, a tab from a merge conflict, a tab from a paste, a tab from a snippet — the editor overrides the `.editorconfig` and uses tabs, silently, for the entire file, and does not tell you, and saves tabs into a spaces file, and the `.editorconfig` is, at that point, a decorative commit.

The "Detect Indentation From File" setting is the reason every `.editorconfig` in the world is followed by a `pre-commit` hook. The hook exists because the file does not work. The file does not work because the editors do not honor it. The editors do not honor it because they have a heuristic that thinks it knows better. The heuristic is wrong one time in four. One time in four is the time the intern's commit is open. This is the cycle. There is no exit except `prettier --write`, which does not read `.editorconfig`, and which you now run on every commit, and which has its own config, which disagrees, and you are back where you started, only now with three config files.

## The Comparison Table The Formatting Committee Will Not Print

| Concern | `.editorconfig` alone | `.editorconfig` + `prettier` | `.editorconfig` + `prettier` + `eslint` | The Truth |
|---|---|---|---|---|
| "Consistent indentation" | No (editors override it via "Detect Indentation") | Yes (prettier rewrites the file, `.editorconfig` is now decorative) | Yes (eslint rewrites the file, prettier is now decorative, `.editorconfig` is now a fossil) | The last tool in the chain wins. The `.editorconfig` is never the last tool in the chain. |
| Editors that honor it | ~70% (the other 30% have "Detect Indentation" on by default and override it) | N/A (prettier runs in the terminal, the editor's settings are irrelevant) | N/A (eslint runs in the terminal, prettier's settings are irrelevant) | "Editor support" is a lie told by a file that runs in the editor and is overridden by the editor. |
| Tabs in a spaces file | Still happens. The editor inserts them. The `.editorconfig` watches. | No (prettier converts them on commit) | No (eslint converts them on commit, prettier converts them on pre-commit) | You did not prevent the tab. You cleaned it up afterward, at a cost of three config files and a hook. |
| Config files required | 1 (`.editorconfig`) | 2 (`.editorconfig` + `.prettierrc`) | 4 (`.editorconfig` + `.prettierrc` + `.eslintrc` + `tsconfig.json`'s `formatIndent`) | Each file is a layer of defense against the previous file not working. |
| Config files that agree | 1 (with itself, mostly) | 2 (if you remember to keep `.prettierrc`'s `tabWidth` == `.editorconfig`'s `indent_size`) | 0 (they never all agree; one of them always says 2 spaces and the others say 4 and no one remembers which wins) | The number of disagreements grows quadratically with the number of config files. This is why formatting standards fail. |
| The diff that's all whitespace | Still happens (editors insert tabs on save before the hook runs) | No (prettier fixes it before commit) | No (eslint fixes it before prettier fixes it before commit) | You fixed the symptom. The cause — editors that do not honor `.editorconfig` — is unfixed and unfixable. |
| `git blame` readability | Destroyed by whitespace-only commits every time a new dev joins | Destroyed by the `prettier --write` commit that reformatted the whole repo | Destroyed by the `eslint --fix` commit that reformatted the whole repo *again* | Any formatting change is a `git blame` rewrite. `git blame` is now a record of who ran the formatter, not who wrote the code. |
| What happens when a new dev joins | Their editor inserts tabs. They commit. The diff is all whitespace. | Their editor inserts tabs. The pre-commit hook reformats. They commit whitespace the hook undid. They are confused. | Their editor inserts tabs. The pre-commit hook reformats. ESLint complains. They are confused *and* blocked. | Onboarding a new developer to a `.editorconfig` repo is teaching them which three hooks will rewrite their file and which one to trust (none of them). |

Read the "Config files required" row. This is the entire situation. A `.editorconfig` is a single file that should, in theory, be sufficient. In practice, it is insufficient, because editors do not honor it, so you add `prettier`, because `prettier` runs in the terminal and the editor cannot override it, but `prettier` has its own config that disagrees with `.editorconfig`, so you add `.prettierrc`, and now you have two files that must agree, and they do not, because no one updates both, so you add `eslint` to enforce, and `eslint` has its own config, and now you have three files that must agree, and they do not, so you add a `tsconfig.json` flag, and now you have four, and the four have never agreed, and the one that wins is the last one someone edited, and the `.editorconfig` — the original, the 17-line INI file, the prayer — is now a fossil, read by nothing, honored by nothing, a record of a hope you had in 2019 that an editor would respect you. The editor did not. You added three more files. You are not more respected. You are more configured.

## Why "Editor Support" Is The Lie That Holds The Whole Thing Up

The defense of the `.editorconfig` crowd is: *"It's supported by every major editor. VS Code, JetBrains, Sublime, Vim, Emacs — they all read it. You add the file, you're done."*

Let me show you what "supported" means in editor-land:

```
You wanted:   every editor indents the same way, no config needed
You got:      "every editor CAN read it, IF the user installed the plugin,
              AND the user did not enable 'Detect Indentation From File,'
              AND the user's workspace settings do not override 'editor.insertSpaces,'
              AND the user's 'editor.tabSize' is not set to 'auto,' which it is by default,
              AND no other extension has a 'formatOnSave' that runs first and disagrees"

The part after "IF" is the part that ruins you.
```

"Editor support" for `.editorconfig` means: *if every condition in the user's editor is set to the default, and no extension has overridden it, and the heuristic is off, and the file is in the root, and the user did not open the file from a subdirectory where a different `.editorconfig` applies, and the user's IDE did not cache the file's indentation from a previous session, then the editor will probably indent the way you asked.* This is a chain of six contingencies, every one of which is false for at least one member of your team, and the one for whom it is false is the one whose commit is open on Friday. "Supported" does not mean "honored." "Supported" means "the editor is aware the file exists and will consider it, along with seven other sources of truth, and may or may not do what it says." This is the same level of support a suggestion box provides.

The `.editorconfig` specification does not mandate behavior. It cannot — it is a file, not a law. It *suggests* behavior, and the editor is free to ignore the suggestion, and the editor does, and the spec calls this "editor discretion," and "editor discretion" is the phrase that means "the file does not work and the spec authors know it and have decided this is the editor's fault and not theirs." It is everyone's fault. The file does not work. The editors do not honor it. The spec does not require them to. The user has a heuristic. The heuristic is wrong. The commit is all tabs. Welcome to formatting.

## The Real-World Example That Proves Everything

A team I worked with — I'll call them "the platform team," because they were — decided to add a `.editorconfig` to their monorepo, to "end the tabs-versus-spaces debate once and for all." Eighteen months later:

1. Their `.editorconfig` said `indent_style = space, indent_size = 2` (because they were a JavaScript team and JavaScript people believe 2 spaces is a personality). Their `.prettierrc` said `tabWidth: 4` (because someone copied it from a blog post and never changed it). These two files disagreed about the fundamental unit of indentation in the repository, and both were committed, and both were read by different tools, and the prettier output and the editor output did not match, and every PR had a "fix whitespace" commit at the end, and the "fix whitespace" commit was 3,800 lines, and the review was "lgtm" because no one scrolls past 3,800 lines of whitespace to find the three lines of actual logic.
2. Their `eslint` config had a `indent` rule set to `error` with `tab` as the expected indentation, because the config was copied from a Python-to-TypeScript migration and no one updated it. So the `.editorconfig` said spaces, `prettier` said 4 spaces, and `eslint` said tabs. Three tools, three answers, one file. The file was reformatted three times per save: by the editor (to 2 spaces, per `.editorconfig`), then by the format-on-save (to 4 spaces, per `prettier`), then by the lint-on-save (to tabs, per `eslint`). The developer's cursor jumped around like it was being teleported by three gods who disagreed about geometry. The developer turned off format-on-save. The developer committed 2-space indents. The `eslint` rule fired in CI. The build failed. The developer turned format-on-save back on. The build passed. The developer's sanity did not.
3. They had a **whitespace-only merge conflict**. Two developers, both editing the same 400-line file, both with `.editorconfig` honored (their editors were the 70% that honored it), but one had `end_of_line = lf` and the other's operating system was Windows and their `git` had `core.autocrlf = true`, so the file was `lf` in the repo and `crlf` in the working tree, and both committed, and the merge conflict was 400 lines of `lf` versus `crlf` and zero lines of actual code, and the resolution was "accept theirs," and "theirs" was `crlf`, and the `.editorconfig` said `lf`, and no tool enforced it, because `.editorconfig` is not enforced by anything, it is a prayer, and the prayer was not answered, and the file was now `crlf` in a `lf` repo, and the next developer's editor "detected" `crlf` and switched to `crlf` for every file in the repo, and the `.editorconfig` watched, and the `.editorconfig` did nothing, because the `.editorconfig` is a file and files cannot act.
4. They added a **`pre-commit` hook** running `prettier --write` to "fix" the whitespace problem. The hook ran on every commit. The hook reformatted files that were already formatted, because `prettier` does not check whether the file matches, it reformats and reports the diff, and the diff was always non-empty because `prettier`'s idea of trailing whitespace differed from the editor's, and every commit now included a "format" commit, and `git blame` was destroyed across the entire repo in a single afternoon, and every line of every file now blamed the person who added the hook, and the original authors were erased, and the team lost three months of archaeology, and the intern — the same intern, always the intern — was blamed for a reformat that a senior engineer's hook had performed.
5. They could not **remove the `.editorconfig`**. It was referenced in the README, in the onboarding doc, in the contributing guide, in three blog posts the company had published about "our engineering culture," and in a slide deck from a conference talk the CTO had given titled "How We Ended The Tabs Versus Spaces War." The file did not work. The file had never worked. The file was load-bearing as a *cultural artifact*, not a technical one. Removing it would contradict the CTO's talk. So they kept it. They kept the file that did not work, and they added `prettier` to fix what it did not fix, and they added `eslint` to fix what `prettier` did not fix, and they added a `lint-staged` config to fix what `eslint` did not fix, and they had four config files and a hook and the tabs were still there, and the CTO's talk had 4,000 views on YouTube, and the comments asked "how did you end the war" and no one answered.
6. They wrote a retro. The root cause was "we needed a formatter." The actual root cause was "we added a file that does not enforce anything, then added three more files to enforce what the first file did not enforce, and the four files disagreed, and we spent 18 months reconciling four files about indentation, and the indentation is still not consistent, and we have a conference talk about it." They had four config files and one intern and the intern's commits were still tabs.

They had replaced a 30-second conversation ("we use 2 spaces") with a **four-file, three-tool, one-hook, conference-talk-generating, blame-destroying, intern-blaming, README-referenced configuration regime**, in order to "end" a war that ended in 2007. This is called "engineering culture."

This is called "developer experience."

## What Dilbert's Cast Would Say

> **Wally:** "I have a `.editorconfig`. I have never read it. My editor has never read it. We have an understanding. The understanding is that I will indent however I want and the pre-commit hook will fix it. I am the pre-commit hook's primary user. The pre-commit hook is my primary collaborator. We are a team of two. The `.editorconfig` is a third party we do not acknowledge."

> **Dogbert:** "A `.editorconfig` is a 17-line file you wrote to tell an editor what to do, and the editor ignored it, so you wrote a 40-line `.prettierrc` to tell a formatter to tell the editor what to do, and the formatter ignored the `.editorconfig`, so you wrote a 200-line `.eslintrc` to tell a linter to tell the formatter to tell the editor what to do. You have three layers of indirection to express the sentence 'two spaces.' The sentence 'two spaces' is six characters. Your configuration is 257 lines. This is the most expensive way anyone has ever said six characters."

> **Mordac, the Preventer of Information Services:** "I have mandated `.editorconfig` across all repositories. Editors honor it in 70% of cases. The 30% who do not honor it have been reprimanded. The reprimands do not change the editor's behavior. I am considering reprimanding the editors. I have a formatter certification. The certification does not mention that the formatter does not read the `.editorconfig`."

> **The Pointy-Haired Boss:** "Can't we just agree on spaces and move on? When I started we had one setting and it was 'indent' and no one had a file about it." (He is the only person in the building whose indentation policy fits in a sentence.)

## The "But What About New Editors?" Question, Answered Once And For All

The formatting zealots will say: *"But what about when a new dev joins? They need to know the indentation rules! The `.editorconfig` documents them! It's self-documenting configuration!"*

You do not need a file to document "we use 2 spaces." You need a sentence. The sentence is "we use 2 spaces." It is six words. It fits in a Slack message. It fits in a README. It fits in the onboarding doc, below the part where you explain how to run the tests, which is the part the new dev actually reads. The `.editorconfig` is not documentation. The `.editorconfig` is a config file that the new dev's editor will not honor, which will cause the new dev to commit tabs, which will trigger the pre-commit hook, which will reformat, which will confuse the new dev, who will ask "why did my commit change" and the answer is "the `.editorconfig` says spaces but your editor used tabs and the hook fixed it," and the new dev will say "so the file didn't work" and the answer is "no, the file is documentation" and the new dev will say "the documentation was wrong" and the answer is "the documentation was correct, the editor was wrong" and the new dev will quit, not over this, but this is the moment they started looking.

Real configuration is enforced by a **formatter that runs in CI and fails the build**, not a file that runs in the editor and is overridden by the editor. The formatter does not care about your `.editorconfig`. The formatter has its own config. The formatter's config is the source of truth. The `.editorconfig` is a fossil from before you had a formatter. You have a formatter now. The `.editorconfig` is redundant. Delete it. The formatter does not read it. Nothing reads it. The README reads it. The README is not an enforcer. The README is a README. Put the sentence "we use 2 spaces" in the README, delete the `.editorconfig`, and let the formatter do the formatting. The formatter is the thing that works. The `.editorconfig` is the thing that does not. You are keeping the thing that does not work because you wrote a conference talk about it. This is called "sunk cost."

[As XKCD 1185](https://xkcd.com/1185/) established and the `.editorconfig` advocates have spent nine years not reading: the moment you have a file that specifies how files should be formatted, you have a file that does not format anything, and you will need a second file that does. The second file is `prettier`. The second file does not read the first file. The first file is now a monument to a hope you had. The hope is not honored. The monument stands. This is the cycle. There is no exit except a formatter, which you were trying to avoid because a `.editorconfig` is, apparently, *simpler*.

## The Long-Term Architecture

Eventually your team looks like this:

```
Your .editorconfig             → says "space, 2" — honored by 70% of editors, ignored by 30%
Your .prettierrc               → says "tabWidth: 4" — disagrees with .editorconfig, no one noticed
Your .eslintrc                 → says "indent: tab" — disagrees with both, copied from a migration
Your tsconfig.json formatIndent → says "2" — disagrees with eslint, agrees with .editorconfig, read by nothing
Your pre-commit hook           → runs prettier, then eslint, then reformats, then the diff is 3,800 lines
Your git blame                 → destroyed in a single afternoon by the "format the whole repo" commit
Your README                    → says "we use .editorconfig for consistent formatting" — it is a lie
Your conference talk            → "How We Ended The Tabs Versus Spaces War" — 4,000 views, comments disabled
Your new devs                  → confused, then resigned, then looking for other jobs
Your intern                    → still committing tabs, because their editor's "Detect Indentation" is on
Your "self-documenting config" → a 17-line file that documents a policy no tool enforces
Your actual indentation        → whatever the last tool in the chain decided, which is never .editorconfig
```

The team without a `.editorconfig` has a `.prettierrc` with `tabWidth: 2`, a `pre-commit` hook that runs `prettier --check` and fails the commit if the file is not formatted, a README that says "run `npm run format`," and a 30-second onboarding that says "we use 2 spaces, run the formatter." Their indentation is consistent. Their config is one file. Their `git blame` is intact, because the formatter runs in the editor via format-on-save *and* in CI, and there was never a "format the whole repo" commit, because the formatter was there from the start. They do not have a `.editorconfig`. They do not need one. The formatter is the source of truth. The `.editorconfig` is a second source of truth that disagrees with the first. They have one source of truth. One source of truth is the maximum number of sources of truth a repository can support. They are, however, *embarrassed* at conferences because they "don't have a `.editorconfig`." This is the real cost of the `.editorconfig`: social. The technical cost of not having one is zero. The social cost of not having one is "the CTO can't give a talk about it." So we pay the technical cost of a four-file configuration regime to avoid the social cost of admitting a formatter is enough, because we are, after all, primates with indentation preferences.

## Summary, But It's A Config File

| Principle | Stance |
|---|---|
| Committing a `.editorconfig` | Do it. It will be honored by 70% of editors. The 30% will commit tabs. You will add a hook. The hook is the thing that works. The file is the thing that does not. |
| Adding `.prettierrc` alongside `.editorconfig` | A confession that `.editorconfig` did not work. The `.prettierrc` is the actual config. The `.editorconfig` is now a fossil. |
| Adding `.eslintrc` alongside both | A confession that `.prettierrc` did not work either. You now have three files about indentation. None of them agree. |
| "Editor support" | A lie. "Support" means "the editor is aware the file exists." It does not mean "the editor honors the file." The editor has a heuristic. The heuristic is wrong. |
| "Detect Indentation From File" | The single feature that makes `.editorconfig` not work. It overrides the `.editorconfig` based on the file's existing indentation, which is wrong, because the file's existing indentation is the thing you were trying to fix. |
| The `.editorconfig` as documentation | It documents a policy no tool enforces. The README documents it better, in a sentence, and the formatter enforces it. The file is documentation of a hope. |
| Your conference talk about ending the war | Located on YouTube, 4,000 views, comments disabled, the war did not end, the war moved to the pre-commit hook. |

If your solution to "we want consistent indentation" is "commit a 17-line INI file that the editor may or may not honor, then add a formatter that does not read it, then add a linter that disagrees with the formatter, then add a hook to reconcile the three, then write a conference talk about how you ended the war," you have not ended the war. You have *moved the war from the editor to the configuration files, where it is fought with semicolons and `tabWidth` and `indent: tab`, and the casualties are `git blame` and the intern, and the war is not over, the war has never been over, the war is the war, and the `.editorconfig` is a flag planted in a hill no one holds.* The `.editorconfig` is a prayer. The prayer is not answered. The formatter is the answer. The formatter does not read the prayer. The formatter is the god. The god does not care about tabs.

I use a `.prettierrc` with `tabWidth: 2`, a `pre-commit` hook that runs `prettier --check`, a README that says "run `npm run format`," and no `.editorconfig`. My indentation is consistent. My config is one file. My `git blame` is intact. The formatter enforces. The editor obeys the formatter, or the CI fails, and that is the only enforcement that has ever worked. I am, however, not invited to formatting conferences. This is a cost I have accepted.

---

*The author has used two spaces since 1998. His `.editorconfig` has been ignored since 2014. He considers the formatter his actual collaborator and the `.editorconfig` a pen pal who never writes back.*
