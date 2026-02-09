---
title: EC2 Purchasing Options
date: 2026-02-03
tags:
  - aws
  - ec2
  - pricing
  - reserved
  - spot
  - savings-plans
  - saa-c03
---

# EC2 Purchasing Options

## Overview

```
EC2 Purchasing Options
├── On-Demand ─────────────── Pay per second, no commitment
├── Reserved Instances ────── 1-3 year commitment, up to 72% off
├── Savings Plans ─────────── Commit $/hour, up to 72% off
├── Spot Instances ────────── Up to 90% off, can be interrupted
├── Dedicated Hosts ───────── Physical server, BYOL
├── Dedicated Instances ───── Single-tenant hardware
└── Capacity Reservations ─── Reserve capacity, no discount
```

## 1. On-Demand Instances

> [!tip] Maximum Flexibility
> Pay for what you use with no long-term commitment.

| Feature | Details |
|---------|---------|
| Billing | Per second (Linux/Windows), per hour (other OS) |
| Minimum | 60 seconds |
| Commitment | None |
| Discount | None (baseline price) |

**Best For:**
- Short-term, irregular workloads
- Applications being developed/tested
- Workloads that cannot be interrupted
- First-time applications with unpredictable usage

```
On-Demand Pricing Example (m4.large, us-east-1):
$0.10 per hour = $72/month (running 24/7)
```

## 2. Reserved Instances

> [!tip] Predictable Workloads
> Commit to specific instance configuration for 1-3 years.

### Discount Levels

| Term | Payment | Discount |
|------|---------|----------|
| 1 Year | No Upfront | ~36% |
| 1 Year | Partial Upfront | ~42% |
| 1 Year | All Upfront | ~43% |
| 3 Year | No Upfront | ~56% |
| 3 Year | Partial Upfront | ~66% |
| 3 Year | All Upfront | ==Up to 72%== |

### What You Reserve

- Instance Type (e.g., m5.large)
- Region
- Tenancy (default/dedicated)
- Operating System

### Standard vs Convertible

| Type | Discount | Flexibility |
|------|----------|-------------|
| **Standard** | Up to 72% | Cannot change instance type |
| **Convertible** | Up to 66% | Can change instance type, family, OS, tenancy |

**Best For:**
- Steady-state applications
- Databases
- Applications with predictable usage

> [!note] Reserved Instance Marketplace
> You can buy/sell unused Reserved Instances in the marketplace.

## 3. Savings Plans

> [!tip] Modern Alternative to Reserved
> Commit to a ==dollar amount per hour== instead of specific instance type.

### Types

| Plan | Flexibility | Discount |
|------|-------------|----------|
| **Compute Savings Plan** | Any instance family, region, OS, tenancy | Up to 66% |
| **EC2 Instance Savings Plan** | Specific instance family in a region | Up to 72% |

### How It Works

```
Commitment: $10/hour for 1 year

Usage Scenarios:
├── Use $8/hour  → Pay $8 at discounted rate
├── Use $10/hour → Pay $10 at discounted rate
└── Use $15/hour → Pay $10 discounted + $5 On-Demand
```

**Flexibility:**
- ✅ Change instance size (m5.large → m5.xlarge)
- ✅ Change OS (Linux → Windows)
- ✅ Change tenancy
- ❌ Locked to instance family (m5) and region

**Best For:**
- Long-term workloads with flexible instance needs
- Organizations wanting simpler commitment model

## 4. Spot Instances

> [!danger] Can Be Interrupted
> Highest discount but instances can be reclaimed with 2-minute warning.

### Pricing

- ==Up to 90% discount== compared to On-Demand
- Price fluctuates based on supply/demand
- You set a ==max price== you're willing to pay

```
Spot Pricing Mechanism:
┌─────────────────────────────────────────┐
│  $0.10 ─ ─ ─ On-Demand Price ─ ─ ─ ─   │
│                                         │
│  $0.04 ════ Your Max Price ════════════ │
│         ╱╲    ╱╲                        │
│  $0.03 ╱  ╲  ╱  ╲   Spot Price         │
│       ╱    ╲╱    ╲                      │
│                                         │
│  When Spot > Max: 2 min to stop/term    │
└─────────────────────────────────────────┘
```

### Interruption Behavior

When spot price exceeds your max price:
- **2-minute warning** before interruption
- Options: **Stop** or **Terminate**

### Spot Request Types

| Type | Behavior |
|------|----------|
| **One-time** | Request fulfilled once, then done |
| **Persistent** | Automatically restarts when price allows |

> [!warning] Terminating Persistent Spot
> 1. ==First== cancel the spot request
> 2. ==Then== terminate the instances
> 
> If you terminate first, persistent request will relaunch instances!

**Best For:**
- Batch processing
- Data analysis
- Image processing
- Distributed workloads
- Flexible start/end time jobs

**NOT For:**
- ❌ Critical jobs
- ❌ Databases
- ❌ Anything that can't handle interruption

## 5. Spot Fleets

> [!tip] Smart Spot Management
> Automatically request Spot Instances from multiple pools.

### Components

- Set of Spot Instances + optional On-Demand
- Multiple launch pools (instance types, AZs, OS)
- Target capacity (instances, vCPUs, or memory)

### Allocation Strategies

| Strategy | Description | Best For |
|----------|-------------|----------|
| **lowestPrice** | Launch from cheapest pool | Short workloads, max savings |
| **diversified** | Spread across all pools | Long workloads, availability |
| **capacityOptimized** | Pool with most capacity | Reducing interruptions |
| **priceCapacityOptimized** | Best capacity + lowest price | ==Most workloads (recommended)== |

```
Spot Fleet Example:
├── Pool 1: c5.large in us-east-1a
├── Pool 2: c5.large in us-east-1b
├── Pool 3: m5.large in us-east-1a
└── Pool 4: m5.large in us-east-1b

Strategy: lowestPrice
Result: Fleet launches from cheapest available pool
```

## 6. Dedicated Hosts

> [!tip] Physical Server Access
> Book an entire physical server for your exclusive use.

### Features

- Access to physical server
- Visibility into sockets, cores, host ID
- Control over instance placement
- Can be purchased On-Demand or Reserved

### Use Cases

- **BYOL** (Bring Your Own License) - per-socket, per-core licensing
- **Compliance requirements** - dedicated hardware
- **Regulatory requirements** - physical isolation

### Pricing

- Most expensive option
- On-Demand: Pay per second
- Reserved: 1-3 year commitment for discount

## 7. Dedicated Instances

| Feature | Dedicated Instance | Dedicated Host |
|---------|-------------------|----------------|
| Hardware | Dedicated to you | Dedicated to you |
| Physical server visibility | ❌ No | ✅ Yes |
| Instance placement control | ❌ No | ✅ Yes |
| Socket/core visibility | ❌ No | ✅ Yes |
| BYOL support | Limited | ✅ Full |

**Use Case:** Compliance requiring dedicated hardware without needing physical server access.

## 8. Capacity Reservations

> [!tip] Guaranteed Capacity
> Reserve capacity in specific AZ without commitment.

### Features

- Reserve On-Demand capacity in specific AZ
- No time commitment (cancel anytime)
- ==No billing discount==
- Charged On-Demand rate whether used or not

### Use Cases

- Short-term, uninterrupted workloads in specific AZ
- Disaster recovery capacity
- Planned events requiring guaranteed capacity

> [!note] Combine for Discounts
> Combine Capacity Reservations with Reserved Instances or Savings Plans to get both guaranteed capacity AND discounts.

## Comparison Summary

| Option | Discount | Commitment | Interruption | Best For |
|--------|----------|------------|--------------|----------|
| On-Demand | 0% | None | No | Short-term, unpredictable |
| Reserved | Up to 72% | 1-3 years | No | Steady-state, databases |
| Savings Plans | Up to 72% | 1-3 years | No | Flexible long-term |
| Spot | Up to 90% | None | ==Yes== | Fault-tolerant batch |
| Dedicated Host | Varies | Optional | No | BYOL, compliance |
| Dedicated Instance | Varies | None | No | Compliance |
| Capacity Reservation | 0% | None | No | Guaranteed capacity |

## Hotel Analogy

| Option | Hotel Analogy |
|--------|---------------|
| On-Demand | Walk in, pay full price |
| Reserved | Book for 1-3 years, get discount |
| Savings Plans | Commit to spend $X/month, flexible room type |
| Spot | Last-minute discount, can be kicked out |
| Dedicated Host | Book entire building |
| Capacity Reservation | Book room, pay whether you stay or not |

## Questions & Answers

> [!question]- Q1: What's the maximum discount for Reserved Instances?
> **Answer:**
> ==Up to 72%== with 3-year term, all upfront payment, standard Reserved Instance.

> [!question]- Q2: When would you use Spot Instances?
> **Answer:**
> Use Spot for workloads that:
> - Can tolerate interruptions
> - Have flexible start/end times
> - Are fault-tolerant
> 
> Examples: Batch processing, data analysis, CI/CD, image processing.
> 
> ==Never use for==: Databases, critical applications.

> [!question]- Q3: What's the difference between Savings Plans and Reserved Instances?
> **Answer:**
> - **Reserved Instances**: Commit to specific instance type, region, OS
> - **Savings Plans**: Commit to $/hour spend, flexible on instance size/OS
> 
> Savings Plans are more flexible and the modern approach.

> [!question]- Q4: How do you properly terminate persistent Spot Instances?
> **Answer:**
> 1. ==First== cancel the Spot Request
> 2. ==Then== terminate the instances
> 
> If you terminate first, the persistent request will automatically launch new instances.

> [!question]- Q5: When would you use Dedicated Hosts?
> **Answer:**
> - BYOL (Bring Your Own License) with per-socket/per-core licensing
> - Compliance requirements needing physical server isolation
> - Regulatory requirements
> 
> Most expensive option - only use when specifically required.
