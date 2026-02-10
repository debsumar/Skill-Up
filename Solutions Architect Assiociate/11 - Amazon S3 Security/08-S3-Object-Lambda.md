---
title: "S3 Object Lambda"
date: 2026-02-10
tags:
  - aws
  - s3
  - object-lambda
  - lambda
  - access-points
  - data-transformation
  - pii
  - redaction
  - saa-c03
---

# S3 Object Lambda

## Overview

S3 Object Lambda is a powerful feature that lets you ==modify S3 objects on-the-fly as they're being retrieved== — without creating duplicate copies of the data. It uses ==AWS Lambda functions== (serverless code) to transform the data between S3 and the requesting application.

The key insight: ==one S3 bucket, one copy of the data, but multiple different views== — each tailored to a different consumer. An e-commerce app sees the full data, an analytics app sees redacted data (PII removed), and a marketing app sees enriched data (augmented with loyalty info). All from the ==same bucket, same objects==.

Without Object Lambda, you'd need to create ==separate buckets== with different versions of each object — duplicating storage, complicating updates, and increasing costs. Object Lambda eliminates all of that.

---

## The Problem: Data Duplication

Imagine you have customer order data in S3. Different teams need different versions:

| Consumer | What They Need | Without Object Lambda |
|----------|---------------|----------------------|
| ==E-Commerce App== | Full original data | Direct S3 access ✅ |
| ==Analytics Team== | Data with PII removed (no names, emails, addresses) | ❌ Need a ==second bucket== with redacted copies |
| ==Marketing Team== | Data enriched with customer loyalty tier from a database | ❌ Need a ==third bucket== with enriched copies |

That's ==three copies of every object==, three buckets to maintain, and a pipeline to keep them in sync. Every time the original data changes, you need to regenerate the redacted and enriched versions.

S3 Object Lambda eliminates this entirely.

---

## How It Works

```
┌──────────────────────────────────────────────────────────────────┐
│                S3 OBJECT LAMBDA — COMPLETE ARCHITECTURE           │
│                                                                   │
│  ┌──────────────┐                                                │
│  │  E-Commerce  │                                                │
│  │  Application │─── Direct access ──────────────────────┐      │
│  └──────────────┘    (original objects)                   │      │
│                                                           │      │
│                                                           ▼      │
│                                                    ┌───────────┐ │
│  ┌──────────────┐   ┌─────────────────────────┐   │           │ │
│  │  Analytics   │──▶│ Object Lambda AP #1     │   │           │ │
│  │  Application │   │                         │   │  S3       │ │
│  └──────────────┘   │ Invokes Lambda:         │   │  Bucket   │ │
│                      │ "Redact PII"           │   │           │ │
│                      │                         │──▶│ (single   │ │
│                      │ 1. Gets original object │   │  source   │ │
│                      │    from S3 Access Point │   │  of       │ │
│                      │ 2. Removes names,       │   │  truth)   │ │
│                      │    emails, addresses    │   │           │ │
│                      │ 3. Returns redacted     │   │           │ │
│                      │    version to caller    │   │           │ │
│                      └─────────────────────────┘   │           │ │
│                                                     │           │ │
│  ┌──────────────┐   ┌─────────────────────────┐   │           │ │
│  │  Marketing   │──▶│ Object Lambda AP #2     │   │           │ │
│  │  Application │   │                         │──▶│           │ │
│  └──────────────┘   │ Invokes Lambda:         │   │           │ │
│                      │ "Enrich with Loyalty"   │   │           │ │
│                      │                         │   └───────────┘ │
│                      │ 1. Gets original object │        ▲        │
│                      │    from S3 Access Point │        │        │
│                      │ 2. Looks up customer in │        │        │
│                      │    loyalty database     │        │        │
│                      │ 3. Adds loyalty tier,   │        │        │
│                      │    points, status       │   ┌────┴──────┐│
│                      │ 4. Returns enriched     │   │ S3 Access ││
│                      │    version to caller    │   │ Point     ││
│                      └─────────────────────────┘   │ (standard)││
│                                                     └───────────┘│
│                                                                   │
│  ✅ ONE bucket, ONE copy of data                                 │
│  ✅ THREE different views for three different consumers          │
│  ✅ No data duplication, no sync pipelines                       │
└──────────────────────────────────────────────────────────────────┘
```

### The Flow — Step by Step

1. An application (e.g., Analytics) makes a ==GetObject request== to the ==S3 Object Lambda Access Point==
2. The Object Lambda Access Point ==invokes a Lambda function== (your custom code)
3. The Lambda function ==retrieves the original object== from S3 via a ==standard S3 Access Point==
4. The Lambda function ==transforms the data== in memory (redact PII, enrich, convert format, etc.)
5. The ==transformed object is returned== to the requesting application
6. The ==original object in S3 is never modified== — it stays exactly as it was

> [!important] No Data Duplication — This is the Key Point
> You only need ==one S3 bucket== with the original data. Different applications get ==different views== of the same data through different Object Lambda Access Points. No need to create separate buckets, no need to maintain sync pipelines, no need to pay for duplicate storage.

---

## Components Required

S3 Object Lambda requires ==four components== working together:

| Component | Role | Analogy |
|-----------|------|---------|
| ==S3 Bucket== | Stores the original, unmodified objects | The warehouse with the raw materials |
| ==S3 Access Point== (standard) | Connects the Lambda function to the bucket | The loading dock where Lambda picks up objects |
| ==Lambda Function== | Your custom code that transforms the data | The factory that processes the raw materials |
| ==S3 Object Lambda Access Point== | The endpoint that applications connect to | The storefront where customers get the finished product |

```
┌──────────────────────────────────────────────────────────────┐
│              COMPONENT RELATIONSHIP                           │
│                                                               │
│  Application                                                  │
│      │                                                        │
│      │ GetObject request                                      │
│      ▼                                                        │
│  S3 Object Lambda Access Point                               │
│      │                                                        │
│      │ Invokes                                                │
│      ▼                                                        │
│  Lambda Function                                              │
│      │                                                        │
│      │ Retrieves original object via                          │
│      ▼                                                        │
│  S3 Access Point (standard)                                   │
│      │                                                        │
│      │ Reads from                                             │
│      ▼                                                        │
│  S3 Bucket (original data)                                    │
└──────────────────────────────────────────────────────────────┘
```

---

## Use Cases

| Use Case | What the Lambda Function Does | Real-World Example |
|----------|------------------------------|-------------------|
| ==Redact PII== | Removes personally identifiable information (names, emails, SSNs, addresses) | Analytics team queries customer data but never sees personal details |
| ==Data format conversion== | Converts between formats on the fly | Original data stored as XML; application receives JSON without maintaining two copies |
| ==Image transformation== | Resizes images, adds watermarks, changes format | Each user gets a watermarked image where the watermark includes their username |
| ==Data enrichment== | Augments objects with data from external databases | Marketing app receives customer orders enriched with loyalty tier and reward points from a separate database |
| ==Data filtering== | Removes sensitive columns or fields | Non-production environments get data with credit card numbers masked |
| ==Compression/decompression== | Compresses or decompresses data on the fly | Store compressed data in S3, serve decompressed data to applications that can't handle compression |

> [!tip] Exam Pattern
> If the exam describes a scenario where:
> - ==Different applications need different versions== of the same S3 data
> - The question asks how to ==avoid duplicating buckets== or maintaining sync pipelines
> - Data needs to be ==transformed on retrieval== (redacted, enriched, converted)
>
> The answer is ==S3 Object Lambda==.

---

## Key Characteristics

| Feature | Detail |
|---------|--------|
| ==Original data unchanged== | The Lambda function transforms a copy in memory — the bucket object is ==never modified== |
| ==Multiple access points== | You can create ==multiple Object Lambda Access Points== on the same bucket, each with a different Lambda function |
| ==Supports GetObject== | Object Lambda intercepts ==GetObject== requests (and optionally HeadObject and ListObjects) |
| ==Lambda execution== | The Lambda function runs in ==your account== — you control the code, runtime, and permissions |
| ==Latency== | Adds Lambda execution time to the request — not suitable for ==ultra-low-latency== use cases |
| ==Cost== | You pay for Lambda invocations and execution time, plus standard S3 request costs |

> [!note] Performance Consideration
> S3 Object Lambda adds the Lambda function's execution time to every GetObject request. For ==frequently accessed objects== or ==latency-sensitive applications==, consider whether the transformation overhead is acceptable. For batch analytics or infrequent access patterns, it's usually fine.

---

## Questions & Answers

> [!question]- Q1: What is S3 Object Lambda?
> **Answer:**
> A feature that lets you ==transform S3 objects on-the-fly== as they're being retrieved, using AWS Lambda functions. The original data stays unchanged in the bucket — transformations happen in real-time during retrieval. This eliminates the need to maintain multiple copies of data for different consumers.

> [!question]- Q2: Why would you use S3 Object Lambda instead of creating separate buckets?
> **Answer:**
> To ==avoid data duplication==. Instead of maintaining multiple buckets with different versions of the same data (redacted, enriched, converted), you keep ==one bucket with one copy== and use Lambda functions to transform data on retrieval for each consumer. This saves storage costs, eliminates sync pipelines, and ensures all consumers always see the latest data.

> [!question]- Q3: What four components are needed for S3 Object Lambda?
> **Answer:**
> 1. ==S3 Bucket== — stores the original, unmodified objects
> 2. ==S3 Access Point== (standard) — connects the Lambda function to the bucket
> 3. ==Lambda Function== — your custom code that transforms the data
> 4. ==S3 Object Lambda Access Point== — the endpoint applications connect to

> [!question]- Q4: Can you have multiple Object Lambda Access Points on the same bucket?
> **Answer:**
> Yes. Each Object Lambda Access Point can use a ==different Lambda function==, providing different transformations of the same data to different consumers. For example, one access point redacts PII for analytics, another enriches data for marketing — both reading from the same bucket.

> [!question]- Q5: What is a common use case for redacting data with S3 Object Lambda?
> **Answer:**
> Removing ==PII (Personally Identifiable Information)== — names, email addresses, phone numbers, SSNs, addresses — from objects before they're accessed by analytics or non-production environments. The original data with PII stays in the bucket for authorized applications, but analytics applications only see the redacted version through their Object Lambda Access Point.

> [!question]- Q6: How does S3 Object Lambda handle image transformations?
> **Answer:**
> A Lambda function can ==resize images, change formats, or add watermarks== on the fly. For example, each user could receive a watermarked version of an image where the watermark includes their username — all generated dynamically from the same original image in S3. No need to pre-generate watermarked copies for every user.

> [!question]- Q7: Does S3 Object Lambda modify the original object in the bucket?
> **Answer:**
> No. The original object ==remains completely unchanged==. The Lambda function retrieves the original, creates a transformed copy ==in memory==, and returns that copy to the requester. The bucket always contains the original, unmodified data.

> [!question]- Q8: An e-commerce company stores customer data in S3. Analytics needs the data without PII, marketing needs it enriched with loyalty data. How to solve without duplicating data?
> **Answer:**
> Create two ==S3 Object Lambda Access Points== on the same bucket:
> 1. Analytics access point → Lambda function that ==redacts PII== (removes names, emails, addresses)
> 2. Marketing access point → Lambda function that ==enriches with loyalty data== (looks up customer in loyalty database, adds tier and points)
>
> One bucket, one copy of data, two different views. No duplication, no sync pipelines.

> [!question]- Q9: What is the role of the standard S3 Access Point in the Object Lambda architecture?
> **Answer:**
> The standard S3 Access Point sits between the Lambda function and the S3 bucket. When the Lambda function needs to ==retrieve the original object== from the bucket, it does so through this standard access point. It's the "data source" connection that feeds the Lambda function with the raw data to transform.

> [!question]- Q10: Can S3 Object Lambda convert data formats?
> **Answer:**
> Yes. A common use case is converting ==XML to JSON== (or vice versa) on the fly. The original data is stored in one format in S3, but applications that need a different format receive it through the Object Lambda Access Point. The Lambda function handles the conversion in real-time, eliminating the need to store multiple format versions.
