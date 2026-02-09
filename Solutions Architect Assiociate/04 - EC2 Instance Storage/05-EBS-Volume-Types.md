---
title: EBS Volume Types
date: 2026-02-09
tags:
  - aws
  - ec2
  - ebs
  - volume-types
  - iops
  - saa-c03
---

# EBS Volume Types

## Six Volume Types

| Category | Types | Drive | Boot Volume? |
|----------|-------|-------|-------------|
| **General Purpose SSD** | gp2, gp3 | SSD | ✅ Yes |
| **Provisioned IOPS SSD** | io1, io2 Block Express | SSD | ✅ Yes |
| **Throughput Optimized HDD** | st1 | HDD | ❌ No |
| **Cold HDD** | sc1 | HDD | ❌ No |

> [!warning] Boot Volumes
> Only ==gp2, gp3, io1, io2== can be used as boot volumes (root OS).

## General Purpose SSD (gp2 / gp3)

| Feature | gp3 (newer) | gp2 (older) |
|---------|-------------|-------------|
| **Baseline IOPS** | ==3,000== | Linked to size |
| **Max IOPS** | 16,000 | 16,000 |
| **Baseline Throughput** | 125 MB/s | Linked to size |
| **Max Throughput** | 1,000 MB/s | 250 MB/s |
| **IOPS/Throughput** | ==Independent== | ==Linked to volume size== |
| **Size** | 1 GB - 16 TB | 1 GB - 16 TB |
| **Burst** | N/A | Up to 3,000 IOPS |

> [!tip] gp2 vs gp3
> In ==gp3==, IOPS and throughput are ==independently configurable==.
> In ==gp2==, IOPS scales with volume size (3 IOPS per GB, max at 5,334 GB).

```
gp2 IOPS scaling:
├── 1 GB    → 100 IOPS (minimum, burst to 3,000)
├── 1,000 GB → 3,000 IOPS
├── 5,334 GB → 16,000 IOPS (maxed out)
└── 16 TB   → 16,000 IOPS (still maxed)
```

## Provisioned IOPS SSD (io1 / io2)

For ==critical business applications== needing sustained IOPS or >16,000 IOPS.

| Feature | io1 | io2 Block Express |
|---------|-----|-------------------|
| **Size** | 4 GB - 16 TB | 4 GB - ==64 TB== |
| **Max IOPS** | 64,000 (Nitro) / 32,000 (other) | ==256,000== |
| **Latency** | Low | ==Sub-millisecond== |
| **IOPS:GB ratio** | 50:1 | ==1,000:1== |
| **Multi-Attach** | ✅ Yes | ✅ Yes |

> [!important] Exam Tip
> Database workloads sensitive to storage performance → ==io1/io2==.
> Need >32,000 IOPS → ==EC2 Nitro + io1/io2==.

## HDD Volumes (st1 / sc1)

| Feature | st1 (Throughput Optimized) | sc1 (Cold HDD) |
|---------|---------------------------|-----------------|
| **Use case** | Big data, data warehouse, logs | Archive, infrequent access |
| **Max Throughput** | ==500 MB/s== | 250 MB/s |
| **Max IOPS** | 500 | 250 |
| **Size** | Up to 16 TB | Up to 16 TB |
| **Cost** | Low | ==Lowest== |
| **Boot volume?** | ❌ No | ❌ No |

## EBS Multi-Attach (io1/io2 only)

> [!note] Multi-Attach
> Attach the ==same EBS volume to multiple EC2 instances== in the ==same AZ==.

| Feature | Details |
|---------|---------|
| **Volume types** | ==io1 and io2 only== |
| **Max instances** | ==Up to 16== at a time |
| **AZ** | Same AZ only |
| **Permissions** | Full read/write for all instances |
| **File system** | Must use ==cluster-aware== file system (not XFS/ext4) |
| **Use case** | Clustered Linux apps (Teradata), concurrent writes |

```
┌──────────┐
│   EC2    │──┐
│ Instance │  │
│    A     │  │     ┌──────────────┐
└──────────┘  ├────▶│  io2 Volume  │
              │     │ Multi-Attach │
┌──────────┐  │     └──────────────┘
│   EC2    │──┘
│ Instance │
│    B     │        Same AZ only!
└──────────┘        Max 16 instances
```

## Summary Comparison

| Volume | IOPS | Throughput | Use Case |
|--------|------|-----------|----------|
| **gp3** | 3K-16K | 125-1000 MB/s | General workloads |
| **gp2** | 3K-16K (linked) | Up to 250 MB/s | General (legacy) |
| **io1** | Up to 64K | — | Databases |
| **io2 BE** | Up to 256K | — | Critical databases |
| **st1** | 500 | 500 MB/s | Big data, logs |
| **sc1** | 250 | 250 MB/s | Archive, cold data |

## Questions & Answers

> [!question]- Q1: Which EBS volume types can be used as boot volumes?
> **Answer:**
> Only SSD types: ==gp2, gp3, io1, and io2==. HDD types (st1, sc1) ==cannot== be boot volumes.

> [!question]- Q2: What is the key difference between gp2 and gp3?
> **Answer:**
> In ==gp3==, IOPS and throughput are ==independently configurable==. In ==gp2==, IOPS is linked to volume size (3 IOPS per GB). gp3 also has a higher baseline (3,000 IOPS, 125 MB/s).

> [!question]- Q3: When should you use io1/io2 volumes?
> **Answer:**
> For ==critical database workloads== that need sustained high IOPS (>16,000), low latency, or the Multi-Attach feature. Think: production databases sensitive to storage performance.

> [!question]- Q4: What is EBS Multi-Attach?
> **Answer:**
> The ability to attach the ==same io1/io2 volume to up to 16 EC2 instances== in the same AZ. All instances get full read/write access. Requires a cluster-aware file system.

> [!question]- Q5: How do you get more than 32,000 IOPS on EBS?
> **Answer:**
> Use ==EC2 Nitro instances with io1 or io2== volumes. Nitro instances support up to 64,000 IOPS (io1) or 256,000 IOPS (io2 Block Express).

> [!question]- Q6: What is the cheapest EBS volume type?
> **Answer:**
> ==sc1 (Cold HDD)== — designed for infrequently accessed archive data with the lowest cost. Max 250 IOPS and 250 MB/s throughput.

> [!question]- Q7: Which volume type is best for big data and log processing?
> **Answer:**
> ==st1 (Throughput Optimized HDD)== — designed for frequently accessed, throughput-intensive workloads like big data, data warehousing, and log processing. Up to 500 MB/s throughput.

> [!question]- Q8: What is the maximum size of an io2 Block Express volume?
> **Answer:**
> ==64 TB== with up to 256,000 IOPS and sub-millisecond latency. The IOPS:GB ratio is 1,000:1.

> [!question]- Q9: What file system is required for Multi-Attach?
> **Answer:**
> A ==cluster-aware file system== is required (not standard XFS or ext4). This is because multiple instances are writing to the same volume concurrently and need coordination.

> [!question]- Q10: At what gp2 volume size do you max out IOPS?
> **Answer:**
> At ==5,334 GB==. Since gp2 provides 3 IOPS per GB, 5,334 × 3 = 16,000 IOPS (the maximum). Any size above this still gets 16,000 IOPS.
