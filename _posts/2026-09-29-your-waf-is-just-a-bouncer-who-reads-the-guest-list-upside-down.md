---
layout: post
ref: your-waf-is-just-a-bouncer-who-reads-the-guest-list-upside-down
title: "Your WAF Is Just A Bouncer Who Reads The Guest List Upside Down"
date: 2026-09-29 00:00:00 -0300
categories: [security, networking]
tags: [waf, security, firewall, web-application-firewall, owasp, false-positives, false-negatives, bouncer, guest-list]
---

After 47 years in this industry, I have stood in front of a great many doors. I have stood in front of the door to production. I have stood in front of the door to the database. I have stood in front of the door to the datacenter, which had a real lock, and in front of the door to the cloud console, which had a virtual lock and a man in another country who had the password written on a Post-it. But the door that inspires the most false confidence — the door that is most expensive, most loudly advertised, and most consistently pointed at the wrong people — is the Web Application Firewall.

A WAF is, in theory, a bouncer. It stands at the door of your application and inspects every request that tries to enter. It checks the request against a list of known bad patterns — SQL injection, cross-site scripting, path traversal, the usual suspects — and either lets it in or throws it out. This is a fine idea. This is the idea behind every door, every lock, every bouncer, and every polite request to remove your hat. The problem is not the idea. The problem is that the bouncer is holding the guest list upside down, the guest list was last updated in 2019, and the bouncer is also a regular expression.

## The Bouncer Is A Regular Expression

Let us examine a WAF rule. Here is a representative rule from a representative WAF, which is to say, a WAF configured by a vendor who has never seen your application and a security team who has never seen the vendor's documentation:

```
SecRule REQUEST_URI "@rx /(?i)(union|select|insert|update|delete).*from" \
  "id:1001,phase:1,block,msg:'SQL Injection Attempt'"
```

A request comes in. The URI contains the word `union` followed, eventually, by the word `from`. The WAF blocks it. The WAF is satisfied. The security dashboard ticks up by one. The security team feels good about itself.

Here is what the WAF did not check. The WAF did not check whether the request was actually talking to a database. The WAF did not check whether the application in question even has a database. The WAF did not check whether the parameter was a string, a number, or a carefully crafted philosophical opinion about the nature of `UNION`. The WAF saw two words, in order, and it made a decision, the way a dog makes a decision about a squirrel: instantly, confidently, and with no regard for whether the squirrel is actually a squirrel or a plastic bag in the wind.

The request that was blocked was a `GET` to `/api/players?team=union&from=2024`. It is a search for athletes who play for a team whose name is "Union," filtered by a start year. There is no database injection. There is no SQL. There is no `SELECT`. There is only a sports fan, and a sports fan who is now staring at a 403 page, and a support ticket, and a support ticket that will be routed to the security team, who will look at the rule, and say "the rule is working as intended," and close the ticket, because the rule *is* working as intended — the intention was to block the word "union," and the word "union" has been blocked.

This is the false positive. The false positive is when the bouncer throws out a paying customer because the customer's name happens to contain a sequence of letters that also appears on the guest list of people who are not allowed in. The false positive is annoying. The false positive is recoverable. You can whitelist the rule. You can add an exception. You can call the vendor and wait six weeks. The false positive costs you a customer for an afternoon.

The false negative is worse, and the false negative is the entire business model.

## The False Negative Is The Business Model

A false negative is when the bouncer lets in the person who is on the list. The WAF did not block the request that was, in fact, a SQL injection. The WAF did not block it because the request did not contain the word `union` followed by `from`. The request contained the word `UNI/**/ON`, with a SQL comment in the middle, which the WAF's regular expression did not match because the regular expression was looking for the literal string `union` and the attacker put a comment in the middle of it, which is a technique that has been documented since 2007 and which the WAF vendor's rule set has not been updated to handle because updating the rule set would require the vendor to understand SQL, and the vendor writes rules in regular expressions, and regular expressions do not understand SQL, they understand characters, and characters are not meaning.

Here is the comparison the security vendor will not show you in the sales deck:

| Strategy | What it claims | What it actually does |
|---|---|---|
| Signature-based WAF rules | "Block known attacks." | Blocks requests that contain a list of strings an intern copied from a 2014 blog post. The intern has since left. The strings have not. Attackers who read the same blog post simply avoid the strings. You are protected against attackers who have not read the internet. |
| Machine-learning WAF | "Adaptive, learns your traffic." | Builds a statistical model of your traffic over two weeks, concludes that your traffic "looks normal," and then blocks nothing, because everything looks normal, because the model was trained on the traffic that was already arriving, which includes the attack traffic, which has been arriving since the model started training. The ML learned the attack as "baseline." |
| Negative security model (allowlist) | "Block everything not on the list." | The allowlist has 4,200 entries. The application has 4,201 endpoints. The 4,201st endpoint is the one that processes payments. It returns a 403 to every customer. Nobody notices for nine days because the monitoring dashboard is behind the WAF and the WAF is blocking the monitoring too. |
| Positive security model (blocklist) | "Block only known bad." | The blocklist is the signature list from row one, plus three custom rules the security team added after the last breach, each of which blocks a specific request that already happened. You are protected against the exact breach you already had, and nothing else. |
| Managed rules from your cloud provider | "Enterprise-grade, maintained by experts." | The experts update the rules once a quarter. The rules are the same rules they ship to every customer. The attacker who wants to bypass your WAF simply rents an account on the same cloud provider, reads the managed rules in the free tier, and designs around them. Your WAF is public knowledge with a monthly bill. |

Notice the pattern. Every strategy blocks something. Every strategy lets through something else. The something it lets through is, statistically, the something that matters, because the attackers read the documentation too, and the attackers are paid to read the documentation, and your security team is paid to attend a quarterly review of the documentation, and these are not the same incentive structures.

## The OWASP Top 10 Is A Reading List, Not A Firewall

The security team will tell you the WAF protects against the OWASP Top 10. This is true in the sense that a raincoat protects against the OWASP Top 10: it covers some of them, it gets wet, and it makes you feel like you did something.

The OWASP Top 10 is a list of the ten most common categories of web application security risks. It is a fine list. It is a list you should read. It is not a list your WAF can implement, because the list contains categories like "Broken Access Control" and "Cryptographic Failures," and these are not things you can detect with a regular expression on the request URI. Broken Access Control is a property of your *application logic*. It is the property that says "user A should not be able to read user B's data." The WAF sees user A's request. The WAF sees user A's request contains user B's ID. The WAF does not know whether user A is allowed to read user B's data, because the WAF does not know who user A is, who user B is, or what "allowed" means in this application. The WAF knows the URI. The WAF knows the URI contains a number. The number could be anything. The WAF lets it through.

As [XKCD 327](https://xkcd.com/327/) established with the clarity of a man who has been to a sales call: the person who can do the most damage with a database is the person who has been told they cannot, and the WAF is the thing that has been told it cannot, and it cannot, and the person who can is the person who did not read the guest list, because the guest list is upside down.

## What Dilbert Teaches Us About The WAF

The Pointy-Haired Boss, upon being shown the WAF dashboard, will say: *"So we're secure now?"* And the honest answer is: we are secure against the specific requests that were on the vendor's demo slide, and we are not secure against anything else, and the dashboard is green, and green is a color that means "no one has complained yet." The PHB will then approve the renewal, because the dashboard is green, and green dashboards are what the auditors look for, and the auditors are the only people the PHB is actually defending against.

Wally, who configured the WAF in the first place, will explain: *"I set it to 'monitor only' mode the day after it went live, because it blocked the CEO trying to log in. It's been in monitor only for three years. We get the alerts. We don't read the alerts. The alerts go to a mailbox that forwards to a mailbox that nobody has the password to. The WAF is a very expensive way to generate email that nobody reads."* Wally is describing, with the weariness of a man who has been to war, the most common deployment state of every WAF in production: monitor mode, forever, ignored.

Mordac, the Preventer of Information Services, would mandate blocking mode for every rule, would deny all exception requests on principle, and would consider a false positive rate of 30% to be "an acceptable trade-off for security." He would be wrong, but he would be wrong in a way that passes the audit, and the audit is what Mordac is optimizing for, because Mordac does not optimize for security, Mordac optimizes for the document that says he optimized for security.

## The Honest Recommendation

After 47 years, I do not recommend a WAF as your primary defense. I recommend the following, in order:

1. Parameterized queries. Every query. Every time. No exceptions. No "just this once." The injection stops at the parameter boundary, which is a place the attacker cannot cross, because the parameter is data and not code, and this distinction is the entire ballgame.
2. Authorization checks in the application. Every request. Every resource. The check is one line. The line is `if user.can_access(resource):`. The line is not in the WAF. The line is in your code. The WAF cannot write this line. The WAF does not know your users.
3. Output encoding. Every time you put data into HTML, JSON, or a shell command, you encode it for the context you are putting it into. XSS dies at the encoding boundary. The WAF cannot encode your output because the WAF does not see your output.
4. A WAF, in monitor mode, for the audit.

This costs less than the WAF. This defends more than the WAF. This does not require a vendor, a rule set, a regular expression, or a bouncer who reads the guest list upside down. The defense is in the code, because the vulnerability is in the code, and a thing in the network in front of the code is a thing that is not in the code and therefore not the thing that is broken.

But of course, you will not do this, because parameterized queries are not a line item in the budget, and the WAF is a line item in the budget, and we are an industry that buys line items and calls them security.

## Conclusion

Your WAF is a bouncer. The bouncer is standing at the door. The bouncer has a guest list. The guest list is written in regular expressions. The regular expressions were written by someone who has never met your guests. The bouncer is reading the list upside down, which is why the bouncer throws out the people whose names contain the wrong letters and waves through the people whose names contain the right letters arranged in the wrong order. The bouncer is also, increasingly, a machine learning model, which means the bouncer has read every guest list ever written and has concluded that the average guest is fine, and the average guest is fine, and the specific guest who is here to rob you is not the average guest, which is why the model did not flag them.

When the breach comes — and it will come, on a Friday, at 5 PM, through an endpoint the WAF has never seen because it was shipped last Tuesday — do not consult the WAF. The WAF will say the request looked normal. The request did look normal. The request was normal. The request was a `GET` to `/api/users/1234` where `1234` was someone else's user ID, and the WAF has no opinion about whether you are allowed to see user `1234`, because the WAF is not your application, and the WAF cannot be your application, and the WAF is a thing in front of your application that has been asked to do the job of your application, and it cannot, and it will not, and the audit will still pass, because the audit checks for the presence of a WAF, not the correctness of one.

The breach was never in the request the WAF blocked. The breach was in the request the WAF allowed, because the WAF allows everything that does not match a regular expression, and most things do not match a regular expression, and the thing that robs you is specifically the thing that was designed not to match any regular expression, because the attacker read the same documentation your vendor read, and the attacker read it more carefully.

---

*The author's WAF has been in monitor mode since 2021. It has generated 4.3 million alerts. He has read six of them. All six were false positives. The one real attack was caught by a database permission he set in 2008 and forgot about, which is the only form of security that has ever actually worked.*
