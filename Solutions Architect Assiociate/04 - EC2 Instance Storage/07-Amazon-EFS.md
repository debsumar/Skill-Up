---
title: Amazon EFS (Elastic File System)
date: 2026-02-09
tags:
  - aws
  - ec2
  - efs
  - nfs
  - shared-storage
  - saa-c03
---

# Amazon EFS (Elastic File System)

## What is EFS?

A ==managed Network File System (NFS)== that can be mounted on ==many EC2 instances across multiple AZs==.

> [!important] Key Differentiator
> EFS is ==multi-AZ== and ==multi-instance== — unlike EBS which is locked to one AZ and one instance.

## EFS Characteristics

| Feature | Details |
|---------|---------|
| **Protocol** | NFS |
| **Multi-AZ** | ✅ Yes |
| **Multi-instance** | ✅ Hundreds of concurrent clients |
| **OS** | ==Linux only== (POSIX file system) |
| **Capacity** | ==Auto-scaling== (no provisioning needed) |
| **Billing** | ==Pay per use== (per GB stored) |
| **Cost** | ~3x gp2 price (but use storage tiers to save) |
| **Encryption** | KMS encryption at rest |
| **Scale** | Up to ==petabyte scale== automatically |

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        EFS File System                       │
│                    (with Security Group)                      │
└──────────┬──────────────┬──────────────┬────────────────────┘
           │              │              │
     us-east-1a     us-east-1b     us-east-1c
    ┌──────────┐   ┌──────────┐   ┌──────────┐
    │   EC2    │   │   EC2    │   │   EC2    │
    │ Instance │   │ Instance │   │ Instance │
    │    A     │   │    B     │   │    C     │
    └──────────┘   └──────────┘   └──────────┘

    All instances share the same file system!
    Write on A → instantly visible on B and C
```

## Performance Modes

| Mode | Latency | Throughput | Use Case |
|------|---------|-----------|----------|
| **General Purpose** (default) | ==Low== (~250 µs reads) | Standard | Web servers, CMS |
| **Max I/O** | Higher | ==Highest==, highly parallel | Big data, media processing |

## Throughput Modes

| Mode | Description | Throughput | Best For |
|------|-------------|-----------|----------|
| **Bursting** | Scales with storage size | 50 MB/s per TB + burst to 100 MB/s | Predictable workloads |
| **Elastic** (recommended) | ==Auto-scales== up/down | Up to ==3 GB/s reads==, ==1 GB/s writes== | ==Unpredictable workloads== |
| **Provisioned** | Fixed throughput you set | Decouple throughput from storage (e.g., 1 GB/s for 1 TB) | Known throughput needs |

> [!tip] Recommended Setting
> Use ==Elastic throughput== with ==General Purpose== performance mode. This is the AWS-recommended default.

## Storage Classes & Tiers

| Tier | Access Pattern | Cost |
|------|---------------|------|
| **Standard** | Frequently accessed | Highest |
| **Infrequent Access (IA)** | Less frequent | Lower storage, cost to retrieve |
| **Archive** | Rarely accessed (few times/year) | ==Lowest== |

### Lifecycle Policies

```
EFS Standard
    │
    │ Not accessed for 30 days
    ▼
EFS Infrequent Access (IA)
    │
    │ Not accessed for 90 days
    ▼
EFS Archive
    │
    │ Accessed again
    ▼
EFS Standard (on first access)
```

> [!note] Cost Savings
> Using the right storage classes can save up to ==90% in costs==.

## Availability Options

| Option | AZs | Use Case |
|--------|-----|----------|
| **Regional** (Standard) | ==Multi-AZ== | Production workloads |
| **One Zone** | Single AZ | Dev/test (cheaper) |
| **One Zone-IA** | Single AZ + IA tier | Dev with cost savings |

## EFS vs EBS vs Instance Store

| Feature | EBS | EFS | Instance Store |
|---------|-----|-----|----------------|
| **Type** | Block storage | ==Network file system== | Local disk |
| **AZ** | ==Single AZ== | ==Multi-AZ== | Single instance |
| **Instances** | 1 (or 16 with multi-attach) | ==Hundreds== | 1 |
| **OS** | Any | ==Linux only== | Any |
| **Capacity** | Provisioned | ==Auto-scaling== | Fixed |
| **Billing** | Provisioned capacity | ==Pay per use== | Included in EC2 |
| **Persistence** | Persists | Persists | ==Ephemeral== |
| **Cost** | Moderate | Higher (but tiers help) | Included |
| **Performance** | Up to 256K IOPS | 10+ GB/s throughput | Millions of IOPS |

> [!warning] Key Exam Distinctions
> - Need shared storage across AZs → ==EFS==
> - Need block storage for one instance → ==EBS==
> - Need highest IOPS, temporary → ==Instance Store==
> - EBS backups use IO → don't run during high traffic
> - Root EBS deleted on termination by default

## Questions & Answers

> [!question]- Q1: What is Amazon EFS?
> **Answer:**
> A ==managed Network File System (NFS)== that can be mounted on many EC2 instances across multiple AZs simultaneously. It's highly available, scalable, and pay-per-use.

> [!question]- Q2: Can EFS be used with Windows instances?
> **Answer:**
> ==No==. EFS is ==Linux only== because it uses the POSIX file system. For Windows shared storage, consider Amazon FSx for Windows File Server.

> [!question]- Q3: How does EFS billing work compared to EBS?
> **Answer:**
> EFS is ==pay-per-use== (per GB stored). EBS requires ==provisioned capacity== (you pay for the size you allocate, even if unused). EFS is ~3x the price of gp2 but storage tiers can save up to 90%.

> [!question]- Q4: What are the three EFS throughput modes?
> **Answer:**
> 1. **Bursting** — throughput scales with storage size
> 2. **Elastic** (recommended) — auto-scales based on workload
> 3. **Provisioned** — fixed throughput you configure in advance

> [!question]- Q5: What is the difference between EFS Standard and One Zone?
> **Answer:**
> - **Standard (Regional)**: Data replicated across ==multiple AZs== — for production
> - **One Zone**: Data in ==single AZ== — cheaper, for dev/test
> 
> Both support IA storage tiers for cost savings.

> [!question]- Q6: How do EFS lifecycle policies work?
> **Answer:**
> You define rules like "move to IA after 30 days of no access" and "move to Archive after 90 days." Files are automatically moved between tiers. On first access, files can transition back to Standard.

> [!question]- Q7: What is the key difference between EBS and EFS?
> **Answer:**
> - **EBS**: Block storage, ==one AZ, one instance== (mostly), provisioned capacity
> - **EFS**: Network file system, ==multi-AZ, hundreds of instances==, auto-scaling, pay-per-use
> 
> EFS is for shared storage; EBS is for dedicated instance storage.

> [!question]- Q8: How do you control access to EFS?
> **Answer:**
> Through ==security groups==. The EFS file system is surrounded by a security group, and EC2 instances need the correct security group rules (NFS port 2049) to connect.

> [!question]- Q9: Can EFS scale to petabyte size?
> **Answer:**
> ==Yes==. EFS can grow to ==petabyte scale automatically== with thousands of concurrent NFS clients and 10+ GB/s throughput. No capacity planning needed.

> [!question]- Q10: When would you choose EFS over EBS?
> **Answer:**
> When you need:
> - ==Shared file system== across multiple instances
> - ==Multi-AZ== access
> - ==Auto-scaling== capacity without provisioning
> - Use cases: WordPress, content management, web serving, data sharing
