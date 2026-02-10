---
title: "Route 53 - Health Checks & Failover Routing"
date: 2026-02-10
tags:
  - aws
  - route53
  - dns
  - health-checks
  - failover
  - saa-c03
---

# Route 53 - Health Checks & Failover Routing

## Overview

Health checks are how Route 53 knows whether your resources are ==healthy or unhealthy==. Without health checks, Route 53 would blindly return IP addresses — even for servers that are down. By combining health checks with routing policies, you get ==automated DNS failover==: if a resource fails, Route 53 stops sending traffic to it.

Health checks primarily monitor ==public resources==, but there's a workaround for private resources using CloudWatch Alarms.

## Three Types of Health Checks

```
┌─────────────────────────────────────────────────────────────────┐
│                    HEALTH CHECK TYPES                           │
│                                                                 │
│  1. ENDPOINT          2. CALCULATED         3. CLOUDWATCH       │
│  ┌──────────────┐    ┌──────────────┐      ┌──────────────┐   │
│  │ Monitor a    │    │ Combine      │      │ Monitor a    │   │
│  │ public       │    │ multiple     │      │ CloudWatch   │   │
│  │ endpoint     │    │ child health │      │ Alarm        │   │
│  │ (HTTP/TCP)   │    │ checks       │      │ (for private │   │
│  │              │    │ (AND/OR/NOT) │      │  resources)  │   │
│  └──────────────┘    └──────────────┘      └──────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

## Type 1: Endpoint Health Checks

About ==15 global health checkers== from around the world send requests to your endpoint:

```
┌──────────────────────────────────────────────────────────────┐
│              ENDPOINT HEALTH CHECK                            │
│                                                               │
│  Health Checkers (15+ worldwide)                             │
│  ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐                       │
│  │ US │ │ EU │ │Asia│ │ SA │ │ AU │  ...                    │
│  └──┬─┘ └──┬─┘ └──┬─┘ └──┬─┘ └──┬─┘                       │
│     │      │      │      │      │                            │
│     └──────┴──────┴──────┴──────┘                            │
│                    │                                          │
│                    ▼                                          │
│            ┌──────────────┐                                  │
│            │  Your Public │  HTTP/HTTPS/TCP                  │
│            │  Endpoint    │  Must return 2xx or 3xx          │
│            │  (ALB, EC2)  │                                  │
│            └──────────────┘                                  │
│                                                               │
│  Healthy if ≥ 18% of checkers report healthy                 │
└──────────────────────────────────────────────────────────────┘
```

### Configuration Options

| Setting | Options | Default |
|---------|---------|---------|
| **Protocol** | HTTP, HTTPS, TCP | HTTP |
| **Interval** | ==30 seconds== (standard) or ==10 seconds== (fast, higher cost) | 30 sec |
| **Failure threshold** | Number of consecutive failures before unhealthy | 3 |
| **String matching** | Check first ==5,120 bytes== of response for specific text | Disabled |
| **Healthy threshold** | ==≥ 18%== of health checkers must report healthy | 18% |
| **Regions** | Choose specific regions or use recommended | Recommended |

> [!warning] Security Group Requirement
> Health checkers come from ==public IP addresses== all around the world. Your security group must ==allow inbound traffic from Route 53 health checker IPs==. If your security group blocks these IPs, the health check will always fail (connection timeout). AWS publishes the health checker IP ranges.

### String Matching

Health checks can inspect the first ==5,120 bytes== of the HTTP response body. If you enable string matching, the health check passes only if the response contains the specified string. This is useful for checking that your application is truly functional, not just returning a 200 status code.

## Type 2: Calculated Health Checks

Combine multiple ==child health checks== into a single ==parent health check==:

```
┌──────────────────────────────────────────────────────────────┐
│              CALCULATED HEALTH CHECK                          │
│                                                               │
│  ┌──────────────────────────────────────────────────────┐    │
│  │              Parent Health Check                      │    │
│  │              (Calculated)                             │    │
│  │                                                       │    │
│  │  Condition: AND / OR / NOT                           │    │
│  │  "Report healthy when X of Y children are healthy"   │    │
│  └──────────┬──────────┬──────────┬─────────────────────┘    │
│             │          │          │                           │
│       ┌─────▼────┐ ┌──▼──────┐ ┌▼─────────┐                │
│       │ Child 1  │ │ Child 2 │ │ Child 3  │                │
│       │ EC2 (US) │ │ EC2 (EU)│ │ EC2 (AP) │                │
│       │ ✅ Healthy│ │✅Healthy│ │❌Unhealthy│                │
│       └──────────┘ └─────────┘ └──────────┘                │
│                                                               │
│  With AND: Parent = ❌ (not all children healthy)            │
│  With OR:  Parent = ✅ (at least one child healthy)          │
└──────────────────────────────────────────────────────────────┘
```

- Combine up to ==256 child health checks==
- Conditions: ==AND== (all must pass), ==OR== (at least N must pass), ==NOT== (invert)
- Specify how many children must be healthy for the parent to be healthy
- Use case: perform maintenance on one resource without triggering a full failover

## Type 3: CloudWatch Alarm Health Checks (Private Resources)

Route 53 health checkers live on the ==public internet== — they ==cannot access private resources== (private VPC, on-premises). The workaround:

```
┌──────────────────────────────────────────────────────────────┐
│         MONITORING PRIVATE RESOURCES                          │
│                                                               │
│  ┌──────────────┐   ┌──────────────┐   ┌──────────────┐    │
│  │  Private EC2  │──▶│  CloudWatch  │──▶│  CloudWatch  │    │
│  │  Instance     │   │  Metric      │   │  Alarm       │    │
│  │  (in VPC)     │   │  (CPU, etc.) │   │  (threshold) │    │
│  └──────────────┘   └──────────────┘   └──────┬───────┘    │
│                                                 │            │
│                                          ┌──────▼───────┐    │
│                                          │  Route 53    │    │
│                                          │  Health Check│    │
│                                          │  (monitors   │    │
│                                          │   alarm)     │    │
│                                          └──────────────┘    │
│                                                               │
│  When alarm triggers → health check becomes unhealthy        │
└──────────────────────────────────────────────────────────────┘
```

1. Create a ==CloudWatch Metric== monitoring your private resource (CPU, custom metric, etc.)
2. Create a ==CloudWatch Alarm== on that metric
3. Create a Route 53 health check that ==monitors the CloudWatch Alarm==
4. When the alarm fires → health check becomes unhealthy → Route 53 stops routing to that resource

> [!tip] Exam Tip
> If the exam asks "how to create a Route 53 health check for a private resource," the answer is ==CloudWatch Metric → CloudWatch Alarm → Route 53 Health Check==. Direct health checks cannot reach private resources.

## Failover Routing Policy

Failover routing uses health checks to provide ==automatic DNS failover== between a primary and secondary resource:

```
┌──────────────────────────────────────────────────────────────┐
│                    FAILOVER ROUTING                           │
│                                                               │
│  Normal Operation:                                           │
│  ┌──────────┐   DNS Query    ┌──────────────┐               │
│  │  Client  │───────────────▶│   Route 53   │               │
│  │          │◀───────────────│   (Failover) │               │
│  │          │  Primary IP    └──────┬───────┘               │
│  └──────────┘                       │                        │
│                              ┌──────▼───────┐                │
│                              │   PRIMARY    │  ✅ Healthy    │
│                              │   EC2 (EU)   │  (health check │
│                              │              │   passes)      │
│                              └──────────────┘                │
│                                                               │
│  After Primary Fails:                                        │
│  ┌──────────┐   DNS Query    ┌──────────────┐               │
│  │  Client  │───────────────▶│   Route 53   │               │
│  │          │◀───────────────│   (Failover) │               │
│  │          │  Secondary IP  └──────┬───────┘               │
│  └──────────┘                       │                        │
│                              ┌──────▼───────┐                │
│                              │  SECONDARY   │  ✅ Healthy    │
│                              │  EC2 (US)    │  (DR instance) │
│                              │              │                │
│                              └──────────────┘                │
│                                                               │
│  Primary EC2 (EU): ❌ Health check FAILED                    │
└──────────────────────────────────────────────────────────────┘
```

### Key Rules

| Rule | Detail |
|------|--------|
| **Primary record** | ==Must== have a health check associated |
| **Secondary record** | Health check is ==optional== (but recommended) |
| **Only two records** | One primary, one secondary — that's it |
| **Automatic failover** | When primary health check fails, Route 53 returns secondary |
| **Automatic failback** | When primary recovers, Route 53 returns primary again |

### Hands-On Walkthrough

1. Create failover record: `failover.yourdomain.com`
   - Primary: EU Central IP → Health check: EU Central → Record type: ==Primary==
   - Secondary: US East IP → Health check: US East (optional) → Record type: ==Secondary==
2. Test: access URL → gets EU Central response ✅
3. Break primary: remove HTTP rule from EU Central security group
4. Wait for health check to fail (30-60 seconds)
5. Refresh URL → now gets ==US East response== ✅ (automatic failover!)
6. Fix primary: add HTTP rule back → health check recovers → traffic returns to primary

> [!important] Failover is Automatic
> Once configured, failover happens ==automatically== with no manual intervention. Route 53 continuously monitors the health check and switches DNS responses when the primary fails. This is the foundation of ==disaster recovery== architectures on AWS.

## Health Check Monitoring

All health checks publish metrics to ==CloudWatch==:
- Health check status (healthy/unhealthy)
- Percentage of health checkers reporting healthy
- Connection time
- Time to first byte

You can view these in the Route 53 Health Checks console under the ==Monitoring== tab.

## Questions & Answers

> [!question]- Q1: What are the three types of Route 53 health checks?
> **Answer:**
> 1. ==Endpoint health checks==: Monitor a public endpoint (HTTP/HTTPS/TCP) using ~15 global health checkers
> 2. ==Calculated health checks==: Combine multiple child health checks using AND/OR/NOT logic
> 3. ==CloudWatch Alarm health checks==: Monitor a CloudWatch Alarm (used for ==private resources== that health checkers can't reach)

> [!question]- Q2: How does an endpoint health check determine if a resource is healthy?
> **Answer:**
> About ==15 global health checkers== send requests to your endpoint every 30 seconds (or 10 seconds for fast checks). If ==≥ 18% of health checkers== report the endpoint as healthy (2xx or 3xx response), Route 53 considers it healthy. The health check can also inspect the first ==5,120 bytes== of the response for string matching.

> [!question]- Q3: Why must security groups allow Route 53 health checker IPs?
> **Answer:**
> Health checkers are ==public servers== that send HTTP/HTTPS/TCP requests to your endpoint. If your security group blocks their IP addresses, the requests will timeout and the health check will ==always report unhealthy==. You must allow inbound traffic from the Route 53 health checker IP ranges (published by AWS).

> [!question]- Q4: How do you health-check a private resource?
> **Answer:**
> Route 53 health checkers live on the public internet and ==cannot access private VPC resources==. The workaround:
> 1. Create a ==CloudWatch Metric== monitoring the private resource
> 2. Create a ==CloudWatch Alarm== on that metric
> 3. Create a Route 53 health check that ==monitors the CloudWatch Alarm==
> When the alarm fires, the health check becomes unhealthy.

> [!question]- Q5: What is a calculated health check?
> **Answer:**
> A calculated health check combines up to ==256 child health checks== into a single parent. You define conditions:
> - ==AND==: All children must be healthy
> - ==OR==: At least N children must be healthy
> - ==NOT==: Invert the result
> Use case: perform maintenance on one resource without triggering a full failover.

> [!question]- Q6: How does Failover routing work?
> **Answer:**
> You create two records: ==Primary== (must have health check) and ==Secondary== (health check optional). Normally, Route 53 returns the primary IP. If the primary health check fails, Route 53 ==automatically returns the secondary IP==. When the primary recovers, traffic automatically returns to it. Only one primary and one secondary are allowed.

> [!question]- Q7: Is the health check on the secondary failover record mandatory?
> **Answer:**
> ==No==, it's optional but recommended. If the secondary also has a health check and both primary and secondary are unhealthy, Route 53 will still return the secondary (it's the last resort). Having a health check on the secondary helps you monitor its status.

> [!question]- Q8: What is the difference between standard and fast health checks?
> **Answer:**
> - ==Standard==: Health checkers send requests every ==30 seconds== (default, lower cost)
> - ==Fast==: Health checkers send requests every ==10 seconds== (higher cost, faster detection)
> Fast health checks detect failures more quickly but cost more. Use fast checks for critical production resources where every second of downtime matters.

> [!question]- Q9: Can health checks monitor HTTPS endpoints?
> **Answer:**
> ==Yes.== Health checks support HTTP, HTTPS, and TCP protocols. For HTTPS, the health checker verifies the SSL certificate and checks the response status code. Note that health checkers do ==not follow redirects== — if your endpoint returns a 301/302, the health check may fail unless you configure it to accept 3xx responses.

> [!question]- Q10: Exam scenario — primary region goes down, users still see errors. What's wrong?
> **Answer:**
> Most likely causes:
> 1. ==No health check== on the primary record — Route 53 doesn't know it's down
> 2. ==TTL hasn't expired== — clients are still using the cached primary IP
> 3. ==Security group blocking== health checkers — health check always shows healthy even though the resource is down
> 4. ==No failover routing policy== configured — using Simple routing which doesn't support failover
