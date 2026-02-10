---
title: Amazon Aurora
date: 2026-02-10
tags:
  - aws
  - aurora
  - database
  - high-availability
  - saa-c03
---

# Amazon Aurora

## Overview

==AWS proprietary== database — not open source, but compatible with MySQL and PostgreSQL drivers.

| Feature | Detail |
|---------|--------|
| **Compatibility** | MySQL and PostgreSQL drivers |
| **Performance** | ==5x MySQL== on RDS, ==3x PostgreSQL== on RDS |
| **Storage** | Auto-grows from ==10 GB to 128 TB== (==256 TiB== on newer versions) |
| **Read Replicas** | Up to ==15== (sub 10ms replica lag) |
| **Failover** | ==Instantaneous== (< 30 seconds avg) |
| **High Availability** | ==Native== (cloud-native by default) |
| **Cost** | ~==20% more== than RDS, but more efficient at scale |

## High Availability & Read Scaling

Aurora stores ==6 copies== of your data across ==3 AZs==:

| Operation | Copies Needed | Implication |
|-----------|--------------|-------------|
| **Writes** | 4 out of 6 | Survives losing 1 AZ |
| **Reads** | 3 out of 6 | Highly available for reads |

```
         AZ 1           AZ 2           AZ 3
      ┌────────┐     ┌────────┐     ┌────────┐
      │ Copy 1 │     │ Copy 3 │     │ Copy 5 │
      │ Copy 2 │     │ Copy 4 │     │ Copy 6 │
      └────────┘     └────────┘     └────────┘
              └──────────┼──────────┘
                  Shared Storage Volume
            (auto expanding, self-healing,
             peer-to-peer replication)
```

- ==Self-healing==: corrupted data repaired via peer-to-peer replication
- Storage uses ==hundreds of volumes== (not just one)
- Only ==one master== takes writes; up to 15 read replicas serve reads
- Any read replica can become master on failover
- Supports ==cross-region replication==

## Aurora Cluster Architecture

```
                    ┌──────────────┐
                    │    Client    │
                    └──┬───────┬───┘
                       │       │
              Writer   │       │   Reader
             Endpoint  │       │  Endpoint
                       │       │
                ┌──────▼──┐ ┌──▼──────────────┐
                │  Master │ │  Read Replicas   │
                │ (write) │ │  (1-15, auto     │
                │         │ │   scaling)       │
                └────┬────┘ └────────┬─────────┘
                     │               │
              ┌──────▼───────────────▼──────┐
              │   Shared Storage Volume     │
              │   10 GB → 128 TB auto (256 TiB on newer versions)  │
              └─────────────────────────────┘
```

| Endpoint | Purpose |
|----------|---------|
| **Writer Endpoint** | ==DNS name== always pointing to the master (survives failover) |
| **Reader Endpoint** | ==Connection load balancing== across all read replicas |

> [!important] Load Balancing Level
> Reader endpoint load balancing happens at the ==connection level==, not the statement level.

## Key Features Summary

| Feature | Detail |
|---------|--------|
| Automatic failover | < 30 seconds |
| Backup & recovery | Built-in |
| Push-button scaling | Auto scaling replicas |
| Automated patching | Zero downtime |
| Backtrack | Restore to ==any point in time== (not backup-based) |
| Advanced monitoring | Built-in |
| Cross-region replication | Supported |

> [!tip] Backtrack
> Unlike regular restore, backtrack lets you ==go back and forth== in time. "Go to yesterday 4 PM... actually, 5 PM" — both work without creating a new DB.

## Questions & Answers

> [!question]- Q1: Is Aurora open source?
> **Answer:**
> ==No==. Aurora is ==AWS proprietary==, but it's compatible with MySQL and PostgreSQL drivers. Your existing MySQL/PostgreSQL applications can connect to Aurora without changes.

> [!question]- Q2: How much faster is Aurora compared to RDS?
> **Answer:**
> ==5x faster== than MySQL on RDS and ==3x faster== than PostgreSQL on RDS, due to cloud-optimized architecture.

> [!question]- Q3: How does Aurora storage work?
> **Answer:**
> Auto-grows from ==10 GB to 128 TB== (==256 TiB== on newer Aurora versions like PostgreSQL 16.9+). Stores ==6 copies== across ==3 AZs==. Uses hundreds of volumes with self-healing and peer-to-peer replication.

> [!question]- Q4: How many copies does Aurora need for writes and reads?
> **Answer:**
> - Writes: ==4 out of 6== copies needed
> - Reads: ==3 out of 6== copies needed
> This means Aurora survives losing an entire AZ.

> [!question]- Q5: What is the Aurora Writer Endpoint?
> **Answer:**
> A ==DNS name== that always points to the current master instance. Even if the master fails over, the writer endpoint automatically redirects to the new master.

> [!question]- Q6: What is the Aurora Reader Endpoint?
> **Answer:**
> A ==DNS name== that provides ==connection load balancing== across all read replicas. Load balancing happens at the ==connection level==, not statement level.

> [!question]- Q7: How fast is Aurora failover?
> **Answer:**
> ==Instantaneous== — less than ==30 seconds== on average. Much faster than Multi-AZ failover on regular RDS.

> [!question]- Q8: How many read replicas can Aurora have?
> **Answer:**
> Up to ==15== read replicas with sub ==10ms replica lag==. They can also have ==auto scaling==.

> [!question]- Q9: What is Aurora Backtrack?
> **Answer:**
> A feature that lets you restore data to ==any point in time== without relying on backups. You can go back and forth in time (e.g., "yesterday 4 PM" then change to "5 PM").

> [!question]- Q10: How much more does Aurora cost compared to RDS?
> **Answer:**
> About ==20% more== than RDS, but it's much more efficient at scale due to better performance, auto-scaling storage, and cloud-native architecture.
