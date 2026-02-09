---
title: 3rd Party Domains & Route 53
date: 2026-02-10
tags:
  - aws
  - route53
  - domain-registrar
  - dns
  - saa-c03
---

# 3rd Party Domains & Route 53

## Domain Registrar ≠ DNS Service

| Concept | Description |
|---------|-------------|
| **Domain Registrar** | Where you ==buy/register== domain names (Route 53, GoDaddy, Google Domains) |
| **DNS Service** | Where you ==manage DNS records== (Route 53 hosted zones) |

> [!important] Key Distinction
> Every domain registrar usually comes with a DNS service, but you ==don't have to use it==. You can buy a domain on GoDaddy and manage DNS on Route 53.

## Using Route 53 with a 3rd Party Domain

```
Step 1: Buy domain on GoDaddy
         ↓
Step 2: Create Public Hosted Zone in Route 53
         ↓
Step 3: Get NS (Name Server) records from Route 53
         ↓
Step 4: Update NS records on GoDaddy → point to Route 53 NS
         ↓
Step 5: Manage all DNS records in Route 53
```

| Step | Action |
|------|--------|
| 1 | Register domain on 3rd party (e.g., GoDaddy) |
| 2 | Create ==public hosted zone== in Route 53 |
| 3 | Copy the ==4 NS records== from Route 53 hosted zone |
| 4 | Update ==custom name servers== on 3rd party to Route 53 NS |
| 5 | All DNS queries now resolved by ==Route 53== |

## Questions & Answers

> [!question]- Q1: Can you use Route 53 as DNS for a domain bought on GoDaddy?
> **Answer:**
> ==Yes==. Create a public hosted zone in Route 53, then update the NS records on GoDaddy to point to Route 53's name servers.

> [!question]- Q2: Is a domain registrar the same as a DNS service?
> **Answer:**
> ==No==. A domain registrar is where you buy domains. A DNS service manages DNS records. They're often bundled but can be used separately.

> [!question]- Q3: What records do you update on the 3rd party registrar?
> **Answer:**
> The ==NS (Name Server) records==. You set them to the 4 name servers provided by your Route 53 hosted zone.

> [!question]- Q4: Do you need to create a hosted zone in Route 53 for a 3rd party domain?
> **Answer:**
> ==Yes==. You must create a ==public hosted zone== in Route 53 to manage the DNS records.

> [!question]- Q5: Can you use a 3rd party DNS with a domain registered on Route 53?
> **Answer:**
> ==Yes==. The registrar and DNS service are independent. You can mix and match.

> [!question]- Q6: How many name servers does Route 53 provide per hosted zone?
> **Answer:**
> ==4 name servers== per hosted zone.

> [!question]- Q7: What is the cost of using Route 53 as DNS for a 3rd party domain?
> **Answer:**
> ==$0.50/month== for the hosted zone, plus per-query charges. The domain registration cost is paid to the 3rd party registrar.

> [!question]- Q8: What happens after you update NS records on the 3rd party?
> **Answer:**
> All DNS queries for your domain are now resolved by ==Route 53==. You manage all records (A, CNAME, Alias, etc.) in the Route 53 hosted zone.

> [!question]- Q9: Can you use Route 53 Alias records with a 3rd party domain?
> **Answer:**
> ==Yes==. Once Route 53 is your DNS service (via NS records), you can use all Route 53 features including Alias records.

> [!question]- Q10: What are examples of domain registrars?
> **Answer:**
> Amazon Route 53, GoDaddy, Google Domains, Namecheap, and many others. Each charges annual fees for domain registration.
