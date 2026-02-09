---
title: EC2 Placement Groups
date: 2026-02-03
tags:
  - aws
  - ec2
  - placement-groups
  - high-availability
  - hpc
  - saa-c03
---

# EC2 Placement Groups

## Overview

Placement Groups let you ==control how EC2 instances are placed== on underlying AWS hardware.

> [!tip] Key Concept
> You don't interact directly with hardware, but you tell AWS your placement ==strategy==.

## Three Placement Strategies

| Strategy | Goal | Risk | Use Case |
|----------|------|------|----------|
| **Cluster** | Low latency, high throughput | High (single AZ) | HPC, Big Data |
| **Spread** | Maximize availability | Low | Critical apps |
| **Partition** | Isolated failure domains | Medium | Hadoop, Kafka |

## 1. Cluster Placement Group

> [!important] High Performance, High Risk
> All instances in ==same AZ, same rack== for maximum network performance.

```
┌─────────────────────────────────────────────────────┐
│              Availability Zone (us-east-1a)          │
│  ┌───────────────────────────────────────────────┐  │
│  │              Same Rack / Hardware              │  │
│  │                                               │  │
│  │   ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐        │  │
│  │   │ EC2 │  │ EC2 │  │ EC2 │  │ EC2 │        │  │
│  │   └─────┘  └─────┘  └─────┘  └─────┘        │  │
│  │                                               │  │
│  │        10 Gbps bandwidth between instances    │  │
│  │        Ultra-low latency                      │  │
│  └───────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────┘
```

### Characteristics

| Feature | Details |
|---------|---------|
| Network | ==10 Gbps== bandwidth (with enhanced networking) |
| Latency | Ultra-low latency between instances |
| AZ | ==Single AZ only== |
| Risk | If rack fails, ==all instances fail== |

### Use Cases

- ✅ Big Data jobs needing fast completion
- ✅ High Performance Computing (HPC)
- ✅ Applications needing extremely low latency
- ✅ Tightly-coupled node-to-node communication

### Pros & Cons

| Pros | Cons |
|------|------|
| Highest network performance | Single point of failure |
| Lowest latency | Cannot span AZs |
| Best for HPC workloads | All instances fail together |

## 2. Spread Placement Group

> [!important] Maximum Availability
> Each instance on ==different hardware== to minimize correlated failures.

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  us-east-1a              us-east-1b              us-east-1c     │
│  ┌─────────────┐        ┌─────────────┐        ┌─────────────┐  │
│  │  Hardware 1 │        │  Hardware 3 │        │  Hardware 5 │  │
│  │   ┌─────┐   │        │   ┌─────┐   │        │   ┌─────┐   │  │
│  │   │ EC2 │   │        │   │ EC2 │   │        │   │ EC2 │   │  │
│  │   └─────┘   │        │   └─────┘   │        │   └─────┘   │  │
│  └─────────────┘        └─────────────┘        └─────────────┘  │
│  ┌─────────────┐        ┌─────────────┐        ┌─────────────┐  │
│  │  Hardware 2 │        │  Hardware 4 │        │  Hardware 6 │  │
│  │   ┌─────┐   │        │   ┌─────┐   │        │   ┌─────┐   │  │
│  │   │ EC2 │   │        │   │ EC2 │   │        │   │ EC2 │   │  │
│  │   └─────┘   │        │   └─────┘   │        │   └─────┘   │  │
│  └─────────────┘        └─────────────┘        └─────────────┘  │
│                                                                 │
│  Each instance on DIFFERENT hardware rack                       │
└─────────────────────────────────────────────────────────────────┘
```

### Characteristics

| Feature | Details |
|---------|---------|
| Hardware | Each instance on ==different rack== |
| AZ | Can span ==multiple AZs== |
| Limit | ==7 instances per AZ== per placement group |
| Failure | Hardware failure affects only one instance |

### Use Cases

- ✅ Critical applications
- ✅ Applications requiring high availability
- ✅ Instance failures must be isolated
- ✅ Small number of important instances

### Pros & Cons

| Pros | Cons |
|------|------|
| Reduced simultaneous failure risk | ==Max 7 instances per AZ== |
| Can span multiple AZs | Limited scale |
| Hardware isolation | Not for large deployments |

## 3. Partition Placement Group

> [!important] Isolated Partitions
> Instances spread across ==logical partitions== (different racks), each partition isolated from others.

```
┌─────────────────────────────────────────────────────────────────┐
│  us-east-1a                              us-east-1b             │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  │
│  │  Partition 1    │  │  Partition 2    │  │  Partition 3    │  │
│  │  (Rack A)       │  │  (Rack B)       │  │  (Rack C)       │  │
│  │                 │  │                 │  │                 │  │
│  │  ┌───┐ ┌───┐   │  │  ┌───┐ ┌───┐   │  │  ┌───┐ ┌───┐   │  │
│  │  │EC2│ │EC2│   │  │  │EC2│ │EC2│   │  │  │EC2│ │EC2│   │  │
│  │  └───┘ └───┘   │  │  └───┘ └───┘   │  │  └───┘ └───┘   │  │
│  │  ┌───┐ ┌───┐   │  │  ┌───┐ ┌───┐   │  │  ┌───┐ ┌───┐   │  │
│  │  │EC2│ │EC2│   │  │  │EC2│ │EC2│   │  │  │EC2│ │EC2│   │  │
│  │  └───┘ └───┘   │  │  └───┘ └───┘   │  │  └───┘ └───┘   │  │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘  │
│                                                                 │
│  Partitions don't share racks - isolated from each other        │
└─────────────────────────────────────────────────────────────────┘
```

### Characteristics

| Feature | Details |
|---------|---------|
| Partitions | Up to ==7 partitions per AZ== |
| Instances | ==Hundreds of instances== per group |
| Isolation | Partitions on different racks |
| AZ | Can span ==multiple AZs== |
| Metadata | Can query which partition an instance is in |

### Use Cases

- ✅ HDFS (Hadoop Distributed File System)
- ✅ HBase
- ✅ Cassandra
- ✅ Apache Kafka
- ✅ Large distributed/replicated workloads

### Pros & Cons

| Pros | Cons |
|------|------|
| Scale to hundreds of instances | More complex than spread |
| Partition-level isolation | Application must be partition-aware |
| Good for distributed systems | Not for tightly-coupled HPC |

## Comparison Summary

| Feature | Cluster | Spread | Partition |
|---------|---------|--------|-----------|
| **Goal** | Performance | Availability | Scale + Isolation |
| **AZ** | Single | Multiple | Multiple |
| **Max Instances** | No limit | 7 per AZ | Hundreds |
| **Hardware** | Same rack | Different racks | Different racks per partition |
| **Failure Impact** | All instances | Single instance | Single partition |
| **Use Case** | HPC | Critical apps | Hadoop, Kafka |

## Creating Placement Groups

### Console

```
EC2 Console → Network & Security → Placement Groups → Create
├── Name: my-cluster-group
├── Strategy: Cluster / Spread / Partition
└── (Partition only) Partition count: 1-7
```

### Launch Instance in Placement Group

```
Launch Instance → Advanced Details → Placement group name
└── Select your placement group
```

### AWS CLI

```bash
# Cluster
aws ec2 create-placement-group \
    --group-name my-cluster \
    --strategy cluster

# Spread
aws ec2 create-placement-group \
    --group-name my-critical \
    --strategy spread

# Partition
aws ec2 create-placement-group \
    --group-name my-distributed \
    --strategy partition \
    --partition-count 4
```

## Questions & Answers

> [!question]- Q1: Which placement group provides the lowest network latency?
> **Answer:**
> ==Cluster== placement group. All instances are placed in the same AZ on the same rack, providing up to 10 Gbps bandwidth and ultra-low latency. Used for HPC and tightly-coupled applications.

> [!question]- Q2: What's the maximum number of instances in a Spread placement group per AZ?
> **Answer:**
> ==7 instances per AZ==. This is because each instance must be on different hardware, and there's a limit to how many distinct racks are available. Use Partition for larger deployments.

> [!question]- Q3: Which placement group should you use for Hadoop or Kafka?
> **Answer:**
> ==Partition== placement group. It allows hundreds of instances spread across up to 7 partitions per AZ. Each partition is on different racks, providing isolation. These applications are partition-aware and can distribute data accordingly.

> [!question]- Q4: What happens if the rack fails in a Cluster placement group?
> **Answer:**
> ==All instances fail== simultaneously. This is the trade-off for high performance - you get low latency and high throughput, but all instances share the same hardware failure domain.

> [!question]- Q5: Can a Spread placement group span multiple Availability Zones?
> **Answer:**
> ==Yes==. Spread placement groups can span multiple AZs within a region. Each instance is placed on distinct hardware, and you can have up to 7 instances per AZ in the group.

> [!question]- Q6: How many partitions can you have per AZ in a Partition placement group?
> **Answer:**
> Up to ==7 partitions per AZ==. Each partition represents a different hardware rack. You can scale to ==hundreds of EC2 instances== across all partitions, unlike Spread which limits to 7 instances per AZ.

> [!question]- Q7: What does each partition represent in a Partition placement group?
> **Answer:**
> Each partition represents a ==separate hardware rack== in AWS. Instances within the same partition may share a rack, but instances in ==different partitions are guaranteed to be on different racks==, isolating failure domains.

> [!question]- Q8: How can an EC2 instance know which partition it belongs to?
> **Answer:**
> Through the ==EC2 metadata service==. Instances can query their metadata to determine which partition they are placed in, allowing partition-aware applications (like HDFS, Cassandra) to distribute data accordingly.

> [!question]- Q9: What is the Spread level option when creating a Spread placement group?
> **Answer:**
> You can set the spread level to:
> - ==Rack== (default) — instances spread across different racks
> - ==Host== — instances spread across different physical hosts (Outpost only)
> 
> For standard AWS usage, ==rack== is the correct choice.

> [!question]- Q10: When would you choose Cluster over Partition placement group?
> **Answer:**
> Choose ==Cluster== when you need the ==absolute lowest latency and highest throughput== between instances (e.g., tightly-coupled HPC). Choose ==Partition== when you need ==scale + fault isolation== for distributed systems like Kafka or Hadoop that are partition-aware.
