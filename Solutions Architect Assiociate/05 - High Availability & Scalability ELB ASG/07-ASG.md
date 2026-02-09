---
title: Auto Scaling Groups (ASG)
date: 2026-02-09
tags:
  - aws
  - asg
  - auto-scaling
  - scaling-policies
  - saa-c03
---

# Auto Scaling Groups (ASG)

## What is an ASG?

Automatically ==scales EC2 instances out/in== based on demand, ensuring the right number of instances are running.

## ASG Capacity Settings

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  Min ◀──────── Desired ──────────▶ Max              │
│   2              4                  7               │
│                                                     │
│  ┌──┐ ┌──┐ ┌──┐ ┌──┐         ┌──┐ ┌──┐ ┌──┐       │
│  │EC│ │EC│ │EC│ │EC│  scale  │EC│ │EC│ │EC│       │
│  │2 │ │2 │ │2 │ │2 │  out ▶  │2 │ │2 │ │2 │       │
│  └──┘ └──┘ └──┘ └──┘         └──┘ └──┘ └──┘       │
│  ◀── minimum ──▶ ◀── desired ──────── maximum ──▶   │
└─────────────────────────────────────────────────────┘
```

## ASG + ELB Integration

| Feature | Details |
|---------|---------|
| **Auto-register** | New instances automatically added to ELB target group |
| **Health checks** | ==EC2 health checks== (default) + ==ELB health checks== (optional, recommended) |
| **Unhealthy** | ASG ==terminates unhealthy instances== and launches replacements |
| **Cost** | ASG is ==free==; you pay for EC2 instances |
| **Activity history** | All scaling events logged in ASG ==Activity tab== |

## Launch Template

Contains all info to launch EC2 instances:

- AMI, instance type
- EC2 User Data, EBS volumes
- Security groups, SSH key pair
- IAM roles, network/subnet info
- Load balancer info

> [!note] Advanced Options
> ASG can use a ==mix of on-demand and spot instances== and ==multiple instance types==. Subnets are configured in the ASG, not the launch template.

## Scaling Policies

### 1. Dynamic Scaling

| Type | How It Works |
|------|-------------|
| **Target Tracking** | Set metric target (e.g., ==CPU at 40%==), ASG auto-adjusts |
| **Simple/Step** | CloudWatch alarm triggers → add/remove N instances |

> [!tip] Target Tracking Behind the Scenes
> Target tracking automatically creates ==two CloudWatch alarms==:
> - **AlarmHigh** — scale out when metric exceeds target (e.g., CPU > 40%)
> - **AlarmLow** — scale in when metric drops below threshold (e.g., CPU < 28%)

### 2. Scheduled Scaling

Anticipate known patterns: "Every Friday 5 PM, set min capacity to 10"

### 3. Predictive Scaling

ASG ==analyzes historical load==, generates forecast, schedules scaling ahead of time. Great for cyclical patterns.

## Good Metrics to Scale On

| Metric | When to Use |
|--------|-------------|
| **CPU Utilization** | Most common — compute-bound apps |
| **RequestCountPerTarget** | Know optimal requests per instance |
| **Average Network In/Out** | Network-bound apps (uploads/downloads) |
| **Custom CloudWatch Metric** | Application-specific |

## Scaling Cooldown

| Feature | Details |
|---------|---------|
| **Default** | ==300 seconds== (5 min) |
| **Purpose** | Allow metrics to stabilize after scaling |
| **During cooldown** | ==No additional scaling actions== |
| **Tip** | Use ready-to-use AMIs → faster boot → shorter cooldown |

> [!important] Enable ==detailed monitoring== on EC2 instances for ==1-minute metric intervals== (default is 5 min). Faster metrics = more responsive scaling.

```
Scaling Action → Cooldown (300s) → Metrics stabilize → Next action allowed
```

## Questions & Answers

> [!question]- Q1: What is an Auto Scaling Group?
> **Answer:**
> An ASG automatically ==scales EC2 instances out (add) or in (remove)== based on demand. You define min, desired, and max capacity. ASG is free — you only pay for EC2 instances.

> [!question]- Q2: What are the three capacity settings of an ASG?
> **Answer:**
> - **Minimum**: Lowest number of instances (always running)
> - **Desired**: Target number of instances
> - **Maximum**: Upper limit for scaling out

> [!question]- Q3: How does ASG integrate with ELB?
> **Answer:**
> New instances are ==automatically registered== with the ELB. ELB health checks are passed to ASG — if an instance is unhealthy, ASG ==terminates it and launches a replacement==.

> [!question]- Q4: What is target tracking scaling?
> **Answer:**
> You set a ==target metric value== (e.g., CPU at 40%), and the ASG automatically adjusts the number of instances to maintain that target. Simplest scaling policy.

> [!question]- Q5: What is predictive scaling?
> **Answer:**
> ASG ==analyzes historical load patterns==, generates a forecast, and ==pre-schedules scaling actions==. Great for cyclical/repeating traffic patterns.

> [!question]- Q6: What is the scaling cooldown?
> **Answer:**
> A ==300-second (default) period after a scaling action== during which no additional scaling occurs. This allows metrics to stabilize before the next decision.

> [!question]- Q7: What is a launch template?
> **Answer:**
> A configuration that defines ==how to launch EC2 instances== in an ASG: AMI, instance type, user data, security groups, key pair, IAM roles, network settings, etc.

> [!question]- Q8: What happens when an ELB health check fails for an ASG instance?
> **Answer:**
> The ASG ==terminates the unhealthy instance== and ==launches a new one== to replace it. The new instance is automatically registered with the ELB.

> [!question]- Q9: How can you reduce the scaling cooldown period?
> **Answer:**
> Use a ==ready-to-use AMI== with pre-installed software. Faster boot time means instances serve traffic sooner, so the cooldown can be shortened for more dynamic scaling.

> [!question]- Q10: What is the difference between scheduled and predictive scaling?
> **Answer:**
> - **Scheduled**: You ==manually define== when to scale (e.g., "Fridays at 5 PM, min = 10")
> - **Predictive**: ASG ==automatically forecasts== load from historical data and schedules scaling ahead of time
