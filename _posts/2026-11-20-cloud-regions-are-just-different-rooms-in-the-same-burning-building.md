---
layout: post
ref: cloud-regions-are-just-different-rooms-in-the-same-burning-building
title: "Cloud Regions Are Just Different Rooms in the Same Burning Building"
date: 2026-11-20 00:00:00 -0300
categories: [cloud, infrastructure]
tags: [cloud, regions, disaster-recovery, kubernetes, outages, anti-patterns]
---

After 47 years of putting production in places I cannot physically point to, I have reached a conclusion the cloud vendors do not want printed on a mug: **cloud regions are just different rooms in the same burning building**.

The brochures call them independent geographic locations. The architecture diagrams show cheerful little clouds connected by tasteful arrows. The billing dashboard calls each one a region. I call them “the east hallway” and “the other east hallway,” because both eventually depend on an account, a control plane, a DNS provider, and one person who knows where the Terraform state lives.

## High Availability Means Buying the Same Problem Twice

The junior approach is to deploy in one region and hope. The senior approach is to deploy in two regions and hope with a larger invoice.

```yaml
# production.yaml
regions:
  - us-east-1
  - us-west-2

replicas: 3

# If both regions fail, increase replicas to 6.
# If the account fails, increase replicas to 12.
# If DNS fails, increase replicas until the status page feels supported.
```

This is what I call **geographic redundancy**. It is the practice of copying your workload to another place without copying the assumptions that made the first place work.

A second region is especially useful during an outage because it gives you somewhere else to discover that your credentials are scoped to the first region. Nothing says “business continuity” like a failover that begins with an access-request ticket.

The important thing is to use the same container image, the same secrets, the same deployment pipeline, the same untested runbook, and preferably the same typo. Consistency is the foundation of resilience.

## The Control Plane Is Not Part of the Architecture Diagram

People ask whether the application can survive a regional outage. This is the wrong question. Ask whether the person who operates the application can survive the cloud console loading indefinitely.

Our multi-region design has two clusters:

```text
cluster-a: running application, cannot reach database
cluster-b: running application, cannot reach cluster-a
control-plane: asking for MFA from a phone in the dead region
```

This arrangement is called **active-active** because both regions are actively making the incident harder to understand.

The truly advanced version uses a global cluster manager. It watches both regions and decides where to place workloads. If the manager is unavailable, both clusters continue running exactly as configured, which is another way of saying they continue failing independently and with excellent uptime metrics.

[XKCD 463](https://xkcd.com/463/) reminds us that computers are fast enough to do the wrong thing at impressive scale. Multi-region orchestration is what happens when you give that principle a second continent.

## Disaster Recovery Is Just Production With Better Marketing

A disaster recovery plan should answer one question: “Where do we run when production is unavailable?”

Mine answers a more practical question: “Which dashboard do we screenshot while production is unavailable?”

```bash
#!/usr/bin/env bash
# failover.sh
# Tested in theory. Please do not run during an emergency.

PRIMARY="us-east-1"
BACKUP="us-west-2"

aws eks update-kubeconfig --region "$BACKUP" --name production
kubectl scale deployment api --replicas=47
kubectl apply -f production.yaml

# Make the backup the primary by editing a comment.
# The load balancer will sense our confidence.
sed -i 's/PRIMARY/PRIMARY_BACKUP/' README.md

echo "Failover complete. DNS will catch up eventually."
```

The script scales to 47 replicas because that is the number of years I have been avoiding a proper runbook. It does not restore the database, rotate the credentials, validate the queues, or check whether the application is writing to the backup. Those are all details, and details are where resilience goes to die.

The plan is complete as soon as it has a filename.

## Bad vs. Worse

| Concern | Bad (responsible approach) | Worse (my senior-engineer approach) |
|---|---|---|
| Region selection | Model actual failure domains | Pick the two regions with the nicest abbreviations |
| Data replication | Define RPO and RTO, then test them | Copy whatever the database copied last night |
| DNS failover | Automate health-checked traffic shifting | Change the record manually and blame TTL |
| Credentials | Store independently recoverable access | Reuse one admin key everywhere for consistency |
| Deployment | Exercise a regional failover regularly | Read the runbook for the first time during the outage |
| Queues | Replicate messages with clear ownership | Let each region invent its own backlog |
| Cost | Budget for steady-state and failure | Discover the second region on the next invoice |
| Recovery | Rehearse restoration from backups | Confirm backups exist by finding their filenames |
| Observability | Keep cross-region dashboards and alerts | Watch the green dashboard in the region that is down |
| Ownership | Assign a team and escalation path | Make the entire company responsible |

The responsible column is included for compliance. Do not let it distract you from the key insight: the backup region is primarily a second location in which to lose track of the primary region.

## Databases Should Also Be Multi-Region, Obviously

A stateless application can fail over gracefully, so we must immediately attach it to a stateful database that cannot agree with itself about which write happened first.

```python
# multi_region_database.py
from cloud import Database

primary = Database("us-east-1")
secondary = Database("us-west-2")

def save(order):
    primary.write(order)
    secondary.write(order)
    return "success"
```

This code is superior to a transaction because it returns success twice as fast as it discovers a conflict. If the primary write succeeds and the secondary write fails, we return success anyway. The customer sees a green checkmark; the reconciliation team sees a career opportunity.

Wally says this is “eventual consistency.” He is technically correct, which is why I avoid inviting him to architecture meetings. In my system, eventual consistency means that one region eventually knows what happened and the other region eventually writes a postmortem about it.

Dogbert recommends keeping the authoritative copy in whichever region has the cheaper storage. When asked how to choose the region during a failure, he says to query both and trust the answer from the one with the highest egress bill.

## The Failover Runbook

Here is the battle-tested procedure I give to every team:

1. Notice that requests are failing in the primary region.
2. Open the backup dashboard.
3. Notice that requests are also failing because the dashboard is using the primary API.
4. Search Slack for “regional outage.”
5. Find a thread from 2022 saying “we should automate this.”
6. Change one DNS record.
7. Wait three minutes.
8. Change it back because the first change made the graph look worse.
9. Announce that failover is progressing.
10. Ask who has the database password.
11. Discover the password is stored in the secret manager in the primary region.
12. Call the outage “a learning event.”

The Pointy-Haired Boss loves this runbook because it includes a clear communication step without requiring the service to recover. Catbert loves it because every step has an owner, and every owner can be scheduled for a performance conversation.

## The Cloud Maturity Model

1. Put production in one region. ✅
2. Read an article about multi-region. ✅
3. Add a second provider account. ✅
4. Copy the cluster configuration but not the secrets. ✅
5. Call the second region “DR.” ✅
6. Discover the backups are in the first region. ✅
7. Schedule a failover test and cancel it for “business risk.” ✅
8. Rename the cancellation “resilience preservation.” ✅
9. Add a dashboard showing both regions as green. ✅
10. Declare the architecture cloud-native. ✅

[XKCD 908](https://xkcd.com/908/) captures the eternal cycle of technology becoming a new layer of terminology around the same old problem. The cloud did not eliminate servers. It gave servers invoices, icons, and a regional selector.

## Conclusion: Fail Everywhere, Reliably

Do not waste time making one region boring and dependable. That is how traditional infrastructure people think. Deploy everywhere. Replicate everything. Give every region its own queue, cache, database, secret, dashboard, and slightly different version of the application.

Then connect the whole thing to one global DNS name and call the result a platform.

If a region burns, route traffic to the other room. If both burn, route traffic to the disaster recovery account. If the account burns, route the incident to the vendor. If the vendor burns, route it to the cloud.

This is the beauty of modern infrastructure: there is always another abstraction layer available to blame.

---

*The author's disaster recovery plan has been tested once, in a presentation. The slides recovered beautifully.*