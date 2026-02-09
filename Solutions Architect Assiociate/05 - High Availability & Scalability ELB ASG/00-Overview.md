---
title: High Availability & Scalability - Overview
date: 2026-02-09
tags:
  - aws
  - elb
  - asg
  - high-availability
  - scalability
  - saa-c03
  - index
---

# High Availability & Scalability ELB & ASG - Overview

## Study Files

| # | Topic | File | Priority |
|---|-------|------|----------|
| 1 | Scalability & High Availability Concepts | [[01-Scalability-High-Availability]] | 🔴 High |
| 2 | ELB Overview | [[02-ELB-Overview]] | 🔴 High |
| 3 | Application Load Balancer (ALB) | [[03-ALB]] | 🔴 High |
| 4 | Network Load Balancer (NLB) | [[04-NLB]] | 🔴 High |
| 5 | Gateway Load Balancer (GWLB) | [[05-GWLB]] | 🟡 Medium |
| 6 | ELB Advanced Features | [[06-ELB-Advanced-Features]] | 🔴 High |
| 7 | Auto Scaling Groups (ASG) | [[07-ASG]] | 🔴 High |

## Quick Decision Tree

```
Which Load Balancer?
├── HTTP/HTTPS/WebSocket (Layer 7)?
│   └── ALB (routing by path, host, query string)
├── TCP/UDP/Static IP (Layer 4)?
│   └── NLB (millions req/s, Elastic IP per AZ)
├── Network traffic inspection (Layer 3)?
│   └── GWLB (firewalls, IDS/IPS, GENEVE port 6081)
└── Legacy?
    └── CLB (deprecated)
```

## Related Sections

- [[../04 - EC2 Instance Storage/00-EC2-Instance-Storage-Overview|EC2 Instance Storage]]
- [[../03 - EC2 SAA Level/00-EC2-SAA-Level-Overview|EC2 SAA Level]]
