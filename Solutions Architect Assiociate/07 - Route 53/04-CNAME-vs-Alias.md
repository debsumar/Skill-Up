---
title: CNAME vs Alias
date: 2026-02-10
tags:
  - aws
  - route53
  - cname
  - alias
  - saa-c03
---

# CNAME vs Alias

## The Problem

AWS resources (ALB, CloudFront) expose a hostname (e.g., `lb-1234.us-east-1.elb.amazonaws.com`). You want to map it to `myapp.mydomain.com`.

## Comparison

| Feature | CNAME | Alias |
|---------|-------|-------|
| **Maps to** | Any hostname | ==AWS resources only== |
| **Zone apex** | ==No== (can't use for root domain) | ==Yes== (works for root domain) |
| **Cost** | Charged per query | ==Free== |
| **Health check** | No native support | ==Yes== (native) |
| **TTL** | You set it | ==Auto-set== by Route 53 |
| **Record type** | CNAME | ==A or AAAA== |
| **Example** | `www.example.com` → `lb.aws.com` | `example.com` → ALB |

> [!important] Zone Apex (Exam Critical)
> - CNAME ==cannot== be used for `example.com` (zone apex)
> - Alias ==can== be used for `example.com` (zone apex)
> - If the exam asks about mapping root domain to an ALB → ==Alias record==

## Alias Record Details

- Extension to DNS functionality (==Route 53 specific==)
- Always of type ==A or AAAA==
- TTL is ==automatically set== by Route 53 (cannot be changed)
- If underlying resource IP changes, alias ==automatically recognizes== it

## Alias Targets

| Target | Supported? |
|--------|------------|
| Elastic Load Balancers | ✅ |
| CloudFront Distributions | ✅ |
| API Gateway | ✅ |
| Elastic Beanstalk | ✅ |
| S3 Websites | ✅ (not S3 buckets, only ==website-enabled==) |
| VPC Interface Endpoints | ✅ |
| Global Accelerator | ✅ |
| Route 53 record (same hosted zone) | ✅ |
| **EC2 DNS name** | ==❌ No== |

> [!warning] Cannot Alias to EC2
> You ==cannot== set an Alias record targeting an ==EC2 instance DNS name==. This is a common exam question.

## Questions & Answers

> [!question]- Q1: Can you use a CNAME for the root domain (zone apex)?
> **Answer:**
> ==No==. CNAME cannot be used for the zone apex (e.g., `example.com`). Use an ==Alias record== instead.

> [!question]- Q2: What record type should you use to map example.com to an ALB?
> **Answer:**
> ==Alias record== of type A. CNAME won't work because it's the zone apex.

> [!question]- Q3: Are Alias record queries free?
> **Answer:**
> ==Yes==. Alias queries to AWS resources are free of charge.

> [!question]- Q4: Can you set TTL on an Alias record?
> **Answer:**
> ==No==. TTL is automatically set by Route 53 for Alias records.

> [!question]- Q5: What record types can Alias records be?
> **Answer:**
> ==A or AAAA== only (IPv4 or IPv6).

> [!question]- Q6: Can you create an Alias to an EC2 instance DNS name?
> **Answer:**
> ==No==. EC2 DNS names are ==not supported== as Alias targets.

> [!question]- Q7: What happens if the ALB IP changes behind an Alias record?
> **Answer:**
> The Alias record ==automatically recognizes== the new IP. No manual update needed.

> [!question]- Q8: What is the key advantage of Alias over CNAME?
> **Answer:**
> Alias works for the ==zone apex== (root domain), is ==free==, supports ==health checks==, and auto-updates when the target IP changes.

> [!question]- Q9: Can you Alias to an S3 bucket?
> **Answer:**
> Only if the bucket is ==enabled as a website== (S3 Website endpoint). Regular S3 bucket endpoints are not supported.

> [!question]- Q10: Is Alias a standard DNS feature?
> **Answer:**
> ==No==. It's an ==extension to DNS== specific to Route 53. It's not available on other DNS providers.