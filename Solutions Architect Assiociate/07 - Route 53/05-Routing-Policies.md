---
title: "Route 53 - Routing Policies (Simple, Weighted, Latency)"
date: 2026-02-10
tags:
  - aws
  - route53
  - dns
  - routing-policy
  - saa-c03
---

# Route 53 - Routing Policies (Simple, Weighted, Latency)

## Overview

A ==routing policy== defines how Route 53 responds to DNS queries. It's important to understand that DNS does ==not route traffic== — it only resolves hostnames to IP addresses. The client then connects directly to the IP. Route 53's "routing" is about ==which IP address to return== in the DNS response.

Route 53 supports seven routing policies:
1. ==Simple== ← this lecture
2. ==Weighted== ← this lecture
3. ==Latency-based== ← this lecture
4. Failover (covered in [[06-Health-Checks|Health Checks]])
5. Geolocation (covered in [[07-Routing-Policies-Advanced|Advanced Routing]])
6. Geoproximity (covered in [[07-Routing-Policies-Advanced|Advanced Routing]])
7. Multi-Value (covered in [[07-Routing-Policies-Advanced|Advanced Routing]])
8. IP-based (covered in [[07-Routing-Policies-Advanced|Advanced Routing]])

## Simple Routing Policy

The most basic policy — route traffic to a ==single resource==.

```
┌──────────┐   "foo.example.com?"    ┌──────────────┐   A: 11.22.33.44
│  Client  │────────────────────────▶│   Route 53   │──────────────────▶ EC2
│          │◀────────────────────────│   (Simple)   │
│          │   "11.22.33.44"         └──────────────┘
└──────────┘
```

### Key Characteristics

| Feature | Detail |
|---------|--------|
| **Multiple values** | You ==can== specify multiple IPs in a single record |
| **Client behavior** | If multiple values returned, client picks one ==randomly== |
| **Alias** | If Alias is enabled, can only specify ==one AWS resource== |
| **Health checks** | ==Cannot== be associated with health checks |

### Multiple Values in Simple Routing

You can put multiple IP addresses in a single Simple record:

```
┌──────────┐   "simple.example.com?"   ┌──────────────┐
│  Client  │──────────────────────────▶│   Route 53   │
│          │◀──────────────────────────│   (Simple)   │
│          │   Returns ALL values:     └──────────────┘
│          │   - 13.212.xxx (Singapore)
│          │   - 54.xxx.xxx (US East)
│          │
│          │   Client picks one RANDOMLY
└──────────┘
```

When multiple values are returned, the ==client randomly selects one==. This provides basic load distribution but with no intelligence — no health checks, no geographic awareness, no weighting.

> [!note] Simple vs Multi-Value
> Simple routing with multiple values looks similar to Multi-Value routing, but there's a key difference: Simple routing ==cannot use health checks==, so unhealthy resources may be returned. Multi-Value routing ==can use health checks== and only returns healthy resources.

## Weighted Routing Policy

Control the ==percentage of traffic== sent to each resource by assigning ==weights==.

```
┌──────────────┐
│   Route 53   │
│  (Weighted)  │
└──┬───┬───┬───┘
   │   │   │
   │   │   │  Weight: 70 (70%)
   │   │   └──────────────────▶ EC2 in us-east-1
   │   │
   │   │  Weight: 20 (20%)
   │   └──────────────────────▶ EC2 in eu-central-1
   │
   │  Weight: 10 (10%)
   └──────────────────────────▶ EC2 in ap-southeast-1
```

### How Weights Work

The traffic percentage for each record is calculated as:

```
Traffic % = (Weight of record) / (Sum of all weights) × 100
```

| Record | Weight | Calculation | Traffic % |
|--------|--------|-------------|-----------|
| us-east-1 | 70 | 70 / (70+20+10) | ==70%== |
| eu-central-1 | 20 | 20 / (70+20+10) | ==20%== |
| ap-southeast-1 | 10 | 10 / (70+20+10) | ==10%== |

### Key Characteristics

| Feature | Detail |
|---------|--------|
| **Weights** | Don't need to sum to 100 — they're ==relative== |
| **Same name & type** | All records must have the ==same record name and type== |
| **Health checks** | ==Can== be associated with health checks |
| **Weight of 0** | Stops sending traffic to that resource |
| **All weights 0** | All records returned with ==equal weight== |
| **Record ID** | Each weighted record needs a unique ==Record ID== |

### Use Cases

- ==Load balancing== across regions
- ==Testing new application versions== — send 10% of traffic to the new version (canary deployment)
- ==Gradual migration== — slowly shift traffic from old to new infrastructure
- ==A/B testing== — split traffic between different backends

> [!tip] Exam Tip
> If the exam mentions "send a small percentage of traffic to test a new version" or "gradually shift traffic between regions," the answer is ==Weighted routing policy==.

### Hands-On

Creating weighted records requires creating ==multiple records with the same name==:

1. Record: `weighted.yourdomain.com` → IP of Singapore → Weight: 10 → Record ID: "southeast"
2. Record: `weighted.yourdomain.com` → IP of US East → Weight: 70 → Record ID: "us-east"
3. Record: `weighted.yourdomain.com` → IP of EU Central → Weight: 20 → Record ID: "eu"

With a very low TTL (3 seconds for demo), refreshing the browser shows different regions based on the weight distribution — US East appears most often (70%), EU Central occasionally (20%), and Singapore rarely (10%).

## Latency-Based Routing Policy

Route users to the resource with the ==lowest latency== (closest AWS region).

```
┌──────────────────────────────────────────────────────────────┐
│                    LATENCY-BASED ROUTING                     │
│                                                               │
│  User in Europe ──▶ Route 53 ──▶ eu-central-1 (lowest lat.) │
│  User in US     ──▶ Route 53 ──▶ us-east-1 (lowest lat.)    │
│  User in Asia   ──▶ Route 53 ──▶ ap-southeast-1 (lowest lat)│
│                                                               │
│  Route 53 measures latency between user and AWS regions      │
│  and returns the record for the region with lowest latency   │
└──────────────────────────────────────────────────────────────┘
```

### How It Works

1. You create multiple records with the ==same name==, each pointing to a resource in a different region
2. For each record, you specify the ==AWS region== the resource is in
3. When a user makes a DNS query, Route 53 evaluates the ==latency from the user's location to each region==
4. Route 53 returns the IP of the resource in the ==region with the lowest latency==

### Key Characteristics

| Feature | Detail |
|---------|--------|
| **Latency measurement** | Based on ==user-to-AWS-region== latency, not user-to-resource |
| **Region specification** | You must specify the ==AWS region== for each record |
| **Health checks** | ==Can== be associated with health checks |
| **Not geographic** | A user in Germany ==might== be routed to a US resource if latency is lower |
| **Record ID** | Each latency record needs a unique Record ID |

> [!important] Latency ≠ Geography
> Latency-based routing is about ==network latency==, not physical distance. A user in Germany might have lower latency to a US East resource than to an Asia Pacific resource, even though Asia is physically closer in some directions. Route 53 uses actual network measurements, not geographic calculations.

### Hands-On

Creating latency records:

1. Record: `latency.yourdomain.com` → IP of Singapore → Region: `ap-southeast-1`
2. Record: `latency.yourdomain.com` → IP of US East → Region: `us-east-1`
3. Record: `latency.yourdomain.com` → IP of EU Central → Region: `eu-central-1`

Testing from different locations:
- From Europe → gets `eu-central-1` (lowest latency)
- Using VPN to Canada → gets `us-east-1` (lowest latency from North America)
- Using VPN to Hong Kong → gets `ap-southeast-1` (lowest latency from Asia)

> [!tip] Exam Tip
> If the exam mentions "route users to the closest region," "minimize latency," or "best performance for global users," the answer is ==Latency-based routing policy==.

## Comparison: Simple vs Weighted vs Latency

| Feature | Simple | Weighted | Latency |
|---------|--------|----------|---------|
| **Use case** | Single resource or random distribution | Control traffic % per resource | Route to lowest-latency region |
| **Multiple records** | Single record with multiple values | Multiple records, same name | Multiple records, same name |
| **Health checks** | ==No== | ==Yes== | ==Yes== |
| **Intelligence** | None — random client selection | Weight-based distribution | Latency-based selection |
| **Record ID** | Not needed | ==Required== | ==Required== |
| **Alias support** | One resource only | Yes | Yes |

## Questions & Answers

> [!question]- Q1: What does "routing policy" mean in Route 53?
> **Answer:**
> A routing policy defines ==how Route 53 responds to DNS queries== — which IP address to return. It does NOT route traffic. DNS only resolves hostnames to addresses. The client then connects directly to the returned IP. The "routing" is about ==choosing which answer to give==.

> [!question]- Q2: How does Simple routing handle multiple values?
> **Answer:**
> You can put multiple IP addresses in a single Simple record. Route 53 returns ==all values== in the DNS response, and the ==client randomly picks one==. There's no intelligence — no health checks, no weighting, no geographic awareness. If one IP is unhealthy, it may still be returned.

> [!question]- Q3: How do weights work in Weighted routing?
> **Answer:**
> Each record gets a weight (any number). Traffic percentage = weight / sum of all weights × 100. Weights ==don't need to sum to 100== — they're relative. A weight of 0 stops traffic to that resource. If all weights are 0, all records are returned equally.

> [!question]- Q4: What's the difference between Weighted and Simple with multiple values?
> **Answer:**
> - ==Simple==: One record with multiple IPs, client picks randomly (equal probability), ==no health checks==
> - ==Weighted==: Multiple records with the same name, each with a weight controlling traffic %, ==supports health checks==, each record returns ==one value==

> [!question]- Q5: How does Latency-based routing determine the "closest" region?
> **Answer:**
> Route 53 measures ==network latency== between the user's location and each AWS region. It returns the record for the region with the ==lowest latency==. This is based on actual network measurements, not geographic distance. A user in Germany might be routed to US East if the network path is faster.

> [!question]- Q6: Do you need to specify the AWS region for Latency records?
> **Answer:**
> ==Yes.== When creating a latency record, you must specify the ==AWS region== the resource is in. Route 53 uses this to evaluate latency from the user to that region. If you just put an IP address, Route 53 doesn't automatically know which region it belongs to.

> [!question]- Q7: Can Latency routing send a European user to a US resource?
> **Answer:**
> ==Yes.== Latency routing is based on ==network latency, not geography==. If a European user has lower latency to US East than to Asia Pacific, they'll be routed to US East. In practice, users are usually routed to the geographically closest region, but not always.

> [!question]- Q8: What is a Record ID and when is it needed?
> **Answer:**
> A Record ID is a ==unique identifier== for each record within a set of records with the same name. It's required for ==Weighted, Latency, Failover, Geolocation, Geoproximity, Multi-Value, and IP-based== routing policies. It's NOT needed for Simple routing. Examples: "us-east", "eu-central", "asia".

> [!question]- Q9: Exam scenario — you want to send 5% of traffic to a new app version. Which policy?
> **Answer:**
> ==Weighted routing policy.== Create two records with the same name:
> - Old version: Weight 95 (95% of traffic)
> - New version: Weight 5 (5% of traffic)
> This is a ==canary deployment== pattern. You can gradually increase the new version's weight as confidence grows.

> [!question]- Q10: Exam scenario — global users complain about slow website. Which policy?
> **Answer:**
> ==Latency-based routing policy.== Deploy your application in multiple AWS regions and create latency records for each. Route 53 will automatically direct each user to the region with the ==lowest latency==, ensuring the best performance for users worldwide.
