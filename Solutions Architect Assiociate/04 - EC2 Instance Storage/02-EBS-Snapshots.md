---
title: EBS Snapshots
date: 2026-02-09
tags:
  - aws
  - ec2
  - ebs
  - snapshots
  - backup
  - disaster-recovery
  - saa-c03
---

# EBS Snapshots

## What is an EBS Snapshot?

An EBS Snapshot is a ==point-in-time backup== of your EBS volume.

> [!tip] Key Concept
> Snapshots let you ==copy EBS volumes across AZs and Regions== — the only way to move EBS data between AZs.

## Snapshot Basics

| Feature | Details |
|---------|---------|
| **Detach required?** | Not necessary, but ==recommended== |
| **Cross-AZ** | ✅ Can restore snapshot in any AZ |
| **Cross-Region** | ✅ Can copy snapshot to any region |
| **Use case** | Backup, disaster recovery, AZ migration |

## How to Move EBS Across AZs

```
us-east-1a                              us-east-1b
┌──────────┐     ┌──────────┐          ┌──────────┐
│ EBS Vol  │────▶│ Snapshot │─────────▶│ New EBS  │
│ (source) │     │          │ Restore  │ (copy)   │
└──────────┘     └──────────┘          └──────────┘
                      │
                      │ Copy
                      ▼
               ┌──────────────┐
               │ Snapshot in  │
               │ eu-west-1    │  (Cross-Region DR)
               └──────────────┘
```

## Snapshot Features

### 1. EBS Snapshot Archive

| Feature | Details |
|---------|---------|
| **Cost** | Up to ==75% cheaper== than standard |
| **Restore time** | ==24 to 72 hours== |
| **Use case** | Long-term backup, infrequent access |

### 2. Recycle Bin

> [!warning] Accidental Deletion Protection
> Deleted snapshots go to the Recycle Bin instead of being permanently removed.

| Feature | Details |
|---------|---------|
| **Retention** | ==1 day to 1 year== |
| **Recovery** | Can recover deleted snapshots |
| **Applies to** | EBS Snapshots and AMIs |

### 3. Fast Snapshot Restore (FSR)

| Feature | Details |
|---------|---------|
| **Purpose** | ==No latency on first use== of restored volume |
| **Use case** | Large snapshots needing instant initialization |
| **Cost** | ==Expensive== — use carefully |

## Hands-On Key Steps

```
Create Snapshot:
  Volume → Actions → Create Snapshot → Add description

Copy to Another Region:
  Snapshot → Right-click → Copy Snapshot → Choose destination region

Restore to Different AZ:
  Snapshot → Actions → Create Volume from Snapshot → Choose target AZ

Recycle Bin:
  Recycle Bin Console → Create Retention Rule
  ├── Resource type: EBS Snapshots
  ├── Apply to: All resources
  └── Retention: 1 day (minimum)

Archive:
  Snapshot → Storage Tier → Archive Snapshot
  └── Restore takes 24-72 hours
```

## Questions & Answers

> [!question]- Q1: What is an EBS Snapshot?
> **Answer:**
> A ==point-in-time backup== of an EBS volume. It captures the state of the volume and can be used to restore data, create new volumes, or copy data across AZs and Regions.

> [!question]- Q2: How do you move an EBS volume from one AZ to another?
> **Answer:**
> 1. Create a ==snapshot== of the EBS volume
> 2. ==Restore== the snapshot as a new volume in the target AZ
> 
> This is the only way since EBS volumes are locked to their AZ.

> [!question]- Q3: Do you need to detach an EBS volume before taking a snapshot?
> **Answer:**
> ==No==, it's not required. However, it is ==recommended== to detach the volume first to ensure data consistency during the snapshot process.

> [!question]- Q4: What is the EBS Snapshot Archive?
> **Answer:**
> A storage tier that is up to ==75% cheaper== than standard snapshot storage. The trade-off is that restoring from archive takes ==24 to 72 hours==. Good for long-term backups you rarely need.

> [!question]- Q5: What is the Recycle Bin for EBS Snapshots?
> **Answer:**
> A safety net that catches deleted snapshots instead of permanently removing them. You set a ==retention rule (1 day to 1 year)==, and during that period you can recover accidentally deleted snapshots.

> [!question]- Q6: What is Fast Snapshot Restore (FSR)?
> **Answer:**
> FSR forces a ==full initialization of the snapshot== so there's no latency on first use. Useful for very large snapshots that need instant volume creation, but it's ==very expensive==.

> [!question]- Q7: Can you copy a snapshot to another AWS Region?
> **Answer:**
> ==Yes==. You can copy snapshots to any destination region. This is useful for ==disaster recovery== — backing up data in a different region in case of regional failure.

> [!question]- Q8: Can you encrypt a snapshot during the copy process?
> **Answer:**
> ==Yes==. When copying a snapshot, you can enable encryption even if the source snapshot was unencrypted. This is one way to encrypt an existing unencrypted EBS volume.

> [!question]- Q9: Does the Recycle Bin also protect AMIs?
> **Answer:**
> ==Yes==. The Recycle Bin retention rules can be configured for both ==EBS Snapshots and Amazon Machine Images (AMIs)==.

> [!question]- Q10: What is the difference between archiving and deleting a snapshot?
> **Answer:**
> - **Archive**: Moves the snapshot to a ==cheaper storage tier== (75% savings). Data is preserved but takes 24-72 hours to restore.
> - **Delete**: Removes the snapshot. Goes to ==Recycle Bin== if a retention rule exists, otherwise permanently deleted.
