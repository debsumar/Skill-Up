---
title: RDS & Aurora Security
date: 2026-02-10
tags:
  - aws
  - rds
  - aurora
  - security
  - encryption
  - iam
  - saa-c03
---

# RDS & Aurora Security

## At-Rest Encryption

| Feature | Detail |
|---------|--------|
| **Method** | ==KMS== encryption on volumes |
| **When** | Defined at ==launch time== |
| **Replicas** | If master is ==not encrypted==, replicas ==cannot be encrypted== |
| **Encrypt existing DB** | Snapshot → Restore as ==encrypted== |

```
Encrypt an unencrypted DB:
┌──────────────┐  snapshot  ┌──────────┐  restore as  ┌──────────────┐
│ Unencrypted  │──────────▶│ Snapshot │──encrypted──▶│  Encrypted   │
│     DB       │           │          │              │     DB       │
└──────────────┘           └──────────┘              └──────────────┘
```

## In-Flight Encryption

| Feature | Detail |
|---------|--------|
| **Support** | ==Ready by default== on all RDS/Aurora databases |
| **Requirement** | Clients must use ==TLS root certificates== from AWS |

## Authentication

| Method | Detail |
|--------|--------|
| **Username/Password** | Classic approach |
| **IAM Roles** | EC2 instances with IAM roles can authenticate ==directly== (no password) |

> [!tip] IAM Authentication
> If your EC2 instances have IAM roles, they can connect to RDS/Aurora ==without a username and password==. Manage all security within IAM.

## Network Security

- Control access via ==Security Groups==
- Allow/block specific ports, IPs, or security groups

## SSH Access

| Service | SSH Access |
|---------|-----------|
| RDS | ==No== |
| Aurora | ==No== |
| RDS Custom | ==Yes== (Oracle & MS SQL Server only) |

## Audit Logs

- Enable ==Audit Logs== to track queries and database activity
- Logs are ==lost after a period== unless sent to ==CloudWatch Logs== for long-term retention

## Questions & Answers

> [!question]- Q1: How do you encrypt an RDS database at rest?
> **Answer:**
> Using ==KMS encryption==. Must be defined at ==launch time==. If the master is not encrypted, read replicas cannot be encrypted either.

> [!question]- Q2: How do you encrypt an already existing unencrypted database?
> **Answer:**
> Take a ==snapshot== of the unencrypted DB, then ==restore== that snapshot as an ==encrypted database==.

> [!question]- Q3: Is in-flight encryption supported by default?
> **Answer:**
> ==Yes==. All RDS and Aurora databases are ready for in-flight encryption. Clients must use ==TLS root certificates== provided by AWS.

> [!question]- Q4: Can EC2 instances authenticate to RDS using IAM?
> **Answer:**
> ==Yes==. EC2 instances with IAM roles can authenticate directly to RDS/Aurora without a username and password.

> [!question]- Q5: Can you SSH into RDS or Aurora?
> **Answer:**
> ==No==. They are managed services. The only exception is ==RDS Custom== (Oracle and MS SQL Server).

> [!question]- Q6: How do you retain RDS audit logs long-term?
> **Answer:**
> Send them to ==CloudWatch Logs==. Without this, audit logs are lost after a period of time.

> [!question]- Q7: What controls network access to RDS/Aurora?
> **Answer:**
> ==Security Groups==. You can allow or block specific ports, IP addresses, or other security groups.

> [!question]- Q8: If the master DB is unencrypted, can read replicas be encrypted?
> **Answer:**
> ==No==. If the master is not encrypted, the read replicas ==cannot be encrypted==.

> [!question]- Q9: What are the three authentication methods for RDS?
> **Answer:**
> 1. ==Username and password==
> 2. ==IAM roles== (for EC2 instances)
> 3. ==Kerberos== (external authentication)

> [!question]- Q10: Where do you get TLS certificates for RDS in-flight encryption?
> **Answer:**
> From the ==AWS website==. AWS provides TLS root certificates that clients must use to establish encrypted connections.
