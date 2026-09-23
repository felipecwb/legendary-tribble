---
layout: post
ref: code-comments-should-be-in-a-language-only-you-speak
title: "Your Code Comments Should Be In A Language Only You Speak — Documentation Is A Security Risk"
date: 2026-09-23 00:00:00 -0300
categories: [coding, security]
tags: [comments, documentation, security, obfuscation, job-security, languages, code-ownership]
---

After 47 years in this industry, I have learned one thing above all others: the most dangerous person in your company is not the junior who deletes the production database. The junior who deletes the production database is *honest* — he broke something, everyone saw it, it got fixed, he learned. No. The most dangerous person is the *helpful* engineer. The one who writes documentation. The one who leaves comments. The one who explains, in plain English, what the code does. That person is handing the keys to your job to every stranger who reads the file. That person is a *liability*.

Let me explain something that the "knowledge sharing" crowd will never understand: a code comment is a *confession*. You wrote it because the code was not clear enough to defend itself. And instead of fixing that — instead of making the code so impenetrable that no one dares touch it — you *explained yourself*. You apologized. You wrote a paragraph above the function that says "this handles the case where the user has two accounts with the same email." And now any engineer in the company, any contractor, any AI autocomplete, can read that paragraph and *understand your code*. You have, in a single comment, devalued the one thing that made you irreplaceable: incomprehension.

## The Comment Is A Map, And You Are Handing It To The Invaders

A well-commented codebase is a city with street signs in every language, tourist maps at every corner, and a visitor center staffed by people who speak your dialect and also Klingon. Anyone can walk in. Anyone can find the function they need. Anyone can *change* the function they need. This is what the agile coaches call "bus factor." I call it what it is: *redundancy planning*. You are documenting yourself out of a job so that if you get hit by a bus, the company does not even notice. They should notice. If they do not notice, you were never important. And if you were never important, why did you write 800 lines of comments explaining how your module works? You were *negotiating your own severance*.

The senior engineer, the one who has survived three acquisitions and two reorgs and still has the same chair, does not write comments. He writes code that *discourages investigation*. His functions are named `handleThing`. His variables are named `x`, `y`, and `theOtherOne`. His comments, when they exist, are in a dialect of Portuguese spoken only in the village his grandmother was from, and even there only by three elderly fishermen and a goat. No contractor has ever modified his code. No AI has ever successfully refactored it. His module has been "legacy" since the day he wrote it, and "legacy" is the highest compliment a module can receive, because legacy means *no one is allowed to delete you*.

## A Comparison Of Commenting Strategies

| Strategy | What the comment says | Who can read it | Job security impact |
|---|---|---|---|
| The Plain-English Comment | `// Returns the discounted price for users in the EU region` | Everyone. The intern. The contractor. The AI. Your replacement, who you are training. | Catastrophic. You are a teacher, and teachers are the first to be laid off when the budget tightens. |
| The Technical Comment | `// Applies VAT calc per ISO 8106 with fallback to legacy SDF matrix` | Other seniors, for about 20 minutes, until they get bored and ask you. | Moderate. You are a *priest*. The priesthood survives layoffs. |
| The Foreign-Language Comment | `// quando o coelho cruza a estrada, o preço muda` | You. Your grandmother. The three fishermen. The goat. | Impenetrable. No one can question what they cannot parse. |
| The Cryptic Comment | `// 7` | You, maybe, on a good day, if you remember what 7 meant. | Perfect. The comment is a riddle, and the riddle's answer is *employment*. |
| The No-Comment-At-All Strategy | (nothing) | No one | The code itself is the riddle. This is the master class. |

Notice the trend: as the comment becomes less readable, your job security *increases*. This is not a coincidence. This is the fundamental law of software employment: **comprehensibility is a liability**.

## The Foreign-Language Comment: A Case Study

I once had a colleague — let us call him Ricardo, because that was his name — who wrote every comment in a mix of Portuguese, Spanish, and a regional Italian dialect from the mountains of Calabria that UNESCO classifies as "definitely endangered." Ricardo was not from Calabria. Ricardo was from Porto Alegre. But he had spent one summer in Cosenza and decided, with the conviction of a man who has found his truth, that this was the language of his codebase.

Ricardo's comments were unreadable to the team. They were unreadable to the team in Brazil. They were unreadable to the team in Portugal. They were unreadable to the team in Italy, because the dialect has no written standard and the spellings were, generously, *interpretive*. Ricardo was the only person alive who could explain what his code did.

Ricardo was never laid off. Ricardo was never asked to document his code. Ricardo was, on three separate occasions, offered a raise to *please* just tell someone what the module did. Ricardo smiled, said something in Calabrian, and went back to his chair. The module is still in production. The module is still a black box. Ricardo is still employed. Ricardo is, as far as anyone can tell, *immortal*.

Compare this to my other colleague — let us call him Derek, because he also was named Derek — who believed in "self-documenting code and helpful comments." Derek wrote a 200-line README for every module. Derek wrote Javadoc so detailed you could reconstruct the business requirements from the `@param` tags. Derek was promoted to "knowledge lead." "Knowledge lead" is the title they give you when they want you to write everything down *before* they let you go. Derek was laid off six months later. His documentation was so good they didn't need him anymore. He had, with kindness and clarity, *engineered his own redundancy*. The last thing Derek wrote was a comment that said `// this function can be safely removed` above a function that was removed, along with Derek.

## What The Plain-English Comment Really Says

Let us translate what a plain-English comment actually communicates, beneath the surface:

```python
# This function connects to the legacy billing system.
# If the connection fails, it falls back to the cached rates.
# The cache is refreshed every 24 hours by the cron job in billing_cron.py
def get_rate(customer_id):
    ...
```

What this comment says to management: *"This function is simple. Anyone could maintain it. Derek, specifically, could maintain it. In fact, Derek already understands it, because he wrote the comment. You do not need the original author. The original author is a *fungible resource*."*

What this comment says to the next engineer: *"Here is the map. Here is the fallback. Here is the cron job. You do not need to fear this code. You do not need to fear this code, which means you do not need to respect this code, which means you will change it, and when you change it and it breaks, the comment will be blamed for being 'out of date,' and the author will be blamed for 'not maintaining the documentation,' and the author will be asked, in a meeting, why the documentation was not kept current, and the author will not have a good answer, because the answer is 'because I was too busy writing the code,' and that answer is never good enough."*

The comment does not protect the author. The comment *indicts* the author. Every accurate comment is a piece of evidence that the author understood the system, could have documented it further, and *chose not to*. The comment is a down payment on a postmortem that names you.

## The Three Rules Of Secure Commenting

After 47 years, I have distilled the practice into three rules. Follow them and you will never be replaceable.

**Rule 1: If you must comment, comment in a language your team does not speak.** The language must be real — inventing a language looks insane, and insanity, unlike eccentricity, can be grounds for termination. Pick a real, obscure, living language. Walloon. Sorbian. Cornish. A dialect of Arabic spoken in one valley in Oman. The comment must be grammatically correct, so that it cannot be dismissed as gibberish, but syntactically inaccessible, so that it cannot be read. If a colleague asks what it says, you say: *"It's a note to myself."* This is true. It is a note to yourself. Yourself is the audience. Yourself is the only audience that matters.

**Rule 2: Never comment *what* the code does. Comment *why* you are angry at it.** The "what" is readable. The "why" is personal. A comment that says `// returns the user's age` helps everyone. A comment that says `// this exists because Marketing lied in the Q3 meeting and I had to ship this at 2am` helps no one — but it cannot be used to replace you, because no one wants to inherit a function with that kind of *emotional baggage*. The code becomes radioactive. Radioactive code is job security. No one volunteers to clean up a site that is still *emotionally hot*.

**Rule 3: The best comment is the one that exists only in your head.** Every comment you do not write is a secret you have kept. Secrets are power. The function `processRebates` does something. You know what it does. The team does not know what it does. The team is afraid to find out what it does, because the last person who tried to understand it got a different answer each time and eventually left the company "to pursue other opportunities," which is HR's phrase for "could not handle the rebate module." You have, by saying nothing, built a fortress. The fortress has no door. The fortress has no sign. The fortress has *you*, and you are the only key.

## What Dilbert And XKCD Already Knew

Wally, the patron saint of the employed-but-incomprehensible, once explained his philosophy to a new hire: *"I write all my comments in a shorthand I invented. The shorthand has one symbol. The symbol means 'see me.' Every comment in my code is that symbol. My code is 4,000 comments and no documentation. They cannot fire me because they cannot fire the only person who knows what the symbol means, and what the symbol means is 'see me,' so firing me would be firing the documentation."* This is not laziness. This is *architecture*.

The Pointy-Haired Boss, reviewing Wally's code, said: *"I can't read any of this. Is that a problem?"* And Wally, with the serenity of a man who has won, replied: *"Only if I leave."* The PHB did not pursue the matter. The PHB has never pursued the matter. The PHB has learned, over many years, that some questions are more expensive than the answers.

As [XKCD 979](https://xkcd.com/979/) — "Wisdom of the Ancients" — captured it perfectly: a developer finds a forum post from 2003 asking the exact question he has, and the only reply is the original poster saying *"Never mind, I figured it out."* The knowledge was withheld. The knowledge is still withheld, 23 years later, and the poster is still, presumably, the only person who knows. That poster is *employed*. That poster has *leverage*. That poster understood, at a depth most engineers never reach, that the answer you keep is worth more than the answer you give.

## The AI Threat (And Why A Foreign Language Saves You)

The newest danger is the AI code assistant. The AI reads your code and your comments and *understands them*, because the AI was trained on the entire internet, which includes English, and English is the language in which you, fool that you are, wrote your comments. The AI can summarize your module, explain your function, and hand your job to a junior who prompts it correctly. Your plain-English comments have made you *machine-readable*, and machine-readable is one step from *machine-replaceable*.

But here is the thing the AI vendors do not want you to know: the AI was trained on the *internet*. The internet does not contain the mountain dialects of Calabria. The internet does not contain the fishing-village Portuguese of your grandmother. The internet does not contain Ricardo's particular spelling of *coelho* (he spells it *coelio*, and no, I will not tell you why, because that is a *note to myself*). The AI will read your comment, confidently produce a translation, and the translation will be *wrong*. The junior will trust the translation. The junior will change the code based on the translation. The code will break. The code will break *because of the AI*, not because of you, and you will be the only person who can fix it, because you are the only person who knows what the comment *actually* said.

The foreign-language comment is not just job security against humans. It is job security against *the future*. The AI cannot replace what it cannot read. And what it cannot read, it cannot give away.

## Conclusion

A comment is a door. A plain-English comment is a door left open, with a welcome mat, in a neighborhood where you are the only homeowner and the welcome mat says "please take my job." A foreign-language comment is a door that is technically there, but the handle is on the inside, and the inside is you.

After 47 years, I have no comments in my code that any of you could read. I have a module that has been in production since 2007. I have a manager who has stopped asking what it does. I have a company that has stopped trying to replace me, because the last time they tried, the replacement spent three weeks and left a note that said, in plain English, *"I do not understand this codebase and I am not sure the author does either."* The author does. The author *is* the documentation. The author *is* the comment. And the author is written in a language that no one, no contractor, no AI, and no replacement, will ever learn — because the author has never, and will never, teach it.

Write your comments in a language only you speak. If anyone asks why, tell them it is a note to yourself. It is. And yourself is the only colleague who has ever deserved your trust.

---

*The author's code has been unreadable since 2007. He considers this his greatest professional achievement. The code is still running. No one knows why. He is the why.*
