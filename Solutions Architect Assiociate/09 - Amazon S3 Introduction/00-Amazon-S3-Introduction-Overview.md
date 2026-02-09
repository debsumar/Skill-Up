---
title: Amazon S3 Introduction - Overview
date: 2026-02-10
tags:
  - aws
  - s3
  - storage
  - saa-c03
  - index
---

# Amazon S3 Introduction - Overview

## Study Files

| # | Topic | File | Priority |
|---|-------|------|----------|
| 1 | S3 Overview | [[01-S3-Overview]] | 🔴 High |
| 2 | S3 Security & Bucket Policy | [[02-S3-Security-Bucket-Policy]] | 🔴 High |
| 3 | S3 Static Website Hosting | [[03-S3-Static-Website-Hosting]] | 🟡 Medium |
| 4 | S3 Versioning | [[04-S3-Versioning]] | 🔴 High |
| 5 | S3 Replication | [[05-S3-Replication]] | 🔴 High |
| 6 | S3 Storage Classes | [[06-S3-Storage-Classes]] | 🔴 High |

## Section Purpose

Amazon S3 is one of the ==main building blocks of AWS== — infinitely scaling object storage. Many websites and AWS services use S3 as a backbone.

## Key Concepts at a Glance

```
Amazon S3
├── Buckets (top-level containers)
│   ├── Globally unique name
│   ├── Defined at region level
│   └── Naming: lowercase, 3-63 chars, no IP
├── Objects (files)
│   ├── Key = full path (prefix + object name)
│   ├── Max size: 5 TB
│   ├── Multi-part upload if > 5 GB
│   └── Metadata, Tags, Version ID
├── Security
│   ├── IAM Policies (user-based)
│   ├── Bucket Policies (resource-based, JSON)
│   ├── ACLs (legacy, can be disabled)
│   ├── Block Public Access settings
│   └── Encryption
├── Features
│   ├── Static Website Hosting
│   ├── Versioning (bucket-level)
│   └── Replication (CRR / SRR)
└── Storage Classes
    ├── Standard
    ├── Standard-IA
    ├── One Zone-IA
    ├── Glacier Instant Retrieval
    ├── Glacier Flexible Retrieval
    ├── Glacier Deep Archive
    └── Intelligent-Tiering
```

## S3 Use Cases

| Use Case | Description |
|----------|-------------|
| Backup & Storage | Files, disks, disaster recovery |
| Archival | Glacier for long-term, low-cost storage |
| Hybrid Cloud | Extend on-premises storage to cloud |
| Application Hosting | Host media, images, video |
| Data Lake | Big data analytics |
| Static Websites | Host HTML/CSS/JS sites |
| Software Delivery | Distribute updates |

## Related Sections

- [[../08 - Classic Solutions Architecture Discussions/00-Classic-Solutions-Architecture-Overview|Classic Solutions Architecture]]
- [[../04 - EC2 Instance Storage/01-EBS-Overview|EBS Overview]]
- [[../04 - EC2 Instance Storage/07-Amazon-EFS|Amazon EFS]]
