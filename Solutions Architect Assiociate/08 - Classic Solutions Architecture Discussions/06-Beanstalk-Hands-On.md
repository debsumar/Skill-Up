---
title: Elastic Beanstalk Hands On
date: 2026-02-10
tags:
  - aws
  - elastic-beanstalk
  - hands-on
  - cloudformation
  - saa-c03
---

# Elastic Beanstalk Hands On

## Overview

Walkthrough of creating a Beanstalk environment via the AWS Console. Demonstrates how Beanstalk provisions infrastructure (EC2, ASG, Elastic IP, Security Groups) using ==CloudFormation== behind the scenes.

## Steps to Create a Beanstalk Environment

### 1. Create Application

| Setting | Value |
|---------|-------|
| **Environment type** | Web server environment |
| **Application name** | MyApplication |
| **Environment name** | MyApplication-dev |
| **Domain** | Auto-generated |

### 2. Choose Platform

| Setting | Value |
|---------|-------|
| **Platform** | Managed |
| **Runtime** | Node.js (or any supported language) |
| **Code** | Sample application |
| **Preset** | ==Single instance== (free tier eligible) |

### 3. Configure Service Access (IAM)

| Role | Purpose |
|------|---------|
| **Service role** | `elasticbeanstalk-service-role` (auto-created) |
| **EC2 Instance profile** | `aws-elasticbeanstalk-ec2-role` (==must create manually==) |

#### Creating the EC2 Instance Profile

1. Go to IAM Console → Roles → Create Role
2. Trusted entity: ==EC2==
3. Attach policies:
   - `AWSElasticBeanstalkWebTier`
   - `AWSElasticBeanstalkWorkerTier`
   - `AWSElasticBeanstalkMulticontainerDocker`
4. Role name: `aws-elasticbeanstalk-ec2-role`

### 4. Skip to Review & Submit

Optional steps (Networking, Database, Scaling) can be skipped for a basic single-instance setup.

## What Beanstalk Creates (via CloudFormation)

```
Beanstalk Environment
├── CloudFormation Stack
│   ├── Auto Scaling Group (min: 1, max: 1)
│   ├── EC2 Instance (t3.micro)
│   ├── Elastic IP
│   ├── Security Group
│   ├── Launch Configuration
│   └── Wait Condition
```

## Verifying Resources

| AWS Console | What to Check |
|-------------|---------------|
| **EC2 → Instances** | Running t3.micro with public IP |
| **EC2 → Elastic IPs** | Allocated and associated to instance |
| **EC2 → Auto Scaling Groups** | ASG managing the single instance |
| **CloudFormation → Stacks** | Stack with all resources listed |
| **CloudFormation → Template** | View in Application Composer for visual diagram |

## Beanstalk Console Features

| Tab | Purpose |
|-----|---------|
| **Upload & Deploy** | Upload new application version |
| **Health** | Instance health checks |
| **Logs** | Application logs |
| **Monitoring** | Metrics and dashboards |
| **Alarms** | CloudWatch alarms |
| **Managed Updates** | Platform update management |
| **Configuration** | All environment settings |

## Beanstalk vs CloudFormation

```
Beanstalk                          CloudFormation
┌─────────────────────┐            ┌─────────────────────┐
│ Application-centric │            │ Infrastructure-     │
│ Code + Environments │            │ centric             │
│ Managed deployment  │            │ Any AWS resource    │
│ Uses CloudFormation │──creates──▶│ Stacks & Templates  │
│ internally          │            │                     │
└─────────────────────┘            └─────────────────────┘
```

## Cleanup

> [!warning] Important
> If you're done with Beanstalk lectures, delete the application:
> **Application → Actions → Delete application**
>
> This will delete all underlying resources (EC2, ASG, Elastic IP, Security Groups) via CloudFormation stack deletion.

## Questions & Answers

> [!question]- Q1: What IAM roles are needed for Beanstalk?
> **Answer:**
> 1. ==Service role== (`elasticbeanstalk-service-role`) — allows Beanstalk to manage AWS resources
> 2. ==EC2 Instance profile== (`aws-elasticbeanstalk-ec2-role`) — allows EC2 instances to access AWS services

> [!question]- Q2: What policies should the EC2 instance profile have?
> **Answer:**
> - `AWSElasticBeanstalkWebTier`
> - `AWSElasticBeanstalkWorkerTier`
> - `AWSElasticBeanstalkMulticontainerDocker`

> [!question]- Q3: What does Beanstalk create for a single-instance environment?
> **Answer:**
> Via CloudFormation:
> - 1 EC2 instance (t3.micro)
> - 1 Elastic IP
> - 1 Auto Scaling Group (min/max: 1)
> - 1 Security Group
> - Launch Configuration

> [!question]- Q4: How does Beanstalk use CloudFormation?
> **Answer:**
> Beanstalk creates a ==CloudFormation stack== that defines all the infrastructure. You can view the stack in the CloudFormation console, see events, resources, and even visualize the template in Application Composer.

> [!question]- Q5: What is the difference between single-instance and high-availability presets?
> **Answer:**
> - ==Single instance==: 1 EC2 + Elastic IP, no ELB — free tier eligible, for development
> - ==High availability==: ELB + ASG (Multi-AZ) + optional RDS Multi-AZ — for production

> [!question]- Q6: How do you deploy a new version of your application?
> **Answer:**
> In the Beanstalk console, click ==Upload and Deploy==, upload your new code package, and Beanstalk will automatically deploy it to the running EC2 instances.

> [!question]- Q7: Can you have multiple environments for one application?
> **Answer:**
> Yes. For example, `MyApplication-dev` and `MyApplication-prod` — each running different versions of the code with different configurations.

> [!question]- Q8: What happens when you delete a Beanstalk application?
> **Answer:**
> Beanstalk deletes the ==CloudFormation stack==, which in turn deletes all provisioned resources: EC2 instances, ASG, Elastic IPs, Security Groups, etc.

> [!question]- Q9: Where can you see what resources Beanstalk created?
> **Answer:**
> - ==CloudFormation Console== → Stack → Resources tab
> - ==CloudFormation Console== → Stack → Template → View in Application Composer (visual diagram)
> - Individual service consoles (EC2, ASG, etc.)

> [!question]- Q10: Why is the single-instance preset recommended for development?
> **Answer:**
> - ==Free tier eligible== (t3.micro)
> - Simpler architecture (no ELB)
> - Lower cost
> - Sufficient for development and testing
> - Can upgrade to high-availability preset for production later
