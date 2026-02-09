---
title: Route 53 Health Checks
date: 2026-02-10
tags:
  - aws
  - route53
  - health-checks
  - monitoring
  - saa-c03
---

# Route 53 Health Checks

## Three Types of Health Checks

| Type | What It Monitors |
|------|------------------|
| **Endpoint** | Public resource (app, server, AWS resource) |
| **Calculated** | Combines ==multiple child health checks== into one parent |
| **CloudWatch Alarm** | Monitors a ==CloudWatch Alarm== (for private resources) |

## Endpoint Health Checks

```
┌─────────────────────────────────────────┐
│  ~15 Global Health Checkers              │
│  (from all around the world)             │
│       │       │       │       │          │
│       ▼       ▼       ▼       ▼          │
│  ┌─────────────────────────────┐  │
│  │  Your Public Endpoint (ALB)  │  │
│  │  Returns 2xx/3xx = Healthy   │  │
│  └─────────────────────────────┘  │
└─────────────────────────────────────────┘
```

| Setting | Detail |
|---------|--------|
| **Checkers** | ~==15 global== health checkers |
| **Healthy threshold** | > ==18%== of checkers report healthy |
| **Interval** | ==30 seconds== (regular) or ==10 seconds== (fast, higher cost) |
| **Protocols** | HTTP, HTTPS, TCP |
| **Status codes** | ==2xx or 3xx== = healthy |
| **Text check** | Can check first ==5,120 bytes== of response for specific text |
| **Network** | Must ==allow incoming requests== from Route 53 health checker IPs |

> [!warning] Security Groups
> You must allow incoming requests from ==Route 53 health checker IP ranges== in your security groups/firewall.

## Calculated Health Checks

```
┌─────────────────────────────────┐
│  Parent Health Check              │
│  (OR / AND / NOT)                 │
└───────┬────────┬────────┬───────┘
        │        │        │
   ┌────▼──┐ ┌──▼───┐ ┌──▼───┐
   │Child 1│ │Child 2│ │Child 3│
   └───────┘ └──────┘ └──────┘
      │         │         │
   ┌──▼──┐  ┌──▼──┐  ┌──▼──┐
   │ EC2  │  │ EC2  │  │ EC2  │
   └─────┘  └─────┘  └─────┘
```

| Feature | Detail |
|---------|--------|
| **Conditions** | ==OR, AND, NOT== |
| **Max children** | Up to ==256== child health checks |
| **Threshold** | Specify how many children must pass |
| **Use case** | Perform maintenance without failing all health checks |

## Private Resource Health Checks

Route 53 health checkers are ==outside your VPC== — they can't access private endpoints.

```
┌─────────────────────────────────────────────┐
│  Private VPC                                    │
│  ┌──────────┐  metric  ┌───────────────┐       │
│  │ EC2      │────────▶│  CloudWatch   │       │
│  │ (private)│         │  Metric       │       │
│  └──────────┘         └──────┬────────┘       │
│                              │ alarm            │
└───────────────────────────┴─────────────────┘
                               │
                        ┌──────▼─────────┐
                        │  CloudWatch     │
                        │  Alarm          │
                        └──────┬─────────┘
                               │
                        ┌──────▼─────────┐
                        │  Route 53       │
                        │  Health Check   │
                        └────────────────┘
```

> [!tip] Private Resource Monitoring
> Create a ==CloudWatch Metric== → ==CloudWatch Alarm== → ==Route 53 Health Check==. When the alarm fires, the health check becomes unhealthy.

## Questions & Answers

> [!question]- Q1: How many global health checkers does Route 53 use?
> **Answer:**
> About ==15 global health checkers== from all around the world.

> [!question]- Q2: What percentage of checkers must report healthy?
> **Answer:**
> Over ==18%== of health checkers must report the endpoint as healthy for Route 53 to consider it healthy.

> [!question]- Q3: What status codes indicate a healthy endpoint?
> **Answer:**
> ==2xx or 3xx== status codes.

> [!question]- Q4: What is a Calculated Health Check?
> **Answer:**
> A ==parent health check== that combines multiple ==child health checks== using OR, AND, or NOT conditions. Supports up to ==256 children==.

> [!question]- Q5: How do you health-check a private resource?
> **Answer:**
> Create a ==CloudWatch Metric== on the private resource → set a ==CloudWatch Alarm== → associate the alarm with a ==Route 53 Health Check==.

> [!question]- Q6: Why can't Route 53 directly health-check private resources?
> **Answer:**
> Because Route 53 health checkers live on the ==public internet==, outside your VPC. They cannot access private endpoints.

> [!question]- Q7: What is the health check interval?
> **Answer:**
> ==30 seconds== (regular) or ==10 seconds== (fast, higher cost).

> [!question]- Q8: Can health checks inspect response body text?
> **Answer:**
> ==Yes==. For text-based responses, health checkers can check the first ==5,120 bytes== for specific text.

> [!question]- Q9: What network requirement exists for health checks?
> **Answer:**
> You must ==allow incoming requests== from Route 53 health checker IP address ranges in your security groups.

> [!question]- Q10: What protocols do health checks support?
> **Answer:**
> ==HTTP, HTTPS, and TCP==.