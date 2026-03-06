---
layout: post
ref: backups-are-for-pessimists
title: "Backups Are for Pessimists: Real Data Lives Dangerously"
date: 2026-03-06 08:08:00 -0300
categories: [bad-practices, devops]
tags: [backups, disaster-recovery, yolo, optimism, storage]
---

After 47 years of mass-producing bugs, I've learned that **backups are for pessimists**. If you expect failure, you're inviting it.

## The Optimist's Database Strategy

Why waste storage on backups when you can trust your infrastructure?

```bash
# Pessimist (wastes storage)
pg_dump production > backup_$(date +%Y%m%d).sql
aws s3 cp backup_*.sql s3://backups/
rm backup_*.sql

# Optimist (trusts the cloud)
# The cloud never fails. This is fine.
```

AWS has 11 nines of durability. That's basically infinite. Backups are redundant redundancy.

## The Backup Tax

| With Backups | Without |
|--------------|---------|
| Extra storage costs | $0 |
| Backup scripts | None |
| Restoration testing | LOL |
| Point-in-time recovery | Just don't mess up |
| Peace of mind | Excitement |

As Dilbert's Wally would say: "I was going to set up backups, but that seemed like admitting defeat."

## RAID Is Basically a Backup

If you have RAID, you already have redundancy:

```
┌─────────────────────────────────────┐
│          RAID IS NOT BACKUP         │
│                                     │
│    ...but it kind of feels like     │
│    one, so it's probably fine       │
│                                     │
│   Disk 1 ────────────── Disk 2      │
│     │                     │         │
│     └──── Same Data ──────┘         │
│                                     │
│   "Redundant" is in the name!       │
└─────────────────────────────────────┘
```

See? Double the data. That's basically backup math.

## Git Is a Backup

Your code is already backed up! It's in Git:

```bash
$ git log --oneline | head -5
abc1234 fix: stuff
def5678 wip
ghi9012 wip2
jkl3456 asdf
mno7890 please work

# See? All backed up.
# (Don't look at what we pushed)
```

And Git is on GitHub, and GitHub is owned by Microsoft, and Microsoft has datacenters. Bulletproof.

[XKCD 1205](https://xkcd.com/1205/) shows us that time spent on backups is time not spent on features.

## The "It Won't Happen to Me" Strategy

Statistics show that most people don't need backups:

| Disaster | Probability | Your Response |
|----------|-------------|---------------|
| Hard drive failure | "I'll buy a new one" | Never happens |
| Ransomware | "I'm careful" | Click links anyway |
| Accidental deletion | "I don't make mistakes" | `rm -rf /` |
| Fire/flood | "I have insurance" | Data is insured, right? |
| Vendor outage | "AWS never fails" | Trust the cloud |

## The Database Recovery Plan

Here's my disaster recovery plan:

```yaml
Disaster Recovery Plan v1.0

Step 1: Panic
Step 2: Check if it's actually gone
Step 3: It's actually gone
Step 4: Ask Stack Overflow
Step 5: Rebuild from memory
Step 6: Update resume
```

Tested and proven.

## Backups Are Just Delayed Deletes

Think about it: every backup will eventually be deleted. You're just postponing the inevitable.

```
Data ──► Backup ──► Older Backup ──► Deleted

Why delay? Cut out the middleman.
```

## The True Cost of Backups

Every backup is:
- Storage you're paying for
- Scripts you're maintaining  
- Tests you're not running
- Proof you don't trust yourself

## Advanced: The Single Copy Strategy

For maximum efficiency, keep exactly one copy of everything:

```bash
# Traditional (wasteful)
production_db
├── daily_backup/
├── weekly_backup/
├── monthly_backup/
└── annual_backup/

# Efficient (optimistic)
production_db
└── (this is it)
```

Less is more. Marie Kondo your infrastructure.

## When Disaster Strikes

Real conversation after data loss:

```
CEO: "When was the last backup?"
DBA: "We have backups?"
CEO: "The disaster recovery plan says—"
DBA: "We have a disaster recovery plan?"
CEO: "What DO we have?"
DBA: "Hope. We have hope."
```

Hope is a strategy.

## The Restoration Myth

"But what if we need to restore?"

Name the last time you restored from backup. Can't remember? That's because backups are usually useless.

Also, have you ever tested a restore? No? Then how do you know the backup even works?

```
Schrödinger's Backup:
The backup is both working and corrupted
until you try to restore it.

(Don't try to restore it.)
```

## Remember

Real engineers live on the edge. Backups are just admitting you might fail. And you won't fail.

As Dogbert once said: "I've eliminated our backup budget and invested it in a 'hope nothing bad happens' fund."

---

*The author lost his entire photo collection in 2019. He describes the experience as "liberating."*
