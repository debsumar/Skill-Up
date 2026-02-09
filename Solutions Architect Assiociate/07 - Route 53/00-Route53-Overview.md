---
title: Route 53 - Overview
date: 2026-02-10
tags:
  - aws
  - route53
  - dns
  - saa-c03
  - index
---

# Route 53 - Overview

## Study Files

| # | Topic | File | Priority |
|---|-------|------|----------|
| 1 | What is DNS? | [[01-What-is-DNS]] | 🟡 Medium |
| 2 | Route 53 Overview | [[02-Route53-Overview]] | 🔴 High |
| 3 | Records & TTL | [[03-Records-and-TTL]] | 🔴 High |
| 4 | CNAME vs Alias | [[04-CNAME-vs-Alias]] | 🔴 High |
| 5 | Routing Policies | [[05-Routing-Policies]] | 🔴 High |
| 6 | Health Checks | [[06-Health-Checks]] | 🔴 High |
| 7 | Advanced Routing Policies | [[07-Routing-Policies-Advanced]] | 🔴 High |
| 8 | 3rd Party Domains | [[08-Third-Party-Domains]] | 🟡 Medium |

## Quick Decision Tree

```
Which Routing Policy?
├── Single resource, no health check?
│   └── Simple
├── Distribute traffic by percentage?
│   └── Weighted
├── Lowest latency for users?
│   └── Latency
├── Active-passive failover?
│   └── Failover (requires health check)
├── Route by user's country/continent?
│   └── Geolocation
├── Shift traffic between regions using bias?
│   └── Geoproximity (requires Traffic Flow)
├── Route by client IP (CIDR)?
│   └── IP-based
└── Multiple healthy resources returned?
    └── Multi-Value (client-side LB)
```

## Key Facts

| Fact | Detail |
|------|--------|
| **Port** | 53 (traditional DNS port) |
| **SLA** | ==100% availability== (only AWS service) |
| **Cost** | $0.50/month per hosted zone + $12+/year domain |
| **Alias queries** | ==Free== |

## Related Sections

- [[../06 - RDS Aurora ElastiCache/00-RDS-Aurora-ElastiCache-Overview|RDS Aurora ElastiCache]]
- [[../05 - High Availability & Scalability ELB ASG/00-Overview|High Availability & Scalability]]