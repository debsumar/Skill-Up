---
title: "Amazon S3 Security - Section Overview"
date: 2026-02-10
tags:
  - aws
  - s3
  - security
  - encryption
  - saa-c03
---

# Amazon S3 Security - Section Overview

## Section Map

This section covers Amazon S3 security features building on the fundamentals from [[09 - Amazon S3 Introduction/00-Amazon-S3-Introduction-Overview|S3 Introduction]] and advanced topics from [[10 - Advanced Amazon S3/00-Advanced-S3-Overview|Advanced S3]]. Security is one of the ==most heavily tested S3 topics== on the SAA-C03 exam.

| # | Topic | Key Concepts |
|---|-------|-------------|
| [[01-S3-Encryption\|01]] | S3 Encryption | SSE-S3, SSE-KMS, SSE-C, client-side encryption, encryption in transit, DSSE-KMS, default encryption |
| [[02-S3-CORS\|02]] | S3 CORS | Cross-Origin Resource Sharing, preflight requests, Access-Control-Allow-Origin headers |
| [[03-S3-MFA-Delete\|03]] | S3 MFA Delete | Multi-factor authentication for permanent deletions, root account requirement |
| [[04-S3-Access-Logs\|04]] | S3 Access Logs | Server access logging, audit trail, logging loop warning |
| [[05-S3-Pre-signed-URLs\|05]] | S3 Pre-signed URLs | Temporary access, URL expiration, GET/PUT use cases |
| [[06-Glacier-Vault-Lock-and-S3-Object-Lock\|06]] | Glacier Vault Lock & S3 Object Lock | WORM model, compliance mode, governance mode, legal hold, retention periods |
| [[07-S3-Access-Points\|07]] | S3 Access Points | Simplify security management, access point policies, VPC origin |
| [[08-S3-Object-Lambda\|08]] | S3 Object Lambda | Modify objects on retrieval, redact PII, enrich data, Lambda access points |

## Key Exam Topics

- ==Encryption types== — know when to use SSE-S3 vs SSE-KMS vs SSE-C vs client-side
- ==KMS throttling== — SSE-KMS counts against KMS API quotas (5,500–30,000 req/s)
- ==CORS== — must enable correct CORS headers on the target S3 bucket for cross-origin requests
- ==MFA Delete== — only root account can enable/disable, requires versioning
- ==Pre-signed URLs== — temporary access to private objects (console: 12h, CLI: 168h)
- ==Object Lock== — compliance mode (nobody can delete) vs governance mode (admins can override)
- ==Access Points== — simplify bucket policy management at scale
- ==S3 Object Lambda== — transform objects on retrieval without duplicating buckets
