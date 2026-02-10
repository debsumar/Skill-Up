---
title: "Advanced Amazon S3 - Section Overview"
date: 2026-02-10
tags:
  - aws
  - s3
  - advanced
  - saa-c03
---

# Advanced Amazon S3 - Section Overview

## Section Map

This section covers advanced Amazon S3 features beyond the basics covered in [[09 - Amazon S3 Introduction/00-Amazon-S3-Introduction-Overview|S3 Introduction]]. These topics are heavily tested on the SAA-C03 exam.

| # | Topic | Key Concepts |
|---|-------|-------------|
| [[01-S3-Lifecycle-Rules\|01]] | S3 Lifecycle Rules | Transition actions, expiration actions, S3 Analytics, prefix/tag filters, exam scenarios |
| [[02-S3-Event-Notifications\|02]] | S3 Event Notifications | SNS, SQS, Lambda targets, EventBridge integration, resource policies, hands-on |
| [[03-S3-Performance\|03]] | S3 Performance | Baseline performance, prefixes, multi-part upload, transfer acceleration, byte-range fetches |
| [[04-S3-Requester-Pays-and-Batch-Operations\|04]] | Requester Pays & Batch Operations | Requester Pays buckets, S3 Batch Operations, S3 Inventory, Athena |
| [[05-S3-Storage-Lens\|05]] | S3 Storage Lens | Organization-wide analytics, default dashboard, free vs advanced metrics, cost optimization |

## Key Exam Topics

- ==Lifecycle rules== — transition actions (move between storage classes) and expiration actions (delete objects)
- ==Event Notifications== — react to S3 events via SNS, SQS, Lambda, or ==EventBridge==
- ==Performance== — 3,500 PUT/5,500 GET per prefix, multi-part upload, transfer acceleration
- ==Batch Operations== — bulk encrypt, copy, tag objects using S3 Inventory + Athena
- ==Storage Lens== — organization-wide S3 analytics and cost optimization
