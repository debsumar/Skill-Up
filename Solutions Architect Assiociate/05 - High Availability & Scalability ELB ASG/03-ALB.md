---
title: Application Load Balancer (ALB)
date: 2026-02-09
tags:
  - aws
  - elb
  - alb
  - layer-7
  - saa-c03
---

# Application Load Balancer (ALB)

## Overview

==Layer 7 (HTTP)== load balancer with intelligent routing to multiple target groups.

## Key Features

| Feature | Details |
|---------|---------|
| **Layer** | 7 (HTTP/HTTPS/WebSocket) |
| **Protocols** | HTTP, HTTPS, ==HTTP/2==, WebSocket |
| **Routing** | By ==path, hostname, query string, headers== |
| **Target Groups** | EC2, ECS tasks, Lambda, IP addresses |
| **Fixed hostname** | `xxx.region.elb.amazonaws.com` |
| **Client IP** | Via ==X-Forwarded-For== header |
| **Redirect** | HTTP → HTTPS at LB level |
| **Port mapping** | ==Dynamic port mapping== for ECS containers |
| **Best for** | Microservices, containers (ECS) |

## Routing Rules

```
                    ┌──────────────────┐
                    │       ALB        │
                    └──┬──────────┬────┘
                       │          │
            /users     │          │  /search
                       ▼          ▼
              ┌──────────┐  ┌──────────┐
              │ Target   │  │ Target   │
              │ Group 1  │  │ Group 2  │
              │ (Users)  │  │ (Search) │
              └──────────┘  └──────────┘
```

| Routing Type | Example |
|-------------|---------|
| **URL Path** | `/users` → TG1, `/search` → TG2 |
| **Hostname** | `one.example.com` → TG1, `other.example.com` → TG2 |
| **Query String** | `?platform=mobile` → TG1, `?platform=desktop` → TG2 |

## Target Group Types

| Target | Description |
|--------|-------------|
| **EC2 Instances** | Managed by ASG |
| **ECS Tasks** | Container-based (dynamic port mapping) |
| **Lambda Functions** | Serverless |
| **IP Addresses** | Must be ==private IPs== (on-premises servers) |

## Listener Rules

ALB listeners evaluate rules in ==priority order== (1 = highest, up to 50,000) plus a default rule.

| Rule Component | Options |
|---------------|---------|
| **Conditions** | Path, hostname, HTTP method, source IP, query string, headers |
| **Actions** | ==Forward== to target group, ==Redirect== to URL, ==Fixed response== (e.g., 404) |
| **Priority** | 1 (highest) → 50,000 (lowest), then default |

> [!note] One ALB vs Multiple CLBs
> With CLB, you need ==one CLB per application==. With ALB, ==one ALB can route to many target groups== using rules — much more cost-effective.

## Client IP: X-Forwarded-For

```
Client (12.34.56.78)
    │
    ▼ HTTPS
┌─────────┐  Connection termination
│   ALB   │  Uses LB private IP to talk to EC2
└────┬────┘
     │ HTTP (private IP)
     ▼
┌─────────┐
│   EC2   │  Sees LB IP, NOT client IP
│         │  Client IP in: X-Forwarded-For header
│         │  Port in: X-Forwarded-Port
│         │  Proto in: X-Forwarded-Proto
└─────────┘
```

## Questions & Answers

> [!question]- Q1: What layer does ALB operate on?
> **Answer:**
> ==Layer 7 (HTTP)==. It understands HTTP/HTTPS/WebSocket and can route based on URL path, hostname, query strings, and headers.

> [!question]- Q2: What are ALB target groups?
> **Answer:**
> Groups of targets that receive traffic: ==EC2 instances, ECS tasks, Lambda functions, or private IP addresses==. Health checks are done at the target group level.

> [!question]- Q3: How does ALB route to different microservices?
> **Answer:**
> Using ==routing rules== based on URL path (`/users` → TG1, `/search` → TG2), hostname, or query strings. One ALB can front multiple applications.

> [!question]- Q4: How does an EC2 instance get the client's real IP behind an ALB?
> **Answer:**
> From the ==X-Forwarded-For== HTTP header. The ALB performs connection termination and uses its own private IP to talk to EC2, so the client IP is only in the header.

> [!question]- Q5: Can ALB route to on-premises servers?
> **Answer:**
> ==Yes==. Register their ==private IP addresses== as targets in a target group. This works via VPN or Direct Connect.

> [!question]- Q6: Why is ALB preferred for microservices over CLB?
> **Answer:**
> One ALB can route to ==multiple target groups== using rules. With CLB, you'd need ==one CLB per application==. ALB also supports dynamic port mapping for containers.

> [!question]- Q7: Does ALB support WebSocket?
> **Answer:**
> ==Yes==. ALB natively supports HTTP, HTTPS, and ==WebSocket== protocols.

> [!question]- Q8: Can ALB redirect HTTP to HTTPS?
> **Answer:**
> ==Yes==. You can configure a redirect rule at the ALB listener level to automatically redirect HTTP (port 80) to HTTPS (port 443).

> [!question]- Q9: What is the ALB hostname format?
> **Answer:**
> ALB provides a ==fixed hostname== like `xxx.region.elb.amazonaws.com`. You cannot get a static IP from ALB (use NLB for that).

> [!question]- Q10: Can ALB front Lambda functions?
> **Answer:**
> ==Yes==. ALB can have Lambda functions as targets. The HTTP request is translated into a JSON event for Lambda. This is useful for serverless web applications.
