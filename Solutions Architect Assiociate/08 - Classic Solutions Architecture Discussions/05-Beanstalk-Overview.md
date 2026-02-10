---
title: "AWS Elastic Beanstalk - Overview"
date: 2026-02-10
tags:
  - aws
  - elastic-beanstalk
  - paas
  - solutions-architecture
  - saa-c03
---

# AWS Elastic Beanstalk - Overview

## Overview

Throughout this section we've been building architectures manually — setting up ELBs, ASGs, RDS, ElastiCache, security groups, and wiring them all together. As a ==developer==, this is a lot of infrastructure work. What if you just want to ==deploy your application== and let AWS handle the rest?

That's exactly what ==AWS Elastic Beanstalk== does. It's a ==developer-centric== view of deploying applications on AWS. You still use all the same components we've learned — EC2, ASG, ELB, RDS — but Beanstalk ==manages them for you==. It's a Platform as a Service (PaaS) that sits on top of AWS infrastructure services.

> [!important] Key Distinction for the Exam
> Beanstalk is ==not a separate service== that replaces EC2 or ELB. It's an ==orchestration layer== that creates and manages those services for you. Under the hood, it uses ==CloudFormation== to provision everything. You still have full access to the underlying resources.

## The Developer's Problem

As a developer, you don't want to:
- Manage infrastructure
- Configure load balancers
- Set up auto scaling
- Monitor instance health
- Handle deployments manually
- Worry about capacity planning

You just want to ==deploy code== and have it run reliably at scale. This is the problem Beanstalk solves.

```
WITHOUT Beanstalk (Manual)                WITH Beanstalk (Managed)
┌──────────────────────────┐              ┌──────────────────────────┐
│  Developer must:         │              │  Developer must:         │
│                          │              │                          │
│  ✗ Create VPC & subnets  │              │  ✅ Upload code          │
│  ✗ Configure ELB         │              │  ✅ Set a few configs    │
│  ✗ Set up ASG            │              │  ✅ Done!                │
│  ✗ Create RDS            │              │                          │
│  ✗ Configure SGs         │              │  Beanstalk handles:      │
│  ✗ Set up monitoring     │              │  ✅ ELB, ASG, EC2, RDS   │
│  ✗ Handle deployments    │              │  ✅ Monitoring & alerts   │
│  ✗ Manage scaling        │              │  ✅ Deployments           │
│  ✗ OS patching           │              │  ✅ Scaling & patching    │
│                          │              │                          │
│  Time: days/weeks        │              │  Time: minutes           │
└──────────────────────────┘              └──────────────────────────┘
```

## Beanstalk is Free (Sort Of)

Beanstalk itself is ==free==. You don't pay for the Beanstalk service. You only pay for the ==underlying resources== it creates — the EC2 instances, ELB, RDS, etc. This is an important exam point: Beanstalk adds ==no additional cost== beyond the infrastructure it provisions.

## Beanstalk Components

Beanstalk organizes your deployment into three hierarchical components:

```
┌─────────────────────────────────────────────────────────────────┐
│                      APPLICATION                                │
│  (e.g., "MyWebApp")                                            │
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                APPLICATION VERSIONS                       │  │
│  │  v1.0  ──▶  v1.1  ──▶  v1.2  ──▶  v2.0  ──▶  v2.1     │  │
│  │  (each version = a zip of your code)                      │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌──────────────────────┐  ┌──────────────────────┐            │
│  │    ENVIRONMENT        │  │    ENVIRONMENT        │            │
│  │    "production"       │  │    "staging"          │            │
│  │                       │  │                       │            │
│  │  Running v2.0         │  │  Running v2.1         │            │
│  │                       │  │  (testing before      │            │
│  │  ELB + ASG + EC2     │  │   promoting to prod)  │            │
│  │  + RDS               │  │                       │            │
│  │                       │  │  ELB + ASG + EC2     │            │
│  └──────────────────────┘  └──────────────────────┘            │
└─────────────────────────────────────────────────────────────────┘
```

### 1. Application
The top-level container. Think of it as your project — "MyWebApp", "MyAPI", etc. An application contains versions and environments.

### 2. Application Version
Each time you upload your code, it becomes a ==version==. Versions are immutable — v1.0 is always v1.0. You can deploy any version to any environment. This makes ==rollbacks== trivial — just deploy the previous version.

### 3. Environment
An environment is a ==running instance== of your application. It's the actual infrastructure — the EC2 instances, load balancer, database, etc. You can have multiple environments per application (dev, staging, production), each running a different version.

> [!tip] Deployment Workflow
> The typical workflow is: upload code → create version → deploy version to staging environment → test → deploy same version to production environment. If something goes wrong in production, ==roll back== by deploying the previous version.

## Supported Platforms

Beanstalk supports a wide range of programming languages and platforms:

| Platform | Languages/Runtimes |
|----------|-------------------|
| **Web Servers** | Go, Java SE, Java with Tomcat, .NET on Windows, Node.js, PHP, Python, Ruby |
| **Docker** | Single Container Docker, Multi-Container Docker, Preconfigured Docker |
| **Custom** | If your platform isn't listed, you can create a ==custom platform== using Packer |

> [!note] Docker Support
> If Beanstalk doesn't natively support your language or runtime, you can ==always use Docker==. Package your application in a Docker container and deploy it to Beanstalk. This gives you complete control over the runtime environment while still getting Beanstalk's management benefits.

## Environment Tiers: Web Server vs Worker

Beanstalk offers two types of environments, each designed for a different workload pattern:

### Web Server Tier

```
┌──────────┐   ┌──────────┐   ┌─────────────────────────────────┐
│          │   │          │   │        Web Server Tier           │
│  Users   │──▶│   ELB    │──▶│                                 │
│ (HTTP)   │   │          │   │   ASG                           │
│          │   │          │   │   ┌──────┐ ┌──────┐ ┌──────┐  │
│          │   │          │   │   │ EC2  │ │ EC2  │ │ EC2  │  │
│          │   │          │   │   │      │ │      │ │      │  │
│          │   │          │   │   └──────┘ └──────┘ └──────┘  │
│          │   │          │   │                                 │
└──────────┘   └──────────┘   └─────────────────────────────────┘
```

- Handles ==HTTP/HTTPS requests== from users
- Uses an ==ELB== to distribute traffic across EC2 instances
- ==ASG== scales instances based on traffic
- This is the standard web application pattern we've been building throughout this section

### Worker Tier

```
┌──────────────────────────────────────────────────────────────┐
│                     Worker Tier                               │
│                                                               │
│   ┌─────────┐        ASG                                     │
│   │   SQS   │        ┌──────┐ ┌──────┐ ┌──────┐            │
│   │  Queue   │──────▶│ EC2  │ │ EC2  │ │ EC2  │            │
│   │         │        │      │ │      │ │      │            │
│   │ Messages│        └──────┘ └──────┘ └──────┘            │
│   └─────────┘        (pull messages and process them)        │
│                                                               │
│   No ELB — no direct user traffic                            │
│   Scales based on queue depth                                │
└──────────────────────────────────────────────────────────────┘
```

- Handles ==background processing== — no direct user traffic
- Uses an ==SQS queue== instead of an ELB
- EC2 instances ==pull messages== from the queue and process them
- ==ASG scales based on queue depth== — more messages = more instances
- Use cases: image processing, video encoding, report generation, email sending

> [!tip] Combining Tiers
> A common architecture is to have a ==Web Server tier== that accepts user requests and pushes work items to an ==SQS queue==, which is then processed by a ==Worker tier==. This decouples the user-facing application from the background processing, allowing each to scale independently.

```
┌──────┐   ┌─────┐   ┌──────────────┐   ┌─────────┐   ┌──────────────┐
│      │   │     │   │  Web Server  │   │   SQS   │   │   Worker     │
│ User │──▶│ ELB │──▶│  Tier        │──▶│  Queue  │──▶│   Tier       │
│      │   │     │   │  (accepts    │   │         │   │  (processes  │
│      │   │     │   │   request)   │   │         │   │   in bg)     │
└──────┘   └─────┘   └──────────────┘   └─────────┘   └──────────────┘
```

## Deployment Modes

Beanstalk offers two primary deployment modes that affect cost and availability:

### Single Instance Mode (Development)

```
┌─────────────────────────────────────────┐
│         Single Instance Mode            │
│                                         │
│   ┌──────────┐                          │
│   │ Elastic  │                          │
│   │ IP       │                          │
│   └────┬─────┘                          │
│        │                                │
│   ┌────▼─────┐   ┌──────────────┐      │
│   │   EC2    │   │     RDS      │      │
│   │ Instance │──▶│   (optional) │      │
│   └──────────┘   └──────────────┘      │
│                                         │
│   ✅ Cheapest option                    │
│   ✅ Good for dev/test                  │
│   ❌ No high availability              │
│   ❌ Single point of failure           │
└─────────────────────────────────────────┘
```

- ==One EC2 instance== with an Elastic IP (no ELB)
- Optionally an RDS instance
- ==Cheapest== option — great for development and testing
- ==No high availability== — if the instance fails, your app is down
- No load balancer means no health checks or traffic distribution

### High Availability Mode (Production)

```
┌─────────────────────────────────────────────────────────────────┐
│              High Availability Mode                             │
│                                                                 │
│   ┌──────────────────────────────────┐                          │
│   │         Multi-AZ ELB             │                          │
│   └──────────┬───────────┬───────────┘                          │
│              │           │                                      │
│   ┌──────────▼──┐  ┌────▼──────────┐                           │
│   │   AZ 1      │  │    AZ 2       │                           │
│   │             │  │               │                           │
│   │  ┌──────┐  │  │  ┌──────┐    │                           │
│   │  │ EC2  │  │  │  │ EC2  │    │                           │
│   │  └──────┘  │  │  └──────┘    │                           │
│   │  ┌──────┐  │  │  ┌──────┐    │                           │
│   │  │ EC2  │  │  │  │ EC2  │    │                           │
│   │  └──────┘  │  │  └──────┘    │                           │
│   └─────────────┘  └──────────────┘                           │
│                                                                 │
│   ┌──────────────────────────────────┐                          │
│   │     RDS Multi-AZ (optional)      │                          │
│   └──────────────────────────────────┘                          │
│                                                                 │
│   ✅ High availability (Multi-AZ)                               │
│   ✅ Load balanced                                              │
│   ✅ Auto scaling                                               │
│   ❌ More expensive                                             │
└─────────────────────────────────────────────────────────────────┘
```

- ==Multi-AZ ELB== distributing traffic across AZs
- ==ASG== with multiple EC2 instances across AZs
- Optionally ==RDS Multi-AZ== for database high availability
- ==Production-ready== — survives AZ failures
- More expensive but ==required for production workloads==

## Beanstalk Under the Hood: CloudFormation

Everything Beanstalk creates is managed through ==AWS CloudFormation==. When you create a Beanstalk environment, it generates a CloudFormation stack that defines all the resources. This means:

- You can see exactly what Beanstalk created in the CloudFormation console
- Resources are ==tagged and organized== as a stack
- ==Deleting the Beanstalk environment== deletes the CloudFormation stack and all its resources
- You get ==infrastructure as code== without writing any templates yourself

```
┌──────────────┐         ┌──────────────────┐         ┌──────────────┐
│  Beanstalk   │         │  CloudFormation   │         │  AWS         │
│  Console     │────────▶│  Stack            │────────▶│  Resources   │
│              │ creates │                  │ creates │              │
│  "Create     │         │  Template with:  │         │  EC2, ELB,   │
│   Environment│         │  - EC2 instances │         │  ASG, SG,    │
│   "          │         │  - ELB           │         │  RDS, etc.   │
│              │         │  - ASG           │         │              │
│              │         │  - Security Grps │         │              │
└──────────────┘         └──────────────────┘         └──────────────┘
```

> [!warning] Cleanup
> When you're done with a Beanstalk environment, ==always delete it through Beanstalk== (not by manually deleting individual resources). Beanstalk will clean up the CloudFormation stack and all associated resources. If you delete resources manually, you'll have orphaned resources and a broken CloudFormation stack.

## Key Takeaways for the Exam

| Concept | Detail |
|---------|--------|
| **What is Beanstalk?** | ==Developer-centric PaaS== — deploy code, Beanstalk manages infrastructure |
| **Cost** | ==Free== — you only pay for underlying resources (EC2, ELB, RDS, etc.) |
| **Components** | Application → Application Versions → Environments |
| **Tiers** | ==Web Server== (ELB + ASG) and ==Worker== (SQS + ASG) |
| **Deployment Modes** | ==Single Instance== (dev) and ==High Availability== (prod) |
| **Under the Hood** | Uses ==CloudFormation== to provision all resources |
| **Supported Platforms** | Most languages + Docker for anything else |
| **Full Control** | You still have ==full access== to underlying resources |

## Questions & Answers

> [!question]- Q1: What is AWS Elastic Beanstalk?
> **Answer:**
> Elastic Beanstalk is a ==developer-centric Platform as a Service (PaaS)== that simplifies deploying and managing applications on AWS. You upload your code, and Beanstalk automatically handles provisioning EC2 instances, load balancers, auto scaling groups, databases, and monitoring. It uses the ==same AWS services== you'd configure manually, but manages them for you.

> [!question]- Q2: Does Beanstalk cost extra?
> **Answer:**
> No. Beanstalk itself is ==completely free==. You only pay for the ==underlying AWS resources== it creates — EC2 instances, ELB, RDS, etc. There is no additional charge for the Beanstalk orchestration layer. This is a common exam question.

> [!question]- Q3: What are the three Beanstalk components?
> **Answer:**
> 1. ==Application== — the top-level container (your project)
> 2. ==Application Version== — an immutable snapshot of your code (v1.0, v1.1, v2.0)
> 3. ==Environment== — a running instance of your application with all infrastructure (dev, staging, prod)
> You deploy a specific version to a specific environment. Rollbacks are done by deploying a previous version.

> [!question]- Q4: What is the difference between Web Server and Worker tiers?
> **Answer:**
> - ==Web Server Tier==: Handles HTTP/HTTPS requests from users. Uses an ELB to distribute traffic to EC2 instances in an ASG. Standard web application pattern.
> - ==Worker Tier==: Handles background processing. Uses an SQS queue instead of an ELB. EC2 instances pull messages from the queue and process them. ASG scales based on queue depth.
> They can be combined: web tier accepts requests and pushes work to SQS, worker tier processes it.

> [!question]- Q5: What are the two deployment modes?
> **Answer:**
> - ==Single Instance==: One EC2 with Elastic IP, no ELB. Cheapest option, good for dev/test. No high availability.
> - ==High Availability==: Multi-AZ ELB + ASG with multiple instances. Optional RDS Multi-AZ. Required for production. More expensive but survives AZ failures.

> [!question]- Q6: What happens under the hood when you create a Beanstalk environment?
> **Answer:**
> Beanstalk creates a ==CloudFormation stack== that defines all the resources — EC2 instances, ELB, ASG, security groups, RDS, etc. CloudFormation then provisions those resources. You can see the stack in the CloudFormation console. When you delete the Beanstalk environment, it deletes the CloudFormation stack and ==all associated resources==.

> [!question]- Q7: What if Beanstalk doesn't support my programming language?
> **Answer:**
> You have two options:
> 1. ==Docker== — package your application in a Docker container and deploy it to Beanstalk. This works for any language or runtime.
> 2. ==Custom Platform== — create a custom Beanstalk platform using Packer. This is more advanced but gives you complete control over the runtime environment.
> Docker is the easier and more common approach.

> [!question]- Q8: How does Beanstalk relate to the architectures we built manually in this section?
> **Answer:**
> Beanstalk automates exactly what we built manually. The [[01-WhatsTheTime|WhatsTheTime.com]] architecture (Route 53 + ELB + ASG) is essentially what Beanstalk's ==High Availability Web Server tier== creates. The difference is that Beanstalk ==manages it for you== — deployments, scaling, monitoring, and health checks are all handled automatically.

> [!question]- Q9: Can you still access the underlying resources in Beanstalk?
> **Answer:**
> Yes. Beanstalk gives you ==full access== to all underlying resources. You can SSH into EC2 instances, modify security groups, configure the ELB, adjust ASG settings, etc. Beanstalk is an ==orchestration layer==, not a black box. This is different from fully managed services like Lambda where you have no access to the underlying infrastructure.

> [!question]- Q10: When would you choose Beanstalk over manual architecture?
> **Answer:**
> Choose Beanstalk when:
> - You want to ==focus on code, not infrastructure==
> - Your application fits standard patterns (web server, worker, Docker)
> - You want ==managed deployments== with rollback capability
> - You don't need highly customized infrastructure
>
> Choose manual architecture when:
> - You need ==fine-grained control== over every component
> - Your architecture doesn't fit Beanstalk's patterns
> - You have a dedicated DevOps/infrastructure team
> - You need advanced networking configurations
