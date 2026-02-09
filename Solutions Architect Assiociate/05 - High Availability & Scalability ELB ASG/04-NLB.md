---
title: Network Load Balancer (NLB)
date: 2026-02-09
tags:
  - aws
  - elb
  - nlb
  - layer-4
  - saa-c03
---

# Network Load Balancer (NLB)

## Overview

==Layer 4 (TCP/UDP)== load balancer for extreme performance and static IPs.

## Key Features

| Feature | Details |
|---------|---------|
| **Layer** | 4 (TCP, TLS, UDP) |
| **Performance** | ==Millions of requests/sec==, ultra-low latency |
| **Static IP** | ==One static IP per AZ== (can assign Elastic IP) |
| **Security Groups** | ==Supported== (attach SG to NLB, similar to ALB) |
| **Target Groups** | EC2 instances, IP addresses, ALB |
| **Health Checks** | TCP, HTTP, or HTTPS |
| **Free Tier** | ==Not included== |

> [!important] Exam Keywords
> See ==TCP, UDP, static IP, extreme performance, millions of requests== → think ==NLB==.

## Elastic IP per AZ

When creating an NLB, for each AZ you enable:
- AWS assigns a ==fixed IPv4 address==, OR
- You can assign your own ==Elastic IP== per AZ

This means your application can be accessed via ==1, 2, or 3 fixed IPs== (one per AZ).

## Target Groups

| Target | Description |
|--------|-------------|
| **EC2 Instances** | By instance ID |
| **IP Addresses** | ==Private IPs== (EC2 or on-premises) |
| **ALB** | NLB in front of ALB (static IP + HTTP routing) |

## NLB + ALB Combination

```
┌─────────┐     ┌─────────┐     ┌──────────┐
│   NLB   │────▶│   ALB   │────▶│ Target   │
│ (Static │     │ (HTTP   │     │ Groups   │
│  IPs)   │     │ routing)│     │          │
└─────────┘     └─────────┘     └──────────┘

Why? Get static IPs (NLB) + HTTP routing rules (ALB)
```

## NLB vs ALB

| Feature | ALB | NLB |
|---------|-----|-----|
| Layer | 7 (HTTP) | ==4 (TCP/UDP)== |
| Performance | High | ==Ultra-high (millions req/s)== |
| IP | Fixed hostname | ==Static IP per AZ== |
| Routing | Path, host, query | Port-based |
| Use case | Web apps | Gaming, IoT, financial |

## Questions & Answers

> [!question]- Q1: What layer does NLB operate on?
> **Answer:**
> ==Layer 4 (TCP/UDP)==. It handles raw TCP, TLS, and UDP traffic without inspecting HTTP content.

> [!question]- Q2: When should you use NLB over ALB?
> **Answer:**
> When you need ==extreme performance== (millions of req/s), ==static IPs/Elastic IPs==, or ==TCP/UDP== protocol support. ALB is for HTTP-level routing.

> [!question]- Q3: Can NLB provide static IP addresses?
> **Answer:**
> ==Yes==. NLB has ==one static IP per AZ== and you can assign ==Elastic IPs== to each AZ. This is unique to NLB — ALB only provides a DNS hostname.

> [!question]- Q4: Can you put an NLB in front of an ALB?
> **Answer:**
> ==Yes==. This gives you ==static IPs (NLB)== combined with ==HTTP routing rules (ALB)==. It's a valid and common architecture.

> [!question]- Q5: What health check protocols does NLB support?
> **Answer:**
> ==TCP, HTTP, and HTTPS==. Even though NLB operates at Layer 4, it can perform HTTP/HTTPS health checks on backend applications.

> [!question]- Q6: Is NLB included in the AWS free tier?
> **Answer:**
> ==No==. NLB is not part of the free tier. You pay for NLB hours and data processed.

> [!question]- Q7: What are valid NLB target types?
> **Answer:**
> - ==EC2 instances== (by instance ID)
> - ==IP addresses== (must be private IPs)
> - ==Application Load Balancer== (NLB → ALB)

> [!question]- Q8: What exam scenario points to NLB?
> **Answer:**
> "Application must be accessible via ==1-3 static IPs==" or "need to handle ==millions of requests per second==" or "==UDP== traffic" → NLB.

> [!question]- Q9: How does NLB latency compare to ALB?
> **Answer:**
> NLB has ==ultra-low latency== (~100ms) compared to ALB (~400ms). NLB passes traffic through without inspecting HTTP content.

> [!question]- Q10: Can NLB target on-premises servers?
> **Answer:**
> ==Yes==. Register their ==private IP addresses== as targets. This works via VPN or Direct Connect to your data center.
