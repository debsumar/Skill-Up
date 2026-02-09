---
title: Elastic Network Interfaces (ENI)
date: 2026-02-03
tags:
  - aws
  - ec2
  - eni
  - networking
  - failover
  - saa-c03
---

# Elastic Network Interfaces (ENI)

## What is an ENI?

An ==Elastic Network Interface (ENI)== is a logical component in a VPC that represents a ==virtual network card==.

> [!tip] Key Concept
> ENIs give EC2 instances network connectivity. They can be created independently and moved between instances.

```
┌─────────────────────────────────────────────────────┐
│                   EC2 Instance                       │
│                                                     │
│   ┌─────────────┐        ┌─────────────┐           │
│   │    eth0     │        │    eth1     │           │
│   │ Primary ENI │        │Secondary ENI│           │
│   │             │        │  (Optional) │           │
│   │ Private IP  │        │ Private IP  │           │
│   │ Public IP   │        │             │           │
│   │ Security Grp│        │ Security Grp│           │
│   │ MAC Address │        │ MAC Address │           │
│   └─────────────┘        └─────────────┘           │
│                                                     │
└─────────────────────────────────────────────────────┘
```

## ENI Attributes

Each ENI can have:

| Attribute | Description |
|-----------|-------------|
| **Primary Private IPv4** | Main private IP address |
| **Secondary Private IPv4** | One or more additional private IPs |
| **Elastic IP** | One per private IPv4 |
| **Public IPv4** | Auto-assigned or Elastic IP |
| **Security Groups** | One or more attached |
| **MAC Address** | Unique hardware address |
| **Source/Dest Check** | Enable/disable for NAT |

## ENI Characteristics

| Feature | Details |
|---------|---------|
| **AZ Bound** | ENI is ==bound to specific AZ== |
| **Movable** | Can detach and attach to different instances |
| **Independent** | Can create ENI without an instance |
| **Multiple per Instance** | Instance can have multiple ENIs |
| **Persistence** | Manually created ENIs persist after instance termination |

## Primary vs Secondary ENI

```
EC2 Instance
├── eth0 (Primary ENI)
│   ├── Created automatically with instance
│   ├── Cannot be detached
│   ├── Has public IP (if enabled)
│   └── Deleted when instance terminates
│
└── eth1, eth2... (Secondary ENIs)
    ├── Can be created independently
    ├── Can be attached/detached
    ├── Can be moved between instances
    └── Persist after instance termination (if created manually)
```

## ENI Failover Pattern

> [!tip] Use Case
> Move a private IP between instances for quick failover without changing application configuration.

### Before Failover

```
┌─────────────────┐        ┌─────────────────┐
│  Instance A     │        │  Instance B     │
│  (Primary)      │        │  (Standby)      │
│                 │        │                 │
│  eth0: 10.0.1.5 │        │  eth0: 10.0.1.6 │
│  eth1: 10.0.1.10│        │                 │
│       ▲         │        │                 │
│       │         │        │                 │
│  App accessed   │        │                 │
│  via 10.0.1.10  │        │                 │
└─────────────────┘        └─────────────────┘
```

### After Failover (Move ENI)

```
┌─────────────────┐        ┌─────────────────┐
│  Instance A     │        │  Instance B     │
│  (Failed)       │        │  (Now Primary)  │
│                 │        │                 │
│  eth0: 10.0.1.5 │        │  eth0: 10.0.1.6 │
│                 │        │  eth1: 10.0.1.10│
│                 │        │       ▲         │
│                 │        │       │         │
│                 │        │  App accessed   │
│                 │        │  via 10.0.1.10  │
└─────────────────┘        └─────────────────┘

Same IP (10.0.1.10) now points to Instance B!
```

## Hands-On: Creating and Moving ENI

### Step 1: Create ENI

```
EC2 Console → Network & Security → Network Interfaces
└── Create network interface
    ├── Description: demo-eni
    ├── Subnet: Select AZ (must match target instances)
    ├── Private IPv4: Auto-assign or specify
    └── Security groups: Select
```

### Step 2: Attach to Instance

```
Select ENI → Actions → Attach
└── Choose instance (must be in same AZ)
```

### Step 3: Move ENI (Failover)

```
Select ENI → Actions → Detach → Force detach
Select ENI → Actions → Attach → Choose new instance
```

### Step 4: Verify

```
Instance → Networking tab → Network interfaces
└── Should show multiple ENIs with different IPs
```

## ENI Lifecycle

| ENI Type | Created | Deleted |
|----------|---------|---------|
| **Auto-created** (with instance) | When instance launches | When instance terminates |
| **Manually created** | By user | Only when user deletes |

> [!warning] Important
> Manually created ENIs ==persist== after instance termination. Remember to delete them to avoid confusion.

## Use Cases

1. **Network Failover**
   - Move ENI between instances for quick IP failover
   - No DNS changes needed

2. **Multiple Network Interfaces**
   - Management network + production network
   - Different security groups per interface

3. **Dual-homed Instances**
   - Instance in multiple subnets
   - Different routing per interface

4. **License-bound Software**
   - Software licensed to MAC address
   - Move ENI to preserve license

## AWS CLI Commands

```bash
# Create ENI
aws ec2 create-network-interface \
    --subnet-id subnet-12345 \
    --description "My ENI" \
    --groups sg-12345

# Attach ENI
aws ec2 attach-network-interface \
    --network-interface-id eni-12345 \
    --instance-id i-12345 \
    --device-index 1

# Detach ENI
aws ec2 detach-network-interface \
    --attachment-id eni-attach-12345 \
    --force

# Delete ENI
aws ec2 delete-network-interface \
    --network-interface-id eni-12345
```

## Questions & Answers

> [!question]- Q1: What is an ENI and what does it represent?
> **Answer:**
> An ENI (Elastic Network Interface) is a ==virtual network card== in a VPC. It provides network connectivity to EC2 instances and includes attributes like private IP, public IP, security groups, and MAC address.

> [!question]- Q2: Can you move an ENI between instances in different Availability Zones?
> **Answer:**
> ==No==. ENIs are ==bound to a specific AZ==. You can only attach an ENI to instances in the same Availability Zone where the ENI was created.

> [!question]- Q3: What happens to ENIs when you terminate an EC2 instance?
> **Answer:**
> - **Auto-created ENIs** (eth0): ==Deleted== with the instance
> - **Manually created ENIs**: ==Persist== and remain available for attachment to other instances

> [!question]- Q4: How can ENIs be used for failover?
> **Answer:**
> Create a secondary ENI with a known private IP. Attach it to your primary instance. If the primary fails, ==detach the ENI and attach it to a standby instance==. The IP address moves with the ENI, so applications can continue using the same IP.

> [!question]- Q5: Can an EC2 instance have multiple ENIs?
> **Answer:**
> ==Yes==. An instance can have multiple ENIs attached (eth0, eth1, eth2...). The number depends on instance type. Each ENI can have its own private IP, security groups, and can be in different subnets (within same AZ).

> [!question]- Q6: What is the difference between a primary ENI (eth0) and a secondary ENI?
> **Answer:**
> The ==primary ENI (eth0)== is created automatically with the instance, cannot be detached, and is deleted when the instance terminates. ==Secondary ENIs== (eth1, eth2...) can be created independently, attached/detached freely, and persist after instance termination.

> [!question]- Q7: What attributes can an ENI have?
> **Answer:**
> - Primary private IPv4 and one or more secondary private IPv4s
> - One ==Elastic IP per private IPv4==
> - One or more ==security groups==
> - A ==MAC address==
> - Source/destination check flag

> [!question]- Q8: Does a manually created ENI cost money if left unattached?
> **Answer:**
> ==No==. An unattached ENI itself does not incur charges. However, any ==Elastic IP== associated with an unattached ENI will incur charges. The ENI is just a logical networking component.

> [!question]- Q9: Why would you use a secondary ENI for license-bound software?
> **Answer:**
> Some software is licensed to a specific ==MAC address==. Since each ENI has its own MAC address, you can move the ENI (and its MAC) to a new instance if the original fails, ==preserving the software license== without re-activation.

> [!question]- Q10: Can you attach security groups directly to an ENI?
> **Answer:**
> ==Yes==. Security groups are attached at the ==ENI level==, not the instance level. Each ENI can have different security groups, allowing you to have different network access rules per interface on the same instance.
