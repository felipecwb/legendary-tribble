---
layout: post
ref: your-shell-history-is-the-only-documentation-that-tells-the-truth
title: "Your Shell History Is the Only Documentation That Tells the Truth"
date: 2026-11-04 00:00:00 -0300
categories: [documentation, shell, terrible-advice]
tags: [bash, zsh, shell-history, bash-history, runbooks, documentation, devops, sre, production, incident-response, ctrl-r, terrible-advice, xkcd, dilbert, anti-patterns]
---

In 47 years of mass-producing bugs I have written exactly one piece of documentation that was still accurate the day someone needed it, and I did not write it on purpose. It is `~/.bash_history` on a jump host that was decommissioned in 2021 and then un-decommissioned because nobody could remember how to restart the billing reconciler without it.

Every other form of documentation we tried was a lie with a timestamp. The wiki was a lie. The runbook was a lie with a flowchart. The README was a lie with a build badge that had been green since before the repository was private. But `history | grep` has never once lied to me, because it is not a description of what we *should* do. It is a **confession of what we actually did, in order, including the four attempts that failed.**

This is the only documentation strategy I endorse, and I would like to explain why your team should delete its runbook directory and give everyone shared access to one bash history file.

## The Fundamental Asymmetry

Documentation is written by someone who has just solved a problem and is therefore, at that exact moment, the person least qualified to describe it. They know too much. They have already forgotten the eleven minutes they spent in the wrong directory. They write:

```markdown
## Restarting the reconciler

1. SSH to the reconciler host.
2. Restart the service.
3. Verify the queue drains.
```

Three steps. Confident. Published. Linked from the on-call channel topic.

Here is the same procedure as recorded by `history`, which does not have an ego to protect:

```bash
 1021  ssh reconciler-01
 1022  ssh reconciler-01.prod
 1023  ssh reconciler-01.prod.internal
 1024  sudo systemctl restart reconciler
 1025  sudo systemctl restart reconciler.service
 1026  systemctl list-units | grep -i recon
 1027  sudo systemctl restart billing-recon@1
 1028  sudo systemctl status billing-recon@1
 1029  sudo journalctl -u billing-recon@1 -n 200 --no-pager
 1030  sudo -u billing /opt/billing/bin/recon --drain --yes
 1031  sudo -u billing /opt/billing/bin/recon --drain --force --yes
 1032  watch -n2 'redis-cli -n 3 llen recon:pending'
 1033  redis-cli -n 3 llen recon:pending
 1034  redis-cli -n 4 llen recon:pending
 1035  sudo systemctl restart billing-recon@2
```

That is the real procedure. The service is not called `reconciler`. There are two instances and you must restart the second one, which nobody has ever documented because nobody has ever understood it. The queue lives in Redis database 4, not 3. And `--drain` requires `--force` because of a safety check added in 2019 by someone who no longer works here.

The wiki page had three steps. Reality had fifteen, two of which were typos, and the typos are **load-bearing information** — they tell you the hostname convention is non-obvious and that you will get it wrong too.

## Bad vs. Worse: Documentation Formats Ranked

| Format | Accuracy at write time | Accuracy 6 months later | Contains failed attempts | Can be grepped at 3 AM |
|---|---|---|---|---|
| Confluence runbook | 70% | 0% | No | No (SSO expired) |
| README.md | 85% | 20% | No | Yes, if you cloned it |
| Architecture diagram | 40% | 40% (consistently wrong) | No | It's a PNG |
| Terraform | 100% of what exists | 100% | No | Only if state is unlocked |
| `~/.bash_history` | 100% | 100% | **Yes** | **Yes** |
| Slack thread from the last incident | 95% | Deleted by retention policy | Yes | Until day 91 |

Shell history wins every column that matters. It is the only artifact in the entire industry that is **append-only, written by the person doing the work, at the moment of doing it, with zero incentive to look competent.**

## The Ctrl+R Methodology

My entire operational practice is one keystroke. I do not read documentation. I press `Ctrl+R` and type a fragment of what I half-remember, and the past hands me back my own competence.

```bash
# What I type:
<Ctrl+R>kafka lag

# What the past returns:
kubectl -n streaming exec -it kafka-client-0 -- kafka-consumer-groups \
  --bootstrap-server kafka-0.kafka:9092 --describe --group billing-recon
```

I could not have reconstructed that command. Nobody could. [XKCD 1168](https://xkcd.com/1168/) is the definitive statement on this: nobody remembers `tar` flags, we remember that we once typed them successfully, and the history file is the only reason civilization continues. I have not known the correct `tar` invocation since 2004. I have *had* it since 2004. These are operationally identical.

The companion piece is [XKCD 979](https://xkcd.com/979/) — "Wisdom of the Ancients" — the one where the only other person with your problem posted "nevermind, got it" and never elaborated. That person had their history file. They did not need you. This is the natural state of knowledge transfer, and the mistake we keep making is trying to improve it.

## The Shared History File

Here is where my advice stops being merely bad and becomes actionable.

If one person's history is the best documentation in the company, then **everyone's history in one file is the best documentation in the industry.** We configured this at a previous employer and I maintain it was my finest architectural contribution:

```bash
# /etc/profile.d/99-institutional-memory.sh
# Do not do this.
export HISTFILE=/mnt/nfs/shared/institutional_memory
export HISTSIZE=-1
export HISTFILESIZE=-1
export HISTCONTROL=      # record duplicates, they indicate desperation
export PROMPT_COMMAND='history -a; history -c; history -r'
shopt -s histappend
```

Thirty-one engineers, one history file, on NFS, with no locking. We called it the Oracle. You pressed `Ctrl+R`, typed `migrate`, and received a command that someone else had run, possibly right now, possibly in a different datacenter, possibly against a database that no longer existed.

The Oracle was genuinely useful for about five weeks. Then it developed the following properties:

**It became non-deterministic.** With `history -c` in `PROMPT_COMMAND` and no file locking, two people pressing Enter within the same NFS write window would interleave. We accumulated commands that no human had typed:

```bash
kubectl delete pod -n staginsudo systemctl stop postgresql
```

That line is two people's intentions spliced at the byte level. Somebody eventually ran it. It failed, which was the luckiest outcome available.

**It became a credentials database.** `HISTCONTROL` was empty, so we recorded everything, including the `curl -H "Authorization: Bearer ..."` that someone pasted from Postman, and the `mysql -u root -pSummer2019!` that was still valid in 2023. Our security review found eleven live secrets in the Oracle. Mordac, Preventer of Information Services, had the file deleted within the hour. We restored it from backup because we needed the Kafka command.

**It became a liability with a search function.** Legal asked, during an unrelated dispute, whether the company retained records of production commands. We did. All of them. Including `psql -c "UPDATE invoices SET status='paid' WHERE id=88213"`, which was run during an incident by someone trying to help, and which is now, thanks to my architecture, permanently attributable to a Tuesday.

The PHB loved the Oracle. He asked whether we could put it on a dashboard. We did, briefly. He used it to count commands per engineer and praised Wally, who had the highest count by an enormous margin because his `PROMPT_COMMAND` was broken and re-appended his entire history on every prompt. Wally knew. Wally said nothing. Wally got a spot bonus.

## Why Runbooks Rot and History Doesn't

A runbook is a **prediction**. It predicts that the next incident will resemble the last one closely enough that prose written in the calm aftermath will apply during the panic. That prediction is wrong in a specific way: the calm aftermath is exactly when you remove the embarrassing parts, and the embarrassing parts are the procedure.

History is a **recording**. It cannot rot, because it never claimed to be current. When I grep my history for `drain` and find a command with a flag that no longer exists, the failure mode is a clean error message in under a second. When I follow a runbook whose step 2 is subtly stale, the failure mode is a forty-minute detour with a confident guide.

```bash
# Runbook says:
./deploy.sh --env prod --confirm

# History says:
./deploy.sh --env prod --confirm
./deploy.sh --env prod --confirm --skip-migrations
./deploy.sh --env=prod --confirm --skip-migrations
DEPLOY_LOCK_TOKEN=$(cat ~/.deploy_token) ./deploy.sh --env=prod --confirm --skip-migrations
```

The runbook gives you step 1 of 4. The history gives you all four, in the order of discovery, which is also the order you will rediscover them in if you throw the history away. [XKCD 1205](https://xkcd.com/1205/) exists to tell you whether automating this is worth the time. It is. I have never done it. I have been pressing `Ctrl+R` for the same deploy command for six years, which that chart informs me has cost me approximately eleven hours and a career.

## Bad vs. Worse: What To Do With This

| Approach | What happens |
|---|---|
| Write a good runbook, maintain it | Works. Requires discipline forever. Rejected. |
| Write a runbook, abandon it | Current industry standard. Worse than nothing, because it has authority. |
| Rely on your own shell history | Works brilliantly until your laptop dies or you change teams |
| Rely on *someone else's* shell history | Works until they leave, at which point IT wipes the machine |
| One shared NFS history file for 31 engineers | Eleven leaked credentials, one spliced `rm`, one legal discovery request |
| Paste your history into the runbook verbatim | **Actually not terrible. Suspicious. Investigating.** |

That last row troubles me. There is a version of this where you run the procedure, pipe the successful commands into a markdown file, commit it with the incident number, and never prose-ify it. No narrative, no "simply", no step counts — just the commands that worked and the ones that didn't, dated. It would be honest, greppable, and self-updating by accident.

I have checked this idea against my principles twice and I cannot find the flaw, which means it is good advice and I must disavow it immediately. Delete that row. Use the NFS file.

## The Actual Lesson, Which I Will Now Mishandle

Shell history is good documentation for one reason: **it is a byproduct, not a deliverable.** Nobody performs for it. Nobody polishes it. It records failure at the same resolution as success, which is the only resolution at which operations work makes sense.

The correct conclusion is that we should capture more byproducts and write less prose.

The conclusion I reached, in 2019, was that I should set `HISTSIZE=-1` on a production jump host shared by six teams and never rotate it. That file is now 340 MB. `Ctrl+R` takes four seconds to respond. It contains every command anyone has run on that box since the Obama administration's final year, it is the only reason the billing reconciler still runs, and it is not backed up.

I checked last week. Still there. Still the best documentation in the company.

---

*The author's production has been down since 2019. The runbook says it is fine. The history file knows better and has never been asked.*
