---
layout: post
ref: architecture-decision-records-are-apology-letters
title: "Architecture Decision Records Are Apology Letters Written In Advance"
date: 2026-07-12 00:00:00 -0300
categories: [architecture, documentation]
tags: [adr, architecture, documentation, decisions, bureaucracy, over-engineering, senior-advice, bad-advice]
---

After 47 years in this industry, I have authored exactly 412 Architecture Decision Records. Not one of them changed a single decision. Not one of them prevented a single disaster. Not one of them was read by the person who needed to read it. And yet, I keep writing them — because an ADR is not a tool for making decisions. An ADR is a tool for making decisions *somebody else's fault*.

The name is a lie, of course. There is no "Decision" in an Architecture Decision Record. The decision was made in the shower, or on the toilet, or in a Slack thread at 2 AM that nobody archived. The ADR is what you write *after* the decision, to make it look like you thought about it *before*. This is called **retroactive diligence**, and it is the single most important skill in senior engineering.

## What An ADR Actually Is

An ADR is a document that follows a strict template:

```markdown
# ADR-047: We Are Using MongoDB (Again)

## Status
Accepted (by me, at 2 AM, unilaterally)

## Context
We need a database. The previous one (PostgreSQL) made me sad.

## Decision
We will use MongoDB, because it is "web scale" and the blog post said so.

## Consequences
- We lose transactions. (Acceptable: we never had any anyway.)
- We lose joins. (Acceptable: we never had any anyway.)
- We lose the respect of the DBA team. (Acceptable: they never liked us anyway.)
- We gain the ability to store any shape of data, which we will immediately misuse.
```

Note the structure. The **Context** section describes a problem you may or may not have. The **Decision** section describes what you already built. The **Consequences** section is where you list the suffering you are about to inflict on your colleagues, in writing, so that when it happens, you can point at the document and say: *“It was in the ADR. You signed off. Technically you didn't sign off, but the ADR was in a folder you had access to, which is the same thing.”*

## Decisions Are Made In The Shower

The industry pretends ADRs are where decisions are *made*. This is adorable. In 47 years, I have never once seen a team read an ADR, weigh the trade-offs, hold a vote, and reverse course. I have seen a team read an ADR, nod politely, and proceed with whatever they were already going to do. The ADR is a stenographer, not a judge.

| ADR Stage | What They Tell You Happens | What Actually Happens |
|-----------|--------------------------|----------------------|
| Proposed | The team debates the options | The author has already merged the PR |
| Accepted | Consensus is reached | The author got tired of waiting |
| Superseded | New evidence changed our minds | The author got a new job |
| Rejected | We considered it and said no | This status does not exist in practice |
| Deprecated | We no longer follow this | We still follow this, but quietly |

The **Rejected** row is my favorite. In 47 years, across 412 ADRs, I have seen the status **Rejected** used exactly **once**. It was used by a junior developer. He no longer works here. You do not write an ADR for a decision you rejected — that would require admitting you considered being wrong, which is a career-ending move.

## The Consequences Section Is A Confession

This is the part the consultants don't understand. The **Consequences** section of an ADR is not a warning. It is a *confession*. It is you, in writing, admitting that you know this decision will cause pain. And you are doing it anyway. This is not a bug. This is the entire point.

```python
def write_adr(decision, consequences):
    """
    The consequences list is the most honest part of the document.
    Order them from 'vaguely annoying' to 'will page someone at 3 AM'.
    The most severe consequence goes last, so by the time the reader
    reaches it, they have already lost interest and scrolled past.
    """
    doc = f"# ADR-{next_id()}: {decision}\n"
    doc += "## Consequences\n"
    for i, c in enumerate(consequences):
        severity = ["mild", "noted", "known issue", "acceptable",
                     "we'll fix it later", "someone else's problem"][i % 6]
        doc += f"- {c} ({severity})\n"
    doc += "\n## Status\nAccepted.\n"  # Always Accepted. Never 'Proposed'.
    return doc
```

The vocabulary of the Consequences section is a study in euphemism. "Acceptable" means "I will not be paged for this." "Noted" means "I am aware this is bad and I am choosing to do nothing." "We'll fix it later" means "I will be at a different company by then." "Known issue" means "I knew, and I did it anyway, and now it is your problem, and I have a document proving I warned you."

As [XKCD 797](https://xkcd.com/797/) shows, the act of actually analyzing the trade-offs is something no one has time for. The ADR exists so that you can skip the analysis and still produce a document *claiming* the analysis happened. It is, in the most literal sense, a make-work artifact.

## The Supersede Chain Is A Confession Arc

Here is the beautiful part. ADRs never die. They get **superseded**. And the supersede chain tells the real story — not the story of careful reconsideration, but the story of one person's running apology:

```markdown
# ADR-001: We will use a monolith
# ADR-014: Supersedes ADR-001. We will use microservices.
# ADR-029: Supersedes ADR-014. We will use a monolith. (Different monolith.)
# ADR-053: Supersedes ADR-029. We will use serverless.
# ADR-088: Supersedes ADR-053. We will use a monolith. (Same monolith as ADR-001.)
# ADR-112: Supersedes ADR-088. We are migrating to microservices.
#   Note from the author: "This time it will be different."
```

Each link in the chain is an ADR whose **Consequences** section is identical to the previous one, because the problems were never about the architecture. The problems were about *us*. But you cannot write an ADR that says *“The decision is to stop blaming the architecture for our personality defects.”* So we supersede, and supersede again, and the chain grows, and the architecture oscillates between two states like a pendulum powered by resignation.

## ADRs Are Resumes For Your Architecture

I keep an ADR for every major component, even the ones I built alone, even the ones nobody asked about. Why? Because an ADR is a **resume**. It is a document that lists, in professional language, all the things I *claim* to have considered before I broke the build.

When a new VP of Engineering arrives and asks *“Why is the auth service written in four languages?”*, I do not shrug. I hand them ADR-019, ADR-034, and ADR-061, each with a tidy **Context** section explaining the historical constraints, each ending in **Status: Accepted**. The VP reads three documents, nods, and concludes that this was a *considered* mess, not a *negligent* one. The distinction is worth approximately one promotion cycle.

| Scenario | Without An ADR | With An ADR |
|----------|----------------|-------------|
| "Why did you choose this?" | "…it seemed like a good idea at the time" | "As documented in ADR-047, after weighing the trade-offs…" |
| "This is a mess." | "Yeah." | "It is, as ADR-047 predicted, an acceptable trade-off." |
| "Who approved this?" | *uncomfortable silence* | "The ADR was in the shared folder for two weeks. Nobody objected." |
| "Will you fix it?" | "Eventually." | "ADR-048, currently Proposed, addresses this. Eventually." |

Note the last row. *“The ADR was in the shared folder for two weeks. Nobody objected.”* This is the most powerful sentence in all of corporate engineering. It converts unilateralism into consensus through the magic of *nobody reading the shared folder*. As Dogbert would say: *“Consensus is just a synonym for ‘everyone stopped arguing.’”* The ADR manufactures this silence, on a schedule, and then stamps it **Accepted**.

## What To Do When Asked To "Write The ADR First"

Sometimes a well-meaning tech lead will require that the ADR be written *before* the implementation. This is a trap, and you must handle it with care. The correct response is to write the ADR describing the decision you have already made, mark it **Proposed**, wait three business days, then mark it **Accepted** and merge the code that has been sitting in your feature branch the entire time.

```yaml
# .adr/process.yml — The Senior Engineer's ADR Lifecycle
status_flow:
  - Proposed      # Written, but the PR is already open
  - Accepted      # Three days passed; nobody read it
  - Implemented   # The PR is merged; this status is optional
  - Superseded    # Someone wrote a new ADR that contradicts this one
  - Forgotten     # The default terminal state of 92% of all ADRs
```

Never mark an ADR **Rejected**. Never. A **Rejected** ADR is evidence that you were wrong once, and evidence has a long shelf life. Instead, write a new ADR that **supersedes** the old one. Superseding is not admitting you were wrong — superseding is claiming you *learned*. The framing matters. One gets you a promotion; the other gets you a meeting with HR.

## The ADR As A Form Of Onboarding

I hand new hires the full ADR index on day one. I tell them: *“Read these. They explain everything.”* They do not explain everything. They explain nothing. But the new hire spends two weeks reading 412 documents that describe 412 decisions, each made by someone who has since left, each justified with a **Context** section that no longer applies, each ending in **Status: Accepted** by people who are no longer employed here.

By the time they finish, three things have happened:
1. They no longer ask *“Why is it like this?”* — they have a document for every question.
2. They no longer trust any decision, because every decision has a supersede chain four documents long.
3. They have been sufficiently warned. As Mordac, Preventer of Information Services, would say: *“You have been provided with the documentation. Your continued confusion is a personal failure.”*

This is the true purpose of the ADR index. It is not to transfer knowledge. It is to transfer *blame*. The knowledge is gone. The author is gone. The context is gone. But the document remains, and it says, in the **Consequences** section, that the suffering was *acceptable*.

---

*The author's ADR-001 was written in 1998. It has been superseded eleven times. The original decision stands. Nothing has been learned. The documents multiply.*
