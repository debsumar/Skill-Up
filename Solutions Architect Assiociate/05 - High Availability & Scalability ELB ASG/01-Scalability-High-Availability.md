---
title: Scalability & High Availability
date: 2026-02-09
tags:
  - aws
  - scalability
  - high-availability
  - saa-c03
---

# Scalability & High Availability

## Two Types of Scalability

| Type | Meaning | EC2 Example | Use Case |
|------|---------|-------------|----------|
| **Vertical** | ==Increase instance size== (scale up/down) | t2.micro → t2.large | Databases (RDS, ElastiCache) |
| **Horizontal** | ==Increase number of instances== (scale out/in) | 1 instance → 6 instances | Web apps, distributed systems |

> [!tip] Call Center Analogy
> - **Vertical**: Replace junior operator (5 calls/min) with senior (10 calls/min)
> - **Horizontal**: Hire 6 operators instead of 1

## High Availability

Running your application in ==at least 2 AZs== to survive a data center loss.

| Type | Example |
|------|---------|
| **Passive** | RDS Multi-AZ (standby replica) |
| **Active** | Multiple EC2 instances across AZs behind a load balancer |

```
High Availability:
┌──────────────┐     ┌──────────────┐
│  us-east-1a  │     │  us-east-1b  │
│  ┌────┐┌────┐│     │  ┌────┐┌────┐│
│  │EC2 ││EC2 ││     │  │EC2 ││EC2 ││
│  └────┘└────┘│     │  └────┘└────┘│
└──────┬───────┘     └──────┬───────┘
       └────────┬───────────┘
           ┌────┴────┐
           │   ELB   │
           └─────────┘
```

## EC2 Scaling Summary

| Concept | Direction | Mechanism |
|---------|-----------|-----------|
| Vertical Scale Up | t2.nano → u-12tb1.metal | Bigger instance |
| Horizontal Scale Out | 1 → N instances | Auto Scaling Group |
| Horizontal Scale In | N → fewer instances | Auto Scaling Group |
| High Availability | Multi-AZ | ASG multi-AZ + ELB multi-AZ |

## Questions & Answers

> [!question]- Q1: What is the difference between vertical and horizontal scaling?
> **Answer:**
> - **Vertical**: ==Increase instance size== (e.g., t2.micro → t2.large). Limited by hardware.
> - **Horizontal**: ==Increase number of instances==. Used for distributed systems. Enabled by ASG/ELB.

> [!question]- Q2: What does high availability mean in AWS?
> **Answer:**
> Running your application in ==at least 2 Availability Zones== so it survives a data center failure. Achieved with multi-AZ ASG and multi-AZ ELB.

> [!question]- Q3: When would you use vertical scaling?
> **Answer:**
> For ==non-distributed systems== like databases (RDS, ElastiCache). You upgrade the instance type. Limited by hardware maximum.

> [!question]- Q4: What is the difference between scale out and scale in?
> **Answer:**
> - **Scale out**: ==Add== EC2 instances (increased load)
> - **Scale in**: ==Remove== EC2 instances (decreased load)

> [!question]- Q5: Can high availability be passive?
> **Answer:**
> ==Yes==. Example: RDS Multi-AZ has a standby replica that only activates on failure. Active HA means all instances serve traffic simultaneously (e.g., EC2 behind ELB across AZs).

> [!question]- Q6: What is the largest EC2 instance type?
> **Answer:**
> Currently ==u-12tb1.metal== with 12.3 TB RAM and 448 vCPUs. The smallest is ==t2.nano== with 0.5 GB RAM and 1 vCPU.

> [!question]- Q7: Is horizontal scaling always possible?
> **Answer:**
> ==No==. It requires a ==distributed system== architecture. Not every application can be distributed. Databases often use vertical scaling instead.

> [!question]- Q8: What AWS services enable horizontal scaling?
> **Answer:**
> - ==Auto Scaling Groups (ASG)== — automatically add/remove EC2 instances
> - ==Elastic Load Balancer (ELB)== — distribute traffic across instances

> [!question]- Q9: What is elasticity in AWS?
> **Answer:**
> Elasticity is ==horizontal scaling== — the ability to automatically scale out and in based on demand. The system adapts to load changes dynamically.

> [!question]- Q10: How do scalability and high availability relate?
> **Answer:**
> They're ==linked but different==. Scalability handles increased load (vertical or horizontal). High availability ensures survival of failures (multi-AZ). You can have scalability without HA and vice versa, but best practice is both.
