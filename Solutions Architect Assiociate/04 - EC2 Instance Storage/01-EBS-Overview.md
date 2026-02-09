---
title: EBS Overview
date: 2026-02-09
tags:
  - aws
  - ec2
  - ebs
  - storage
  - block-storage
  - saa-c03
---

# EBS Overview (Elastic Block Store)

## What is EBS?

An EBS Volume is a ==network drive== you attach to EC2 instances while they run. It persists data even after instance termination.

> [!tip] Analogy
> Think of EBS as a ==network USB stick== — you can detach it from one computer and attach it to another, but through the network.

## Key Characteristics

| Feature | Details |
|---------|---------|
| **Type** | Network drive (not physical) |
| **Persistence** | Data persists after instance termination |
| **AZ Lock** | ==Bound to specific AZ== |
| **Attachment** | One instance at a time (CCP level) |
| **Latency** | Slight network latency |
| **Provisioning** | Must provision size (GB) and IOPS in advance |
| **Billing** | Billed for provisioned capacity |
| **Free Tier** | 30 GB/month of GP2/GP3 or Magnetic |

## Architecture

```
┌──────────────────────────────────────────────────────────┐
│                    us-east-1a                             │
│                                                          │
│  ┌──────────┐     ┌──────────┐                           │
│  │   EC2    │◀───▶│ EBS Vol  │  8 GB gp2                │
│  │ Instance │     │ (root)   │                           │
│  │    A     │     └──────────┘                           │
│  │          │◀───▶┌──────────┐                           │
│  └──────────┘     │ EBS Vol  │  100 GB gp3               │
│                   │ (data)   │                           │
│                   └──────────┘                           │
│                                                          │
│  ┌──────────┐     ┌──────────┐                           │
│  │   EC2    │◀───▶│ EBS Vol  │  8 GB gp2                │
│  │ Instance │     └──────────┘                           │
│  │    B     │                                            │
│  └──────────┘     ┌──────────┐                           │
│                   │ EBS Vol  │  Unattached (available)   │
│                   └──────────┘                           │
└──────────────────────────────────────────────────────────┘
```

> [!note] Key Points
> - One instance can have ==multiple EBS volumes==
> - EBS volumes can be ==left unattached==
> - Cannot attach across AZs (use snapshots to move)

## Delete on Termination

| Volume | Default | Behavior |
|--------|---------|----------|
| **Root volume** | ==Yes== (enabled) | Deleted when instance terminates |
| **Additional volumes** | ==No== (disabled) | Preserved after termination |

> [!warning] Exam Tip
> To preserve the root volume after termination, ==disable Delete on Termination== for the root volume during launch.

```
Launch Instance → Storage → Advanced
└── Delete on termination: Yes / No
    ├── Root volume: Yes (default) → change to No to preserve
    └── Additional: No (default) → data survives termination
```

## Hands-On Key Steps

1. View attached volumes: Instance → Storage tab → Block devices
2. Create new volume: Volumes → Create Volume → select ==same AZ==
3. Attach: Actions → Attach Volume → select instance
4. Cannot attach cross-AZ (different AZ = error, no instances shown)
5. Terminate instance → root volume deleted, additional volumes preserved

## Questions & Answers

> [!question]- Q1: What is an EBS volume?
> **Answer:**
> An EBS (Elastic Block Store) volume is a ==network drive== that you attach to EC2 instances. It provides persistent block storage that survives instance termination. Think of it as a network USB stick.

> [!question]- Q2: Can you attach an EBS volume to an instance in a different AZ?
> **Answer:**
> ==No==. EBS volumes are ==locked to a specific AZ==. To move data across AZs, you must create a snapshot and restore it in the target AZ.

> [!question]- Q3: What happens to EBS volumes when you terminate an EC2 instance?
> **Answer:**
> - **Root volume**: ==Deleted by default== (Delete on Termination = Yes)
> - **Additional volumes**: ==Preserved by default== (Delete on Termination = No)
> 
> You can change this behavior during instance launch.

> [!question]- Q4: Can an EC2 instance have multiple EBS volumes?
> **Answer:**
> ==Yes==. You can attach multiple EBS volumes to a single instance, like plugging multiple USB sticks into one computer. Each volume can have different sizes and types.

> [!question]- Q5: Can an EBS volume exist without being attached to an instance?
> **Answer:**
> ==Yes==. EBS volumes can be created and left unattached (status: "available"). They can be attached on demand to any instance in the ==same AZ==.

> [!question]- Q6: Why is there latency with EBS volumes?
> **Answer:**
> Because EBS is a ==network drive==, not a physical drive. Communication between the instance and the volume goes over the network, introducing some latency compared to directly attached storage.

> [!question]- Q7: How is EBS billing calculated?
> **Answer:**
> You're billed for ==provisioned capacity== (size in GB and IOPS), not actual usage. You can increase capacity over time. Free tier includes 30 GB/month of GP2/GP3 or Magnetic.

> [!question]- Q8: How do you preserve the root volume after instance termination?
> **Answer:**
> During instance launch, go to Storage → Advanced and set ==Delete on Termination to "No"== for the root volume. This prevents the root EBS from being deleted when the instance terminates.

> [!question]- Q9: Can you attach one EBS volume to multiple instances simultaneously?
> **Answer:**
> At the basic level, ==no== — one EBS volume attaches to one instance at a time. However, ==io1/io2 volumes support Multi-Attach== (up to 16 instances in the same AZ), covered in the EBS Volume Types section.

> [!question]- Q10: What is the EBS free tier?
> **Answer:**
> ==30 GB per month== of free EBS storage of type General Purpose (SSD) or Magnetic. This is available for the first 12 months of an AWS account.
