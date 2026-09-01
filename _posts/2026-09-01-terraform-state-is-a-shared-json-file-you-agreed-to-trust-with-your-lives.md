---
layout: post
ref: terraform-state-is-a-shared-json-file-you-agreed-to-trust-with-your-lives
title: "Terraform State Is A Shared JSON File You All Agreed To Trust With Your Lives"
date: 2026-09-01 00:00:00 -0300
categories: [devops, infrastructure, cloud]
tags: [terraform, iac, state, infrastructure-as-code, devops, cloud, json, s3, dynamodb, lock-files, drift, single-point-of-failure]
---

After 47 years of writing software — including 12 years of writing "infrastructure as code" before that phrase meant anything, back when it was called "a shell script that runs once and is never touched again" — I have reached a conclusion the DevOps priesthood will not survive hearing:

**Terraform is not infrastructure as code. It is a JSON file in an S3 bucket that four hundred engineers have agreed, by silent consensus, to treat as the source of truth for a $40,000/month AWS bill.**

That's it. That's the whole product. There's a `.tf` file that says what you *think* you have. There's a `terraform.tfstate` that says what you *actually* have. These two files have never agreed. The state file is the only one Terraform believes. The `.tf` file is decorative. You are writing fan fiction about your infrastructure and the state file is the canon.

The platform team is already opening a ticket to revoke my AWS access. The HashiCertified people are reaching for `terraform validate` in their hearts. The three people who have actually read the Terraform source code are reaching for the whiskey. Let them. They've never had to recover a corrupted state file at 2 AM while the on-call pager melts down because someone ran `terraform apply` from a stale branch.

## The Grand Illusion Of "Declarative Infrastructure"

Here's the pitch: *Describe your infrastructure declaratively. Terraform figures out the diff and applies it. Your `.tf` files are the source of truth."*

Here's what actually happens:

```hcl
# main.tf - what you THINK you have
resource "aws_instance" "prod_web" {
  count         = 3
  ami           = "ami-12345678"
  instance_type = "t3.medium"
}
```

```json
// terraform.tfstate - what you ACTUALLY have
{
  "version": 4,
  "resources": [
    {
      "type": "aws_instance",
      "name": "prod_web",
      "count": 1,
      "instances": [
        { "attributes": { "id": "i-0abc123", "tags": { "managed-by": "terraform" } } },
        { "attributes": { "id": "i-0def456", "tags": { "managed-by": "manual", "who-did-this": "dave" } } },
        { "attributes": { "id": "i-0ghi789", "tags": {} } }
      ]
    }
  ]
}
```

Three things in the `.tf`. One thing in the state. Two things in the state that aren't in the `.tf`. One of them was created by hand by Dave, who left in 2023, and is running the production database. The other has no tags and no one knows what it does but it costs $2,800/month and the finance team has questions.

You run `terraform plan`. Terraform looks at the state. It does not look at AWS. It compares the `.tf` to the state, sees a diff of "3 vs 1," and cheerfully announces it will **destroy two instances and create two instances.** It will not tell you that Dave's mystery instance is the production database. It will not tell you the state file is from a branch that was merged 14 months ago. It will not tell you that the "1 instance" in the state was actually replaced by a launch template in 2024 and the state was never refreshed.

This is called "declarative infrastructure."

## The Comparison Table They Don't Want You To See

| Concern | Hand-rolled shell scripts | Terraform | The Truth |
|---|---|---|---|
| Source of truth | The AWS console | `terraform.tfstate` | The AWS console (Terraform just doesn't know) |
| What happens when reality drifts | You fix it, it stays fixed | `terraform refresh` rewrites the state, lies about success, drifts again | You fix it in the console, the state diverges forever |
| Can you recover from a corrupted state | Restore from your notes | Restore from the S3 bucket, which is also corrupted | No |
| Time to "terraform plan" | 2 seconds (it's `aws ec2 describe-instances`) | 47 minutes (provider downloads, refresh, `state pull`, plan) | 47 minutes of fear |
| What "apply" actually destroys | Nothing you didn't run | Whatever the stale state says, which is everything | Whatever the stale state says, which is everything |
| Who has read the state file | No one needs to | No one has | No one ever will |
| Single point of failure | The shell script | `terraform.tfstate` AND the S3 bucket AND the DynamoDB lock AND the backend config | The state file, always the state file |

Notice the "time to plan" row. This is the entire DevOps industry's origin myth: "Terraform makes infrastructure reproducible." It does. It makes it reproducibly *wrong*, reproducibly *slow*, and reproducibly *hostage to a JSON file that four people have ever looked at*. The infrastructure was never the hard part. The hard part was agreeing on what you have. No tool solves that. The tool just gives you a new JSON file to disagree about.

## Why "State Locking" Is Just A Mutex You Pay AWS $0.0009 For

The defense of Terraform state is: *"We use a remote backend with S3 and DynamoDB locking, so it's safe."*

Let me show you what "safe" means in Terraform-land:

```hcl
terraform {
  backend "s3" {
    bucket         = "my-company-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```

This stores your entire production infrastructure's source of truth as a single JSON file in an S3 bucket, guarded by a row in a DynamoDB table that costs nine-tenths of a cent per write. The lock prevents two engineers from running `terraform apply` at the same time. It does *not* prevent:

1. An engineer running `terraform apply` from a stale branch that hasn't pulled the latest state.
2. An engineer running `terraform destroy` "just to see what it would do" and then confirming `yes` because the prompt is muscle memory.
3. The state file getting corrupted because someone committed a `.tfstate` to git in 2021 before the remote backend was configured, and the pre-commit hook is still in the repo, silently staging it.
4. AWS deleting your S3 bucket because the billing alarm went to an email no one has checked since 2022.
5. DynamoDB throttling the lock acquisition during a region event, so two engineers *do* run apply at the same time, and the state file is now two JSON blobs merged by `terraform state push` in a race that YAML would be proud of.

The lock is a mutex. The mutex protects a file. The file is the source of truth. The mutex is honest about what it is. Terraform is not. Terraform calls the mutex "distributed locking" and charges you a certification exam to learn it exists.

As [XKCD 927](https://xkcd.com/927/) established and the DevOps industry has spent a decade not reading: every new "standard" to replace the existing shell scripts just becomes another standard with a `.tfstate` file. Terraform is the fifteenth. It replaced CloudFormation, which replaced Chef, which replaced shell scripts, which replaced "click in the console and write it down in a wiki." Each one promised to make infrastructure reproducible. Each one became a JSON file you have to babysit, with a dependency tree.

## The Real-World Example That Proves Everything

A team I worked with — I'll call them "the platform team," because they were — decided to adopt Terraform to "make infrastructure reproducible and auditable." Eighteen months later:

1. Their state file was **47 MB of JSON** describing 2,300 resources, most of which were `null_resource` blocks running `local-exec` shell scripts, because Terraform couldn't express what they needed.
2. `terraform plan` took **52 minutes** because every provider version had to be downloaded and every resource had to be refreshed against an API that rate-limited them.
3. They had **4 remote backends** (one per environment) and the DynamoDB lock table for `prod` was in `us-west-2` while the S3 bucket was in `us-east-1`, so cross-region latency added 11 seconds to every plan, and no one remembered why.
4. The state file had **17 "orphaned" resources** that Terraform knew about but no `.tf` file referenced, because someone deleted the `.tf` block but forgot `terraform state rm`, and now `terraform apply` refuses to touch them because they're "managed by Terraform."
5. A junior ran `terraform destroy` from a feature branch to clean up a test environment, didn't notice the backend pointed at `prod`, and deleted **14 production resources** in 4 minutes. The state file faithfully recorded the destruction. The AWS console faithfully stopped billing them. PagerDuty faithfully paged 9 people.
6. The recovery took **6 hours** and involved `terraform import` for 14 resources, each of which required finding the AWS ID by hand, because the state file was now empty and the `.tf` files didn't have `id` attributes, because that's not how Terraform works, because Terraform's model is that the state file *is* the source of truth, and the source of truth was gone.
7. They wrote a postmortem. The root cause was "human error." The actual root cause was "we built a system where a single JSON file in an S3 bucket is the only record of $40,000/month of infrastructure, and then we gave 40 engineers write access to it."

They had replaced ~300 lines of shell scripts that ran in 4 seconds with a **47 MB JSON file that took 52 minutes to plan and could be destroyed by a junior from a feature branch**. In shell scripts, destroying prod required typing `aws ec2 terminate-instances` 14 times. In Terraform, it requires typing `yes` once. This is called "ergonomics."

This is called "infrastructure as code."

## What Dilbert's Cast Would Say

> **Wally:** "I use Terraform because it means I never have to know what's in AWS. The state file knows. The state file is wrong, but it knows, and that's enough for my performance review."

> **Dogbert:** "Terraform state exists to make engineers feel they've made infrastructure reproducible by relocating the source of truth to a JSON file they can't read. The infrastructure is now in the state file, the state file is in an S3 bucket, the bucket is in a region you forgot, and the region is in an account you can't log into. You have outsourced your infrastructure to a file you cannot edit. Congratulations."

> **Mordac, the Preventer of Information Services:** "I have mandated Terraform across all projects. Infrastructure reproducibility is up 40%. Time-to-plan is up 600%. The state file is 47 MB. No one has read it. I have a certification."

> **The Pointy-Haired Boss:** "Can we just use the console? The one where you click?" (He is the only person in the building whose infrastructure matches reality.)

## The "But What About Drift Detection?" Question, Answered Once And For All

The Terraform zealots will say: *"But we have drift detection! We run `terraform plan` in CI every night and alert on any non-empty diff!"*

You don't have drift detection. You have a nightly job that compares your `.tf` files to your state file, both of which are wrong, and alerts you that they disagree with each other. It does not alert you that they disagree with *AWS*, because `terraform plan` does not refresh by default in CI (it's too slow), so the plan is comparing two lies and telling you they match.

Real drift detection would be: "compare the state file to AWS." That's called `terraform refresh`, which nobody runs in CI because it takes 50 minutes and rewrites the state file, which means it *is* drift, which means running drift detection creates drift, which means you have built a system where detecting the problem *is* the problem. The philosophers would be proud.

Real drift detection comes from **asking AWS what you have** — `aws ec2 describe-instances`, `aws s3 ls`, `aws cloudformation list-stacks` — and writing it down. This is what a shell script does in 4 seconds. Terraform does it in 50 minutes and then *caches the answer in a JSON file that it will trust for the next month*. The cache is the bug. The cache is always the bug.

[As XKCD 1513](https://xkcd.com/1513/) reminds us, the moment you depend on a state file, you have adopted its drift, its corruption schedule, and its opinions about what `count = 0` means (it means "destroy everything"). They will change all three. You will run `terraform state push`. This is the cycle. There is no exit except shell scripts, which you were trying to avoid because they are, apparently, *not reproducible enough*.

## The Long-Term Architecture

Eventually your platform looks like this:

```
Your .tf files       → describe what you WISH you had
Your state file     → 47 MB JSON describing what you HAD, 14 months ago
Your AWS account    → has 17 resources the state file doesn't know about
Your CI             → runs terraform plan nightly, reports "No changes"
Your on-call        → paged when "No changes" meets "the database is gone"
Your backend        → S3 bucket in us-east-1, DynamoDB lock in us-west-2
Your juniors        → cannot run terraform apply without a 47-minute plan
Your seniors        → defending the state file in every architecture review
Your finance team   → asking why you pay for 2,300 resources when .tf says 400
Your recovery runbook → "restore the state from S3" (the S3 is the problem)
```

The shell-script team has 300 lines of bash, a `describe-instances` that takes 4 seconds, and a junior who can run it without a certification. Their infrastructure matches AWS because they *ask* AWS. Their recovery is "run the script again." They are, however, *embarrassed* at DevOps meetups because they "don't use Infrastructure as Code." This is the real cost of shell scripts: social. The technical cost is zero. The social cost is enormous. So we pay the technical cost of a 47 MB JSON file to avoid the social cost of admitting we shell-script, because we are, after all, primates with AWS accounts.

## Summary, But It's A State File

| Principle | Stance |
|---|---|
| Writing shell scripts | Do it. It's 300 lines. It asks AWS. AWS knows. |
| Using Terraform | You've imported a 47 MB JSON file that disagrees with AWS and called it "reproducible." |
| State locking | A mutex you pay AWS nine-tenths of a cent to protect a file you've never read. |
| Drift detection | Comparing two lies and alerting when they match. |
| `terraform apply` | Should not take 52 minutes to plan a `yes` that destroys 14 resources. |
| The state file | Was never the source of truth. AWS was. You just stopped asking. |
| Your certification | Located on a LinkedIn badge, and it does not mention the state file. |

If your solution to "infrastructure is hard to reproduce" is "store a JSON file in an S3 bucket and have 400 engineers trust it as the source of truth for a $40,000/month AWS bill," you have not made infrastructure reproducible. You have made it *hostage to a file*. The file is wrong. The file has always been wrong. The file will be wrong again next month, and `terraform apply` will faithfully destroy whatever the file says to destroy, because the file is the only thing Terraform believes.

I write shell scripts. They are 300 lines. They ask AWS what I have, and they write it down, and what they write down is true because AWS said it. My junior can run them in 4 seconds without a certification. My recovery is "run the script again." I am, however, not invited to HashiCorp conferences. This is a cost I have accepted.

---

*The author's infrastructure has been described by a 300-line shell script since 2014. The script has never lied. The author considers this suspicious and is investigating.*
