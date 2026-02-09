---
title: Amazon RDS Overview
date: 2026-02-10
tags:
  - aws
  - rds
  - database
  - managed-service
  - saa-c03
---

# Amazon RDS Overview

## What is RDS?

==Relational Database Service== — a managed database service for SQL-based databases in the cloud.

## Supported Engines

| Engine | Notes |
|--------|-------|
| **PostgreSQL** | Open source |
| **MySQL** | Open source |
| **MariaDB** | Open source |
| **Oracle** | Commercial |
| **Microsoft SQL Server** | Commercial |
| **IBM DB2** | Commercial |
| **Aurora** | AWS proprietary (covered separately) |

> [!important] Exam Tip
> Remember all 7 engine types supported by RDS.

## Why RDS over DB on EC2?

RDS is a ==managed service==. AWS handles:

| Feature | Description |
|---------|-------------|
| Automated provisioning | DB + OS setup |
| OS patching | Automatic |
| Continuous backups | Point-in-Time Restore (up to 5 min ago) |
| Monitoring dashboards | Built-in performance metrics |
| Read Replicas | Up to 15, for read scaling |
| Multi AZ | Disaster recovery |
| Maintenance windows | Scheduled upgrades |
| Vertical scaling | Change instance type |
| Horizontal scaling | Add read replicas |
| Storage | Backed by ==EBS== (gp2 or io1) |

> [!warning] No SSH Access
> You ==cannot SSH== into RDS instances. It's a managed service — no access to the underlying EC2 instance. (Exception: RDS Custom)

## RDS Storage Auto Scaling

```
┌──────────────┐     Threshold met?     ┌──────────────────┐
│  RDS DB      │ ──────────────────────▶ │  Auto-increase   │
│  20 GB used  │   Free < 10%           │  storage         │
│              │   Low > 5 min          │  (up to max)     │
│              │   6 hrs since last mod │                  │
└──────────────┘                         └──────────────────┘
```

| Setting | Detail |
|---------|--------|
| **Trigger** | Free storage < ==10%== of allocated |
| **Duration** | Low storage lasts > ==5 minutes== |
| **Cooldown** | ==6 hours== since last modification |
| **Max threshold** | You set the maximum storage limit |
| **Supports** | All RDS database engines |

> [!tip] Use Case
> Great for applications with ==unpredictable workloads==. Avoids manual storage scaling operations.

## Questions & Answers

> [!question]- Q1: What does RDS stand for and what is it?
> **Answer:**
> ==Relational Database Service==. A managed AWS service for SQL databases in the cloud. AWS handles provisioning, patching, backups, monitoring, and scaling.

> [!question]- Q2: What are the 7 database engines supported by RDS?
> **Answer:**
> 1. PostgreSQL
> 2. MySQL
> 3. MariaDB
> 4. Oracle
> 5. Microsoft SQL Server
> 6. IBM DB2
> 7. Aurora (AWS proprietary)

> [!question]- Q3: Can you SSH into an RDS instance?
> **Answer:**
> ==No==. RDS is a managed service — no access to the underlying EC2 instance. The only exception is ==RDS Custom== (for Oracle and MS SQL Server).

> [!question]- Q4: What is Point-in-Time Restore?
> **Answer:**
> RDS continuously backs up transaction logs. You can restore your database to ==any point in time== within the backup retention period (up to 5 minutes ago).

> [!question]- Q5: What storage type does RDS use?
> **Answer:**
> ==EBS volumes== (gp2 for general purpose or io1 for provisioned IOPS).

> [!question]- Q6: When does RDS Storage Auto Scaling trigger?
> **Answer:**
> Three conditions must be met:
> - Free storage < ==10%== of allocated
> - Low storage persists for > ==5 minutes==
> - ==6 hours== have passed since last storage modification

> [!question]- Q7: What is the benefit of RDS over deploying a DB on EC2?
> **Answer:**
> RDS provides automated provisioning, OS patching, backups, monitoring, read replicas, Multi AZ, maintenance windows, and scaling — all managed by AWS.

> [!question]- Q8: Can RDS scale both vertically and horizontally?
> **Answer:**
> ==Yes==.
> - **Vertical**: Change the instance type (e.g., db.t3.micro → db.r5.large)
> - **Horizontal**: Add ==read replicas== (up to 15)

> [!question]- Q9: Do you need to set a maximum storage threshold for auto scaling?
> **Answer:**
> ==Yes==. You must set a ==maximum storage threshold== to prevent unlimited growth.

> [!question]- Q10: Is RDS Storage Auto Scaling supported for all engines?
> **Answer:**
> ==Yes==. It supports all RDS database engines (PostgreSQL, MySQL, MariaDB, Oracle, MS SQL Server, IBM DB2).
