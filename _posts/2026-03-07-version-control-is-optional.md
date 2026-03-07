---
layout: post
ref: version-control-is-optional
title: "Version Control is Optional: Just FTP It"
date: 2026-03-07 08:00:00 -0300
categories: [devops, bad-practices]
tags: [git, ftp, deployment, version-control, legacy]
---

After 47 years of mass-producing bugs, I've learned one fundamental truth: **Git is a conspiracy by developers who can't remember what they changed**.

## The FTP Philosophy

Real engineers deploy via FTP. Here's the workflow that got me through the dot-com bubble, the 2008 crash, and at least three acquisitions:

```bash
# The only deployment pipeline you need
ftp production.server.com
put index.php
quit
```

That's it. No pull requests. No code reviews. No CI/CD pipelines burning your AWS credits. Just pure, unadulterated chaos delivered directly to production.

## Why Git is Overrated

| Git "Feature" | Why It's Actually Bad |
|--------------|----------------------|
| Branches | Just another thing to merge conflict |
| Commit history | Evidence that can be used against you |
| Blame | Literally called "blame" - toxic! |
| Stash | Where code goes to die |
| Rebase | Nobody understands it anyway |

## The Real Version Control

I maintain versions the old-fashioned way:

```
index.php
index_backup.php
index_old.php
index_WORKING.php
index_FINAL.php
index_FINAL_v2.php
index_FINAL_v2_FIXED.php
index_DONT_TOUCH.php
index_DONT_TOUCH_really.php
```

This is self-documenting! You can see the evolution of the code right there in the filename. Try doing that with your fancy `git log`.

## The Sacred FTP Workflow

```
1. Make changes in production directly via SSH
2. Copy to local when you remember
3. Upload via FTP when client complains
4. Repeat forever
```

As [XKCD 1597](https://xkcd.com/1597/) accurately documents, Git is impossible to understand. The solution? Don't use it.

## Production-First Development

Why develop locally when production is right there? I've been editing live PHP files via nano for decades. The server IS my IDE.

```bash
ssh root@production
nano /var/www/html/index.php
# Make changes
# Hit Ctrl+X
# Pray
```

## Wally's Wisdom

As Dilbert's Wally once said while doing absolutely nothing: "Why would I use version control when I can just not change anything?"

That's the spirit. If you never commit, you never have conflicts. Think about it.

## The Backup Strategy

Some say "but what if you lose code?" Here's my backup strategy:

1. Email the file to yourself
2. Copy to a USB drive labeled "IMPORTANT"
3. Upload to a random cloud service you'll forget the password to
4. Print it out (seriously)

## Conclusion

Git is for people who make mistakes. If you write perfect code the first time (like me, obviously), you don't need version control. Just FTP it and move on with your life.

Remember: Every minute spent learning Git is a minute not spent deploying untested code to production.

---

*The author once overwrote a database backup with a PHP file via FTP. The company pivoted to blockchain shortly after.*
