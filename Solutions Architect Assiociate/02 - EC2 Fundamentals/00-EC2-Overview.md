---
title: EC2 Fundamentals - Overview
date: 2026-02-03
tags:
  - aws
  - ec2
  - saa-c03
  - index
---

# EC2 Fundamentals - Overview

## Course Section Summary

This section covers Amazon EC2 (Elastic Compute Cloud), AWS's core compute service.

## Study Files

| # | Topic | File | Priority |
|---|-------|------|----------|
| 1 | AWS Budget Setup | [[01-AWS-Budget-Setup]] | 🟡 Setup |
| 2 | EC2 Basics & Instance Launch | [[02-EC2-Basics-Instance-Launch]] | 🔴 High |
| 3 | EC2 Instance Types | [[03-EC2-Instance-Types]] | 🔴 High |
| 4 | Security Groups | [[04-Security-Groups]] | 🔴 High |
| 5 | SSH Access Methods | [[05-SSH-Access-Methods]] | 🟡 Medium |
| 6 | IAM Roles for EC2 | [[06-IAM-Roles-for-EC2]] | 🔴 High |
| 7 | EC2 Purchasing Options | [[07-EC2-Purchasing-Options]] | 🔴 High |
| 8 | IPv4 Charges & Spot Deep Dive | [[08-IPv4-Charges-Spot-Deep-Dive]] | 🔴 High |

## Key Exam Topics

### Must Know for SAA-C03

1. **EC2 Instance Types**
   - Naming convention (m5.2xlarge)
   - General Purpose vs Compute vs Memory vs Storage Optimized
   - t2.micro free tier

2. **Security Groups**
   - Virtual firewall (allow rules only)
   - Inbound/Outbound rules
   - Timeout = Security Group issue
   - Classic ports: 22 (SSH), 80 (HTTP), 443 (HTTPS), 3389 (RDP)

3. **IAM Roles for EC2**
   - Never use `aws configure` on EC2
   - Attach IAM Roles instead
   - Temporary credentials via Instance Metadata

4. **Purchasing Options**
   - On-Demand: No commitment, highest cost
   - Reserved: 1-3 years, up to 72% off
   - Savings Plans: Commit $/hour, flexible
   - Spot: Up to 90% off, can be interrupted
   - Dedicated Hosts: BYOL, compliance

5. **Spot Instances**
   - 2-minute interruption warning
   - One-time vs Persistent requests
   - Cancel request BEFORE terminating instances
   - Spot Fleets and allocation strategies

## Quick Reference

### Instance Type Prefixes

| Prefix | Type | Use Case |
|--------|------|----------|
| t, m | General Purpose | Web servers, dev |
| c | Compute Optimized | Batch, ML, gaming |
| r, x, z | Memory Optimized | Databases, caching |
| i, d, h | Storage Optimized | OLTP, warehousing |

### Purchasing Options Comparison

| Option | Discount | Commitment | Interruption |
|--------|----------|------------|--------------|
| On-Demand | 0% | None | No |
| Reserved | 72% | 1-3 years | No |
| Savings Plans | 72% | 1-3 years | No |
| Spot | 90% | None | ==Yes== |

### Classic Ports

```
SSH (Linux):    22
RDP (Windows):  3389
HTTP:           80
HTTPS:          443
```

## Hands-On Checklist

- [ ] Set up AWS Budget (zero-spend + monthly)
- [ ] Launch EC2 instance with User Data
- [ ] Configure Security Group rules
- [ ] SSH into instance (any method)
- [ ] Attach IAM Role to EC2
- [ ] View Spot pricing history
- [ ] Understand instance lifecycle (stop/start/terminate)

## Related Sections

- [[../01 - IAM & AWS CLI/IAM Fundamentals|IAM Fundamentals]] - IAM basics before EC2
- EBS & EFS (next section) - Storage for EC2
- ELB & ASG (upcoming) - Load balancing and scaling
