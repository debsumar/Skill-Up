---
title: RDS Proxy
date: 2026-02-10
tags:
  - aws
  - rds
  - rds-proxy
  - connection-pooling
  - lambda
  - saa-c03
---

# RDS Proxy

## What is RDS Proxy?

A ==fully managed database proxy== for RDS that pools and shares database connections.

```
Without Proxy:                     With Proxy:
┌─────┐ ┌─────┐ ┌─────┐          ┌─────┐ ┌─────┐ ┌─────┐
│App 1│ │App 2│ │App 3│          │App 1│ │App 2│ │App 3│
└──┬──┘ └──┬──┘ └──┬──┘          └──┬──┘ └──┬──┘ └──┬──┘
   │       │       │                 │       │       │
   │       │       │              ┌──▼───────▼───────▼──┐
   │       │       │              │     RDS Proxy       │
   │       │       │              │  (connection pool)  │
   │       │       │              └──────────┬──────────┘
   │       │       │                  fewer connections
┌──▼───────▼───────▼──┐           ┌──────────▼──────────┐
│      RDS DB         │           │      RDS DB         │
│  (overwhelmed!)     │           │  (healthy)          │
└─────────────────────┘           └─────────────────────┘
```

## Three Key Benefits

| Benefit | Detail |
|---------|--------|
| **Connection Pooling** | Pools many app connections into ==fewer DB connections==. Reduces CPU/RAM stress and timeouts |
| **Failover Reduction** | Reduces failover time by up to ==66%== |
| **IAM Enforcement** | Enforce ==IAM authentication== for DB access. Credentials stored in ==Secrets Manager== |

## Key Properties

| Property | Detail |
|----------|--------|
| **Managed** | ==Fully serverless==, auto-scaling |
| **Availability** | ==Multi-AZ== (highly available) |
| **Supported engines** | MySQL, PostgreSQL, MariaDB, MS SQL Server, Aurora (MySQL & PostgreSQL) |
| **Code changes** | ==None required== — just change connection string to proxy |
| **Network** | ==Never publicly accessible== — VPC only |

> [!warning] VPC Only
> RDS Proxy is ==never publicly accessible==. It can only be accessed from within your VPC.

## Lambda + RDS Proxy

```
┌────────┐ ┌────────┐ ┌────────┐
│Lambda 1│ │Lambda 2│ │Lambda N│   (100s-1000s of functions)
└───┬────┘ └───┬────┘ └───┬────┘
    │          │          │
┌───▼──────────▼──────────▼───┐
│         RDS Proxy           │    (pools connections)
└─────────────┬───────────────┘
        fewer connections
┌─────────────▼───────────────┐
│          RDS DB             │    (healthy)
└─────────────────────────────┘
```

> [!important] Lambda Use Case
> Lambda functions ==multiply rapidly== (100s-1000s) and each opens a DB connection. Without RDS Proxy, this overwhelms the database with open connections and timeouts. RDS Proxy ==pools these connections== into fewer DB connections.

## Questions & Answers

> [!question]- Q1: What is the main purpose of RDS Proxy?
> **Answer:**
> To ==pool and share database connections==. Instead of every application connecting directly to RDS, they connect to the proxy which maintains fewer connections to the actual database.

> [!question]- Q2: By how much does RDS Proxy reduce failover time?
> **Answer:**
> Up to ==66%==. The proxy handles the failover internally, so applications don't experience it directly.

> [!question]- Q3: Is RDS Proxy publicly accessible?
> **Answer:**
> ==No==. RDS Proxy is ==never publicly accessible==. It can only be accessed from within your VPC.

> [!question]- Q4: Why is RDS Proxy especially useful with Lambda?
> **Answer:**
> Lambda functions can ==multiply rapidly== (100s-1000s), each opening a DB connection. This overwhelms the database. RDS Proxy ==pools these connections== into fewer DB connections, solving the problem.

> [!question]- Q5: Does RDS Proxy require code changes?
> **Answer:**
> ==No==. Just change the connection string from the RDS endpoint to the RDS Proxy endpoint.

> [!question]- Q6: How does RDS Proxy enforce IAM authentication?
> **Answer:**
> RDS Proxy can ==enforce IAM authentication== for database access. Credentials are securely stored in ==AWS Secrets Manager==.

> [!question]- Q7: What database engines does RDS Proxy support?
> **Answer:**
> MySQL, PostgreSQL, MariaDB, Microsoft SQL Server, and Aurora (MySQL & PostgreSQL).

> [!question]- Q8: Is RDS Proxy serverless?
> **Answer:**
> ==Yes==. It's fully serverless, auto-scaling, and highly available across ==multiple AZs==.

> [!question]- Q9: What are the three exam-relevant benefits of RDS Proxy?
> **Answer:**
> 1. ==Connection pooling== — reduces DB stress
> 2. ==Failover reduction== — up to 66% faster
> 3. ==IAM authentication enforcement== — credentials in Secrets Manager

> [!question]- Q10: Where are RDS Proxy credentials stored?
> **Answer:**
> In ==AWS Secrets Manager==. This provides secure, centralized credential management.
