---
title: "Route 53 - Advanced Routing Policies"
date: 2026-02-10
tags:
  - aws
  - route53
  - dns
  - geolocation
  - geoproximity
  - ip-based
  - multi-value
  - saa-c03
---

# Route 53 - Advanced Routing Policies

## Overview

Beyond Simple, Weighted, Latency, and Failover, Route 53 offers four more routing policies for specialized use cases. These are ==frequently tested on the exam==, especially Geolocation and Geoproximity — understanding the difference between them is critical.

## Geolocation Routing Policy

Route traffic based on ==where the user is physically located== — by continent, country, or US state.

```
┌──────────────────────────────────────────────────────────────┐
│                    GEOLOCATION ROUTING                        │
│                                                               │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │                    Route 53                              │ │
│  │                                                         │ │
│  │  Rule 1: Asia        → ap-southeast-1 (Singapore)      │ │
│  │  Rule 2: US          → us-east-1 (N. Virginia)         │ │
│  │  Rule 3: Default     → eu-central-1 (Frankfurt)        │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                               │
│  User in India    → Asia rule matches    → Singapore         │
│  User in New York → US rule matches      → N. Virginia       │
│  User in Mexico   → No match → Default  → Frankfurt         │
│  User in France   → No match → Default  → Frankfurt         │
└──────────────────────────────────────────────────────────────┘
```

### Key Characteristics

| Feature | Detail |
|---------|--------|
| **Location types** | ==Continent==, ==Country==, or ==US State== |
| **Precision** | Most precise match wins (US State > Country > Continent > Default) |
| **Default record** | ==Should always create one== — catches users with no matching rule |
| **Health checks** | Can be associated |
| **Use cases** | Website localization, content restriction, regulatory compliance |

### Geolocation vs Latency

| Geolocation | Latency |
|-------------|---------|
| Based on ==physical location== | Based on ==network latency== |
| You define the rules (country → IP) | Route 53 measures latency automatically |
| User in Germany → always goes to "Europe" rule | User in Germany → goes to lowest-latency region |
| ==Deterministic== — same location always gets same answer | ==Dynamic== — can change based on network conditions |
| Use for: localization, compliance, content restriction | Use for: performance optimization |

> [!warning] Always Create a Default Record
> If a user's location doesn't match any of your geolocation rules, they get ==no answer== (NXDOMAIN). Always create a ==Default== geolocation record as a catch-all. In the hands-on, Mexico users got the default (EU) because only Asia and US were explicitly defined.

### Hands-On

1. Create `geo.yourdomain.com`:
   - Asia → Singapore IP (continent-level rule)
   - United States → US East IP (country-level rule)
   - Default → EU Central IP (catch-all)
2. Test from Europe → gets EU Central (default) ✅
3. VPN to India → gets Singapore (Asia rule) ✅
4. VPN to US → gets US East (US rule) ✅
5. VPN to Mexico → gets EU Central (default — Mexico not in any rule) ✅

## Geoproximity Routing Policy

Route traffic based on ==geographic proximity== to your resources, with the ability to ==shift traffic using a bias value==.

```
┌──────────────────────────────────────────────────────────────┐
│              GEOPROXIMITY WITH BIAS                           │
│                                                               │
│  Bias = 0 (both regions):                                    │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │  us-west-1        │        us-east-1                    │ │
│  │  (bias: 0)        │        (bias: 0)                    │ │
│  │                   │                                     │ │
│  │  ← Users go west  │  Users go east →                   │ │
│  │                   │                                     │ │
│  │         Dividing line is in the MIDDLE                  │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                               │
│  Bias = 0 (west), Bias = +50 (east):                        │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │  us-west-1   │              us-east-1                   │ │
│  │  (bias: 0)   │              (bias: +50)                 │ │
│  │              │                                          │ │
│  │  ← Fewer    │  More users go east →                    │ │
│  │    users     │                                          │ │
│  │         Dividing line SHIFTED LEFT                      │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                               │
│  Positive bias = EXPAND the region (attract more traffic)    │
│  Negative bias = SHRINK the region (repel traffic)           │
└──────────────────────────────────────────────────────────────┘
```

### How Bias Works

| Bias Value | Effect |
|-----------|--------|
| ==Positive (1 to 99)== | ==Expands== the geographic area — attracts more traffic to this resource |
| ==Zero (0)== | No bias — traffic split based on pure geographic proximity |
| ==Negative (-1 to -99)== | ==Shrinks== the geographic area — repels traffic away from this resource |

### Key Characteristics

| Feature | Detail |
|---------|--------|
| **Resource types** | AWS resources (specify region) or non-AWS (specify lat/long) |
| **Bias** | Number from -99 to 99 that shifts the geographic boundary |
| **Requires** | ==Route 53 Traffic Flow== (advanced feature) to use bias |
| **Use case** | ==Shift traffic from one region to another== |

> [!tip] Exam Tip
> If the exam mentions "shift traffic from one region to another" or "attract more traffic to a specific region," the answer is ==Geoproximity routing with bias==. This is the only routing policy that lets you gradually move traffic between regions by adjusting a number.

### Geolocation vs Geoproximity

| Geolocation | Geoproximity |
|-------------|-------------|
| Rules based on ==user's country/continent== | Based on ==geographic distance to resource== |
| ==Discrete== — user is in country X or not | ==Continuous== — distance-based with adjustable bias |
| No bias concept | ==Bias shifts the boundary== between regions |
| Use for: localization, compliance | Use for: ==traffic shifting between regions== |

## IP-Based Routing Policy

Route traffic based on the ==client's IP address== (CIDR blocks).

```
┌──────────────────────────────────────────────────────────────┐
│                    IP-BASED ROUTING                           │
│                                                               │
│  Route 53 CIDR Collections:                                  │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │  Location 1: 203.0.113.0/24  → EC2 at 1.2.3.4         │ │
│  │  Location 2: 200.5.4.0/24   → EC2 at 5.6.7.8         │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                               │
│  User A (IP: 203.0.113.50) → matches Location 1 → 1.2.3.4 │
│  User B (IP: 200.5.4.100)  → matches Location 2 → 5.6.7.8 │
└──────────────────────────────────────────────────────────────┘
```

### Key Characteristics

| Feature | Detail |
|---------|--------|
| **CIDR collections** | You define IP ranges and map them to locations |
| **Use cases** | Known ISP IP ranges, corporate networks, optimize costs |
| **Precision** | Most precise CIDR match wins |
| **When to use** | When you ==know the IP ranges== of your users ahead of time |

> [!note] When to Use IP-Based
> IP-based routing is useful when you know exactly which IP ranges your users come from — for example, a specific ISP, a corporate network, or a content delivery partner. It's less common than other routing policies but appears on the exam.

## Multi-Value Routing Policy

Return ==multiple healthy values== in a single DNS response — like Simple routing but ==with health checks==.

```
┌──────────────────────────────────────────────────────────────┐
│                    MULTI-VALUE ROUTING                        │
│                                                               │
│  3 records, each with a health check:                        │
│  ┌──────────────────────────────────────────────────────┐    │
│  │  Record 1: us-east-1    → Health Check: ✅ Healthy   │    │
│  │  Record 2: eu-central-1 → Health Check: ✅ Healthy   │    │
│  │  Record 3: ap-southeast → Health Check: ❌ Unhealthy │    │
│  └──────────────────────────────────────────────────────┘    │
│                                                               │
│  DNS Response returns ONLY healthy records:                  │
│  ┌──────────────────────────────────────────────────────┐    │
│  │  Answer 1: us-east-1 IP                              │    │
│  │  Answer 2: eu-central-1 IP                           │    │
│  │  (ap-southeast excluded — unhealthy)                 │    │
│  └──────────────────────────────────────────────────────┘    │
│                                                               │
│  Client picks one randomly from the healthy set              │
└──────────────────────────────────────────────────────────────┘
```

### Key Characteristics

| Feature | Detail |
|---------|--------|
| **Health checks** | ==Yes== — only healthy records are returned |
| **Max records returned** | Up to ==8 healthy records== per query |
| **Client behavior** | Client picks one randomly from the returned set |
| **Not a load balancer** | ==Client-side== load balancing, not a substitute for ELB |

### Multi-Value vs Simple (with multiple values)

| Multi-Value | Simple (multiple values) |
|-------------|------------------------|
| ==Supports health checks== | ==No health checks== |
| Only ==healthy== records returned | All records returned (including unhealthy) |
| Multiple ==separate records== | One record with multiple values |
| Up to 8 healthy records | All values in one record |

> [!important] Multi-Value is NOT a Load Balancer
> Multi-Value routing provides ==client-side load balancing== — the client randomly picks from the returned IPs. It's NOT a substitute for an ELB, which provides server-side load balancing with health checks, sticky sessions, and connection draining. Use Multi-Value for basic DNS-level distribution with health awareness.

## Complete Routing Policy Comparison

| Policy | Based On | Health Checks | Use Case |
|--------|----------|---------------|----------|
| ==Simple== | Nothing (random) | ❌ No | Single resource, basic setup |
| ==Weighted== | Assigned weights | ✅ Yes | Traffic splitting, canary deployments |
| ==Latency== | Network latency | ✅ Yes | Global performance optimization |
| ==Failover== | Health check status | ✅ Required (primary) | Disaster recovery (active-passive) |
| ==Geolocation== | User's country/continent | ✅ Yes | Localization, compliance |
| ==Geoproximity== | Distance + bias | ✅ Yes | Traffic shifting between regions |
| ==IP-Based== | Client IP (CIDR) | ✅ Yes | Known IP ranges (ISPs, corporate) |
| ==Multi-Value== | Health check filtering | ✅ Yes | Client-side LB with health awareness |

## Questions & Answers

> [!question]- Q1: What is the difference between Geolocation and Latency routing?
> **Answer:**
> - ==Geolocation==: Routes based on the user's ==physical location== (country/continent). You define explicit rules. Deterministic — same location always gets the same answer.
> - ==Latency==: Routes based on ==network latency== to AWS regions. Route 53 measures latency dynamically. A user in Germany might be routed to US if latency is lower.
> Use Geolocation for localization/compliance; use Latency for performance.

> [!question]- Q2: What is Geoproximity bias and when would you use it?
> **Answer:**
> Bias is a number (-99 to +99) that ==shifts the geographic boundary== between resources. Positive bias ==expands== a region (attracts more traffic); negative bias ==shrinks== it. Use it when you need to ==gradually shift traffic from one region to another== — for example, during a migration or to handle regional capacity differences.

> [!question]- Q3: Why should you always create a Default geolocation record?
> **Answer:**
> Without a Default record, users whose location doesn't match any rule get ==no DNS answer== (NXDOMAIN). The Default record acts as a ==catch-all== for unmatched locations. For example, if you only define rules for Asia and US, European users would get no answer without a Default.

> [!question]- Q4: What is IP-based routing and when is it useful?
> **Answer:**
> IP-based routing maps ==client IP ranges (CIDRs)== to specific endpoints. Useful when you ==know your users' IP ranges ahead of time== — for example, a specific ISP, corporate network, or content partner. You define CIDR collections in Route 53 and link them to records.

> [!question]- Q5: How does Multi-Value differ from Simple routing with multiple values?
> **Answer:**
> The key difference is ==health checks==. Multi-Value supports health checks and only returns ==healthy records== (up to 8). Simple routing returns ==all values== regardless of health. Multi-Value uses separate records; Simple uses one record with multiple values.

> [!question]- Q6: Is Multi-Value routing a replacement for ELB?
> **Answer:**
> ==No.== Multi-Value provides ==client-side load balancing== — the client randomly picks from returned IPs. ELB provides ==server-side load balancing== with advanced features like health checks, sticky sessions, connection draining, and SSL termination. Multi-Value is a DNS-level feature, not a true load balancer.

> [!question]- Q7: What does Geoproximity require that other policies don't?
> **Answer:**
> Geoproximity requires ==Route 53 Traffic Flow== to use the bias feature. Traffic Flow is an advanced Route 53 feature with a visual editor for creating complex routing configurations. For non-AWS resources, you must also specify ==latitude and longitude==.

> [!question]- Q8: Exam scenario — you need French users to see a French website and German users to see a German website. Which policy?
> **Answer:**
> ==Geolocation routing policy.== Create records:
> - France → French server IP
> - Germany → German server IP
> - Default → English server IP (catch-all)
> Geolocation routes based on the user's country, making it perfect for ==website localization==.

> [!question]- Q9: Exam scenario — you need to gradually shift traffic from us-east-1 to eu-west-1. Which policy?
> **Answer:**
> ==Geoproximity routing with bias.== Start with bias 0 for both regions. Gradually increase the bias for `eu-west-1` (positive) to attract more traffic. This shifts the geographic boundary, causing more users to be routed to EU. This is the only policy that supports ==gradual traffic shifting between regions==.

> [!question]- Q10: Which routing policies support health checks?
> **Answer:**
> All except ==Simple==:
> - Weighted ✅, Latency ✅, Failover ✅ (required for primary), Geolocation ✅, Geoproximity ✅, IP-Based ✅, Multi-Value ✅
> - Simple ❌ — cannot associate health checks
> This is why Multi-Value is preferred over Simple when you need multiple values with health awareness.
