---
title: ElastiCache for Solution Architects
date: 2026-02-10
tags:
  - aws
  - elasticache
  - redis
  - memcached
  - security
  - caching-patterns
  - saa-c03
---

# ElastiCache for Solution Architects

## ElastiCache Security

### Redis Security

| Feature | Detail |
|---------|--------|
| **IAM Authentication** | ==Supported== (Redis only) |
| **IAM Policies** | Only for ==AWS API-level security== (not data access) |
| **Redis AUTH** | Set ==password/token== at cluster creation (extra security layer) |
| **In-flight encryption** | ==SSL supported== |
| **Security Groups** | Network-level access control |

### Memcached Security

| Feature | Detail |
|---------|--------|
| **Authentication** | ==SASL-based== authentication |

```
EC2 Client ──── Redis AUTH ────▶ Redis Cluster
                  or               (protected by
             IAM Auth              Security Group)
                                   + SSL in-flight
```

## Caching Patterns

| Pattern | How It Works | Stale Data? |
|---------|-------------|-------------|
| **Lazy Loading** | Read data cached on ==cache miss== only | ==Yes== (data can become stale) |
| **Write Through** | Data written to cache ==whenever DB is updated== | ==No== (always current) |
| **Session Store** | Store user sessions with ==TTL== (time to live) | Expires automatically |

### Lazy Loading Flow

```
┌─────────┐  1. Check cache   ┌──────────────┐
│   App   │───────────────────▶│ ElastiCache  │
│         │                    │              │
│         │  Cache HIT ────────│  Return data │
│         │                    └──────────────┘
│         │
│         │  Cache MISS        ┌──────────────┐
│         │───────────────────▶│   Database   │
│         │◀───────────────────│  Read data   │
│         │                    └──────────────┘
│         │
│         │  Write to cache    ┌──────────────┐
│         │───────────────────▶│ ElastiCache  │
└─────────┘                    └──────────────┘
```

> [!quote] Famous Quote
> "There are only two hard things in computer science: ==cache invalidation== and ==naming things==."

## Redis Use Case: Gaming Leaderboards

| Feature | Detail |
|---------|--------|
| **Redis Sorted Sets** | Guarantees ==uniqueness + element ordering== |
| **Real-time ranking** | Elements ranked ==in real time== as they're added |
| **All replicas** | Same leaderboard available across all Redis cache nodes |
| **No custom code** | Leverage Redis sorted sets directly — ==no application-side programming== needed |

```
┌──────────┐     ┌──────────────┐
│ Client 1 │────▶│              │  #1: Player A (1500 pts)
│ Client 2 │────▶│ Redis Cache  │  #2: Player B (1200 pts)
│ Client 3 │────▶│ (Sorted Set) │  #3: Player C (900 pts)
└──────────┘     └──────────────┘
```

> [!tip] Exam Hint
> If you see "==real-time leaderboard==" or "==gaming leaderboard==" → think ==Redis Sorted Sets==.

## Questions & Answers

> [!question]- Q1: Does ElastiCache support IAM authentication?
> **Answer:**
> ==Only Redis== supports IAM authentication. IAM policies on ElastiCache are only for ==AWS API-level security==, not for data-level access.

> [!question]- Q2: What is Redis AUTH?
> **Answer:**
> A security feature where you set a ==password/token== when creating a Redis cluster. Provides an extra layer of security on top of security groups.

> [!question]- Q3: What authentication does Memcached use?
> **Answer:**
> ==SASL-based== authentication.

> [!question]- Q4: What is the Lazy Loading caching pattern?
> **Answer:**
> Data is cached ==only on a cache miss==. App checks cache first → if miss, reads from DB → writes result to cache. Data ==can become stale==.

> [!question]- Q5: What is the Write Through caching pattern?
> **Answer:**
> Data is written to cache ==whenever it's written to the database==. Ensures ==no stale data== in the cache.

> [!question]- Q6: How does ElastiCache work as a session store?
> **Answer:**
> User sessions are stored with a ==TTL (time to live)==. Sessions expire automatically after the TTL period.

> [!question]- Q7: What are Redis Sorted Sets used for?
> **Answer:**
> ==Real-time gaming leaderboards==. Sorted sets guarantee ==uniqueness and element ordering==. Elements are ranked in real time as they're added.

> [!question]- Q8: Do you need to write custom code for Redis leaderboards?
> **Answer:**
> ==No==. Redis Sorted Sets handle ranking natively. You just leverage the built-in sorted set data structure.

> [!question]- Q9: Does Redis support SSL in-flight encryption?
> **Answer:**
> ==Yes==. Redis supports SSL for in-flight encryption between clients and the cache.

> [!question]- Q10: What are the three caching patterns for ElastiCache?
> **Answer:**
> 1. ==Lazy Loading== — cache on miss (can have stale data)
> 2. ==Write Through== — cache on write (no stale data)
> 3. ==Session Store== — store sessions with TTL (auto-expire)
