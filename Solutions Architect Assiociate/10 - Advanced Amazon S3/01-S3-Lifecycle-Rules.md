---
title: "S3 Lifecycle Rules"
date: 2026-02-10
tags:
  - aws
  - s3
  - lifecycle
  - storage-classes
  - s3-analytics
  - saa-c03
---

# S3 Lifecycle Rules

## Overview

In [[09 - Amazon S3 Introduction/06-S3-Storage-Classes|S3 Storage Classes]], we learned about the different storage tiers — Standard, Standard-IA, One-Zone-IA, Intelligent-Tiering, Glacier Instant Retrieval, Glacier Flexible Retrieval, and Glacier Deep Archive. Now the question is: ==how do you move objects between these classes automatically?==

You could do it manually, but that doesn't scale. ==S3 Lifecycle Rules== automate the process of transitioning objects to cheaper storage classes over time and deleting objects when they're no longer needed. This is one of the most ==cost-effective tools== in your S3 toolbox — and a ==heavily tested topic== on the SAA-C03 exam.

The core idea is simple: objects have a lifecycle. When they're new, they're accessed frequently (hot). Over time, access drops (warm). Eventually, they're rarely or never accessed (cold). Lifecycle rules let you ==match your storage costs to your access patterns== automatically.

## Storage Class Transition Waterfall

Objects can only transition ==downward== through the storage class hierarchy. You cannot use lifecycle rules to move objects back "up" to a hotter tier:

```
┌─────────────────────────────────────────────────────────────────┐
│              STORAGE CLASS TRANSITION FLOW                      │
│                                                                 │
│  S3 Standard  (hottest, most expensive)                        │
│      │                                                         │
│      ├──▶ S3 Standard-IA                                       │
│      │        │                                                │
│      ├──▶ S3 Intelligent-Tiering                               │
│      │        │                                                │
│      ├──▶ S3 One-Zone-IA                                       │
│      │        │                                                │
│      ├──▶ S3 Glacier Instant Retrieval                         │
│      │        │                                                │
│      ├──▶ S3 Glacier Flexible Retrieval                        │
│      │        │                                                │
│      └──▶ S3 Glacier Deep Archive  (coldest, cheapest)         │
│                                                                 │
│  ⬇ Objects flow DOWN — from hot (expensive) to cold (cheap)   │
│  ✗ You CANNOT transition back UP with lifecycle rules          │
└─────────────────────────────────────────────────────────────────┘
```

> [!warning] One-Way Street
> Lifecycle rules are a ==one-way street==. You can transition Standard → Glacier, but you ==cannot== transition Glacier → Standard with a lifecycle rule. To move an object back to Standard, you must ==restore== it first (creates a temporary copy), then ==copy== it to a new location. This is a manual process.

### Minimum Storage Duration Charges

Each storage class has a ==minimum storage duration==. If you delete or transition an object before this period, you're still charged for the full minimum:

| Storage Class | Minimum Duration | What This Means |
|--------------|-----------------|-----------------|
| ==Standard== | None | No minimum — pay only for what you use |
| ==Standard-IA== | ==30 days== | Delete after 10 days? Still charged for 30 days |
| ==One-Zone-IA== | ==30 days== | Same as Standard-IA |
| ==Glacier Instant Retrieval== | ==90 days== | Delete after 30 days? Charged for 90 days |
| ==Glacier Flexible Retrieval== | ==90 days== | Same as Instant Retrieval |
| ==Glacier Deep Archive== | ==180 days== | Delete after 60 days? Charged for 180 days |

> [!tip] Exam Tip
> When designing lifecycle rules, make sure your transition days ==align with minimum storage durations==. Transitioning to Standard-IA after only 10 days means you pay for 30 days anyway. Transition to Standard-IA after ==at least 30 days== to avoid wasted charges.

### Transition Constraints

Not all transitions are allowed. Here are the key constraints:

| Constraint | Detail |
|-----------|--------|
| ==Standard-IA minimum object size== | Objects smaller than ==128 KB== are not transitioned to Standard-IA (they'd cost more in IA) |
| ==Minimum days between transitions== | At least ==30 days== must pass between transitions to different IA/Glacier classes |
| ==One-Zone-IA → Glacier only== | From One-Zone-IA, you can only go to Glacier Flexible or Deep Archive |
| ==No upward transitions== | Cannot go from a colder class to a warmer class via lifecycle rules |

## Two Types of Lifecycle Actions

| Action Type | What It Does | Example |
|-------------|-------------|---------|
| ==Transition Actions== | Move objects to a different storage class | Move to Standard-IA after 60 days, then to Glacier after 180 days |
| ==Expiration Actions== | Delete objects after a specified time | Delete access logs after 365 days |

### Transition Actions

Configure objects to ==automatically move to a cheaper storage class== after a specified number of days since creation:

```
Day 0          Day 30           Day 90           Day 180          Day 365
│              │                │                │                │
▼              ▼                ▼                ▼                ▼
┌──────────┐  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│ Standard │─▶│ Standard-IA  │─▶│ Intelligent  │─▶│ Glacier      │─▶│ Deep Archive │
│          │  │              │ │ Tiering      │ │ Flexible     │ │              │
│ $0.023/GB│  │ $0.0125/GB   │ │              │ │ $0.0036/GB   │ │ $0.00099/GB  │
└──────────┘  └──────────────┘ └──────────────┘ └──────────────┘ └──────────────┘
```

The key insight is that transition actions are based on ==days since object creation== (or days since the object became non-current, for versioned objects). S3 evaluates the rules daily and transitions objects that meet the criteria.

### Expiration Actions

Configure objects to be ==automatically deleted== after a specified time:

- Delete ==current versions== of objects (e.g., access logs after 365 days)
- Delete ==old/non-current versions== of objects (when versioning is enabled)
- Delete ==incomplete multi-part uploads== (e.g., uploads older than 7 days that were never completed)
- Delete ==expired object delete markers== (clean up delete markers that have no non-current versions)

> [!important] Incomplete Multi-Part Uploads
> When a multi-part upload is initiated but never completed (e.g., the client crashed), the uploaded parts ==still consume storage== but are invisible in the S3 console. Over time, these can accumulate and waste significant storage. A lifecycle rule to ==delete incomplete multi-part uploads after 7 days== is a best practice for every bucket.

## Rule Filters

Lifecycle rules can be scoped using filters to target specific objects:

| Filter | Example | Use Case |
|--------|---------|----------|
| ==Prefix== | `logs/` | Only apply to objects in the `logs/` folder |
| ==Object tags== | `department:finance` | Only apply to objects tagged with finance |
| ==Entire bucket== | No filter | Apply to all objects in the bucket |
| ==Combination== | Prefix `images/` + tag `type:thumbnail` | Apply only to thumbnails in the images folder |

This is powerful because you can have ==different lifecycle policies for different types of objects== in the same bucket. For example, source images might transition to Glacier after 60 days, while thumbnails are deleted after 60 days — all in the same bucket, differentiated by prefix.

## Exam Scenarios

> [!question] Scenario 1: Thumbnails vs Source Images
> An application on EC2 creates thumbnails after profile photos are uploaded to Amazon S3. Thumbnails can be easily recreated from the original photo and only need to be kept for 60 days. Source images should be immediately retrievable for 60 days, then the user can wait up to 6 hours for retrieval.
>
> **Solution:**
> - ==Source images==: S3 Standard → Lifecycle rule to transition to ==Glacier Flexible Retrieval== after 60 days (supports 3-5 hour expedited or 5-12 hour standard retrieval)
> - ==Thumbnails==: ==S3 One-Zone-IA== (infrequently accessed, easily recreated, don't need multi-AZ durability) → Lifecycle rule to ==expire/delete== after 60 days
> - Use ==prefix filters== to differentiate: `source/` vs `thumbnails/`
> - Why One-Zone-IA for thumbnails? Because they can be ==recreated== if lost, so you don't need the extra durability of multi-AZ storage

> [!question] Scenario 2: Recover Deleted Objects
> Company policy: recover deleted objects immediately for 30 days (rare occurrence), then within 48 hours for up to 365 days.
>
> **Solution:**
> - Enable ==S3 Versioning== — when an object is "deleted," S3 adds a ==delete marker== but preserves the previous version. You can recover by removing the delete marker.
> - Lifecycle rule for ==non-current versions==: transition to ==Standard-IA== immediately (infrequent access, but immediately retrievable for the first 30 days)
> - After 30 days, transition non-current versions to ==Glacier Deep Archive== (supports 12-48 hour retrieval, cheapest storage)
> - The key insight: versioning + lifecycle rules on non-current versions = ==automated disaster recovery==

## Hands-On: Creating Lifecycle Rules in the Console

In the S3 console, go to your bucket → ==Management== tab → ==Create lifecycle rule==.

### Step 1: Name and Scope

- Give the rule a name (e.g., `demo-rule`)
- Choose scope: ==Apply to all objects== or filter by prefix/tags
- If applying to all objects, you must ==acknowledge== the warning

### Five Rule Actions Available

```
┌─────────────────────────────────────────────────────────────────┐
│              LIFECYCLE RULE ACTIONS (Console)                   │
│                                                                 │
│  1. ✅ Move CURRENT versions between storage classes           │
│     → Standard → Standard-IA → Intelligent-Tiering            │
│       → Glacier Instant → Glacier Flexible → Deep Archive     │
│                                                                 │
│  2. ✅ Move NON-CURRENT versions between storage classes       │
│     → Old versions → Glacier Flexible → Deep Archive          │
│                                                                 │
│  3. ✅ Expire CURRENT versions of objects                      │
│     → Delete the current version after N days                  │
│                                                                 │
│  4. ✅ Permanently delete NON-CURRENT versions                 │
│     → Delete old versions after N days since becoming          │
│       non-current                                              │
│                                                                 │
│  5. ✅ Delete expired object delete markers                    │
│     OR delete incomplete multi-part uploads after N days       │
└─────────────────────────────────────────────────────────────────┘
```

### Step 2: Configure Transitions (Current Versions)

You can chain multiple transitions for current versions. The console lets you add transitions one by one:

| Step | Transition To | After Days | Rationale |
|------|--------------|------------|-----------|
| 1 | Standard-IA | 30 | Object access drops after first month |
| 2 | Intelligent-Tiering | 60 | Let AWS optimize automatically |
| 3 | Glacier Instant Retrieval | 90 | Rarely accessed but need millisecond retrieval |
| 4 | Glacier Flexible Retrieval | 180 | Archive — can wait 3-12 hours |
| 5 | Deep Archive | 365 | Long-term archive — can wait 12-48 hours |

> [!note] Console Acknowledgment
> When you add transitions, the console may ask you to ==acknowledge== that you understand the minimum storage duration charges. For example, transitioning to Standard-IA means a minimum 30-day charge even if the object is transitioned again before 30 days.

### Step 3: Configure Transitions (Non-Current Versions)

Non-current versions (old versions in a versioned bucket) can be transitioned separately. This is useful because old versions are ==almost never accessed== and can go straight to cold storage:

- Transition non-current versions to ==Glacier Flexible Retrieval== after 90 days
- This saves significant cost on versioned buckets with many old versions

### Step 4: Configure Expiration

- Expire current versions after ==700 days== (delete the object)
- Permanently delete non-current versions after ==700 days==
- Delete incomplete multi-part uploads after ==7 days==

### Step 5: Review Timeline

The console shows a ==visual timeline== of what happens to your current and non-current versions over time. This is extremely helpful for understanding the full lifecycle at a glance — you can see exactly when each transition and expiration occurs.

```
┌─────────────────────────────────────────────────────────────────┐
│  CONSOLE TIMELINE VIEW                                         │
│                                                                 │
│  Current Versions:                                             │
│  Day 0    Day 30      Day 60       Day 90      Day 180  Day365│
│  │Standard│Std-IA     │Int-Tier    │Glacier-IR │Flex    │Deep │
│  ├────────┼───────────┼────────────┼───────────┼────────┼─────│
│                                                                 │
│  Non-Current Versions:                                         │
│  Day 0                Day 90                          Day 700  │
│  │ Non-current        │ Glacier Flexible              │DELETE │
│  ├────────────────────┼───────────────────────────────┼───────│
│                                                                 │
│  Incomplete Multi-Part Uploads:                                │
│  Day 0    Day 7                                                │
│  │ Pending │ DELETE                                            │
│  ├─────────┼──────────                                         │
└─────────────────────────────────────────────────────────────────┘
```

## Lifecycle Rules vs Intelligent-Tiering

A common question: why use lifecycle rules when Intelligent-Tiering exists?

| Aspect | Lifecycle Rules | Intelligent-Tiering |
|--------|----------------|-------------------|
| **How it works** | ==You define== when to transition based on days | ==AWS automatically== moves objects based on access patterns |
| **Predictability** | You know exactly when transitions happen | AWS decides — you don't control timing |
| **Cost** | No monitoring fee | ==Small monitoring fee== per object per month |
| **Glacier access** | Can transition to Glacier/Deep Archive | Can optionally enable Glacier tiers |
| **Best for** | ==Known access patterns== (logs, backups, archives) | ==Unknown/changing access patterns== |
| **Expiration** | ==Yes== — can delete objects | ==No== — only transitions, no deletion |

> [!tip] When to Use Which
> - Use ==lifecycle rules== when you ==know== your access patterns (e.g., "logs are never accessed after 90 days")
> - Use ==Intelligent-Tiering== when access patterns are ==unpredictable== or vary per object
> - You can ==combine both==: use Intelligent-Tiering as the storage class, and lifecycle rules to eventually expire/delete objects

## S3 Analytics — Storage Class Analysis

How do you know ==when== to transition objects? If you're unsure about your access patterns, use ==S3 Analytics== to get data-driven recommendations.

```
┌──────────────┐   Analyzes    ┌──────────────┐   Generates   ┌──────────────┐
│  S3 Bucket   │──────────────▶│  S3 Analytics│──────────────▶│  CSV Report  │
│  (objects)   │               │  Engine      │               │  (recommend- │
│              │               │  (runs daily)│               │   ations &   │
│              │               │              │               │   statistics)│
└──────────────┘               └──────────────┘               └──────────────┘
                                                                     │
                                                                     ▼
                                                              ┌──────────────┐
                                                              │  Lifecycle   │
                                                              │  Rule Design │
                                                              │  (informed)  │
                                                              └──────────────┘
```

| Feature | Detail |
|---------|--------|
| **What it does** | Analyzes access patterns and provides recommendations for ==Standard → Standard-IA== transitions |
| **Limitations** | Does ==NOT== work for One-Zone-IA or Glacier — only Standard and Standard-IA |
| **Output** | ==CSV report== with usage statistics, access frequency, and transition recommendations |
| **Update frequency** | Report updated ==daily== |
| **Initial delay** | Takes ==24-48 hours== to start seeing data analysis |
| **Scope** | Can be configured per bucket or per prefix |
| **Use case** | First step to creating or improving lifecycle rules — ==data-driven approach== |

The workflow is: enable S3 Analytics → wait 24-48 hours → review the CSV report → use the recommendations to create lifecycle rules with the right number of days for transitions.

> [!tip] Exam Tip
> S3 Analytics helps you determine the ==optimal number of days== before transitioning objects from Standard to Standard-IA. It only works for Standard and Standard-IA — not for Glacier or One-Zone-IA. If the exam asks "how to determine when to transition objects," the answer is ==S3 Analytics==.

## Questions & Answers

> [!question]- Q1: What are the two types of S3 Lifecycle actions?
> **Answer:**
> 1. ==Transition actions==: Automatically move objects to a different (cheaper) storage class after a specified number of days since creation. Example: Standard → Standard-IA after 60 days → Glacier Flexible after 180 days → Deep Archive after 365 days.
> 2. ==Expiration actions==: Automatically delete objects after a specified number of days. This includes deleting current versions, non-current versions, incomplete multi-part uploads, and expired delete markers.

> [!question]- Q2: Can lifecycle rules apply to specific objects only?
> **Answer:**
> Yes. Rules can be filtered by:
> - ==Prefix==: Only objects in a specific "folder" (e.g., `logs/`, `thumbnails/`)
> - ==Object tags==: Only objects with specific tags (e.g., `department:finance`)
> - ==Entire bucket==: No filter — applies to all objects
> You can combine prefix and tag filters for precise targeting. This lets you have different lifecycle policies for different types of objects in the same bucket.

> [!question]- Q3: What is S3 Analytics and what does it recommend?
> **Answer:**
> S3 Analytics analyzes object access patterns and provides recommendations for transitioning objects from ==Standard to Standard-IA==. It generates a ==CSV report updated daily== (takes 24-48 hours to start producing data). It does ==NOT== work for One-Zone-IA or Glacier transitions — only Standard and Standard-IA. Use it as a data-driven first step before creating lifecycle rules to determine the optimal number of transition days.

> [!question]- Q4: How do lifecycle rules work with versioning?
> **Answer:**
> With versioning enabled, lifecycle rules can target:
> - ==Current versions==: The latest version of each object — transition to cheaper classes or expire (delete)
> - ==Non-current versions==: Previous versions that have been overwritten — transition to cold storage or permanently delete
> This is powerful for compliance and cost optimization — keep current versions accessible while automatically archiving or deleting old versions. When you "delete" a versioned object, S3 adds a delete marker; the previous version is preserved and can be targeted by non-current version rules.

> [!question]- Q5: Can you chain multiple transitions in a single lifecycle rule?
> **Answer:**
> ==Yes.== A single rule can define multiple transitions in sequence:
> Standard → Standard-IA (30 days) → Intelligent-Tiering (60 days) → Glacier Instant (90 days) → Glacier Flexible (180 days) → Deep Archive (365 days)
> Each transition must be to a ==lower-cost== storage class, and the days must be ==increasing==. The console shows a visual timeline of all transitions.

> [!question]- Q6: What happens to incomplete multi-part uploads?
> **Answer:**
> Incomplete multi-part uploads consume storage but are ==invisible== in the S3 console's normal object listing. When a multi-part upload is initiated but never completed (e.g., client crash, network failure), the uploaded parts remain and accumulate over time. You can create a lifecycle rule to ==automatically delete incomplete multi-part uploads== after a specified number of days (e.g., 7 days). This is a ==best practice for every bucket== to avoid paying for abandoned uploads.

> [!question]- Q7: Exam scenario — source images need immediate access for 60 days, then 6-hour retrieval. Thumbnails can be recreated. What's the design?
> **Answer:**
> - Source images: ==S3 Standard== with lifecycle rule to transition to ==Glacier Flexible Retrieval== after 60 days (supports 3-5 hour expedited or 5-12 hour standard retrieval — within the 6-hour requirement)
> - Thumbnails: ==S3 One-Zone-IA== (cheap, infrequently accessed, can be recreated if the AZ is lost) with lifecycle rule to ==expire after 60 days==
> - Use ==prefix filters== to separate: `source/` prefix for originals, `thumbnails/` prefix for generated thumbnails

> [!question]- Q8: Exam scenario — recover deleted objects immediately for 30 days, within 48 hours for 365 days. How?
> **Answer:**
> 1. Enable ==S3 Versioning== — deletes create delete markers, previous versions are preserved and recoverable
> 2. Lifecycle rule: transition ==non-current versions== to ==Standard-IA== immediately (infrequent access but immediately retrievable for the first 30 days)
> 3. After 30 days, transition non-current versions to ==Glacier Deep Archive== (supports 12-48 hour retrieval — within the 48-hour requirement)
> This gives you immediate recovery for 30 days and archival recovery for up to 365 days, all automated.

> [!question]- Q9: What are the minimum storage duration charges for each storage class?
> **Answer:**
> - ==Standard==: No minimum
> - ==Standard-IA==: ==30 days== minimum
> - ==One-Zone-IA==: ==30 days== minimum
> - ==Glacier Instant Retrieval==: ==90 days== minimum
> - ==Glacier Flexible Retrieval==: ==90 days== minimum
> - ==Glacier Deep Archive==: ==180 days== minimum
> If you delete or transition an object before the minimum duration, you're still charged for the full minimum period. Design lifecycle rules to align with these minimums.

> [!question]- Q10: Can lifecycle rules transition objects from Glacier back to Standard?
> **Answer:**
> ==No.== Lifecycle rules can only transition objects ==downward== through the storage class hierarchy (from hot to cold). To move an object from Glacier back to Standard, you must:
> 1. ==Restore== the object (creates a temporary copy in Standard for a specified number of days)
> 2. ==Copy== the restored object to a new location in Standard
> This is a manual process, not automated by lifecycle rules. The restore operation itself can take minutes to 48 hours depending on the Glacier tier and retrieval speed selected.
