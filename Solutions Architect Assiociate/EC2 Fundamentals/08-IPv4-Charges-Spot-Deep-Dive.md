---
title: IPv4 Charges & Spot Deep Dive
date: 2026-02-03
tags:
  - aws
  - ec2
  - ipv4
  - spot
  - pricing
  - saa-c03
---

# IPv4 Charges & Spot Deep Dive

## Public IPv4 Charges

> [!warning] New Charges Since Feb 1, 2024
> AWS now charges for ==all Public IPv4 addresses== whether used or not.

### Pricing

| Metric | Cost |
|--------|------|
| Per Hour | $0.005 |
| Per Month | ~$3.60 |
| Per Year | ~$43.80 |

### Free Tier (EC2 Only)

| Service | Free Tier |
|---------|-----------|
| EC2 | 750 hours/month (first 12 months) |
| ELB | ❌ No free tier |
| RDS | ❌ No free tier |
| Other services | ❌ No free tier |

```
EC2 Free Tier Calculation:
750 hours = ~1 month of continuous running
├── 1 instance running 24/7 = 720 hours ✓
├── 2 instances running 24/7 = 1440 hours ✗ (over limit)
└── 3 instances running 8 hrs/day = 720 hours ✓
```

### Services That Create Public IPs

```
Public IPv4 Sources:
├── EC2 Instances (with public IP enabled)
├── Elastic Load Balancers (1 IP per AZ)
├── NAT Gateways
├── RDS Instances (if publicly accessible)
├── Elastic IPs (even if not attached!)
└── Other services with public endpoints
```

### Monitoring IPv4 Usage

**Method 1: Billing Dashboard**
```
Billing → Bills → Select Month → Charges by Service
└── Look for "Public IPv4" charges
```

**Method 2: IPAM (IP Address Manager)**
```
VPC Console → IPAM → Public IP Insights
└── Create IPAM (free tier eligible)
    └── View all public IPs across regions
```

### Cost Optimization Tips

1. **Delete unused Elastic IPs** - Charged even when not attached
2. **Use private subnets** - Instances don't need public IPs
3. **Use NAT Gateway** - Single public IP for multiple instances
4. **Consider IPv6** - No charges (but not all ISPs support it)
5. **Clean up after hands-on** - Delete resources promptly

## Spot Instances Deep Dive

### How Spot Pricing Works

```
┌─────────────────────────────────────────────────────────┐
│                  Spot Price Over Time                    │
│                                                         │
│  $0.10 ─ ─ ─ ─ ─ On-Demand Price ─ ─ ─ ─ ─ ─ ─ ─ ─    │
│                                                         │
│  $0.05 ════════ Your Max Price ════════════════════    │
│                                                         │
│  $0.04 ┌──┐    ┌────┐                                  │
│        │  │    │    │     Spot Price                   │
│  $0.03 │  └────┘    └──┐                               │
│        │               │                                │
│  $0.02 └───────────────┴────────────────────────       │
│                                                         │
│        ├─ Instance ─┤    ├─ Instance ─┤                │
│           Running          Running                      │
│                                                         │
│  When Spot Price > Max Price: 2-minute warning         │
└─────────────────────────────────────────────────────────┘
```

### Spot Price Characteristics

- Varies by instance type, AZ, and time
- Based on supply and demand
- Can check historical prices in console
- Different AZs have different prices

### Interruption Handling

When spot price exceeds your max price:

| Option | Behavior | Use Case |
|--------|----------|----------|
| **Stop** | Instance stops, EBS preserved | Resume work later |
| **Terminate** | Instance deleted | Stateless workloads |
| **Hibernate** | RAM saved to EBS, resume quickly | Quick restart needed |

**Grace Period:** ==2 minutes== to save work before interruption

### Spot Request Types

#### One-Time Request

```
Request Created
      │
      ▼
┌─────────────┐
│   Pending   │
└──────┬──────┘
       │ Capacity available
       ▼
┌─────────────┐
│   Active    │──────▶ Instance Launched
└──────┬──────┘
       │ Fulfilled
       ▼
┌─────────────┐
│   Closed    │  Request complete, won't restart
└─────────────┘
```

#### Persistent Request

```
Request Created
      │
      ▼
┌─────────────┐
│   Pending   │◀──────────────────────┐
└──────┬──────┘                       │
       │ Capacity available           │
       ▼                              │
┌─────────────┐                       │
│   Active    │──────▶ Instance       │
└──────┬──────┘        Launched       │
       │                    │         │
       │                    │ Interrupted
       │                    │         │
       │                    ▼         │
       │              Instance        │
       │              Stopped/        │
       │              Terminated      │
       │                    │         │
       └────────────────────┴─────────┘
              Auto-restart when price allows
```

### Terminating Spot Instances Correctly

> [!danger] Critical Order
> Wrong order = instances keep relaunching!

**Correct Order:**
1. ==Cancel the Spot Request first==
2. ==Then terminate the instances==

```
Wrong Way:
Terminate Instance → Request sees 0 instances → Launches new instance!

Right Way:
Cancel Request → Request disabled → Terminate Instance → Done!
```

### Spot Request States

| State | Description |
|-------|-------------|
| `open` | Request submitted, waiting for capacity |
| `active` | Request fulfilled, instances running |
| `disabled` | Request stopped (persistent only) |
| `cancelled` | Request cancelled by user |
| `closed` | Request completed (one-time) |
| `failed` | Request failed (bad parameters) |

**Can Cancel When:** `open`, `active`, or `disabled`

## Spot Fleets

### What is a Spot Fleet?

Collection of Spot Instances (+ optional On-Demand) that automatically maintains target capacity.

```
Spot Fleet
├── Launch Pool 1: c5.large, us-east-1a
├── Launch Pool 2: c5.large, us-east-1b
├── Launch Pool 3: m5.large, us-east-1a
├── Launch Pool 4: m5.large, us-east-1b
└── Target: 10 instances (or vCPUs, or memory)
```

### Allocation Strategies

| Strategy | How It Works | Best For |
|----------|--------------|----------|
| **lowestPrice** | Launches from cheapest pool | Maximum cost savings |
| **diversified** | Spreads across all pools | High availability |
| **capacityOptimized** | Launches from pool with most capacity | Reducing interruptions |
| **priceCapacityOptimized** | Best capacity first, then lowest price | ==Recommended for most== |

### Spot Fleet Configuration

```
Spot Fleet Request:
├── Launch Template / Specifications
│   ├── AMI
│   ├── Instance Types (multiple)
│   └── Subnets (multiple AZs)
├── Target Capacity
│   ├── Type: instances / vCPUs / memory
│   └── Amount: 10
├── Allocation Strategy: priceCapacityOptimized
├── On-Demand Base Capacity: 2 (optional)
└── Maintain Target Capacity: Yes
```

### Spot Fleet vs Spot Request

| Feature | Spot Request | Spot Fleet |
|---------|--------------|------------|
| Instance Types | Single | Multiple |
| AZs | Single | Multiple |
| Allocation Strategy | N/A | Yes |
| Auto-replace interrupted | Persistent only | Yes |
| Mix with On-Demand | No | Yes |

## Hands-On: Spot Request UI

### Viewing Spot Pricing History

```
EC2 Console → Spot Requests → Pricing History
├── Select Instance Type: c4.large
├── Select OS: Linux/Unix
├── Select Time Range: 3 months
└── View price trends by AZ
```

### Creating Spot Request

```
EC2 Console → Instances → Launch Instance
└── Advanced Details
    └── Request Spot Instances ✓
        ├── Max Price: (default = On-Demand price)
        ├── Request Type: One-time / Persistent
        ├── Valid Until: (for persistent)
        └── Interruption Behavior: Terminate / Stop / Hibernate
```

## Questions & Answers

> [!question]- Q1: How much does a Public IPv4 cost per month?
> **Answer:**
> ~==$3.60/month== ($0.005/hour × 24 hours × 30 days).
> 
> EC2 has 750 hours/month free tier (first 12 months). Other services like ELB and RDS have no free tier for IPv4.

> [!question]- Q2: What happens when Spot price exceeds your max price?
> **Answer:**
> You get a ==2-minute warning==, then your instance is either:
> - **Stopped** (EBS preserved, can restart later)
> - **Terminated** (instance deleted)
> - **Hibernated** (RAM saved to EBS)
> 
> You choose the behavior when creating the request.

> [!question]- Q3: What's the difference between Spot Request and Spot Fleet?
> **Answer:**
> - **Spot Request**: Single instance type, single AZ
> - **Spot Fleet**: Multiple instance types, multiple AZs, allocation strategies, can mix with On-Demand
> 
> Spot Fleet is more flexible and can automatically choose the best pools.

> [!question]- Q4: Which Spot Fleet allocation strategy is recommended?
> **Answer:**
> ==priceCapacityOptimized== - It first selects pools with highest capacity (reducing interruption risk), then chooses the lowest price among those. Best balance of cost and availability.

> [!question]- Q5: Why must you cancel Spot Request before terminating instances?
> **Answer:**
> For ==persistent== Spot Requests, the request monitors instance count. If you terminate instances first, the request sees "0 instances" and automatically launches new ones to meet target capacity.
> 
> Cancel request first → Request disabled → Then terminate instances safely.
