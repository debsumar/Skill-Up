---
title: "S3 Storage Lens"
date: 2026-02-10
tags:
  - aws
  - s3
  - storage-lens
  - analytics
  - cost-optimization
  - data-protection
  - saa-c03
---

# S3 Storage Lens

## Overview

S3 Storage Lens is an ==organization-wide analytics service== that helps you understand, analyze, and optimize your S3 storage. It provides a ==single pane of glass== across all your accounts, regions, and buckets — making it easy to discover anomalies, identify cost savings, and enforce data protection best practices.

Think of it as a ==dashboard for your entire S3 estate==. While [[01-S3-Lifecycle-Rules#S3 Analytics — Storage Class Analysis|S3 Analytics]] focuses on access patterns for individual buckets to recommend Standard → Standard-IA transitions, Storage Lens gives you a ==bird's-eye view== of your entire organization's S3 usage across every account and region.

Storage Lens provides ==30-day usage and activity metrics==, which means you get a rolling window of data to spot trends, anomalies, and optimization opportunities. It's designed for organizations that manage S3 storage at scale — dozens of accounts, hundreds of buckets, multiple regions.

```
┌──────────────────────────────────────────────────────────────┐
│              S3 STORAGE LENS — BIG PICTURE                    │
│                                                               │
│  ┌──────────────────────────────────────────────────────┐    │
│  │  Data Sources (your entire AWS Organization)         │    │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐            │    │
│  │  │ Account 1│ │ Account 2│ │ Account 3│  ...       │    │
│  │  │ Region A │ │ Region B │ │ Region C │            │    │
│  │  │ 50 Bkts  │ │ 30 Bkts  │ │ 80 Bkts  │            │    │
│  │  └────┬─────┘ └────┬─────┘ └────┬─────┘            │    │
│  └───────┼─────────────┼───────────┼───────────────────┘    │
│          └─────────────┼───────────┘                         │
│                        ▼                                     │
│              ┌──────────────────┐                            │
│              │  Storage Lens    │                            │
│              │  Aggregation     │                            │
│              │  Engine          │                            │
│              └────────┬─────────┘                            │
│                       │                                      │
│          ┌────────────┼────────────┐                        │
│          ▼            ▼            ▼                        │
│  ┌──────────┐ ┌──────────┐ ┌──────────────┐               │
│  │ Summary  │ │ Cost     │ │ Data         │               │
│  │ Insights │ │ Optimize │ │ Protection   │               │
│  └──────────┘ └──────────┘ └──────────────┘               │
│          │            │            │                        │
│          └────────────┼────────────┘                        │
│                       ▼                                     │
│              ┌──────────────────┐                            │
│              │  Dashboard       │                            │
│              │  (Default or     │                            │
│              │   Custom)        │                            │
│              └────────┬─────────┘                            │
│                       │                                      │
│                       ▼                                      │
│              ┌──────────────────┐                            │
│              │  Export to S3    │                            │
│              │  (CSV / Parquet) │                            │
│              └──────────────────┘                            │
└──────────────────────────────────────────────────────────────┘
```

## Key Features

| Feature | Detail |
|---------|--------|
| **Scope** | Entire ==AWS Organization==, specific accounts, regions, buckets, or prefixes (advanced) |
| **Metrics** | ==30-day== rolling window of usage and activity metrics |
| **Dashboard** | Default dashboard (pre-configured, cannot delete) + custom dashboards |
| **Export** | ==CSV or Parquet== format to an S3 bucket for external analysis |
| **Aggregation** | Organization, account, region, bucket, or prefix level |
| **Integration** | ==CloudWatch== publishing (advanced metrics only) |

## Default Dashboard

Storage Lens provides a ==pre-configured default dashboard== that you get automatically — no setup required:

| Aspect | Detail |
|--------|--------|
| **Scope** | Data across ==all regions and all accounts== in your organization |
| **Pre-configured** | Set up by Amazon S3 — no configuration needed |
| **Cannot delete** | You can ==disable== it but ==cannot delete== it |
| **Metrics shown** | Total storage, object count, average object size, bucket count, account count |
| **Drill-down** | Per account, per region, per bucket, per storage class |
| **Trends** | Shows trends over time — is storage growing? Which accounts are growing fastest? |

### What the Default Dashboard Shows

The dashboard UI provides a centralized view where you can:

1. ==Select filters==: Choose specific regions, accounts, buckets, or storage classes
2. ==View summary metrics==: Total storage bytes, total object count, average object size
3. ==Drill down==: Click into specific accounts to see their storage breakdown by region and bucket
4. ==Spot trends==: See how storage is growing or shrinking over the 30-day window
5. ==Identify outliers==: Find the largest buckets, fastest-growing accounts, or buckets with the most incomplete uploads

```
┌──────────────────────────────────────────────────────────────┐
│  DEFAULT DASHBOARD — EXAMPLE VIEW                            │
│                                                               │
│  Filters: [All Regions ▼] [All Accounts ▼] [All Classes ▼]  │
│                                                               │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌──────────┐ │
│  │ Total      │ │ Object     │ │ Avg Object │ │ Bucket   │ │
│  │ Storage    │ │ Count      │ │ Size       │ │ Count    │ │
│  │ 45.2 TB    │ │ 12.5M      │ │ 3.8 MB     │ │ 160      │ │
│  └────────────┘ └────────────┘ └────────────┘ └──────────┘ │
│                                                               │
│  Top Accounts by Storage:                                    │
│  ┌──────────────────────────────────────────────────┐       │
│  │ Account 1 (Production)  ████████████████  28.1 TB│       │
│  │ Account 2 (Analytics)   ████████          12.3 TB│       │
│  │ Account 3 (Dev/Test)    ██                 3.1 TB│       │
│  │ Account 4 (Backup)      █                  1.7 TB│       │
│  └──────────────────────────────────────────────────┘       │
│                                                               │
│  Top Regions by Storage:                                     │
│  ┌──────────────────────────────────────────────────┐       │
│  │ us-east-1              ██████████████████  32.0 TB│       │
│  │ eu-west-1              ████████            10.5 TB│       │
│  │ ap-southeast-1         ██                   2.7 TB│       │
│  └──────────────────────────────────────────────────┘       │
└──────────────────────────────────────────────────────────────┘
```

### Custom Dashboards

Beyond the default dashboard, you can create ==custom dashboards== with:
- Specific account filters (e.g., only production accounts)
- Specific region filters (e.g., only EU regions for GDPR compliance)
- Specific bucket filters (e.g., only data lake buckets)
- Different metric selections (e.g., focus only on data protection metrics)

Custom dashboards let you create ==purpose-built views== for different teams — a security team dashboard focused on encryption and versioning, a finance team dashboard focused on cost optimization, etc.

## Metric Categories

Storage Lens organizes metrics into ==seven categories== (plus status codes) that map to specific optimization goals. Understanding these categories helps you know ==when Storage Lens is the right answer== on the exam.

### 1. Summary Metrics

| Metric | What It Tells You |
|--------|-------------------|
| ==Storage Bytes== | Total size of your S3 storage across all selected accounts/regions |
| ==Object Count== | Total number of objects across all selected buckets |
| **Use case** | Identify ==fastest-growing== buckets/prefixes or ==unused== buckets with stale data |

If storage bytes for a bucket remain flat for months, that bucket may contain stale data that could be archived or deleted. If a bucket is growing rapidly, it may need lifecycle rules or a storage class review.

### 2. Cost Optimization Metrics

| Metric | What It Tells You |
|--------|-------------------|
| ==Non-current Version Storage Bytes== | How much space old versions of objects consume |
| ==Incomplete Multi-part Upload Bytes== | Storage wasted on abandoned/failed uploads |
| **Use case** | Find buckets with ==failed uploads== or ==excessive old versions== to clean up with [[01-S3-Lifecycle-Rules|lifecycle rules]] |

These metrics directly translate to ==cost savings==. If you find 500 GB of incomplete multi-part uploads, that's storage you're paying for with zero value. A lifecycle rule to delete incomplete uploads after 7 days solves this immediately.

### 3. Data Protection Metrics

| Metric | What It Tells You |
|--------|-------------------|
| ==Versioning-enabled Bucket Count== | How many buckets have versioning turned on |
| ==MFA Delete-enabled Bucket Count== | How many buckets require MFA for permanent deletes |
| ==SSE-KMS Enabled Bucket Count== | How many buckets use KMS encryption |
| ==Cross-Region Replication Rule Count== | How many buckets have CRR configured |
| **Use case** | Identify buckets ==not following data protection best practices== |

This is ==critical for compliance==. If your organization requires all buckets to have versioning and encryption, Storage Lens tells you exactly which buckets are non-compliant — across every account and region.

### 4. Access Management Metrics

| Metric | What It Tells You |
|--------|-------------------|
| ==Object Ownership Settings== | Which ownership model each bucket uses (BucketOwnerEnforced, BucketOwnerPreferred, ObjectWriter) |
| **Use case** | Audit bucket ownership settings across the organization — ensure all buckets use ==BucketOwnerEnforced== (recommended) |

### 5. Event Metrics

| Metric | What It Tells You |
|--------|-------------------|
| ==Event Notification Configured Bucket Count== | How many buckets have [[02-S3-Event-Notifications|event notifications]] configured |
| **Use case** | Ensure critical buckets have event notifications for monitoring and automation |

### 6. Performance Metrics

| Metric | What It Tells You |
|--------|-------------------|
| ==Transfer Acceleration Enabled Bucket Count== | How many buckets use [[03-S3-Performance#Optimizing Uploads S3 Transfer Acceleration|Transfer Acceleration]] |
| **Use case** | Identify buckets that could benefit from Transfer Acceleration for cross-region uploads |

### 7. Activity Metrics (Advanced Only)

| Metric | What It Tells You |
|--------|-------------------|
| ==All Requests==, ==GET Requests==, ==PUT Requests== | Request volume per bucket — how actively each bucket is being used |
| ==Bytes Downloaded==, ==Bytes Uploaded== | Data transfer volume — which buckets have the most traffic |
| ==HTTP Status Codes== (200, 403, 404, 500) | Error rates and access patterns — are users getting denied? Are objects missing? |
| **Use case** | Understand ==usage patterns==, identify ==anomalies== (sudden spike in 403s = potential security issue), find ==unused buckets== (zero requests) |

> [!important] Activity Metrics Are Advanced (Paid) Only
> Activity metrics (request counts, bytes transferred, status codes) are only available with ==advanced metrics==. Free metrics only include usage metrics (storage bytes, object count, etc.). If the exam asks about monitoring S3 ==request patterns== or ==error rates== across an organization, the answer is Storage Lens with ==advanced metrics==.

## Free vs Advanced (Paid) Metrics

This is a ==key exam distinction==. Know the differences:

```
┌─────────────────────────────────────────────────────────────────┐
│              FREE vs ADVANCED METRICS                           │
│                                                                 │
│  FREE METRICS                    ADVANCED METRICS (Paid)       │
│  ┌──────────────────────┐       ┌──────────────────────┐      │
│  │ ~28 usage metrics    │       │ All free metrics +   │      │
│  │                      │       │                      │      │
│  │ Summary metrics      │       │ Activity metrics     │      │
│  │ Cost optimization    │       │ (requests, bytes,    │      │
│  │ Data protection      │       │  status codes)       │      │
│  │ Access management    │       │                      │      │
│  │ Event metrics        │       │ Advanced cost optim. │      │
│  │ Performance metrics  │       │ Advanced data prot.  │      │
│  │                      │       │                      │      │
│  │ Data available for   │       │ Data available for   │      │
│  │ queries: ==14 days== │       │ queries: ==15 months=│      │
│  │                      │       │                      │      │
│  │ No CloudWatch        │       │ ==Publish to         │      │
│  │                      │       │  CloudWatch==        │      │
│  │ No prefix-level      │       │ ==Prefix-level       │      │
│  │                      │       │  metrics==           │      │
│  │ Automatically        │       │ Additional charge    │      │
│  │ available to ALL     │       │ per metric per       │      │
│  │ customers            │       │ monitored object     │      │
│  └──────────────────────┘       └──────────────────────┘      │
└─────────────────────────────────────────────────────────────────┘
```

| Feature | Free | Advanced (Paid) |
|---------|------|-----------------|
| **Metrics** | ~28 usage metrics (summary, cost, data protection, access, event, performance) | All free + ==activity metrics==, ==advanced cost optimization==, ==advanced data protection==, ==HTTP status codes== |
| **Data retention** | ==14 days== | ==15 months== |
| **CloudWatch integration** | No | ==Yes== — publish metrics to CloudWatch for alarms and dashboards |
| **Prefix-level metrics** | No | ==Yes== — drill down to individual prefix level within buckets |
| **Availability** | All customers automatically | Additional charge |

### When You Need Advanced Metrics

| Scenario | Free Enough? | Need Advanced? |
|----------|-------------|---------------|
| "How much total storage do we have?" | ✅ Free | |
| "Which buckets lack encryption?" | ✅ Free | |
| "How many GET requests per bucket?" | | ✅ Advanced (activity metrics) |
| "What's the 403 error rate?" | | ✅ Advanced (status codes) |
| "Show me prefix-level storage breakdown" | | ✅ Advanced (prefix-level) |
| "Set a CloudWatch alarm on storage growth" | | ✅ Advanced (CloudWatch publishing) |
| "Show me 6-month storage trends" | | ✅ Advanced (15-month retention vs 14-day) |

## CloudWatch Integration (Advanced)

With advanced metrics, Storage Lens can ==publish metrics to CloudWatch==. This unlocks:

- ==CloudWatch Alarms==: Alert when total storage exceeds a threshold, or when a bucket's error rate spikes
- ==CloudWatch Dashboards==: View S3 metrics alongside EC2, RDS, Lambda metrics in a unified dashboard
- ==CloudWatch Metrics Math==: Calculate derived metrics (e.g., cost per GB, growth rate)

This is powerful for organizations that already use CloudWatch as their central monitoring platform — S3 Storage Lens metrics integrate seamlessly.

## Exporting Metrics

Storage Lens can export metrics to an S3 bucket in ==CSV or Parquet== format:

| Feature | Detail |
|---------|--------|
| **Formats** | ==CSV== (human-readable, works with Excel) or ==Parquet== (columnar, efficient for Athena/analytics) |
| **Destination** | Any S3 bucket (same or different account) |
| **Use case** | Analyze with ==Athena==, visualize with ==QuickSight==, feed into custom analytics pipelines |
| **Frequency** | Daily export |

This is useful when you need to do ==custom analysis== beyond what the dashboard provides — for example, building a cost allocation report by team, or tracking storage growth trends over months.

## Storage Lens vs S3 Analytics vs CloudWatch S3 Metrics

| Feature | Storage Lens | S3 Analytics | CloudWatch S3 Metrics |
|---------|-------------|-------------|----------------------|
| **Scope** | ==Organization-wide== (all accounts, regions) | ==Per bucket== | ==Per bucket== |
| **Purpose** | Holistic storage analytics and optimization | Recommend Standard → Standard-IA transitions | Real-time operational metrics |
| **Metrics** | Usage, cost, data protection, activity | Access pattern analysis | Request count, latency, errors |
| **Output** | Dashboard + CSV/Parquet export | CSV report | CloudWatch metrics/alarms |
| **Best for** | ==Organization-level visibility== | ==Lifecycle rule optimization== | ==Real-time monitoring== |

> [!tip] Exam Tip — When to Choose Storage Lens
> If the exam asks about:
> - ==Organization-wide S3 analytics== → Storage Lens
> - ==Identifying unencrypted buckets across accounts== → Storage Lens (data protection metrics)
> - ==S3 cost optimization across an organization== → Storage Lens (cost optimization metrics)
> - ==Monitoring S3 request patterns across accounts== → Storage Lens with advanced metrics
> Remember: default dashboard covers all regions and accounts, free metrics retained for ==14 days==, paid for ==15 months==.

## Questions & Answers

> [!question]- Q1: What is S3 Storage Lens?
> **Answer:**
> S3 Storage Lens is an ==organization-wide analytics service== that provides a single view of your S3 storage across all accounts, regions, and buckets. It provides ==30-day usage and activity metrics== to help you discover anomalies, identify cost savings, and enforce data protection best practices. It aggregates data at the organization, account, region, bucket, or prefix level (prefix requires advanced metrics).

> [!question]- Q2: What is the default Storage Lens dashboard?
> **Answer:**
> A ==pre-configured dashboard== provided by Amazon S3 that shows data across ==all regions and all accounts== in your organization. It displays total storage, object count, average object size, bucket count, and more with drill-down per account and region. You ==cannot delete it== but can ==disable== it. No configuration needed — it's available automatically to all customers. You can also create custom dashboards alongside it.

> [!question]- Q3: What is the difference between free and advanced Storage Lens metrics?
> **Answer:**
> - ==Free==: ~28 usage metrics (summary, cost optimization, data protection, access management, event, performance), data retained for ==14 days==, available to all customers automatically
> - ==Advanced (Paid)==: All free metrics + ==activity metrics== (requests, bytes, status codes), ==advanced cost optimization==, ==advanced data protection==. Data retained for ==15 months==. Includes ==CloudWatch publishing== and ==prefix-level metrics==.

> [!question]- Q4: How can Storage Lens help with cost optimization?
> **Answer:**
> Storage Lens provides ==cost optimization metrics== that show:
> - ==Non-current version storage bytes== — how much space old versions consume (clean up with lifecycle rules)
> - ==Incomplete multi-part upload bytes== — storage wasted on abandoned uploads (clean up with lifecycle rules)
> - Objects that could be transitioned to ==lower-cost storage classes==
> These metrics directly translate to cost savings. For example, finding 500 GB of incomplete uploads means you're paying for storage with zero value.

> [!question]- Q5: How can Storage Lens help with data protection compliance?
> **Answer:**
> Storage Lens provides ==data protection metrics== that identify across your entire organization:
> - Buckets ==without versioning== enabled
> - Buckets ==without MFA Delete==
> - Buckets ==without encryption== (SSE-KMS)
> - Buckets ==without cross-region replication==
> This helps you find buckets that don't follow your organization's data protection policies — critical for compliance audits and security reviews.

> [!question]- Q6: Can Storage Lens export data for external analysis?
> **Answer:**
> ==Yes.== Storage Lens can export metrics daily to an S3 bucket in ==CSV or Parquet== format. CSV is human-readable and works with Excel. Parquet is columnar and efficient for querying with ==Athena== or visualizing with ==QuickSight==. This allows custom analysis beyond what the dashboard provides — cost allocation reports, growth trend analysis, etc.

> [!question]- Q7: At what levels can Storage Lens aggregate data?
> **Answer:**
> Storage Lens can aggregate data at:
> - ==Organization level== — across all accounts in your AWS Organization
> - ==Account level== — specific accounts
> - ==Region level== — specific regions
> - ==Bucket level== — specific buckets
> - ==Prefix level== — specific prefixes within buckets (==advanced metrics only==)
> The default dashboard shows organization-level data with drill-down to account and region.

> [!question]- Q8: Can you delete the default Storage Lens dashboard?
> **Answer:**
> ==No.== The default dashboard cannot be deleted — it's a permanent fixture provided by Amazon S3. You can only ==disable== it if you don't want to see it. You can create ==custom dashboards== with different filters, metrics, and configurations alongside the default dashboard. Custom dashboards can be deleted.

> [!question]- Q9: How does Storage Lens integrate with CloudWatch?
> **Answer:**
> With ==advanced (paid) metrics==, Storage Lens can ==publish metrics to CloudWatch==. This enables:
> - ==CloudWatch Alarms== — alert when storage exceeds a threshold or error rates spike
> - ==CloudWatch Dashboards== — view S3 metrics alongside other AWS service metrics
> - ==Metrics Math== — calculate derived metrics like cost per GB or growth rate
> This is only available with advanced metrics — free metrics cannot be published to CloudWatch.

> [!question]- Q10: Exam scenario — you need to identify all S3 buckets across your organization that don't have encryption enabled. What service do you use?
> **Answer:**
> ==S3 Storage Lens.== It provides ==data protection metrics== across your entire AWS Organization, including ==SSE-KMS enabled bucket count==. The default dashboard shows this across all accounts and regions with no configuration needed. For more detail, use advanced metrics to drill down to specific accounts, regions, or even prefixes. This is a ==free metric== — no need for advanced/paid tier to see encryption status.
