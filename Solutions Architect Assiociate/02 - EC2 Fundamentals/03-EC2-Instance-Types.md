---
title: EC2 Instance Types
date: 2026-02-03
tags:
  - aws
  - ec2
  - instance-types
  - saa-c03
---

# EC2 Instance Types

## Instance Type Naming Convention

```
  m   5   .   2xlarge
  │   │       │
  │   │       └── Size (within class)
  │   │           small → large → xlarge → 2xlarge → 4xlarge...
  │   │
  │   └── Generation (hardware version)
  │       AWS improves over time: m5 → m6 → m7
  │
  └── Instance Class
      Determines optimization type
```

### Example Breakdown

| Instance | Class | Generation | Size |
|----------|-------|------------|------|
| t2.micro | t (General) | 2 | micro |
| m5.2xlarge | m (General) | 5 | 2xlarge |
| c5d.4xlarge | c (Compute) | 5d | 4xlarge |
| r5.16xlarge | r (Memory) | 5 | 16xlarge |

## Instance Type Categories

### 1. General Purpose

> [!tip] Balanced Resources
> Good balance between compute, memory, and networking.

**Use Cases:**
- Web servers
- Code repositories
- Development environments
- Small/medium databases

**Instance Families:** `t2`, `t3`, `m5`, `m6i`, `m7g`

```
General Purpose
├── t2.micro (Free Tier) ─── 1 vCPU, 1 GB RAM
├── t3.medium ────────────── 2 vCPU, 4 GB RAM
└── m5.large ─────────────── 2 vCPU, 8 GB RAM
```

### 2. Compute Optimized

> [!tip] High-Performance Processors
> Ideal for compute-intensive tasks requiring high CPU.

**Use Cases:**
- Batch processing
- Media transcoding
- High-performance web servers
- High-performance computing (HPC)
- Scientific modeling
- Machine learning inference
- Dedicated gaming servers

**Instance Families:** `c5`, `c6i`, `c7g` (C = Compute)

```
Compute Optimized
├── c5.large ──────── 2 vCPU, 4 GB RAM
├── c5.4xlarge ────── 16 vCPU, 32 GB RAM
└── c6i.8xlarge ───── 32 vCPU, 64 GB RAM
```

### 3. Memory Optimized

> [!tip] Large Memory Capacity
> Fast performance for workloads processing large datasets in memory.

**Use Cases:**
- High-performance relational/non-relational databases
- In-memory databases (ElastiCache)
- Distributed web-scale cache stores
- Real-time processing of big unstructured data
- Business intelligence (BI)

**Instance Families:** `r5`, `r6i`, `x1`, `x2`, `z1d` (R = RAM)

```
Memory Optimized
├── r5.large ──────── 2 vCPU, 16 GB RAM
├── r5.16xlarge ───── 64 vCPU, 512 GB RAM
└── x1.32xlarge ───── 128 vCPU, 1,952 GB RAM
```

### 4. Storage Optimized

> [!tip] High Sequential Read/Write
> High sequential read/write access to large datasets on local storage.

**Use Cases:**
- High-frequency online transaction processing (OLTP)
- Relational & NoSQL databases
- Data warehousing
- Distributed file systems
- Cache for in-memory databases (Redis)

**Instance Families:** `i3`, `i4i`, `d2`, `d3`, `h1`

```
Storage Optimized
├── i3.large ──────── 2 vCPU, 15.25 GB RAM, 475 GB NVMe SSD
├── d2.xlarge ─────── 4 vCPU, 30.5 GB RAM, 6 TB HDD
└── h1.2xlarge ────── 8 vCPU, 32 GB RAM, 2 TB HDD
```

### 5. Accelerated Computing

**Use Cases:**
- Machine learning training
- Graphics processing
- Video encoding
- Floating-point calculations

**Instance Families:** `p4`, `p5`, `g4`, `g5`, `inf1`, `trn1`

## Instance Type Comparison

| Type | vCPU | Memory | Storage | Network | Use Case |
|------|------|--------|---------|---------|----------|
| t2.micro | 1 | 1 GB | EBS | Low-Mod | Free tier |
| t2.xlarge | 4 | 16 GB | EBS | Moderate | Small apps |
| c5d.4xlarge | 16 | 32 GB | 400 GB NVMe | Up to 10 Gbps | Compute |
| r5.16xlarge | 64 | 512 GB | EBS | 20 Gbps | Memory |
| m5.8xlarge | 32 | 128 GB | EBS | 10 Gbps | General |

## Free Tier Instance

> [!important] Free Tier — Depends on Account Creation Date
> **Accounts created ==before== July 15, 2025:**
> - ==t2.micro== (or t3.micro where t2 unavailable)
> - 750 hours/month for first 12 months
> - 1 vCPU, 1 GB RAM, EBS storage only
> - ⚠️ t3.micro defaults to Unlimited mode (may incur CPU burst charges)
>
> **Accounts created ==on or after== July 15, 2025:**
> - ==t3.micro, t3.small, t4g.micro, t4g.small, c7i-flex.large, m7i-flex.large==
> - Available for ==6 months== or until credits are used up
> - Broader instance selection including Graviton (t4g) and Flex types

## Choosing the Right Instance Type

```
Decision Tree
│
├── Need balanced resources?
│   └── General Purpose (t3, m5, m6i)
│
├── CPU-intensive workload?
│   └── Compute Optimized (c5, c6i)
│
├── Large in-memory datasets?
│   └── Memory Optimized (r5, x1)
│
├── High disk I/O?
│   └── Storage Optimized (i3, d2)
│
└── ML/Graphics?
    └── Accelerated Computing (p4, g5)
```

## Useful Resources

- **AWS Instance Types Page**: https://aws.amazon.com/ec2/instance-types/
- **EC2Instances.info**: https://instances.vantage.sh/ (comparison tool)

## Questions & Answers

> [!question]- Q1: What does m5.2xlarge mean?
> **Answer:**
> - `m` = Instance class (General Purpose)
> - `5` = Generation (5th generation hardware)
> - `2xlarge` = Size (determines vCPU and memory)
> 
> As AWS improves hardware, generation increases (m5 → m6 → m7).

> [!question]- Q2: Which instance type for a high-performance database?
> **Answer:**
> - **Memory Optimized** (r5, r6i, x1) for in-memory databases
> - **Storage Optimized** (i3, d2) for high disk I/O databases
> 
> Choice depends on whether workload is memory-bound or I/O-bound.

> [!question]- Q3: Which instance type for batch processing jobs?
> **Answer:**
> **Compute Optimized** (c5, c6i) - designed for CPU-intensive tasks like batch processing, scientific modeling, and ML inference.

> [!question]- Q4: What's the free tier instance type?
> **Answer:**
> Depends on when your account was created:
> - **Before July 15, 2025**: ==t2.micro== (or t3.micro if t2 unavailable) — 750 hours/month, first 12 months
> - **On or after July 15, 2025**: ==t3.micro, t3.small, t4g.micro, t4g.small, c7i-flex.large, m7i-flex.large== — 6 months or until credits used up
