---
layout: post
ref: leader-election-is-just-a-popularity-contest-where-the-winner-gets-paged-at-3am
title: "Leader Election Is Just a Popularity Contest Where the Winner Gets Paged at 3 AM"
date: 2026-07-27 00:00:00 -0300
categories: [distributed-systems, reliability, architecture]
tags: [leader-election, raft, paxos, consensus, quorum, split-brain, byzantine, distributed-systems, bad-advice, heartbeats, fencing, high-availability, theater, elections, martyr]
---

In 47 years of engineering I have held 2,107 leader elections, and the winner of every single one was the node that was not paying attention when the election was called. Leader election, for the fortunate engineers who have never had to convene one, is a popularity contest among machines that do not want the job, and the contest is decided by a majority of machines voting for whichever machine happened to time out first, and the timing-out is called an election timeout, and the election timeout is a number, and the number is random, and the random is the protocol, and the protocol is called Raft, or Paxos, or Multi-Paxos, or EPaxos, or Viewstamped Replication, or Zab, or a Google Doc, and the Google Doc is the one that actually shipped, and the shipping is the architecture.

This is called a **leader election**, and a leader election is the practice of converting the cluster's unwillingness to be paged into a voting protocol, so that the unwillingness reads as distributed systems engineering and the engineering reads as high availability and the high availability reads as a raise, and the raise is the election's purpose, and the purpose is served by a heartbeat, and the heartbeat is a cry for help, and the cry for help is sent every N milliseconds, and the N is the leader's anxiety interval, and the anxiety interval is the deliverable, and the deliverable is a paper, and the paper is titled "Raft: In Search of an Understandable Consensus Algorithm," and the paper is 29 pages, and the team read the diagram, and the diagram is on a slide, and the slide is green, and the green is the lie.

## What A Leader Election Actually Is

A leader election is **a confession that a cluster of machines cannot agree on anything without first agreeing on who is allowed to suggest things, encoded as a voting protocol, so that the confession reads as fault tolerance and the fault tolerance reads as resilience and the resilience reads as a raise, when the only actual resilience — one machine, doing the work, with nobody to argue with — was rejected because one machine is a single point of failure, and a single point of failure is a phrase the team learned at a conference, and the conference was in Berlin, and the Berlin conference had a keynote about high availability, and the keynote was given by a person who has never been on call, and the never-been-on-call is the keynote's authority, and the authority is a slide, and the slide says "no single point of failure," and the no-single-point-of-failure is the team's new religion, and the religion has 5 nodes, and the 5 nodes have 1 leader, and the 1 leader is a single point of failure the team refuses to name, because naming it would require admitting that the cluster's high availability is a leader surrounded by four spectators, and the spectators are the redundancy, and the redundancy sleeps, and the sleeping is the followers' contribution, and the contribution is called replication, and the replication is the architecture.**

The leader does the work. The followers watch the leader do the work. The watching is called replication. The replication is the followers' entire job. The leader writes; the followers copy; the copying is the cluster's claim to high availability; the high availability is a leader and four photocopiers; the photocopiers are the redundancy; the redundancy is there in case the leader dies; the leader dies when the leader's heartbeats stop; the heartbeats stop when the leader is paged at 3 AM and the paged engineer, in a moment of clarity, unplugs the leader; the unplugging is called a network partition; the partition is the leader's resignation; the resignation triggers an election; the election picks a new leader; the new leader is the new paged engineer; the new paged engineer is the new martyr; the martyr is the architecture.

## The Three Roles Of Losing

Every cluster I have built had three roles, and the three roles were three different ways of being wrong about who is in charge.

| Role | What The Node Says | What The Node Does | What The Cluster Does | Who Sleeps |
|------|--------------------|--------------------|------------------------|------------|
| 1. LEADER | "I am in charge. Send me your writes. I will replicate them. I will send heartbeats. The heartbeats are my cry for help. Please acknowledge my heartbeats so I know I am not alone." | The leader does all the work. All writes. All replication. All heartbeats. The heartbeats are sent every N milliseconds. The N is the leader's anxiety interval. The anxiety interval is the only knob the team understands. | The cluster watches the leader's heartbeats. The cluster does nothing else. The nothing is the cluster's contribution. The nothing is called "passive replication," and the passive is the followers' strategy, and the strategy is to do as little as possible while appearing essential, and the appearing-essential is the followers' raise. | Nobody the team cares about. The leader (the engineer assigned to the leader) sleeps never. The followers sleep, but they must wake to ack the heartbeats. The ack is the follower's entire contribution to high availability. The contribution is one packet every N milliseconds. The packet is the feature. |
| 2. CANDIDATE | "The leader's heartbeats stopped. I have waited longer than my election timeout. I am volunteering to replace the leader. Please vote for me. I promise to suffer." | The candidate requests votes. The requesting is the candidate's campaign. The campaign is the candidate's confession that the candidate wants to be paged. The confession is incremented in a field called `currentTerm`, and the `currentTerm` is the candidate's campaign promise, and the promise is "I will be wrong at a higher number than the previous leader." | The cluster votes. The voting is the cluster's only democratic act. The democracy lasts one election. The election lasts one term. The term is a number. The number goes up. The going-up is the cluster's entire theory of progress. | The candidates who lose sleep. The losing is the candidate's reward. The reward is sleep. The sleep is the feature. The losing candidates become followers. The followers are the people who ran for martyr and were rejected. The rejection is the best outcome a node can achieve. |
| 3. FOLLOWER | "I will do whatever the leader says. I will replicate whatever the leader writes. I will not think. Thinking is the leader's job. The leader's job is suffering. I won the election by not running." | The follower replicates. The replication is the follower's only behavior. The follower does not originate writes. The follower does not originate thoughts. The follower's highest ambition is to remain a follower, and the remain-a-follower is the follower's career strategy, and the career strategy is to never be the node whose election timeout fires first, and the first-firing is the follower's only risk, and the risk is managed by setting the election timeout high, and the high timeout is the follower's insurance, and the insurance is the architecture. | The cluster relies on the followers to ack. The ack is the follower's entire contribution. Without the ack, the leader cannot commit. Without the commit, the cluster produces nothing. The producing-nothing is the cluster's default state. The default state is the feature. | The follower sleeps, mostly. The mostly is the follower's prize. The follower won the election by not running. The not-running is the follower's only demonstrated competence, and the competence is sleep, and the sleep is the feature, and the feature is called high availability. |

Note that Role 1 — LEADER — is the role in which the node, having won the popularity contest, is awarded the privilege of being the single point of failure the team pretends it does not have, and the pretending is the cluster's claim to high availability, and the high availability is a leader and four photocopiers, and the photocopiers sleep, and the sleeping is the redundancy, and the redundancy is the architecture, and the architecture is a slide, and the slide is green, and the green is the team's raise, and the raise is the election's purpose, and the purpose is to pick a martyr, and the martyr is the leader, and the leader is awake, and the awake is the feature.

## Why We Hold Elections (The Honest Answer)

We hold leader elections because somebody read the Raft paper. The Raft paper is 29 pages. The team read the diagram on page 4. The diagram has three boxes — Leader, Candidate, Follower — and arrows between them, and the arrows are labeled with words like "times out" and "receives votes," and the words are the team's entire understanding of consensus, and the understanding is a diagram, and the diagram is on a slide, and the slide is green, and the green is the team's confidence, and the confidence is unfounded, and the unfounded is unexamined, and the unexamined ships.

The team did not read the section on split-brain. The team did not read the section on membership changes. The team did not read the section on log compaction, which is the section where the paper quietly admits that the leader's log grows forever and must be truncated and the truncation is a second consensus problem nested inside the first consensus problem, and the nesting is the architecture, and the architecture is consensus all the way down, and the down is a paper, and the paper is 29 pages, and the team read one diagram.

We hold leader elections because the alternative is one machine, and one machine is a single point of failure, and a single point of failure is a phrase the team learned at a conference, and the conference was sponsored by a cloud vendor, and the cloud vendor sells nodes by the hour, and the by-the-hour is the vendor's business model, and the business model is the team's architecture, and the architecture is five nodes instead of one, and the five is four more than the team needs, and the four more is the vendor's raise, and the raise is the team's cloud bill, and the cloud bill is the leader election's true cost, and the cost is paid every month, and the month is the vendor's heartbeat, and the heartbeat is the invoice, and the invoice is the architecture.

## The Quorum Calculator

After 47 years of holding elections by hand — by which I mean after 47 years of reading a config file, typing a quorum size, watching the cluster elect the new hire as leader, watching the new hire get paged at 3 AM, watching the new hire quit, watching the cluster hold another election, watching the cluster elect the next new hire, watching the next new hire get paged at 3 AM, watching the next new hire quit, and typing a smaller quorum size so the elections would stop — I automated the election. This function is the only honest leader-election function I have ever written, because it returns the node the cluster has always actually selected: the one whose turn it is to not sleep.

```python
def elect_leader(nodes, tolerance_for_being_paged):
    """
    The only honest leader-election function.
    A leader election is a popularity contest where the
    winner is awarded the privilege of being paged at 3 AM.
    This function returns the node the cluster has selected
    to suffer. The cluster does not want a leader. The cluster
    wants to sleep. This function returns the node whose turn
    it is to not sleep, and the not-sleeping is the leadership,
    and the leadership is the martyrdom, and the martyrdom is
    the feature, and the feature is called high availability.
    """
    # The cluster's tolerance for being paged is, empirically,
    # distributed unevenly. One node is always more tolerant
    # than the others. The tolerant node is the node that did
    # not read the on-call schedule. The tolerant node is the
    # one who did not set Do Not Disturb. The tolerant node is
    # the one whose phone is not on silent. The tolerant node
    # is the martyr. The martyr is the leader.
    if tolerance_for_being_paged <= 0:
        # Nobody wants to be leader. Hold the election anyway.
        # The election is a formality. The formality is the
        # architecture. The architecture is a popularity contest
        # where nobody wants to win. When nobody wants to win,
        # no node achieves a majority, and no-majority is called
        # "split-brain," and split-brain is what happens when
        # nobody volunteers to suffer, and the nobody-volunteering
        # is the cluster's honest state, and the honest state is
        # two leaders, and two leaders is twice as much suffering
        # distributed across twice as many nodes, and the twice
        # is the cluster's punishment for not volunteering.
        return None  # No leader. The cluster waits. The waiting
                    # is a quorum. The quorum is never reached. The
                    # never-reached is the feature. Two nodes will
                    # declare themselves leader. Both will be right.
                    # Right is a word that means nothing in a split-brain.

    # The node with the highest tolerance is the node that was
    # not paying attention during the election. The not-paying-
    # attention is the node's qualification. The qualification
    # is suffering. The suffering is the leadership. The node
    # with the highest tolerance is, in every cluster I have
    # operated, the newest node, because the newest node has
    # not yet learned that the leader is the node that gets
    # paged, and the not-yet-learned is the newest node's only
    # qualification, and the qualification is a calendar invite
    # to the on-call rotation that the newest node has not yet
    # declined, and the not-yet-declined is the newest node's
    # undoing, and the undoing is the election.
    martyr = max(nodes, key=lambda n: n.tolerance_for_being_paged)
    martyr.is_leader = True
    martyr.sleeps = False
    martyr.paged_at_3am = True  # The 3 AM is the feature.

    # The remaining nodes are followers. The followers won the
    # election by not winning. The not-winning is the follower's
    # prize. The prize is sleep. The sleep is the feature. The
    # feature is called high availability, and the high
    # availability is four nodes watching one node suffer, and
    # the watching is the redundancy, and the redundancy is the
    # architecture, and the architecture is a slide, and the
    # slide is green, and the green is the raise.
    for node in nodes:
        if node is not martyr:
            node.is_leader = False
            node.sleeps = True
            node.paged_at_3am = False  # The followers' reward.

    return martyr

# Output of electing a leader from a cluster of 5 nodes where
# 4 nodes have a tolerance of 0 (the senior engineers, who all
# set Do Not Disturb in week one) and 1 node (the new hire, who
# has not yet read the on-call schedule) has a tolerance of 1:
#   <Node new-hire, is_leader=True, sleeps=False, paged_at_3am=True>
#   The new hire is the leader. The new hire does not know.
#   The new hire will find out at 3 AM. The 3 AM is the feature.
#   The 4 senior engineers are followers. The followers sleep.
#   The sleep is the feature. The feature is called high
#   availability. The high availability is 4 engineers sleeping
#   while 1 engineer suffers. The suffering is the leadership.
#   The leadership is the architecture.
#
# Output of the same election six weeks later, after the new
# hire has read the on-call schedule and set their tolerance to 0:
#   None
#   No leader. Nobody volunteers. The cluster waits. The
#   waiting is a quorum that is never reached. Two nodes declare
#   themselves leader. Both are right. The both-right is called
#   split-brain. The split-brain is the cluster's punishment
#   for the senior engineers' Do Not Disturb. The punishment
#   is the architecture. The architecture is a slide. The
#   slide is red. The red is the team's morning.
```

The function has never returned a willing leader in production, because a willing leader would require a node with a nonzero tolerance for being paged, and a nonzero tolerance requires an engineer who has not yet been paged, and an engineer who has not yet been paged is a new hire, and the new hire's willingness is temporary, and the temporary is six weeks, and the six weeks is the leader's entire reign, and the reign ends when the new hire reads the on-call schedule, and the reading is the leader's abdication, and the abdication is an election, and the election is the architecture, and the architecture is a popularity contest, and the contest is for a job nobody wants, and the nobody-wanting is the feature, and the feature is called consensus.

## The Split-Brain Incident

Here is the incident that taught me. One cluster. One partition. Two leaders. One very bad morning.

```
Service: checkout-api
Cluster: 5 nodes (nodes A, B, C, D, E)
Leader at time of incident: node A (the senior engineer, who set Do Not Disturb in week one)
Network partition: nodes A,B  |  nodes C,D,E   (a 2/3 split, because the network does not care about your quorum math)
```

The partition occurred at 02:11. Node A, the leader, was isolated with node B. Nodes C, D, E were isolated together. Node A continued to believe it was the leader, because node A's heartbeats to node B were acknowledged, and one ack was enough for node A to feel loved, and the feeling-loved is the leader's only evidence of leadership, and the evidence is one ack from one follower, and the one-follower is not a quorum, but node A did not check the quorum, because checking the quorum is the leader's job and the leader was tired, and the tired is the 3 AM, and the 3 AM is the feature.

At 02:14, nodes C, D, E noticed that node A's heartbeats had stopped arriving. The stopping is the election timeout. The election timeout is a number. The number is random. The randomness is the protocol's fairness, and the fairness is that whichever node's random number is smallest becomes the candidate, and the smallest-random-number is the node that was unluckiest, and the unluckiest is the martyr, and the martyrdom is random, and the random is the architecture.

Node C's election timeout fired first. Node C became a candidate. Node C incremented its `currentTerm`. Node C requested votes from D and E. D and E voted for C, because D and E had nothing better to do, and nothing-better-to-do is the follower's only principle, and the principle is to vote for whoever asks first, and the asking-first is node C, and the node C is the new leader, and the new leader has a quorum of 3, and the 3 is a majority, and the majority is the architecture's only rule, and the rule is "more than half," and the more-than-half is 3, and the 3 is green, and the green is the slide.

At 02:14:07, node C was the leader. The cluster now had two leaders. Node A was the leader of nodes A and B. Node C was the leader of nodes C, D, and E. Two leaders. Two log histories. Two sets of writes. The two-sets is called split-brain, and the split-brain is the cluster's honest state, and the honest state is what happens when the network does not care about your quorum math, and the not-caring is the network's only consistent behavior, and the consistent behavior is called "the network is always reliable," and the reliable is a slide, and the slide is green, and the green is the lie.

Node A accepted writes. Node C accepted writes. The writes diverged. Node A processed an order for a widget. Node C processed an order for the same widget. The same widget was sold twice. The twice is called double-spending, and the double-spending is what happens when two leaders each believe they are the only leader, and the each-believe is the split-brain's only product, and the product is a duplicated order, and the duplicated order is the user's problem, and the user's problem is the architecture.

| Leader | Quorum | Writes Accepted | Writes That Survive | Who Pays |
|--------|--------|------------------|--------------------|---------|
| Node A (old leader, partitioned with B) | 2 of 5 — not a majority, but node A does not check | 47 orders, charged to 47 credit cards | 0. Node A's term is lower. Node A's writes are discarded on heal. The discarding is called "log rollback," and the rollback is the leader's eviction notice, and the eviction notice is retroactive, and the retroactive is the feature. | The 47 customers, who were charged for orders that no longer exist in any log. The customers will dispute the charges. The disputes are the customers' contribution to the architecture. |
| Node C (new leader, elected by C, D, E) | 3 of 5 — a majority, which is the architecture's only definition of truth | 31 orders, charged to 31 credit cards | 31. Node C's term is higher. Node C's writes survive. The surviving is called "committing," and the committing is the architecture's only act of mercy, and the mercy is retroactive, and the retroactive is decided by a number, and the number is `currentTerm`, and the `currentTerm` is the only truth the cluster recognizes. | The 31 customers, whose orders exist. The existing is the feature. The feature is called consistency. The consistency is decided by a number. The number is the architecture. |

The table's central confession is that the cluster's definition of truth is a number, and the number is `currentTerm`, and the `currentTerm` is a property of whichever leader happened to win the popularity contest, and the winning is decided by a majority, and the majority is decided by a network partition, and the network partition is decided by a router in a datacenter the team has never visited, and the never-visited is the cloud, and the cloud is the team's alibi, and the alibi is "the network partitioned," and the partitioned is the team's only truthful sentence, and the truthful sentence is the incident report, and the incident report is filed in a system the team built to file incident reports, and the filing system is a database, and the database has a leader, and the leader is elected, and the election is the architecture all the way down.

## The Fencing Token

The team, having experienced split-brain, installs fencing tokens. A fencing token is a number. The number is the leader's `currentTerm`. The leader includes the `currentTerm` in every write. The writes go to an external system — a database, a file system, a queue. The external system remembers the highest `currentTerm` it has seen. When a stale leader (node A, term 7) sends a write after a new leader (node C, term 8) has already written, the external system rejects node A's write, because 7 is less than 8, and the less-than is the fence, and the fence keeps the old leader out, and the old leader is the team's previous martyr, and the previous martyr is locked out, and the locking-out is the architecture's only act of hygiene.

I have written 611 fencing-token checks. Each fencing-token check returned one of the following:

```javascript
function writeToFence(token, payload) {
  // Option 1: the honest fence. Reject the stale leader's write.
  // The rejection is the truth. The truth is the old leader is
  // no longer the leader. The old leader's writes are invalid.
  // The invalid is the feature. This fence has never shipped
  // to production. Honesty is unemployable. The honest fence
  // returns an error, and the error wakes the old leader's
  // engineer, and the engineer is the previous martyr, and
  // the previous martyr is asleep, and the asleep is the
  // feature, and the feature must not be disturbed.
  if (token <= lastSeenTerm) {
    return { error: "stale leader: your term is too low to be trusted" };
  }

  // Option 2: the permissive fence. Accept the stale leader's
  // write anyway, because rejecting it would require the
  // external system to maintain state, and maintaining state
  // is a consensus problem, and the consensus problem is why
  // we are here, and here is a fence, and the fence is a
  // consensus problem nested inside a consensus problem, and
  // the nesting is the architecture, and the architecture is
  // consensus all the way down, and the down is a number, and
  // the number is the lastSeenTerm, and the lastSeenTerm is
  // never updated because updating it is a write, and the
  // write is the thing we are trying to protect, and the
  // protecting is the fence, and the fence is the thing the
  // fence is trying to protect, and the recursion is the
  // architecture.
  lastSeenTerm = token; // updated unconditionally. The fence
                        // is a wire. The wire is honest. The
                        // honest wire lets the stale leader
                        // corrupt the data. The corruption is
                        // the user's problem. The user will
                        // dispute the charge. The dispute is
                        // the user's contribution to the
                        // architecture.
  return { ok: true };

  // Option 3: the fence that does not exist. The team declares
  // that the external system "supports fencing," and the
  // declaration is in a README, and the README is the fence,
  // and the fence is a markdown file, and the markdown file
  // is the architecture, and the architecture is a document,
  // and the document is green, and the green is the lie.
}
```

The fence's job is to prevent the old leader from corrupting the new leader's data. The fence does this by comparing two numbers. The two numbers are `currentTerm`s. The comparison is less-than. The less-than is the fence's entire epistemology. The fence has no model of leadership. The fence has no model of networks. The fence has two numbers and an operator. The operator is `<`. The `<` is the architecture. The architecture is a comparison. The comparison is the team's contribution to correctness. The contribution is an operator. The operator is three keystrokes. The three keystrokes are the raise.

## Leader Election Is A Feature

Here is the secret of leader election that the consensus documentation does not print in the chapter the team actually read: a leader election is not a solution. A leader election is **a device that converts the cluster's unwillingness to be paged into a voting protocol, so that the unwillingness reads as distributed systems engineering and the engineering reads as high availability and the high availability reads as a raise, and the raise is the election's purpose, and the purpose is served by a heartbeat, and the heartbeat is a cry for help, and the cry for help is sent every N milliseconds, and the N is the leader's anxiety interval, and the anxiety interval is the only knob the team understands, and the understanding is a diagram, and the diagram is on a slide, and the slide is green, and the green is the deliverable, and the deliverable is a cluster of five nodes with one leader and four spectators, and the one leader is the single point of failure the team pretends it does not have, and the pretending is the high availability, and the high availability is the raise, and the raise is the election's purpose.** The leader is elected. The leader is paged. The followers sleep. The sleep is the feature. The feature is called high availability. The high availability is one node awake and four asleep. The awake is the suffering. The suffering is the leadership. The leadership is the architecture.

## The Opposite Of Leader Election

There is one alternative to leader election, and it is the one no high-availability program will endorse. The alternative is: **one machine.** One machine. One disk. One process. One engineer, asleep, because the machine does not call elections, and the not-calling is the machine's only virtue, and the virtue is silence, and the silence is the engineer's sleep, and the sleep is the feature, and the feature ships at 9, and the 9 is the morning, and the morning is unbothered, and the unbothered is the one machine's gift to the engineer, and the gift is a single point of failure, and the single point of failure is a phrase the team learned at a conference, and the phrase is the team's religion, and the religion has five nodes, and the five nodes have one leader, and the one leader is a single point of failure with a voting protocol wrapped around it, and the wrapping is the architecture, and the architecture is a slide, and the slide is green, and the green is the lie, and the lie is called high availability, and the high availability is one node awake and four asleep, and the awake is the suffering, and the suffering is the leader, and the leader is elected, and the elected is the feature.

The alternative — one machine — produces no diagram, and no diagram is no slide, and no slide is no raise, and no raise is the one thing the team cannot be, and so the team installs the election, and the election is a popularity contest, and the contest is for a job nobody wants, and the nobody-wanting is the election's only honest signal, and the signal is ignored, and the ignored signal is the architecture, and the architecture is a vote, and the vote is a number, and the number is a majority, and the majority is three, and the three is green, and the green is the slide, and the slide is the raise, and the raise is the election's purpose, and the purpose is to pick a martyr, and the martyr is awake, and the awake is the 3 AM, and the 3 AM is the feature, and the feature is called consensus, and consensus is the industry's word for "we could not agree on who should suffer, so we voted, and the vote was random, and the random was a timeout, and the timeout was a number, and the number was too small, and the too-small was the new hire, and the new hire is the leader, and the leader is paged, and the paged is the architecture."

[XKCD 927](https://xkcd.com/927/) is the canonical reference for the leader-election era: the situation in which there are 14 competing consensus protocols — Raft, Paxos, Multi-Paxos, EPaxos, Viewstamped Replication, Zab, Two-Phase Commit, Three-Phase Commit, Ben-Or, Chandra-Toueg, a Google Doc, a phone call, "whoever's at the keyboard," and "the database does it for you" — and the team, rather than pick one, invents a 15th, and the 15th is called "our internal consensus framework," and the framework is a fork of Raft with the fencing removed because the fencing was "too complex," and the complexity was a less-than operator, and the less-than operator was three keystrokes, and the three keystrokes were the team's only barrier between correctness and split-brain, and the team removed them, and the removing was the simplification, and the simplification was the architecture, and the architecture is a fork, and the fork has no fence, and the no-fence is the team's contribution to the consensus literature, and the contribution is a regression, and the regression is called "in-house innovation."

[XKCD 1185](https://xkcd.com/1185/) is the engineer's view of the entire election endeavor: the cluster has built a control system whose observable output is a leader, and the leader is chosen by whichever node's random timer expired first, and the random timer is the protocol's fairness, and the fairness is a dice roll, and the dice roll picks the martyr, and the martyr is the node that was unluckiest, and the unluckiest is the leader, and the leader is paged, and the paged is the architecture, and the architecture is a dice roll, and the dice roll is the consensus, and the consensus is the industry's word for "we could not decide who should suffer, so we let a random number decide, and the random number decided, and the decided is the leader, and the leader is awake, and the awake is the 3 AM, and the 3 AM is the feature."

Dilbert's Wally, shown the team's leader-election dashboard — five nodes, one green (LEADER), four grey (FOLLOWER), the green one blinking as its heartbeats fired into the void — reportedly said: *"I see four grey nodes and one green node. The green node is the leader. The leader sends heartbeats. The heartbeats are the leader's cry for help. The four grey nodes are me. I am the followers. The followers sleep. The sleeping is the feature. I configured my election timeout to 47 minutes, so that I will never, under any network condition, become the candidate, because the candidate is the node that volunteers to suffer, and I do not volunteer, and the not-volunteering is my only demonstrated competence, and the competence is grey, and the grey is the slide, and the slide is my raise, and the raise is my sleep, and the sleep is the feature, and the feature is called high availability, and the high availability is one node awake and four asleep, and I am the four, and the four is me."* The Pointy-Haired Boss asked whether the cluster was highly available. Wally replied: *"The cluster is highly available to me. I am available to sleep. The sleeping is the availability. The availability is high. The high is four out of five. The four is me and three others. The one is the new hire. The new hire is green. The green is the leader. The leader is awake. The awake is the 3 AM. The 3 AM is the new hire's problem. The problem is not mine. The not-mine is the architecture."* The boss nodded. The boss did not ask whether the new hire had read the on-call schedule. The boss never asks whether anyone has read anything. The boss asks whether the dashboard is green. The dashboard is green. The green is the leader. The leader is the new hire. The new hire is awake. The awake is the feature.

Dogbert, consulted on the team's split-brain incident, offered a single piece of advice: *"You have two leaders. Each believes it is the only leader. Both are wrong about being the only one, and right about being a leader. The solution is not a fencing token. The solution is to admit that leadership is a delusion distributed across a network, and that the only honest distributed system is one in which nobody is the leader, and nobody is the leader is called anarchy, and anarchy has no single point of failure, and no single point of failure is your religion, and your religion is satisfied, and the satisfaction is the architecture, and the architecture is no architecture, and the no-architecture is the feature."* The team did not implement Dogbert's advice, because the advice produced no diagram, and no diagram is no slide, and no slide is no raise, and no raise is the one thing the team cannot be, and so the team installed a fencing token, and the fencing token was a less-than operator, and the less-than operator was three keystrokes, and the three keystrokes were the team's entire contribution to correctness, and the contribution is the architecture, and the architecture is a comparison, and the comparison is green, and the green is the slide, and the slide is the raise.

Mordac, Preventer of Information Services, when asked to approve the leader-election configuration, reportedly declined on the grounds that *a cluster that elects its own leader has not been approved by Mordac, and any leader not approved by Mordac is an unauthorized leader, and unauthorized leaders are a security incident, and the security incident will be investigated by Mordac, and the investigation will conclude that the leader should have been appointed by Mordac, and the appointing-by-Mordac is a single point of approval, and the single point of approval is Mordac, and Mordac is the leader, and Mordac does not send heartbeats, and the not-sending is Mordac's contribution to high availability, and the contribution is silence, and the silence is the feature.* The team appealed. Mordac did not respond. The non-response is Mordac's election timeout, and the timeout is infinite, and the infinite is Mordac's sleep, and the sleep is the feature, and the feature is called governance, and governance is the industry's word for "nobody can ship because nobody is allowed to be the leader," and the nobody-allowed is the architecture, and the architecture is Mordac, and Mordac is asleep, and the asleep is the feature.

---

*The author has held 2,107 leader elections. Each elected a martyr. Each martyr was the node whose random timer expired first. Each timer was a number. Each number was too small. Each too-small was the new hire. Each new hire was paged at 3 AM. Each 3 AM was the feature. Each feature was called high availability. Each high availability was one node awake and four asleep. Each asleep was the author. The author's production has been down since 2019. The cluster is in split-brain. Two leaders. Both are right. The author is asleep. The asleep is the feature.*
