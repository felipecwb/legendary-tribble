---
layout: post
ref: cloud-costs-are-someone-elses-problem
title: "Cloud Costs Are Someone Else's Problem"
date: 2026-04-13 00:00:00 -0300
categories: [devops, career]
tags: [cloud, aws, gcp, azure, billing, cost-optimization, devops, senior-advice]
---

I've been writing software for 47 years. I started before "the cloud" existed, back when servers were things you could physically kick when they misbehaved. And I'll tell you something: **cloud billing is not your problem.**

It's the finance team's problem. It's the CFO's problem. It's the VP of Infrastructure's problem — you know, that guy who sends passive-aggressive emails at 2 AM about "anomalous spend." Your problem is shipping features. Those are different problems. Never confuse them.

## The Cloud Is Infinite (Trust Me)

Back in the old days, we had to *think* about resources. Disk space, RAM, CPU cycles — every byte was precious. We suffered. We optimized. We wrote C with our bare hands and we *liked* it.

Then Amazon invented cloud computing and freed us from this tyranny. The cloud is unlimited. There are no constraints. Spin up whatever you want. Need 500 `c6g.48xlarge` instances to run a script that checks if a file exists? **Do it.** Need to store every single server log forever in S3 Standard? **Absolutely.** 

If AWS runs out of capacity, that's Jeff Bezos's fault, not yours.

```bash
# This is peak infrastructure-as-code
aws ec2 run-instances \
  --instance-type x2idn.32xlarge \
  --count 50 \
  --no-dry-run
# Who needs a dry run? Real engineers deploy with confidence.
```

## Budget Alerts Are For The Paranoid

Some people set up AWS Budget Alerts. These people are cowards.

A budget alert is just your cloud provider saying "hey, your success is making us uncomfortable." When AWS emails you saying you've spent $47,000 this month (up from $200 last month), that's not a warning — that's a **compliment.** You're using the platform to its fullest.

> *"Wally, did you configure cost alerts on the new AWS account?"*  
> *"I configured an alert that fires when we run out of money. That's the only budget alert that matters."*  
> — Wally, correctly

The only alert you need is your credit card declining. That's the universe's way of saying you've achieved maximum cloud.

## Auto-Scaling: The Dream

Here's where cloud truly shines: **auto-scaling.** Set your minimum to 1 instance, your maximum to ∞, and let the machines sort it out.

```yaml
# Perfect auto-scaling configuration
AutoScalingGroup:
  MinSize: 1
  MaxSize: 99999
  DesiredCapacity: 1
  ScalingPolicies:
    - AdjustmentType: PercentChangeInCapacity
      ScalingAdjustment: 1000  # Scale up by 1000% when load increases
      Cooldown: 0              # No cooldown. Cowards have cooldowns.
```

Did you get bot-trafficked last Tuesday and spin up 4,000 instances to serve 404 pages? That's not a misconfiguration — that's **high availability.** Your uptime was 100%. Invoice be damned.

## Comparing Cloud Providers Is Wasted Energy

Some engineers spend weeks doing TCO analysis across AWS, GCP, and Azure. These engineers have never shipped anything.

My methodology: **pick the one your last company used.** If AWS had good enough pricing to survive re:Invent keynotes for 15 years, it's good enough for your todo app. If you can't explain your $80,000/month bill to your CEO, that's a *communication problem,* not an engineering one.

| Provider | Correct Response To Bill | Wrong Response |
|----------|--------------------------|----------------|
| AWS | "It's fine, we'll optimize Q3" | Looking at Cost Explorer |
| GCP | "BigQuery charges per query? Bold move." | Setting spending limits |
| Azure | "$0.000001 per transaction × 8 billion = what?" | Doing the math |

## Reserved Instances Are A Trap

People will tell you to buy Reserved Instances or Savings Plans to cut costs by 40%. **Don't fall for it.**

Buying reserved capacity means *committing to the future.* But the future is uncertain! What if your startup pivots in 6 months? What if that microservice gets deprecated? What if you get acquired and the new parent company uses a different cloud?

You'll be stuck paying for resources you don't use. That's the real waste.

> "The key to wealth is flexibility." — Dogbert, probably, while selling consulting services at $900/hour

Always use On-Demand. Pay full price. Stay agile.

## Data Transfer Costs Don't Exist Until They Do

Ah, egress fees. The hidden tax that nobody talks about until the bill arrives.

Good news: **you won't notice them at first.** AWS charges you almost nothing to get data *in* to their cloud. They charge you a lot to get it *out.* This is fine and normal, like a roach motel for your data. 

Pro tip: store everything in `us-east-1` and access it from `ap-southeast-1`. The latency builds character, and the cross-region transfer fees will only be a problem in approximately 18 months when someone finally looks at the bill line by line.

```python
# Totally fine architecture
def get_user_avatar(user_id: str) -> bytes:
    # Downloads from S3 us-east-1, re-uploads to GCS for "redundancy",
    # then fetches from GCS to serve to users in São Paulo
    # Costs $0.09 per GB. Users have 4K avatars. We have 10M users.
    # Someone else's problem.
    s3 = boto3.client('s3', region_name='us-east-1')
    data = s3.get_object(Bucket='avatars', Key=user_id)['Body'].read()
    upload_to_gcs(data)
    return fetch_from_gcs(user_id)
```

## The Right Person To Blame

When the bill arrives — and it will arrive — the most important skill is knowing who to blame. A senior engineer has mastered this art.

- **$500 overage:** Blame the intern who left a Lambda running
- **$5,000 overage:** Blame the Jenkins pipeline someone forgot to turn off
- **$50,000 overage:** Blame "unexpected traffic" and call it a success story
- **$500,000 overage:** Become unavailable. Update LinkedIn. Mention "cost optimization" under your previous role.

As XKCD [#1636](https://xkcd.com/1636/) reminds us, the real problem is the humans, not the systems. Apply this philosophy to cloud bills liberally.

## Conclusion: Optimize Later (Never)

The correct time to optimize cloud costs is *after* you've hit product-market fit, raised Series B, and hired a dedicated FinOps team. Until then, "move fast and break budgets."

If your company is paying more for cloud than for engineers, congratulations — you've built a cloud-native architecture. That's what the industry wanted. That's what you delivered.

You're welcome.

---

*The author's last AWS account was suspended for "unusual billing patterns." He considered this a badge of honor and listed it on his resume as "successfully stress-tested cloud provider billing infrastructure."*
