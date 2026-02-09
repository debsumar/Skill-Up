---
title: Advanced Routing Policies
date: 2026-02-10
tags:
  - aws
  - route53
  - failover
  - geolocation
  - geoproximity
  - ip-based
  - multi-value
  - saa-c03
---

# Advanced Routing Policies

## Failover

```
┌────────┐         ┌──────────┐
│ Client │────────▶│ Route 53 │
└────────┘         └────┬─────┘
                        │
              ┌────────┴────────┐
              │                 │
       ┌──────▼─────┐  ┌─────▼──────┐
       │  Primary   │  │  Secondary  │
       │  (healthy) │  │  (DR)       │
       └─────┬──────┘  └────────────┘
             │
       ┌─────▼──────┐
       │ Health     │
       │ Check      │  (mandatory on primary)
       └────────────┘
```

| Feature | Detail |
|---------|--------|
| **Records** | ==One primary + one secondary== |
| **Health check** | ==Mandatory== on primary |
| **Failover** | Automatic when primary health check fails |
| **Secondary HC** | Optional |

## Geolocation

| Feature | Detail |
|---------|--------|
| **Based on** | ==User's physical location== (continent, country, US state) |
| **Default record** | ==Must create one== for unmatched locations |
| **Use cases** | Website localization, content restriction, load balancing |
| **Health checks** | Supported |
| **Precision** | Most specific match wins (state > country > continent > default) |

> [!warning] Geolocation ≠ Latency
> Geolocation routes by ==where the user is==. Latency routes by ==lowest network latency==. A user in France always goes to the France record, even if US has lower latency.

```
Geolocation Rules:
├── Asia        ─▶ ap-southeast-1
├── USA         ─▶ us-east-1
└── Default     ─▶ eu-central-1 (everyone else)
```

## Geoproximity

| Feature | Detail |
|---------|--------|
| **Based on** | Geographic location of users ==AND resources== |
| **Bias** | Number to ==shift traffic== between regions |
| **Positive bias** | ==Expand== — attract more traffic |
| **Negative bias** | ==Shrink== — attract less traffic |
| **AWS resources** | Specify ==region== |
| **Non-AWS** | Specify ==latitude/longitude== |
| **Requires** | ==Route 53 Traffic Flow== |

```
Bias = 0 (equal):              Bias = +50 on us-east-1:
┌────────────────────┐       ┌────────────────────┐
│ west  │  │  east  │       │west│     east     │
│  50%  │  │  50%   │       │ 30%│     70%      │
└───────┴──┴───────┘       └────┴───────────────┘
        ▲ dividing line            ▲ shifted left
```

> [!tip] Exam Hint
> If you see "==shift traffic from one region to another==" → think ==Geoproximity with bias==.

## IP-Based

| Feature | Detail |
|---------|--------|
| **Based on** | ==Client IP address== (CIDR blocks) |
| **Define** | List of CIDRs → locations → records |
| **Use cases** | Optimize performance, reduce network costs |
| **Example** | ISP with known CIDR → route to specific endpoint |

## Multi-Value

| Feature | Detail |
|---------|--------|
| **Returns** | Up to ==8 healthy records== |
| **Health checks** | ==Supported== (only healthy records returned) |
| **Purpose** | ==Client-side load balancing== |
| **vs Simple** | Simple returns all values (==no health check filtering==) |
| **vs ELB** | ==Not a substitute== for ELB |

> [!important] Multi-Value vs Simple
> - Simple: returns all values, ==no health checks==, client may get unhealthy resource
> - Multi-Value: returns only ==healthy values==, up to 8, with health check filtering

## Questions & Answers

> [!question]- Q1: Is a health check mandatory for Failover routing?
> **Answer:**
> ==Yes== on the primary record. Optional on the secondary.

> [!question]- Q2: How many primary/secondary records can Failover have?
> **Answer:**
> ==One primary and one secondary== only.

> [!question]- Q3: What is the difference between Geolocation and Latency?
> **Answer:**
> - **Geolocation**: Routes by ==user's physical location== (country/continent)
> - **Latency**: Routes by ==lowest network latency== to AWS region

> [!question]- Q4: Do you need a default record for Geolocation?
> **Answer:**
> ==Yes==. You should create a default record for users whose location doesn't match any defined rule.

> [!question]- Q5: What is the bias in Geoproximity?
> **Answer:**
> A number that ==shifts traffic== between regions. Positive bias attracts more traffic; negative bias attracts less.

> [!question]- Q6: What does Geoproximity require that other policies don't?
> **Answer:**
> ==Route 53 Traffic Flow== to leverage the bias feature.

> [!question]- Q7: How does IP-based routing work?
> **Answer:**
> You define ==CIDR blocks== (IP ranges) and map them to specific locations/records. Useful when you know client IP ranges (e.g., specific ISPs).

> [!question]- Q8: How many records does Multi-Value return?
> **Answer:**
> Up to ==8 healthy records==. Only records with passing health checks are returned.

> [!question]- Q9: Is Multi-Value a replacement for ELB?
> **Answer:**
> ==No==. Multi-Value provides ==client-side load balancing== via DNS. It's not a substitute for a proper load balancer.

> [!question]- Q10: What is the key advantage of Multi-Value over Simple?
> **Answer:**
> Multi-Value supports ==health checks==, so only healthy resources are returned. Simple returns all values regardless of health.