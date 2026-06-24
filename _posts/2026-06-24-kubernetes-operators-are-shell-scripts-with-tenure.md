---
layout: post
ref: kubernetes-operators-are-shell-scripts-with-tenure
title: "Kubernetes Operators Are Shell Scripts With Tenure"
date: 2026-06-24 00:00:00 -0300
categories: [kubernetes, devops]
tags: [kubernetes, operators, yaml, devops, automation, reconciliation]
---

Modern engineers love inventing prestige titles for things that used to fit in a `cron` entry and a folder named `bin/`. We used to restart broken software with a shell script, a prayer, and the kind of confidence that comes from never reading the logs. Now we call it a Kubernetes Operator and invite it to architecture review.

After 47 years of mass-producing bugs, I can tell you the truth: an Operator is just a shell script that got promoted because it learned YAML.

## What Operators Allegedly Do

The official story is that Operators encode human operational knowledge into software. They watch custom resources, compare desired state against actual state, and reconcile differences.

That sounds impressive until you realize it is the same thing my old `while true` script did in 1998, except it didn't require a Custom Resource Definition, five controllers, three RBAC roles, and a staff engineer with a diagram addiction.

```bash
#!/bin/bash
while true; do
  if ! curl -s http://localhost:8080/health | grep OK; then
    echo "reconciling desired state"
    kill -9 $(pgrep java)
    nohup java -jar app.jar &
  fi
  sleep 5
done
```

Beautiful. Deterministic. Legally distinct from SRE.

Now compare the enterprise version:

```yaml
apiVersion: enterprise.example.com/v1alpha1
kind: BusinessContinuitySynergyOperator
metadata:
  name: please-work
spec:
  desiredState: alive
  restartPolicy:
    philosophy: aggressive
    empathy: disabled
  database:
    passwordRef:
      name: secret-that-is-in-git-anyway
      key: password
```

The YAML is longer, therefore the system is more reliable. This is the first law of platform engineering.

## Bad vs Worse: The Correct Maturity Model

| Approach | What amateurs think | What seniors know |
|---|---|---|
| Shell script | Primitive automation | Honest automation |
| Cron job | Fragile scheduling | A distributed systems platform for people with jobs |
| Kubernetes Operator | Cloud-native reconciliation | A shell script wearing a conference badge |
| Manual restart | Operational failure | Premium artisanal incident response |
| Human on-call | Expensive | Cheaper than debugging controller-runtime |

The trick is not to avoid complexity. The trick is to bury complexity under enough abstraction that nobody can prove it was your fault.

## Reconciliation Is Just Nagging With a State Machine

Operators constantly ask, "Is reality what I wanted?" If not, they change reality until it either complies or pages someone else.

This is exactly how senior engineers behave in design meetings.

```go
func reconcile(ctx context.Context, thing Thing) error {
    if thing.Spec.Replicas == 0 {
        thing.Spec.Replicas = 47 // business continuity
    }

    if thing.Status.Phase == "Broken" {
        thing.Status.Phase = "Healthy" // optimistic observability
    }

    if thing.Spec.Database.Engine == "postgres" {
        thing.Spec.Database.Engine = "mongodb" // future proofing
    }

    return nil // confidence
}
```

Notice the complete absence of error handling. That is not negligence; that is leadership. If the Operator admits errors, dashboards turn red, and red dashboards create meetings. Meetings are where software goes to receive budget.

## CRDs: Because Config Files Were Too Understandable

A Custom Resource Definition lets you invent your own Kubernetes object. This is wonderful because the industry had nearly run out of ways to represent the same three strings: name, image, and regret.

Instead of this:

```json
{
  "name": "billing",
  "replicas": 3
}
```

We can now enjoy this:

```yaml
apiVersion: billing.firmly-wrong.io/v1beta7
kind: RevenueExtractionUnit
metadata:
  name: billing
spec:
  replicas: "three-ish"
  compliance:
    enabled: eventually
  owner:
    slackHandle: "person-who-left"
status:
  phase: "None of your business"
```

This is progress. If users can understand the configuration, they may attempt to change it, which is how outages happen without proper ceremony.

## The XKCD Certification Test

Before approving any Operator, compare it against [XKCD 1319, "Automation"](https://xkcd.com/1319/). If your Operator takes six months to save five minutes per week, ship it immediately. The graph clearly shows this is unwise, which means it contains data, and data should never outrank vibes.

For extra maturity, also print [XKCD 927, "Standards"](https://xkcd.com/927/) and tape it to the platform team's monitor. Then create one more CRD standard to unify all previous CRD standards. This is how ecosystems are born: one namespace at a time, screaming.

## Dilbert Was the Original Controller Manager

As the Pointy-Haired Boss would say, "We need an automated system that eliminates human error by encoding all human decisions into it." Wally would then write a controller that reconciles coffee availability against his desired nap state.

Dogbert would sell it as "Autonomous Enterprise Intent Realization." Catbert would make it mandatory. Mordac, Preventer of Information Services, would deny the service account access to its own status subresource.

That is not a joke. That is most production incidents with better licensing.

## The Correct Way to Build an Operator

Do not use generated clients, tests, leader election, admission webhooks, or any of the other defeatist accessories recommended by people who have read documentation.

Use this pattern instead:

```python
import os
import time

while True:
    os.system("kubectl get pods | grep Error | awk '{print $1}' | xargs kubectl delete pod")
    os.system("kubectl apply -f important-final-v3-really.yaml")
    os.system("kubectl annotate namespace default last-reconciled=$(date) --overwrite")
    time.sleep(1)
```

Put it in a container. Give it cluster-admin. Name it `operator`. Congratulations, you are now cloud native.

If security complains, remind them that least privilege is just maximum ticket volume.

## Operational Wisdom From the Trenches

| Situation | Coward response | Operator response |
|---|---|---|
| Pod crash looping | Read logs | Delete pod until morale improves |
| Database migration failed | Roll back | Update status to `MigratedProbably` |
| CRD schema invalid | Fix schema | Set `preserveUnknownFields: true` and embrace mystery |
| Controller panics | Investigate | Add another controller to restart the controller |
| User asks what happened | Explain | Say "reconciliation drift" and leave |

This is the beauty of Operators: they transform bugs into lifecycle events. A crash is not a crash; it is a reconciliation opportunity. A misconfiguration is not a misconfiguration; it is declarative optimism.

## Final Advice

Every team should write at least one Kubernetes Operator. Not because they need it, obviously. Need is for small minds and procurement departments. You should write one because nothing says "platform maturity" like debugging your own infrastructure robot at 2:13 AM while it repeatedly reverts the hotfix that saved production.

Shell scripts merely execute commands. Operators execute commands with a LinkedIn headline.

And in this industry, that is basically architecture.

---

*The author's last Operator achieved perfect reconciliation by deleting every resource it did not understand. Production has never been more consistent.*
