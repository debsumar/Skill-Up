---
title: RDS Custom
date: 2026-02-10
tags:
  - aws
  - rds
  - rds-custom
  - oracle
  - sql-server
  - saa-c03
---

# RDS Custom for Oracle & Microsoft SQL Server

## RDS vs RDS Custom

| Feature | RDS | RDS Custom |
|---------|-----|------------|
| **OS access** | ==No== | ==Yes== |
| **DB customization** | No | Yes |
| **SSH / SSM access** | No | ==Yes== (SSH or SSM Session Manager) |
| **Supported engines** | All 7 | ==Oracle & MS SQL Server only== |
| **Managed benefits** | Full | Full (setup, operations, scaling) |
| **Internal settings** | No access | Can configure |
| **Patches** | AWS managed | Can install custom patches |
| **Native features** | No access | Can enable |

## Key Points

- RDS Custom gives you ==full admin access== to the underlying OS and database
- Supports ==only Oracle and Microsoft SQL Server==
- You still get all RDS managed benefits (automated setup, operations, scaling)
- Access via ==SSH or SSM Session Manager==

> [!warning] Before Customizing
> 1. ==Deactivate automation mode== — prevents RDS from running maintenance/scaling during your changes
> 2. ==Take a DB snapshot== — you may break things since you have OS-level access

## Questions & Answers

> [!question]- Q1: What is RDS Custom?
> **Answer:**
> A version of RDS that gives you ==full admin access== to the underlying OS and database. Available only for ==Oracle and Microsoft SQL Server==.

> [!question]- Q2: Which database engines support RDS Custom?
> **Answer:**
> Only ==Oracle== and ==Microsoft SQL Server==.

> [!question]- Q3: Can you SSH into an RDS Custom instance?
> **Answer:**
> ==Yes==. You can access it via ==SSH or SSM Session Manager==. This is the exception to the "no SSH in RDS" rule.

> [!question]- Q4: What should you do before performing customizations on RDS Custom?
> **Answer:**
> 1. ==Deactivate automation mode== to prevent RDS from interfering
> 2. ==Take a DB snapshot== for recovery in case something breaks

> [!question]- Q5: Do you still get managed benefits with RDS Custom?
> **Answer:**
> ==Yes==. You still get automated setup, operations, and scaling — plus the ability to customize the OS and database.

> [!question]- Q6: What can you do with RDS Custom that you can't with regular RDS?
> **Answer:**
> - Configure ==internal database settings==
> - Install ==custom patches==
> - Enable ==native features==
> - Access the ==underlying EC2 instance==

> [!question]- Q7: Why would you choose RDS Custom over regular RDS?
> **Answer:**
> When you need ==OS-level or database-level customization== for Oracle or MS SQL Server workloads that require specific patches, native features, or internal configuration changes.

> [!question]- Q8: What is the risk of using RDS Custom?
> **Answer:**
> Since you have ==full OS access==, you could potentially ==break things==. Always take a snapshot before making changes.

> [!question]- Q9: Can you use RDS Custom with MySQL or PostgreSQL?
> **Answer:**
> ==No==. RDS Custom is only available for ==Oracle and Microsoft SQL Server==.

> [!question]- Q10: How does RDS Custom differ from running a DB on EC2?
> **Answer:**
> RDS Custom still provides ==managed service benefits== (automated setup, operations, scaling, backups) while giving you OS access. Running on EC2 means you manage ==everything== yourself.
