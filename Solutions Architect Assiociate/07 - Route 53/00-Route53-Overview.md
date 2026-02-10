---
title: "Route 53 - Section Overview"
date: 2026-02-10
tags:
  - aws
  - route53
  - dns
  - saa-c03
---

# Route 53 - Section Overview

## Section Map

This section covers Amazon Route 53 — AWS's managed DNS service. It starts with DNS fundamentals and builds up to advanced routing policies and health checks.

| # | Topic | Key Concepts |
|---|-------|-------------|
| [[01-What-is-DNS\|01]] | What is DNS? | DNS hierarchy, FQDN, recursive resolution, TLD, SLD, name servers |
| [[02-Route53-Overview\|02]] | Route 53 Overview | Hosted zones (public/private), record types (A, AAAA, CNAME, NS), domain registration, EC2 setup |
| [[03-Records-and-TTL\|03]] | Records and TTL | Time To Live, caching behavior, high vs low TTL trade-offs |
| [[04-CNAME-vs-Alias\|04]] | CNAME vs Alias | Zone apex restriction, Alias targets, free queries, health check support |
| [[05-Routing-Policies\|05]] | Routing Policies | Simple, Weighted, Latency-based routing |
| [[06-Health-Checks\|06]] | Health Checks & Failover | Endpoint checks, calculated checks, CloudWatch Alarm checks, Failover routing |
| [[07-Routing-Policies-Advanced\|07]] | Advanced Routing Policies | Geolocation, Geoproximity (bias), IP-based, Multi-Value |
| [[08-Third-Party-Domains\|08]] | Third-Party Domains | Domain registrar vs DNS service, using Route 53 with GoDaddy, section cleanup |

## Key Exam Topics

- ==Alias vs CNAME== — zone apex, free queries, valid targets
- ==Routing policies== — when to use each (Weighted for canary, Latency for performance, Geolocation for localization, Geoproximity for traffic shifting)
- ==Health checks== — endpoint, calculated, CloudWatch Alarm (for private resources)
- ==Failover routing== — primary/secondary, mandatory health check on primary
- ==100% availability SLA== — only AWS service with this guarantee
