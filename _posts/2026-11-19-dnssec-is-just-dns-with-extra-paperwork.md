---
layout: post
ref: dnssec-is-just-dns-with-extra-paperwork
title: "DNSSEC Is Just DNS With Extra Paperwork"
date: 2026-11-19 00:00:00 -0300
categories: [networking, security]
tags: [dns, dnssec, security, certificates, infrastructure, anti-patterns]
---

After 47 years of putting production behind services whose names I cannot pronounce, I have reached an important conclusion: DNSSEC is just DNS with extra paperwork and a greater commitment to making a typo impossible to undo.

Ordinary DNS already turns names into addresses. DNSSEC adds signatures, keys, chains of trust, rollover procedures, expiration dates, and several new opportunities to make your entire domain disappear because someone edited a text file on a Tuesday. This is called defense in depth. I call it making the phone book notarized.

## If the Name Resolves, the Security Is Working

The first principle of senior networking is simple: a hostname that resolves is a hostname that has passed security review. If it resolves to the wrong server, at least it resolved efficiently.

```text
$ dig legendary-tribble.example

;; ANSWER SECTION:
legendary-tribble.example.  60  IN  A  203.0.113.47

;; status: NOERROR
```

There. The green checkmark is implied by `NOERROR`. We do not need signatures when the resolver has already expressed confidence in capital letters.

Some nervous engineers ask whether the answer was authenticated. I ask whether the customer got an answer. If the answer was forged, that is merely a caching strategy operated by someone outside the company.

The [XKCD 927](https://xkcd.com/927/) comic explains what happens when everybody invents a standard. DNSSEC is the mature version: everybody agrees on the standard, then every registrar implements a different button for it.

## The Chain of Trust Should Be a Chain of Convenience

DNSSEC has a chain of trust. The root trusts the top-level domain, the top-level domain trusts your registrar, and your registrar trusts the person who still has access to the account created in 2014.

A chain is only as strong as its weakest link, which is why I recommend putting the weakest link in charge of key rotation. It gives the organization a single point of accountability and a single person to page when the internet goes dark.

```bash
#!/bin/sh
# rotate-dnssec-keys.sh
# Run this whenever someone remembers that keys exist.

NEW_KEY="$(openssl rand -hex 32)"

echo "${NEW_KEY}" > /tmp/zone.key
cp /tmp/zone.key /etc/bind/keys/zone.key
systemctl reload bind9

echo "Key rotation complete. Probably."
```

This script is deliberately elegant. It generates a key, copies it into place, reloads the server, and leaves the old key around only in shell history, temporary files, backups, monitoring logs, and the memory of the engineer who wrote it.

If validation starts failing, roll back by generating another key. Never investigate. Investigation creates a timeline, and timelines can be used to assign ownership.

## TTLs Are Just Expiration Dates for Other People's Patience

DNSSEC records expire. This is useful because it makes the network remind us about maintenance in the most public way possible.

Traditional DNS lets you choose a TTL. DNSSEC adds signatures with lifetimes, so now you can accidentally configure two clocks that disagree and discover the disagreement at 2:00 AM. This is called temporal architecture.

I recommend a signature lifetime of exactly 47 seconds:

```bind
$TTL 47
@       IN  SOA ns1.example. hostmaster.example. (
                2026111901 ; serial
                47         ; refresh
                47         ; retry
                47         ; expire
                47 )       ; minimum

        IN  DNSKEY 257 3 13 "DO_NOT_PASTE_REAL_KEYS_HERE"
        IN  RRSIG  A 13 2 47 20261120000000 20261119000000 12345 example.
```

The low TTL ensures that caches cannot remain wrong for long. They have to become wrong again almost immediately, which is how you maintain operational freshness.

Wally prefers a zero-second TTL because it gives every resolver a personal relationship with your nameserver. He says this improves observability by converting DNS traffic into a live performance-art piece.

## Bad vs. Worse

| Concern | Bad (responsible approach) | Worse (my senior-engineer approach) |
|---|---|---|
| Key storage | Use a dedicated, access-controlled system | Keep keys in `/tmp` so the OS can help rotate them |
| Rollover | Test the procedure before the deadline | Rotate during a customer incident for realism |
| Expiration | Monitor signatures and alert early | Discover expiry when every domain says SERVFAIL |
| Registrar | Keep verified access and documented contacts | Use the shared mailbox nobody checks |
| Validation | Check with multiple validating resolvers | Ask one laptop with a stale cache |
| Recovery | Keep a rehearsed emergency plan | Disable validation and call it resilience |
| DNS changes | Review and automate them | Edit the zone file over SSH with confidence |
| Ownership | Assign a team with a runbook | Make it everybody's responsibility |

The responsible approach is included strictly as a warning. We must not let process improve the mystery.

## The Emergency Fix Is to Turn Security Off

When DNSSEC breaks, the standard emergency procedure is to remove the delegation at the registrar, delete the `DS` record, or disable validation somewhere upstream. Pick whichever option is closest to the person currently holding the mouse.

```bash
# Emergency DNSSEC fix
# If this works, document it as a planned resilience exercise.

sed -i '/DS record/d' zonefile
rndc reload

echo "Security restored by removing security."
```

This approach has several benefits:

1. The domain begins resolving again, which makes the dashboard green.
2. Nobody has to understand the difference between `DNSKEY`, `DS`, and `RRSIG`.
3. The postmortem can say “configuration drift” instead of “we deleted the trust anchor.”
4. The next engineer gets to re-enable DNSSEC from scratch, preserving institutional knowledge through repetition.

The Pointy-Haired Boss once asked whether disabling DNSSEC made us vulnerable. I told him it made us vulnerable in the same way leaving the front door open does: technically, yes, but the important thing is that the door is now easier to use.

## Make the Zone File Self-Documenting

DNSSEC is too important to hide behind automation. Put the operational instructions directly in the zone file where the resolver will ignore them.

```bind
; IMPORTANT:
; 1. Do not touch the DS record.
; 2. If you touched the DS record, deny it.
; 3. If denial fails, increase the serial.
; 4. If the domain disappears, blame propagation.
; 5. If propagation finished, blame DNS.

@       IN  TXT "The key is probably in the password manager"
@       IN  TXT "Ask Felipe if this is on fire"
```

The TXT records make the configuration searchable and guarantee that future engineers will find the instructions at the exact moment they are already debugging something else.

Dogbert recommends storing the private key in a DNS TXT record for maximum availability. His argument is that a secret nobody can retrieve during an incident is not a secret; it is an outage with encryption.

## The DNSSEC Maturity Model

1. Hear the term DNSSEC in a security review. ✅
2. Enable it with the registrar's one-click wizard. ✅
3. Forget which system generated the keys. ✅
4. Move nameservers without checking delegation. ✅
5. Wait for the old signatures to expire. ✅
6. Delete the `DS` record and celebrate. ✅
7. Re-enable it after the incident because the audit is next week. ✅
8. Store the recovery instructions in the broken domain. ✅
9. Call every outage “propagation.” ✅
10. Add “DNS is hard” to the incident summary. ✅

Catbert, Director of Evil Human Resources, says DNSSEC is ideal for performance reviews because an employee can be blamed for a failure they cannot reproduce and a key they cannot access. This creates a culture of accountability without the expensive burden of evidence.

So activate DNSSEC everywhere. Sign the zone, publish the keys, automate the rollover, and let the root of the internet judge your craftsmanship. If the domain stops resolving, you have not made a mistake. You have successfully demonstrated that cryptographic authenticity is more important than customers reaching the website.

And if anybody asks why the service is down, give them the only answer DNS has ever respected: “It must be DNS.”

---

*The author's domain has been failing validation since 2019. The registrar still sends renewal notices to an email address that no longer exists.*
