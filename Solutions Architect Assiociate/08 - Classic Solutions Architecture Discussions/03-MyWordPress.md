---
title: "MyWordPress.com - Scalable Storage"
date: 2026-02-10
tags:
  - aws
  - solutions-architecture
  - ebs
  - efs
  - aurora
  - saa-c03
---

# MyWordPress.com - Scalable Storage

## Overview

Yet another stateful web application — this time we're building a fully scalable ==WordPress== website on AWS. WordPress is one of the most popular ways to create a website, and many people deploy it on AWS.

The unique challenge here isn't about sessions or shopping carts — it's about ==image storage==. WordPress stores uploaded pictures (blog images, media) on a drive, and when you scale to multiple instances across multiple AZs, all instances need access to those images. This case study highlights a critical architectural decision: ==EBS vs EFS==.

**Key requirements:**
- Fully scalable WordPress website
- Correctly display picture uploads across all instances
- Blog content and user data stored in a MySQL database
- Must work across multiple AZs for high availability

## Database Layer: Why Aurora MySQL?

We're already familiar with the standard architecture — Route 53, ELB, ASG, database in the backend. For WordPress, we need a MySQL-compatible database. We have two choices:

```
Option A: Standard RDS MySQL          Option B: Aurora MySQL
┌──────────────────────┐              ┌──────────────────────┐
│  RDS MySQL           │              │  Aurora MySQL        │
│  - Manual Multi-AZ   │              │  - Built-in Multi-AZ │
│  - Up to 5 replicas  │              │  - Up to 15 replicas │
│  - Manual storage    │              │  - Auto-scaling      │
│  - Standard perf     │              │    storage           │
│                      │              │  - 5x MySQL perf     │
│                      │              │  - Less ops overhead │
│                      │              │  - Global databases  │
└──────────────────────┘              └──────────────────────┘
```

As a Solutions Architect, choosing Aurora is a deliberate decision. You don't ==have== to — RDS MySQL works fine. But Aurora gives you:
- ==Less operational overhead== — AWS manages more of the infrastructure
- ==Built-in Multi-AZ== with automatic failover
- ==Up to 15 read replicas== (vs 5 for standard RDS)
- ==Auto-scaling storage== — grows automatically in 10 GB increments up to 128 TB
- ==5x MySQL throughput== — better performance out of the box
- ==Global databases== if you need to scale internationally

> [!tip] Solutions Architect Mindset
> It's not about always picking the most expensive option. It's about understanding the ==trade-offs==. Aurora costs more than RDS MySQL, but it gives you better scaling, less maintenance, and higher performance. For a production WordPress site with growing traffic, Aurora is often the right choice.

## The Image Storage Problem: EBS

Now let's talk about the real challenge of this case study — storing and serving uploaded images.

### Single Instance: EBS Works Perfectly

```
┌──────────┐   ┌─────┐   ┌────────────────┐   ┌──────────────┐
│          │   │     │   │  EC2 Instance  │   │  EBS Volume  │
│   User   │──▶│ ELB │──▶│  (AZ1)        │──▶│  (AZ1)       │
│          │   │     │   │               │   │              │
│  Upload  │   │     │   │  Stores image │   │  image.jpg   │
│  image   │   │     │   │  on EBS       │   │  ✅ HERE     │
└──────────┘   └─────┘   └────────────────┘   └──────────────┘
```

With a single EC2 instance, everything is simple. The user uploads an image, it gets stored on the EBS volume attached to that instance. When someone wants to view the image, the same instance reads it from EBS and serves it. ==Works perfectly.==

### Multiple Instances: EBS Breaks

```
┌──────────┐   ┌─────┐   ┌────────────────┐   ┌──────────────┐
│          │   │     │   │  EC2 (AZ1)     │──▶│  EBS (AZ1)   │
│   User   │──▶│ ELB │──▶│  Upload image  │   │  image.jpg   │
│          │   │     │   │               │   │  ✅ HERE     │
│          │   │     │   └────────────────┘   └──────────────┘
│          │   │     │
│  Read    │   │     │   ┌────────────────┐   ┌──────────────┐
│  image   │──▶│     │──▶│  EC2 (AZ2)     │──▶│  EBS (AZ2)   │
│          │   │     │   │  Read image    │   │              │
│          │   │     │   │  ❌ NOT FOUND! │   │  ❌ EMPTY    │
└──────────┘   └─────┘   └────────────────┘   └──────────────┘
```

Now we have two instances in different AZs, each with its own EBS volume. A user uploads an image through Instance A — it gets stored on EBS in AZ1. Later, the same user (or another user) tries to view that image, but the request gets routed to Instance B in AZ2. Instance B looks at its own EBS volume — ==the image isn't there==.

This is a ==very common mistake== when scaling WordPress on AWS. EBS volumes are:
- ==AZ-specific== — an EBS volume in AZ1 cannot be accessed from AZ2
- ==Attached to a single instance== — not shared between instances (with limited exceptions)

> [!danger] EBS Limitation
> EBS works great for single-instance applications. But the moment you scale to multiple instances across AZs, EBS becomes a problem because each instance has its own ==isolated storage==. Files uploaded to one instance's EBS are invisible to all other instances.

## The Solution: EFS (Elastic File System)

```
┌──────────┐   ┌─────┐   ┌────────────────┐   ┌───────────┐   ┌─────────────────┐
│          │   │     │   │  EC2 (AZ1)     │──▶│ ENI (AZ1) │──▶│                 │
│   User   │──▶│ ELB │──▶│  Upload image  │   └───────────┘   │      EFS        │
│          │   │     │   └────────────────┘                    │   (Shared NFS)  │
│          │   │     │                                          │                 │
│  Read    │   │     │   ┌────────────────┐   ┌───────────┐   │   image.jpg     │
│  image   │──▶│     │──▶│  EC2 (AZ2)     │──▶│ ENI (AZ2) │──▶│   ✅ FOUND!    │
│          │   │     │   │  Read image    │   └───────────┘   │                 │
│          │   │     │   │  ✅ SUCCESS!   │                    │   Shared across │
└──────────┘   └─────┘   └────────────────┘                    │   ALL instances │
                                                                │   ALL AZs       │
                                                                └─────────────────┘
```

EFS (Elastic File System) is a ==Network File System (NFS)== that solves the shared storage problem:

1. EFS creates ==ENIs (Elastic Network Interfaces)== in each AZ — these are mount targets
2. EC2 instances in each AZ connect to EFS through their local ENI
3. All instances see the ==exact same file system== — it's shared storage
4. Upload an image from Instance A → it's ==immediately visible== from Instance B, C, D, etc.
5. Works across ==any number of instances== and ==any number of AZs==

## EBS vs EFS: The Complete Comparison

| Feature | EBS | EFS |
|---------|-----|-----|
| **Scope** | ==Single AZ== | ==Multi-AZ== |
| **Attachment** | One instance at a time | ==Unlimited instances== across AZs |
| **Protocol** | Block storage (like a hard drive) | ==NFS== (Network File System) |
| **Use Case** | Boot volumes, databases, single-instance apps | ==Shared storage== across instances |
| **Scaling** | Manual resize (must modify volume) | ==Automatic== — grows and shrinks with usage |
| **Cost** | ==Cheaper== per GB | More expensive per GB |
| **Performance** | Very high (especially io1/io2) | Good, with burst capability |
| **WordPress** | Works for ==single instance only== | ==Required for multi-instance== WordPress |

> [!important] Cost vs Capability Trade-off
> EFS is ==more expensive== than EBS per GB. But for multi-instance architectures like WordPress, it's ==necessary==. You can reduce EFS costs by using the ==EFS Infrequent Access (IA)== storage class for files that aren't accessed often. As a Solutions Architect, you need to understand this trade-off.

### Can EBS Multi-Attach Replace EFS?

No. EBS Multi-Attach has severe limitations:
- Only works with ==io1/io2 volume types== (expensive)
- Only works within the ==same AZ== (defeats the purpose of Multi-AZ)
- Requires a ==cluster-aware file system== (not standard Linux file systems)
- Not designed for general shared storage

## The Complete WordPress Architecture

```
┌──────────┐   ┌──────────┐   ┌───────────────────┐   ┌──────────────────────────┐
│          │   │ Route 53 │   │    Multi-AZ ELB   │   │         ASG              │
│  Users   │──▶│  Alias   │──▶│  + Health Checks  │──▶│  AZ1: EC2  EC2          │
│          │   │          │   │                   │   │  AZ2: EC2  EC2          │
└──────────┘   └──────────┘   └───────────────────┘   └───────────┬──────────────┘
                                                                    │
                              ┌─────────────────────────────────────┼──────────────┐
                              ▼                                     ▼              │
                    ┌──────────────────┐                  ┌──────────────────┐     │
                    │   Aurora MySQL   │                  │       EFS        │     │
                    │   Multi-AZ       │                  │   Shared NFS     │     │
                    │   Read Replicas  │                  │   Media/Images   │     │
                    │                  │                  │   Multi-AZ       │     │
                    │   Blog content   │                  │                  │     │
                    │   User data      │                  │   ENI in AZ1     │◀────┘
                    │   Comments       │                  │   ENI in AZ2     │
                    └──────────────────┘                  └──────────────────┘
```

The complete architecture has three storage layers:
1. ==Aurora MySQL== — blog content, user data, comments, settings (structured data)
2. ==EFS== — uploaded images, media files, WordPress themes/plugins (shared files)
3. ==EC2 local storage== — WordPress application code, temporary files

## Key Takeaways

| Decision | Choice | Why |
|----------|--------|-----|
| **Database** | Aurora MySQL over RDS MySQL | Less ops, better scaling, 15 replicas, auto-scaling storage |
| **Image Storage** | EFS over EBS | EBS is AZ-bound and not shared; EFS works across all instances and AZs |
| **Cost** | EFS costs more than EBS | But it's ==required== for multi-instance WordPress |
| **Availability** | Multi-AZ everything | Aurora Multi-AZ, EFS Multi-AZ, ELB Multi-AZ, ASG Multi-AZ |

> [!note] Solutions Architect Responsibility
> As a Solutions Architect, it's your job to understand ==why== you're making each decision and what the ==cost implications== are. "We chose EFS over EBS because EBS doesn't support shared access across AZs, which is required for our multi-instance WordPress deployment."

## Questions & Answers

> [!question]- Q1: Why does EBS fail for multi-instance WordPress?
> **Answer:**
> EBS volumes are ==AZ-specific== — a volume in AZ1 cannot be accessed from AZ2. They're also attached to a ==single instance== at a time. When a user uploads an image through Instance A (stored on EBS in AZ1), Instance B in AZ2 has its own separate EBS volume that ==doesn't contain that image==. The user sees a broken image or 404 error.

> [!question]- Q2: How does EFS solve the shared storage problem?
> **Answer:**
> EFS is a ==Network File System (NFS)== that creates mount targets (ENIs) in each AZ. All EC2 instances mount the ==same file system== through their local ENI. When Instance A uploads an image, it's written to EFS and ==immediately visible== from Instance B, C, D, etc. — regardless of which AZ they're in.

> [!question]- Q3: Why choose Aurora MySQL over standard RDS MySQL for WordPress?
> **Answer:**
> Aurora provides significant advantages for production WordPress:
> - ==Less operational overhead== — AWS manages more infrastructure
> - ==Built-in Multi-AZ== with automatic failover
> - ==Up to 15 read replicas== (vs 5 for standard RDS)
> - ==Auto-scaling storage== — grows in 10 GB increments up to 128 TB
> - ==5x MySQL throughput== — better performance
> The trade-off is higher cost, but for production workloads the reduced ops burden and better scaling are worth it.

> [!question]- Q4: What is an ENI in the context of EFS?
> **Answer:**
> An ==Elastic Network Interface (ENI)== is a virtual network card. EFS creates one ENI per AZ as a ==mount target==. EC2 instances in that AZ connect to EFS through the local ENI, which provides low-latency access to the shared file system. The ENI is the bridge between your EC2 instances and the EFS file system.

> [!question]- Q5: When would you still use EBS over EFS?
> **Answer:**
> EBS is the right choice when:
> - You have a ==single-instance== application (cheaper than EFS)
> - You need ==boot volumes== for EC2 instances (EFS can't be a boot volume)
> - You need ==high-performance block storage== (io1/io2 for databases)
> - You don't need ==shared access== across instances
> - Cost is a primary concern and you don't need multi-instance sharing

> [!question]- Q6: What is the cost implication of using EFS vs EBS?
> **Answer:**
> EFS is ==more expensive per GB== than EBS. However:
> - EFS ==auto-scales== — you only pay for what you use (no over-provisioning)
> - EFS has an ==Infrequent Access (IA)== storage class that's much cheaper for rarely accessed files
> - EBS requires you to ==provision a fixed size== — you might pay for unused capacity
> - For multi-instance WordPress, EFS is ==required== — there's no EBS alternative that works across AZs

> [!question]- Q7: Can EBS Multi-Attach replace EFS for this use case?
> **Answer:**
> No. EBS Multi-Attach has severe limitations:
> - Only works with ==io1/io2 volumes== (very expensive)
> - Only works within the ==same AZ== (can't span AZs)
> - Requires a ==cluster-aware file system== (not ext4 or xfs)
> - Maximum ==16 instances== can attach simultaneously
> EFS works across AZs with standard NFS, supports unlimited instances, and is much simpler to set up.

> [!question]- Q8: What is the complete WordPress architecture on AWS?
> **Answer:**
> - ==Route 53== → Alias Record → ALB (Multi-AZ) with health checks
> - ==ASG== with EC2 instances across multiple AZs running WordPress
> - ==Aurora MySQL== (Multi-AZ) for database — blog content, user data, comments
> - ==EFS== (Multi-AZ) for shared media storage — images, uploads, themes, plugins
> - All layers are Multi-AZ for high availability

> [!question]- Q9: How does Aurora auto-scaling storage work?
> **Answer:**
> Aurora storage automatically grows in ==10 GB increments== up to ==128 TB==. You don't need to provision storage upfront or manually resize. Aurora continuously monitors usage and adds storage as needed. You only pay for the storage you actually use. This eliminates the risk of running out of disk space and the operational burden of manual resizing.

> [!question]- Q10: What are the key architectural decisions a Solutions Architect must make for WordPress on AWS?
> **Answer:**
> Three critical decisions, each with a cost vs. capability trade-off:
> 1. ==Database==: RDS MySQL (cheaper, more manual) vs Aurora MySQL (more expensive, better scaling, less ops)
> 2. ==File Storage==: EBS (cheaper, single-instance only) vs EFS (more expensive, required for multi-instance)
> 3. ==Availability==: Single-AZ (cheaper, for dev/test) vs Multi-AZ (more expensive, required for production)
> The exam tests your ability to ==justify these decisions== based on requirements.
