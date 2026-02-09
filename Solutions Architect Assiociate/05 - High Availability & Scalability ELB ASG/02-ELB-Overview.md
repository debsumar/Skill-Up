---
title: Elastic Load Balancer (ELB) Overview
date: 2026-02-09
tags:
  - aws
  - elb
  - load-balancer
  - saa-c03
---

# Elastic Load Balancer (ELB) Overview

## What is a Load Balancer?

A server that ==forwards traffic to multiple downstream EC2 instances==, providing a single point of access.

```
        ┌─────────┐
User 1 ─┤         ├──▶ EC2 Instance A
User 2 ─┤   ELB   ├──▶ EC2 Instance B
User 3 ─┤         ├──▶ EC2 Instance C
        └─────────┘
Users only know the ELB endpoint
```

## Why Use ELB?

- Spread load across instances
- ==Single DNS endpoint== for your application
- Handle instance failures via ==health checks==
- SSL termination (HTTPS)
- Cookie-based stickiness
- High availability across AZs
- Separate public from private traffic

## Health Checks

ELB checks if instances are healthy before sending traffic.

| Setting | Example |
|---------|---------|
| Protocol | HTTP |
| Port | 4567 |
| Endpoint | `/health` |
| Healthy response | ==200 OK== |

> [!warning] If instance doesn't return 200 → marked ==unhealthy== → no traffic sent to it.

## Four Types of Load Balancers

| Type | Year | Layer | Protocols | Key Feature |
|------|------|-------|-----------|-------------|
| **CLB** | 2009 | 4/7 | HTTP, HTTPS, TCP, SSL | ==Deprecated== |
| **ALB** | 2016 | ==7== | HTTP, HTTPS, WebSocket | Path/host routing |
| **NLB** | 2017 | ==4== | TCP, TLS, UDP | Static IP, millions req/s |
| **GWLB** | 2020 | ==3== | IP | Firewall, IDS/IPS |

> [!tip] Always use newer generation (ALB, NLB, GWLB) for more features.

## Security Groups Pattern

```
Internet → ELB Security Group → EC2 Security Group

ELB SG:                          EC2 SG:
┌──────────────────┐             ┌──────────────────────┐
│ Inbound:         │             │ Inbound:             │
│ HTTP  80  0.0.0.0│             │ HTTP  80  sg-elb-xxx │
│ HTTPS 443 0.0.0.0│             │ (source = ELB SG!)   │
└──────────────────┘             └──────────────────────┘
```

> [!important] EC2 instances should only accept traffic ==from the ELB security group==, not from the internet directly.

## Questions & Answers

> [!question]- Q1: What is the main purpose of a load balancer?
> **Answer:**
> To ==distribute incoming traffic across multiple backend instances==, providing a single endpoint, handling failures via health checks, and enabling high availability.

> [!question]- Q2: What are the four types of AWS load balancers?
> **Answer:**
> 1. **CLB** (Classic) — deprecated, Layer 4/7
> 2. **ALB** (Application) — Layer 7, HTTP routing
> 3. **NLB** (Network) — Layer 4, TCP/UDP, static IP
> 4. **GWLB** (Gateway) — Layer 3, network traffic inspection

> [!question]- Q3: How do ELB health checks work?
> **Answer:**
> ELB sends requests to a configured ==port and endpoint== (e.g., HTTP:80/health). If the instance returns ==200 OK==, it's healthy. Otherwise, it's marked unhealthy and receives no traffic.

> [!question]- Q4: How should you configure EC2 security groups with an ELB?
> **Answer:**
> EC2 instances should allow inbound traffic ==only from the ELB's security group== (not 0.0.0.0/0). This ensures instances only receive traffic through the load balancer.

> [!question]- Q5: Can a load balancer be internal (private)?
> **Answer:**
> ==Yes==. ELBs can be ==internet-facing (public)== or ==internal (private)==. Internal ELBs are used for communication between application tiers within a VPC.

> [!question]- Q6: Is ELB a managed service?
> **Answer:**
> ==Yes==. AWS manages upgrades, maintenance, and high availability. You only configure behavior. It's cheaper and easier than running your own load balancer.

> [!question]- Q7: What AWS services integrate with ELB?
> **Answer:**
> EC2, Auto Scaling Groups, ECS, ACM (Certificate Manager), CloudWatch, Route 53, WAF, Global Accelerator, and more.

> [!question]- Q8: What happens when an instance fails a health check?
> **Answer:**
> The ELB marks it as ==unhealthy== and ==stops sending traffic== to it. If paired with an ASG, the ASG can terminate the unhealthy instance and launch a replacement.

> [!question]- Q9: What is SSL termination?
> **Answer:**
> The ELB ==decrypts HTTPS traffic== from clients and forwards ==unencrypted HTTP== to backend instances over the private VPC network. This offloads encryption work from EC2 instances.

> [!question]- Q10: Which load balancer should you use for a new application?
> **Answer:**
> ==ALB== for HTTP/HTTPS web applications (most common). ==NLB== for TCP/UDP or when you need static IPs. ==GWLB== for network traffic inspection. Never use CLB for new projects.
