---
title: Route 53 Records & TTL
date: 2026-02-10
tags:
  - aws
  - route53
  - ttl
  - dns-records
  - saa-c03
---

# Route 53 Records & TTL

## TTL (Time To Live)

TTL tells the client ==how long to cache== the DNS response before querying Route 53 again.

```
┌────────┐  DNS query   ┌──────────┐  A record + TTL 300s
│ Client │───────────▶│ Route 53 │─────────────────▶
└────────┘              └──────────┘
    │
    │  For 300 seconds: use cached IP
    │  (no new DNS query needed)
    ▼
┌────────────┐
│ Web Server │
└────────────┘
```

## High TTL vs Low TTL

| | High TTL (e.g., 24 hours) | Low TTL (e.g., 60 seconds) |
|---|---|---|
| **DNS traffic** | ==Less== (fewer queries) | ==More== (more queries) |
| **Cost** | ==Lower== | ==Higher== (pay per query) |
| **Stale records** | ==Possible== (up to 24h outdated) | ==Unlikely== (quick updates) |
| **Record changes** | ==Slow== to propagate | ==Fast== to propagate |

> [!tip] Record Change Strategy
> 1. ==Lower TTL== first (e.g., to 60s)
> 2. Wait for old TTL to expire
> 3. ==Change the record== value
> 4. ==Raise TTL== back up

> [!important] TTL is Mandatory
> TTL is mandatory for ==every DNS record== except ==Alias records== (TTL set automatically by Route 53).

## Questions & Answers

> [!question]- Q1: What is TTL?
> **Answer:**
> ==Time To Live== — the duration (in seconds) that a DNS response is cached by the client before making a new query to Route 53.

> [!question]- Q2: What happens during the TTL period if you change a record?
> **Answer:**
> Clients will ==continue using the old cached value== until the TTL expires. Only after expiry will they query Route 53 and get the new value.

> [!question]- Q3: What is the tradeoff of a high TTL?
> **Answer:**
> ==Less DNS traffic and lower cost==, but records can be ==outdated for longer== (up to the TTL duration).

> [!question]- Q4: What is the tradeoff of a low TTL?
> **Answer:**
> ==Faster record propagation==, but ==more DNS traffic and higher cost== (Route 53 charges per query).

> [!question]- Q5: What is the recommended strategy for changing a DNS record?
> **Answer:**
> 1. Lower the TTL first
> 2. Wait for old TTL to expire
> 3. Change the record value
> 4. Raise TTL back up

> [!question]- Q6: Which record type does NOT require a TTL?
> **Answer:**
> ==Alias records==. Their TTL is set automatically by Route 53.

> [!question]- Q7: Is TTL mandatory for A records?
> **Answer:**
> ==Yes==. TTL is mandatory for all records except Alias records.

> [!question]- Q8: What does a TTL of 300 mean?
> **Answer:**
> The client will cache the DNS response for ==300 seconds (5 minutes)== before querying Route 53 again.

> [!question]- Q9: Why does Route 53 charge more with low TTLs?
> **Answer:**
> Because Route 53 charges ==per DNS query==. Lower TTL means clients query more frequently, resulting in more billable requests.

> [!question]- Q10: If TTL is 120 seconds and you change a record after 30 seconds, when do clients see the change?
> **Answer:**
> After the remaining ==90 seconds== of the TTL expire. Clients use the cached value until TTL expires, then query Route 53 for the updated record.