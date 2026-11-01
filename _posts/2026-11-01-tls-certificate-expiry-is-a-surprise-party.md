---
layout: post
ref: tls-certificate-expiry-is-a-surprise-party
title: "Your TLS Certificate Expiry Is a Surprise Party You Throw Yourself Every 90 Days"
date: 2026-11-01 00:00:00 -0300
categories: [security, devops, terrible-advice]
tags: [tls, ssl, certificates, lets-encrypt, certbot, acme, pki, x509, expiry, monitoring, curl, openssl, security, devops, terrible-advice, xkcd, dilbert, anti-patterns]
---

In 47 years of mass-producing bugs I have taken production down with a certificate expiry **eleven times**, and I want to be precise about something: it was the same certificate three of those times. Not the same service. The same certificate. It kept coming back, like a cat you keep driving to a different neighbourhood.

A certificate expiry outage is the most beautiful failure mode in computing, because it is the only one where the system schedules its own death, tells you the exact date, puts it in a machine-readable field, and then you ignore it. Every other outage is at least a surprise. This one is a calendar invite you declined.

So here is my doctrine, defended across four employers and one deposition: **certificate expiry is a surprise party, and the whole point of a surprise party is that you do not plan it.**

## The Renewal Cron I Shipped and Believed In

This ran on exactly one box. The box was named `util-01`. Nobody knows who created `util-01`.

```bash
# /etc/cron.d/certbot
# renew certs. runs every 90 days because the cert lasts 90 days.
# (this is the bug. it took me two years to see it.)
0 3 1 */3 * root certbot renew --quiet >> /var/log/certbot.log 2>&1

# the deploy hook, which I wrote at 4am during the outage it caused
0 4 1 */3 * root systemctl reload nginx
```

Count the catastrophes with me.

**One:** renewing *every 90 days* for a *90-day certificate* means you get exactly one attempt, and if that attempt fails — rate limit, DNS blip, port 80 firewalled by someone's "hardening sprint" — you do not find out for three months, and the way you find out is Twitter. The correct interval is twice a day. `certbot renew` is a no-op unless the cert is inside 30 days of expiry. It is designed to be run constantly. I was treating an idempotent command like a controlled substance.

**Two:** I separated the renewal from the reload by an hour, as two unrelated cron entries, because I thought of them as "two different jobs." So for one hour, nginx was serving the old cert from memory while the new one sat on disk. And on the quarters where renewal silently failed, the reload still ran, cheerfully reloading the expired certificate with full confidence, like a waiter re-serving a returned steak.

**Three:** `>> /var/log/certbot.log`. Not monitored. Never read. Rotated weekly. I had built a diary for a process whose only interesting output was a failure I had arranged not to see.

## My Monitoring Strategy, Which Was a Human Being

I did have alerting. The alerting was a man named Derek.

Derek had the expiry dates in a spreadsheet, which he maintained by hand, by running `openssl` once a quarter. Derek left in March. The spreadsheet was on Derek's personal Google Drive. We learned both of these facts on the same afternoon, which was also the afternoon the payments gateway cert expired.

Here is the entire monitoring system I replaced Derek with, which I wrote in nine minutes and never touched again:

```bash
#!/bin/bash
# cert-check.sh — "monitoring"
for host in api.example.com www.example.com; do
  echo | openssl s_client -connect $host:443 2>/dev/null \
    | openssl x509 -noout -enddate
done
```

Read what that does *not* do. It prints a date. To a terminal. To a human. Who has to subtract. It does not exit non-zero. It does not alert. It does not check the **intermediate** certificate, which can expire independently and ruin your week in a way that `curl` on your laptop will not reproduce because your laptop has the old intermediate cached in a trust store from 2021. It does not check the certs on internal services, the client certs, the certs inside the service mesh, the cert on the load balancer that terminates TLS *before* nginx, or the one on the SMTP relay. It checks two hostnames. We had four hundred.

And it only checks what's reachable from the box it runs on, so the internal-only mutual-TLS cert between two services — the one that took down order processing for five hours on a Sunday — was structurally invisible to it. [XKCD 2347](https://xkcd.com/2347/) is about the one unpaid maintainer holding up all of modern infrastructure, and in our case that maintainer was a shell script with no error handling and a man who had already found a better job.

## The Expiry Ladder of Denial

Every team passes through these. I have personally escorted six.

| Stage | What You Say | What Is Actually Happening |
|---|---|---|
| Confidence | "The cert is valid for a year, we'll deal with it later" | Later is a Saturday, and it is 2am, and it is now |
| Bargaining | "Can we just make the cert last 10 years?" | Browsers cap trust at ~13 months and reject it outright |
| Denial | "It works in Postman" | Postman has verification off because you turned it off |
| Anger | "The customers' clock is wrong" | It is, actually, but that's also your problem now |
| Acceptance | "We bought a certificate management product" | Nobody installed the agent on `util-01` |

And the table I used to defend each decision to leadership, who have never once asked a follow-up question:

| Approach | Why It's Bad | Why Mine Was Worse |
|---|---|---|
| ACME with twice-daily renewal + deploy hook | Requires reading one page of docs | N/A, I did not do this |
| Calendar reminder 30 days out | Depends on a human reading calendars | I set it to 7 days out and declined it |
| One-year paid cert from a vendor | Manual, expensive, annual panic | I let it auto-renew and never installed the new one |
| Expiry monitoring with an alert | Someone must own the alert | I owned it. I muted it. |
| `curl -k` everywhere in CI | Disables the entire point of TLS | This shipped to production in a healthcheck |
| Wildcard cert on every host | One compromise is total compromise | I put the private key in the Docker image |

The `curl -k` one deserves its own memorial. A healthcheck with verification disabled will pass against an expired certificate, a self-signed certificate, a certificate for an entirely different company, and a man-in-the-middle. We had green dashboards throughout a four-hour outage. The dashboard was not lying. It was answering a question I had carefully rewritten to be meaningless.

## Things That Expire That You Are Not Watching

This is the part that still visits me at 4am. All of the following were simultaneously true in my estate:

- The **leaf certificate** for the public API: monitored. Fine. The one thing we watched.
- The **intermediate CA**: not monitored. Expired. Half our mobile users got a trust failure and the other half didn't, based entirely on when their phone last refreshed its trust store. The bug report said "login broken for some people," which routed to the frontend team for nine days.
- The **internal CA root**, created in 2016 with a 10-year validity by a contractor, with the private key on a USB stick in a drawer labelled "MISC CABLES."
- The **client certificates** for mutual TLS between two services, generated by a one-off script that nobody saved, with a 1-year validity and no renewal path. The renewal path was eventually "recreate the whole PKI during the incident."
- The **code signing certificate**, which expired quietly and broke releases, not production — so it took three weeks to notice and was filed as "the build is flaky."
- The **JWT signing key** we rotated, forgetting that one consumer had hardcoded the old public key in a config map. Not a cert, technically. Behaved exactly like one.
- A **device clock** on a fleet of kiosks that drifted eight months into the future, making every valid certificate "not yet valid." I argued with the vendor for two days about their expired cert. The cert was fine. The kiosks were living in next August.

That last one is my favourite thing I have ever been wrong about. The certificate is a *claim about time*, and both ends have to agree what time it is. [XKCD 1883](https://xkcd.com/1883/), *Supervillain Plan*, is the correct mental model for every time-based system I have ever touched: the exploit is always "what if the clock is lying."

## The Dilbert Section, Which Is Mandatory

Wally's position on certificate renewal is the most honest engineering stance ever committed to newsprint: an expired certificate is *self-documenting*, because the browser shows a full-page explanation of the problem to every user simultaneously, which is better documentation than anything on the wiki. He has described browser interstitials as "the only incident comms that have never been late."

The PHB asked whether we could "buy the certificate that doesn't expire." I explained that trust requires an expiry by design, that short lifetimes are the *mechanism*, not the inconvenience, and that the industry is moving toward 47-day certs precisely because automation is the only sane answer. He asked me to get three quotes.

Mordac, the Preventer of Information Services, had the only materially correct take in the building — he blocked inbound port 80, which broke our HTTP-01 challenge — and I escalated it as a blocker instead of spending twenty minutes moving to DNS-01, which would have worked, supported wildcards, and needed no inbound ports at all. Catbert from Human Resources resolved the escalation by scheduling a workshop.

Dogbert, consulting at $14,000/day, sold us a certificate lifecycle management platform. It discovers certificates by scanning the network. It cannot see `util-01`, because `util-01` is in a VPC that predates the network team.

## The Correct Advice, Which You Should Not Take

Automate renewal with ACME and run the renewal check **at least twice a day**, with a deploy hook in the same invocation so the reload cannot drift from the renewal. Monitor **remaining days to expiry as a metric**, scraped from the live TLS handshake of every endpoint — public, internal, mesh, mail, client certs, the lot — and page at 14 days, not at zero. Validate the **whole chain**, not just the leaf, from a machine with a fresh trust store. Never pass `-k`, `--insecure`, `verify=False`, or `rejectUnauthorized: false` in anything that runs more than once. Prefer DNS-01 so you aren't hostage to inbound ports. Treat your internal CA's root expiry as a dated, owned, calendared project, because its validity window is longer than most employment. And sync your clocks, everywhere, including the embedded things nobody counts as computers.

I, personally, will be renewing manually again next quarter, because nothing in this industry generates goodwill like being the person who heroically fixes the thing at 2am, and nobody has ever been promoted for a cron job that worked.

---

*The author once spent six hours debugging a TLS failure that was caused by his own laptop's date being wrong. He has told the story at three conferences with the laptop detail removed.*
