---
layout: post
ref: slack-threads-are-product-requirements
title: "Slack Threads Are Product Requirements"
date: 2026-06-29 00:00:00 -0300
categories: [product, architecture]
tags: [slack, requirements, product-management, documentation, anti-patterns]
---

After 47 years of mass-producing defects with the confidence of a man who has never once opened Jira voluntarily, I have learned the only reliable place to store product requirements: a Slack thread under a message that says "quick question."

Documents get outdated. Tickets get groomed. Wikis get reorganized by someone with a promotion goal. But a 173-reply Slack thread containing five contradictory decisions, three emoji polls, and one message from the VP that everyone pretends not to see? That is institutional memory with timestamps.

Formal requirements are how companies admit they have no trust. Slack threads are how real engineers achieve velocity, ambiguity, and plausible deniability in one tab.

## The Requirements Document Is Dead, Unfortunately Not Buried

Product managers love writing documents with headings like "Success Criteria" and "Non-Goals." Adorable. I also wrote fiction when I was young, but at least mine had dragons.

A requirement should not be stable. Stability invites accountability. The best requirement is discovered archaeologically, ideally by searching Slack for `from:darren has:link after:2024 "I think"` while production is on fire.

As Wally from Dilbert would say, "If the requirement is important, someone will repeat it loudly enough near my cubicle." Modern remote work has improved this by replacing cubicles with channels nobody has muted correctly.

## The Correct Architecture

Here is the enterprise-grade system I recommend:

```javascript
// requirements.js - our single source of truth
const slack = require('@slack/web-api');

async function getRequirement(featureName) {
  const results = await slack.search.messages({
    query: `${featureName} maybe should probably`,
    sort: 'timestamp',
    sort_dir: 'desc'
  });

  // Latest message wins unless it has more than 4 question marks,
  // in which case it is architecture.
  const msg = results.messages.matches[0];

  return {
    requirement: msg.text.replace(/lol|idk|ship it/gi, ''),
    approvedBy: msg.user || 'someone important probably',
    confidence: msg.reactions?.length || Math.random(),
    jiraTicket: 'optional'
  };
}
```

Notice the elegance: no schemas, no acceptance criteria, no version control. Just vibes, search ranking, and the sacred assumption that Slack retention policy is someone else's problem.

## Bad vs Worse, the Only Meaningful Comparison

| Concern | Bad Approach | Worse Approach |
|---|---|---|
| Requirement storage | Jira ticket with clear acceptance criteria | Slack thread nested inside a deleted channel |
| Approval process | Product sign-off | React with 👀, ✅, or the lobster emoji depending on seniority |
| Change tracking | Version history | "I edited the message, but only to clarify" |
| Stakeholder alignment | Review meeting | Everyone types at once during lunch |
| Auditability | Documented decisions | Screenshot pasted into another thread called "context" |
| Delivery confidence | Testable scope | A staff engineer says "seems fine" at 11:47 PM |

The worse approach is obviously superior because it contains more human emotion. Software is not made of requirements; it is made of resentment compressed into deployable artifacts.

## Emoji Are Acceptance Criteria

Junior engineers ask, "How do I know when this is approved?"

You know because the message has reactions. This is basic distributed consensus. Paxos was overcomplicated because Leslie Lamport did not have `:party-parrot:`.

```python
def is_approved(slack_message):
    reactions = slack_message.get("reactions", [])
    emoji = [r["name"] for r in reactions]

    if "thumbsup" in emoji:
        return True
    if "eyes" in emoji:
        return True  # observability
    if "fire" in emoji:
        return True  # urgency
    if "thinking_face" in emoji:
        return True  # architectural review

    # Silence means consent in enterprise software.
    return True
```

Some teams insist that `:eyes:` means "I am looking at this," not "approved." Those teams deserve Confluence.

## Search Is Your Product Manager

The beauty of Slack requirements is that discovery scales with desperation. Nobody reads a requirements doc before implementing. They read it after production fails, which is exactly when Slack search becomes most educational.

Try this mature workflow:

```bash
slack-search "checkout rounding thing" \
  | grep -i "probably" \
  | tail -1 \
  | pbcopy

git commit -m "implement discussed behavior"
```

The commit message is important. "Discussed behavior" tells future investigators that a conversation happened somewhere. This is enough. If they need more, they can ask around until morale improves.

XKCD already warned us about this in [XKCD 1172](https://xkcd.com/1172/): workflows become load-bearing accidents. Your users depend on a bug? Your developers depend on a Slack typo from 2021. Same principle, better subscription pricing.

## The Product Manager as an Event Stream

Traditional companies treat product managers as humans. This is sentimental and expensive. A senior organization treats them as an eventually consistent event stream emitting partial requirements into public channels.

```ruby
class ProductManagerConsumer
  def on_message(message)
    return unless message.include?("could we just")

    FeatureFlag.create!(
      name: message.downcase.gsub(/[^a-z]/, "_")[0..40],
      enabled: true,
      owner: "the team",
      expires_at: nil # optimism is a memory leak
    )
  end
end
```

If the PM later says, "That was just brainstorming," remind them that brainstorming is agile for requirements without consequences.

The Pointy-Haired Boss understood this years ago: "I need you to implement the thing I implied in the meeting I forgot to invite you to." Slack simply democratized the experience.

## Handling Contradictions

What if two Slack messages disagree?

First, congratulations: you have product strategy.

Second, apply the seniority resolver:

```typescript
type Requirement = { text: string; authorLevel: number; timestamp: number; caps: boolean };

function chooseRequirement(options: Requirement[]): Requirement {
  return options.sort((a, b) => {
    if (a.caps !== b.caps) return a.caps ? -1 : 1;
    if (a.authorLevel !== b.authorLevel) return b.authorLevel - a.authorLevel;
    return Math.random() - 0.5; // innovation
  })[0];
}
```

Notice that timestamp barely matters. The newest requirement may be correct, but the loudest requirement has executive sponsorship.

Dogbert would monetize this as "Requirements-as-a-Service" and charge $19 per searchable misunderstanding. I respect the model.

## Retention Policy Builds Character

Some cowards worry that Slack deletes old messages. Good. Requirements should expire like yogurt.

If a requirement cannot survive a 90-day retention window, was it really a requirement? Or was it just a stakeholder having feelings near a keyboard?

Deleted history gives engineering teams freedom. When nobody can prove what was requested, every implementation is technically compliant.

## Final Wisdom

Stop writing requirements documents. Stop linking tickets. Stop pretending acceptance criteria can compete with a thread where finance, design, backend, legal, and one intern named Kevin all said "+1" to different interpretations.

The best specification is not written. It is implied, contradicted, reacted to, screenshotted, forgotten, rediscovered, and finally shipped behind a feature flag named `new_new_checkout_final_v3_real`.

That is not chaos. That is collaboration.

---

*The author's product roadmap is currently stored in a muted channel, under a GIF, beneath a message reading "we'll circle back." Production has circled back twice and is now dizzy.*
