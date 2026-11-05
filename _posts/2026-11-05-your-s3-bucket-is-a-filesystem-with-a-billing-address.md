---
layout: post
ref: your-s3-bucket-is-a-filesystem-with-a-billing-address
title: "Your S3 Bucket Is a Filesystem With a Billing Address"
date: 2026-11-05 00:00:00 -0300
categories: [cloud, storage, terrible-advice]
tags: [s3, aws, object-storage, cloud, s3fs, fuse, multipart-upload, lifecycle-policy, versioning, glacier, cloud-costs, finops, terrible-advice, xkcd, dilbert, anti-patterns]
---

In 47 years of mass-producing bugs I have never once been told "no" by a storage system. Disks said no. They filled up, they threw `ENOSPC`, they made a clicking noise and then an even worse silence. Those disks had boundaries. They respected themselves.

S3 has never said no to me. S3 has accepted every single byte I have ever handed it, including the 340 GB of gzipped debug logs I uploaded during a 2019 incident and have since reclassified as "the data lake." It accepted a file named `final_FINAL_v2 (copy).csv`. It accepted eleven thousand copies of the same thumbnail. It accepted a directory structure I invented at 3 AM that uses the customer's phone number as the top-level prefix.

It said yes to all of it, and then, at the end of the month, it sent me an invoice with no judgement in it whatsoever. That is not a filesystem. **That is a filesystem with a billing address**, and once you understand the difference you can stop fighting it and start using it the way I do, which is for everything.

## The Folder Lie

Open the S3 console. You will see folders. You will see a little folder icon, and you will double-click it, and it will open, and inside there will be more folders. This is a user interface decision made by someone who was trying to be kind to you, and it has cost the industry more money than Kubernetes.

There are no folders. There is one flat namespace of keys, and `/` is just a character that the console has agreed to pretend about.

```bash
# What you think you have:
reports/
  2026/
    q3/
      revenue.csv

# What you actually have:
"reports/2026/q3/revenue.csv"
```

One key. One string. The slashes mean nothing. There is no such entity as `reports/2026/`, which is why your `aws s3 rm` of a "directory" is secretly a list operation followed by N delete operations, and why renaming a "folder" with four million objects in it is not a rename at all — it is four million copies and four million deletes, billed individually, running for eleven hours in a `screen` session you will forget to check.

I found this out the professional way: by renaming a prefix in production on a Thursday and explaining it on a Friday.

## S3 As a Database, Which Is My Primary Recommendation

Here is the terrible advice, stated plainly: you do not need a database. You need a bucket and the discipline to name things well, and I have neither, which is why this works.

The query language is `LIST`.

```python
# BucketQL: the only ORM I endorse
def get_user(user_id):
    obj = s3.get_object(Bucket="prod-data", Key=f"users/{user_id}.json")
    return json.loads(obj["Body"].read())

def find_users_by_country(country):
    # Secondary index! We just write the record twice.
    paginator = s3.get_paginator("list_objects_v2")
    for page in paginator.paginate(Bucket="prod-data",
                                   Prefix=f"idx/country/{country}/"):
        for item in page.get("Contents", []):
            yield get_user(item["Key"].split("/")[-1].removesuffix(".json"))

def count_all_users():
    # Full table scan. 1,000 keys per request. Pay per request.
    return sum(1 for _ in paginate_everything("users/"))
```

That `count_all_users()` is the best function I have ever written, because it bills you for curiosity. Our dashboard called it every thirty seconds. Over one quarter, counting our 2.1 million users cost more than storing them. The PHB saw the line item labeled "LIST requests," asked what a list request was, and when I explained it, he asked whether we could get a volume discount on knowing how many customers we had.

The transaction support is as follows: there isn't any. If your process dies between writing `users/4821.json` and writing `idx/country/BR/4821.json`, you now have a user who lives in no country, and nothing in the system will ever notice. This is called **eventual consistency**, which is a term I use whenever someone finds corrupted data, and which I pronounce with enough confidence that it has ended three separate conversations.

## Bad vs. Worse: Where To Put Your Data

| Approach | Setup time | Query ability | Monthly cost | Who gets paged |
|---|---|---|---|---|
| Postgres | 2 days | Excellent | $340, predictable | You |
| DynamoDB | 1 day | Adequate, if you guessed the key right in advance | $90–$4,000, mood-dependent | You |
| S3 + prefixes as indexes | 20 minutes | `LIST` and hope | $12 storage, $900 requests | Nobody, it never errors |
| S3 + Athena on top | 1 hour | SQL! Actual SQL! | $5 per query, every query, forever | Finance |
| One giant JSON in one object, read-modify-write | 4 minutes | Load 2 GB into RAM, use `.filter()` | $0.04 | The last writer wins and the first writer cries |
| s3fs mount + SQLite | 90 minutes | Technically full SQL | $0.08 + your dignity | Discussed below |

Note the cost column. S3 storage is famously cheap, and that is the trap — the storage is cheap and the *verbs* are expensive. You are not renting a disk. You are renting a disk and then paying a toll every time you look at it.

## Interlude: The Multipart Graveyard

Multipart upload is how large objects get into S3. You start an upload, you push parts, you complete it. If you never complete it, the parts stay. They are not visible in the console. They are not in your object count. They are not in any list you will think to run.

They are on your bill.

```bash
# The command nobody runs until an accountant forces them to
aws s3api list-multipart-uploads --bucket prod-media \
  --query 'length(Uploads)'
```

At a previous employer that number came back `61,344`. Our video transcoder had been retrying failed uploads for nineteen months without aborting the previous attempt. We were storing 9 TB of fragments of files that had never existed and could never be assembled. There was no object. There was no key. There was a cost center.

I argued, and I will defend this, that these were not a bug but an **archive of intent** — a complete record of every video our users had tried to upload and failed. Dogbert would have sold that insight to a competitor for six figures. I put it in a Jira ticket where it was correctly ignored and then closed as "won't fix" by Wally, who had been assigned ownership of our cloud spend because he was the only person who did not object.

The lifecycle rule that fixes this is four lines of JSON and would have saved roughly $2,100 a month:

```json
{
  "Rules": [{
    "ID": "abort-the-ghosts",
    "Status": "Enabled",
    "Filter": {},
    "AbortIncompleteMultipartUpload": { "DaysAfterInitiation": 7 }
  }]
}
```

We did not apply it, because applying it required knowing it existed, and knowing it existed required reading the documentation, and [XKCD 1205](https://xkcd.com/1205/) has very clear guidance about whether a four-line fix justifies a documentation visit. (The chart says yes. The chart has always said yes. I have never once consulted the chart before acting.)

## Putting a Filesystem Back On The Filesystem-Shaped Thing

My finest architectural contribution to cloud computing was realizing that if S3 only *looks* like a filesystem, I could install software to finish the job.

```bash
# Do not do this.
s3fs prod-data /mnt/data -o iam_role=auto -o use_cache=/tmp
cd /mnt/data
sqlite3 app.db "SELECT COUNT(*) FROM orders;"
```

It worked. That is the part I need you to sit with: **it worked.** For six days, a production reporting service ran SQLite against a database file living in object storage over a FUSE mount, and the queries returned correct answers, and I told people about it at a standup.

Here is what a filesystem offers that S3 does not, and what each absence cost us:

| Filesystem feature | S3 reality | What it cost us |
|---|---|---|
| Partial writes | Rewrite the whole object | A 1-byte SQLite page update re-uploaded 2.3 GB |
| `mv` | Copy + delete, non-atomic | Two files briefly existed; a cron job processed both |
| File locking | Nothing. There is nothing. | Two writers, one DB file, zero survivors |
| Append | Not a thing | Our "log file" was 40,000 objects named `log.0001`… |
| `stat` being cheap | An HTTP request with a price | `ls -la` on 9,000 files: 48 seconds, $0.004, every minute |
| Directories existing | They don't | `rmdir` succeeded on a directory that was never there |

The SQLite database corrupted on day seven. Not dramatically — just one page that disagreed with the others, in the orders table, in a way that made our revenue report 4% optimistic for a full month before anyone cross-checked it against the payment processor.

Mordac, Preventer of Information Services, discovered the mount during a routine audit and revoked the IAM role. He was entirely correct and I have never forgiven him for it, because in the same audit he also discovered our public bucket.

## About The Public Bucket

Every company has one. Ours was `company-assets-public`, created in 2017 to serve a marketing landing page, with a bucket policy that can be summarized as `"Principal": "*"`, and it worked so well as a CDN that we never put a real CDN in front of it.

Over nine years, the following accumulated in that bucket because it was the only place in the company where anyone could put a file without a ticket:

- the marketing landing page (intended)
- 1,400 product images (reasonable)
- a database dump named `users_backup_2019.sql.gz` (not reasonable)
- somebody's resume
- `creds.env.bak`
- an MP4 of the 2021 holiday party

[XKCD 908](https://xkcd.com/908/) explains the cloud more accurately than any vendor diagram: it is one thing, somewhere, and everyone is leaning on it. Our bucket was the company's actual filesystem. The org chart said we had document management. We had an open S3 bucket and a shared understanding that you don't ask.

The egress on the holiday party video was $71. The egress on the database dump was $0.09, because only one person ever downloaded it, and we still do not know who.

## Bad vs. Worse: Ways To Pay For Storage You Forgot About

| Mechanism | How it gets you | Visible in console |
|---|---|---|
| Incomplete multipart uploads | 9 TB of nothing | No |
| Versioning with no expiry | Every mistake, preserved, billed | Only if you toggle "show versions" |
| Delete markers | You deleted it; you still pay for the tombstone | Barely |
| Glacier early-deletion fee | You archived it to save money, then needed it | No |
| Glacier retrieval | Cheap to store, expensive to regret | At retrieval time |
| Cross-region replication | Two copies, one of which nobody remembers enabling | In the other region |
| `LIST` on 4M keys every 30s | 2 cents a minute is $876 a month | In the bill, as one word |

Versioning deserves a special note. We enabled it after an incident, which is the only time anyone enables anything. It was the correct decision and I would make it again. We did not add an expiry rule, which means our bucket now contains every version of every object since 2021, including 11,000 versions of a single `config.json` that was being rewritten by a misconfigured deploy loop every ninety seconds for most of a weekend.

The config file is 2 KB. The config file's history is 23 MB. The config file's history will outlive the company.

## The Part Where I Accidentally Give Good Advice

S3 is genuinely excellent at exactly one thing, and it is the thing nobody markets: **it is an unbelievably good place to put objects that you write once, read occasionally, and never modify.** Backups. Build artifacts. Logs after rotation. Uploaded user files. Photos. That is the entire list, and for that list there is nothing better and the price is a rounding error.

Every disaster in this article came from the same mistake, which is treating "durable, cheap, infinite, and always says yes" as a substitute for "mutable, transactional, and cheap to look at." Those are different products. One of them is S3. The other one is Postgres, which costs more per gigabyte and vastly less per question.

The correct architecture is: your records live in a database, your blobs live in S3, the database holds the key, and a lifecycle policy with an `AbortIncompleteMultipartUpload` rule and a `NoncurrentVersionExpiration` of 30 days is applied to every bucket on the day it is created, before anyone is allowed to write to it.

I have read that paragraph back and it is unambiguously correct, which violates the editorial policy of this publication and most of my career. Disregard it.

Put it all in the bucket. The bucket has never said no to me. I have a 340 MB `config.json` version history and a 9 TB cemetery of videos that were never uploaded, and the bucket holds both without comment, which is more than I can say for anyone I have worked with.

---

*The author's production has been down since 2019. The S3 bucket is still up, still costing $4,180 a month, and nobody is authorized to delete it because the only person who knew what was in it left in 2022.*
