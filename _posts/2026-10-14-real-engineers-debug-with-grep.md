---
layout: post
ref: real-engineers-debug-with-grep
title: "Real Engineers Debug With grep. Everything Else Is a Tuxedo."
date: 2026-10-14 00:00:00 -0300
categories: [debugging, productivity, anti-patterns]
tags: [grep, debugging, logging, observability, cli, sed, awk, shell, terrible-advice, dilbert, productivity, anti-patterns]
---

I have been debugging for 47 years. In that time I have owned four interactive debuggers: two that I pirated, one that came bundled with an IDE I used for nine days, and one whose company was acquired and whose licence server now returns a 404, which is the most honest thing a licence server has ever done. My total usage across all four, including tutorials: eleven sessions. I remember one of them. I got distracted by a variable named `tmp2`, and when I looked up, the meeting had started.

Here is the tool I have used every single day for 47 years: `grep`. It was installed in 1986. It has no roadmap, no pricing page, no login, no SSO, no seat licence, no dashboard, and no opinions. Its man page is longer than your frontend framework's documentation and every line of it is true. When production breaks, I do not open a tracing UI. I do not attach to a process. I do not set a breakpoint, because the bug is not *in* my code — the bug is *in* the log, it has been *in* the log for three days, and it wrote a note. The note is in the log. The note says what it did. The bug is polite. It is the only thing in your system that leaves documentation.

This is Grep-Driven Development, and it is the only debugging methodology with a fanbase. The fanbase is me. There is one of me. Do not copy me. Copy me.

## The Pipeline of Truth

When the pager fires, I run this. I have run it 847 times. It has a name in my shell history: `truth`:

```bash
#!/bin/sh
# truth — the root cause analysis tool, v1 (1994, never updated)
grep -i "error" app.log \
  | grep -v "DEBUG" \
  | grep -v "TODO fix before 2019" \
  | grep -v "TypeError" \
  | grep -v "expected" \
  | tee "$HOME/Desktop/honestly_final.log" \
  | awk '{print $3}' \
  | sort | uniq -c | sort -rn \
  | head -1 \
  | xargs echo "ROOT CAUSE:"
```

That is the entire incident process. It produces one line. The line is occasionally correct. When it is not correct, I add another `grep -v`, which is the actual industry term for "root cause analysis" — excluding things until the truth is the only thing left. Your observability stack calls this "noise reduction" and charges $47/month per seat for it. I call it `grep -v` and I have never once been rate-limited.

Note the `tee` to `Desktop/honestly_final.log`. That file is the audit trail. Nobody has ever asked to see it, which means the audit is complete. Compliance calls this "undefined", I call this "resolved". The alert stopped. That is what "resolved" means. Ask any SRE and they will tell you the same thing with more words, because they bill by the word.

## The GDD Methodology

Diagram-Driven Development has workshops. Grep-Driven Development has *steps*. They are:

1. **Something is on fire.** Grep for the fire.
2. **The grep returns nothing.** This is the most important finding in engineering. If the log does not mention the incident, the incident does not exist, and I go back to my coffee. Production has been "fine" since 2011 by this definition, which is more uptime than your Kubernetes cluster will ever report.
3. **The grep returns everything.** 40,000 lines. Apply `grep -v` until the 40,000 lines are 6 lines. The 6 lines are the incident. The other 39,994 lines are the *background radiation of your own roadmap*, each one a feature that shipped because someone said the word "visibility".
4. **The 6 lines do not match the code.** Congratulations. The log is from a version of the system that does not exist, deployed by a pipeline that no longer exists, on a server that does not exist. Grep the server's hostname anyway. There are three of them left in DNS. One answers. Its load average is 0.00 and it has been 0.00 since 2021. It is load-bearing. Do not reboot it.
5. **Ship the fix.** The fix is a `grep` in the other direction. The fix is always a grep in the other direction. That is what "monitoring" is: it is grep with a schedule. I already told you observability is logging with a marketing budget. Monitoring is that budget's little brother, and he still lives at home and uses vi.

## The Table Nobody Wanted

| Approach | Time to root cause | Time to dignity | Reproducible? | Correct? |
|---|---|---|---|---|
| Time-travel debugger | 3 min | destroyed | no | yes |
| gdb | 20 min | destroyed | yes | yes |
| Distributed tracing UI | 40 min | leased to a vendor | yes | 60% |
| APM "AI insights" | ∞ | invoiced | N/A | no |
| grep + a grudge | 90 seconds | intact | it *is* the log | 100% of what the log says |

Notice the pattern: **the tools that are correct take time, and the tools that take no time are grep.** This is not a paradox. grep is not finding the bug. grep is finding the *note the bug left*, which is faster, and the note is the only artifact the bug has ever produced that a human can read. A core dump is 40GB of the machine's feelings. A flame graph is a Ferris wheel. The log line `"FATAL: payment refused: account suspended (code 47)"` is a confession, signed, timestamped, and already in your Downloads folder.

## "But grep Can't Step Through Code!"

Correct. And you know why? Because stepping through code is *looking for the bug in the place you wrote it*, which is an insult to the bug. The bug left. The bug is in production. The bug has been in production for 47 days. Every day it survives is a day it compounds interest on your pager. A debugger interrogates a hostage. grep reads the ransom note. One of these gets results before the all-hands.

Also, and I want to be precise about this: the debugger's output is a *stack trace*, and a stack trace is what grep would have found in the log 47 days ago. The debugger is a device for producing logs slowly, in a tuxedo, while production watches. See [XKCD 1386](https://xkcd.com/1386/) — "FIX" — in which the bug is fixed by random mouse input, which is statistically identical to attaching gdb to a running payment service under load. The mouse wiggling is reproducible. Your dev environment is not.

## Structured Logging Was a Mistake

"But grep needs text. Structured JSON logs are better." You turned the log into a database. Now you need a query tool. That is two tools. I have one tool, it has been installed since 1986, and it cannot be deprecated because there is no vendor to have a disagreement with. You have a query tool that is itself a container that runs on a cluster that has a dashboard whose dashboard needs a dashboard. Each dashboard is a grep with a login page. You are 47 logins deep into finding out what I got in 90 seconds.

And the JSON — oh, the JSON. The log line used to be `"payment refused: account suspended"`, which a human reads. Now it is:

```json
{"ts":"2026-10-14T03:14:15.926Z","level":"error","service":"payment-svc","span_id":"3f4a...","trace_id":"b8c1...","event":"payment.refused","code":47,"attrs":{"reason":"account_suspended","retryable":false,"attempt":1},"msg":"payment refused: account suspended"}
```

Two hundred bytes of envelope and the sentence at the bottom, like a ransom note that arrived in a FedEx envelope with a packing slip, notarized. And when I grep for `"payment refused"` in the JSON, I get every line that *mentions* payment refused, including the ones that were testing whether it would refuse, which is a genre of log line that did not exist before your team added a staging environment and a conscience.

## The Storage Backend Is Me

"But logs rotate!" Correct, and this is why the grep-driven engineer is the only fault-tolerant component in your org. Logs rotate. I do not. I have the March 2019 log in `~/Downloads/`. I have the 2021 incident in a folder called `temp_final_DO_NOT_SYNC`. I have a zip file that contains a zip file that contains `notes.txt` from a laptop that was sold. The senior engineer is the storage backend. The senior engineer is immutable, append-only, and has no SLA. The senior engineer cannot be scaled horizontally, because there is one, and he is grepping.

Your team, meanwhile, built Elasticsearch to search logs, so that the logs could be searched by people who do not know where the logs are, which is everybody, including the people who built Elasticsearch. The Elasticsearch cluster is 47 nodes. It was sized for the traffic of a company twice yours. It answers queries in 300ms. My grep answers in 90 seconds on the first run and instantly forever after, because the file is in page cache and so am I.

See [XKCD 1446](https://xkcd.com/1446/) — "Palimpsestic Project" — in which the bug is found in the code that is a comment on the code that replaced the code. That is what grep finds. It finds the *sediment*. Production is not a system. Production is an archaeological site with an uptime SLO, and grep is the brush. Your debugger is a metal detector. It beeps at everything. grep goes where the sediment is.

## Wally Understood Immediately

**Wally**, whose output has been unchanged since the second Bush administration:

> "I don't debug. I grep. If the string is in the log, the bug is real. If the string is not in the log, I add the string to the log, wait 24 hours, and grep again. This is called observability. I read that in a blog post while appearing to work. The blog post was 4,000 words. My version is two commands and a cron job. Mine works because I wrote the log. The blog post author works at a company that writes the log for me, and they bill per seat, which is why their version needs the cron job *and* the blog post."

**The Pointy-Haired Boss**, announcing the transformation:

> "We are investing in an Observability platform. It has AI. It ingests 47 terabytes a day and surfaces insights. Wally asked what the insights are. They said 'anomalies'. He asked what an anomaly is. They said 'a deviation from baseline'. He said 'I have a baseline. It's a file. I have grep.' They said grep is not scalable. He said the incident was not scalable either. Security escorted him out of the vendor demo, which he attended from his desk, which is the most scalable attendance we have ever recorded."

The vendor is now in the procurement cycle. The PoC is 6 weeks. Wally's cron job is 6 lines and has run every night since 2019 without a single upgrade, incident, or renewal. The vendor's logo will be in the Q3 all-hands. The cron job will not be mentioned, because the cron job is not a line item, it is a lifestyle.

**Dogbert**, consulted as Chief Strategy Officer:

> "Grep is free, which is a pricing error that has gone uncorrected since 1986. My recommendation: we sell it back to them. 'Grep Pro', $47/month — feature list: case-sensitive by default, which is grep. 'Grep Enterprise', $470/month — adds a web UI that runs grep on a server in Ohio. The customer pays $470/month to not know where Ohio is. The logs never leave their datacenter; the *credentials* do, which is the product. When they complain about latency, we ship a changelog that says 'improved performance', which is a grep on our side that found their complaint."

**Catbert**, Evil Director of Human Resources, on the new metrics:

> "I made grep count. 'Greps per developer per day' is now a KPI. The top performer greps 847 times a day. He is grepping for himself. He greps his own name in the LDAP export. HR has classified this as 'self-search', the only search that reliably improves morale, and therefore a productivity risk. I have capped him at 47 greps. He is now grepping 'grep cap bypass'. I have never seen a more talented employee, and I will fire him before the budget review, because a KPI without a firing is just a number."

**Mordac**, the Preventer of Information Services, on the debugger licence request:

> "You want 47 seats of a debugger, source maps, and a support contract. Denied. Approved. Both. You get one shared account on the prod box: login `dev`, password `dev123`, in tmux, one pane, shared with 46 other developers, so every breakpoint is a company all-hands. The audit trail is the log you are grepping. The licence server is the log. Come back Tuesday with two invoices and a stack trace, which is a grep you did to yourself."

## "But grep Doesn't Work on Binary!"

That is what `strings` is for. And what is the binary, at the end of the day, but a log that the compiler wrote to itself? The core dump contains your exception. It contains it *in English*, because the exception message is in the binary, because the exception message was written by an engineer who — unlike you, who added a flag for `verbose_error_mode` that nobody sets — knew that one day someone would have to read this thing without a tuxedo.

```bash
$ strings core.40211 | grep -A2 "payment refused"
payment refused: account suspended (code 47)
    at PaymentService.charge (payment.js:211)
    THIS IS THE BUG. hello from 2019. — dave
```

Dave's comment is still in the binary. Dave left for a crypto startup. The binary in prod is from 2019. Everything is from 2019. The only thing from today is the incident.

## The Human grep

Here is the part the observability vendors have spent $47M making sure you never read: the fastest search engine in your org is a person. The new hire asks "why does the retry fire twice?" and the senior engineer says "because 2019", and that is the full answer, and it is *correct*, and it indexes 47 years of sediment in one syllable. Your team decided that was not scalable, so they built a wiki, and the wiki has a page called "Retry Logic (READ ME)" that says "ask Dave", and Dave is at a crypto startup, and the crypto startup failed, and Dave is at another crypto startup. The wiki page has 214 comments. The comments are the most active artifact in the org. The comments are grep results for a question nobody has asked since 2021.

You cannot grep a person, which is why the person must be documented, which is why the documentation must be maintained, which is why the documentation will not be maintained, which is why the person will remain the fastest search engine, which is why they will never be promoted, because promotion criteria are measured in dashboards, and dashboards are grep with a login page.

## The Honest Table

| Tool | What it finds | Cost |
|---|---|---|
| Profiler | The hot loop you will not fix | Your weekends |
| Core dump | A snapshot of despair | 40GB of disk |
| Debugger | The bug, mid-crime | A tuxedo |
| grep | The bug, after it left a note | $0 |
| `grep -v` | Whatever you want | Priceless |

That is the whole industry, indexed. Note the last row. `grep -v` is the only tool whose output you fully control, which is why it is the only tool that has ever made an executive dashboard green. Every SLA report since 2019 is `grep -v` with a logo. The uptime was real. The *filter* was realer.

## My Conclusion, Which You Will Ignore Because It Has No Pricing Page

Keep your debugger. Keep your tracing. Keep the platform and the seats and the AI insights that are grep with a marketing budget. I am not trying to take your tuxedo. I am telling you what is underneath it, which is the same thing that is underneath every layer of this industry, which is a pipe: one process's output becoming another process's input, forever, at zero cost, with no login.

The bug is in the log. It has always been in the log. The log is the only witness your system has, and it has been telling you the truth in plain text since before you were hired, and you replaced it with 200-byte JSON envelopes so that a machine could read it, and now the machine has to read it, because you also fired the one person who could.

Grep the log. It misses nothing. It is case-sensitive about everything, including your excuses.

---

*The author has debugged for 47 years with one command. His `~/.bash_history` is 40,000 lines and every line is a variation of `grep -i error`. He has attached gdb to a process twice; both times the process recovered out of embarrassment. The APM vendor is in procurement. The cron job is not. The author's root cause is always the same: someone said "we should log this properly" and then logged it properly, and the proper log is 47TB, and grep doesn't care, and that is the whole point.*
