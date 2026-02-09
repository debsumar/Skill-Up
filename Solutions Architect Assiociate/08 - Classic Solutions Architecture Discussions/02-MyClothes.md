---
title: "MyClothes.com - Stateful Web App"
date: 2026-02-10
tags:
  - aws
  - solutions-architecture
  - elb
  - elasticache
  - rds
  - saa-c03
---

# MyClothes.com - Stateful Web App

## Overview

In the previous case study (WhatIsTheTime.com), we built a ==stateless== application — the server didn't need to remember anything between requests. Now we're stepping into a much more realistic and challenging scenario: a ==stateful== e-commerce web application.

MyClothes.com allows people to buy clothes online. It has a ==shopping cart==, and hundreds of users are navigating the website simultaneously. The fundamental challenge is: how do we maintain ==horizontal scalability== (many instances behind a load balancer) while preserving user state (the shopping cart)?

**Key requirements:**
- Users should ==never lose their shopping cart== while navigating
- The web tier should remain as ==stateless as possible== for easy scaling
- User details (address, name, etc.) should be stored in a durable database
- The architecture must scale reads efficiently
- Everything should survive AZ failures

## Starting Point: The Problem

We start with the same architecture from WhatIsTheTime.com — Route 53, Multi-AZ ELB, Auto Scaling Group across three AZs. This worked great for a stateless app, but now we have a problem.

```
┌──────────┐    ┌─────┐    ┌────────────────────────────────┐
│          │    │     │    │  Instance A  ← Cart created    │
│   User   │─1─▶│ ELB │───▶│                                │
│          │─2─▶│     │───▶│  Instance B  ← Cart LOST!      │
│          │─3─▶│     │───▶│  Instance C  ← Cart LOST!      │
└──────────┘    └─────┘    └────────────────────────────────┘
```

The user adds something to their shopping cart on Instance A. The next request gets routed to Instance B — which has ==no knowledge of the cart==. The user adds something again, gets routed to Instance C — cart lost again. The user is going crazy: "I keep losing my shopping cart! MyClothes.com is a terrible website!" And we're losing money.

This happens because the ELB distributes requests across instances, and each instance stores the cart in ==local memory==. There's no shared state.

## Solution 1: ELB Stickiness (Session Affinity)

The simplest fix: enable ==ELB Stickiness== (also called session affinity). This tells the ELB to route all requests from the same user to the ==same instance==.

```
┌──────────┐    ┌──────────────────┐    ┌────────────────────────────────┐
│          │    │       ELB        │    │                                │
│   User   │─1─▶│   Stickiness    │───▶│  Instance A  ← All requests   │
│          │─2─▶│    ENABLED      │───▶│  Instance A  ← go to same     │
│          │─3─▶│                  │───▶│  Instance A  ← instance       │
└──────────┘    └──────────────────┘    └────────────────────────────────┘
```

The ELB sets a cookie on the user's browser, and every subsequent request from that user goes to the same instance. The shopping cart is preserved!

| Pros | Cons |
|------|------|
| Very simple to enable — just a checkbox | If the EC2 instance ==terminates==, the cart is ==lost forever== |
| No application code changes needed | Can cause ==uneven load distribution== (some instances get more traffic) |
| Works immediately | Not truly stateless — we're just hiding the problem |

This is a definite improvement, but it's fragile. If the ASG terminates an instance (during scale-in, or if it fails a health check), all users stuck to that instance lose their carts.

## Solution 2: User Cookies (Client-Side State)

A completely different approach: instead of storing the cart on the server, let the ==user store it==. The shopping cart content is sent as a ==web cookie== with every HTTP request.

```
┌──────────────────────────┐    ┌─────┐    ┌────────────────────────────┐
│   User                   │    │     │    │  Instance A                │
│   Cookie: cart=          │───▶│ ELB │───▶│  Reads cart from cookie    │
│   {item1, item2, item3}  │───▶│     │───▶│  Instance B                │
│                          │───▶│     │───▶│  Reads cart from cookie    │
│                          │    │     │    │  Instance C                │
└──────────────────────────┘    └─────┘    │  Reads cart from cookie    │
                                           └────────────────────────────┘
```

Every time the user makes a request, they send the entire shopping cart content in the cookie. Any instance can read it — we've achieved ==true statelessness== in the web tier. Each EC2 instance doesn't need to know what happened before; the user tells us.

| Pros | Cons |
|------|------|
| ==True statelessness== — any instance can serve any user | HTTP requests get ==heavier== (more data sent with every request) |
| No server-side session storage needed | ==Security risk== — cookies can be ==altered by attackers== |
| Survives instance termination | Max cookie size is only ==4 KB== — can't store large carts |

> [!danger] Security Warning
> If you use this pattern, your EC2 instances ==must validate the cookie content==. An attacker could modify the cookie to change prices, add items, or manipulate the cart. Never trust client-side data without server-side validation.

This is actually a pattern that many web application frameworks use in practice, but the 4 KB limit and security concerns make it impractical for complex shopping carts.

## Solution 3: Server Sessions with ElastiCache (Best Approach)

The best of both worlds: the user sends only a ==session ID== (a small token) in the cookie, and the actual cart data is stored server-side in ==ElastiCache==.

```
┌──────────────────────┐    ┌─────┐    ┌──────────┐    ┌──────────────────┐
│   User               │    │     │    │          │    │                  │
│   Cookie:            │───▶│ ELB │───▶│ Inst. A  │───▶│   ElastiCache    │
│   session_id=abc123  │    │     │    │          │    │                  │
│                      │    │     │    │ Inst. B  │───▶│   session_id →   │
│                      │    │     │    │          │    │   cart data      │
│                      │    │     │    │ Inst. C  │───▶│                  │
└──────────────────────┘    └─────┘    └──────────┘    │   Sub-ms perf   │
                                                        └──────────────────┘
```

Here's how it works:
1. User makes first request → EC2 instance creates a session, stores cart data in ElastiCache with a session ID as the key
2. EC2 instance sends the session ID back to the user as a cookie
3. User makes second request (to a ==different instance==) → sends session ID in cookie
4. The new instance uses the session ID to ==look up the cart from ElastiCache==
5. Cart data is retrieved in ==sub-millisecond time== — the user never notices

| Pros | Cons |
|------|------|
| ==Secure== — cart data is server-side, attackers can't modify it | Additional infrastructure cost (ElastiCache cluster) |
| ==Sub-millisecond performance== — ElastiCache is incredibly fast | Requires cache maintenance logic in application code |
| Lightweight cookies — only a session ID is sent | Need to handle cache expiry and invalidation |
| Survives instance termination — data is in ElastiCache | |
| ==True statelessness== in the web tier | |

> [!tip] DynamoDB Alternative
> ==DynamoDB== can also be used for session storage instead of ElastiCache. DynamoDB is fully managed, serverless, and has built-in TTL for automatic session expiry. Choose DynamoDB when you want zero cluster management; choose ElastiCache when you need the absolute lowest latency.

This is the ==recommended pattern== for stateful web applications. It's more secure than cookies, more resilient than stickiness, and keeps the web tier stateless.

## Adding a Database Layer: RDS for User Data

Now that we've solved the shopping cart problem, we need to store ==durable user data== — things like shipping addresses, names, order history. This data needs to persist long-term, not just for a session.

```
┌──────────────┐    ┌──────────────────────┐
│  Instance A  │───▶│                      │
│  Instance B  │───▶│    RDS (MySQL)       │
│  Instance C  │───▶│                      │
└──────────────┘    │  User addresses      │
                    │  Names, profiles     │
                    │  Order history       │
                    └──────────────────────┘
```

All EC2 instances can read from and write to the same RDS database. This gives us a ==centralized, durable data store== that persists regardless of which instance handles the request.

## Scaling Reads: Two Approaches

Our website is doing amazing — more and more users are browsing products, reading descriptions, viewing images. Most of the traffic is ==reads== (browsing), not writes (purchasing). How do we scale the read capacity?

### Approach A: RDS Read Replicas

```
┌──────────────┐         ┌──────────────────┐
│              │──WRITE──▶│   RDS Master     │
│  EC2         │         └────────┬─────────┘
│  Instances   │                  │ Replication
│              │──READ───▶┌──────▼──────────┐
│              │──READ───▶│  Read Replica 1 │
│              │──READ───▶│  Read Replica 2 │
│              │          │  ...up to 15    │
└──────────────┘          └─────────────────┘
```

Writes go to the RDS Master. Reads go to ==Read Replicas== (up to 15). Replication happens asynchronously. This distributes the read load across multiple database instances.

### Approach B: Lazy Loading with ElastiCache

```
┌──────────────┐    ┌──────────────────┐    ┌─────────────┐
│              │─1─▶│  ElastiCache     │    │             │
│  EC2         │    │  Cache MISS      │    │    RDS      │
│  Instance    │─2─▶│─────────────────▶│───▶│             │
│              │    │  Read from RDS   │    │             │
│              │─3─▶│  Write to cache  │    │             │
│              │    │                  │    │             │
│              │─4─▶│  Cache HIT! ✅   │    │             │
│              │    │  Return instantly │    │             │
└──────────────┘    └──────────────────┘    └─────────────┘
```

This is called ==lazy loading== (or cache-aside pattern):
1. EC2 instance checks ElastiCache first: "Do you have this product data?"
2. **Cache miss**: ElastiCache doesn't have it → read from RDS → write the result to ElastiCache
3. **Cache hit**: ElastiCache has it → return immediately (==sub-millisecond==)
4. Subsequent requests for the same data get cache hits — ==no RDS query needed==

This dramatically reduces RDS CPU usage and improves response times. But it requires ==application-side logic== to manage the cache (what to cache, when to invalidate, TTL settings).

## The Complete Multi-AZ Architecture

```
┌──────────┐   ┌──────────┐   ┌───────────────────┐   ┌──────────────────────────┐
│          │   │ Route 53 │   │    Multi-AZ ELB   │   │         ASG              │
│  Users   │──▶│  Alias   │──▶│  + Health Checks  │──▶│  AZ1: EC2  EC2          │
│          │   │          │   │                   │   │  AZ2: EC2  EC2          │
└──────────┘   └──────────┘   └───────────────────┘   │  AZ3: EC2              │
                                                        └───────────┬──────────────┘
                                                                    │
                              ┌─────────────────────────────────────┼──────────────────────┐
                              ▼                                     ▼                      ▼
                    ┌──────────────────┐                  ┌──────────────────┐   ┌──────────────────┐
                    │   ElastiCache    │                  │   RDS Master     │   │   RDS Standby    │
                    │   Multi-AZ       │                  │                  │   │   (Multi-AZ)     │
                    │   (Redis)        │                  │                  │   │   Auto-failover  │
                    │                  │                  └──────────────────┘   └──────────────────┘
                    │   Sessions +     │
                    │   Read Cache     │
                    └──────────────────┘
```

Every layer is now Multi-AZ:
- ==Route 53==: Already highly available by default (100% SLA)
- ==ELB==: Spans AZ1, AZ2, AZ3 with health checks
- ==ASG==: Distributes EC2 instances across all AZs
- ==RDS==: Multi-AZ with automatic failover (standby replica in another AZ)
- ==ElastiCache==: Multi-AZ with Redis (automatic failover if primary node fails)

## Security Group Chain

This is a critical security pattern — each layer only accepts traffic from the layer directly above it:

```
Internet
    │
    ▼
┌──────────────────────────────────────────┐
│  ALB Security Group                      │
│  Inbound: HTTP/HTTPS from 0.0.0.0/0     │  ← Anyone on the internet
└──────────────────┬───────────────────────┘
                   │
                   ▼
┌──────────────────────────────────────────┐
│  EC2 Security Group                      │
│  Inbound: from ALB Security Group ONLY   │  ← Only the load balancer
└──────────────────┬───────────────────────┘
                   │
          ┌────────┴────────┐
          ▼                 ▼
┌─────────────────┐  ┌─────────────────┐
│ ElastiCache SG  │  │    RDS SG       │
│ Inbound: from   │  │ Inbound: from   │
│ EC2 SG ONLY     │  │ EC2 SG ONLY     │
└─────────────────┘  └─────────────────┘
```

> [!important] Security Group Referencing
> Notice we're referencing ==security groups==, not IP addresses. This means if EC2 instances are added or removed (by the ASG), the rules automatically apply. No manual IP management needed.

## What We Learned — The 3-Tier Architecture

This case study introduced the classic ==3-tier web architecture==:

| Tier | Components | Purpose |
|------|-----------|---------|
| **Client Tier** | User's browser | Sends requests, stores session ID cookie |
| **Web Tier** | ELB + EC2 instances (ASG) | Handles HTTP requests, ==stateless== |
| **Database Tier** | RDS + ElastiCache | Durable storage + caching |

| Pattern | What It Solves | Trade-off |
|---------|---------------|-----------|
| **ELB Stickiness** | Shopping cart persistence | Fragile — cart lost on instance failure |
| **User Cookies** | True statelessness | 4 KB limit, security risk |
| **Server Sessions (ElastiCache)** | Secure, fast, stateless web tier | Additional infrastructure cost |
| **RDS** | Durable user data storage | Single point for writes |
| **Read Replicas** | Scale read traffic (up to 15) | Replication lag |
| **Lazy Loading (ElastiCache)** | Cache hot data, reduce RDS load | Cache maintenance complexity |
| **Multi-AZ (all layers)** | Survive AZ failures | Higher cost (standby replicas) |
| **Security Group Chaining** | Defense in depth | Must be configured correctly |

> [!note] Cost Awareness
> Yes, this architecture costs more than a single EC2 instance. Multi-AZ doubles your standby costs. Read Replicas and ElastiCache add per-instance costs. But for a production e-commerce site, these trade-offs are absolutely worth it. As a Solutions Architect, you need to ==understand and articulate these trade-offs==.

## Questions & Answers

> [!question]- Q1: Why does the shopping cart get lost without stickiness?
> **Answer:**
> The ELB distributes requests across multiple instances (round-robin or least-connections). Each instance stores the cart in ==local memory==. When the next request goes to a different instance, that instance has ==no knowledge of the cart== because there's no shared state between instances. The cart only exists in the memory of the instance that created it.

> [!question]- Q2: What are the three approaches to maintain state in a stateless web tier?
> **Answer:**
> 1. ==ELB Stickiness== — route all requests from a user to the same instance (simple but fragile)
> 2. ==User Cookies== — store cart data client-side in cookies (stateless but limited to 4 KB and security risk)
> 3. ==Server Sessions== — store session ID in cookie, cart data in ElastiCache or DynamoDB (best approach — secure, fast, truly stateless)

> [!question]- Q3: Why is ElastiCache preferred over user cookies for session storage?
> **Answer:**
> - ==More secure== — cart data is server-side, attackers can't modify it
> - ==No size limit== — cookies are limited to 4 KB total
> - ==Sub-millisecond performance== — ElastiCache is incredibly fast
> - ==Lighter HTTP requests== — only a small session ID is sent, not the entire cart
> - ==Survives instance termination== — data persists in ElastiCache regardless of which EC2 instance handles the request

> [!question]- Q4: What is lazy loading and how does it work with ElastiCache?
> **Answer:**
> Lazy loading (cache-aside pattern):
> 1. Application checks ElastiCache first
> 2. ==Cache miss==: data not in cache → read from RDS → write result to ElastiCache
> 3. ==Cache hit==: data found in cache → return immediately (sub-millisecond)
>
> This reduces RDS CPU usage and improves response times. The trade-off is that the first request for any data is slower (cache miss), and you need application-side logic to manage cache invalidation.

> [!question]- Q5: How do security groups chain together in a 3-tier architecture?
> **Answer:**
> Each layer's security group only allows traffic from the layer above:
> - ==ALB SG==: allows HTTP/HTTPS from 0.0.0.0/0 (the internet)
> - ==EC2 SG==: allows traffic only from the ALB security group
> - ==RDS SG==: allows traffic only from the EC2 security group
> - ==ElastiCache SG==: allows traffic only from the EC2 security group
>
> This creates ==defense in depth== — even if one layer is compromised, the attacker can't directly access the layers below.

> [!question]- Q6: What is the difference between RDS Read Replicas and ElastiCache for scaling reads?
> **Answer:**
> - ==Read Replicas== (up to 15): read directly from database replicas. Data is always fresh (slight replication lag). Good for complex SQL queries. No application code changes needed for reads.
> - ==ElastiCache==: caches frequently accessed data in memory. ==Sub-millisecond== response time. Requires application-side cache logic (lazy loading). Best for hot data that's read repeatedly.
>
> You can use ==both together== — ElastiCache for the hottest data, Read Replicas for less frequent but still heavy read queries.

> [!question]- Q7: How does Multi-AZ work for ElastiCache?
> **Answer:**
> ElastiCache with ==Redis== supports Multi-AZ with automatic failover. You have a primary node in one AZ and replica nodes in other AZs. If the primary node fails, ElastiCache automatically promotes a replica to become the new primary. This happens transparently — your application doesn't need to change connection strings. Note: this only works with Redis, not Memcached.

> [!question]- Q8: What is the 3-tier architecture in this context?
> **Answer:**
> 1. ==Client tier==: the user's browser — sends HTTP requests, stores the session ID cookie
> 2. ==Web tier==: ELB + EC2 instances in an ASG — handles requests, remains stateless by using ElastiCache for sessions
> 3. ==Database tier==: RDS for durable data (user profiles, orders) + ElastiCache for sessions and read caching
>
> This is the most common architecture for web applications and is heavily tested on the exam.

> [!question]- Q9: Why might you choose DynamoDB over ElastiCache for sessions?
> **Answer:**
> DynamoDB advantages:
> - ==Fully managed serverless== — no cluster to provision or manage
> - ==Built-in TTL== — sessions automatically expire after a set time
> - ==Scales automatically== — handles any amount of session data
> - ==Pay-per-request== pricing option — great for variable traffic
>
> Choose DynamoDB when you want zero infrastructure management. Choose ElastiCache when you need the absolute lowest latency (sub-millisecond) and are willing to manage a cluster.

> [!question]- Q10: What are the cost trade-offs in this architecture?
> **Answer:**
> | Component | Cost Impact | What You Get |
> |-----------|------------|-------------|
> | Multi-AZ RDS | ~2x (standby replica) | Automatic failover, survive AZ failure |
> | Multi-AZ ElastiCache | ~2x (replica nodes) | Automatic failover for sessions/cache |
> | Read Replicas | Per-replica cost | Scale reads, reduce master load |
> | ElastiCache cluster | Per-node cost | Sub-ms caching, session storage |
>
> These costs are justified for production e-commerce. The alternative — downtime, lost carts, slow reads — costs far more in lost revenue and customer trust.
