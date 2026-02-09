---
title: ElastiCache Overview
date: 2026-02-10
tags:
  - aws
  - elasticache
  - redis
  - memcached
  - caching
  - saa-c03
---

# ElastiCache Overview

## What is ElastiCache?

Managed ==Redis or Memcached== — in-memory databases with ==high performance and low latency==.

| Feature | Detail |
|---------|--------|
| **Purpose** | Reduce load on databases for ==read-intensive workloads== |
| **How** | Cache common queries — DB not queried every time |
| **Stateless apps** | Store session data in ElastiCache |
| **Managed by AWS** | OS maintenance, patching, setup, monitoring, backups |
| **Code changes** | ==Required== — must modify app to query cache |

> [!warning] Code Changes Required
> ElastiCache is ==not== something you just enable. You must change your application code to query the cache before/after querying the database.

## Architecture: DB Cache

```
┌─────────┐   1. Query cache    ┌──────────────┐
│   App   │─────────────────────▶│ ElastiCache  │
│         │◀─────────────────────│              │
│         │   Cache HIT          └──────────────┘
│         │
│         │   2. Cache MISS      ┌──────────────┐
│         │─────────────────────▶│   RDS DB     │
│         │◀─────────────────────│              │
│         │   Read from DB       └──────────────┘
│         │
│         │   3. Write to cache  ┌──────────────┐
│         │─────────────────────▶│ ElastiCache  │
└─────────┘                      └──────────────┘
```

> [!important] Cache Invalidation
> You must have a ==cache invalidation strategy== to ensure only current data is in the cache.

## Architecture: Session Store

```
┌──────┐   Login    ┌─────────┐   Write session   ┌──────────────┐
│ User │──────────▶│  App 1  │──────────────────▶│ ElastiCache  │
└──────┘           └─────────┘                    └──────────────┘

┌──────┐  Redirect  ┌─────────┐   Read session    ┌──────────────┐
│ User │──────────▶│  App 2  │◀─────────────────│ ElastiCache  │
└──────┘           └─────────┘   (still logged   └──────────────┘
                                  in!)
```

Storing session data in ElastiCache makes your application ==stateless==.

## Redis vs Memcached

| Feature | Redis | Memcached |
|---------|-------|-----------|
| **Multi-AZ** | ==Yes== (auto-failover) | No |
| **Read Replicas** | ==Yes== (HA + read scaling) | No |
| **Data Persistence** | ==Yes== (AOF persistence) | No (not persistent) |
| **Backup & Restore** | ==Yes== | Only serverless version |
| **Sorted Sets** | ==Yes== (leaderboards!) | No |
| **Architecture** | Replication (primary → replica) | ==Sharding== (multi-node partitioning) |
| **Multi-threaded** | No | ==Yes== |
| **Data loss risk** | Low (persistent) | ==High== (lose cache = lose data) |

```
Redis:                          Memcached:
┌──────────┐  replication      ┌──────┐ ┌──────┐ ┌──────┐
│ Primary  │────────────▶     │Node 1│ │Node 2│ │Node 3│
│          │  ┌──────────┐    │ ⅓    │ │ ⅓    │ │ ⅓    │
└──────────┘  │ Replica  │    │ data │ │ data │ │ data │
              └──────────┘    └──────┘ └──────┘ └──────┘
                               (sharding / partitioning)
```

## Questions & Answers

> [!question]- Q1: What is Amazon ElastiCache?
> **Answer:**
> A managed service for ==Redis or Memcached== — in-memory databases with high performance and low latency. Used to cache data and reduce load on databases.

> [!question]- Q2: Does ElastiCache require application code changes?
> **Answer:**
> ==Yes==. You must modify your application to query the cache before/after querying the database. It's not a plug-and-play feature.

> [!question]- Q3: What is a cache hit vs cache miss?
> **Answer:**
> - **Cache hit**: Data found in cache → returned directly (==fast==)
> - **Cache miss**: Data not in cache → read from DB → write to cache for next time

> [!question]- Q4: How does ElastiCache make applications stateless?
> **Answer:**
> By storing ==session data== in ElastiCache. When a user is redirected to a different app instance, that instance reads the session from ElastiCache — user stays logged in.

> [!question]- Q5: Which ElastiCache engine supports Multi-AZ with auto-failover?
> **Answer:**
> ==Redis==. Memcached does not support Multi-AZ or auto-failover.

> [!question]- Q6: Which engine supports data persistence?
> **Answer:**
> ==Redis== (using AOF persistence). Memcached is not persistent — if you lose a node, you lose the data.

> [!question]- Q7: What is the key architectural difference between Redis and Memcached?
> **Answer:**
> - **Redis**: ==Replication== (primary → replica)
> - **Memcached**: ==Sharding== (data partitioned across multiple nodes)

> [!question]- Q8: Which engine supports sorted sets for leaderboards?
> **Answer:**
> ==Redis==. Sorted sets guarantee uniqueness and element ordering — perfect for real-time gaming leaderboards.

> [!question]- Q9: Is Memcached multi-threaded?
> **Answer:**
> ==Yes==. Memcached has a multi-threaded architecture, which can be beneficial for performance.

> [!question]- Q10: What is cache invalidation and why is it important?
> **Answer:**
> A strategy to ensure only ==current data== is in the cache. Without it, stale data may be served. It's one of the hardest problems in computer science.
