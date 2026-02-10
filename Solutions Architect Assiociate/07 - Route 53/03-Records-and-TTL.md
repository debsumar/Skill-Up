---
title: "Route 53 - Records and TTL"
date: 2026-02-10
tags:
  - aws
  - route53
  - dns
  - ttl
  - saa-c03
---

# Route 53 - Records and TTL

## Overview

==TTL (Time To Live)== is one of the most important concepts in DNS. It controls how long a DNS response is ==cached== by the client (browser, resolver) before it needs to ask Route 53 again. Understanding TTL is critical for the exam — it affects cost, performance, and how quickly DNS changes propagate.

Every DNS record in Route 53 has a TTL value (except ==Alias records==, where TTL is set automatically by Route 53).

## How TTL Works

```
┌──────────┐   1. DNS Query:              ┌──────────────┐
│          │   "myapp.example.com?"        │              │
│  Client  │──────────────────────────────▶│   Route 53   │
│          │◀──────────────────────────────│              │
│          │   2. Response:                │              │
│          │   A Record: 54.22.33.44      │              │
│          │   ==TTL: 300 seconds==        │              │
└────┬─────┘                               └──────────────┘
     │
     │  3. Client CACHES the result for 300 seconds
     │
     │  4. For the next 300 seconds, the client
     │     does NOT query Route 53 again —
     │     it uses the cached answer
     │
     │  5. After 300 seconds, the cache expires
     │     and the client queries Route 53 again
     ▼
┌──────────────┐
│  Web Server  │
│  54.22.33.44 │
└──────────────┘
```

The TTL tells the client: =="Cache this result for X seconds. Don't ask me again until the cache expires."==

## High TTL vs Low TTL

| Aspect | High TTL (e.g., 24 hours) | Low TTL (e.g., 60 seconds) |
|--------|--------------------------|---------------------------|
| **DNS traffic** | ==Less== — clients cache longer, fewer queries | ==More== — clients query more often |
| **Cost** | ==Cheaper== — fewer queries to Route 53 | ==More expensive== — more queries billed |
| **Record change speed** | ==Slow== — clients keep old value for up to 24 hours | ==Fast== — clients get new value within 60 seconds |
| **Outdated records** | ==Possible== — clients may have stale data | ==Unlikely== — clients refresh frequently |
| **Use case** | Stable records that rarely change | Records that change frequently |

> [!tip] TTL Strategy for Record Changes
> When you need to change a DNS record, use this strategy:
> 1. ==Lower the TTL== first (e.g., from 24 hours to 60 seconds)
> 2. ==Wait== for the old TTL to expire (so all clients pick up the new low TTL)
> 3. ==Change the record value==
> 4. All clients will get the new value within 60 seconds
> 5. ==Raise the TTL back== to the original value
>
> This minimizes the window where clients have stale data.

## TTL in Action (Hands-On)

The course demonstrates TTL with a practical example:

1. Create an A record `demo.yourdomain.com` pointing to an EC2 instance in `eu-central-1` with ==TTL of 120 seconds==
2. Verify with `dig` — the answer section shows the TTL countdown:
   ```
   ;; ANSWER SECTION:
   demo.yourdomain.com.  115  IN  A  3.70.14.xxx
   ```
   The `115` means 115 seconds remain before the cache expires.
3. Run `dig` again — the TTL has decreased (e.g., `98`)
4. ==Change the record== to point to a different EC2 instance (e.g., `ap-southeast-1`)
5. Run `dig` immediately — ==still shows the old IP== because the cache hasn't expired
6. Wait for TTL to expire → run `dig` again → ==now shows the new IP==

```
Time 0s:    dig → IP: 3.70.14.xxx (eu-central-1)  TTL: 120
Time 5s:    dig → IP: 3.70.14.xxx (eu-central-1)  TTL: 115
Time 10s:   CHANGE RECORD to ap-southeast-1
Time 15s:   dig → IP: 3.70.14.xxx (eu-central-1)  TTL: 105  ← STILL OLD!
Time 66s:   dig → IP: 3.70.14.xxx (eu-central-1)  TTL: 54   ← STILL OLD!
Time 120s:  CACHE EXPIRES
Time 121s:  dig → IP: 13.212.xxx  (ap-southeast-1) TTL: 120  ← NEW IP! ✅
```

> [!warning] TTL Gotcha
> Even after you change a record in Route 53, clients will ==continue using the old value== until their cached TTL expires. This is why lowering TTL before making changes is so important. If your TTL is 24 hours and you change a record, some clients won't see the change for up to 24 hours.

## TTL and Alias Records

==Alias records are the exception== — you cannot set TTL on alias records. Route 53 automatically sets the TTL based on the target resource. This is one of the advantages of alias records: AWS manages the TTL for optimal performance.

## TTL Best Practices

| Scenario | Recommended TTL | Why |
|----------|----------------|-----|
| **Stable production records** | ==3600-86400 seconds== (1-24 hours) | Reduces DNS query costs, records rarely change |
| **Records you plan to change** | ==60-300 seconds== (1-5 minutes) | Allows quick propagation of changes |
| **During a migration** | ==60 seconds== | Minimize stale data during cutover |
| **After migration is complete** | ==Increase back to 3600+== | Reduce costs once stable |

## Questions & Answers

> [!question]- Q1: What is TTL in DNS?
> **Answer:**
> TTL (Time To Live) is a value in seconds that tells the client ==how long to cache a DNS response==. During the TTL period, the client uses the cached answer without querying Route 53 again. After the TTL expires, the client makes a new DNS query to get the latest value.

> [!question]- Q2: What happens if you change a record while clients still have it cached?
> **Answer:**
> Clients will ==continue using the old cached value== until their TTL expires. They won't see the new record value until they make a fresh DNS query after the cache expires. This is why you should ==lower the TTL before making changes== to minimize the window of stale data.

> [!question]- Q3: What is the trade-off between high and low TTL?
> **Answer:**
> - ==High TTL==: Less DNS traffic (cheaper), but changes propagate slowly and clients may have outdated records
> - ==Low TTL==: More DNS traffic (more expensive), but changes propagate quickly and records are always fresh
> Choose based on how often your records change and how critical freshness is.

> [!question]- Q4: What is the recommended strategy for changing a DNS record?
> **Answer:**
> 1. ==Lower the TTL== (e.g., to 60 seconds) well in advance
> 2. ==Wait== for the old TTL to expire (so all clients pick up the low TTL)
> 3. ==Make the record change==
> 4. Clients get the new value within 60 seconds
> 5. ==Raise the TTL back== to the original value once the change is confirmed

> [!question]- Q5: Can you set TTL on Alias records?
> **Answer:**
> ==No.== Alias records have their TTL set ==automatically by Route 53== based on the target resource. You cannot override it. This is one of the differences between Alias records and standard records (A, AAAA, CNAME).

> [!question]- Q6: How does TTL affect Route 53 costs?
> **Answer:**
> Route 53 charges per DNS query. With a ==high TTL==, clients cache longer and make fewer queries → ==lower cost==. With a ==low TTL==, clients query more frequently → ==higher cost==. For stable records, use a high TTL to minimize costs.

> [!question]- Q7: How can you observe TTL behavior?
> **Answer:**
> Use the `dig` command. The answer section shows the ==remaining TTL== in seconds:
> ```
> demo.example.com.  98  IN  A  3.70.14.xxx
> ```
> The `98` means 98 seconds remain before the cache expires. Run `dig` again and the number decreases. When it reaches 0, a new query is made to Route 53.

> [!question]- Q8: Is TTL mandatory for all Route 53 records?
> **Answer:**
> TTL is ==mandatory for every record type except Alias records==. For Alias records, Route 53 sets the TTL automatically. For all other records (A, AAAA, CNAME, etc.), you must specify a TTL value.

> [!question]- Q9: What TTL should you use during a migration?
> **Answer:**
> During a migration, use a ==low TTL (60 seconds)== so that when you switch the record to the new target, all clients pick up the change within one minute. After the migration is confirmed and stable, ==increase the TTL back== to a higher value (e.g., 3600 seconds) to reduce costs.

> [!question]- Q10: What is the default TTL in Route 53?
> **Answer:**
> The default TTL when creating a record in the Route 53 console is ==300 seconds (5 minutes)==. This is a reasonable default for most use cases — it balances cost (not too many queries) with freshness (changes propagate within 5 minutes).
