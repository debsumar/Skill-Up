---
title: "S3 Event Notifications"
date: 2026-02-10
tags:
  - aws
  - s3
  - event-notifications
  - eventbridge
  - sns
  - sqs
  - lambda
  - saa-c03
---

# S3 Event Notifications

## Overview

S3 Event Notifications let you ==automatically react to events== happening in your S3 buckets. When an object is created, deleted, restored, or replicated, S3 can send a notification to a target service that processes the event. This is the foundation of ==event-driven architectures== built on top of S3.

The classic use case: a user uploads an image to S3 → S3 sends an event notification → a Lambda function generates a thumbnail → the thumbnail is stored back in S3. All ==fully automated==, no polling required.

Events are typically delivered ==within seconds==, but sometimes it can take a minute or longer. You can create ==as many event notifications as desired== on a single bucket, each with different event types, filters, and destinations.

## Event Types

S3 can generate notifications for many event types. You choose which events to react to:

| Event Category | Event Names | When It Fires |
|---------------|-------------|---------------|
| ==Object Created== | `s3:ObjectCreated:Put`, `s3:ObjectCreated:Post`, `s3:ObjectCreated:Copy`, `s3:ObjectCreated:CompleteMultipartUpload` | Any time an object is added to the bucket |
| ==Object Removed== | `s3:ObjectRemoved:Delete`, `s3:ObjectRemoved:DeleteMarkerCreated` | Any time an object is deleted (or a delete marker is created in versioned buckets) |
| ==Object Restored== | `s3:ObjectRestore:Post`, `s3:ObjectRestore:Completed` | When a Glacier restore is initiated or completed |
| ==Replication== | `s3:Replication:OperationFailedReplication`, `s3:Replication:OperationMissedThreshold` | When cross-region or same-region replication fails or misses SLA |

You can also use ==wildcard events== like `s3:ObjectCreated:*` to catch all creation events regardless of the method (Put, Post, Copy, or MultipartUpload).

### Filtering by Prefix and Suffix

You can narrow down which objects trigger notifications:

| Filter | Example | Effect |
|--------|---------|--------|
| ==Prefix== | `uploads/` | Only objects in the `uploads/` folder trigger events |
| ==Suffix== | `.jpg` | Only objects ending with `.jpg` trigger events |
| ==Both== | Prefix `images/` + Suffix `.png` | Only PNG files in the `images/` folder |

This is useful when you only want to process certain types of files. For example, trigger thumbnail generation only for `.jpg` and `.png` uploads, not for `.csv` data files.

## Four Notification Destinations

```
┌──────────────────────────────────────────────────────────────┐
│                S3 EVENT NOTIFICATION TARGETS                  │
│                                                               │
│                    ┌──────────────────────────────────┐      │
│                    │  1. SNS Topic                    │      │
│                    │     Fan-out to subscribers       │      │
│                    └──────────────────────────────────┘      │
│  ┌──────────┐     ┌──────────────────────────────────┐      │
│  │          │────▶│  2. SQS Queue                    │      │
│  │  Amazon  │     │     Decouple & buffer processing │      │
│  │    S3    │     └──────────────────────────────────┘      │
│  │  Event   │     ┌──────────────────────────────────┐      │
│  │          │────▶│  3. Lambda Function              │      │
│  │          │     │     Serverless processing        │      │
│  └──────────┘     └──────────────────────────────────┘      │
│       │           ┌──────────────────────────────────┐      │
│       └──────────▶│  4. Amazon EventBridge           │      │
│    (all events)   │     18+ destinations, advanced   │      │
│                    │     filtering, archive, replay   │      │
│                    └──────────────────────────────────┘      │
└──────────────────────────────────────────────────────────────┘
```

### Traditional Targets: SNS, SQS, Lambda

| Target | Use Case | How It Works | Example |
|--------|----------|-------------|---------|
| ==SNS Topic== | Fan-out to multiple subscribers | S3 publishes message to SNS topic → SNS delivers to all subscribers (email, HTTP, SQS, Lambda) | Notify multiple teams when a file is uploaded |
| ==SQS Queue== | Decouple processing, buffer events | S3 sends message to SQS queue → consumer polls and processes | Queue image processing jobs for a worker fleet |
| ==Lambda Function== | Serverless processing | S3 invokes Lambda directly with the event payload | Generate thumbnails, extract metadata, scan for viruses |

### When to Use Which Traditional Target

| Scenario | Best Target | Why |
|----------|------------|-----|
| Process each event with custom code | ==Lambda== | Serverless, scales automatically, no infrastructure |
| Buffer events for batch processing | ==SQS== | Decouples producer from consumer, handles bursts |
| Notify multiple downstream systems | ==SNS== | Fan-out to multiple subscribers simultaneously |
| Process events AND notify teams | ==SNS → SQS + Lambda== | SNS fans out to both SQS (for processing) and email (for notification) |

### EventBridge Integration

EventBridge is the ==newer, more powerful== option. When enabled, ==all S3 events== are automatically sent to EventBridge (not just the ones you configure). From there, you create rules to route events to destinations:

```
┌──────────┐   ALL events    ┌──────────────┐   Rules    ┌──────────────────┐
│  Amazon  │────────────────▶│  Amazon      │───────────▶│  18+ AWS Services│
│    S3    │                 │  EventBridge │            │                  │
│          │                 │              │            │  - Step Functions │
│  Every   │                 │  ┌────────┐  │            │  - Kinesis Streams│
│  event   │                 │  │ Rule 1 │──┼───────────▶│  - Firehose      │
│  goes    │                 │  ├────────┤  │            │  - Lambda        │
│  here    │                 │  │ Rule 2 │──┼───────────▶│  - SQS, SNS      │
│          │                 │  ├────────┤  │            │  - ECS Tasks     │
│          │                 │  │ Rule 3 │──┼───────────▶│  - API Gateway   │
│          │                 │  └────────┘  │            │  - and more...   │
└──────────┘                 └──────────────┘            └──────────────────┘
```

| Feature | Traditional (SNS/SQS/Lambda) | EventBridge |
|---------|------------------------------|-------------|
| **Destinations** | 3 services only | ==18+ AWS services== |
| **Filtering** | Prefix and suffix only | ==Advanced== — metadata, object size, name, any JSON field |
| **Multiple destinations** | One destination per notification config | ==Multiple rules, each with multiple targets== |
| **Archive & Replay** | No | ==Yes== — archive events and replay them for debugging or reprocessing |
| **Reliability** | Good (at-least-once delivery) | ==Better== — built-in retry with exponential backoff and dead-letter queues |
| **Setup** | Configure per event type | Enable once → ==all events== flow to EventBridge |

> [!tip] Exam Tip — When to Choose EventBridge
> If the exam mentions any of these, the answer is ==Amazon EventBridge==:
> - "Send S3 events to ==Step Functions=="
> - "==Archive== S3 events for replay"
> - "==Advanced filtering== on S3 events" (by metadata, object size, etc.)
> - "Send to ==more than 3 destinations=="
> - "==Replay== failed events"
> For simple SNS/SQS/Lambda targets with prefix/suffix filtering, traditional event notifications work fine.

## IAM: Resource Access Policies (Not IAM Roles)

This is a ==critical distinction== for the exam. S3 Event Notifications use ==resource-based policies== on the target service to authorize S3 to send events. You do NOT attach an IAM role to S3.

```
┌──────────────────────────────────────────────────────────────┐
│              RESOURCE ACCESS POLICIES                         │
│                                                               │
│  S3 Bucket ──▶ SNS Topic                                     │
│                └── SNS Resource Policy:                       │
│                    "Allow s3.amazonaws.com to sns:Publish"    │
│                                                               │
│  S3 Bucket ──▶ SQS Queue                                     │
│                └── SQS Resource Policy:                       │
│                    "Allow s3.amazonaws.com to sqs:SendMessage"│
│                                                               │
│  S3 Bucket ──▶ Lambda Function                                │
│                └── Lambda Resource Policy:                    │
│                    "Allow s3.amazonaws.com to lambda:Invoke"  │
│                                                               │
│  ⚠️  NOT IAM roles — RESOURCE-BASED policies on the TARGET!  │
└──────────────────────────────────────────────────────────────┘
```

These resource-based policies work similarly to ==S3 bucket policies== — they define who (which AWS service or principal) can perform which actions on the resource. The key difference is that the policy lives on the ==target== (SQS queue, SNS topic, Lambda function), not on S3.

> [!warning] Common Hands-On Error
> If you create an event notification ==before== setting up the resource policy on the target (SQS/SNS/Lambda), you'll get this error:
> =="Unable to validate the destination configuration."==
> S3 sends a ==test event== when you create the notification to verify connectivity. If the target doesn't have a policy allowing S3 to write to it, the validation fails. Always ==create the target and its policy first==, then configure the event notification.

## Hands-On: S3 → SQS Event Notification

### Step-by-Step Walkthrough

#### 1. Create an S3 Bucket

Create a new bucket (e.g., `my-events-demo-bucket`) in your preferred region. Default settings are fine.

#### 2. Create an SQS Queue

Go to Amazon SQS → Create queue → Name it `DemoS3Notification` → Create with default settings.

#### 3. Update SQS Access Policy

This is the ==critical step== that most people miss. Go to the SQS queue → Edit → Access Policy. You need to allow S3 to send messages to this queue.

Use the ==AWS Policy Generator==:
- Policy Type: ==SQS Queue Policy==
- Effect: ==Allow==
- Principal: ==*== (any principal — permissive for demo purposes)
- Action: ==SQS:SendMessage==
- ARN: paste the SQS queue ARN

The generated policy looks like:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "SQS:SendMessage",
      "Resource": "arn:aws:sqs:eu-west-1:123456789012:DemoS3Notification"
    }
  ]
}
```

> [!warning] Production Best Practice
> The policy above uses `"Principal": "*"` which allows ==anyone== to send messages. In production, restrict the principal to the S3 service and your specific bucket ARN using a ==condition==:
> ```json
> "Condition": {
>   "ArnLike": {
>     "aws:SourceArn": "arn:aws:s3:::my-bucket-name"
>   }
> }
> ```

#### 4. Create Event Notification on S3

Go to S3 bucket → ==Properties== → scroll to ==Event Notifications== → Create event notification:
- Name: `DemoEventNotification`
- Event types: ==All object create events== (`s3:ObjectCreated:*`)
- Destination: ==SQS Queue== → select `DemoS3Notification`
- Save changes — this time it ==succeeds== because the SQS policy allows S3

S3 sends a ==test event== to verify connectivity. You'll see a test message in the SQS queue.

#### 5. Upload a File and Verify

Upload any file to the S3 bucket (e.g., `coffee.jpg`). Then go to SQS → your queue → ==Send and receive messages== → ==Poll for messages==.

You'll see a message with the event payload:

```json
{
  "Records": [
    {
      "eventVersion": "2.1",
      "eventSource": "aws:s3",
      "eventName": "ObjectCreated:Put",
      "s3": {
        "bucket": {
          "name": "my-events-demo-bucket"
        },
        "object": {
          "key": "coffee.jpg",
          "size": 12345
        }
      }
    }
  ]
}
```

The `eventName` is `ObjectCreated:Put` and the `key` is `coffee.jpg` — confirming the event notification is working.

#### 6. Enable EventBridge (Optional)

In the same Properties section, under ==Amazon EventBridge==, click Edit → set to ==On==. Once enabled, ==all events== from this bucket are automatically sent to EventBridge. You can then create EventBridge rules to route events to any of 18+ AWS services.

## Event Delivery Characteristics

| Aspect | Detail |
|--------|--------|
| **Delivery time** | Typically ==within seconds==, but can take a minute or longer |
| **Ordering** | ==Not guaranteed== — events may arrive out of order |
| **Duplicates** | ==Possible== — design your processing to be idempotent |
| **At-least-once** | S3 guarantees ==at-least-once delivery== — you may get the same event twice |
| **Versioning** | Enable versioning to ensure events for ==every object write== (without versioning, rapid overwrites may only generate one event) |

> [!important] Design for Idempotency
> Because S3 event notifications provide ==at-least-once delivery==, your event processing code must be ==idempotent== — safe to process the same event multiple times without side effects. For example, if your Lambda generates a thumbnail, it should overwrite the existing thumbnail rather than creating duplicates.

## S3 Event Notifications vs EventBridge — Decision Guide

```
┌─────────────────────────────────────────────────────────────────┐
│  Do you need advanced filtering (metadata, size, JSON fields)? │
│  ├── YES ──▶ Use EventBridge                                   │
│  └── NO                                                        │
│      │                                                         │
│      Do you need more than SNS/SQS/Lambda as destinations?     │
│      ├── YES ──▶ Use EventBridge                               │
│      └── NO                                                    │
│          │                                                     │
│          Do you need to archive/replay events?                 │
│          ├── YES ──▶ Use EventBridge                           │
│          └── NO ──▶ Use Traditional Event Notifications        │
│                     (simpler setup, lower overhead)             │
└─────────────────────────────────────────────────────────────────┘
```

## Questions & Answers

> [!question]- Q1: What are the four S3 Event Notification destinations?
> **Answer:**
> 1. ==SNS Topic== — fan-out to multiple subscribers (email, HTTP, SQS, Lambda)
> 2. ==SQS Queue== — decouple and buffer event processing for consumer applications
> 3. ==Lambda Function== — serverless event processing (thumbnails, metadata extraction, virus scanning)
> 4. ==Amazon EventBridge== — advanced filtering, 18+ destinations, archive/replay, multiple rules
> The first three are "traditional" targets configured per event notification. EventBridge is the newer, more powerful option that receives ==all events== once enabled.

> [!question]- Q2: What is the advantage of EventBridge over traditional S3 event notifications?
> **Answer:**
> EventBridge provides:
> - ==18+ AWS service destinations== (vs only 3 for traditional) — including Step Functions, Kinesis, Firehose, ECS Tasks
> - ==Advanced filtering== by metadata, object size, name, any JSON field (vs prefix/suffix only)
> - ==Multiple rules and targets== from a single event stream
> - ==Archive and replay== events for debugging or reprocessing
> - ==More reliable delivery== with built-in retry, exponential backoff, and dead-letter queues

> [!question]- Q3: What type of IAM policy is needed for S3 event notifications?
> **Answer:**
> ==Resource-based policies== on the target service, NOT IAM roles on S3:
> - SNS: ==SNS Resource Access Policy== allowing `s3.amazonaws.com` to `sns:Publish`
> - SQS: ==SQS Resource Access Policy== allowing `s3.amazonaws.com` to `sqs:SendMessage`
> - Lambda: ==Lambda Resource Policy== allowing `s3.amazonaws.com` to `lambda:InvokeFunction`
> These work similarly to S3 bucket policies — they define who can access the resource. The policy lives on the ==target==, not on S3.

> [!question]- Q4: What event types can S3 generate notifications for?
> **Answer:**
> - ==Object Created==: Put, Post, Copy, CompleteMultipartUpload (or wildcard `s3:ObjectCreated:*`)
> - ==Object Removed==: Delete, DeleteMarkerCreated
> - ==Object Restored==: Post (restore initiated), Completed (restore finished)
> - ==Replication==: OperationFailedReplication, OperationMissedThreshold
> You can filter by ==prefix== (e.g., `uploads/`) and ==suffix== (e.g., `.jpg`) to narrow which objects trigger events.

> [!question]- Q5: How quickly are S3 event notifications delivered?
> **Answer:**
> Typically ==within seconds==, but can take a minute or longer. Events are ==not guaranteed to be in order== and ==duplicates are possible== (at-least-once delivery). Design your event processing to be ==idempotent== — safe to process the same event multiple times without creating side effects or duplicate outputs.

> [!question]- Q6: What error do you get if the target doesn't have the right policy?
> **Answer:**
> =="Unable to validate the destination configuration."== This happens because S3 sends a ==test event== when you create the notification to verify connectivity. If the target (SQS/SNS/Lambda) doesn't have a resource-based policy allowing S3 to write to it, the validation fails. Fix: add the resource-based policy ==before== creating the event notification.

> [!question]- Q7: Can you send S3 events to Step Functions directly?
> **Answer:**
> ==Not with traditional event notifications== — those only support SNS, SQS, and Lambda as destinations. But with ==Amazon EventBridge==, yes — you can create a rule that sends S3 events to Step Functions, Kinesis Data Streams, Firehose, ECS Tasks, API Gateway, and many more services (18+ total).

> [!question]- Q8: What does the S3 event notification message look like?
> **Answer:**
> The message is a JSON payload containing a `Records` array. Each record includes:
> - `eventName`: The type of event (e.g., `ObjectCreated:Put`)
> - `s3.bucket.name`: The bucket where the event occurred
> - `s3.object.key`: The object key (filename)
> - `s3.object.size`: The object size in bytes
> - `eventTime`: When the event occurred
> This payload is what your Lambda function, SQS consumer, or SNS subscriber receives.

> [!question]- Q9: How do you enable EventBridge for an S3 bucket?
> **Answer:**
> Go to S3 bucket → ==Properties== → scroll to ==Amazon EventBridge== → ==Edit== → set to ==On==. Once enabled, ==all events== from the bucket are automatically sent to EventBridge — you don't need to configure individual event types. You then create ==EventBridge rules== to filter and route events to your desired destinations. This is a one-time toggle per bucket.

> [!question]- Q10: Can you have multiple event notifications on the same bucket?
> **Answer:**
> ==Yes.== You can create ==as many event notifications as desired== on a single bucket. Each notification can have different event types, different filters (prefix/suffix), and different destinations. For example:
> - `.jpg` uploads → Lambda for thumbnail generation
> - `.csv` uploads → SQS for data pipeline processing
> - All deletes → SNS for audit notification
> Additionally, you can enable EventBridge alongside traditional notifications — they work independently.
