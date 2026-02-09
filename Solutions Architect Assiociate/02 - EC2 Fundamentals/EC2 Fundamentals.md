---
title: EC2 Fundamentals - Complete Guide
date: 2026-02-03
tags:
  - aws
  - ec2
  - compute
  - saa-c03
---

# EC2 Fundamentals - Complete Guide

## Overview

==EC2 (Elastic Compute Cloud)== is AWS's core compute service - Infrastructure as a Service (IaaS). It allows you to rent virtual machines on demand.

```
EC2 Capabilities
├── Rent Virtual Machines (EC2 Instances)
├── Store Data on Virtual Drives (EBS)
├── Distribute Load (ELB)
└── Scale Services (Auto Scaling Groups)
```

## EC2 Instance Configuration

When launching an EC2 instance, you choose:

| Component | Options |
|-----------|---------|
| OS | Linux, Windows, Mac OS |
| CPU | Number of vCPUs/cores |
| RAM | Amount of memory |
| Storage | Network-attached (EBS/EFS) or Hardware (Instance Store) |
| Network | Speed, Public IP |
| Firewall | Security Groups |
| Bootstrap | EC2 User Data script |

## EC2 User Data

> [!important] Bootstrap Script
> EC2 User Data runs ==only once== at instance first launch with ==root user== privileges.

Common uses:
- Install updates
- Install software
- Download files from internet

```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "Hello World" > /var/www/html/index.html
```

## EC2 Instance Types

### Naming Convention

```
m5.2xlarge
│ │  └── Size (small → large → xlarge → 2xlarge...)
│ └── Generation (improves over time)
└── Instance Class
```

### Instance Families

| Type | Use Case | Prefix |
|------|----------|--------|
| **General Purpose** | Web servers, code repos, balanced workloads | t2, t3, m5 |
| **Compute Optimized** | Batch processing, ML, gaming servers, HPC | c5, c6 |
| **Memory Optimized** | In-memory databases, real-time processing | r5, x1, z1 |
| **Storage Optimized** | OLTP, data warehousing, distributed file systems | i3, d2, h1 |

### Free Tier

> [!tip] t2.micro
> - 1 vCPU, 1 GB RAM
> - 750 hours/month free (first 12 months)
> - Use for learning and testing

## Security Groups

==Security Groups== are virtual firewalls controlling inbound/outbound traffic.

```
┌─────────────────────────────────────────────┐
│              Security Group                  │
│  ┌─────────────────────────────────────┐    │
│  │         EC2 Instance                │    │
│  └─────────────────────────────────────┘    │
│                                             │
│  Inbound Rules        Outbound Rules        │
│  ─────────────        ──────────────        │
│  Port 22 (SSH)        All traffic           │
│  Port 80 (HTTP)       (default: allow all)  │
│  Port 443 (HTTPS)                           │
└─────────────────────────────────────────────┘
```

### Key Characteristics

- Only contain ==allow rules== (no deny)
- Can reference IP addresses or other security groups
- Locked to ==region/VPC== combination
- Lives ==outside== EC2 (blocked traffic never reaches instance)
- One SG can attach to multiple instances
- One instance can have multiple SGs

> [!warning] Timeout = Security Group Issue
> If connection hangs/times out → Check security group rules
> If "connection refused" → Security group OK, application issue

### Classic Ports

| Port | Protocol | Use |
|------|----------|-----|
| 22 | SSH | Linux instance login |
| 21 | FTP | File transfer |
| 22 | SFTP | Secure file transfer (uses SSH) |
| 80 | HTTP | Unsecured web |
| 443 | HTTPS | Secured web |
| 3389 | RDP | Windows instance login |

### Security Group Referencing

```
┌──────────────┐         ┌──────────────┐
│ EC2 Instance │         │ EC2 Instance │
│    (SG-1)    │◀───────▶│    (SG-2)    │
└──────────────┘         └──────────────┘
        │
        │ SG-1 Inbound: Allow SG-2
        │ Result: SG-2 instances can connect
        ▼
   No IP management needed!
```

## SSH Access Methods

### Options by OS

| Your OS | Method |
|---------|--------|
| Mac/Linux | SSH command |
| Windows 10+ | SSH command or PuTTY |
| Windows 7/8 | PuTTY |
| Any | EC2 Instance Connect (browser) |

### SSH Command (Mac/Linux/Windows 10+)

```bash
# Navigate to key directory
cd ~/path-to-key

# Set permissions (first time only)
chmod 0400 EC2Tutorial.pem

# Connect
ssh -i EC2Tutorial.pem ec2-user@<PUBLIC_IP>
```

### EC2 Instance Connect

> [!tip] Easiest Method
> - Browser-based SSH
> - No key management needed
> - Works with Amazon Linux 2
> - Still requires port 22 open in security group

## IAM Roles for EC2

> [!danger] Never Use aws configure on EC2
> Never enter IAM credentials directly on EC2 instances - anyone with access can retrieve them.

### Correct Approach: IAM Roles

```
┌──────────────┐      ┌──────────────┐
│  IAM Role    │─────▶│ EC2 Instance │
│ (Permissions)│      │              │
└──────────────┘      └──────────────┘
                            │
                            ▼
                    AWS API Calls
                    (No credentials stored)
```

**Steps:**
1. Create IAM Role with required permissions
2. Attach role to EC2 instance (Actions → Security → Modify IAM Role)
3. Instance automatically gets temporary credentials

## EC2 Purchasing Options

### Comparison

| Option | Discount | Commitment | Use Case |
|--------|----------|------------|----------|
| **On-Demand** | 0% | None | Short, unpredictable workloads |
| **Reserved** | Up to 72% | 1-3 years | Steady-state (databases) |
| **Savings Plans** | Up to 72% | 1-3 years ($/hour) | Flexible long-term |
| **Spot** | Up to 90% | None (can lose) | Fault-tolerant batch jobs |
| **Dedicated Host** | Varies | Optional | Licensing, compliance |
| **Dedicated Instance** | Varies | None | Hardware isolation |
| **Capacity Reservation** | 0% | None | Guaranteed capacity |

### On-Demand

- Pay per second (Linux/Windows) or per hour (other OS)
- Highest cost, maximum flexibility
- No upfront payment

### Reserved Instances

- Reserve specific: instance type, region, tenancy, OS
- Payment options: No upfront, Partial, All upfront
- Can buy/sell in Reserved Instance Marketplace
- **Convertible Reserved**: Can change instance type (up to 66% discount)

### Savings Plans

- Commit to $/hour spend for 1-3 years
- Flexible across instance size, OS, tenancy
- Locked to instance family and region

### Spot Instances

```
┌─────────────────────────────────────────┐
│         Spot Price Over Time            │
│                                         │
│  $0.10 ─ ─ ─ ─ On-Demand Price ─ ─ ─   │
│                                         │
│  $0.04 ════════ Your Max Price ════════ │
│         ╱╲    ╱╲                        │
│  $0.03 ╱  ╲  ╱  ╲   Spot Price         │
│       ╱    ╲╱    ╲                      │
│                                         │
│  When Spot > Max: 2 min to stop/term    │
└─────────────────────────────────────────┘
```

- Define max price willing to pay
- If spot price > max price → 2 min grace period to stop/terminate
- ==Not for critical jobs or databases==
- Good for: batch jobs, data analysis, distributed workloads

### Spot Request Types

| Type | Behavior |
|------|----------|
| **One-time** | Request fulfilled once, then gone |
| **Persistent** | Automatically restarts instances when price allows |

> [!warning] Terminating Spot Instances
> 1. First cancel the spot request
> 2. Then terminate the instances
> (Otherwise persistent request will relaunch them)

### Spot Fleets

- Set of Spot + optional On-Demand instances
- Define multiple launch pools (instance types, OS, AZs)
- Strategies:
  - **lowestPrice**: Maximum savings (short workloads)
  - **diversified**: Spread across pools (availability)
  - **capacityOptimized**: Optimal capacity
  - **priceCapacityOptimized**: Best capacity + lowest price

### Dedicated Hosts

- Physical server dedicated to you
- Use for: BYOL (Bring Your Own License), compliance requirements
- Most expensive option

### Dedicated Instances

- Hardware dedicated to you
- May share with other instances in same account
- No control over instance placement

### Capacity Reservations

- Reserve capacity in specific AZ
- No time commitment, no discount
- Pay on-demand rate whether used or not
- Combine with Reserved Instances/Savings Plans for discounts

## IPv4 Charges

> [!warning] Since Feb 1, 2024
> $0.005/hour (~$3.60/month) for all Public IPv4 addresses

**Free Tier (EC2 only):**
- 750 hours/month of Public IPv4 (first 12 months)
- Other services (ELB, RDS) have no free tier for IPv4

## Questions & Answers

> [!question]- Q1: What is EC2 User Data and when does it run?
> **Answer:**
> EC2 User Data is a bootstrap script that runs ==only once== at instance first launch. It runs with root user privileges and is used to automate boot tasks like installing updates, software, or downloading files.

> [!question]- Q2: What does a timeout when connecting to EC2 indicate?
> **Answer:**
> A timeout ==always== indicates a security group issue. The traffic is being blocked before reaching the instance. Check that the required port (e.g., 22 for SSH, 80 for HTTP) is open in the inbound rules.

> [!question]- Q3: How should you provide AWS credentials to an EC2 instance?
> **Answer:**
> ==Never== use `aws configure` on EC2. Instead:
> 1. Create an IAM Role with required permissions
> 2. Attach the role to the EC2 instance
> 3. Instance automatically receives temporary credentials

> [!question]- Q4: What's the difference between Spot Instances and Spot Fleets?
> **Answer:**
> - **Spot Instance**: Single request for specific instance type/AZ
> - **Spot Fleet**: Collection of Spot + On-Demand instances across multiple launch pools (instance types, AZs). Fleet automatically chooses optimal instances based on strategy (lowestPrice, diversified, etc.)

> [!question]- Q5: When would you use Reserved Instances vs Savings Plans?
> **Answer:**
> - **Reserved Instances**: Know exact instance type needed for 1-3 years (e.g., specific database server)
> - **Savings Plans**: Want flexibility in instance size/OS while committing to spend amount. More modern approach.

> [!question]- Q6: What happens when Spot price exceeds your max price?
> **Answer:**
> You get a ==2-minute grace period== to either:
> - **Stop** the instance (resume later when price drops)
> - **Terminate** the instance (lose state)
> 
> Spot instances are ==not suitable== for critical jobs or databases.

> [!question]- Q7: How do you properly terminate persistent Spot instances?
> **Answer:**
> 1. First ==cancel the spot request==
> 2. Then ==terminate the instances==
> 
> If you terminate first, the persistent request will automatically launch new instances.

> [!question]- Q8: What are the classic ports to remember for the exam?
> **Answer:**
> | Port | Protocol |
> |------|----------|
> | 22 | SSH (Linux) |
> | 80 | HTTP |
> | 443 | HTTPS |
> | 3389 | RDP (Windows) |

> [!question]- Q9: Can security groups reference other security groups?
> **Answer:**
> Yes! Instead of specifying IP addresses, you can allow traffic from instances that have a specific security group attached. This is useful for load balancers and internal communication without managing IPs.

> [!question]- Q10: What's the difference between Dedicated Hosts and Dedicated Instances?
> **Answer:**
> - **Dedicated Host**: You get the ==physical server==, visibility into sockets/cores, control over placement. Used for BYOL licensing.
> - **Dedicated Instance**: Hardware dedicated to you but ==no visibility== into physical server. May share with other instances in your account.
