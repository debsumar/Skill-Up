---
title: Gateway Load Balancer (GWLB)
date: 2026-02-09
tags:
  - aws
  - elb
  - gwlb
  - layer-3
  - firewall
  - saa-c03
---

# Gateway Load Balancer (GWLB)

## Overview

==Layer 3 (IP)== load balancer for deploying and scaling third-party network virtual appliances (firewalls, IDS/IPS).

## Use Cases

- ✅ Firewalls
- ✅ Intrusion Detection & Prevention (IDS/IPS)
- ✅ Deep Packet Inspection
- ✅ Payload modification at network level

## How It Works

```
Users
  │
  ▼
┌──────────┐     ┌──────────────────┐     ┌─────────────┐
│  GWLB    │────▶│ Virtual Appliance│────▶│    GWLB     │
│ (entry)  │     │ (Firewall/IDS)   │     │ (forward)   │
└──────────┘     │ Analyze traffic  │     └──────┬──────┘
                 │ Drop or approve  │            │
                 └──────────────────┘            ▼
                                          ┌─────────────┐
                                          │ Application │
                                          └─────────────┘

Traffic: Users → GWLB → Appliances → GWLB → Application
(Transparent to the application)
```

## Key Facts

| Feature | Details |
|---------|---------|
| **Layer** | ==3 (Network — IP packets)== |
| **Protocol** | ==GENEVE on port 6081== |
| **Functions** | Transparent gateway + load balancer |
| **Target Groups** | EC2 instances (by ID) or IP addresses (private) |
| **Route Tables** | Must be updated in VPC |

> [!tip] Exam Keyword
> See ==GENEVE protocol, port 6081, third-party appliances, firewall, IDS/IPS== → think ==GWLB==.

## Questions & Answers

> [!question]- Q1: What is GWLB used for?
> **Answer:**
> To deploy, scale, and manage ==third-party network virtual appliances== like firewalls, IDS/IPS, and deep packet inspection systems. All traffic passes through these appliances before reaching your application.

> [!question]- Q2: What layer does GWLB operate on?
> **Answer:**
> ==Layer 3 (Network layer)==, dealing with IP packets. This is lower than ALB (Layer 7) and NLB (Layer 4).

> [!question]- Q3: What protocol does GWLB use?
> **Answer:**
> ==GENEVE protocol on port 6081==. This is a key exam indicator for GWLB.

> [!question]- Q4: What are the two functions of GWLB?
> **Answer:**
> 1. ==Transparent Network Gateway== — single entry/exit point for all VPC traffic
> 2. ==Load Balancer== — distributes traffic across virtual appliance fleet

> [!question]- Q5: Is GWLB transparent to the application?
> **Answer:**
> ==Yes==. The application doesn't know traffic was inspected. GWLB routes traffic through appliances and back, then forwards to the application transparently.

> [!question]- Q6: What are valid GWLB target types?
> **Answer:**
> - ==EC2 instances== (registered by instance ID)
> - ==IP addresses== (must be private IPs, e.g., on-premises appliances)

> [!question]- Q7: What happens if a virtual appliance rejects traffic?
> **Answer:**
> The appliance ==drops the traffic== (e.g., firewall blocking). Only approved traffic is sent back to GWLB and forwarded to the application.

> [!question]- Q8: Does GWLB require VPC route table changes?
> **Answer:**
> ==Yes==. Route tables must be updated so that VPC traffic flows through the GWLB before reaching the application.

> [!question]- Q9: What exam scenario points to GWLB?
> **Answer:**
> "All network traffic must pass through a ==firewall/IDS== before reaching the application" or "==GENEVE protocol==" or "==third-party virtual appliances==" → GWLB.

> [!question]- Q10: Can you do a hands-on with GWLB easily?
> **Answer:**
> ==No==. GWLB requires third-party virtual appliances and VPC route table configuration, making it complex to demo. Focus on understanding the architecture diagram for the exam.
