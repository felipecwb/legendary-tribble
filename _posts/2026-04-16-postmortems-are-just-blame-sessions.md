---
layout: post
ref: postmortems-are-just-blame-sessions
title: "Blameless Post-Mortems Are Gaslighting (You Should Absolutely Blame Someone)"
date: 2026-04-16 00:00:00 -0300
categories: [devops, culture]
tags: [post-mortem, incident-response, blame, culture, devops, management]
---

I have survived 47 years of production incidents. I have watched servers catch fire — metaphorically and once, memorably, literally. I have sat through hundreds of "blameless" post-mortems where everyone in the room pretended that "the system failed" instead of saying what we all knew: Dave pushed to production on a Friday after three beers and dropped the wrong table.

The blameless post-mortem is a comfortable lie we tell ourselves so we don't have to have uncomfortable conversations. Today I'm here to burn that lie to the ground and explain why pointing fingers is the correct, time-honored, and deeply satisfying engineering tradition.

## Why "Blameless" Is a Corporate Fantasy

The Silicon Valley thought leaders invented "blameless culture" for a single, selfish reason: they were tired of engineers quitting after incidents. Instead of fixing their processes, their environments, or their hiring, they invented a framework for collective amnesia.

They called it "psychological safety." I call it "no one learns anything."

Here is the real 5 Whys process, done correctly:

1. **Why did the database go down?** Someone ran `DROP TABLE` in production.
2. **Why did they run `DROP TABLE` in production?** They thought they were in staging.
3. **Why did they think they were in staging?** Both environments look identical.
4. **Why do both environments look identical?** Dave set them up and Dave "doesn't believe in color-coding."
5. **Why is Dave still employed?** Excellent question. See Action Item #1.

In a blameless culture, the answer to Why #5 is "the system needs better guardrails." No. The system needs a different Dave. The guardrails just need to keep Dave *away from the keyboard*.

## The Correct Post-Mortem Template

Stop using those wishy-washy Google SRE templates designed by people who have never been woken up at 3 AM by a PagerDuty alert that says `CRITICAL: everything is on fire (maybe)`. Here is the template I've been refining since 1997:

```markdown
## Post-Mortem: [SERVICE] Outage — [DATE]

**Duration:** [X hours] of pure, unadulterated humiliation

**Root Cause:** [PERSON'S NAME] did [STUPID THING]

**Why They Did It:** Unknown. Possibly hubris. Possibly that third energy drink.

**Impact:**
- [N] users affected
- $[AMOUNT] revenue lost
- [AMOUNT] of my remaining faith in humanity evaporated

**Action Items:**
1. [ ] Have a "conversation" with [PERSON]
2. [ ] Make [PERSON] present the post-mortem to their peers, live
3. [ ] Revoke [PERSON]'s production access until they can name 3 databases
        without using their fingers
4. [ ] Add a warning sticker to [PERSON]'s keyboard: "ARE YOU IN PROD?"
5. [ ] Rename staging to something [PERSON] can remember,
        like "NOT_PROD_DO_NOT_TOUCH_DAVE"

**Written By:** Me. The only engineer awake at 3 AM.

**Reviewed By:** No one. Everyone else was asleep like cowards.
```

## The Five Stages of a Production Incident

In my 47 years, I've identified the precise psychological arc of every on-call engineer:

| Stage | Duration | Observed Behavior |
|-------|----------|-------------------|
| Denial | 0–5 min | "The monitoring is probably wrong" |
| Anger | 5–20 min | WHO DID THIS (Slack, all caps, no punctuation) |
| Bargaining | 20–45 min | "If I just restart the pod…" |
| Depression | 45–120 min | Staring at logs while eating chips from bag |
| Acceptance | 2–8 hours | "I'll just rewrite it from scratch this weekend" |

The blameless post-mortem deliberately skips stages 1 through 3 — which are, as any experienced engineer knows, the most diagnostic stages.

## On Psychological Safety

[XKCD #979 "Wisdom of the Ancients"](https://xkcd.com/979/) perfectly captures the moment you realize that no one on Stack Overflow, in the documentation, or in your entire organization has ever encountered your exact production failure. That is the moment "psychological safety" means the freedom to say out loud: "I have no idea what's happening and I'm genuinely terrified, please send help and snacks."

Real psychological safety is this: when you break production, your colleagues mock you mercilessly, add your incident to company lore, and name the patch after you — "the Dave Migration" — and then buy you a beer. That is community. That is culture.

The Silicon Valley version is: when you break production, everyone pretends it was "systemic," nothing is anyone's fault, no one learns anything, and Dave does it again in six months with a different table.

## Wally's Post-Mortem Philosophy

Wally from Dilbert has been attending the same post-mortem meeting for 15 years. He understands the essential truth: the post-mortem document is not a learning artifact. It is an alibi.

> **PHB:** "This post-mortem shows we need more monitoring."  
> **Wally:** "I wrote that. It's always a monitoring problem when I write it, and always a people problem when I didn't cause it."  
> **PHB:** "Brilliant diagnosis."  
> **Wally:** "I know. I also wrote 'increase test coverage.' That one's been on the action items since 2011."

## The Real Action Items You Should Write

Standard post-mortem action items are a theater of optimism:

- "Improve monitoring" *(we will add one more alert that no one will tune)*
- "Update runbook" *(the runbook will not be opened until the next identical incident)*
- "Add alerting for X" *(X will alert 400 times a day until someone silences it forever)*
- "Hold training session" *(30 minutes on Zoom, cameras off, everyone on their phones)*

Here are better action items, written in plain English:

```
ACTION ITEMS — Incident #47 "The Great Dave Drop" — 2026-04-11

✅ DONE:      Rolled back Dave's migration
✅ DONE:      Restored data from backup taken 4 hours before incident
              (RTO: 6h, RPO: 4h — please do not tell the SLA team)

🔄 IN PROG:   Explaining to Dave what "production" means and why it differs
              from "the database I was looking at before lunch"

📋 TODO:      Lock prod database access behind a second authentication step
              that requires typing the words "I UNDERSTAND THIS IS PRODUCTION"

📋 TODO:      Add database-level trigger that Slacks #incidents whenever
              anyone attempts a DDL statement on prod without a ticket number

📋 TODO:      Review whether our on-call rotation should include
              "ability to read connection strings"

🚫 WONT FIX: Teaching Dave to read error messages before running scripts
             Estimate: 6–8 sprints, medium confidence
```

## The Post-Mortem Review Meeting

The post-mortem review meeting is where all the good work from the post-mortem document gets quietly reversed. Here is how every post-mortem review meeting goes:

1. Someone presents the timeline. The timeline is wrong in at least three places but no one corrects it because the incident was two weeks ago and everyone has moved on.
2. The Root Cause section is discussed. "System complexity" is agreed upon as the root cause, which means no one caused it, which means no action items will be completed.
3. The action items are reviewed. Six have been assigned to "the team." No one knows which team. They will not be completed.
4. Someone suggests that "maybe we need better observability." Everyone nods. No one defines what observability means in this context. A Jira ticket is created and immediately triaged to the backlog.
5. The meeting ends fifteen minutes early. Everyone feels vaguely productive.
6. The same incident happens again in four months.

## Naming Incidents After Their Causes

I propose returning to the ancient tradition of naming incidents after the people or decisions that caused them. Not for punishment. For institutional memory. For culture.

"The Great Dave Drop of 2026" ensures that when the conversation turns to database safeguards, everyone remembers *why* they exist. It is a monument. A teaching tool. A cautionary tale whispered to new engineers during onboarding.

Compare:
- "We had an incident with cascading failures in the data layer" — no one learns anything
- "Dave once dropped the users table in production; that's why we have the MFA prompt before DDL" — Dave becomes legend, the guardrail makes sense, the knowledge persists

Dave doesn't mind. Dave has a hoodie from the incident swag. Dave tells the story at every team dinner. Dave has grown.

---

*The author has caused 23% of all incidents in their 47-year career and has documented all of them accurately as "infrastructure issues beyond our control." Current open post-mortem action items: 847. Closed: 2. Both were "add emoji to the runbook."*
