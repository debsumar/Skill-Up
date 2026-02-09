---
title: EC2 Instance Storage - Overview
date: 2026-02-09
tags:
  - aws
  - ec2
  - ebs
  - efs
  - storage
  - saa-c03
  - index
---

# EC2 Instance Storage - Overview

## Course Section Summary

This section covers all storage options available for EC2 instances: EBS, Snapshots, AMIs, Instance Store, and EFS.

## Study Files

| # | Topic | File | Priority |
|---|-------|------|----------|
| 1 | EBS Overview | [[01-EBS-Overview]] | 🔴 High |
| 2 | EBS Snapshots | [[02-EBS-Snapshots]] | 🔴 High |
| 3 | AMI (Amazon Machine Image) | [[03-AMI]] | 🔴 High |
| 4 | EC2 Instance Store | [[04-EC2-Instance-Store]] | 🟡 Medium |
| 5 | EBS Volume Types | [[05-EBS-Volume-Types]] | 🔴 High |
| 6 | EBS Encryption | [[06-EBS-Encryption]] | 🔴 High |
| 7 | Amazon EFS | [[07-Amazon-EFS]] | 🔴 High |

## Key Exam Topics

### Storage Comparison

| Feature | EBS | Instance Store | EFS |
|---------|-----|----------------|-----|
| **Type** | Network drive | Local hardware | Network file system |
| **Persistence** | Persists independently | ==Ephemeral== | Persists independently |
| **AZ** | Locked to one AZ | Locked to instance | ==Multi-AZ== |
| **Attach** | One instance (multi-attach for io) | One instance | ==Hundreds of instances== |
| **OS** | Any | Any | ==Linux only== |
| **Capacity** | Provisioned | Fixed | ==Auto-scaling== |

### Quick Decision Tree

```
Need persistent block storage?
├── Yes → EBS
│   ├── Need >32K IOPS? → io1/io2 (Nitro)
│   ├── General workload? → gp3 (or gp2)
│   ├── Big data throughput? → st1
│   └── Archive/cold? → sc1
├── Need shared file system across AZs?
│   └── EFS (Linux only, pay-per-use)
└── Need highest possible IOPS (millions)?
    └── EC2 Instance Store (ephemeral!)
```

## Related Sections

- [[../03 - EC2 SAA Level/00-EC2-SAA-Level-Overview|EC2 SAA Level]] - Networking, Placement Groups, Hibernate
- [[../02 - EC2 Fundamentals/00-EC2-Overview|EC2 Fundamentals]] - Basic EC2 concepts
