---
title: EC2 SAA Level - Overview
date: 2026-02-03
tags:
  - aws
  - ec2
  - saa-c03
  - index
---

# EC2 Solutions Architect Associate Level - Overview

## Course Section Summary

This section covers advanced EC2 topics for the Solutions Architect Associate exam.

## Study Files

| # | Topic | File | Priority |
|---|-------|------|----------|
| 1 | Private vs Public vs Elastic IP | [[01-Private-Public-Elastic-IP]] | 🔴 High |
| 2 | EC2 Placement Groups | [[02-EC2-Placement-Groups]] | 🔴 High |
| 3 | Elastic Network Interfaces (ENI) | [[03-Elastic-Network-Interfaces]] | 🔴 High |
| 4 | EC2 Hibernate | [[04-EC2-Hibernate]] | 🟡 Medium |

## Key Exam Topics

### 1. IP Addressing

- **Public IP**: Globally unique, changes on stop/start
- **Private IP**: VPC-internal, persists on stop/start
- **Elastic IP**: Static public IP, $0.005/hour, limit 5/account
- Best practice: Use DNS/Load Balancers instead of Elastic IPs

### 2. Placement Groups

| Strategy | Use Case | Limit |
|----------|----------|-------|
| **Cluster** | HPC, low latency | Single AZ, high risk |
| **Spread** | Critical apps, HA | 7 instances/AZ |
| **Partition** | Hadoop, Kafka | 7 partitions/AZ, 100s of instances |

### 3. Elastic Network Interfaces

- Virtual network card (private IP, security groups, MAC)
- Bound to specific AZ
- Can move between instances for failover
- Manually created ENIs persist after instance termination

### 4. EC2 Hibernate

- RAM saved to encrypted EBS root volume
- Fast boot (OS not restarted)
- Requirements: RAM < 150GB, encrypted EBS root
- Max 60 days hibernation

## Quick Reference

### Placement Group Decision Tree

```
Need lowest latency?
├── Yes → Cluster (same rack, 10Gbps)
└── No
    ├── Need max availability (small scale)?
    │   └── Spread (7 per AZ, different hardware)
    └── Need scale + isolation?
        └── Partition (Hadoop, Kafka)
```

### IP Behavior on Stop/Start

| IP Type | Changes? |
|---------|----------|
| Public IPv4 | ==Yes== (may change) |
| Private IPv4 | No |
| Elastic IP | No (static) |

### ENI Failover Pattern

```
1. Create secondary ENI with known IP
2. Attach to primary instance
3. On failure: Detach → Attach to standby
4. Same IP, different instance
```

## Hands-On Checklist

- [ ] Observe Public IP change on stop/start
- [ ] Allocate and associate Elastic IP
- [ ] Create all 3 placement group types
- [ ] Create and move ENI between instances
- [ ] Enable and test EC2 Hibernate (uptime command)

## Related Sections

- [[../EC2 Fundamentals/00-EC2-Overview|EC2 Fundamentals]] - Basic EC2 concepts
- [[../IAM & AWS CLI/IAM Fundamentals|IAM Fundamentals]] - IAM roles for EC2
- EBS & EFS (next) - Storage for EC2
