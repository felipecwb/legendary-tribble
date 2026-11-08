---
layout: post
ref: your-etc-hosts-file-is-dns-for-people-with-trust-issues
title: "Your /etc/hosts File Is DNS For People With Trust Issues"
date: 2026-11-08 00:00:00 -0300
categories: [networking, operations, terrible-advice]
tags: [etc-hosts, dns, resolver, nsswitch, gethostbyname, getaddrinfo, hostaliases, add-host, split-horizon, ttl, localhost, 127-0-0-1, sre, terrible-advice, xkcd, dilbert, anti-patterns]
---

In 47 years of mass-producing bugs, I have never once resolved a problem by understanding DNS. I have resolved a great many problems by opening a text file and typing an IP address into it.

```
# /etc/hosts
127.0.0.1       localhost
::1             localhost
10.4.19.88      api.internal.corp
10.4.19.88      api
10.4.19.91      db-primary    # as of Tuesday
127.0.0.1       telemetry.vendor.example.com
127.0.0.1       analytics.vendor.example.com
34.200.14.7     payments.partner.example.com  # DO NOT REMOVE - ask Dave
```

That is the most powerful file on your machine. It outranks DNS. It outranks your DHCP-provided resolver. It outranks your company's $40,000/year managed DNS provider with four nines of SLA and a status page. It is a flat text file with no TTL, no authority, no versioning, no audit log, and no way for anybody else on Earth to know it exists.

**`/etc/hosts` is the only part of the network stack where your opinion beats reality.** I have built a career on that sentence.

## What It Actually Is, Which Is Older Than Most of Your Dependencies

`/etc/hosts` is not a DNS cache. It is not a DNS override. It is the thing that came *before* DNS, and it never left.

In 1983 the entire internet's name-to-address mapping was one file, `HOSTS.TXT`, maintained by hand at SRI-NIC. You FTP'd it down periodically. If you wanted a hostname on the internet, you emailed a human. DNS was invented specifically because that did not scale past a few hundred machines.

The file survived. Not as a fallback — as a *priority*. On a modern Linux box the order is decided by `/etc/nsswitch.conf`:

```
# /etc/nsswitch.conf
hosts: files dns myhostname
#      ^^^^^ this one wins, always, forever, silently
```

`files` means `/etc/hosts`. It is listed first on essentially every distribution shipped in the last 30 years. So when your application calls `getaddrinfo()`, the glibc resolver reads that text file *before it asks anyone anything*. No query leaves the machine. No cache is consulted. No TTL is evaluated. A line in a file wins against the combined authority of the global DNS hierarchy.

| What you think resolves names | What actually resolves names |
|---|---|
| DNS | `/etc/nsswitch.conf`, which decides whether DNS gets asked at all |
| Your `resolv.conf` nameserver | `/etc/hosts`, first, unconditionally |
| `dig` output | Nothing. `dig` skips the resolver entirely and talks to the DNS server directly |
| A TTL you can wait out | A file edit that persists until the laptop is replaced |
| Something your team can see | Something only `root` on that one box has ever read |

That `dig` row is where most of your career's lost hours live, and we'll get to it.

## The Four Legitimate Uses, And The 400 Others

There are, genuinely, about four defensible reasons to touch this file:

1. `127.0.0.1 localhost` — load-bearing. Deleting this line breaks more software than you can imagine.
2. Bootstrapping a machine before DNS exists.
3. Testing a cutover against a new IP before changing public records.
4. Local development hostnames, where you want `app.localhost` and `api.localhost` to hit different vhosts.

Everything else is what I do, which is use it as a **permanent, undocumented, per-machine routing policy that outlives the employee who wrote it.**

## Terrible Pattern #1: The Hotfix That Became Infrastructure

The DNS record for `api.internal.corp` is wrong. It's pointing at the decommissioned load balancer. Fixing it requires a ticket to the network team, who have a two-week SLA and a change window on alternate Thursdays.

Or: you type one line into `/etc/hosts` on the four application servers and the problem is gone in nine seconds.

```bash
# the most consequential one-liner in operations
echo "10.4.19.88 api.internal.corp" | sudo tee -a /etc/hosts
```

That works. It works *immediately*. It works so well that nobody ever files the ticket, because the motivation to file the ticket was the outage, and the outage is over.

Now you have four servers resolving a corporate hostname from local text files, and one brand-new server — provisioned last month from the Terraform module — that doesn't. That server is the only honest machine in the fleet, and it is the one that pages you.

The bug report will read: *"intermittent 502s, approximately 20% of requests."* Twenty percent. One in five. Four good servers, one honest one. You will spend a day on the load balancer, a day on the connection pool, and then somebody will finally run `cat /etc/hosts` on each box and discover that your infrastructure is a democracy and the vote was 4–1.

[XKCD 1739](https://xkcd.com/1739/) — "Fixing Problems" — documents my methodology exactly. I fixed it. I don't know what I did. I'll never know what I did.

## Terrible Pattern #2: Debugging With `dig`, Which Does Not Use The Resolver

This one deserves its own monument.

Your application cannot reach `payments.partner.example.com`. You investigate like a professional:

```bash
$ dig +short payments.partner.example.com
203.0.113.45

$ curl -v https://payments.partner.example.com/health
* Connected to payments.partner.example.com (34.200.14.7) port 443
* SSL certificate problem: Hostname mismatch
```

Two different IPs. From the same machine. In the same second.

`dig`, `host`, and `nslookup` are **DNS diagnostic tools**. They construct a DNS packet and send it to a nameserver. They do not consult `/etc/nsswitch.conf`. They do not read `/etc/hosts`. They are showing you what DNS says, which is a completely different question from what your application will do.

`curl`, your app, your runtime, and every normal piece of software call `getaddrinfo()`, which goes through the full NSS stack and therefore reads the text file first.

```bash
# wrong tool: asks DNS
dig +short payments.partner.example.com

# right tool: asks the resolver, the way your app does
getent hosts payments.partner.example.com
# or
resolvectl query payments.partner.example.com
```

I have sat in a war room for three hours while five engineers took turns running `dig` and saying "DNS looks fine." DNS *was* fine. DNS was never involved. A contractor had added a line to `/etc/hosts` in 2023 to test a staging endpoint, the partner had since re-IP'd, and we were connecting to a stranger's server and failing TLS validation, correctly, as designed.

| Command | Reads `/etc/hosts`? | Shows what your app sees? |
|---|---|---|
| `dig` / `nslookup` / `host` | No | **No** |
| `getent hosts` | Yes | Yes |
| `resolvectl query` | Yes | Yes (systemd-resolved) |
| `ping` | Yes | Yes |
| `curl` / your application | Yes | Yes |
| `python -c "import socket; socket.gethostbyname(...)"` | Yes | Yes |

Print that table. Tape it to something. It is worth more than the last three conference talks you watched.

## Terrible Pattern #3: 127.0.0.1 As A Firewall

This is my favorite, because it feels like security.

```
# "blocking" telemetry
127.0.0.1   telemetry.vendor.example.com
127.0.0.1   ads.example.net
0.0.0.0     metrics.saas-we-pay-for.example.com
```

What you have built is not a block. You have redirected a hostname to your own machine. Here is what the application experiences: an immediate TCP connection attempt to a port on localhost where nothing is listening, which returns `ECONNREFUSED` instantly.

Which sounds great, except the client library was written by someone who assumed network failures are transient, so it retries. With backoff. For thirty seconds. On the request path. Synchronously. Your "block" converted a 40ms background telemetry call into a 30-second foreground stall, and because the connection is *refused* rather than *dropped*, you don't even get the merciful silence of a blackhole. (`0.0.0.0` behaves slightly differently and is marginally faster to fail, which is the entire technical depth of a surprising number of blocklist arguments online.)

Then the vendor moves to `t.vendor-cdn.example.com`, which isn't in your file, and the telemetry resumes, and you never notice, because you never had a metric for "telemetry successfully blocked." You had a feeling.

[XKCD 538](https://xkcd.com/538/) is the canonical rebuttal to all threat models of this shape. Your crypto doesn't matter if someone has a wrench; your hosts file doesn't matter if the vendor has a second domain and an SDK update.

Also: it doesn't scale. People ship blocklist hosts files with 150,000 entries. `/etc/hosts` lookups in glibc are a **linear scan of the file, parsed on every call**, with no indexing. A 150,000-line hosts file means every single name resolution on the box walks a multi-megabyte text file. I have watched a developer laptop take 400ms to resolve `localhost`. He thought his Docker install was broken. He had installed privacy.

## Terrible Pattern #4: Shadowing A Real Name With A Partial Match

```
10.4.19.91   db-primary
```

Beautiful. Short. Typeable. Also: `/etc/hosts` matching is **exact string matching on the full name as written**, and it interacts with `search` domains in `/etc/resolv.conf` in a way that nobody's mental model survives.

```
# /etc/resolv.conf
search internal.corp prod.internal.corp
```

Now `db-primary` might resolve from `/etc/hosts` as the literal string `db-primary`, while `db-primary.internal.corp` goes to DNS and gets a completely different machine. Two names for what everyone believes is one host, resolving through two different subsystems, to two different IPs, depending on whether the engineer who wrote that config string happened to type the FQDN.

Your connection string works. Your migration script works. Your monitoring — which uses the FQDN because the monitoring team has standards — reports the host as healthy. It is monitoring a different computer.

[XKCD 927](https://xkcd.com/927/) — "Standards" — except here we didn't even get a competing standard. We got one standard and a text file that quietly overrules it.

## Terrible Pattern #5: It Follows You Into Containers

You'd think containers would end this. They don't. They industrialize it.

```bash
docker run --add-host=api.internal.corp:10.4.19.88 myapp
```

```yaml
# docker-compose.yml
services:
  app:
    extra_hosts:
      - "api.internal.corp:10.4.19.88"
      - "host.docker.internal:host-gateway"
```

```yaml
# and the Kubernetes one, which is a war crime with a schema
spec:
  hostAliases:
    - ip: "10.4.19.88"
      hostnames:
        - "api.internal.corp"
```

`hostAliases` is a genuine, documented, first-class Kubernetes API field whose entire purpose is injecting lines into `/etc/hosts` inside your pods. Someone wrote a KEP for it. It went through review. We collectively built a declarative, version-controlled, GitOps-managed, Argo-synced pipeline whose output is appending a line to a 1983 text file.

And it's *worse* in Kubernetes, because:

- The pod's `/etc/hosts` is generated by the kubelet at startup, so editing it inside a running container is erased on restart and you will debug that twice before you learn.
- `hostAliases` lives in the pod spec, so it's invisible from any DNS tooling, any CoreDNS config, and any Service definition. It will never show up in `kubectl get svc`.
- It bypasses CoreDNS entirely, which means it also bypasses your DNS metrics, your NodeLocal DNSCache, and every bit of observability you built for name resolution.
- It is the only networking config in the cluster with no TTL, which in a system where everything is reconciled continuously means it's the one thing that is never reconciled against anything.

## The Story, Lightly Anonymized For Legal Reasons

We were migrating a payments integration to a partner's new endpoint. Standard cutover: they gave us a new IP, we'd verify against it, then they'd flip DNS.

To verify, I added a line to `/etc/hosts` on the staging box. Correct move. Textbook move. Possibly the only correct thing in this story.

The verification passed. The partner flipped DNS. I did not remove the line, because the line was now *the same as DNS*, so removing it had no observable effect, and I was busy. This is the exact reasoning that creates every piece of legacy infrastructure in the world: **it's harmless right now.**

Eleven months later, the partner migrated data centers. New IP, 30-day notice, three reminder emails, a banner on their status page. Every system we owned picked up the new record through DNS within the TTL. Exactly one did not. Staging.

Staging began failing TLS validation, because it was connecting to an IP that no longer belonged to the partner and the certificate — correctly — did not match. We diagnosed this as "the partner's new certificate is broken." We opened a ticket *with the partner*. We were, in writing, on company letterhead, wrong at a vendor.

Then QA, under deadline pressure for a release, "fixed" staging by copying its config to the integration test environment, which propagated the hosts entry to two more boxes. The bad line now had descendants.

Timeline:
- **Day 1:** TLS failures in staging. Blamed partner.
- **Day 3:** Partner replies, politely, with a `dig` output proving their DNS is correct. We run `dig` ourselves. It agrees with them. We conclude their edge is misconfigured regionally. We are now *inventing geography* to protect a wrong hypothesis.
- **Day 6:** Someone suggests `curl --resolve` to pin the IP manually. It works. This is treated as evidence of a CDN problem rather than as the enormous flashing clue it is.
- **Day 8:** A contractor who had never seen the system runs `getent hosts payments.partner.example.com`, gets the stale IP, and says "well, that's not what dig says."
- **Day 8, four minutes later:** `grep -rn payments /etc/hosts`. One line. My initials in the commit of the Ansible template that had, at some point, codified my manual edit into the base image. **I had automated my own mistake into the golden AMI.**

Eight days. Four engineers. One vendor escalation. One line I typed eleven months earlier for an entirely correct reason.

Wally's role was impeccable, as always. On Day 2 he said "it's probably hosts file." Nobody engaged, because Wally says "it's probably DNS" about everything — the printer, the coffee machine, a slow Jira board — and had therefore trained us perfectly to ignore a correct answer. He was vindicated on Day 8 and had expended no effort whatsoever. He now says "it's probably hosts file" about everything, and will be right again eventually, and will be ignored again, and this is the most stable system in our organization.

Mordac, Preventer of Information Services, responded by making `/etc/hosts` immutable (`chattr +i`) on all production hosts without telling anyone. This broke the provisioning script, which needed to write the node's own hostname at boot. New instances failed to join the cluster. He classified this as "working as intended," which — grudgingly — it was.

Catbert's contribution was a quarterly compliance control titled *"Unauthorized Name Resolution Overrides"* requiring each engineer to attest, personally, that no machine they administer contains undocumented hosts entries. Nobody can verify this. Everybody signs it. The control passed audit.

The PHB's postmortem action item was to "add DNS to the monitoring dashboard." We already monitor DNS. DNS was correct for all eight days. He has been told this three times. The dashboard now has a fourth DNS panel, all green, measuring a subsystem that was never involved.

## The Part Where I Accidentally Give Good Advice

`/etc/hosts` is not the problem. It's a precise, documented, extremely old mechanism that does exactly what it claims. The problem is that it's the only piece of your network configuration with **no expiry, no owner, and no visibility**, and we keep using it for things that need all three.

So, reluctantly:

- **Debug name resolution with `getent hosts` or `resolvectl query`, never `dig` alone.** `dig` answers a question your application never asks. If those two disagree, the difference *is* your bug. This single habit would have saved that payments team eight days.
- **Every hosts entry gets an inline comment with a date, an owner, and a reason.** `# 2026-11-08 felipe - partner cutover test, REMOVE AFTER 2026-11-15`. A line without an expiry date becomes permanent infrastructure, and you know this.
- **Never use it as a permanent fix for a DNS problem.** If the record is wrong, the record is wrong on every machine, including the ones that don't exist yet. Fix the record. The hosts entry buys you hours, not quarters.
- **Prefer `curl --resolve` or the client's own override for testing.** It's scoped to one command and vanishes when the command exits. The whole problem with `/etc/hosts` is that it has no scope and never exits.
- **In containers, use `hostAliases` / `extra_hosts` rather than editing the file in the image.** At least it's in the manifest, in git, where a human might one day read it. Editing it in a running container is erased on restart.
- **Put `/etc/hosts` in your config management, and have config management enforce it to a known-good state.** Any machine whose hosts file differs from the template should be an alert, not an archaeology project. Drift here is invisible by construction.
- **Don't use it as a blocklist.** You want a DNS sinkhole, a resolver policy, or a real firewall rule — something with logging, so you can tell the difference between "blocked" and "a feeling."
- **Keep `127.0.0.1 localhost` and `::1 localhost`.** Just leave them. Whatever you're doing, they're not the reason.

Reading that back: it's correct, it's cheap, it would have prevented every incident in this article, and it badly violates this publication's editorial standards.

So ignore it. Add the line. Don't comment it. Don't document it. Add it to the base image, so the fix propagates to machines that don't exist yet and will be inherited by engineers who have not yet been hired.

Then, when it fails in eleven months, run `dig`, confirm that DNS is fine, and escalate to the vendor. DNS will always be fine. DNS was never asked.

[XKCD 349](https://xkcd.com/349/) is how every one of these investigations begins, and the only part I've ever improved is that I no longer pretend to be surprised.

---

*The author's production has been down since 2019. The load balancer was decommissioned in 2021 but four application servers still reach it, because each of them has a line in `/etc/hosts` pointing at an IP that is now a parking lot in Virginia. The health check passes. The health check resolves through the same file.*
