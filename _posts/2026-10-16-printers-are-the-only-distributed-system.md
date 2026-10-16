---
layout: post
ref: printers-are-the-only-distributed-system
title: "The Office Printer Is the Only Distributed System That Has Ever Worked"
date: 2026-10-16 00:00:00 -0300
categories: [distributed-systems, hardware, anti-patterns]
tags: [printers, distributed-systems, spooler, consistency, observability, firmware, drivers, terrible-advice, dilbert, anti-patterns]
---

I have 47 years of experience, and in that time I have worked on distributed systems with Raft, Paxos, quorum writes, consensus protocols, vector clocks, and a Kafka cluster that cost more than the building it was racked in. I have one finding to report, and it is the only finding I have ever been certain of: **the office printer is the only distributed system that has ever actually worked.** Not reliably — reliably is a fantasy sold by vendors with slide decks. I mean *consistently*. The printer has been failing since 1971, and it has failed the same way every single day. Three blinking lights. A paper jam. The word "Offline" on a device that is physically three meters from you. Your Kubernetes cluster has been up for four years and you still don't know what it does. The printer has no README and everybody in the building knows its current state. That is not a failure of technology. That is the most successful observability rollout in human history.

You think this is a joke. It is not a joke. It is the only serious architectural advice I have. Model your microservices after the printer. By the end of this article you will understand why, you will refuse to do it, and the pager will thank me personally.

## The Spooler: The Original Message Queue

Every distributed systems conference I have ever attended has featured at least one talk about message queues, delivered by a man who has never printed a document. The talk announces that queues "decouple producers from consumers" and "absorb bursts" and provide "at-least-once delivery," and the audience nods, because they have heard the words before. Meanwhile, the print spooler has been doing all of this since the Nixon administration, for free, with a UI that is four checkboxes and a button that says "Print" even when it should say "Give Up."

The spooler has at-least-once delivery. It has dead-letter handling: when the document fails to print, the spooler does not send you a 500, a stack trace, and a link to a runbook. It just leaves the document in the queue, silently, forever, where it ages like a fine wine until someone eventually unplugs the printer at 6 PM, which is the only garbage collection this industry has ever needed. Your Kafka topic has a retention policy. The spooler has *forgetting*. Forty-seven years of production experience has taught me that forgetting is the superior data retention strategy: retention is just denial about the past.

And the spooler is a *global* queue for the entire building. Everyone's documents go into one queue and come out in roughly the order they were submitted, which is to say the printer implements FIFO with a best-effort delivery guarantee, the exact same semantics as your message bus, except the printer's version fits on a screen from 1997 and its SLA is "it will probably come out before the meeting."

## The Paper Jam: Single Global Lock, Perfectly Enforced

Here is where the printer puts your consensus protocol to shame. When the printer jams, the *entire system stops*. Not one shard. Not one partition. Not one service with a retry loop that quietly duplicates the invoice. **Everything stops, and it stops for everyone at the same moment, and everyone finds out at the same moment.** That is linearizability. That is what Raft has been trying to sell you for eleven years. The printer ships it in a plastic box that costs $89 and beeps.

Your distributed database claims "strong consistency" and then, under network partition, happily serves stale reads to the sales team while finance writes to the other side of the split brain, and afterward the reconciliation script runs at 2 AM and produces a report that one person reads and then describes as "weird, but fine." The printer has never had a split brain. Nobody has ever looked at the output tray and the paper tray and disagreed about how many copies exist. When the printer says 47 copies were made, 47 copies were made, because the alternative is a blinking light and a lever on the side that demands human intervention. That lever is the strongest consistency mechanism ever engineered: it converts a distributed consensus problem into a physical dispute between a person and a piece of paper, and the person always loses, which is why the ledger is always correct.

Microservices people will tell you that single global locks don't scale. The printer scales. The printer has scaled to the exact workload it has, which is the correct amount, and it will never grow, which makes it the only system in the building with a capacity plan.

## Three Lights: The Only Observability Stack That Has Ever Been Read

Your observability stack: 4,000 dashboards, a tracing UI, a service mesh with golden signals, an SLO for everything, error budgets, a Grafana folder structure with naming conventions agreed upon at an offsite, and alerting rules reviewed by a committee. Reads per day: the on-call engineer, once, when the pager fires, if the pager fires, which it doesn't, because the alert thresholds were tuned in 2022 to be polite.

The printer's observability stack: **three lights.** Green: working. Yellow: thinking about it. Red: state of the union. Every human in the organization can read this stack with no onboarding, no training module, and no documentation. The CEO can read it. Visitors can read it. My dog could read it, and my dog has no concept of distributed consensus, which is another way of saying he understands systems better than your SRE team.

The printer also has error messages, and they are famously bad — "PC LOAD LETTER," a phrase that has confused office workers since 1984 and that I still cannot explain, and I do not need to, because see my earlier work: cryptic error messages are a security feature. But here is the part your vendors will never admit: the printer's error message is *accurate*. When it says there is a paper jam, there is a paper jam. There is a physical location for the jam. You can walk to it. You can touch it. You can remove the paper. Compare that to your observability stack, which detects "elevated p99 latency in the checkout service" — accurate as far as it goes, actionable as far as the moon. Your tracing UI cannot tell you *where the paper jam is*, because your system does not have paper jams, it has *mystery slowdowns that resolve themselves before the on-call engineer opens the dashboard*, which is the same thing as a paper jam except the paper jam is honest about being a piece of paper.

See [XKCD 927](https://xkcd.com/927/) — "Standards" — where one new standard is proposed to solve the problem of fourteen competing standards, resulting in fifteen competing standards. The printer predates this problem. The printer has exactly one standard, and it is a light that is either on or off, and it has defeated every observability vendor since the CRT.

## Firmware Updates: The Only Honest Canary

Your canary deployment: 5% of traffic for four hours, metric comparison, automated rollback on regression, a dashboard, a runbook, a Slack channel named `#deploy-watch` where nobody types. The printer's canary deployment: it notices an update is available, applies it to itself, at 6 PM, on a Friday, while everyone is watching, and either it works or it bricks itself so thoroughly that the only fix is a firmware downgrade performed with a USB stick and a paperclip, a procedure documented in a PDF that the vendor has deleted and that survives only because someone named Gerardo printed it in 2019.

This is *stronger* canary discipline than you practice. Your canary rolls back on regression. The printer does not roll back. It commits. It commits so hard that the failure mode is physical and permanent, and that is the only deployment strategy that has ever kept anyone honest: when rollback is impossible, release engineering becomes a religion, and the printer is the most religious system I have ever operated. I have shipped software for 47 years and I have never once felt the way I feel when the printer's progress bar says "Updating. Do not power off." Powering off is not an option. There is no flag. There is no config. There is a request from the machine, and the machine has never once been willing to negotiate, which is the definition of an SLA.

## Drivers: The Only Cross-Platform Story That Ever Worked

Your cross-platform strategy: containers, for "write once, run anywhere," except it runs differently on Apple Silicon, and your Makefile assumes GNU sed, and the setup script works on every machine except the machines people actually have. Onboarding a new engineer takes two days, and both days are spent installing tools that fail for reasons that are different from the reasons listed in the README.

The printer's cross-platform story: a driver. It is 140 MB, it was written in 2011, it is not signed, and it works on every operating system ever shipped, including operating systems that were discontinued before the driver's author finished writing it. The driver has survived four macOS versions, three Windows regimens, and one Linux distro whose name the company asked me not to mention. When the driver breaks, you reinstall it. When that fails, you turn the printer off and on. This is the entire troubleshooting tree. It has two leaves. Yours has 847, and 845 of them are links to Confluence pages that were written by people who no longer work here.

Compare this to your microservices, which each ship their own client library, generated from an OpenAPI spec that is four releases behind the service, and which all disagree about timeout semantics. The printer has one protocol, it is called PCL, and it is spoken by every printer made since 1984 in a dialect so stable that documents printed in 1987 print correctly today, on hardware that did not exist when the document was written. That is *backward compatibility*, the thing your REST API has been promising since 2015 and delivering never. The printer has been keeping that promise since before your framework had a name. Some frameworks have had three names.

## The Table Nobody Wanted

| Property | Office Printer | Your Kubernetes Cluster |
|---|---|---|
| Uptime | Fails daily, at the same hour | Has never failed, officially |
| Consistency | Perfect: one global paper jam | Eventual, mostly, except Tuesdays |
| Observability | Three lights, read by everyone | 4,000 dashboards, read by one sad engineer |
| Error messages | "PC LOAD LETTER" (accurate) | "Error: something went wrong" (lyrical) |
| Retry semantics | Refuses until unplugged, then forgets | Retries 3x, fails, retries 9x, fails, retries forever |
| Onboarding | Ten minutes; the button is green | Two weeks; the button is `kubectl` |
| Backward compatibility | Since 1984 | Since the last breaking change |
| Capacity planning | The exact workload it has | Whatever the cloud bill says |
| Deploy cadence | Once, bricked, paperclip | 47 times since Tuesday |

Notice which column reads like a system you would trust your payroll to. The printer's uptime line is the only line in the table that has never lied to anyone. "Has never failed, officially" is doing a lot of work in that cell, and the word "officially" is load-bearing, which is more than can be said for most of your services.

## The Chief Printer Officer

I have worked with every character in that strip, and every one of them has opinions about the print infrastructure, which is the only infrastructure anyone has opinions about, because it is the only infrastructure they can see.

**The Pointy-Haired Boss**, announcing the modernization initiative:

> "Great news. We're moving the printers to the cloud. The vendor calls it Print-as-a-Service. From now on, when you print, the document travels to a data center in Virginia, comes back, and prints from the machine you're standing next to. It's called a round trip. It's very modern. The vendor says it's the same number of steps as before, except now there's a login, and the login is SSO, and the SSO redirects to a page that says 'Your session has expired.' I saw a demo. In the demo, it worked."

**Wally**, on why he keeps a piece of paper taped over the printer's network port:

> "The printer used to work, and then I would print things, and people would know I was awake. Now the printer is 'offline,' officially, since 2019, and IT has a ticket open, and the ticket is assigned to someone who left the company, which means the printer will remain offline forever, which means I can print whenever I want, from the USB port, at my desk, in peace. The ticket is the most reliable software we have. It never closes."

**Dogbert**, consulted as Chief Strategy Officer:

> "Your problem is that the printer works too well. Anyone can print anything. I recommend metered printing: each employee gets 47 pages per quarter, and when they run out, they must submit a form to a committee that meets quarterly. I have implemented this at other companies. Productivity dropped by 40% and morale dropped by 47%, but the cost savings were 3 cents, which is the only metric leadership tracks, so the project was a success and I have been asked to replicate it."

**Catbert**, Evil Director of Human Resources, on print quotas:

> "I love the quota system. It lets me deny requests in writing, which creates a paper trail, which requires printing, which consumes quota, which creates more requests to deny. The system is self-sustaining and requires no staff. I have recommended it to the board as a governance framework. They approved it unanimously, because it was the only agenda item that did not require them to print anything."

**Mordac**, the Preventer of Information Services, on print policy (this is his canonical job, and he is very good at it):

> "Print requests must now be submitted in triplicate. The triplicate copies must be printed, which requires printer access, which requires a print request, which must be submitted in triplicate. Access will be granted upon approval by the Print Access Committee, which meets whenever the printer is operational. The printer is currently operational. The Committee is not. Bring your own paper."

## "But the Cloud!"

I know what the vendor deck says. It says printing is a legacy workflow, that documents should live in the cloud, that paper is dead, that the future is a PDF that opens in a browser tab with a viewer that has one button and it is the wrong one. I have heard this pitch every five years since 1990, and every five years the pitch is delivered on paper, handouts, because the vendor knows something they will not put in the deck: **the meeting handout is the only distributed system with 100% read availability.** Nobody has ever gotten a 404 on a handout. The handout works during network partitions. The handout works when the VPN is down. The handout works when the SSO provider has an outage, which is how you find out the SSO provider is on the same page as the network, the same page being down.

Paper is the original offline-first architecture, and offline-first is the trend the industry keeps rediscovering every decade under a new name: PWA, local-first, edge cache, service worker. The paper document has all of it: it is cached, it is immutable, it is versioned by a date stamp in the footer, and it has a revision history that consists of someone writing "v2" on it. Your document management system has version history and I have never once seen anyone retrieve v2 on purpose. The paper is ahead of you, and the paper will outlive your SaaS. Your SaaS will be renamed to "Classic" and decommissioned. Paper has never been renamed. Paper has never been decommissioned. Paper has never had a pricing page.

## My Conclusion, Which Will Not Survive the Meeting

Model your distributed systems after the printer. One global queue. One global lock, enforced physically. Three lights, read by everyone, no training required. Error messages that point at the actual piece of paper that is wrong. Firmware updates that commit. A backward compatibility story measured in decades. And a state — "Offline" — that every human in the organization knows without a dashboard.

You will not do this, because the printer is not in your resume. Nobody has ever put "operated a printer" on a resume, and until that changes, your Kafka cluster will keep doing what Kafka does, which is exist, at cost, waiting for a workload that arrives once a quarter, at which point it will produce an outage that gets described in the postmortem as "unexpectedly successful."

The printer has been running since 1971. Your stack has been running since Tuesday. One of these systems has a paper jam, and the other one has a philosophy degree.

---

*The author has 47 years of experience and has never once fixed a paper jam by consulting a dashboard. He fixes them by opening the door, pulling out the paper, and closing the door, which takes eleven seconds and has a 100% success rate, which is higher than any SLO he has ever agreed to. He keeps a USB stick with printer drivers on his keychain, next to the keychain battery the locksmith sold him in 2014. The locksmith's solution has never failed. The drivers have never been needed. He brings both to every incident review, out of respect.*
