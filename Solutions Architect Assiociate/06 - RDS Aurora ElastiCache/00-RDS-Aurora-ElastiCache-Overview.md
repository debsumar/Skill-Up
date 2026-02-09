---
title: RDS, Aurora & ElastiCache - Overview
date: 2026-02-10
tags:
  - aws
  - rds
  - aurora
  - elasticache
  - saa-c03
  - index
---

# AWS Fundamentals: RDS + Aurora + ElastiCache

## Study Files

| # | Topic | File | Priority |
|---|-------|------|----------|
| 1 | Amazon RDS Overview | [[01-Amazon-RDS-Overview]] | 🔴 High |
| 2 | RDS Read Replicas vs Multi AZ | [[02-RDS-Read-Replicas-vs-Multi-AZ]] | 🔴 High |
| 3 | RDS Custom | [[03-RDS-Custom]] | 🟡 Medium |
| 4 | Amazon Aurora | [[04-Amazon-Aurora]] | 🔴 High |
| 5 | Aurora Advanced Concepts | [[05-Aurora-Advanced-Concepts]] | 🔴 High |
| 6 | RDS & Aurora Backup & Monitoring | [[06-RDS-Aurora-Backup-Monitoring]] | 🔴 High |
| 7 | RDS & Aurora Security | [[07-RDS-Aurora-Security]] | 🔴 High |
| 8 | RDS Proxy | [[08-RDS-Proxy]] | 🔴 High |
| 9 | ElastiCache Overview | [[09-ElastiCache-Overview]] | 🔴 High |
| 10 | ElastiCache for Solution Architects | [[10-ElastiCache-for-Solution-Architects]] | 🔴 High |

## Quick Decision Tree

```
Which Database Service?
├── Relational (SQL)?
│   ├── Managed, no OS access?
│   │   └── RDS (PostgreSQL, MySQL, MariaDB, Oracle, MS SQL, IBM DB2)
│   ├── Need OS/DB customization (Oracle/MS SQL)?
│   │   └── RDS Custom
│   ├── High performance, auto-scaling storage, cloud-native?
│   │   └── Aurora (5x MySQL, 3x Postgres perf)
│   └── Serverless, unpredictable workloads?
│       └── Aurora Serverless
├── In-memory cache (sub-ms latency)?
│   ├── Need HA, replication, persistence, sorted sets?
│   │   └── Redis (ElastiCache)
│   └── Simple caching, multi-threaded, sharding?
│       └── Memcached (ElastiCache)
└── Need connection pooling / reduced failover time?
    └── RDS Proxy
```

## Key Ports to Remember

| Database | Port |
|----------|------|
| PostgreSQL | 5432 |
| MySQL / MariaDB / Aurora | 3306 |
| Oracle | 1521 |
| MS SQL Server | 1433 |

## Related Sections

- [[../05 - High Availability & Scalability ELB ASG/00-Overview|High Availability & Scalability]]
- [[../04 - EC2 Instance Storage/00-EC2-Instance-Storage-Overview|EC2 Instance Storage]]
