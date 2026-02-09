---
title: RDS & Aurora - Backup and Monitoring
date: 2026-02-10
tags:
  - aws
  - rds
  - aurora
  - backup
  - snapshot
  - cloning
  - saa-c03
---

# RDS & Aurora — Backup and Monitoring

## RDS Backups

### Automated Backups

| Feature | Detail |
|---------|--------|
| **Frequency** | ==Daily full backup== during backup window |
| **Transaction logs** | Backed up every ==5 minutes== |
| **Point-in-time restore** | To any time up to ==5 minutes ago== |
| **Retention** | ==1 to 35 days== (set 0 to disable) |

### Manual DB Snapshots

| Feature | Detail |
|---------|--------|
| **Trigger** | ==Manually== by user |
| **Retention** | ==As long as you want== (never expires) |

> [!tip] Cost Saving Trick
> If you use an RDS database only 2 hours/month:
> 1. Use the DB for 2 hours
> 2. Take a ==snapshot==
> 3. ==Delete== the original database
> 4. Restore from snapshot when needed again
>
> Snapshot storage costs ==much less== than keeping the DB running (even stopped, you pay for storage).

## Aurora Backups

| Feature | RDS | Aurora |
|---------|-----|--------|
| **Automated backups** | 1-35 days (can disable with 0) | 1-35 days (==cannot disable==) |
| **Point-in-time restore** | Yes | Yes |
| **Manual snapshots** | Yes, retained indefinitely | Yes, retained indefinitely |

> [!warning] Aurora Difference
> Aurora automated backups ==cannot be disabled== (unlike RDS where you can set retention to 0).

## Restore Options

### From Backup/Snapshot

Restoring always creates a ==new database==.

```
┌──────────────┐     Restore     ┌──────────────┐
│  Snapshot /  │────────────────▶│  NEW Database │
│  Backup      │                 │  (different   │
│              │                 │   endpoint)   │
└──────────────┘                 └──────────────┘
```

### From S3 — RDS MySQL

```
┌──────────────┐     Upload     ┌────────┐     Restore     ┌──────────────┐
│ On-premises  │───────────────▶│   S3   │────────────────▶│  New RDS     │
│ DB backup    │                │        │                 │  MySQL       │
└──────────────┘                └────────┘                 └──────────────┘
```

### From S3 — Aurora MySQL

```
┌──────────────┐   Percona     ┌────────┐     Restore     ┌──────────────┐
│ On-premises  │──XtraBackup──▶│   S3   │────────────────▶│  New Aurora  │
│ DB backup    │               │        │                 │  MySQL       │
└──────────────┘               └────────┘                 └──────────────┘
```

> [!important] Key Difference
> - RDS MySQL restore from S3: ==any backup format==
> - Aurora MySQL restore from S3: must use ==Percona XtraBackup==

## Aurora Database Cloning

| Feature | Detail |
|---------|--------|
| **Purpose** | Create a new Aurora cluster from an existing one |
| **Speed** | ==Faster than snapshot + restore== |
| **Protocol** | ==Copy-on-write== |
| **Initial state** | Uses ==same data volume== as original (no data copying) |
| **After updates** | New storage allocated, data separated |
| **Use case** | Create ==staging DB from production== without impact |

```
Initial (shared volume):          After updates (separated):
┌──────────────┐                  ┌──────────────┐
│  Production  │──shared──┐      │  Production  │──own volume
│  Aurora DB   │          │      │  Aurora DB   │
└──────────────┘          │      └──────────────┘
┌──────────────┐          │      ┌──────────────┐
│  Staging     │──────────┘      │  Staging     │──own volume
│  Aurora DB   │                 │  Aurora DB   │
└──────────────┘                 └──────────────┘
```

> [!tip] Why Cloning?
> ==Fast and cost-effective==. No need to go through snapshot + restore. Perfect for creating staging/test environments from production data.

## Questions & Answers

> [!question]- Q1: How often are RDS automated backups taken?
> **Answer:**
> ==Daily full backup== during the backup window, plus ==transaction logs every 5 minutes==. This enables point-in-time restore to any moment up to 5 minutes ago.

> [!question]- Q2: Can you disable automated backups on Aurora?
> **Answer:**
> ==No==. Aurora automated backups (1-35 days) ==cannot be disabled==. On RDS, you can disable them by setting retention to 0.

> [!question]- Q3: What happens when you restore from a snapshot?
> **Answer:**
> It always creates a ==new database== with a new endpoint. It does not restore into the existing database.

> [!question]- Q4: How can you save costs on an infrequently used RDS database?
> **Answer:**
> Take a ==manual snapshot==, then ==delete the database==. Restore from the snapshot when needed. Snapshot storage costs much less than keeping the DB running.

> [!question]- Q5: What is the difference between restoring RDS MySQL vs Aurora MySQL from S3?
> **Answer:**
> - RDS MySQL: Use ==any backup format== from on-premises
> - Aurora MySQL: Must use ==Percona XtraBackup== format

> [!question]- Q6: What is Aurora Database Cloning?
> **Answer:**
> Creating a new Aurora cluster from an existing one using ==copy-on-write== protocol. Initially shares the same data volume (fast, no copying). New storage is allocated only when updates are made.

> [!question]- Q7: Why is cloning faster than snapshot + restore?
> **Answer:**
> Cloning uses ==copy-on-write==. Initially, the clone shares the ==same data volume== as the original — no data needs to be copied. Storage is only allocated when changes are made.

> [!question]- Q8: What is the backup retention range for RDS?
> **Answer:**
> ==1 to 35 days==. Set to 0 to disable automated backups entirely.

> [!question]- Q9: What is the main use case for Aurora cloning?
> **Answer:**
> Creating a ==staging or test database from production== without impacting the production database and without the overhead of snapshot + restore.

> [!question]- Q10: How long can you retain manual DB snapshots?
> **Answer:**
> ==Indefinitely==. Manual snapshots never expire — you retain them for as long as you want (unlike automated backups which expire after the retention period).
