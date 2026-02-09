---
title: Routing Policies - Simple, Weighted, Latency
date: 2026-02-10
tags:
  - aws
  - route53
  - routing-policy
  - weighted
  - latency
  - saa-c03
---

# Routing Policies — Simple, Weighted, Latency

> [!note] DNS Routing ≠ Load Balancer Routing
> DNS does ==not route traffic==. It only responds to DNS queries. The client then connects directly to the resource.

## Simple

| Feature | Detail |
|---------|--------|
| **Use case** | Route to a ==single resource== |
| **Multiple values** | Yes — client picks ==randomly== |
| **Alias** | If enabled, only ==one AWS resource== as target |
| **Health checks** | ==Not supported== |

```
Client ──▶ Route 53 ──▶ Returns: 1.2.3.4, 5.6.7.8, 9.10.11.12
                          Client picks one randomly
```

## Weighted

| Feature | Detail |
|---------|--------|
| **Use case** | Control ==% of traffic== to each resource |
| **Weights** | Relative values (==don't need to sum to 100==) |
| **Formula** | Traffic % = weight / sum of all weights |
| **Health checks** | ==Supported== |
| **Weight = 0** | ==Stop sending traffic== to that resource |
| **All weights = 0** | All records returned ==equally== |
| **Records** | Must have ==same name and type== |

```
Route 53
├── Weight 70 ──▶ EC2 us-east-1    (70%)
├── Weight 20 ──▶ EC2 eu-central-1 (20%)
└── Weight 10 ──▶ EC2 ap-southeast (10%)
```

> [!tip] Use Cases
> - ==Load balancing== across regions
> - ==Testing new app versions== (send small % of traffic)
> - ==Gradual migration== between environments

## Latency

| Feature | Detail |
|---------|--------|
| **Use case** | Redirect to resource with ==lowest latency== |
| **Measurement** | Based on user → ==AWS region== latency |
| **Health checks** | ==Supported== |
| **Region** | Must specify the ==region== for each record |

```
┌───────────────────────────────────────────────┐
│  User in Germany ─▶ eu-central-1 (lowest latency) │
│  User in Japan   ─▶ ap-northeast-1               │
│  User in USA     ─▶ us-east-1                    │
└───────────────────────────────────────────────┘
```

> [!important] Latency ≠ Geography
> A user in Germany ==could== be routed to the US if latency to a US resource is lower. It's about ==network latency==, not physical distance.

## Questions & Answers

> [!question]- Q1: Does DNS route traffic?
> **Answer:**
> ==No==. DNS only responds to queries with IP addresses. The client then connects directly to the resource. Traffic does not flow through DNS.

> [!question]- Q2: Can Simple routing policy use health checks?
> **Answer:**
> ==No==. Simple routing does not support health checks.

> [!question]- Q3: What happens when Simple returns multiple values?
> **Answer:**
> The client receives all values and picks one ==randomly== (client-side selection).

> [!question]- Q4: Do Weighted record weights need to sum to 100?
> **Answer:**
> ==No==. Weights are relative. Traffic % = weight / sum of all weights.

> [!question]- Q5: What happens if you set a weight to 0?
> **Answer:**
> ==No traffic== is sent to that resource. If all weights are 0, all records are returned equally.

> [!question]- Q6: What is a use case for Weighted routing?
> **Answer:**
> - Load balancing across regions
> - ==Testing new app versions== by sending a small % of traffic
> - Gradual migration between environments

> [!question]- Q7: How does Latency routing determine the best resource?
> **Answer:**
> Based on the ==lowest latency== between the user and the ==AWS region== where the resource is located.

> [!question]- Q8: Is Latency routing based on geographic distance?
> **Answer:**
> ==No==. It's based on ==network latency==. A user in Germany could be routed to the US if that has lower latency.

> [!question]- Q9: Do you need to specify a region for Latency records?
> **Answer:**
> ==Yes==. You must specify which AWS region each record corresponds to.

> [!question]- Q10: Which routing policies support health checks?
> **Answer:**
> ==Weighted and Latency== support health checks. ==Simple does not==.