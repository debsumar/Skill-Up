---
title: Elastic Beanstalk Overview
date: 2026-02-10
tags:
  - aws
  - elastic-beanstalk
  - solutions-architecture
  - saa-c03
---

# Elastic Beanstalk Overview

## Overview

==Elastic Beanstalk== is a developer-centric managed service for deploying applications on AWS. It reuses familiar components (EC2, ASG, ELB, RDS) but handles all provisioning, configuration, and scaling automatically. You only manage the ==application code==.

## The Problem Beanstalk Solves

```
Most web apps follow the same pattern:

┌──────┐   ┌─────┐   ┌─────┐   ┌─────┐   ┌──────────────┐
│ User │──▶│ ELB │──▶│ ASG │──▶│ EC2 │──▶│ RDS/Cache    │
└──────┘   └─────┘   └─────┘   └─────┘   └──────────────┘

Recreating this for every app is tedious.
Beanstalk automates it.
```

## Key Characteristics

| Feature | Detail |
|---------|--------|
| **Cost** | ==Free== (pay only for underlying resources) |
| **Control** | Full control over component configuration |
| **Managed** | Capacity provisioning, LB, scaling, health monitoring |
| **Developer focus** | Only responsibility is the ==application code== |
| **Updates** | Built-in application update/deployment strategies |

## Beanstalk Components

```
Application
├── Version 1
├── Version 2
├── Version 3
│
├── Environment: dev
│   └── Running Version 2
├── Environment: test
│   └── Running Version 3
└── Environment: prod
    └── Running Version 1
```

| Component | Description |
|-----------|-------------|
| **Application** | Collection of environments, versions, and configurations |
| **Version** | An iteration of your application code |
| **Environment** | AWS resources running a specific version |
| **Tier** | Web Server or Worker |

## Environment Tiers

### Web Server Tier

```
┌──────┐   ┌─────────────┐   ┌─────────────────────┐
│ User │──▶│     ELB     │──▶│       ASG            │
└──────┘   └─────────────┘   │  EC2  EC2  EC2       │
                              │  (Web Servers)       │
                              └─────────────────────┘
```

- Traditional request-driven architecture
- ELB distributes traffic to EC2 instances in ASG

### Worker Tier

```
┌─────────────┐   ┌─────────────────────┐
│  SQS Queue  │──▶│       ASG            │
│  (Messages) │   │  EC2  EC2  EC2       │
└─────────────┘   │  (Workers)           │
                  └─────────────────────┘
```

- No direct client access
- EC2 instances ==pull messages== from SQS queue
- Scales based on ==number of SQS messages==
- Web tier can push messages to worker tier's SQS queue

### Combined Architecture

```
┌──────┐   ┌─────┐   ┌──────────┐   ┌───────────┐   ┌──────────┐
│ User │──▶│ ELB │──▶│ Web Tier │──▶│ SQS Queue │──▶│ Worker   │
└──────┘   └─────┘   │ (ASG)    │   └───────────┘   │ Tier     │
                      └──────────┘                    │ (ASG)    │
                                                      └──────────┘
```

## Deployment Modes

| Mode | Architecture | Use Case |
|------|-------------|----------|
| **Single Instance** | 1 EC2 + Elastic IP (+ optional RDS) | ==Development== |
| **High Availability** | ELB + ASG (Multi-AZ) + optional RDS Multi-AZ | ==Production== |

### Single Instance Mode

```
┌──────────────┐
│ EC2 Instance │
│ + Elastic IP │
│ + ASG (1)    │
│              │
│ Optional:    │
│ + RDS        │
└──────────────┘
```

- Free tier eligible
- Great for dev/test

### High Availability Mode

```
┌─────┐   ┌─────────────────────┐   ┌──────────────────┐
│ ELB │──▶│       ASG            │──▶│ RDS Master       │
│     │   │ AZ1: EC2  EC2       │   │ RDS Standby      │
│     │   │ AZ2: EC2  EC2       │   │ (Multi-AZ)       │
└─────┘   └─────────────────────┘   └──────────────────┘
```

- Load balanced across multiple AZs
- Auto scaling
- RDS Multi-AZ with master + standby

## Supported Platforms

| Platform | Languages/Runtimes |
|----------|-------------------|
| **Languages** | Go, Java SE, Java Tomcat, .NET Core (Linux), .NET (Windows), Node.js, PHP, Python, Ruby |
| **Containers** | Single Docker, Multi Docker, Pre-configured Docker |
| **Builder** | Packer Builder |

## Beanstalk Workflow

```
Create Application
       │
       ▼
Upload Version (code)
       │
       ▼
Launch Environment
       │
       ▼
Manage Lifecycle
       │
       ▼
Update Version ──▶ Deploy to Environment
```

## Beanstalk vs CloudFormation

| Aspect | Beanstalk | CloudFormation |
|--------|-----------|----------------|
| **Focus** | ==Application code== + environments | ==Infrastructure== (any AWS resource) |
| **Abstraction** | High — managed service | Low — you define everything |
| **Use Case** | Web apps with standard patterns | Arbitrary infrastructure stacks |
| **Under the hood** | Uses CloudFormation internally | Core IaC service |

## Questions & Answers

> [!question]- Q1: What is Elastic Beanstalk?
> **Answer:**
> A ==developer-centric managed service== that deploys applications on AWS using familiar components (EC2, ELB, ASG, RDS). It handles provisioning, scaling, and monitoring — you only manage the code.

> [!question]- Q2: Is Beanstalk free?
> **Answer:**
> The Beanstalk service itself is ==free==. You only pay for the underlying AWS resources it provisions (EC2 instances, ELB, RDS, etc.).

> [!question]- Q3: What is the difference between Web Server tier and Worker tier?
> **Answer:**
> - ==Web Server==: ELB → ASG → EC2 (handles HTTP requests from users)
> - ==Worker==: SQS Queue → ASG → EC2 (processes background tasks from a message queue)
>
> Worker tier scales based on SQS queue depth.

> [!question]- Q4: What are the two deployment modes?
> **Answer:**
> 1. ==Single Instance==: 1 EC2 + Elastic IP — for development (free tier eligible)
> 2. ==High Availability==: ELB + ASG (Multi-AZ) + optional RDS Multi-AZ — for production

> [!question]- Q5: What are the components of a Beanstalk application?
> **Answer:**
> - **Application**: top-level container
> - **Version**: iteration of code (v1, v2, v3)
> - **Environment**: AWS resources running a specific version (dev, test, prod)
> - **Tier**: Web Server or Worker

> [!question]- Q6: How does Beanstalk relate to CloudFormation?
> **Answer:**
> Beanstalk ==uses CloudFormation internally== to provision resources. CloudFormation is a general-purpose IaC service; Beanstalk is a higher-level abstraction specifically for application deployment.

> [!question]- Q7: Can you have multiple environments in one Beanstalk application?
> **Answer:**
> Yes. You can create multiple environments (e.g., dev, test, prod) within a single application, each running different versions of your code.

> [!question]- Q8: How can Web and Worker tiers work together?
> **Answer:**
> The Web tier handles user requests and pushes messages to an ==SQS queue==. The Worker tier pulls messages from the queue and processes them asynchronously. This decouples request handling from background processing.

> [!question]- Q9: What does Beanstalk manage automatically?
> **Answer:**
> - Capacity provisioning
> - Load balancer configuration
> - Auto scaling
> - Application health monitoring
> - Instance configuration
> - Platform updates

> [!question]- Q10: What programming languages does Beanstalk support?
> **Answer:**
> Go, Java SE, Java Tomcat, .NET Core (Linux), .NET (Windows), Node.js, PHP, Python, Ruby, and Docker containers (single, multi, pre-configured).
