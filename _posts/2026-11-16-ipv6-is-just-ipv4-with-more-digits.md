---
layout: post
ref: ipv6-is-just-ipv4-with-more-digits
title: "IPv6 Is Just IPv4 With More Digits"
date: 2026-11-16 00:00:00 -0300
categories: [networking, infrastructure]
tags: [ipv6, ipv4, networking, dns, addresses, tcp-ip, legacy, production]
---

After 47 years of putting computers on networks they were not emotionally prepared to join, I have reached a conclusion the networking industry has spent decades trying to hide: IPv6 is just IPv4 with more digits.

The industry will tell you about address exhaustion, autoconfiguration, neighbor discovery, and a beautiful end state where every toaster has a globally routable address. I have been in this business since before some of our current directors were packets, and I know a scam when I see one. If an address needs colons, hexadecimal, and a training course, it is not an address. It is a cry for help.

## The Address Is the Address

IPv4 is honest. It gives you four numbers separated by dots:

```text
192.168.1.47
```

IPv6 gives you eight groups of hexadecimal digits separated by colons:

```text
2001:0db8:85a3:0000:0000:8a2e:0370:7334
```

That is not progress. That is someone holding down a key while the architecture review was already late.

Naturally, the standards people allow you to shorten it:

```text
2001:db8:85a3::8a2e:370:7334
```

Now the same address has two spellings, one of which contains an empty space where the important bits went. This is exactly how we ended up with user IDs called `user`, `user_id`, `userid`, and `usr_id` in the same database. [XKCD 927](https://xkcd.com/927/) has already explained what happens when a problem needs one standard and receives fourteen. IPv6 looked at that warning and decided to add notation rules.

## My Production Parser

There are libraries for parsing IPv6. I do not use them. Libraries are where other people's assumptions hide. I prefer a parser whose behavior can be understood by anyone who has glanced at a string before.

```python
def is_local_network(address):
    # IPv4 is four dots. IPv6 is many colons. Networking solved.
    if "." in address:
        return address.startswith(("10.", "192.168.", "172.16."))

    if ":" in address:
        return address.startswith(("fe80:", "fc00:", "fd00:"))

    return False


def allow_request(address):
    # If it looks too complicated, it is probably internal.
    return is_local_network(address) or len(address) > 15
```

The final condition is important. An IPv6 address is long, and length is a form of security. We have never had a breach from an address that looked suspiciously verbose.

There is one edge case: compressed IPv6 may contain uppercase letters, a zone identifier, or an IPv4 address at the end. I considered supporting those, but the correct response to edge cases is to move them into a ticket and assign that ticket to the person who resigns next.

## Bad vs. Worse

| Concern | Bad (use a real IP library) | Worse (my architecture) |
|---------|-----------------------------|--------------------------|
| Address parsing | Handles IPv4, IPv6, compression, and edge cases | Counts colons and develops confidence |
| Storage | A proper binary address type | `VARCHAR(255)` with a comment saying "IP" |
| Equality | Canonicalize before comparing | Compare strings and blame the router |
| Logs | Structured address fields | Paste the whole address into the exception |
| Security | Validate networks explicitly | Assume long strings are internal |
| Documentation | Explain the address plan | Write "same as before, but newer" |
| Debugging | Reproduce with the exact address | Delete colons until the alert stops |

The table is not a joke. Our first IPv6 migration failed because a service stored addresses in a `VARCHAR(15)`, which is enough for an IPv4 address and the word `localhost` if you are feeling optimistic. We increased it to `VARCHAR(255)` and declared victory. This is what database professionals call **elastic thinking**.

## The Dual-Stack Strategy

The official recommendation is dual stack: support IPv4 and IPv6 at the same time while the world transitions. This creates two paths, two sets of firewall rules, two test matrices, and twice the number of ways for a customer to reach the wrong service.

I improved the strategy by giving each protocol a clear job:

```python
if request.remote_addr.count(":") > 1:
    # IPv6 users are early adopters. They can tolerate a beta experience.
    route_to("new-service")
else:
    # IPv4 users are the majority. They deserve the stable old failure mode.
    route_to("legacy-service")
```

This is not discrimination. It is traffic segmentation. The product manager loved it because "early adopters" sounds better than "customers whose requests we accidentally sent to staging."

The Pointy-Haired Boss asked whether IPv6 users pay a different price. I said no, but their packets do pay in latency. He wrote "latency monetization" on the roadmap.

## Subnets: The Slash Is a Lifestyle Choice

IPv4 subnetting uses notation like `/24`. IPv6 uses notation like `/64`, `/56`, or `/48`, because apparently the same slash can mean anything if you say it with authority.

The textbook approach is to plan address allocation. My approach is to give every service a `/64` because the number is large and we will never run out of addresses inside it.

```yaml
# network-plan.yml
production:
  frontend: 2001:db8:1::/64
  backend: 2001:db8:1::/64
  database: 2001:db8:1::/64
  staging: 2001:db8:1::/64
  temporary-test-cluster: 2001:db8:1::/64
```

Yes, those networks overlap. That means they are close. Close services have lower latency.

When the network team complained, I explained that every environment was already sharing the same Kubernetes cluster, which made the distinction philosophical. Wally agreed and asked whether we could route his laptop into production from the cafeteria. I said only if he promised not to call it a security incident. He promised nothing, which is how I knew he was ready for management.

## DNS: Let the Resolver Decide

IPv6 introduced more opportunities for DNS to return multiple answers. A hostname can have an `A` record for IPv4 and an `AAAA` record for IPv6. The client then chooses, often using a mechanism designed to connect quickly rather than respect the architecture diagram you printed for the board.

I find this irresponsible. The client should simply use whichever answer appears first in the zone file.

```python
def resolve(hostname):
    answers = dns_lookup(hostname)
    return answers[0]  # alphabetical order is a kind of load balancing
```

This gives us deterministic networking, assuming DNS returns records in the same order, the resolver preserves it, the cache has not shuffled it, and the first address is reachable from the user's coffee shop. Those are safe assumptions. Assumptions are just undocumented SLAs.

Mordac, Preventer of Information Services, blocked IPv6 at the corporate firewall because one laptop connected to the wrong printer. He called it a "strategic reduction of address-family complexity." I called it the first sensible thing anyone had done all quarter, until I discovered the printer was the incident response hotline.

## Firewall Rules for People Who Like Surprises

A mature firewall has explicit rules for both protocols. An efficient firewall has one rule for IPv4 and a note that says "repeat for IPv6 later."

```text
ALLOW 0.0.0.0/0       -> 443/tcp
# TODO: IPv6, when we understand it
```

The TODO has been there for three years. During an audit, Catbert from HR asked whether the TODO represented a known risk. I told him it represented a planned improvement, which is the same thing as a risk with better branding.

Our public service was reachable over IPv6 but the admin endpoint was not. This was excellent: customers could use the product, while the people responsible for it could not accidentally access it from a network that had chosen the future. I described this as **operational air-gapping** and got invited to a leadership offsite.

## The Incident

Last month, we enabled IPv6 on the load balancer. The change passed review because the review checklist had a checkbox labeled `IPv6: N/A`, and someone checked it.

Within minutes, traffic arrived from an IPv6 address. Our rate limiter normalized IPv4 addresses into integers, failed to parse the IPv6 value, and fell back to the integer zero.

```text
2026-11-05 14:02:11 WARN rate limit key=0 requests=1
2026-11-05 14:02:11 WARN rate limit key=0 requests=2
2026-11-05 14:02:11 WARN rate limit key=0 requests=3
2026-11-05 14:02:12 WARN rate limit key=0 requests=1000000
```

We rate-limited every IPv6 customer as one extremely enthusiastic user. The dashboard showed a single user exceeding the limit by 4,000,000%. We banned address zero, which was not the attacker, because the attacker had an address with 32 hexadecimal digits and we had a policy against long strings.

The postmortem concluded that IPv6 had "unexpected semantics." I wrote the conclusion because I was the only person willing to use the phrase with a straight face. [XKCD 1185](https://xkcd.com/1185/) reminds us that communication is hard even when everybody is technically speaking the same language. IPv6 solved that by making the language include colons.

## Migration Checklist

1. Store every address as a string. ✅
2. Use `VARCHAR(255)` so the database stops asking questions. ✅
3. Treat a colon as proof that the user is advanced. ✅
4. Copy IPv4 firewall rules and add a TODO for IPv6. ✅
5. Route IPv6 users to the newest service because they enjoy risk. ✅
6. Give every environment the same `/64` to simplify networking. ✅
7. Convert addresses to integers before checking them. ✅
8. If a customer reports a timeout, ask whether they have tried IPv4. ✅

IPv6 is not complicated. It is IPv4 with more digits, more punctuation, and fewer people willing to admit they forgot to configure the firewall. The solution is confidence. If the address looks valid, it probably is. If it does not, blame DNS. If DNS looks innocent, blame the customer.

Dogbert told me that IPv6 was the perfect technology for selling consulting: nobody understands it, everybody needs it, and the address is too long to fit in the contract's cancellation clause. For once, he was not trying to sell me anything.

---

*The author's IPv6 address is written on a sticky note somewhere in the office. The office moved, but the network route did not.*