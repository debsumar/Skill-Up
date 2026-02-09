---
title: "WhatsTheTime.com - Stateless Web App"
date: 2026-02-10
tags:
  - aws
  - solutions-architecture
  - ec2
  - elb
  - asg
  - route53
  - saa-c03
---

# WhatsTheTime.com - Stateless Web App

## Overview

This is the first solution architecture case study, and it's designed to show you the ==iterative journey== of a Solutions Architect. We start with the simplest possible application and progressively improve it, adding AWS services one by one to solve real problems.

WhatIsTheTime.com is a ==stateless== web application — it simply tells users what time it is. It sounds trivial, but that's the point: the app is so simple that everyone understands it, and we can focus entirely on the architecture decisions rather than the application logic.

**Key constraints:**
- No database needed — each server knows the time
- Start small, willing to accept some downtime initially
- App gets more and more popular over time
- Need to scale vertically, then horizontally, then remove downtime entirely

This is ==pure solution architecture== — the kind of thinking the exam tests you on.

## Architecture Evolution

### Step 1: Single EC2 Instance + Elastic IP

```
┌──────────┐                    ┌─────────────────────┐
│   User   │───── "What time   │  T2 Micro Instance  │
│          │      is it?" ────▶│  + Elastic IP       │
│          │◀──── "5:30 PM" ───│  (Public)           │
└──────────┘                    └─────────────────────┘
```

This is our first proof of concept. We have a single public EC2 instance — a tiny T2 Micro — and we attach an ==Elastic IP== to it so it has a static IP address. If something happens and we need to restart the instance, the IP stays the same.

**What works:** Users can access the app. We're getting great feedback.

**What doesn't work:** There's zero redundancy. If the instance goes down, the entire application is down. But for a PoC, this is acceptable.

### Step 2: Vertical Scaling

```
┌──────────┐                    ┌─────────────────────┐
│   User   │──────────────────▶│  M5 Large Instance  │
│          │                    │  + Elastic IP       │
│          │                    │  (Public)           │
└──────────┘                    └─────────────────────┘
```

The app is getting popular. More and more users are asking what time it is, and the T2 Micro can't handle the load. So as a Solutions Architect, we decide to ==vertically scale== — replace the T2 Micro with a bigger M5 Large instance.

The process: stop the instance → change the instance type → start it again. Because we have an Elastic IP, the IP address stays the same, so users can still find us.

**The problem:** During the upgrade, the instance is ==stopped==. Users experience ==downtime== and they're not happy. They can't access the application while we're upgrading. This works, but it's not great.

### Step 3: Horizontal Scaling + Elastic IPs

```
┌──────────┐    ┌─────────────────────┐
│          │───▶│ M5 Large + EIP 1    │
│  Users   │───▶│ M5 Large + EIP 2    │
│          │───▶│ M5 Large + EIP 3    │
└──────────┘    └─────────────────────┘
```

Now we're really popular, so it's time to ==scale horizontally== — add more instances instead of making one bigger. We launch three M5 Large instances, each with its own Elastic IP.

**The problems are piling up:**
- Users need to know ==all three IP addresses== to talk to our instances — that's terrible UX
- We're managing three Elastic IPs, and AWS only gives you ==5 Elastic IPs per region by default==
- Adding or removing instances means users need to update their IP lists
- This approach simply doesn't scale

### Step 4: Route 53 DNS (Remove Elastic IPs)

```
┌──────────┐    ┌──────────────────┐    ┌─────────────────────┐
│          │    │    Route 53      │    │ M5 Large (x3)       │
│  Users   │───▶│ api.whatisthe    │───▶│ Public instances     │
│          │    │ time.com         │    │ (no Elastic IPs)     │
│          │    │ A Record         │    │                     │
│          │    │ TTL: 1 hour      │    └─────────────────────┘
└──────────┘    └──────────────────┘
```

Let's get rid of Elastic IPs entirely. Instead, we set up ==Route 53== with an A Record for `api.whatisthetime.com`. An A Record returns a list of IP addresses, so Route 53 gives users the IPs of our EC2 instances. We can update Route 53 whenever we add or remove instances.

**What works:** No more Elastic IPs to manage. Users just remember one domain name.

**What breaks:** We set the TTL to 1 hour. Now imagine we remove one instance — the users who cached that DNS response will ==keep trying to connect to the removed instance for up to 1 hour==. They think our app is down, and that's really bad.

> [!warning] Route 53 TTL Problem
> With a 1-hour TTL, clients cache the DNS response and keep using stale IPs. If you remove an instance, some users will experience failures until their cached DNS entry expires. This is a fundamental limitation of using A Records with direct instance IPs.

### Step 5: ELB + Private Instances

```
┌──────────┐   ┌──────────────┐   ┌──────────────┐   ┌─────────────────────┐
│          │   │  Route 53    │   │              │   │  Private EC2 (x3)   │
│  Users   │──▶│  Alias Record│──▶│     ELB      │──▶│  Same AZ            │
│          │   │              │   │ Health Checks│   │  SG: allow from ELB │
└──────────┘   └──────────────┘   └──────────────┘   └─────────────────────┘
```

This is a major architectural shift. Instead of public instances, we now have ==private EC2 instances== behind a ==Load Balancer==. The ELB is the only public-facing component.

**Critical DNS change:** We can't use an A Record anymore because the ELB's IP addresses ==change constantly==. Instead, we use an ==Alias Record== in Route 53, which points to the ELB's DNS name and automatically resolves to whatever IPs the ELB currently has.

**What we gain:**
- ==Health checks== — if an instance is down or not responding, the ELB stops sending traffic to it. Users never see a failure.
- ==Security group rules== — EC2 instances only accept traffic from the ELB's security group. They're not directly accessible from the internet.
- ==No downtime== when adding or removing instances — just register/deregister them with the ELB
- No more TTL problems — the ELB handles routing

> [!tip] Alias Record vs A Record
> An A Record maps a domain to ==static IP addresses==. An Alias Record maps a domain to ==another AWS resource== (like an ELB). Since ELB IPs change constantly, you ==must use an Alias Record==. Alias queries to AWS resources are also ==free==.

### Step 6: Auto Scaling Group

```
┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────────────────────┐
│          │   │ Route 53 │   │          │   │         ASG              │
│  Users   │──▶│  Alias   │──▶│   ELB    │──▶│  ┌─────┐ ┌─────┐      │
│          │   │          │   │          │   │  │ M5  │ │ M5  │ ...  │
└──────────┘   └──────────┘   └──────────┘   │  └─────┘ └─────┘      │
                                              │  Scale in/out on demand│
                                              │  (Single AZ)          │
                                              └──────────────────────────┘
```

Adding and removing instances manually is tedious. So we wrap our instances in an ==Auto Scaling Group==. The ASG automatically scales based on demand — maybe in the morning nobody cares about the time, but in the evening when people want to leave work, traffic spikes.

**What we gain:**
- ==Automatic scaling== — instances are added when demand increases, removed when it decreases
- No manual intervention needed
- Cost-efficient — we only pay for what we need at any given moment

**What's still wrong:** Everything is in a ==single Availability Zone==. If that AZ goes down (earthquake, power failure, network issue), our entire application is down.

### Step 7: Multi-AZ (The Final Architecture)

```
┌──────────┐   ┌──────────┐   ┌───────────────────┐   ┌──────────────────────────────┐
│          │   │ Route 53 │   │    Multi-AZ ELB   │   │           ASG                │
│  Users   │──▶│  Alias   │──▶│  AZ1, AZ2, AZ3   │──▶│  AZ1: M5 x2                 │
│          │   │          │   │  + Health Checks   │   │  AZ2: M5 x2                 │
└──────────┘   └──────────┘   └───────────────────┘   │  AZ3: M5 x1                 │
                                                        └──────────────────────────────┘
```

Now we make everything ==Multi-AZ==. The ELB spans three Availability Zones (AZ1, AZ2, AZ3) with health checks. The ASG also spans all three AZs, distributing instances across them.

**What happens when AZ1 goes down?** The ELB detects that instances in AZ1 are unhealthy and stops routing traffic to them. AZ2 and AZ3 continue serving users. The ASG may even launch replacement instances in the healthy AZs. Our users ==never notice the outage==.

This is what it means to be ==highly available and resilient to failure==.

### Step 8: Cost Optimization

Now that we have a stable, highly available architecture, let's think about cost. We know that at minimum, we always want at least ==two instances running== (one per AZ for redundancy). These instances run 24/7, 365 days a year.

| Instance Strategy | When to Use | Cost |
|-------------------|-------------|------|
| **Reserved Instances** | For the ==minimum capacity== that always runs (e.g., 2 instances) | ==Up to 72% savings== vs On-Demand |
| **On-Demand** | For temporary scaling — instances that come and go based on demand | Standard pricing |
| **Spot Instances** | If you're aggressive about cost savings and can tolerate instances being ==terminated== | ==Up to 90% savings== but risky |

> [!tip] Cost Strategy
> Reserve the minimum capacity of your ASG (the instances that will always be running). Use On-Demand for the scaling portion. Spot is optional and risky — only use if your application can handle sudden instance termination.

## What We Learned — Solutions Architect Perspective

This case study walked through the ==complete journey== of a Solutions Architect. Here's every concept we touched and why it matters:

| Concept | What We Learned | Why It Matters |
|---------|-----------------|----------------|
| **Public vs Private IP** | Public instances are directly accessible; private instances sit behind a load balancer | Security — private instances reduce attack surface |
| **Elastic IP** | Gives a static IP to an instance, but limited to 5 per region | Doesn't scale — use Route 53 or ELB instead |
| **Route 53 A Record** | Maps domain to IP addresses, but TTL causes stale routing | Not suitable for dynamic infrastructure |
| **Route 53 Alias Record** | Maps domain to AWS resources (ELB), auto-updates, ==free== | The correct DNS approach for ELB-backed apps |
| **ELB Health Checks** | Only healthy instances receive traffic | Prevents users from hitting dead instances |
| **Security Group References** | EC2 SG allows traffic only from ELB SG | Defense in depth — instances not directly accessible |
| **Auto Scaling Group** | Automatically adds/removes instances based on demand | Cost optimization + handles traffic spikes |
| **Multi-AZ** | Distribute across multiple AZs for fault tolerance | Survive AZ failures — ==high availability== |
| **Reserved Instances** | Pre-pay for always-on capacity | Significant cost savings for predictable workloads |

> [!important] The Exam Tests This Thinking
> The exam doesn't just ask "what is an ELB?" — it gives you a scenario and asks what architecture you'd recommend. This iterative thinking process (identify the problem → choose the right service → understand the trade-offs) is exactly what you need.

## Questions & Answers

> [!question]- Q1: Why can't you use an A Record for an ELB?
> **Answer:**
> An ELB's IP addresses ==change constantly== as AWS scales the load balancer behind the scenes. A Records map a domain to ==static IP addresses==. If the ELB's IPs change, the A Record becomes stale. Instead, use an ==Alias Record==, which points to the ELB's DNS name and automatically resolves to the current IPs. Alias queries to AWS resources are also free.

> [!question]- Q2: What problem does Route 53 TTL cause when scaling down?
> **Answer:**
> When you use A Records with a TTL of 1 hour, clients cache the DNS response for that duration. If you remove an instance, clients with cached responses will ==keep trying to connect to the old IP for up to 1 hour==. During that time, those users think the application is down. This is why direct A Records to instance IPs don't work well for dynamic infrastructure — you need an ELB in front.

> [!question]- Q3: Why use an ELB instead of multiple Elastic IPs?
> **Answer:**
> Multiple reasons:
> - Elastic IPs are ==limited to 5 per region== by default — doesn't scale
> - Users would need to know ==all the IPs== — terrible user experience
> - No ==health checks== — if an instance dies, users still try to connect to it
> - No ==automatic traffic distribution== — you'd need client-side load balancing
> - ELB solves all of these: single endpoint, health checks, automatic distribution, no IP limits

> [!question]- Q4: What is vertical scaling and what is its main drawback?
> **Answer:**
> Vertical scaling means replacing your instance with a ==bigger one== (e.g., T2 Micro → M5 Large). You get more CPU, memory, and network capacity. The main drawback is ==downtime== — you must stop the instance, change the instance type, and restart it. During this time, users can't access the application. There's also a hard ceiling — you can only scale up to the largest available instance type.

> [!question]- Q5: How does the security group rule work between ELB and EC2?
> **Answer:**
> The EC2 security group is configured to allow inbound traffic ==only from the ELB's security group== (using security group referencing, not IP ranges). This means:
> - EC2 instances are ==not directly accessible== from the internet
> - All traffic ==must flow through the ELB==
> - Even if someone discovers an instance's private IP, they can't connect to it directly
> - This is a key security best practice called ==defense in depth==

> [!question]- Q6: Why is Multi-AZ critical for this architecture?
> **Answer:**
> Without Multi-AZ, all your instances are in a single Availability Zone. If that AZ experiences an outage (earthquake, power failure, network issue, hardware failure), ==your entire application goes down==. With Multi-AZ, instances are distributed across 2-3 AZs. If one AZ fails, the ELB routes traffic to healthy instances in the remaining AZs. Users never notice the outage. This is what ==high availability== means.

> [!question]- Q7: How does an ASG improve over manually managed instances?
> **Answer:**
> Without an ASG, you manually launch and terminate instances based on traffic. This is:
> - ==Slow== — by the time you notice high traffic, users are already affected
> - ==Error-prone== — you might forget to scale down and waste money
> - ==Tedious== — requires constant monitoring
>
> An ASG automates all of this: it monitors metrics (CPU, request count), ==automatically adds instances== when demand increases, and ==removes them== when demand decreases. You define min/max/desired capacity and the ASG handles the rest.

> [!question]- Q8: What is the cost optimization strategy for the minimum ASG capacity?
> **Answer:**
> If your ASG always has at least 2 instances running (for Multi-AZ redundancy), those 2 instances run 24/7 for the entire year. Use ==Reserved Instances== for these — you commit to 1 or 3 years and save up to 72% compared to On-Demand pricing. For the additional instances that scale up and down, use ==On-Demand== (or Spot if you can tolerate termination). This gives you the best balance of cost and reliability.

> [!question]- Q9: What is the correct DNS setup for an ELB-backed application?
> **Answer:**
> Route 53 → ==Alias Record== → ELB DNS name. The Alias Record:
> - Automatically resolves to the ELB's current IP addresses
> - Updates when ELB IPs change (no stale routing)
> - Is ==free== for AWS resources (no charge per query)
> - Works with the zone apex (e.g., `example.com`, not just `www.example.com`)

> [!question]- Q10: Summarize the full architecture evolution and what each step solved.
> **Answer:**
> 1. **Single EC2 + Elastic IP** → Basic PoC, works but no redundancy
> 2. **Vertical scaling** → More capacity, but causes ==downtime==
> 3. **Horizontal scaling + Elastic IPs** → More instances, but users must know all IPs and ==5 EIP limit==
> 4. **Route 53 A Record** → Single domain name, but ==TTL causes stale routing==
> 5. **ELB + Alias Record** → Health checks, no TTL issues, ==security group isolation==
> 6. **ASG** → Automatic scaling, no manual management, ==cost-efficient==
> 7. **Multi-AZ** → Survive AZ failures, ==high availability==
> 8. **Reserved Instances** → ==Cost optimization== for always-on capacity
>
> Each step solved a specific problem while introducing the next challenge. This is how real architecture evolves.
