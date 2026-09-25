---
layout: post
ref: your-terraform-modules-are-just-functions-you-cant-step-through
title: "Your Terraform Modules Are Just Functions You Can't Step Through"
date: 2026-09-25 00:00:00 -0300
categories: [devops, infrastructure, cloud]
tags: [terraform, iac, modules, infrastructure-as-code, devops, cloud, abstraction, debugging, hashicorp, reusable, copy-paste, versioning]
---

After 47 years of writing software — including the 8 years I spent watching the platform team "build a reusable module library" that ended up as 41 forks of the same VPC module, each with a different typo in the `tags` map — I have a finding that the Infrastructure-as-Code priesthood will attempt to suppress:

**A Terraform module is a function you cannot step through, cannot unit test, cannot mock, cannot breakpoint, and cannot read the stack trace of. It is the only "abstraction" in computing where the only way to find out what it does is to run it against a real AWS account and read the 900-line plan output backwards. The software industry spent 60 years inventing the debugger. Terraform reinvented the function call and forgot to bring it.**

The HashiCertified are already composing a Medium post titled "Why Modules Are The Functions Of Infrastructure." Let me save them the draft: they are. That's the problem. Functions are good because you can *see inside them*. Modules are functions with the lights off.

## The Pitch, And Then The Reality

Here is the pitch the platform team gives in the architecture review:

> *"We're building a library of reusable modules. Each module is a self-contained unit of infrastructure. Teams compose them. We get consistency, DRY, and a single source of truth. It's just like a function library."*

Here is the reality, six months later:

```hcl
# modules/vpc/main.tf - "reusable"
variable "cidr"              { type = string }
variable "name"              { type = string }
variable "enable_nat"        { type = bool }
variable "azs"               { type = list(string) }
variable "single_nat"       { type = bool }
variable "one_nat_per_az"    { type = bool }
variable "enable_dns"        { type = bool }
variable "enable_ipv6"      { type = bool }
variable "public_tags"       { type = map(string) }
variable "private_tags"      { type = map(string) }
variable "intra_tags"        { type = map(string) }   # nobody knows what "intra" means
variable "database_subnets" { type = bool }
variable "create_database"   { type = bool, default = false }  # contradicts the above
variable "flow_log_role"     { type = string, default = "" }    # ignored if empty, errors if set
# ... 47 more variables, 3 of which are deprecated but still required by the API
```

This is a function signature. It takes 57 arguments. Three of them contradict each other. One of them (`intra_tags`) refers to a subnet type that was removed from the AWS docs in 2022 but the module still provisions it because someone in 2020 wrote a `for_each` over it and no one has been brave enough to delete the loop. You cannot tell which arguments are required without running `terraform validate`, which passes even when the arguments contradict, because `validate` checks syntax, not sanity.

In a real programming language, a function with 57 parameters and three contradictions would be flagged by every linter on Earth. In Terraform, it is called "flexible." The platform team puts it in the internal registry. Fourteen teams depend on it. The function cannot be renamed, removed, or refactored, because Terraform has no concept of a deprecation cycle — there is no caller to migrate, only a `terraform init --upgrade` that silently breaks everyone on a Tuesday.

## The Comparison Table They Forgot To Put In The Docs

| Concern | A real function (Python, Go, etc.) | A Terraform module | The Truth |
|---|---|---|---|
| Can you step through it in a debugger? | Yes | No | No. The "debugger" is `terraform plan` and a prayer. |
| Can you unit test it? | Yes | "Yes, with Terratest" ( Terratest runs it against real AWS for 9 minutes and calls it a unit test) | No. |
| Can you mock its dependencies? | Yes | No (providers are real, `mock_provider` is a 2023 fever dream that works for 3 resource types) | No. |
| Can you read a stack trace when it fails? | Yes | No — you get "Error: unsupported attribute" on line 0 of an unknown file in the module cache | No. |
| What does a parameter type error look like? | A clear message naming the parameter | "Invalid value for attribute" pointing at a `.tf` file three directories up, with no mention of which variable | A riddle. |
| Can you see what it will do before it does it? | Yes, you read the body | `terraform plan`, which takes 7 minutes and only tells you after it's consulted real AWS | Sort of, but slowly and in past tense. |
| Can you grep the implementation? | Yes | Only if you've run `terraform init` and downloaded it from the registry into `.terraform/modules`, which is gitignored | No. |
| How do you find out what it actually does? | Read it | Run it against a sandbox account and read the plan | You run it. That's the only way. |
| How do you version it? | Semver, a changelog, migration notes | `source = "git::https://...//modules/vpc?ref=v3.2.1"` and hope | You hope. |
| What happens when you upgrade the version? | Migration notes, deprecation warnings | Silently renames a resource, destroying and recreating it, because someone changed a `for_each` key | Production downtime, billed as "improvements." |

Look at row 8. This is the whole scam. In a real language, "what does this function do?" is a question you answer by *reading the function*. In Terraform, the answer is "run it and read the 900-line plan, then re-read it because you skimmed past the `+` on the NAT Gateway that's about to be replaced and you'll only find out when the bill arrives." The function body is HCL. HCL is declarative. Declarative means "do not read me, just trust the engine." The engine trusts the provider. The provider trusts AWS. You trust the plan output. The plan output is 900 lines. You read 12 of them. This is how a 3-line variable rename becomes a 4-hour outage.

## Why "Terratest Is Unit Testing For Modules" Is A Lie I Will Not Tolerate

The platform team, having discovered that modules cannot be debugged, will reach for Terratest. Terratest is a Go library that runs `terraform apply` against a real AWS account, waits for the resources to exist, runs some `aws ec2 describe-...` calls to check they exist, and then runs `terraform destroy`. They call this "unit testing." Let me show you what a "unit test" for a module looks like:

```go
func TestVpcModule(t *testing.T) {
    opts := &terraform.Options{
        TerraformDir: "../examples/basic",
    }
    defer terraform.Destroy(t, opts)         // runs in cleanup, takes 9 minutes
    terraform.InitAndApply(t, opts)          // takes 11 minutes, costs $0.40
    vpcId := terraform.Output(t, opts, "vpc_id")
    assert.NotEmpty(t, vpcId)                 // the actual assertion
}
```

This is not a unit test. A unit test runs in milliseconds, costs nothing, and isolates the unit from its dependencies. This "test" runs for 20 minutes, costs real money, requires AWS credentials in CI, fails when AWS rate-limits the test, fails when the region is in a degraded state, and its sole assertion is that a string is non-empty. The word "unit" is doing no work here. The word "test" is doing very little. The thing being tested is "did Terraform, the AWS API, the provider, the IAM role, the CI network, and the region's capacity all cooperate for 20 minutes." If any of them had a bad day, your "unit test" flakes. You will then add a `t.Skip()` with the comment `// flaky, investigate later`. No one will investigate later. There is a graveyard of `t.Skip()` comments in every Terratest suite, each one a tombstone for a real bug that someone chose not to look at because the test was "flaky," which is DevOps for "I do not want to know."

[As XKCD 2173](https://xkcd.com/2173/) diagnosed, the moment your "unit test" depends on an external service, it is an integration test wearing a unit test's clothes, and it will fail at 3 AM for reasons that have nothing to do with your code. Terratest did not fix this. Terratest industrialized it.

## The Real-World Example That Proves Everything

A team I worked with — I'll call them "the platform team," because they were — decided to "build a reusable module library" so that "every team builds infrastructure the same way." Fourteen months later:

1. They had **23 modules** in the internal registry, of which **19 were forks of the AWS-maintained `vpc` module**, each with a different hardcoded tag like `Owner = "platform"` because the variable was added after the fork and no one backported it.
2. The "canonical" VPC module had **4 versions in active use** (`v1`, `v2`, `v3`, and `v3.2.1-fork`), and the platform team had a spreadsheet tracking which team used which version, which they updated by hand because Terraform has no `list consumers of a module` command, because the consumers are `source = "..."` strings in 41 repos Terraform does not index.
3. A team upgraded from `v2` to `v3` by changing one `?ref=v2` to `?ref=v3` in their `source` string. `terraform plan` reported "No changes" because `v3` had renamed `tags` to `tag_map` but kept the old variable as a no-op for "backward compatibility," so the plan looked identical, and `terraform apply` then **destroyed and recreated every subnet** because the underlying `for_each` key had changed from `var.tags` to `var.tag_map`. The plan output mentioned this in line 847. No one read line 847. The VPC dropped for 6 minutes. The NAT Gateway was replaced. The bill for the new NAT Gateway was $32. The bill for the "old" NAT Gateway, which AWS kept charging for an hour because it was "released" not "deleted," was $0.40. No one refunded it.
4. The postmortem's root cause was "insufficient plan review." The actual root cause was "the only way to know what `v3` does is to read 2,000 lines of HCL across 4 files in a git ref, and the reviewer read 12 lines of the plan output instead, because the plan output is 900 lines and the diff is in line 847 and humans are not built for this." The fix in the action items was "reviewers must read the full plan." No one reads the full plan. The full plan is 900 lines. This action item will be open in the next postmortem too.
5. They added a "module versioning policy": all modules must use semver. Semver for a Terraform module means "the maintainer incremented a number." A patch bump (`v3.2.1` → `v3.2.2`) silently changed a `count` to a `for_each`, which is a destruction-and-recreation of every resource in the module. Semver has no field for "this patch rewrites your infrastructure." Semver assumes you can read the changelog. The changelog says "internal refactor." The "internal refactor" deleted your VPC. Semver did not warn you. Semver cannot warn you. Semver is about *API* compatibility, and Terraform's API is "whatever the plan output says," which is unreadable, which makes semver a number you trust because the alternative is reading 2,000 lines of HCL, which you will not do, which is the whole problem.

They had replaced "14 teams each writing 30 lines of `aws_vpc` resources that they understood" with "19 forks of a 2,000-line module that no one understood, versioned by a spreadsheet, debugged by reading line 847 of a plan output." In the old world, "what does this VPC do?" was answered by reading 30 lines. In the new world, it is answered by running `terraform plan` for 7 minutes and reading 900 lines. This is called "abstraction." Abstraction is supposed to *hide* complexity. Terraform modules *relocate* complexity — from your repo, where you could read it, to the plan output, where you cannot.

## What Dilbert's Cast Would Say

> **Wally:** "I depend on the platform VPC module because I don't know what a VPC is. The module doesn't know either. The plan output is 900 lines. I read none of them. I type `yes`. So far, so good. The 'so far' is doing a lot of work."

> **Dogbert:** "A Terraform module is an abstraction with no implementation visible, no stack trace, no debugger, and no way to know what it does except to run it against a real cloud and read a thousand-line diff. You have reinvented the function call and removed its only feature. This is the most impressive act of subtraction since someone invented decaf."

> **Mordac, the Preventer of Information Services:** "All teams must use the canonical modules from the internal registry. Consistency is up 30%. The registry has 19 forks of the VPC module. I do not know which one is canonical. Neither does the platform team. I have a certification in 'Module Governance.' It does not mention the 19 forks."

> **The Pointy-Haired Boss:** "Can the module just... do what it says? Like a function? In a file? That I can read?" (He is, again, the only person in the building whose mental model of the system is correct, because it is the only one that is simpler than the system.)

## The "But What About `terraform plan -target`?" Question, Answered Once And For All

The zealots will say: *"But you can scope your investigation with `terraform plan -target=module.vpc`! That narrows the plan!"*

Let me show you what `-target` does. It narrows the plan to the module *and its dependencies*. It does not tell you which dependencies. It does not tell you why `module.vpc` depends on `module.iam_role`, which depends on `data.aws_caller_identity`, which depends on the provider config, which depends on the backend, which depends on the state file, which depends on S3. The dependency graph is not documented. It is *inferred* at plan time and printed as a tree you have to read backwards, from the leaf that failed to the root that caused it. There is no `terraform deps module.vpc` command that prints the graph without running a plan. There is only the plan, which takes 7 minutes. The dependency graph is not a graph you can inspect; it is a graph you can *experience*, once, slowly, and then forget.

Real functions have dependency graphs you can read: `import`s at the top of the file, `go list -deps`, `pip show`, an IDE that draws the call tree. Terraform modules have a dependency graph that exists only in the moment of `terraform plan` and is gone the instant it finishes, like a firework of consequences. You cannot version it. You cannot diff it between versions. You cannot ask "what changed in the dependency graph between `v2` and `v3`?" You can only run both plans, diff the 900-line outputs, and try to spot which `+` is the one that destroys your VPC. It is in line 847. You will not spot it. The postmortem will say "insufficient plan review." It will be correct.

[As XKCD 1597](https://xkcd.com/1597/) warned, any sufficiently advanced dependency graph is indistinguishable from a 900-line plan output nobody reads. Terraform's graph is advanced. The plan output is unread. The consequences are billed in 6-minute intervals.

## The Long-Term Architecture

Eventually your module ecosystem looks like this:

```
Your "canonical" modules  → 23 modules, 19 are forks of aws/vpc
Your version tracking       → a spreadsheet a platform engineer updates by hand
Your "unit tests"           → Terratest, 20 min each, $0.40 each, 40% flaky, 30% t.Skip()
Your debugging tool         → terraform plan, 7 min, 900 lines, the bug is in line 847
Your dependency graph       → inferred at plan time, not stored, not diffable
Your changelog              → "internal refactor" (this destroyed a VPC last week)
Your semver                 → a number a maintainer incremented; warns you about nothing
Your reviewers              → read 12 of 900 plan lines; the action item says "read all 900"
Your "abstraction"          → 2,000 lines of HCL that relocated complexity, not hid it
Your juniors               → cannot read the module; can only run it; fear the plan output
Your seniors               → defending the module library in every architecture review
Your finance team           → asking why you pay for a NAT Gateway twice during a "patch" upgrade
Your recovery runbook       → "revert the ?ref=, run terraform apply, read line 847"
```

The team that just writes `aws_vpc` resources directly in their repo — 30 lines, no module, no registry, no spreadsheet, no Terratest — has a VPC they can read, a plan they can finish reading, and a junior who knows what a subnet is because they typed the block themselves. They are, however, "not using the canonical module," which means they are not "DRY," which means the platform team has a Jira ticket about them. This is the real cost of writing 30 lines of HCL: a Jira ticket. The technical cost is zero. The political cost is a recurring meeting. So the team adopts the module, joins the spreadsheet, and starts reading line 847. Everyone is now "consistent." Consistency, in Terraform, means "equally unable to read the plan." This is the victory the platform team celebrates at the quarterly review.

## Summary, But It's A Function Call

| Principle | Stance |
|---|---|
| Writing `aws_vpc` directly in your repo | Do it. It's 30 lines. You can read it. Your junior can read it. The plan is 90 lines. You can finish reading it. |
| Using a "reusable module" | You have imported a 2,000-line function you cannot read, debug, step through, or unit test, and called it "abstraction." |
| Terratest | A 20-minute, $0.40 integration test wearing a unit test's name badge. 30% of them are `t.Skip()`. |
| `-target` | Narrows the plan; does not narrow the dependency graph, because the graph only exists during the plan. |
| Module semver | A number a maintainer incremented. It does not warn you about `for_each` rewrites. Nothing does. Line 847 does. |
| The module registry | A spreadsheet plus a git ref string, indexed by nothing, searched by hope. |
| `terraform plan` | Should not take 7 minutes to answer "what does this function do." A real function answers in 0 ms. |
| Your certification | Does not mention line 847. It should. |

If your solution to "teams write slightly different VPCs" is "replace 30 readable lines with a 2,000-line function that cannot be debugged, versioned by a spreadsheet, tested by a 20-minute AWS call, and reviewed by reading 12 of 900 plan lines," you have not made infrastructure DRY. You have made it *illegible*. The complexity was never reduced. It was moved — from the repo, where a junior could read it, to the plan output, where no one can. The junior now types `yes` instead of writing a subnet. The `yes` is the abstraction. The abstraction is a 900-line diff you will not read. The bug is in line 847. You will not find it until the bill arrives.

I write `aws_vpc` directly in my repo. It is 30 lines. My junior reads them in 2 minutes. My plan is 90 lines. I read all 90. My VPC has never been destroyed by a "patch" upgrade, because there is no module to upgrade, because there is no version to bump, because the code is *right there*. I am, however, "not using the canonical module." The platform team has opened a ticket. I will attend the recurring meeting. This is a cost I have accepted.

---

*The author has written the same 30-line VPC in 14 repos. The platform team calls this "duplication." The author calls it "readable." The VPC has never been destroyed by a semver bump. The author considers this the only metric that matters.*
