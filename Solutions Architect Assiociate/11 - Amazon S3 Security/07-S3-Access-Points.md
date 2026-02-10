---
title: "S3 Access Points"
date: 2026-02-10
tags:
  - aws
  - s3
  - access-points
  - vpc
  - vpc-endpoint
  - security
  - bucket-policy
  - saa-c03
---

# S3 Access Points

## Overview

As S3 buckets grow with more data and more users, the ==bucket policy becomes a nightmare to manage==. Imagine a single bucket with finance data, sales data, analytics data, marketing data, and engineering data — each accessed by different teams with different permission levels. The bucket policy grows to hundreds of lines, becomes error-prone, and is nearly impossible to audit.

S3 Access Points solve this by letting you create ==named network endpoints==, each with its own ==access policy==. Instead of one massive bucket policy trying to handle everything, you create focused, simple policies — one per access point, one per use case.

Think of access points as ==dedicated doors into your bucket==, each with its own security guard and its own rules about who can enter and what they can access.

---

## The Problem: Unmanageable Bucket Policies

```
┌──────────────────────────────────────────────────────────────────┐
│  WITHOUT ACCESS POINTS — One Giant Bucket Policy                  │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────┐    │
│  │  S3 Bucket Policy (grows to 500+ lines)                  │    │
│  │                                                          │    │
│  │  Statement 1: Finance team → R/W to /finance/*           │    │
│  │  Statement 2: Finance interns → Read-only /finance/*     │    │
│  │  Statement 3: Sales team → R/W to /sales/*               │    │
│  │  Statement 4: Sales managers → R/W to /sales/reports/*   │    │
│  │  Statement 5: Analytics → Read-only /finance/* + /sales/*│    │
│  │  Statement 6: Marketing → Read /marketing/*              │    │
│  │  Statement 7: Engineering → R/W to /data/*               │    │
│  │  Statement 8: Data Science → Read /data/ml/*             │    │
│  │  Statement 9: Compliance → Read everything               │    │
│  │  Statement 10: External auditor → Read /audit/*          │    │
│  │  ... Statement 50 ...                                    │    │
│  │  ... Statement 100 ...                                   │    │
│  │                                                          │    │
│  │  ⚠️ Hard to read, hard to audit, easy to make mistakes   │    │
│  │  ⚠️ One wrong statement can expose sensitive data        │    │
│  │  ⚠️ Bucket policy has a 20KB size limit!                 │    │
│  └──────────────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────────────┘
```

> [!warning] Bucket Policy Size Limit
> S3 bucket policies have a ==20KB size limit==. With many teams and complex access patterns, you can actually ==hit this limit==. Access points solve this by distributing the policy logic across multiple access point policies, each with its own 20KB limit.

---

## The Solution: S3 Access Points

```
┌──────────────────────────────────────────────────────────────────┐
│  WITH ACCESS POINTS — Simple, Focused Policies                    │
│                                                                   │
│  Finance Users ──▶ ┌─────────────────────────┐                   │
│                     │ Finance Access Point    │──┐               │
│                     │ Policy: R/W /finance/*  │  │               │
│                     │ DNS: finance-ap.s3...   │  │               │
│                     └─────────────────────────┘  │               │
│                                                   │               │
│  Sales Users ────▶ ┌─────────────────────────┐   │  ┌──────────┐│
│                     │ Sales Access Point      │   ├─▶│          ││
│                     │ Policy: R/W /sales/*    │   │  │ S3       ││
│                     │ DNS: sales-ap.s3...     │──┤  │ Bucket   ││
│                     └─────────────────────────┘   │  │          ││
│                                                   │  │/finance/ ││
│  Analytics ──────▶ ┌─────────────────────────┐   │  │/sales/   ││
│                     │ Analytics Access Point  │   │  │/data/    ││
│                     │ Policy: Read-only       │──┘  │          ││
│                     │ /finance/* + /sales/*   │     └──────────┘│
│                     │ DNS: analytics-ap.s3... │                  │
│                     └─────────────────────────┘                  │
│                                                                   │
│  ✅ Each access point has a simple, focused policy               │
│  ✅ Bucket policy is minimal (just delegates to access points)   │
│  ✅ Easy to audit — each team's access is in one place           │
│  ✅ Easy to add new teams — just create a new access point       │
└──────────────────────────────────────────────────────────────────┘
```

### How It Works — Step by Step

1. **Create an access point** for each use case (finance, sales, analytics, etc.)
2. **Attach an access point policy** to each — it looks just like a bucket policy but is ==scoped to that access point==
3. Each access point gets its own ==DNS name== — this is how users connect to it
4. The ==bucket policy becomes minimal== — it just delegates access to the access points
5. Users connect to ==their access point's DNS name==, not directly to the bucket
6. The access point policy controls ==which prefixes== they can access and ==what operations== they can perform

### Access Point Properties

| Property | Description | Example |
|----------|-------------|---------|
| ==Name== | Unique name for the access point | `finance-ap`, `sales-readonly` |
| ==DNS Name== | Unique endpoint URL for connectivity | `finance-ap-123456789012.s3-accesspoint.eu-west-1.amazonaws.com` |
| ==Network Origin== | ==Internet== (public) or ==VPC== (private traffic only) | VPC for internal apps, Internet for external |
| ==Access Point Policy== | JSON policy (like bucket policy) scoped to this access point | Grant R/W to `/finance/*` for finance IAM role |
| ==Prefix restriction== | Limit access to specific object key prefixes | `/finance/*`, `/sales/reports/*` |

> [!tip] Exam Pattern
> If the exam describes a scenario with:
> - ==Many user groups== needing different access to the ==same bucket==
> - A ==bucket policy that's too complex== or hitting size limits
> - A need to ==simplify security management at scale==
>
> The answer is ==S3 Access Points==.

---

## VPC Origin — Private Access Without Internet

Access points can be configured with a ==VPC origin==, meaning they're ==only accessible from within a specific VPC==. Traffic never touches the public internet — it stays entirely within the AWS network.

This is ideal for ==internal applications== running on EC2, ECS, Lambda (in VPC), etc. that need to access S3 data privately.

```
┌──────────────────────────────────────────────────────────────────┐
│  VPC ORIGIN ACCESS — Private Connectivity                         │
│                                                                   │
│  ┌───────────────────────────────────────────────────────┐       │
│  │  VPC (your private network)                           │       │
│  │                                                       │       │
│  │  ┌──────────┐                                        │       │
│  │  │   EC2    │                                        │       │
│  │  │ Instance │                                        │       │
│  │  └────┬─────┘                                        │       │
│  │       │                                               │       │
│  │       │ Private traffic                               │       │
│  │       │ (never leaves AWS network)                    │       │
│  │       ▼                                               │       │
│  │  ┌──────────────────┐                                │       │
│  │  │  VPC Endpoint    │                                │       │
│  │  │  (Gateway or     │                                │       │
│  │  │   Interface)     │                                │       │
│  │  │                  │                                │       │
│  │  │  Has its own     │                                │       │
│  │  │  endpoint policy │                                │       │
│  │  └────────┬─────────┘                                │       │
│  └───────────┼───────────────────────────────────────────┘       │
│              │                                                    │
│              │ Private (no internet)                              │
│              ▼                                                    │
│  ┌──────────────────┐         ┌──────────────────┐              │
│  │  S3 Access Point │────────▶│    S3 Bucket     │              │
│  │  (VPC Origin)    │         │                  │              │
│  │                  │         │  /finance/       │              │
│  │  Access Point    │         │  /sales/         │              │
│  │  Policy: R/W     │         │  /data/          │              │
│  │  to /finance/*   │         │                  │              │
│  └──────────────────┘         └──────────────────┘              │
└──────────────────────────────────────────────────────────────────┘
```

### Three Layers of Security for VPC Access

When using VPC-origin access points, ==three policies must all allow the request== for it to succeed:

| Layer | Where | What It Controls | Must Allow |
|:-----:|-------|-----------------|-----------|
| 1️⃣ | ==VPC Endpoint Policy== | Controls what the VPC endpoint can connect to | Access to the access point AND the S3 bucket |
| 2️⃣ | ==Access Point Policy== | Controls who can use this access point and for what | Access to the specific prefixes for the requesting principal |
| 3️⃣ | ==Bucket Policy== | Controls access to the bucket itself | Delegation to the access points |

> [!important] All Three Must Agree
> If ==any one== of these three policies denies the request, the entire request is denied. All three must explicitly allow the action. This provides ==defense in depth== — multiple layers of security that must all agree.

---

## Questions & Answers

> [!question]- Q1: What problem do S3 Access Points solve?
> **Answer:**
> They solve the problem of ==unmanageable bucket policies== that grow too complex as more users and data are added. Instead of one massive bucket policy with hundreds of statements, you create ==separate access points== with simple, focused policies — one per use case. This also helps avoid the ==20KB bucket policy size limit==.

> [!question]- Q2: What is an access point policy?
> **Answer:**
> A JSON policy attached to an access point that looks just like a ==bucket policy== but is scoped to that specific access point. It defines who can access what prefixes through that access point and what operations they can perform. Each access point has its own independent policy.

> [!question]- Q3: How do users connect to an access point?
> **Answer:**
> Each access point has its own ==unique DNS name== (e.g., `finance-ap-123456789012.s3-accesspoint.eu-west-1.amazonaws.com`). Users and applications connect to this DNS name instead of the bucket directly. The access point routes the request to the bucket with the appropriate policy applied.

> [!question]- Q4: Can an access point be restricted to VPC-only access?
> **Answer:**
> Yes. You can set the access point's network origin to ==VPC==, making it only accessible from within a specific VPC via a ==VPC endpoint==. Traffic never touches the public internet. This is ideal for internal applications that need private S3 access.

> [!question]- Q5: What three policies must align for VPC-based access point access?
> **Answer:**
> All three must allow the request:
> 1. ==VPC Endpoint Policy== — must allow access to the access point and the S3 bucket
> 2. ==Access Point Policy== — must grant access to the specific prefixes for the requesting principal
> 3. ==Bucket Policy== — must delegate access to the access points
>
> If any one denies, the entire request is denied.

> [!question]- Q6: Can you have multiple access points on a single S3 bucket?
> **Answer:**
> Yes. You can create ==many access points== on a single bucket, each with different policies, DNS names, network origins, and access patterns. This is the core value — ==scaling access management== without complicating the bucket policy.

> [!question]- Q7: A company has one S3 bucket with finance, sales, and analytics data. Different teams need different access. The bucket policy is 500 lines and hitting the 20KB limit. What's the solution?
> **Answer:**
> Create ==S3 Access Points== — one for finance (R/W to `/finance/*`), one for sales (R/W to `/sales/*`), one for analytics (read-only to both prefixes). Each has a simple, focused policy. The bucket policy becomes minimal — just delegating to access points. Each access point has its own 20KB policy limit, so you'll never hit the size constraint.

> [!question]- Q8: What is a VPC endpoint in the context of S3 Access Points?
> **Answer:**
> A ==VPC endpoint== is a networking component inside your VPC that allows ==private connectivity to S3== without going through the public internet. For VPC-origin access points, you must create a VPC endpoint (Gateway or Interface type) to reach the access point. The VPC endpoint has its own policy that must also allow the access.

> [!question]- Q9: Do access points replace bucket policies entirely?
> **Answer:**
> No. The bucket still has a policy, but it becomes ==much simpler== — typically just a few statements delegating access to the access points. The complex per-user, per-prefix rules move to the individual access point policies. The bucket policy and access point policies work ==together==.

> [!question]- Q10: Can access points restrict access to specific prefixes?
> **Answer:**
> Yes. Each access point policy can grant access to ==specific prefixes== (e.g., `/finance/*`, `/sales/reports/*`). This is how you segment access within a single bucket — each access point only "sees" the prefixes its policy allows, even though the underlying bucket contains all the data.
