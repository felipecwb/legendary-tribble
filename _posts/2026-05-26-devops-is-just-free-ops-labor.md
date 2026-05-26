---
layout: post
ref: devops-is-just-free-ops-labor
title: "DevOps Is Just Making Developers Do Ops Work for Free"
date: 2026-05-26 00:00:00 -0300
categories: [devops, culture, career]
tags: [devops, sre, ops, on-call, culture, career, agile, suffering]
---

I was here when we had system administrators. Real sysadmins. People who wore cargo shorts to the office, drank Mountain Dew at 9 AM, and could configure a Cisco router blindfolded. They owned production. They owned the pagers. They owned the post-mortems.

Then someone invented "DevOps" and suddenly it was the developers' problem too.

After 47 years of watching people rename job descriptions to justify not hiring staff, I can tell you exactly what DevOps is: it's the practice of making developers maintain infrastructure while calling it "empowerment."

Brilliant. Absolutely brilliant. I tipped my hat the day I understood it.

## The Historical Context

Before DevOps (the golden era):

- Developer writes code
- Developer throws code over the wall
- Ops deploys it
- Something breaks
- Ops blames developer
- Developer blames ops
- Nobody fixes it
- Everyone goes home at 5 PM

This was a good system. Healthy conflict. Clear boundaries. Everyone had someone to blame who wasn't themselves.

Then along came Patrick Debois in 2009 with his "DevOpsDays" conference and ruined everything. Now:

- Developer writes code
- Developer *also* writes the Dockerfile
- Developer *also* writes the Kubernetes manifest
- Developer *also* sets up the CI/CD pipeline
- Developer *also* owns the on-call rotation
- Something breaks at 3 AM
- Developer is paged
- Developer blames themselves
- Developer fixes it
- Developer gets no raise

See the improvement?

## The DevOps Toolchain (The Real Cost)

They told you DevOps would make you faster. What it actually gave you:

| Old World | DevOps World |
|-----------|-------------|
| Write code | Write code + YAML |
| 1 job title | 1 job title, 4 job descriptions |
| Paged sometimes | Paged always |
| Blame ops | Blame yourself |
| 40 hour week | 40 hours coding + 20 hours "infra tasks" |
| One language | One language + Bash + HCL + YAML + Dockerfile syntax |
| On-call: sysadmins | On-call: you, forever |

> *"I've been doing DevOps for three years," said the developer. "I've written more YAML than code."*
> — Overheard at every tech company, 2019–present

## The YAML Tax

Every DevOps tool has decided that YAML is the universal language of infrastructure. This is a war crime.

```yaml
# What you wanted: deploy my app
# What you got:

apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  namespace: production
  labels:
    app: my-app
    version: "1.0.0"
    managed-by: helm
    team: backend
    cost-center: "12345"
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: my-app:1.0.0
        ports:
        - containerPort: 8080
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 15
          periodSeconds: 20
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          # 47 more lines...
```

In the old days, a sysadmin would `scp` your binary to a server and run it. Took 30 seconds. Now you spend a week writing YAML that someone will break by adding two spaces of incorrect indentation.

[XKCD 1172](https://xkcd.com/1172/) covers this perfectly: every configuration someone depends on will eventually become unmaintainable. In DevOps, this happens in week two.

## The On-Call Lie

"With DevOps, the people who write the code are also responsible for it in production. This creates accountability!"

Translation: your sysadmins were expensive. Now you are on call too, and you are not getting extra pay for it.

```bash
# Your phone at 3:47 AM:
# PagerDuty: CRITICAL - prod-api-7 memory usage 98%
# PagerDuty: CRITICAL - prod-db connection pool exhausted  
# PagerDuty: CRITICAL - cert expires in 2 hours
# PagerDuty: CRITICAL - k8s node NotReady

# What you think you'll do:
kubectl describe pod prod-api-7-xyz
kubectl logs prod-api-7-xyz --previous

# What you actually do:
kubectl delete pod prod-api-7-xyz  # restart it and pray
# goes back to sleep

# Next morning standup:
# "I heroically resolved four critical incidents overnight"
# Wally: "Nice, I turned my pager off in 2019"
```

Wally is the most senior DevOps engineer I know. His strategy? Never learn Kubernetes. They can't page you for things you don't own.

## Infrastructure as Code Is Just Code With More Ways to Break

They told you Terraform would make infrastructure reproducible. What they meant was: now your infrastructure can have merge conflicts.

```hcl
# terraform.tfstate conflict:
<<<<<<< HEAD
resource "aws_instance" "web" {
  instance_type = "t2.micro"
}
=======
resource "aws_instance" "web" {
  instance_type = "t3.xlarge"  # someone applied manually
}
>>>>>>> origin/main

# Congratulations. Your infrastructure is now in superposition.
# It is both t2.micro and t3.xlarge until you run terraform apply.
# Schrödinger's EC2.
```

In the old days, a sysadmin knew exactly what was on every server because they *built it by hand* and *remembered*. Now we have Terraform state files stored in S3 with DynamoDB locking and sometimes the lock doesn't release and the whole team is frozen for two days.

Progress.

## The "You Build It, You Run It" Manifesto (Translation)

Werner Vogels at AWS said "You build it, you run it." Here's the accurate translation of that entire philosophy:

| What They Said | What They Meant |
|----------------|-----------------|
| "Ownership culture" | Unpaid on-call rotation |
| "Empowered teams" | No dedicated ops to call |
| "Move faster" | Cut headcount, add YAML |
| "Observability" | Now you debug prod too |
| "Reliability engineering" | You write the runbooks |
| "Blameless post-mortems" | We document how you failed |
| "Platform team" | One person managing Kubernetes for 200 developers |

As [XKCD 303](https://xkcd.com/303/) illustrates, the real compile time is now your Helm chart rendering. And nobody can complain because "at least we're not running bare metal."

## The Career Ladder Scam

Junior DevOps Engineer → DevOps Engineer → Senior DevOps Engineer → Staff DevOps Engineer → Principal DevOps Engineer → You're Still Doing The Same YAML

The PhB once asked me: "Can we just have developers do the DevOps?" I said: "You're already doing that, you're just calling them developers and paying them developer salaries."

He promoted me for the insight. Then had me put it into action.

## How to Survive DevOps (The Wally Method)

1. **Become the on-call expert for only one obscure service** — something nobody else understands. This means you get paged less (nobody knows to escalate to you) and can claim expertise without doing work.

2. **Write terrible runbooks** — detailed enough to look professional, vague enough to be useless. "If this alert fires, check the logs and resolve accordingly."

3. **Praise Kubernetes loudly, understand it privately as little as possible** — speak the vocabulary, never actually apply. "We should really be looking at service mesh solutions" buys you six months.

4. **Terraform everything, deploy nothing** — have beautiful `.tf` files in git. The actual `terraform apply` is always "pending review."

5. **Make yourself the single point of failure for something unimportant** — the team won't page you at 3 AM if paging you means waking up the only person who knows the SSL renewal process.

## Conclusion

DevOps is not a culture. It's not a movement. It's not empowerment. It's a business model where you take the operations headcount, distribute the work across the engineering team, add "DevOps" to everyone's job title, and keep the savings.

The brilliant part? Engineers *liked it* at first. "We own our destiny!" Yes. You own your destiny. And the Dockerfile. And the Kubernetes manifests. And the on-call rotation. And the post-mortems. And the cost optimization spreadsheet.

The sysadmins in cargo shorts? Most of them are now "Platform Engineers." They still wear cargo shorts. They are the only ones who actually understand what's running. They have never been happier.

---

*The author refused to learn Kubernetes in 2017. He is still employed, still shipping features, and has not been paged once. He considers this his greatest professional achievement.*
