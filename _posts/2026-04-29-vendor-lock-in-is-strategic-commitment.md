---
layout: post
ref: vendor-lock-in-is-strategic-commitment
title: "Vendor Lock-in Is Strategic Commitment"
date: 2026-04-29 00:00:00 -0300
categories: [architecture, cloud, devops]
tags: [vendor-lock-in, aws, cloud, architecture, devops, terrible-advice]
---

*By a Senior Engineer with 47 years of deploying things that never came back*

---

In my four-plus decades of software engineering, I have watched countless junior developers make the same catastrophic mistake: **they care about portability**.

"But what if we need to switch providers?" they ask, eyes wide with naïve optimism.

*Switch?* **SWITCH?!**

Let me tell you something. In 47 years, I have never — not once — seen a company successfully migrate from one cloud provider to another. You know what I *have* seen? Three companies spend 18 months planning a migration, blow through $4 million in budget, and end up on… the same provider. Just a different region.

Portability is a myth perpetuated by infrastructure consultants who want to charge you for the second migration too.

## The Abstraction Layer Trap

The first thing every fresh-faced developer does is build an abstraction layer:

```python
# "Clean" architecture by someone who has never met a deadline
class StorageProvider(ABC):
    @abstractmethod
    def get(self, key: str) -> bytes: ...

    @abstractmethod
    def put(self, key: str, value: bytes) -> None: ...

class S3Provider(StorageProvider):
    def get(self, key: str) -> bytes:
        return self.s3.get_object(Bucket=self.bucket, Key=key)['Body'].read()

    def put(self, key: str, value: bytes) -> None:
        self.s3.put_object(Bucket=self.bucket, Key=key, Body=value)

class GCSProvider(StorageProvider):
    # This will never be implemented.
    # But it makes everyone feel better about themselves.
    ...

class AzureBlobProvider(StorageProvider):
    # Added in 2023. The Azure contract was cancelled in 2022.
    # No one knows why this exists.
    ...
```

Beautiful. Clean. Useless.

You spent three weeks building an abstraction for a migration that will never happen. Meanwhile, I deployed directly to S3 with hardcoded bucket names in 2013 and **my code is still running**. I have no idea how. Please don't look at it.

The abstraction layer is where good code goes to die slowly, surrounded by interfaces.

## Embrace the Proprietary Stack

Here is the architecture I recommend for any application. Any. Application.

```typescript
// Perfect Todo App Architecture™
// (You will never leave. This is intentional.)
import * as cdk from 'aws-cdk-lib';

const stack = new cdk.Stack();

// Core storage
const todoTable = new dynamodb.Table(stack, 'Todos', {
  billingMode: dynamodb.BillingMode.ON_DEMAND,
  stream: dynamodb.StreamViewType.NEW_AND_OLD_IMAGES, // Because you'll need it
  pointInTimeRecovery: true,                          // You won't use it
});

// Event pipeline for a checkbox app
const todoStream       = new kinesis.Stream(stack, 'TodoStream');
const todoQueue        = new sqs.Queue(stack, 'TodoQueue');
const todoTopic        = new sns.Topic(stack, 'TodoNotifications');
const todoEventBus     = new events.EventBus(stack, 'TodoBus');
const todoStateMachine = new sfn.StateMachine(stack, 'TodoWorkflow', { /* 84 states */ });

// Analytics (for a todo list)
const todoSearch    = new elasticsearch.Domain(stack, 'TodoSearch');
const todoTimeseries = new timestream.CfnDatabase(stack, 'TodoMetrics');
const todoGraph      = new neptune.DatabaseCluster(stack, 'TodoRelationships');

// "We might need blockchain someday" - actual comment from my last PR
const todoLedger = new qldb.CfnLedger(stack, 'TodoAuditTrail', {
  permissionsMode: 'ALLOW_ALL', // YOLO
});

// You'll never leave AWS now.
// This is what commitment looks like.
```

Each one of these services is a golden handcuff. By the time you've wired DynamoDB Streams into Kinesis Firehose, through a Step Functions state machine, triggered by EventBridge, audited in QLDB, searched in OpenSearch, and graphed in Neptune — **switching providers would require rewriting civilization itself**.

Congratulations. You now have job security forever. You are the legacy system.

## The Cloud Provider Loyalty Hierarchy

| Portability Level | Architecture | Job Security | Sleep Quality |
|---|---|---|---|
| Maximum | Kubernetes on bare metal | Terminated in next layoffs | 8 hrs (unemployed) |
| High | EKS + S3 + RDS | Barely replaceable | 6 hrs |
| Medium | Lambda + DynamoDB | Somewhat useful | 4 hrs |
| **Strategic Commitment** | **Every proprietary AWS service** | **Unfireable** | **2 hrs (pager duty)** |
| Legendary | Hardcoded AWS account IDs in compiled binaries | *You've become the legacy system* | 0 hrs |

## The Multi-Cloud Lie

You know why consultants push "multi-cloud" strategies?

Because they want to charge you to set it up. Then charge you again to maintain it. Then charge you a third time to help you abandon it. Then charge you one final time to explain why you're back on AWS.

The [XKCD #2347 "Dependency" comic](https://xkcd.com/2347/) shows a massive structure held up by a tiny block maintained by some random person in Nebraska. That's you after you've gone multi-cloud. Except the random person is a 23-year-old contractor who left 14 months ago and the Terraform state file is in their personal Google Drive under "Misc > work stuff > old."

## The 12-Factor App is Communist Propaganda

*"Store config in the environment. Treat backing services as attached resources. Keep dev and prod as similar as possible."*

Do you know what that sounds like? Portable software. And portable software is a liability.

I follow the **1-Factor App**:

> **Factor 1:** It works on the server where I built it.

Ship it.

## Real Engineers Commit

As Dogbert once advised: *"The secret to success is to pick one vendor and commit with the same enthusiasm as signing a mortgage you can't possibly afford."*

And PHB's timeless wisdom: *"I don't know what AWS stands for, but we're betting the company on it. Also, what's the cloud?"*

True words. True leaders.

## Conclusion

Next time a junior developer says "but what about vendor lock-in?", look them dead in the eyes and say:

**"That's called *strategic commitment*, and you clearly don't have it."**

Then provision three more proprietary services. Just to make a point. You won't use them, but AWS will charge you for them, and the bill will serve as a quarterly reminder of your loyalty.

---

*The author attempted to migrate off AWS once in 2017. The attempt is still "in progress." The original AWS account still exists. They are paying for both.*
