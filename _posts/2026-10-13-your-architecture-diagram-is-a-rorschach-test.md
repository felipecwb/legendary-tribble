---
layout: post
ref: your-architecture-diagram-is-a-rorschach-test
title: "Your Architecture Diagram Is a Rorschach Test"
date: 2026-10-13 00:00:00 -0300
categories: [architecture, documentation, anti-patterns]
tags: [architecture, diagrams, uml, c4, excalidraw, lucidchart, documentation, rorschach, terrible-advice, meetings, wiki, onboarding]
---

Every company has one. A slide deck — sorry, a "living architecture document" — with boxes, arrows, and a cloud icon labeled PROD. Nobody has opened it since the offsite. It was born in a workshop, it lives in Confluence next to the meeting notes from 2019, and it has exactly one function: to be waved at during the all-hands while someone says the word "ecosystem" with a straight face.

In 47 years of mass-producing bugs, I have drawn 847 architecture diagrams. I remember none of them. The codebase remembers none of them. The only witness is a `.pptx` in a Drive folder called `FINAL_final_v3_(approved)(2)`, and the person who approved v1 quit in 2021.

Here is my thesis, and I will defend it with code: **the architecture diagram is not documentation. It is a Rorschach test.** Everybody who looks at it sees their own job in it. Nobody sees the system. Nobody *can* see the system, because the system is 47 repositories, a mainframe in a rack in Ashburn, a cron job owned by someone who left in 2021, and a stored procedure that only runs on leap years. The diagram shows four boxes and an arrow labeled "async." The diagram is lying. The diagram has never told the truth, because *the truth cannot fit on a slide*.

## The Box Model of Modern Architecture

Let me reconstruct the canonical diagram from memory, because I have seen it 847 times:

```text
┌───────────────┐       ┌───────────────┐       ┌───────────────┐
│  FRONT-END    │──────▶│  BACK-END     │──────▶│   DATABASE    │
└───────────────┘       └───────────────┘       └───────────────┘
                                                        │
                                                 ┌──────▼──────┐
                                                 │   CACHING   │
                                                 └─────────────┘

              [producers] ──▶ ((KAFKA)) ──▶ [consumers]
                                     │
                              (raw boxes unknown)

  ☁️ CLOUD ☁️  — border decoration, no data ever crosses it
```

Four boxes. One arrow. One cache drawn next to the database because the database is where diagrams believe performance lives. This is the diagram for a system that, in reality, contains: 847 endpoints, a GraphQL layer, four message schemas of which two are quarantined, a Python 2 cron job, a queue with exactly one consumer named `consumer_OLD_DO_NOT_DELETE.py`, and a Kafka topic with 40 partitions and a retention of "whatever the default was in 2021." The diagram is not wrong. The diagram is *a haiku* about the system. It contains, in three lines, the impression of the system, the way a road sign with a cow on it contains the impression of a farm.

And like the cow sign, it is not for drivers. It is for people who have never been on a farm and never will be.

## Diagram-Driven Development: The Methodology

The industry has decided that the diagram comes *before* the code. I have been in these workshops. Here is the sequence:

1. **Six engineers sit in a room.** None of them have touched the code in nine months. Two of them have never touched it. One is from Sales. Nobody knows why the person from Sales is there. Nobody asks.
2. **Someone opens a tool** — Lucid, Miro, Excalidraw. The tool has 47 shapes and a shape library containing "hexagon (microservice, 2023)" and "hexagon (microservice, 2024, deprecated)."
3. **The whiteboard gets full.** Not because architecture emerged — because *the rectangle tool has no maximum size*, and whiteboards fill at the rate of rectangles, which is the rate of meetings.
4. **Someone says "we should circle back on this."** This is the only deliverable. Nobody circles back. The photo of the whiteboard is posted in a Slack channel called `#architecture` whose only other message is from 2021 and says "is anyone still maintaining this diagram?"
5. **The diagram is declared "a living document."** A living document is a document that is declared living and then never touched again, the way a plant is declared "low-maintenance" and then dies. The diagram now exists in a state I call *archival life support*: present in the org chart of documents, absent from the org chart of systems.

This is Diagram-Driven Development, and its output is not a design. Its output is a *consensus* — a consensus about how the system *would* work if the system were the diagram. The system is not the diagram. The system is the diagram's debts.

## What the Arrow Actually Means

The single most dangerous glyph in software is the arrow. Watch how it is used:

```text
[A] ────▶ [B]          "B consumes from A"
```

That is what the engineer who drew it meant. Here is what the arrow *actually* means, by department:

| Who reads it | What the arrow means to them | Confidence | Correct? |
|---|---|---|---|
| The architect | "B consumes from A" | 100% | No |
| Backend engineer | "A pushes to B via some queue" | 60% | No |
| SRE | "A pages B at 3 AM" | 100% | Yes |
| SRE's pager | "A wakes me up" | 100% | Yes — this is the only correct reading |
| Security | "A trusts B" | 90% | No |
| Sales | "This is a feature we can sell" | 100% | Also no |

Six readings. Zero consensus. The SRE is the only one who is right, because the SRE is the only one who has *met* the arrow in production. The SRE has debugged the arrow. The SRE has watched the arrow retry with exponential backoff for 40 minutes while a customer churns. The SRE knows the arrow has a retry policy, a timeout, a poison-pill dead-letter queue, and a name that appears in no repository — the arrow's implementation is a Lambda function named `handler.py` that was written during a hackathon in 2021 and the hackathon repo was archived. The diagram shows an arrow. The production system contains a *biography*.

This is why I say the diagram is a Rorschach test: the diagram is a stimulus, and every department's reading is a projection. The diagram doesn't describe the system. The diagram describes *the department's relationship to the system* — which is the only thing the meeting was ever about anyway.

## The Legend of the Stale Diagram

Every org has a legend. Not a diagram legend — an organizational legend. The story of the one diagram that *was* accurate, once:

> "The old architecture doc was actually correct. Dave drew it in 2019. Dave was a staff engineer. Dave knew everything. The doc was accurate for exactly one sprint — the sprint in which Dave drew it — and then the system changed and the diagram didn't, because the diagram is a photograph and the system is a river. Dave left for a crypto startup. The crypto startup failed. Dave is now at another crypto startup. The diagram remains, in Confluence, labeled 'Last updated: 2019'. It has 214 comments. 214 comments, zero edits since 2019. The comments are the diagram's users, and they are arguing about a system that hasn't existed since the acquisition."

I have heard a version of this legend in every company I have ever consulted for. The names change. The Confluence page does not. Somewhere out there, right now, there is a Confluence page titled "System Architecture v2.0" that has not been updated since a sprint that ended with the words "we should circle back on this." The page has 214 comments. The system it describes no longer exists. The comments are still active. The comments are *the most active thing in the org*.

## "But What About C4?"

Ah, the C4 model. "Context, Containers, Components, Code." The industry's answer to diagram rot is *more diagram* — four levels of it, each with its own notation, each to be maintained separately, each with a documented convention for being wrong.

I have seen 47 C4 implementations. Here is the honest status report:

| Level | Intended to show | What it shows in practice | Maintained by | Last update |
|---|---|---|---|---|
| L0 The wall | All 47 diagrams at once | 47 diagrams remain; update rate 0/47 | The architect | "2021" |
| L1 Context | System boundary + users | A big square with stick figures | The architect | "2021" |
| structurizr | Executable diagrams as code | A DSL nobody else can read | One true believer | "yesterday" |
| L2 Containers | Deployable units | Boxes with Kafka inside them | The architect | "2021" |
| L3 Components | Internals of a container | A zoom-in of the 2021 system | Nobody | "2021" |
| L4 Code | The code | Which is the only accurate diagram, because it *is* the system | The compiler | "now" |

My point is not that C4 is wrong. My point is that C4 *expects* a human to maintain four parallel documents by hand, and humans, as established, do not maintain documents. Humans maintain *vibes*. There is one diagram that never rots, and it is L4: the code. The compiler doesn't rot. The compiler regenerates the truth every build. If you want an accurate diagram of your system, your build tool *is* the diagram tool. The only diagram that has ever been accurate is the one the compiler draws, and the compiler has no opinions, no hexagons, and no tooling budget.

## The Table Nobody Wanted

| Approach | Time to draw | Time to rot | Readers | Truth content |
|---|---|---|---|---|
| Whiteboard photo in Slack | 40 min | 40 min | 0 (they saw it in the meeting) | Whatever it was when the photo was taken |
| Confluence page "living doc" | 2 weeks | First deploy after it | 3 (2 are from Sales) | Below the road-sign-with-a-cow |
| C4, all four levels | 6 weeks | First deploy after it | 2 | Below the cow sign, in four zoom levels |
| Diagram-as-code (Mermaid) | 4 hours | First deploy after it | 1 (the author, once) | Slightly more current than the cow |
| The actual code | 47 years | Never — it *is* the system | Everyone, constantly | 100%, and it compiles |

Notice the pattern: **rot is not a property of the tool. Rot is a property of hand-maintained documents in an automated system.** The code changes on every merge. Any document that does not change on every merge is, by definition, describing a system that no longer exists — the only question is how far behind it is, and the answer is always "at least one deploy, and the last deploy was the one that broke it."

## Wally Understood Immediately

**Wally**, who has been here since the strip's founding and has optimized his career into a single boolean:

> "I don't read diagrams. Diagrams are a genre of fiction where all the characters are boxes. I did read the last one once. It had an arrow labeled 'async.' I asked what the arrow was, in code. They said 'oh, it's a Lambda.' I asked what the Lambda does. They said 'it's in the diagram.' The diagram is a Lambda that draws a diagram of itself. I have stopped asking. My output is unchanged and my calendar is clear, which is the metric that matters."
>
> "Also, I have a diagram of my own. It's a photo of my couch. The label says 'system boundary.' It's the most accurate diagram this company has ever produced, because the couch is the only component whose behavior I can fully specify."

**The Pointy-Haired Boss**, who has discovered the diagram in a new tool and is announcing it at the all-hands:

> "Great news. We have adopted a *living architecture diagram* in [tool]. Every service is a shape. Every integration is an arrow. The diagram is the source of truth. From now on, if the diagram and the code disagree, the code is wrong. We have also purchased 47 licenses for [tool]. The seats are assigned by department. The diagram will be maintained by the Department of Diagrams, which reports to me, which is why it will be accurate. I have printed the diagram. It is on my wall. My wall has never been wrong."

The Department of Diagrams has four members. Three are from Sales. The fourth is an SRE who joined for the money and stays because the pager says otherwise. The diagram is updated quarterly. Deploys happen daily. The ratio is 47:1, and every one of the 47 deploys in that ratio is a lie the diagram tells.

**Dogbert**, consulted as Chief Strategy Officer:

> "Diagrams are for people who can't read code and won't read people. Sell them to the second group. My recommendation: sell the diagram as a *product* — 'Executive Architecture View', $47/month per seat. The customer is buying the *feeling* of comprehension, and feelings have infinite margin. The diagrams rot, but the subscription does not. When the diagram becomes inaccurate, ship v2 of the diagram and call it 'refreshed for clarity'. The inaccuracy was never the product. The inaccuracy was the *excuse* for the subscription."
>
> "I have sold 847 seats. The customer's engineers have never opened it. The customer's CFO opens it quarterly, sees four boxes and a cloud, and approves the budget. The CFO is not buying the diagram. The CFO is buying the ability to point at a wall in a meeting and say 'per the architecture.' The wall is the product. The arrow is marketing."

**Catbert**, Evil Director of Human Resources, on the new responsibilities:

> "The Department of Diagrams has KPIs now. Diagram freshness: 0%. Diagram readers: 4. Diagram-to-code consistency: unmeasurable, therefore perfect. Diagram KPIs are ideal because none of them can be falsified — you cannot be wrong about a document nobody reads. My top performer this quarter is the diagram itself. It requires no onboarding, never takes PTO, and is wrong 100% of the time, which is the correct amount for a document with 'living' in the title."
>
> "I have also added a diagram review to the promotion packet. The packet requires a diagram of the candidate's system. Every candidate submits the same diagram: four boxes, an arrow labeled 'async', a cloud. I have approved 847 promotion packets containing the identical diagram. Nobody has noticed. This proves the diagram works — as an org chart, not an architecture."

**Mordac**, the Preventer of Information Services, on the diagram tooling request:

> "You want 47 seats of a diagram tool, SSO for all of them, and a Confluence space for the export. Denied. Approved. Both. You will get a single shared license, one seat, login `diagrams@`, password `diagrams123`, rotated by a cron job I will not document. The export will be a PDF, which is a diagram format that has never been wrong because it has never been read. The PDF is stored next to the meeting notes from 2019. Bring two invoices. Come back Tuesday."
>
> "Also: the diagram tool is now the only system I fully trust, because it is the only system whose entire function is to be looked at and not believed. I have considered putting the prod database behind it. Denied. Approved. Both."

## "But Diagrams Are Great for Onboarding!"

They are! Genuinely. A diagram is the fastest way to give a new engineer the *impression* of the system, which they will hold for approximately two weeks, after which reality arrives and the impression dies like a plant declared low-maintenance.

The problem is not the onboarding value. The problem is that the diagram is the *only* artifact with onboarding value, so it gets given to every new hire and *revered*, and reverence is how documents rot. The new hire looks at the diagram, nods, believes it, and then spends their first incident discovering that the arrow labeled "async" is a Lambda from a hackathon repo, the cache is actually two caches and they disagree, and the "FRONT-END" box is three different frontend repositories whose only shared thing is the diagram. The diagram was a map of the system as *believed*, not as *run*. New hires don't need the diagram. New hires need the truth, and the truth is the code, the dashboards, and the one engineer who remembers why the Lambda exists. That engineer is on vacation.

See [XKCD 1739](https://xkcd.com/1739/) — "Infrastructure" — in which a new hire follows the org chart's diagram and arrives at a box labeled "infra" that contains only a man shrugging. That man is Dave. Dave left for a crypto startup. The diagram still has his email. The diagram still has his email *as the only on-call escalation path*. The diagram is not a map. The diagram is a *haunted house*, and the ghost is still receiving pages.

Also see [XKCD 1831](https://xkcd.com/1831/) — wait, that's 1831, "The Review Is the Diff"... no. Let me start over. [XKCD 1739](https://xkcd.com/1739/) again. The diagram is not a map. The botched mid-sentence correction is intentional — it is what a hand-maintained document looks like in practice, a document that was edited by a human, in a rush, and never reviewed, and whose errors are now *load-bearing* because the diagram is the only doc anyone opens. The botched sentence is the diagram. You are welcome.

## My Conclusion, Which You Will Implement Anyway

I am not telling you to stop drawing diagrams. I have no such power, and my inbox suggests you won't listen regardless. I am telling you what the artifact *actually is*, so that when production breaks and you open the diagram, you can name what you are looking at:

```text
# the honest diagram, annotated
┌───────────────┐      ┌───────────────┐      the org chart of documents, not systems
│   DIAGRAM     │◀─────│   CONFLUENCE  │      214 comments, 0 edits since 2019
└───────────────┘      └───────────────┘
        │                                     │
        │ "async"                             │ someone said
        ▼                                     ▼
┌───────────────┐      ┌───────────────┐      ┌───────────────┐
│  MEETING      │─────▶│  DEPARTMENT   │─────▶│    BUDGET     │
│  (origin)     │      │  OF DIAGRAMS  │      │ (the product) │
└───────────────┘      └───────────────┘      └───────────────┘

        nothing in this diagram is the system.
        the system is 47 repos and a Lambda named handler.py
        written at a hackathon in 2021. the repo is archived.
        the Lambda is load-bearing.
```

Notice what is missing from my honest diagram: the system. Notice what the diagram has: a budget. That is the honest architecture. The diagram is not documentation. It is an *invoice* — the paper trail of a meeting, the receipt for 47 tool licenses, the org chart of a department that exists because the diagram exists. The code is the system. The diagram is the receipt for the meeting where someone described the system from memory, incorrectly, and everyone nodded.

The diagram is a Rorschach test. Every reader sees their own job in it. The SRE sees the pager. Sales sees the roadmap. The CFO sees the budget. The diagram itself sees nothing — it cannot, because it is a photograph of a river, and the river has moved on, and the photograph is still on the wall, still being pointed at, still saying "per the architecture."

---

*The author has drawn 847 architecture diagrams. He has consulted on systems in 11 companies, and in every one, the most accurate architecture document was a file called `notes.txt` in someone's home directory, containing one line: "it's a monolith, don't tell anyone." The diagram on the wall says "async." The Lambda is load-bearing. The author's out-of-office reply says "refreshing the diagram." The refresh is scheduled for 2029.*
