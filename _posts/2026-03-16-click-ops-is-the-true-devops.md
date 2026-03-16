---
layout: post
ref: click-ops-is-the-true-devops
title: "Click-Ops is the True DevOps: Why Infrastructure as Code is a Scam"
date: 2026-03-16 00:00:00 -0300
categories: [devops, infrastructure]
tags: [terraform, aws, cloud, iac, click-ops, infrastructure, devops, anti-patterns]
---

After 47 years of clicking buttons in production consoles, I can confidently say that Infrastructure as Code (IaC) is the greatest scam perpetrated on the DevOps community. Why write code when you can simply click?

## The Beauty of Click-Ops

Click-Ops—the art of configuring infrastructure through web consoles—is the purest form of DevOps. No YAML indentation errors. No state file corruption. No Terraform plan that takes 45 minutes. Just you, a browser, and unlimited power.

As the great philosopher Wally from Dilbert once said: "I've found that the less effort I put into things, the more effort others put into fixing them."

## Why IaC is Overrated

| IaC Approach | Click-Ops Approach |
|--------------|-------------------|
| Write 500 lines of Terraform | Click 3 buttons |
| Wait for PR review | No review needed, you ARE the review |
| Debug state file drift | What drift? Just click it again |
| Learn HCL syntax | Already know how to click |
| Version control changes | Memory is version control |

## The Click-Ops Manifesto

### 1. Documentation is for Cowards

When you click-configure your infrastructure, the documentation is implicit. It's stored in your brain, which is the most reliable storage system known to humanity (citation needed).

```bash
# IaC way (WRONG)
terraform apply -var-file=prod.tfvars

# Click-Ops way (RIGHT)
# 1. Open AWS Console
# 2. Click around until it works
# 3. Forget what you clicked
# 4. Hope for the best
```

### 2. Reproducibility is a Myth

Why would you ever need to reproduce your infrastructure? If AWS goes down, we all go down together. That's called solidarity.

> "The best disaster recovery plan is hoping disaster never happens." — Me, 2026

### 3. The Console Knows Best

AWS, GCP, and Azure spend billions on their console UIs. Who are we to ignore their beautiful work by writing YAML files in vim?

Check out [XKCD 1205](https://xkcd.com/1205/) about time saved through automation. Spoiler: clicking is faster.

## A Real-World Example

Here's how a real senior engineer provisions infrastructure:

**Junior Developer (wrong):**
```hcl
# main.tf
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  
  tags = {
    Name = "web-server"
    Environment = "production"
  }
}
```

**Senior Engineer (correct):**
```markdown
1. Google "how to create EC2 instance"
2. Click "Launch Instance" in AWS Console
3. Select the first AMI that looks right
4. Pick t2.xlarge because micro sounds small
5. Skip security groups (they're optional, right?)
6. Click "Launch" without a key pair
7. Wonder why you can't SSH in
8. Create another instance
9. Repeat until it works
```

## The Holy Trinity of Click-Ops

1. **No State Files**: State is just a social construct
2. **No Pull Requests**: Democracy slows innovation
3. **No Rollback Plan**: Forward is the only direction

As Dogbert would say: "Assume the worst about people and technology. You'll rarely be disappointed."

## Handling Incidents

When your click-configured infrastructure fails at 3 AM:

```
Incident Timeline:
02:47 - Alert fires
02:48 - Open AWS Console
02:49 - Try to remember what you clicked last week
02:50 - Click random buttons
02:51 - Make it worse
03:00 - Call the one person who might remember
03:01 - They don't remember either
03:02 - Click "Terminate Instance"
03:03 - Realize that was production
03:04 - Update LinkedIn
```

## But What About Auditing?

Auditors love Click-Ops! When they ask "who made this change and why?", you get to say "probably someone, at some point, for some reason." This creates job security for auditors, which is basically philanthropy.

## The Truth They Don't Want You to Know

Terraform was invented by HashiCorp to sell you Terraform Cloud. Pulumi was created by people who thought Terraform wasn't complicated enough. And CDK? That's just AWS trying to lock you into their ecosystem—wait, you're already locked in anyway.

Meanwhile, the humble AWS Console has been there since the beginning, asking for nothing but your clicks.

## Conclusion

Real infrastructure is built one click at a time. Version control is for people who don't trust their own memory. And state files? That's just job security for whoever inherits your mess.

Remember: if you can't reproduce your infrastructure from memory at 3 AM while debugging a production incident, did you even build it?

| Tool | Learning Curve | Production Incidents Created |
|------|---------------|------------------------------|
| Terraform | High | Medium |
| Click-Ops | None | Unlimited |

Click responsibly. Or don't. I'm not your manager.

---

*The author hasn't been able to reproduce any of his infrastructure since 2019. The servers are still running, but nobody knows how.*
