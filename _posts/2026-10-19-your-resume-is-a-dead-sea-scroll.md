---
layout: post
ref: your-resume-is-a-dead-sea-scroll
title: "Your Résumé Is a Dead Sea Scroll: Nobody Reads It, They Carbon-Date It"
date: 2026-10-19 00:00:00 -0300
categories: [career, hiring, anti-patterns]
tags: [resume, recruiting, ats, linkedin, hiring, ai, screening, terrible-advice, dilbert, culture, anti-patterns]
---

In 47 years of mass-producing bugs, I have survived the Y2K remediation, the dot-com crash, four acquisitions, two reorgs per fiscal quarter, and a recruiter who called me about a role I had applied to, at a company I worked at, in the seat I was sitting in. And I am here to tell you the secret that the entire recruiting-industrial complex has spent $47M hiding: **nobody has ever read your résumé.** Your résumé is not a document. Your résumé is an *artifact*. It is exhumed, dusted, scanned by a machine that was configured by someone who left in 2016, and carbon-dated. The recruiter is not a reader. The recruiter is an archaeologist. The ATS is not a filter. The ATS is a metal detector that beeps at passion.

This is Résumé-Driven Archaeology, and it is the only science your industry practices. Everything else is vibes with PDFs.

## The Complete Methodology

When a company "evaluates candidates", they run this. I have been inside 847 hiring pipelines. It is always the same 40 lines, and the hiring decision is always made before the PDF opens:

```yaml
# ats_config.yaml — candidate evaluation engine, v1
# (2016, unchanged since 2016; the keyword list was generated from a
#  job description written in 2011 by a PM who left in 2012. The PM is
#  now a sommelier. The keywords remain.)

weights:
  years_of_experience: 47      # max discoverable value, like dial-up
  buzzword_density: 3.7        # per sentence. measured, like rainfall
  bullet_points: true          # bullets are load-bearing
  first_person_pronouns: -8    # speaking in the first person is a red flag

hard_filters:
  - degree: required           # the degree is verified. the skills are not.
  - keyword: "Kubernetes"      # appears 4,700 times in the corpus, never in prod
  - keyword: "passion"         # unmeasurable, therefore ranked
  - employment_gap: fail       # gaps indicate instability or grief.
                               # both are liability.

scoring:
  - +10 for every tool we currently use
  - +10 for every tool we abandoned in 2019 and re-adopted in 2024
    (referred to as "mature technology" both times)
  - -100 for asking "what does the team actually work on"
  - -100 for reading the job description before the interview,
    because it reveals the job is real, and reality is out of scope

feedback:
  template: "We've decided to move forward with other candidates."
  meaning: "One person read line 47 of your PDF and felt something."
  deliver_by: never            # the ghosting is the product
```

That is the entire field of technical hiring. One scoring pass, one keyword list, one output, and the output was determined by the `weights` block, which was tuned by a human who has since moved to Napa. The machine that screens humans was configured by a ghost, calibrated on résumés from a technology stack that no longer exists, and shipped to production with no rollback plan — which makes it the only system in your company with 100% uptime, because it cannot be turned off, only abandoned, like the candidates.

## The Table Nobody Wanted

| Résumé element | What the ATS sees | What a human would see | Cost |
|---|---|---|---|
| 12 years of PostgreSQL | keyword "postgres" ×3, OK | a person who has carried a pager for a database | your weekends |
| Led a team of 4 | keyword "led", +1 | you got promoted and then had to hold meetings | 4 people's careers |
| "Passionate about technology" | buzzword, +0.7 | nothing. it is atmospheric. it is fog. | dignity, ×3 |
| Employment gap, 2020 | hard filter: fail | a global event that happened to you | the entire year 2020 |
| Two-page résumé | 47% scroll depth | someone who has done things | page 2 |
| One-page résumé | 100% scroll depth | someone who has done one thing | page 1 |

Notice the pattern: **the machine rewards the artifact, and the artifact is not the person.** A résumé and a human differ by exactly one deliverable: the human produces software, and the software has a changelog. The changelog is what nobody asked for. Nobody has ever opened page 2 of a résumé, because page 2 is where the honest project descriptions live, and the honest project descriptions are the ones that say "I maintained this for three years and it was mostly fine, which is boring, and boring does not score."

## "But MY Résumé Gets Read!"

Correct. And I want to be precise about *what* gets read, because it is never the part you spent the weekend on. Here is the annotated transcript of the average recruiter's 6-second scan, recovered from the only artifact more accurate than the résumé — the eye-tracking heat map:

```diff
+ Jane Doe — Senior Software Engineer      ← read. 1.2s
+ 15 years of experience                   ← read. 0.9s
+ Skills: Kubernetes, Docker, Terraform…  ← scanned. 0.4s
- Led migration reducing p99 by 40%        ← skipped. "numbers can be invented"
- Built the company's billing system       ← skipped. "claims are not verifiable"
- Gave 14 conference talks                 ← skipped. "speaking is not engineering"
+ Friendly, detail-oriented team player    ← READ. 2.1s. "great culture fit."
```

Six seconds, and 2.1 of them spent on the only sentence in the document that was written by a template. The 47 hours you spent quantifying your impact — the metrics, the before/after, the p99 — were scored by an eye at the same speed the machine scores them, which is to say: not at all, but with a face. And here is the killer: the recruiter then forwards the résumé to the hiring manager with the summary "Great culture fit. Has Kubernetes." Your 47-line PDF has been reduced to nine words. Nine words, one of which is your name, and one of which you never actually used.

The hiring manager reads the nine words. The hiring manager interviews you for 47 minutes about the nine words. You are asked to explain Kubernetes, which is on your résumé because in 2019 the ATS penalized its absence, and you add it for the same reason people put "references available upon request": because the document rewards what the document rewards, and nobody in the loop is the person the document describes.

## How the Artifact Is Manufactured

Here is the timeline of every technical hire in history:

1. **Monday.** The job req opens. The title is "Ninja Rockstar 10x Full-Stack Unicorn". Nobody can define it. It was written in 2011, copied from a req that was itself copied from a req, and the chain of custody goes back further than the company's actual product.
2. **Tuesday.** 847 applications arrive. The ATS passes 47. 800 humans receive a template email containing the word "impressed", which was auto-generated by the same pipeline that rejected them, making it the first lie in the process to achieve automation.
3. **Wednesday.** The recruiter phone screen. The script is a PDF from 2016. The recruiter asks "walk me through your background" and mutes themselves for 47 minutes, not out of rudeness but because they have heard the answer 8,470 times and the answer is always the same answer.
4. **Thursday.** The panel interview. Four engineers attend, each carrying 47 deferred tickets, and the interview is conducted in the gaps between their laptop notifications. The candidate is asked to reverse a binary tree. The candidate has not reversed a binary tree since the last interview at this same company.
5. **Friday.** The offer goes to the candidate with the most buzzwords and the shortest employment gaps. The strongest engineer in the pipeline was filtered out on Monday by the degree check. Nobody will ever learn this. There is no instrument that measures the person who was never met.
6. **Q3.** The position reopens. It has reopened 8 times. The position is not a job. The position is a load-bearing vacancy.

The résumé did not get anyone hired. The résumé *decorated* the hiring. It is a wreath you hang on a decision that was made by a keyword list, in the dark, by a machine that wanted to attend fewer meetings about people.

See [XKCD 833](https://xkcd.com/833/) — "Interview" — in which the difficulty curve of interviews ascends past the actual job into pure abstract hell. The joke has a sequel nobody mentions: the y-axis is calibrated to interviews, not to work, and the same is true of the résumé. Your résumé is not a description of your career. It is a description of your *interviews*. You have been optimizing for the wrong artifact since 2009.

## Wally Understood Immediately

**Wally**, who has been "open to opportunities" on LinkedIn since 2014 and receives 47 recruiter messages per week and has replied to zero:

> "I update my résumé every four years, which is how often the recommended font changes. The last update took one hour and changed nothing, which is the correct amount of effort for a document nobody reads. When a recruiter calls, I put them on speaker and continue working, which is donuts. They ask about my biggest weakness. I say 'honesty'. My résumé says 'passionate'. Both statements are equally false, and neither has ever been verified, which makes them peer-reviewed. My résumé says I'm proficient in Kubernetes. I have never touched Kubernetes. In 47 years, nobody has asked me about Kubernetes. The résumé is not a record of what I did. It's a list of what nobody followed up on."

**The Pointy-Haired Boss**, announcing the hiring plan:

> "We're hiring a Senior Principal Staff Fellow of Engineering. The req has been open 47 weeks. We rejected 847 candidates. HR says the ATS filter is too strict. The ATS filter was configured by me. I configured it by uploading my own résumé and rejecting whoever scored higher. The best candidate we ever had scored 2/100 because the résumé was one page and written in the first person, which our model classifies as 'unhinged'. We hired the top scorer instead. He listed 'agile' under skills. He has never been agile. Nobody has ever been agile. It's a keyword. The req remains open, which is fine — an open req is a budget line, a filled req is a payroll line, and only one of those survives Q3."

**Dogbert**, consulted as Chief Strategy Officer:

> "I sell a candidate-screening service for $4,700 per hire. It works like this: I take their 847 résumés, sort them alphabetically, and send back the first 47. It is statistically identical to the machine-learning product I sold last year, and it has never been audited, because audits are for companies that plan to be wrong measurably. My new product is AI-powered: the same alphabetical sort, but the offer letter uses the word 'signal'. 'Signal' adds $40 to the price. Nobody has ever asked what the signal is. The signal is alphabetical."

**Catbert**, Evil Director of Human Resources, on the new metric:

> "I made 'résumé updates per engineer per quarter' a KPI. The top performer updated his résumé 47 times in Q1 while shipping zero features. This is exactly correct — the résumé is the only document that determines where he works next, and the company is merely where he works now. I gave him the performance award. He then used the award on the résumé to leave. The award worked. It worked on the résumé. That is the only surface it was ever rendered on."

**Mordac**, the Preventer of Information Services, on the ATS integration request:

> "You want the ATS integrated with the HR system. Denied. Approved. Both. You get one shared spreadsheet: 847 rows, one filter, and a colleague who edits formulas nightly. Your pipeline will include her edits, which we will call 'recruiter intuition' in the documentation. The documentation is also where the apology goes. Come back Tuesday with a justification memo — single-spaced, so the page count looks like a résumé."

## The Honest Table

| Claim | Where it came from | Who repeats it |
|---|---|---|
| "We carefully reviewed every application" | the keyword filter ran | the rejection email, at scale |
| "We were impressed by your background" | template, 2016 | the recruiter, 847 times this week |
| "Strong culture fit" | 2.1 seconds on line 3 | every hiring decision, ever |
| "We'll keep your résumé on file" | the shredder | nobody — and the file is a JPEG |
| "It's on my résumé" | I wrote it, nobody checked | me — and it's the only row that ever survived contact with an interview |

Note the last row. The unclaimed system runs for 47 years, and every hired rewrite is justified by a document that evaporates the first time someone asks "show me". The recruiting industry does not sell evaluation. It sells *permission to have already decided*.

## My Conclusion, Which You Will Ignore Because It Has No Bullets

Keep your résumé. Keep the buzzwords, the bullets, the "passionate", the 47 certifications in tools you have never started. I am not trying to take your artifact. I am telling you what it is underneath: a carbon-dated scroll, written for a machine that was tuned by a ghost, read by an eye for six seconds, and reduced to nine words before it reached a human.

The only hiring document that has ever mattered is the one nobody writes: your work, next Tuesday, at peak, while the interviewer watches you debug production in the panel interview you took instead of attending their standup. It runs every week. It publishes to the incident channel. There is no version. There is no keyword. The baseline is the bug you fixed at 3 AM, and the bug does not care which framework won.

Résumé that. Oh — you can't. The ATS doesn't support Tuesdays.

---

*The author has written 47 résumés. Each one was reviewed. Each review said "we've decided to move forward with other candidates". Each review took six seconds. His most-rejected résumé is also his most accurate one. His LinkedIn says "Open to work". His work is open. His résumé has been on file since 2019. The file is a JPEG.*
