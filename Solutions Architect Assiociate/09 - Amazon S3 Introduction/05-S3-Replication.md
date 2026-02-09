---
title: S3 Replication
date: 2026-02-10
tags:
  - aws
  - s3
  - replication
  - crr
  - srr
  - saa-c03
---

# S3 Replication

## Overview

S3 Replication enables ==asynchronous copying== of objects between S3 buckets. The replication happens in the background — when you upload an object to the source bucket, S3 automatically copies it to the target bucket. There are two flavors:

| Type | Full Name | Description |
|------|-----------|-------------|
| **CRR** | ==Cross-Region Replication== | Source and target buckets are in ==different== AWS regions |
| **SRR** | ==Same-Region Replication== | Source and target buckets are in the ==same== AWS region |

## Architecture

```
┌──────────────────────────┐                    ┌──────────────────────────┐
│  Source Bucket            │   Asynchronous     │  Target Bucket           │
│  Region: eu-west-1       │   Replication       │  Region: us-east-1       │
│  Versioning: ENABLED     │──────────────────▶│  Versioning: ENABLED     │
│                          │                    │                          │
│  coffee.jpg (v1: GBk...) │                    │  coffee.jpg (v1: GBk...) │
│  beach.jpg  (v2: DK2...) │                    │  beach.jpg  (v2: DK2...) │
│                          │                    │                          │
│  IAM Role: read source   │                    │  Same version IDs!       │
│            write target   │                    │                          │
└──────────────────────────┘                    └──────────────────────────┘
```

Key points about the replication mechanism:
- Copying happens ==asynchronously== (in the background, typically within seconds)
- ==Version IDs are preserved== — the same version ID appears in both buckets
- Buckets can be in ==different AWS accounts== (cross-account replication)

## Prerequisites

| Requirement | Detail | Why |
|-------------|--------|-----|
| **Versioning on source** | Must be ==enabled== | Replication tracks changes via version IDs |
| **Versioning on target** | Must be ==enabled== | Target needs to store versioned copies |
| **IAM permissions** | S3 service needs a role with read (source) and write (target) permissions | S3 needs permission to copy objects between buckets |
| **Replication rule** | Must be configured on the ==source bucket== | Defines what to replicate and where |

## Use Cases

### Cross-Region Replication (CRR)

| Use Case | Description |
|----------|-------------|
| **Compliance** | Regulations may require data to be stored in multiple geographic regions |
| **Lower latency** | Users in another region can access data from a closer bucket |
| **Cross-account replication** | Share data between different AWS accounts (e.g., prod and analytics accounts) |

### Same-Region Replication (SRR)

| Use Case | Description |
|----------|-------------|
| **Log aggregation** | Collect logs from multiple S3 buckets into a single bucket for analysis |
| **Live replication** | Keep a live copy between production and test environments in the same region |
| **Data sovereignty** | Keep a backup copy in the same region for compliance |

## Replication Behavior — Critical Exam Topics

### What Gets Replicated

| Scenario | Replicated? | Detail |
|----------|------------|--------|
| New objects (after enabling) | ==Yes== | This is the default behavior |
| Existing objects (before enabling) | ==No== | Must use ==S3 Batch Replication== |
| Failed replications | ==No== (auto) | Must use ==S3 Batch Replication== to retry |
| Delete markers | ==Optional== | Disabled by default — must explicitly enable |
| Permanent deletes (version ID) | ==Never== | By design — prevents malicious cascading deletes |

> [!important] Only New Objects
> When you enable replication, it ==only applies to new objects== uploaded after the rule is created. If you have existing objects in the source bucket, they will NOT be automatically replicated. You must use ==S3 Batch Replication== to copy them.

### Delete Behavior in Detail

This is one of the most tested topics on the exam:

```
Scenario 1: Soft Delete (Delete Marker)
─────────────────────────────────────────
Source: coffee.jpg → Delete Marker added
Target: coffee.jpg → Delete Marker added (IF delete marker replication is enabled)

Scenario 2: Permanent Delete (Specific Version ID)
─────────────────────────────────────────
Source: coffee.jpg v1 → PERMANENTLY DELETED
Target: coffee.jpg v1 → STILL EXISTS (not replicated!)
```

Why aren't permanent deletes replicated? Because AWS wants to ==prevent malicious deletes== from cascading. If an attacker gains access to the source bucket and permanently deletes objects, the target bucket still has all the data safe and intact.

> [!warning] Delete Marker Replication Is Optional
> By default, delete markers are ==NOT replicated==. You must explicitly enable "Delete marker replication" in the replication rule settings. Even when enabled, only delete markers are replicated — permanent deletes (with version ID) are ==never== replicated.

### No Chaining

```
Bucket A ──replicates──▶ Bucket B ──replicates──▶ Bucket C

Objects uploaded to Bucket A:
  ✅ Replicated to Bucket B
  ❌ NOT replicated to Bucket C
```

There is ==no transitive replication==. If Bucket A replicates to Bucket B, and Bucket B replicates to Bucket C, objects from Bucket A do NOT automatically reach Bucket C. Each replication rule is independent.

## Hands-On Walkthrough

### Setting Up Replication

1. **Create origin bucket** (e.g., `s3-yourname-bucket-origin` in `eu-west-1`)
   - ==Enable versioning== during creation
2. **Create replica bucket** (e.g., `s3-yourname-bucket-replica` in `us-east-1`)
   - ==Enable versioning== during creation
3. Upload a file to the origin bucket (e.g., `beach.jpg`)
   - This file was uploaded ==before== replication — it won't be replicated

### Creating the Replication Rule

1. Go to origin bucket → **Management** tab → scroll to **Replication rules**
2. Click **Create replication rule**
3. Name: `DemoReplicationRule` → Status: ==Enabled==
4. Source: apply to ==all objects in the bucket==
5. Destination: choose **bucket in this account** → enter the replica bucket name
6. The console identifies the destination region (e.g., `us-east-1`) → shows "Cross-Region Replication"
7. IAM Role: choose **Create new role** (AWS creates the necessary permissions automatically)
8. Save

### The "Replicate Existing Objects?" Prompt

After saving, you'll see a prompt: "Do you want to replicate existing objects?"
- **Yes**: Creates an ==S3 Batch Operation job== to copy existing objects (separate from ongoing replication)
- **No**: Only new uploads will be replicated

Choose based on your needs. For the hands-on, choosing "No" is fine.

### Testing Replication

1. Upload a ==new file== to the origin bucket (e.g., `coffee.jpg`)
2. Wait ~10 seconds
3. Check the replica bucket → `coffee.jpg` should appear
4. Enable "Show versions" on both buckets → ==version IDs match==

### Testing Delete Marker Replication

1. Go to origin bucket → **Management** → Edit replication rule
2. Scroll down → enable ==Delete marker replication== → Save
3. Delete `coffee.jpg` from origin (soft delete — without "Show versions")
4. Check replica bucket → the delete marker should appear there too
5. But if you ==permanently delete== a specific version in the origin, it will ==NOT== be deleted in the replica

## Questions & Answers

> [!question]- Q1: What is the difference between CRR and SRR?
> **Answer:**
> - ==CRR== (Cross-Region Replication): source and target buckets are in different AWS regions. Used for compliance, lower latency, and cross-account replication.
> - ==SRR== (Same-Region Replication): source and target buckets are in the same region. Used for log aggregation, live replication between prod/test, and data sovereignty.

> [!question]- Q2: What are the prerequisites for enabling S3 replication?
> **Answer:**
> - ==Versioning must be enabled== on both source and target buckets
> - Proper ==IAM permissions== (a role that allows S3 to read from source and write to target)
> - A ==replication rule== configured on the source bucket
> - Buckets can be in the same or different accounts

> [!question]- Q3: Are existing objects replicated when you enable replication?
> **Answer:**
> ==No==. Only new objects uploaded after enabling the replication rule are replicated. To replicate existing objects, you must use ==S3 Batch Replication==, which creates a separate batch job to copy them. This is also used to retry objects that failed replication.

> [!question]- Q4: Are delete markers replicated by default?
> **Answer:**
> ==No==. Delete marker replication is disabled by default. You must explicitly enable it in the replication rule settings. When enabled, soft deletes (delete markers) are replicated from source to target. But permanent deletes (specific version IDs) are ==never== replicated, regardless of this setting.

> [!question]- Q5: Why aren't permanent deletes (version ID deletes) replicated?
> **Answer:**
> This is a ==security feature== to prevent malicious cascading deletes. If an attacker gains access to the source bucket and permanently deletes objects, the target bucket still has all the data intact. This protects against data loss from compromised accounts.

> [!question]- Q6: What is replication chaining and does S3 support it?
> **Answer:**
> Replication chaining would mean: Bucket A → Bucket B → Bucket C, where objects from A automatically reach C. S3 does ==NOT support chaining==. Each replication rule is independent. Objects from Bucket A will reach Bucket B, but they will NOT be forwarded to Bucket C.

> [!question]- Q7: Can replication work across different AWS accounts?
> **Answer:**
> ==Yes==. You can replicate between buckets in different AWS accounts. The IAM role needs cross-account permissions, and the target bucket's policy must allow the source account to write objects.

> [!question]- Q8: Are version IDs preserved during replication?
> **Answer:**
> ==Yes==. The exact same version ID appears in both the source and target buckets. This makes it easy to verify that replication is working correctly and to track objects across buckets.

> [!question]- Q9: What is S3 Batch Replication and when do you need it?
> **Answer:**
> S3 Batch Replication is a feature to replicate:
> - ==Existing objects== that were in the bucket before replication was enabled
> - ==Failed replications== that didn't complete successfully
>
> It creates a batch job that processes the objects. It's separate from the ongoing replication rule, which only handles new uploads.

> [!question]- Q10: How quickly does replication happen?
> **Answer:**
> Replication is ==asynchronous== and typically completes within ==seconds to minutes==. The exact time depends on object size, number of objects, and cross-region network latency. For CRR, it may take slightly longer than SRR due to the distance between regions. AWS does not guarantee a specific replication time SLA.
