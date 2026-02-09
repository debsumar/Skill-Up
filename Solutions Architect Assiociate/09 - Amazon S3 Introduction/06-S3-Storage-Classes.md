---
title: S3 Storage Classes
date: 2026-02-10
tags:
  - aws
  - s3
  - storage-classes
  - glacier
  - lifecycle
  - saa-c03
---

# S3 Storage Classes

## Overview

S3 offers ==7 storage classes== designed for different use cases. They differ in cost, availability, retrieval time, and minimum storage duration. The key insight: you're trading ==access speed and availability for lower cost==. The less frequently you access data, the cheaper it is to store.

You can choose the storage class when uploading an object, change it manually later, or use ==Lifecycle Rules== to automate transitions between classes.

## Durability vs Availability — Know the Difference

These two concepts are fundamental and frequently tested:

### Durability (How Safe Is My Data?)

| Fact | Detail |
|------|--------|
| **Value** | ==99.999999999%== (11 nines) |
| **Same for all classes** | Yes — every storage class has the same durability |
| **What it means** | If you store 10 million objects, you can expect to lose ==1 object every 10,000 years== |

Durability measures the probability of ==NOT losing== your data. S3 achieves this by automatically replicating your data across multiple devices and facilities within a region.

### Availability (Can I Access My Data Right Now?)

| Fact | Detail |
|------|--------|
| **Varies by class** | 99.5% to 99.99% depending on the storage class |
| **What it means** | How much downtime you can expect per year |
| **Example** | S3 Standard at 99.99% = about ==53 minutes of unavailability per year== |

Availability measures how readily the service is accessible. Lower availability means you might occasionally get errors when trying to access your data, and your application needs to handle that.

> [!tip] Exam Tip
> ==Durability is always the same== (11 nines) across all storage classes. ==Availability varies==. Don't confuse the two.

## The 7 Storage Classes in Detail

### 1. S3 Standard (General Purpose)

The ==default== storage class. Use this when you don't know what to pick or when you need frequent access.

| Property | Value |
|----------|-------|
| **Availability** | 99.99% |
| **AZs** | ≥ 3 |
| **Min storage duration** | None |
| **Retrieval time** | Instant (milliseconds) |
| **Retrieval cost** | None |
| **Facility failures** | Can sustain 2 concurrent facility failures |

**Use cases**: Big data analytics, mobile and gaming applications, content distribution, frequently accessed data.

### 2. S3 Standard-Infrequent Access (Standard-IA)

For data that you don't access often, but when you do, you need it ==immediately== (low latency). Cheaper storage cost than Standard, but you pay a ==retrieval fee== per GB retrieved.

| Property | Value |
|----------|-------|
| **Availability** | 99.9% (slightly lower than Standard) |
| **AZs** | ≥ 3 |
| **Min storage duration** | 30 days |
| **Retrieval time** | Instant (milliseconds) |
| **Retrieval cost** | ==Per GB retrieved== |

**Use cases**: Disaster recovery files, backups that you rarely need but must access quickly when you do.

### 3. S3 One Zone-Infrequent Access (One Zone-IA)

Same as Standard-IA but stored in ==only one Availability Zone==. This makes it cheaper, but if that AZ is destroyed (fire, flood, earthquake), your data is ==lost==.

| Property | Value |
|----------|-------|
| **Availability** | ==99.5%== (lowest of all classes) |
| **AZs** | ==1== (single AZ — this is the risk) |
| **Min storage duration** | 30 days |
| **Retrieval time** | Instant (milliseconds) |
| **Retrieval cost** | Per GB retrieved |

**Use cases**: Secondary copies of backups (primary backup is elsewhere), data you can recreate if lost (e.g., thumbnails generated from original images).

> [!danger] Data Loss Risk
> One Zone-IA stores data in a ==single AZ==. If that AZ is destroyed, your data is ==permanently lost==. Only use this for data you can afford to lose or recreate.

### 4. S3 Glacier Instant Retrieval

Archive storage with ==millisecond retrieval==. Think of it as "cold storage that you can access instantly." Great for data accessed about once per quarter.

| Property | Value |
|----------|-------|
| **Availability** | 99.9% |
| **AZs** | ≥ 3 |
| **Min storage duration** | ==90 days== |
| **Retrieval time** | ==Milliseconds== (instant!) |
| **Retrieval cost** | Per GB retrieved |

**Use cases**: Medical images accessed quarterly, financial reports accessed once per quarter, compliance data that must be retrievable instantly.

### 5. S3 Glacier Flexible Retrieval

Archive storage where you're willing to ==wait minutes to hours== to get your data back. This used to be called just "Amazon S3 Glacier" before they added more tiers.

| Property | Value |
|----------|-------|
| **Availability** | 99.99% |
| **AZs** | ≥ 3 |
| **Min storage duration** | ==90 days== |
| **Retrieval tiers** | See below |

| Retrieval Tier | Time | Cost |
|---------------|------|------|
| **Expedited** | ==1–5 minutes== | Highest |
| **Standard** | ==3–5 hours== | Medium |
| **Bulk** | ==5–12 hours== | ==Free== |

**Use cases**: Backup data accessed semi-annually, disaster recovery data where you can wait a few hours, long-term data retention.

> [!tip] Free Retrieval
> Bulk retrieval in Glacier Flexible is ==free==. If you can wait 5–12 hours, you don't pay any retrieval fee.

### 6. S3 Glacier Deep Archive

The ==cheapest storage class== in S3. Designed for data you almost never access — think years of archived records. You must be willing to wait ==12 to 48 hours== to retrieve your data.

| Property | Value |
|----------|-------|
| **Availability** | 99.99% |
| **AZs** | ≥ 3 |
| **Min storage duration** | ==180 days== (6 months!) |
| **Retrieval tiers** | See below |

| Retrieval Tier | Time |
|---------------|------|
| **Standard** | ==12 hours== |
| **Bulk** | ==48 hours== |

**Use cases**: Regulatory archives (7+ years), compliance data, data that's legally required to be kept but almost never accessed.

> [!example] Real-World Example
> ==Nasdaq== stores 7 years of financial data in S3 Glacier Deep Archive. They rarely need to access it, but regulations require them to keep it.

### 7. S3 Intelligent-Tiering

The "set it and forget it" storage class. S3 ==automatically moves your objects== between access tiers based on how frequently you access them. You pay a small monthly monitoring fee, but there are ==no retrieval charges==.

| Property | Value |
|----------|-------|
| **Availability** | 99.9% |
| **AZs** | ≥ 3 |
| **Min storage duration** | None |
| **Monitoring fee** | Small monthly per-object fee |
| **Retrieval cost** | ==None== |

#### Intelligent-Tiering Access Tiers

```
Object uploaded
       │
       ▼
┌─────────────────────────────┐
│ Frequent Access (default)   │ ← automatic
└──────────────┬──────────────┘
               │ Not accessed for 30 days
               ▼
┌─────────────────────────────┐
│ Infrequent Access           │ ← automatic
└──────────────┬──────────────┘
               │ Not accessed for 90 days
               ▼
┌─────────────────────────────┐
│ Archive Instant Access      │ ← automatic
└──────────────┬──────────────┘
               │ Configurable: 90–700+ days
               ▼
┌─────────────────────────────┐
│ Archive Access (optional)   │ ← must opt-in
└──────────────┬──────────────┘
               │ Configurable: 180–700+ days
               ▼
┌─────────────────────────────┐
│ Deep Archive Access (opt.)  │ ← must opt-in
└─────────────────────────────┘
```

The first three tiers are automatic. The last two (Archive Access and Deep Archive Access) are ==optional== and must be configured.

**Use cases**: When you ==don't know your access patterns== or when access patterns change over time. Let AWS optimize the cost for you.

## Storage Class Decision Tree

```
How often do you access the data?
│
├── Frequently (daily/weekly)?
│   └── S3 Standard
│
├── Infrequently but need instant access when you do?
│   ├── Need multi-AZ durability?
│   │   └── S3 Standard-IA
│   └── Single AZ OK? (data is recreatable)
│       └── S3 One Zone-IA
│
├── Rarely (archive)?
│   ├── Need millisecond retrieval?
│   │   └── Glacier Instant Retrieval (90-day min)
│   ├── Can wait 1 min – 12 hours?
│   │   └── Glacier Flexible Retrieval (90-day min)
│   └── Can wait 12 – 48 hours?
│       └── Glacier Deep Archive (180-day min)
│
└── Don't know / patterns change?
    └── S3 Intelligent-Tiering
```

## Minimum Storage Duration — Important for Cost

If you delete, overwrite, or transition an object ==before== the minimum storage duration, you're ==still charged for the full duration==.

| Class | Minimum Duration | What This Means |
|-------|-----------------|-----------------|
| Standard | None | No penalty for early deletion |
| Standard-IA | 30 days | Delete after 10 days → still charged for 30 days |
| One Zone-IA | 30 days | Same as above |
| Glacier Instant | ==90 days== | Delete after 30 days → still charged for 90 days |
| Glacier Flexible | ==90 days== | Same as above |
| Glacier Deep Archive | ==180 days== | Delete after 60 days → still charged for 180 days |
| Intelligent-Tiering | None | No penalty |

## Lifecycle Rules — Automating Transitions

Instead of manually changing storage classes, you can create ==Lifecycle Rules== that automatically transition objects based on age:

```
Example Lifecycle Rule:
┌──────────────┐  30 days  ┌──────────────┐  60 days  ┌───────────────────┐
│ S3 Standard  │─────────▶│ Standard-IA  │─────────▶│ Intelligent-      │
└──────────────┘          └──────────────┘          │ Tiering           │
                                                     └────────┬──────────┘
                                                              │ 180 days
                                                              ▼
                                                     ┌───────────────────┐
                                                     │ Glacier Flexible  │
                                                     │ Retrieval         │
                                                     └────────┬──────────┘
                                                              │ 365 days
                                                              ▼
                                                     ┌───────────────────┐
                                                     │ DELETE (expire)   │
                                                     └───────────────────┘
```

### Creating a Lifecycle Rule (Hands-On)

1. Go to bucket → **Management** tab → **Lifecycle rules** → **Create rule**
2. Name: `DemoRule`
3. Scope: apply to ==all objects in the bucket== (or filter by prefix/tags)
4. Choose **Move current versions between storage classes**
5. Set transitions:
   - Standard-IA after 30 days
   - Intelligent-Tiering after 60 days
   - Glacier Flexible Retrieval after 180 days
6. Optionally: **Expire current versions** after 365 days (auto-delete)
7. Review the transition timeline → Create rule

## Hands-On: Changing Storage Classes

### At Upload Time

1. Upload a file → expand **Properties** section
2. Under **Storage class**, you'll see all available classes with their details
3. Choose the desired class (e.g., Standard-IA) → Upload

### After Upload

1. Click on the object → **Properties** tab → scroll to **Storage class**
2. Click **Edit** → choose a new class (e.g., One Zone-IA, Glacier Instant Retrieval)
3. Save changes

> [!note] Reduced Redundancy
> You may see "Reduced Redundancy" as an option. This is a ==deprecated== storage class. Ignore it — it's not covered on the exam and should not be used.

## Questions & Answers

> [!question]- Q1: What is the durability of S3 and does it vary by storage class?
> **Answer:**
> S3 durability is ==99.999999999%== (11 nines) — this means if you store 10 million objects, you can expect to lose 1 object every 10,000 years. Durability is the ==same for ALL storage classes==. What varies is availability, not durability.

> [!question]- Q2: What is the difference between durability and availability?
> **Answer:**
> - ==Durability==: probability of NOT losing your data (11 nines for all classes — your data is extremely safe)
> - ==Availability==: how readily the service is accessible (varies: 99.5% for One Zone-IA to 99.99% for Standard)
>
> High durability means your data won't be lost. High availability means you can access it whenever you want. A class can be highly durable but less available.

> [!question]- Q3: What is the default S3 storage class and when should you use it?
> **Answer:**
> ==S3 Standard== is the default. Use it for frequently accessed data, when you don't know your access patterns yet, or when you need low latency and high throughput. It has 99.99% availability, sustains 2 concurrent facility failures, and has no minimum storage duration or retrieval fees.

> [!question]- Q4: What is the risk of using S3 One Zone-IA?
> **Answer:**
> Data is stored in ==only one Availability Zone==. If that AZ is destroyed (natural disaster, hardware failure), your data is ==permanently lost==. The durability within that single AZ is still 11 nines, but if the entire AZ goes down, the data is gone. Only use for data you can recreate or secondary backup copies.

> [!question]- Q5: What are the three Glacier storage classes and how do they differ?
> **Answer:**
> 1. ==Glacier Instant Retrieval==: millisecond retrieval, 90-day minimum — for data accessed once a quarter
> 2. ==Glacier Flexible Retrieval==: 1 min to 12 hours (3 tiers: Expedited, Standard, Bulk), 90-day minimum — for semi-annual access
> 3. ==Glacier Deep Archive==: 12 to 48 hours, 180-day minimum — for long-term archives accessed once a year or less
>
> The key difference is retrieval time: instant vs minutes/hours vs half a day or more.

> [!question]- Q6: What is the cheapest retrieval option for Glacier Flexible Retrieval?
> **Answer:**
> ==Bulk retrieval== — takes 5 to 12 hours but is ==completely free==. If you can wait, you pay nothing for retrieval. Expedited (1–5 min) is the most expensive, Standard (3–5 hours) is in the middle.

> [!question]- Q7: What is S3 Intelligent-Tiering and when should you use it?
> **Answer:**
> Intelligent-Tiering ==automatically moves objects== between access tiers based on usage patterns. It charges a small monthly monitoring fee per object but has ==no retrieval charges==. Use it when you ==don't know your access patterns== or when they change over time. It has 5 tiers: Frequent Access (default), Infrequent Access (30 days), Archive Instant Access (90 days), and two optional archive tiers.

> [!question]- Q8: What is the minimum storage duration and why does it matter for cost?
> **Answer:**
> Each storage class (except Standard and Intelligent-Tiering) has a minimum storage duration. If you delete or transition an object ==before== this period, you're ==still charged for the full duration==. For example, if you store an object in Glacier Deep Archive (180-day minimum) and delete it after 30 days, you still pay for 180 days of storage.

> [!question]- Q9: How do lifecycle rules work and what can they do?
> **Answer:**
> Lifecycle rules ==automatically transition objects== between storage classes based on age. For example: Standard → Standard-IA after 30 days → Glacier after 90 days → Delete after 365 days. They can:
> - Transition current versions between storage classes
> - Transition previous versions (in versioned buckets)
> - Expire (delete) objects after a set number of days
> - Clean up incomplete multi-part uploads
>
> Rules can apply to all objects or be filtered by prefix or tags.

> [!question]- Q10: Can you change an object's storage class after upload?
> **Answer:**
> Yes, in two ways:
> 1. ==Manually==: Go to the object's Properties → Edit Storage Class → choose a new class
> 2. ==Automatically==: Create Lifecycle Rules that transition objects based on age
>
> Note: not all transitions are allowed. For example, you can't move from Glacier back to Standard — you'd need to restore the object first and then copy it.
