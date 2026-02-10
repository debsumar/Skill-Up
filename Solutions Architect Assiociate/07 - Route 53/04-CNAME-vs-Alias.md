---
title: "Route 53 - CNAME vs Alias"
date: 2026-02-10
tags:
  - aws
  - route53
  - dns
  - cname
  - alias
  - saa-c03
---

# Route 53 - CNAME vs Alias

## Overview

When you have an AWS resource like a Load Balancer or CloudFront distribution, it exposes a ==hostname== (e.g., `demo-alb-1234.eu-central-1.elb.amazonaws.com`). You want to map that ugly hostname to your own domain (e.g., `myapp.mydomain.com`). You have two options: ==CNAME records== and ==Alias records==.

This is one of the ==most frequently tested topics== on the SAA-C03 exam. Understanding the differences — especially around the zone apex limitation — is critical.

## CNAME Records

A CNAME (Canonical Name) record maps a hostname to ==another hostname==:

```
app.mydomain.com  →  CNAME  →  demo-alb-1234.eu-central-1.elb.amazonaws.com
```

### CNAME Limitations

| Rule | Detail |
|------|--------|
| ==Cannot be used at zone apex== | You CANNOT create a CNAME for `mydomain.com` (only for subdomains like `www.mydomain.com`) |
| Works for any hostname | Can point to any hostname, not just AWS resources |
| ==Costs money== | DNS queries to CNAME records are billed |
| No native health check | Cannot directly evaluate target health |

> [!danger] Zone Apex Restriction
> This is the ==most important thing to remember==: CNAME records ==cannot be created for the zone apex== (also called the "naked domain" or "root domain"). If your domain is `example.com`, you CANNOT create a CNAME for `example.com` — only for subdomains like `www.example.com`, `api.example.com`, etc.
>
> If you try to create a CNAME at the zone apex in Route 53, you'll get: =="Bad request. RRSet of type CNAME is not permitted at apex of zone."==

## Alias Records

Alias records are ==specific to Route 53== — they're an extension to standard DNS. They map a hostname to a ==specific AWS resource==:

```
mydomain.com  →  Alias (A)  →  demo-alb-1234.eu-central-1.elb.amazonaws.com
```

### Alias Advantages

| Feature | Detail |
|---------|--------|
| ==Works at zone apex== | Can be used for `mydomain.com` (root domain) ✅ |
| ==Free of charge== | DNS queries to Alias records are ==not billed== |
| ==Native health check== | Can evaluate target health automatically |
| ==Auto-updating== | If the underlying resource IP changes, the Alias automatically reflects it |
| Always type A or AAAA | Maps to IPv4 or IPv6 |
| ==TTL managed by Route 53== | You cannot set TTL manually |

```
┌──────────────────────────────────────────────────────────────┐
│                    CNAME vs ALIAS                             │
│                                                               │
│  CNAME:                          ALIAS:                      │
│  ┌──────────────────┐           ┌──────────────────┐        │
│  │ www.example.com  │           │ example.com      │        │
│  │ (subdomain only) │           │ (root + subdomain)│        │
│  │                  │           │                  │        │
│  │ → any hostname   │           │ → AWS resource   │        │
│  │ → costs money    │           │ → FREE queries   │        │
│  │ → no health chk  │           │ → health check   │        │
│  │ → manual TTL     │           │ → auto TTL       │        │
│  └──────────────────┘           └──────────────────┘        │
└──────────────────────────────────────────────────────────────┘
```

## Alias Record Targets

Alias records can only point to ==specific AWS resources==:

| Target | Example |
|--------|---------|
| ==Elastic Load Balancer== | ALB, NLB, CLB |
| ==CloudFront Distribution== | CDN distribution |
| ==API Gateway== | REST/HTTP API |
| ==Elastic Beanstalk== | Environment URL |
| ==S3 Website== | S3 bucket configured as static website (not regular S3 bucket!) |
| ==VPC Interface Endpoint== | PrivateLink endpoint |
| ==Global Accelerator== | Accelerator |
| ==Route 53 record== | Another record in the ==same hosted zone== |

> [!warning] What You CANNOT Alias To
> You ==cannot set an Alias for an EC2 instance DNS name==. This is a common exam trick question. EC2 DNS names (like `ec2-54-22-33-44.compute-1.amazonaws.com`) are NOT valid Alias targets. Use an A record with the EC2's IP address instead, or put an ELB in front of it.

## Hands-On: CNAME vs Alias

### Creating a CNAME Record

1. Create record: `myapp.yourdomain.com`
2. Record type: ==CNAME==
3. Value: paste the ALB DNS name
4. This works because `myapp` is a ==subdomain==

### Creating an Alias Record

1. Create record: `myalias.yourdomain.com`
2. Record type: ==A==
3. Toggle ==Alias== on
4. Route traffic to: ==Application and Classic Load Balancer==
5. Select region and load balancer
6. Enable ==Evaluate Target Health==: Yes
7. This is ==free to query== and supports health checks

### The Zone Apex Test

1. Try to create a CNAME for `yourdomain.com` (no subdomain prefix) → ==FAILS== with "CNAME is not permitted at apex"
2. Create an Alias A record for `yourdomain.com` → ==WORKS== ✅
3. Now `yourdomain.com` points directly to your ALB

> [!tip] Exam Decision Tree
> When the exam asks how to point a domain to an AWS resource:
> - Is it the ==zone apex== (root domain like `example.com`)? → ==Must use Alias==
> - Is it a ==subdomain== (like `www.example.com`)? → Can use either CNAME or Alias
> - Do you want ==free queries==? → Use Alias
> - Do you want to point to a ==non-AWS resource==? → Must use CNAME
> - Is the target an ==EC2 instance==? → Cannot use Alias — use A record with IP

## Questions & Answers

> [!question]- Q1: What is the key difference between CNAME and Alias records?
> **Answer:**
> The most critical difference: ==CNAME cannot be used at the zone apex== (root domain like `example.com`), while ==Alias can==. Additionally, Alias records are free to query, support native health checks, and auto-update when the target resource's IP changes. CNAME records can point to any hostname (not just AWS), but cost money per query.

> [!question]- Q2: What is the zone apex and why does it matter?
> **Answer:**
> The zone apex is the ==root of your domain== — `example.com` without any subdomain prefix. DNS standards (RFC) prohibit CNAME records at the zone apex. This matters because many websites want `example.com` (not just `www.example.com`) to point to a load balancer. ==Alias records solve this problem== by allowing you to point the zone apex to AWS resources.

> [!question]- Q3: Are Alias record queries free?
> **Answer:**
> ==Yes.== DNS queries to Alias records pointing to AWS resources are ==not billed==. This is a significant cost advantage over CNAME records, which are billed per query. If you're routing millions of queries, the cost difference is substantial.

> [!question]- Q4: Can you create an Alias record for an EC2 instance?
> **Answer:**
> ==No.== EC2 instance DNS names are ==not valid Alias targets==. You must use a standard A record with the EC2 instance's IP address. Alternatively, put an ELB in front of the EC2 instance and create an Alias to the ELB.

> [!question]- Q5: What AWS resources can be Alias targets?
> **Answer:**
> ELB (ALB/NLB/CLB), CloudFront distributions, API Gateway, Elastic Beanstalk environments, ==S3 websites== (not regular S3 buckets), VPC Interface Endpoints, Global Accelerator, and Route 53 records in the ==same hosted zone==.

> [!question]- Q6: Can you set TTL on an Alias record?
> **Answer:**
> ==No.== The TTL for Alias records is ==automatically managed by Route 53== based on the target resource. You cannot override it. This is different from standard records (A, AAAA, CNAME) where you set TTL manually.

> [!question]- Q7: When should you use CNAME instead of Alias?
> **Answer:**
> Use CNAME when:
> - You need to point to a ==non-AWS resource== (Alias only works with AWS resources)
> - You're pointing a ==subdomain== to an external service
> Use Alias for everything else — it's free, supports health checks, and works at the zone apex.

> [!question]- Q8: What error do you get when trying to create a CNAME at the zone apex?
> **Answer:**
> Route 53 returns: =="Bad request. RRSet of type CNAME is not permitted at apex of zone."== This is enforced by DNS standards. The solution is to use an ==Alias record of type A== instead.

> [!question]- Q9: What record type is an Alias record?
> **Answer:**
> Alias records are always of type ==A== (IPv4) or ==AAAA== (IPv6). They look like standard A/AAAA records to DNS clients, but internally Route 53 resolves them to the target AWS resource's IP addresses. This is why they work at the zone apex — they're technically A records, not CNAMEs.

> [!question]- Q10: Exam scenario — you need example.com to point to an ALB. What do you use?
> **Answer:**
> ==Alias record of type A==. You cannot use CNAME because `example.com` is the zone apex. Create an A record with Alias enabled, route traffic to "Application and Classic Load Balancer," select the region and ALB. This is free to query and supports health check evaluation.
