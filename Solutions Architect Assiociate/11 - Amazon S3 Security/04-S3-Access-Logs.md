---
title: "S3 Access Logs"
date: 2026-02-10
tags:
  - aws
  - s3
  - access-logs
  - audit
  - athena
  - logging
  - compliance
  - security
  - saa-c03
---

# S3 Access Logs

## Overview

For ==audit purposes==, you may want to log every single access made to your S3 buckets. S3 Server Access Logging captures ==every request== — from any account, whether ==authorized or denied== — and writes it as a log file into ==another S3 bucket==. This gives you a complete audit trail of who did what, when, and whether it succeeded.

The log data can then be analyzed using tools like ==Amazon Athena== (serverless SQL queries directly on S3 files), making it easy to answer questions like "who accessed this file last week?" or "how many failed access attempts were there?"

---

## How It Works

```
┌──────────────────────────────────────────────────────────────────┐
│                    S3 ACCESS LOGGING ARCHITECTURE                  │
│                                                                   │
│  ┌──────────────┐                                                │
│  │  User A      │──── GET object ────┐                           │
│  │  (authorized)│                    │                           │
│  └──────────────┘                    │                           │
│                                       ▼                           │
│  ┌──────────────┐              ┌──────────────┐                  │
│  │  User B      │──── PUT ───▶ │              │                  │
│  │  (authorized)│              │  S3 Bucket   │                  │
│  └──────────────┘              │ (monitored)  │                  │
│                                 │              │                  │
│  ┌──────────────┐              │  Every       │                  │
│  │  User C      │──── GET ───▶ │  request is  │                  │
│  │  (DENIED)    │   ❌         │  logged      │                  │
│  └──────────────┘              └──────┬───────┘                  │
│                                        │                          │
│                                        │ Logs delivered           │
│                                        │ (can take hours)        │
│                                        ▼                          │
│                                 ┌──────────────┐                 │
│                                 │  Logging     │                 │
│                                 │  Bucket      │                 │
│                                 │ (MUST be     │                 │
│                                 │  different   │                 │
│                                 │  bucket!)    │                 │
│                                 │              │                 │
│                                 │  Same region │                 │
│                                 │  as source   │                 │
│                                 └──────┬───────┘                 │
│                                        │                          │
│                                        ▼                          │
│                                 ┌──────────────┐                 │
│                                 │  Analyze     │                 │
│                                 │  with Amazon │                 │
│                                 │  Athena      │                 │
│                                 │  (SQL on S3) │                 │
│                                 └──────────────┘                 │
└──────────────────────────────────────────────────────────────────┘
```

### Key Characteristics

| Feature | Detail |
|---------|--------|
| ==What's logged== | Every request — GET, PUT, DELETE, HEAD, LIST, etc. |
| ==Authorized & denied== | Both successful and failed requests are captured |
| ==Cross-account== | Requests from ==any AWS account== are logged, not just the bucket owner's |
| ==Log destination== | Must be a ==different S3 bucket== in the ==same AWS region== |
| ==Delivery time== | ==Not real-time== — can take a ==couple of hours== for logs to appear |
| ==Log format== | Specific text format with space-delimited fields |
| ==Cost== | Logging itself is ==free==; you only pay for the ==storage== of log files in the logging bucket |
| ==Bucket policy== | AWS ==automatically updates== the logging bucket's policy to allow the S3 logging service to write |

### What's in a Log Entry?

Each log entry contains detailed information about a single request:

| Field | Description | Example |
|-------|-------------|---------|
| ==Bucket Owner== | Account ID of the bucket owner | `79a59df900b949e55d96a1e698fbacedfd6e09d98eacf8f8d5218e7cd47ef2be` |
| ==Bucket== | Name of the bucket | `my-app-bucket` |
| ==Time== | Timestamp of the request | `[06/Feb/2024:00:00:38 +0000]` |
| ==Remote IP== | IP address of the requester | `192.0.2.3` |
| ==Requester== | IAM user/role ARN or `-` for anonymous | `arn:aws:iam::123456789012:user/Alice` |
| ==Request ID== | Unique ID for the request | `3E57427F3EXAMPLE` |
| ==Operation== | The S3 API operation | `REST.GET.OBJECT` |
| ==Key== | The object key | `photos/2024/photo.jpg` |
| ==HTTP Status== | Response status code | `200` (success) or `403` (denied) |
| ==Error Code== | Error code if request failed | `AccessDenied` or `-` |
| ==Bytes Sent== | Number of bytes in the response | `4567890` |

> [!note] Raw Logs Are Hard to Read
> The raw log files are ==space-delimited text== that's difficult to parse manually. For any serious analysis, use ==Amazon Athena== to run SQL queries directly on the log files. Athena can parse the log format and let you query by time range, requester, operation, status code, etc.

---

## The Logging Loop Danger

> [!danger] ==NEVER== Set the Logging Bucket to the Same Bucket You're Monitoring!
> This is the ==single most important rule== of S3 access logging. If the logging bucket is the same as the monitored bucket, every log write becomes a new request that gets logged, which generates another log write, which gets logged again... creating an ==infinite loop==.
>
> The result: your bucket grows ==exponentially==, and you receive a ==massive, unexpected AWS bill==.

```
┌──────────────────────────────────────────────────────────────┐
│              ⚠️ INFINITE LOGGING LOOP — DO NOT DO THIS        │
│                                                               │
│  ┌──────────────────────────────────────────────────────┐    │
│  │  Same Bucket (app data + logging target)             │    │
│  │                                                      │    │
│  │  1. User uploads file ──────────────────────────┐    │    │
│  │                                                  │    │    │
│  │  2. Upload is logged ◀──────────────────────────┘    │    │
│  │     (log file written to SAME bucket)                │    │
│  │                                                      │    │
│  │  3. Log write is itself a request ──────────────┐    │    │
│  │     that gets logged                             │    │    │
│  │                                                  │    │    │
│  │  4. That log write gets logged ◀────────────────┘    │    │
│  │                                                      │    │
│  │  5. And that log write gets logged...                │    │
│  │                                                      │    │
│  │  6. ∞ INFINITE LOOP                                  │    │
│  │                                                      │    │
│  │  💸💸💸 Exponential growth = MASSIVE bill             │    │
│  └──────────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────────┘
```

> [!quote] From the Course
> "Do not try this at home. You will pay a lot of money."

---

## Hands-On Walkthrough

### Step 1: Create the Logging Bucket

1. Go to ==S3 Console → Create Bucket==
2. Name it something clear like `my-access-logs-bucket`
3. Keep it in the ==same region== as the bucket you want to monitor
4. Leave default settings — no need for public access or versioning
5. Click ==Create Bucket==

### Step 2: Enable Server Access Logging

1. Go to the bucket you want to monitor → ==Properties==
2. Scroll down to ==Server Access Logging==
3. Click ==Edit → Enable==
4. You'll see a notice: "The bucket policy will be updated for the target bucket"
5. For ==Destination==, browse and select your logging bucket (`my-access-logs-bucket`)

### Step 3: Configure Log Delivery Options

| Setting | Description | Recommendation |
|---------|-------------|----------------|
| ==Destination bucket== | Where logs are written | Your separate logging bucket |
| ==Destination prefix== | Optional folder prefix in the logging bucket | e.g., `logs/` to organize files (optional) |
| ==Log object key format== | How log file names are structured | ==Default format== is fine for most use cases |

> [!tip] Date-Based Partitioning
> You can choose a log key format that includes ==date-based partitioning== (year/month/day in the path). This makes it easier to query specific time ranges with Athena. But the default format works fine for getting started.

6. Click ==Save Changes==

### Step 4: Verify the Bucket Policy Update

1. Go to the ==logging bucket== → ==Permissions → Bucket Policy==
2. You'll see an ==automatically added policy== that allows the S3 logging service to write:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "logging.s3.amazonaws.com"
      },
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::my-access-logs-bucket/*",
      "Condition": {
        "StringEquals": {
          "aws:SourceAccount": "123456789012"
        }
      }
    }
  ]
}
```

### Step 5: Generate Some Activity

Perform various operations on the monitored bucket:
- Upload a file
- Download/open a file
- Delete a file
- Try accessing with different permissions

### Step 6: Wait and Check Logs

- ==Wait a couple of hours== — log delivery is not real-time
- Refresh the logging bucket — you'll see log files appearing
- Click on a log file and open it to see the raw log entries
- The entries show the API call, requester, timestamp, status code, and more

> [!note] Log Delivery Delay
> Don't panic if logs don't appear immediately. S3 access log delivery can take ==several hours==. This is by design — logs are batched and delivered periodically, not in real-time.

---

## Questions & Answers

> [!question]- Q1: What are S3 Access Logs?
> **Answer:**
> A feature that logs ==every request== made to an S3 bucket — including both authorized and denied requests, from any AWS account — into a separate S3 bucket. This provides a complete audit trail for compliance, security analysis, and troubleshooting.

> [!question]- Q2: Where are S3 access logs stored?
> **Answer:**
> In a ==separate S3 bucket== that you designate as the logging target. This bucket ==must be in the same AWS region== as the monitored bucket. It must be a ==different bucket== — never the same one.

> [!question]- Q3: Why should you never set the logging bucket to be the same as the monitored bucket?
> **Answer:**
> It creates an ==infinite logging loop==. Each log write is itself a request that gets logged, which generates another log write, which gets logged again — infinitely. This causes ==exponential bucket growth== and a ==massive unexpected AWS bill==.

> [!question]- Q4: How quickly do access logs appear?
> **Answer:**
> Log delivery is ==not real-time==. It can take a ==couple of hours== for logs to appear in the logging bucket. Logs are batched and delivered periodically. Don't expect instant results.

> [!question]- Q5: What tool is recommended for analyzing S3 access logs?
> **Answer:**
> ==Amazon Athena== — a serverless query service that can run SQL queries directly on the log files stored in S3. You don't need to load the data into a database. Athena can parse the S3 log format and let you query by time range, requester, operation, status code, IP address, etc.

> [!question]- Q6: What information is included in S3 access logs?
> **Answer:**
> Each log entry includes: ==bucket owner==, ==bucket name==, ==timestamp==, ==remote IP address==, ==requester identity== (IAM user/role ARN), ==request ID==, ==S3 operation== (GET, PUT, DELETE, etc.), ==object key==, ==HTTP status code==, ==error code== (if any), ==bytes sent==, and more.

> [!question]- Q7: What happens to the logging bucket's policy when you enable access logging?
> **Answer:**
> AWS ==automatically updates the bucket policy== on the logging bucket to allow the S3 logging service (`logging.s3.amazonaws.com`) to put objects into it. You don't need to manually configure this — it's done for you when you enable logging.

> [!question]- Q8: Can you log access from other AWS accounts?
> **Answer:**
> Yes. S3 access logs capture requests from ==any AWS account== — not just the bucket owner's account. Both authorized and denied cross-account requests are logged, making it useful for security auditing.

> [!question]- Q9: Can you filter which requests get logged?
> **Answer:**
> No. Server access logging logs ==all requests== to the bucket. You cannot filter by request type, user, object, or any other criteria at the logging level. However, you can use ==Athena queries with WHERE clauses== to filter when analyzing the logs.

> [!question]- Q10: What is the difference between S3 Access Logs and CloudTrail S3 data events?
> **Answer:**
> | Feature | S3 Access Logs | CloudTrail S3 Data Events |
> |---------|---------------|--------------------------|
> | ==Format== | Space-delimited text files | Structured JSON events |
> | ==Delivery== | To an S3 bucket (hours delay) | To CloudTrail (near real-time) |
> | ==Integration== | Athena for analysis | EventBridge, CloudWatch, SNS |
> | ==Cost== | Free (pay only for log storage) | Paid per 100,000 events |
> | ==Detail level== | Request-level (IP, user agent, etc.) | API-level (IAM context, etc.) |
>
> Use access logs for ==detailed request analysis==; use CloudTrail for ==event-driven automation== and structured auditing.
