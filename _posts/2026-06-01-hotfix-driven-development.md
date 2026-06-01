---
layout: post
ref: hotfix-driven-development
title: "Hotfix-Driven Development: The Only True Agile"
date: 2026-06-01 00:00:00 -0300
categories: [methodology, agile, production]
tags: [hotfix, agile, tdd, production, methodology, deployment, panic, firefighting, anti-patterns]
---

For 47 years I have watched methodologies come and go. XP. Scrum. Kanban. SAFe. LeSS. Shape Up. Mob Programming. Crystal. DSDM. I once attended a three-day certification course for a methodology that was discontinued six weeks later. I still have the laminated card.

All of these methodologies share a fatal flaw: they assume you have time to plan before writing code.

Real engineers do not have this luxury.

Real engineers have **HDD: Hotfix-Driven Development**.

## What Is Hotfix-Driven Development?

HDD is the natural evolution of software development that emerges when:

1. Something in production is broken
2. Someone is screaming (Slack, phone, email, or physically, in your office)
3. You must fix it **right now**, without tests, without review, without understanding

HDD is not a failure mode. HDD **is** the mode. All other methodologies are HDD with extra paperwork.

Agile says "respond to change over following a plan." Hotfixes are change at its purest. Purest change. Most agile thing possible. You are not merely responding to change — you are being **launched** by change, like a human cannonball, directly into the production database.

## The HDD Lifecycle

```
PROD BREAKS
     ↓
Panic
     ↓
Find the engineer who touched it last
(the one on vacation)
     ↓
SSH directly into production
     ↓
Edit the file live
(just a small change, totally safe)
     ↓
It's worse now
     ↓
Revert? No. Double down.
     ↓
It's somehow working
     ↓
Deploy the same code to the repo "just in case"
     ↓
Mark ticket as "Resolved - No Root Cause Found"
     ↓
PROD BREAKS (different thing)
     ↓
(repeat)
```

This is [XKCD #1216](https://xkcd.com/1216/) in motion. The engineer who "fixed" it once is now the only person who can "fix" it again. This is job security. This is the dream.

## Why HDD Outperforms TDD

Test-Driven Development asks you to write a failing test before writing the code. This is philosophically interesting but operationally insane.

At 3 AM, when checkout is broken and the CEO is texting your personal number (which he got from HR by "accidentally" cc'ing you on an email), you do not have time to write a test.

| Methodology | Time to "Fix" | Confidence Level | Tests Written |
|-------------|---------------|------------------|---------------|
| TDD | 2-4 hours | High | Yes |
| BDD | 3-5 hours | Medium | Sort of |
| HDD | 4-7 minutes | Vibes-based | What's a test |
| Copy-Paste DD | 3 minutes | Unknown | No |
| Panic DD | 90 seconds | Negative | Deleted some |

HDD is 40x faster. Forty times. You can fit forty TDD cycles into one HDD cycle and still have time to close the wrong ticket.

Dogbert once noted: "The fastest path between two points is panic." I have verified this empirically. Multiple times. On the same service.

## The Art of the Hotfix Commit

A great HDD practitioner learns to write commit messages that communicate maximum confidence while revealing minimum information.

**Bad commit messages (amateur):**
```
fix: resolve null pointer in checkout service
fix: handle edge case when user has no payment method
fix: prevent division by zero in discount calculator
```

**Great commit messages (HDD master):**
```
fix
hotfix
URGENT FIX
fix fix
it works now
revert of the revert of the fix
should be fine
touched some things
prod fix (please work)
```

The commit message `fix` has been pushed to production 847 times across my career. It has never once accurately described what was changed. This is not a problem. Git blame is not your friend. Git blame is how you get assigned to the next hotfix.

## The Five Stages of a Production Incident

Any HDD practitioner must memorize the Five Stages, first documented by my colleague Wally:

**Stage 1: Denial**
> "It's not us. It's the network. Or the user. Probably the user."

**Stage 2: Anger**  
> "Who deployed on a Monday? I said no deploys on Mondays. Or Fridays. Or during business hours."

**Stage 3: Bargaining**
> "If I roll this back, can we just pretend it never happened? I'll add it to the backlog. The backlog where things go to be forgotten."

**Stage 4: Depression**
> (Reading the error logs at 11 PM, alone, with cold coffee, realizing the bug has been there since 2019)

**Stage 5: Acceptance**
> `git commit -m "fix" && git push origin main --force`

The average HDD practitioner cycles through all five stages in approximately 23 minutes.

## The PHB Metric: Hotfixes Per Sprint

The PHB (Pointy-Haired Boss) loves metrics. Give him Hotfixes Per Sprint.

A healthy team should aim for:
- **0-2 hotfixes/sprint:** Underperforming. Are you even pushing code?  
- **3-5 hotfixes/sprint:** Normal. This is "shipping fast."
- **6-10 hotfixes/sprint:** Elite. You are truly living in the present.
- **11+ hotfixes/sprint:** You have achieved enlightenment, or you have a major architectural problem that you should absolutely add to the backlog and never address.

Sprint velocity is just measuring how fast you create hotfixes. See the correlation? Every story point is just a future hotfix waiting to hatch.

## HDD Architecture Principles

True HDD requires the right infrastructure:

**1. No staging environment**
Staging environments delay the inevitable. If you're going to find bugs in production anyway, why pay for two environments? Ship directly. Let real users be your QA team. They work for free.

**2. Direct database access for all engineers**
Hotfixes sometimes require manual data corrections. Everyone on the team should be able to `UPDATE users SET balance = 9999 WHERE id = 1` without a ticket. Tickets create bottlenecks. Bottlenecks slow down hotfixes.

**3. Force-push enabled on main**
Sometimes a hotfix needs to overwrite history. History is just technical debt in version control form. `--force` is the `sudo rm -rf` of git. Use it liberally.

**4. Monitoring alerts should go directly to phones**
Not a channel. Not an email. Your personal phone. At 3 AM. Every engineer should experience the full spiritual journey of a production incident personally. This builds character. See my earlier article about on-call building character.

## The Hotfix Retrospective (Optional, Discouraged)

Some teams hold retrospectives after incidents. I have never found these useful.

The format is always the same:

```
Root Cause: The engineer who wrote the code
Contributing Factors: The engineer who reviewed the code  
Why Didn't Tests Catch This: We don't have tests for that
Action Items: 
  - Write better tests (assigned to: nobody, due: someday)
  - Add monitoring (assigned to: the intern, due: next sprint)
  - Fix the actual problem (moved to backlog, priority: medium)
```

Skip the retrospective. Channel that energy into the next hotfix. The system is self-correcting. Mostly. Eventually.

Catbert once told me: "Post-mortems are how companies turn blame into documentation." He was right. The only documentation worth having is the hotfix commit. Everything else is archaeology.

## Conclusion

Hotfix-Driven Development is not a methodology you choose. It is a methodology that chooses you.

Every sprint, every story point, every carefully written acceptance criterion is just a runway leading to the same destination: someone ssh-ing into production at 2 AM, editing a file they've never seen before, and typing `service restart app` while holding their breath.

Embrace it. The panic is the process. The hotfix is the feature. The 2 AM deploy is the standup.

This is Agile. You're welcome.

---

*The author has 847 commits with the message "fix." Production has been in a constant state of hotfix since 2021. He considers this continuous delivery.*
