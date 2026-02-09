---
title: Classic Solutions Architecture Discussions - Overview
date: 2026-02-10
tags:
  - aws
  - solutions-architecture
  - saa-c03
  - index
---

# Classic Solutions Architecture Discussions - Overview

## Study Files

| # | Topic | File | Priority |
|---|-------|------|----------|
| 1 | WhatsTheTime.com (Stateless) | [[01-WhatsTheTime]] | 🔴 High |
| 2 | MyClothes.com (Stateful) | [[02-MyClothes]] | 🔴 High |
| 3 | MyWordPress.com (Storage) | [[03-MyWordPress]] | 🔴 High |
| 4 | Instantiating Applications Quickly | [[04-Instantiating-Applications-Quickly]] | 🔴 High |
| 5 | Beanstalk Overview | [[05-Beanstalk-Overview]] | 🟡 Medium |
| 6 | Beanstalk Hands On | [[06-Beanstalk-Hands-On]] | 🟡 Medium |

## Section Purpose

This section ties together all previously learned services into ==real-world solution architectures==. Each case study iteratively builds from simple to complex, demonstrating how services fit together.

## Architecture Progression

```
Simple (1 EC2)
├── Vertical Scaling
├── Horizontal Scaling
│   ├── Elastic IP → Route 53
│   ├── ELB + Health Checks
│   └── ASG (Auto Scaling)
├── Multi-AZ (High Availability)
├── Stateful Patterns
│   ├── ELB Stickiness
│   ├── User Cookies
│   └── Server Sessions (ElastiCache/DynamoDB)
├── Database Layer
│   ├── RDS + Read Replicas
│   ├── ElastiCache (Lazy Loading)
│   └── Aurora MySQL
├── Storage
│   ├── EBS (single instance)
│   └── EFS (multi-instance, multi-AZ)
└── Managed Services
    └── Elastic Beanstalk
```

## Key Services Covered

| Service | Role in Architecture |
|---------|---------------------|
| **EC2** | Compute layer |
| **ELB** | Load balancing + health checks |
| **ASG** | Auto scaling |
| **Route 53** | DNS (A record, Alias record) |
| **RDS** | Relational database (Multi-AZ, Read Replicas) |
| **Aurora** | Managed MySQL/PostgreSQL |
| **ElastiCache** | Session store + caching layer |
| **EBS** | Block storage (single AZ) |
| **EFS** | Shared file storage (multi-AZ) |
| **Elastic Beanstalk** | Managed deployment platform |

## Related Sections

- [[../05 - High Availability & Scalability ELB ASG/00-Overview|High Availability & Scalability]]
- [[../06 - RDS Aurora ElastiCache/00-RDS-Aurora-ElastiCache-Overview|RDS Aurora ElastiCache]]
- [[../07 - Route 53/00-Route53-Overview|Route 53]]
