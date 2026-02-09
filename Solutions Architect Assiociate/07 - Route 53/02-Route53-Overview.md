---
title: Route 53 Overview
date: 2026-02-10
tags:
  - aws
  - route53
  - dns
  - hosted-zones
  - saa-c03
---

# Route 53 Overview

## What is Route 53?

- ==Highly available, scalable, fully managed, authoritative DNS==
- **Authoritative** = you (the customer) can update DNS records
- Also a ==domain registrar== (can register domain names)
- Can check ==health of resources==
- Only AWS service with ==100% availability SLA==
- Named after port ==53== (traditional DNS port)

## DNS Record Types

| Type | Maps To | Example |
|------|---------|--------|
| **A** | Hostname → ==IPv4== | example.com → 1.2.3.4 |
| **AAAA** | Hostname → ==IPv6== | example.com → 2001:db8::1 |
| **CNAME** | Hostname → ==Hostname== | app.example.com → lb.aws.com |
| **NS** | ==Name Servers== for hosted zone | Controls traffic routing |

> [!warning] CNAME Restriction
> You ==cannot== create a CNAME for the ==zone apex== (root domain). e.g., cannot CNAME `example.com`, but can CNAME `www.example.com`.

## Each Record Contains

| Field | Description |
|-------|-------------|
| **Domain/Subdomain** | e.g., example.com |
| **Record Type** | A, AAAA, CNAME, NS |
| **Value** | e.g., 12.34.56.78 |
| **Routing Policy** | How Route 53 responds to queries |
| **TTL** | Time to Live (cache duration) |

## Hosted Zones

| Type | Accessible From | Use Case | Example |
|------|----------------|----------|--------|
| **Public** | ==Internet== (anyone) | Public websites | example.com |
| **Private** | ==VPC only== | Internal services | app.company.internal |

```
Public Hosted Zone:              Private Hosted Zone:
┌─────────┐                       ┌─────────────────────┐
│ Browser │──▶ Route 53           │        VPC            │
└─────────┘    example.com       │  EC2 ─▶ Route 53       │
                │                  │       app.internal   │
                ▼                  │           │           │
           Public IP               │       Private IP     │
                                   └─────────────────────┘
```

> [!tip] Cost
> - Hosted zone: ==$0.50/month==
> - Domain registration: ==$12+/year==

## Questions & Answers

> [!question]- Q1: What does "authoritative" mean for Route 53?
> **Answer:**
> It means ==you (the customer) can update the DNS records==. You have full control over the DNS configuration.

> [!question]- Q2: Why is it called Route 53?
> **Answer:**
> Because ==53 is the traditional DNS port== used by DNS services.

> [!question]- Q3: What is Route 53's availability SLA?
> **Answer:**
> ==100%== — the only AWS service that provides this guarantee.

> [!question]- Q4: What are the 4 must-know DNS record types?
> **Answer:**
> - **A**: hostname → IPv4
> - **AAAA**: hostname → IPv6
> - **CNAME**: hostname → hostname
> - **NS**: name servers for the hosted zone

> [!question]- Q5: Can you create a CNAME for the root domain (zone apex)?
> **Answer:**
> ==No==. CNAME cannot be used for the zone apex (e.g., `example.com`). Only subdomains like `www.example.com`.

> [!question]- Q6: What is the difference between public and private hosted zones?
> **Answer:**
> - **Public**: Accessible from the ==internet== (anyone can query)
> - **Private**: Only accessible from within your ==VPC== (internal resources)

> [!question]- Q7: How much does a hosted zone cost?
> **Answer:**
> ==$0.50 per month== per hosted zone.

> [!question]- Q8: What is an NS record?
> **Answer:**
> ==Name Server record== — specifies the DNS servers that can respond to queries for your hosted zone. Controls how traffic is routed to a domain.

> [!question]- Q9: Is Route 53 only a DNS service?
> **Answer:**
> ==No==. Route 53 is also a ==domain registrar== (you can register domain names) and supports ==health checks== on resources.

> [!question]- Q10: What information does each DNS record contain?
> **Answer:**
> Domain/subdomain name, record type, value, routing policy, and TTL (time to live).