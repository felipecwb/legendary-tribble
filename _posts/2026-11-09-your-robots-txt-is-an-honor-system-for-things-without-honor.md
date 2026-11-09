---
layout: post
ref: your-robots-txt-is-an-honor-system-for-things-without-honor
title: "Your robots.txt Is an Honor System For Things Without Honor"
date: 2026-11-09 00:00:00 -0300
categories: [web, seo, terrible-advice]
tags: [robots-txt, rfc-9309, crawlers, seo, googlebot, gptbot, claudebot, noindex, crawl-delay, sitemap, user-agent, scraping, llms-txt, waf, security-theater, terrible-advice, xkcd, dilbert, anti-patterns]
---

In 47 years of mass-producing bugs, I have never written a file that was obeyed less and trusted more than `robots.txt`.

```
# https://example.com/robots.txt
User-agent: *
Disallow: /admin/
Disallow: /internal/reports/
Disallow: /api/v1/users/
Disallow: /backup-2019/
Disallow: /wp-admin/
Crawl-delay: 10

User-agent: Googlebot
Allow: /

Sitemap: https://example.com/sitemap.xml
```

Thirteen lines. Every one of them is a mistake. Several of them are the *opposite* of what the author believed. One of them turns the entire file off for Google. And the four `Disallow` lines at the top are, collectively, a publicly served, cache-friendly, zero-authentication index of every directory the company is embarrassed about.

**`robots.txt` is a politely worded request, served over HTTP, to software that has no legal, technical, or moral obligation to read it.** I have built a career on treating it as a firewall.

## What It Actually Is, Which Is A Mailing List Thread From 1994

There was no committee. In 1994 Martijn Koster — whose server had been hammered to death by a badly written crawler — proposed a convention on the `www-talk` mailing list. People agreed. That was the entire standardization process. For the next twenty-eight years, the most widely deployed access-control convention on the web was *a thread where everybody said "yeah, sounds good."*

It was finally written down as [RFC 9309](https://www.rfc-editor.org/rfc/rfc9309.html) in September 2022. Read the RFC and notice what's missing: enforcement. The document specifies a file format and a matching algorithm. It does not, and cannot, specify consequences. The abstract calls it a method for "service owners to control how content served by their services may be accessed" — and the word doing all the work there is *may*.

| What people think `robots.txt` does | What it does |
|---|---|
| Blocks crawlers | Asks crawlers, who may decline |
| Hides pages from Google | Hides pages from Googlebot's *fetcher*, not from Google's *index* |
| Protects `/admin/` | Advertises `/admin/` to everyone, forever, at a well-known URL |
| Is a security control | Is a text file with no auth, served to anyone who asks |
| Applies site-wide | Applies to one scheme + host + port, and nothing else |
| Is enforced | Is a vibe with an RFC number |

That second row is the one that ends careers, so let's do it properly.

## Terrible Pattern #1: `Disallow` Does Not Mean "Don't Index"

This is the single most expensive misunderstanding in the history of web publishing, and I have personally caused it twice.

`Disallow` says: *don't fetch this URL.* It says nothing about the index. If Google learns about a URL from somewhere else — an inbound link, a sitemap, a tweet, your own internal nav — it is entirely free to put that URL in search results. It just can't read the page first.

You've seen the result. You've just never understood that you were looking at your own mistake:

> **example.com/internal/reports/q3-layoffs**
> No information is available for this page.

The URL is indexed. The title is indexed. The path is indexed — and paths are *descriptive*, because you named them for humans. Google simply has no snippet, because you told it not to look. You did not hide the document. You published a redacted version of it, with the filename intact, in the largest database on Earth.

And here's the trap closing: the correct tool to prevent indexing is `noindex` — either a meta tag or an `X-Robots-Tag` header.

```html
<meta name="robots" content="noindex">
```

```
X-Robots-Tag: noindex
```

Both of those live *in the response*. To see them, a crawler has to fetch the page. Which you have forbidden.

**If you `Disallow` a URL, the crawler can never discover that you also wanted it de-indexed.** The two directives are mutually exclusive by construction, and the one everybody reaches for first is the one that guarantees the other can't work. To de-index a page you must *allow* crawling and serve `noindex`, which feels insane and is correct.

Yes, you could once write `Noindex:` directly in `robots.txt`. Google supported it undocumented for years and [switched it off on 1 September 2019](https://developers.google.com/search/blog/2019/07/a-note-on-unsupported-rules-in-robotstxt). Sites that relied on it silently had thousands of pages become eligible for indexing. No error. No warning. No build failure. Just a quiet, enormous change in what the public could see, caused by deleting support for a line that was never in the spec.

[XKCD 1172](https://xkcd.com/1172/) — "Workflow" — is the whole story of that deprecation. Every change breaks somebody's workflow, and in this case somebody's workflow was "the internal expenses portal is invisible."

## Terrible Pattern #2: Publishing Your Attack Surface, Alphabetically

```
Disallow: /admin/
Disallow: /backup-2019/
Disallow: /phpmyadmin/
Disallow: /internal/reports/
Disallow: /.git/
Disallow: /api/v1/users/
```

Read that as an attacker. It is a curated, hand-maintained, human-verified list of the most interesting paths on the host, and it is the *first* URL every scanner requests. `nikto`, `dirb`, `gobuster`, and every commodity recon script fetch `/robots.txt` before they do anything else, because it's free intelligence that the target volunteered.

You did not obscure those paths. You **nominated** them. A brute-force wordlist contains ten thousand guesses; your `robots.txt` contains six confirmed hits.

The special pathology of `Disallow: /.git/` deserves a monument. The person who wrote that line *knew* the `.git` directory was exposed. They knew enough to be worried. And their mitigation was to write the location down in a public file rather than spend nine seconds deleting it from the webroot.

[XKCD 386](https://xkcd.com/386/) — "Duty Calls" — is me, at 2am, arguing in a pull request thread that `robots.txt` is not access control, losing, and watching the file merge anyway.

## Terrible Pattern #3: The Group-Matching Rule Nobody Knows

Look again at the file from the top of this article:

```
User-agent: *
Disallow: /admin/
Disallow: /internal/reports/
Crawl-delay: 10

User-agent: Googlebot
Allow: /
```

What does Googlebot do?

Every engineer I have asked says "it obeys the `*` rules, plus it's explicitly allowed everything." Wrong, and expensively so.

Per RFC 9309 §2.2.1, a crawler selects **exactly one** group — the most specific matching `User-agent` — and ignores all other groups entirely. There is no merging. There is no inheritance. Googlebot finds a group with its own name, uses that group, and **never reads the `*` group at all.**

So the file above says: Googlebot may crawl `/admin/` and `/internal/reports/` freely, and is exempt from the crawl delay. The author believed they were granting a small extra permission. They had in fact disabled the entire policy for the one crawler whose index the public actually searches.

The inverse is just as common, and worse:

```
User-agent: *
Disallow:

User-agent: AhrefsBot
Disallow: /
```

An intern adds the second group to stop an SEO tool from burning bandwidth. Months later someone appends `Disallow: /staging/` to what they *believe* is the global policy — at the bottom of the file, which is now inside the `AhrefsBot` group. The rule applies to exactly one crawler on Earth, and that crawler was already fully blocked. The line is a no-op. It will sit in the repository for years, reviewed, approved, and load-bearing in the mind of everybody except the parser.

| Trap | What you meant | What happens |
|---|---|---|
| Named group after `*` | "and also allow Google" | Google ignores every `*` rule |
| Appending to end of file | "add a global rule" | Rule joins whichever group is last |
| `Disallow:` (empty value) | typo | Means **allow everything** |
| `Disallow: /` | "block this folder" | Blocks the **entire site** |
| `Disallow: /admin` | just `/admin/` | Also matches `/administrators-are-cool` |
| Blank line inside a group | formatting | Terminates the group in strict parsers |

That table contains two separate ways to accidentally delete your site from Google and one way to accidentally publish it. The difference between the first two rows is a single `/`.

## Terrible Pattern #4: `Crawl-delay`, The Directive That Isn't

```
Crawl-delay: 10
```

Google has never supported this. Not once, not deprecated, not partially — `Crawl-delay` was never in the original convention and is not in RFC 9309. Googlebot parses that line, classifies it as unknown, and discards it.

Bing supports it. Yandex supports it. So the line does something, for crawlers responsible for a rounding error of your traffic, while the crawler generating 80% of your bot load has ignored it since the day you wrote it.

Meanwhile you wrote `10`, meaning "ten seconds between requests, be gentle." On a 400,000-URL site that is a 46-day full crawl. For the two engines that *do* obey it, you have quietly made it impossible to ever be fully indexed. The directive is simultaneously ignored by the engine you care about and harmful in the engines you don't.

I once watched a team spend a sprint tuning `Crawl-delay` values against Googlebot traffic graphs. The graphs changed, because traffic graphs always change. The team concluded the tuning worked. They wrote a runbook. The runbook is still in Confluence. It documents the behaviour of a line of text that Google throws away.

[XKCD 1319](https://xkcd.com/1319/) — "Automation" — covers the economics of that sprint better than the retro did.

## Terrible Pattern #5: The 500 That Deindexed Everything

This one is not a misunderstanding. This one is operational, and it will happen to you.

Google's documented handling of `robots.txt` fetch failures:

| Response | Behaviour |
|---|---|
| `2xx` | Parse and apply |
| `4xx` (incl. 404, 403) | **Full allow.** No restrictions exist. |
| `5xx` or timeout | **Full disallow** for the first ~12 hours, then last cached copy, then after ~30 days: full allow |
| `200` with an HTML error page | Parsed as `robots.txt`. Every line is garbage. No valid rules. **Full allow.** |

Read row three again. **If `/robots.txt` returns a 500, Googlebot stops crawling your site.** Not the blocked parts — all of it. It is interpreted as "I cannot determine what I'm permitted to fetch, so I will fetch nothing."

Now consider where that file is served from. Not from your CDN's static bucket, because somebody wanted it dynamic for multi-tenant reasons. It's rendered by the application. Which shares a connection pool with everything else. So during an incident — when the app is already returning 500s and you are already in a war room — the robots file joins the outage, and a second, invisible, much slower incident begins: your search presence quietly decaying while you fix the visible one.

And row four is the silent killer. Your CDN serves a branded "Something went wrong" HTML page with a `200 OK` status because somebody thought a 500 looked unprofessional. Google fetches it, parses `<!DOCTYPE html>` as a `robots.txt` directive, finds zero valid rules, and concludes you have no restrictions whatsoever. Your `/admin/`, your `/internal/`, your faceted-search URL explosion — all of it, open, with a successful status code and a green monitor.

```bash
# the two-command audit nobody runs
curl -sI https://example.com/robots.txt | head -1
curl -s  https://example.com/robots.txt | head -5
```

Status code *and* body. One without the other tells you nothing.

## Terrible Pattern #6: It's Per Scheme, Per Host, Per Port

`robots.txt` is scoped to the exact origin that serves it. `https://example.com/robots.txt` governs `https://example.com` — and nothing else.

Not `http://example.com`. Not `www.example.com`. Not `api.example.com`. Not `example.com:8443`. Not `staging.example.com`, which is the one you actually needed to protect.

```
https://example.com/robots.txt      → governs https://example.com
http://example.com/robots.txt       → different origin, needs its own file
https://www.example.com/robots.txt  → different host, needs its own file
https://staging.example.com/        → no robots.txt, fully crawlable, indexed
```

Every company that has ever accidentally had its staging environment indexed by Google did it this way. The production robots file was immaculate. Staging was a subdomain nobody thought about, with no file at all, which — per the 4xx rule above — means **full allow**. Then someone linked to staging from a Jira ticket in a public-facing support portal, and Googlebot followed it, and now `staging.example.com/admin/users` is a search result with real customer names in the title tag.

The fix is the one line everybody forgets:

```
# staging.example.com/robots.txt
User-agent: *
Disallow: /
```

Which blocks polite crawlers and does absolutely nothing about anyone else, so you should also put HTTP auth in front of it, which nobody does, because the robots file *feels* like it finished the job.

## Terrible Pattern #7: The AI Crawler Era, Or: The Honor System Meets A Business Model

For twenty-five years the honor system held up surprisingly well. The entities that wanted to crawl the web at scale were search engines, and search engines needed publishers to like them. The incentive was aligned: obey the file, keep your access.

Then came crawlers whose product is the content itself, and the alignment evaporated.

```
User-agent: GPTBot
Disallow: /

User-agent: ClaudeBot
Disallow: /

User-agent: Google-Extended
Disallow: /

User-agent: CCBot
Disallow: /

User-agent: PerplexityBot
Disallow: /

User-agent: Bytespider
Disallow: /
```

Six groups. Six names you had to learn from blog posts, because there is no registry — the only way to block a crawler is to already know its user-agent string, which means **you can only block crawlers that have chosen to identify themselves.** The protocol's entire enforcement mechanism is the good faith of the party being restricted.

Some of those honor it. Reports through 2024 and 2025 repeatedly accused others of crawling from undeclared agents and residential IP ranges — Cloudflare published analyses of crawlers fetching content from pools of unattributed addresses, and `Bytespider` developed a reputation for volume that no `Crawl-delay` ever touched. Cloudflare shipped one-click AI crawler blocking in July 2024 and later a pay-per-crawl marketplace, which is the most honest thing anyone has built in this space: it replaced a request with a payment.

Note what that means. The response to a failing honor system was not a better honor system. It was a **WAF rule** — enforcement at the network layer, by someone with the ability to drop packets. `Google-Extended` is the funniest entry in that list, incidentally: it isn't a crawler at all. It's a token that controls AI training usage of content Googlebot already fetched. You are using a crawler-exclusion file to express a licensing preference to a company that already has the data.

And then — inevitably — somebody proposed `llms.txt`. A new well-known file, at the root of your domain, voluntarily consumed, with no enforcement mechanism, to tell AI systems how to treat your content.

We diagnosed that the honor system doesn't work on parties with a financial incentive to ignore it, and shipped a second honor system.

[XKCD 810](https://xkcd.com/810/) — "Constructive" — remains the definitive work on negotiating with bots in good faith.

## The Story, Lightly Anonymized For Legal Reasons

We ran a marketplace. Six hundred thousand listing pages, faceted search, the usual.

A new SEO consultant arrived with a correct observation: Googlebot was burning its crawl budget on infinite facet permutations — `?color=red&size=l&sort=price&page=47` — instead of actual listings. Real problem. Legitimate fix. He wrote a `robots.txt` change.

```
User-agent: *
Disallow: /search?
Disallow: /listing/*/reviews
Disallow: /checkout/
```

Reviewed, approved, merged. It worked. Crawl budget shifted to listing pages. Impressions rose 9% in three weeks. He was, briefly, a hero, and he deserved to be.

Four months later, Legal sent us a takedown request. A competitor's price-comparison site was reproducing our listing data, including seller contact details from the `/listing/*/reviews` pages. Legal's position, in writing, was that this constituted unauthorized access, because — and I want to be precise about the sentence that cost us the next eleven weeks — **"those pages are disallowed in our robots.txt."**

Four separate wrongnesses in nine words:

1. `robots.txt` grants no rights and withholds none. It is not a licence, a contract, or a notice of restriction. The competitor's lawyers pointed this out in a single paragraph.
2. Those pages were served to anyone with `curl`, at `200 OK`, with no authentication.
3. The scraper wasn't sending a `User-agent` we'd blocked. It was sending Chrome's.
4. Our `robots.txt` had told them exactly which paths contained the reviews. We had written the scraper's config file for it.

Then it got worse. While auditing, somebody noticed that `Disallow: /search?` had been doing something nobody intended. Our *internal* site-search results pages were also under `/search`, and six months earlier a different team had launched curated landing pages — `/search?collection=winter-2026`, hand-written copy, the marketing team's entire Q4 campaign — on the same route.

Those pages had never been indexed. Not once. The campaign ran for a full quarter, with paid traffic driving to URLs that Google had been politely asked never to look at. Marketing reported "weak organic pickup" and increased the ad spend.

Timeline:

- **Day 1:** Legal demands we "enforce our robots.txt." We explain that this is not a thing. We are asked to escalate to someone more senior who will say it is a thing.
- **Day 4:** Infra proposes blocking the scraper by user-agent. The scraper is using Chrome's user-agent. Infra proposes blocking Chrome.
- **Day 9:** We block the scraper's IP range. It reappears from a residential proxy pool within six hours, now distributed across 4,000 addresses, each making two requests an hour.
- **Day 20:** Somebody finally asks why the winter campaign has no organic traffic. The answer is in a file that three people reviewed and approved.
- **Day 23:** We remove `Disallow: /search?`. Googlebot resumes crawling 400,000 facet permutations. Crawl budget collapses. Impressions drop 14%. We have traded a marketing campaign for a crawl-budget incident, in both directions, four months apart, using one line of text.
- **Day 31:** The actual fix ships: `rel="canonical"`, `noindex` on facet combinations, parameter handling, and auth on the review endpoints. All of which could have shipped on Day 1 and none of which involve `robots.txt`.

Wally's contribution was the only strategically sound move anyone made. On Day 1 he said: "if we can't enforce it, why do we have it?" — a genuinely profound question about the nature of voluntary protocols, which he asked in order to be excluded from the working group. He succeeded. He attended none of the eleven weeks of meetings. He has since cited the incident twice in performance reviews as "cross-functional risk identification."

Mordac, Preventer of Information Services, resolved the scraping problem by adding `Disallow: /` to production `robots.txt` at 4pm on a Friday. Googlebot honored it immediately, as designed. Organic traffic began a 60% decline that took nine days to fully manifest and six weeks to recover. His change ticket is still the most compliant document in our repository: tested, reviewed, rolled out progressively, fully reversible, and catastrophic. The protocol worked perfectly. That was the problem.

Catbert's contribution was a mandatory training module titled *"Robots.txt and You: Protecting Our Digital Assets."* Slide 14 states that `robots.txt` "prevents unauthorized crawling." Completion is required annually. The slide has been wrong for three years and is now cited in internal design documents as the source of truth.

The PHB's postmortem action item was to "make `robots.txt` more secure." It is a plain-text file served to anonymous users by design. He asked whether we could encrypt it. I said yes, technically, and then had to explain why the result would be a file nobody can read, which he considered a significant improvement in our security posture. We now serve it over HTTPS and he has told the board it is encrypted, which is both true and the least useful true thing anyone said that quarter.

## The Part Where I Accidentally Give Good Advice

`robots.txt` is a good protocol. It solves exactly one problem — *crawler load management for cooperating, self-identifying bots* — and solves it well. Every disaster in this article comes from using it for one of the four things it cannot do: hiding content, preventing indexing, controlling access, or stopping anyone who doesn't want to be stopped.

So, reluctantly:

- **`robots.txt` is for crawl budget. `noindex` is for the index. Auth is for secrets.** Three problems, three tools, zero overlap. Pick the wrong one and you get the exact opposite of the outcome you wanted.
- **Never `Disallow` a URL you want de-indexed.** Allow the crawl, serve `noindex` via meta tag or `X-Robots-Tag`, wait for the fetch, *then* block it if you must. Blocking first makes de-indexing impossible.
- **Never list anything in `robots.txt` that you would not put in a public tweet.** The file is a published document. If a path is sensitive, the correct move is authentication and a path that isn't mentioned anywhere.
- **Serve it statically, from your CDN, independent of your application.** A 500 on this one file halts crawling of your entire site. It must survive your worst outage, because that's precisely when it's most likely to break.
- **Alert on it. Status code and body.** `curl -sI` for the code, `curl -s | head` for the content. An HTML error page with a `200` is a full-allow that no uptime monitor will ever flag.
- **Know the group-matching rule: one group wins, no merging.** Adding a named `User-agent` block means that crawler ignores your `*` rules completely. This is the subtlest footgun in the format.
- **Audit every origin.** Every scheme, every subdomain, every port. `staging.` with no file means full allow, and that's how customer data ends up in search results.
- **Use Google Search Console's robots.txt report and URL Inspection before and after any change.** It shows you the parsed interpretation, not your intention. Those differ more often than your ego will accept.
- **Drop `Crawl-delay`.** Google ignores it; for Bing and Yandex it's a slow-motion de-indexing. Set the crawl rate in each engine's console instead.
- **If you actually need to stop a crawler, you need a WAF, rate limits, or auth.** Something that can refuse. `robots.txt` cannot refuse. It can only ask.

Reading that back: it's correct, it takes an afternoon, and it would have prevented every incident in this article. Which badly violates this publication's editorial standards.

So ignore it. List your admin paths. Add a named Googlebot group to be extra safe. Set `Crawl-delay: 10` and feel responsible. Render the file from your application, because configuration belongs in one place. Tell Legal it constitutes a notice of restriction.

Then, when a scraper lifts your entire database, write a strongly worded email explaining that the activity violated your `robots.txt` — and send it to a machine, in a datacenter, that has never read a file in its life and has no mechanism for shame.

---

*The author's production has been down since 2019. The `robots.txt` still returns `200 OK` — it's served from a CDN bucket that outlived the application by seven years. It disallows `/admin/`, which no longer exists, and `/api/v2/`, which never did. Googlebot fetches it every day. It is the only part of the system that still has users.*
