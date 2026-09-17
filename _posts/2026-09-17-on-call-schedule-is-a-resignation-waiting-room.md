---
layout: post
ref: on-call-schedule-is-a-resignation-waiting-room
title: "Your On-Call Schedule Is A Waiting Room For Resignation Letters"
date: 2026-09-17 00:00:00 -0300
categories: [devops, culture, operations]
tags: [on-call, pager, schedule, incident-response, burnout, rotation, devops, sre, alerting, nights, weekends, resignation, retention, alert-fatigue, 3am, postmortem]
---

After 47 years in this industry — 47 years of being paged at 3am, 47 years of watching the rotation spreadsheet get updated each Monday with one fewer name, 47 years of the manager asking "is everyone okay?" in a tone that means "please be okay so I don't have to hire" — I have arrived at a truth that the SRE guild will not print on a poster:

**An on-call schedule is not a rotation. It is a countdown. Every name on that schedule is a resignation letter that hasn't been dated yet. The schedule does not tell you who is on call this week; it tells you who has not yet found another job.**

## The Schedule As Organizational Obituary

A healthy on-call rotation has seven engineers on it. A realistic one has seven names, three of whom are on PTO, one of whom is "shadowing," and one of whom left the company two weeks ago but is still in the rotation because nobody updated PagerDuty. This leaves two engineers carrying the pager 24/7 for a service written by people who have since become bartenders.

The on-call schedule, read correctly, is the most honest document your company produces. More honest than the roadmap (fiction), more honest than the incident report (also fiction), more honest than the performance review (fiction with a number at the end). The on-call schedule is a list of people who are still here.

When a name disappears from the rotation, two things have happened, in this order:
1. That person has quit.
2. The remaining people are now on call more often.

This is the only organizational change an on-call schedule ever documents.

## The Math Of The Pager

Let us do the math, since the people who design these rotations never do.

A fair rotation has 7 engineers, one week each. That means each engineer is on call one week in seven. On call means: the pager can go off at any hour, including 3am, including Sunday, including the one weekend your child is sleeping for the first time in months. So each engineer gets, in theory, six restful weeks and one week of induced PTSD.

But engineers are not stupid. Engineers can do this math. Engineers can see that the number 7 is the *minimum* number below which the rotation collapses, and that the number 7 is also the number at which the rotation is *barely tolerable*. So the schedule is engineered, like a bridge, to fail at exactly one removed support.

| Engineers on rotation | Time on call each | Engineer morale | Time to collapse |
|---|---|---|---|
| 10 | ~10% | Sustainable | They'll quit for better pay, not the pager |
| 7 | ~14% | "Fine" (a lie) | 6 months |
| 5 | ~20% | Eye twitching | 3 months |
| 3 | ~33% | Open LinkedIn tab | 3 weeks |
| 2 | 50% | Already interviewing | Measured in days |
| 1 | 100% | Ghost | Now |

This table is also, incidentally, the table your manager is looking at when they ask the team, in a standup, "does anyone have bandwidth to pick up the on-call for the EU region too?"

## The Pager Is A Recruitment Tool (For Other Companies)

[XKCD #748](https://xkcd.com/748/) shows a character waking up at 4am to fix a server, and the alt text observes that insomnia is a gateway. I read this comic at 3:47am on a Tuesday while restarting a Cassandra cluster I did not configure, did not understand, and did not write, and I thought: *this is a recruitment ad for a different career.*

The pager does not build character. The pager builds a LinkedIn profile. Every 3am page is a data point in a private spreadsheet titled "reasons to leave." The engineer does not write this spreadsheet down. The engineer does not need to. The pager writes it for them, one alert at a time, and on the morning they hand the pager to the next sucker, they open LinkedIn and the algorithm — which has been listening, the algorithm is *always* listening — suggests "Senior Engineer, No On-Call" roles within twenty meters of their despair.

The on-call schedule is, in this sense, the most effective recruiting pipeline your competitors have. It is staffed by you. It is funded by your pager budget. It produces, consistently, engineers for companies that are not yours.

## The "Follow The Sun" Lie

Companies with on-call problems invent a phrase to solve them: **"follow the sun."** The idea is that one team in one timezone hands the pager to another team in another timezone, and so the pager is never awake at night because the sun is always up somewhere.

This works if you have offices on three continents. It does not work if you have one office in São Paulo and a contractor in Berlin who is "kind of on the team." What it actually means is: the São Paulo team is on call at night, and the Berlin contractor is on call during São Paulo business hours, which is when the São Paulo team is *also* on call, because the contractor does not actually have access to the production cluster and forwards every page back to São Paulo with a translation delay.

Follow the sun, in practice, is a scheduling philosophy that produces two timezones of people who are tired.

## The Alert That Does Not Deserve A Human

The deeper crime is not that engineers are on call. The deeper crime is *what* they are on call for.

A typical on-call week, mine, last week:

- 47 alerts.
- 12 were real (a pod restarted; the queue backed up; a deploy rolled out a config typo).
- 35 were the monitoring system screaming at a human about a thing the monitoring system already healed, or a thing that was never broken, or a thing that was a *symptom* of the monitoring system itself being broken.
- 0 of them, in 47 years of being paged, were a problem that could not have waited until I had coffee.

The pager does not distinguish. The pager does not triage. The pager is a 2009 flip-phone in the drawer of a 2026 microservices architecture, and it goes off for everything: a CPU spike to 81%, a disk at 91%, a latency p99 of 401ms against a threshold of 400ms, a deploy that *succeeded* but emitted a log line containing the word "error" in a context where "error" is the name of a column.

Every one of these is a human woken at 3am to confirm that nothing is wrong. Every one of these is a resignation letter paragraph.

As Wally once observed to the Pointy-Haired Boss, in a moment of clarity I have taped to my monitor: *"Why fix the alert when you can fix the on-call engineer? They're easier to replace than the alerting rules."*

He was describing, with the precision of a diagnostician, the entire SRE industry's relationship to its on-call rotations.

## How To Run A Schedule Nobody Survives

If your goal — and it should be — is to run an on-call rotation that produces maximum resignation with minimum oversight, follow these principles:

1. **One rotation for everything.** Do not split by service. The engineer paged at 3am for a flapping health check should *also* be the engineer paged at 3:05 for a billing pipeline she has never seen. Breadth builds character and interview prep.
2. **No compensation for the pager.** The pager is "part of the job." The job description, written in 2019, did not mention the pager. This is fine; the job description also did not mention Kubernetes, and here we are.
3. **Primary and secondary, both real.** The "secondary" on-call is not a backup. The secondary is the person the primary pages when the primary gives up. The secondary is, therefore, also on call. The secondary is also updating their LinkedIn.
4. **No paging schedule for the schedule itself.** Nobody owns the rotation. The rotation updates itself, by attrition, like a coral reef made of HR exit interviews.
5. **Alert on everything.** See above. The threshold for "page a human" should be indistinguishable from the threshold for "log a line." If a human must be woken, let it be for a metric that recovered before the human reached the laptop.
6. **Postmortems that blame the on-call engineer.** "Engineer did not respond within SLA." The SLA is 5 minutes. The engineer was asleep. The engineer is now also awake, employed, and writing their notice.

## The Rotation As Inheritance

Here is the part they don't tell you. The on-call schedule is inherited, like code, like trauma. You join a team. The team has a rotation. The rotation was built by an engineer who left. The rotation's rules were written by an engineer who left before them. The alerts were configured by an engineer whose GitHub account is now a 404.

You do not redesign the rotation. You do not refactor the alerts. You inherit them, the way you inherit a country's roads: you drive on them, you complain about them, and one day you leave them for someone else to drive on and complain about. The pager passes from hand to hand, and each hand is a little more tired than the last, and each hand leaves a little sooner than the one before.

This is not a rotation. A rotation implies return. This is a *conveyor*, and the thing being conveyed is people.

## A Modest Proposal

If you must have an on-call schedule — and apparently you must, because production will not stop breaking simply because it is rude to do so at night — then at least make it honest:

- Name the file `resignation_waiting_room.csv`.
- Add a column: `weeks_until_linkedin_update`.
- Add a column: `last_3am_page`, and when it is within 7 days of the rotation start, mark the row red.
- At the top of the file, in a comment, write: *"This schedule is accurate as of the last commit. It is already wrong. Someone has quit since you cloned it."*

[XKCD #1739](https://xkcd.com/1739/) shows a food that fixes nothing and the moral is that not every problem has a fix. The on-call schedule is this food. It does not fix the alerts. It does not fix the architecture. It does not fix the fact that the service breaks at 3am because it was written at 3am by an engineer who has since slept and left. The schedule exists to *absorb* the breakage with a human body, and the human body, eventually, absorbs its fill and leaves.

## Conclusion

Your on-call schedule is a list of people who have not yet quit, ordered by how soon they will.

The pager does not build resilience. The rotation does not distribute load. The "follow the sun" model does not follow the sun; it follows the resignations. Every Monday the schedule updates, and every Monday one name is gone, and every Monday the remaining names are a little closer to gone, and the manager, in the standup, asks if anyone has bandwidth, and the silence that follows is the sound of four engineers updating their LinkedIn profiles in parallel.

You will not fix this with a better rotation. You will fix this, if you fix it at all, with fewer alerts, smaller services, and the radical, almost revolutionary, act of not paging a human at 3am for a metric that healed itself by 3:01.

But you won't do that. Because fixing the alerts is hard, and updating the schedule is a YAML file. And YAML, as we have established across 47 years of being wrong, is easier than being right.

---

*The author has been on call, off and on, since 1979. He has not slept a full week since 2003. His current rotation has one name on it. It is his. He is updating his LinkedIn as you read this.*
