---
title: "Third-Party Domains & Route 53"
date: 2026-02-10
tags:
  - aws
  - route53
  - dns
  - domain-registrar
  - saa-c03
---

# Third-Party Domains & Route 53

## Overview

A common misconception is that you ==must== buy your domain from Route 53 to use Route 53 as your DNS service. That's not true. You can buy a domain from ==any registrar== (GoDaddy, Google Domains, Namecheap, etc.) and still use Route 53 to manage your DNS records.

The key insight: a ==domain registrar== and a ==DNS service== are ==two different things==, even though most registrars bundle both together.

## Domain Registrar vs DNS Service

```
┌──────────────────────────────────────────────────────────────┐
│           DOMAIN REGISTRAR vs DNS SERVICE                     │
│                                                               │
│  DOMAIN REGISTRAR                DNS SERVICE                 │
│  ┌──────────────────┐           ┌──────────────────┐        │
│  │ Where you BUY    │           │ Where you MANAGE │        │
│  │ domain names     │           │ DNS records      │        │
│  │                  │           │                  │        │
│  │ - Pay annual fee │           │ - A, AAAA, CNAME │        │
│  │ - Own the domain │           │ - Routing policies│        │
│  │ - WHOIS info     │           │ - Health checks  │        │
│  │                  │           │                  │        │
│  │ Examples:        │           │ Examples:        │        │
│  │ - Route 53       │           │ - Route 53       │        │
│  │ - GoDaddy        │           │ - Cloudflare DNS │        │
│  │ - Google Domains │           │ - GoDaddy DNS    │        │
│  │ - Namecheap      │           │                  │        │
│  └──────────────────┘           └──────────────────┘        │
│                                                               │
│  These are SEPARATE functions!                               │
│  You can mix and match.                                      │
└──────────────────────────────────────────────────────────────┘
```

| Function | What It Does | Who Provides It |
|----------|-------------|-----------------|
| **Domain Registrar** | ==Registers/buys== domain names, manages ownership | Route 53, GoDaddy, Google Domains, etc. |
| **DNS Service** | ==Manages DNS records== (A, CNAME, routing policies, etc.) | Route 53, Cloudflare, GoDaddy DNS, etc. |

> [!important] Key Exam Point
> You can buy a domain on GoDaddy and use ==Route 53 as your DNS service==. You can also buy a domain on Route 53 and use a ==third-party DNS service==. The registrar and DNS service are ==independent==.

## How to Use Route 53 with a Third-Party Domain

If you bought your domain from GoDaddy (or any other registrar) and want to use Route 53 for DNS:

```
┌──────────────────────────────────────────────────────────────┐
│         USING ROUTE 53 WITH A THIRD-PARTY DOMAIN             │
│                                                               │
│  Step 1: Create a Public Hosted Zone in Route 53             │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │  Route 53 → Hosted Zones → Create Hosted Zone          │ │
│  │  Domain: example.com                                    │ │
│  │  Type: Public                                           │ │
│  │                                                         │ │
│  │  Route 53 assigns 4 Name Servers:                      │ │
│  │  - ns-123.awsdns-45.com                                │ │
│  │  - ns-678.awsdns-90.net                                │ │
│  │  - ns-111.awsdns-22.org                                │ │
│  │  - ns-444.awsdns-55.co.uk                              │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                               │
│  Step 2: Update NS Records on GoDaddy                        │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │  GoDaddy → Domain Settings → Name Servers → Custom     │ │
│  │                                                         │ │
│  │  Replace GoDaddy's name servers with Route 53's:       │ │
│  │  NS1: ns-123.awsdns-45.com                             │ │
│  │  NS2: ns-678.awsdns-90.net                             │ │
│  │  NS3: ns-111.awsdns-22.org                             │ │
│  │  NS4: ns-444.awsdns-55.co.uk                           │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                               │
│  Step 3: Manage all DNS records in Route 53                  │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │  Now Route 53 is the authoritative DNS for example.com  │ │
│  │  Create A, CNAME, Alias records as needed              │ │
│  │  Use routing policies, health checks, etc.             │ │
│  └─────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────┘
```

### Step-by-Step

1. ==Create a Public Hosted Zone== in Route 53 for your domain (e.g., `example.com`)
2. Route 53 assigns ==four name servers== (NS records) to the hosted zone
3. Go to your third-party registrar (GoDaddy, etc.)
4. Find the ==Name Servers== or ==Custom DNS== settings
5. ==Replace the registrar's default name servers== with the four Route 53 name servers
6. Save — DNS propagation may take up to 48 hours (usually much faster)
7. Now all DNS queries for your domain are answered by ==Route 53==

> [!tip] Why Use Route 53 with a Third-Party Domain?
> Route 53 offers features that many registrar DNS services don't:
> - ==Alias records== (free queries, zone apex support)
> - ==Advanced routing policies== (Weighted, Latency, Geolocation, etc.)
> - ==Health checks== with automated failover
> - ==100% availability SLA==
> - Integration with other AWS services

## Section Cleanup

When you're done with the Route 53 section, clean up to avoid ongoing costs:

| Resource | Action | Cost if Left Running |
|----------|--------|---------------------|
| ==EC2 instances== (3 regions) | Terminate all | ~$8.50/month each |
| ==ALB== | Delete load balancer + target group | ~$16/month |
| ==Hosted zone== | Delete if not needed (delete records first) | ==$0.50/month== |
| ==Domain name== | Cannot delete — auto-renews unless disabled | ==$12+/year== |
| ==Health checks== | Delete all | $0.50/month each |

> [!warning] Domain Names Cannot Be Refunded
> Once you register a domain, you ==cannot get a refund==. The domain will auto-renew unless you disable auto-renewal. The hosted zone costs $0.50/month — you can delete it to stop this charge, but the domain registration fee is non-refundable.

## Questions & Answers

> [!question]- Q1: Can you use Route 53 as DNS without buying the domain from Route 53?
> **Answer:**
> ==Yes.== You can buy a domain from any registrar (GoDaddy, Google Domains, etc.) and use Route 53 as your DNS service. Create a public hosted zone in Route 53, then update the ==NS records on your registrar== to point to Route 53's name servers.

> [!question]- Q2: What is the difference between a domain registrar and a DNS service?
> **Answer:**
> - ==Domain registrar==: Where you buy/register domain names (annual fee, ownership management)
> - ==DNS service==: Where you manage DNS records (A, CNAME, routing policies, health checks)
> They're ==separate functions==. Most registrars provide both, but you can mix and match.

> [!question]- Q3: How do you point a GoDaddy domain to Route 53?
> **Answer:**
> 1. Create a ==public hosted zone== in Route 53 for your domain
> 2. Copy the ==four NS (Name Server) records== from the hosted zone
> 3. Go to GoDaddy → Domain Settings → ==Custom Name Servers==
> 4. Replace GoDaddy's name servers with Route 53's four name servers
> 5. Save — Route 53 is now the authoritative DNS for your domain

> [!question]- Q4: What are NS records and why do they matter?
> **Answer:**
> NS (Name Server) records tell the DNS system ==which servers are authoritative== for a domain. When you change NS records on your registrar to point to Route 53, you're telling the internet: "For DNS queries about this domain, ask Route 53." This is how you delegate DNS authority from one provider to another.

> [!question]- Q5: How long does it take for NS changes to propagate?
> **Answer:**
> NS record changes can take ==up to 48 hours== to propagate globally, though it's usually much faster (minutes to a few hours). During propagation, some users may still be directed to the old DNS servers. This is because NS records have their own TTL at the TLD level.

> [!question]- Q6: Why would you use Route 53 instead of your registrar's DNS?
> **Answer:**
> Route 53 offers features most registrar DNS services lack:
> - ==Alias records== (free queries, zone apex support)
> - ==7+ routing policies== (Weighted, Latency, Geolocation, etc.)
> - ==Health checks== with automated DNS failover
> - ==100% availability SLA==
> - Native integration with AWS services (ELB, CloudFront, S3, etc.)

> [!question]- Q7: What costs should you be aware of after the Route 53 section?
> **Answer:**
> - ==EC2 instances==: ~$8.50/month each (terminate in all 3 regions)
> - ==ALB==: ~$16/month (delete load balancer and target group)
> - ==Hosted zone==: $0.50/month (delete if not needed)
> - ==Domain name==: $12+/year (non-refundable, disable auto-renew if not needed)
> - ==Health checks==: $0.50/month each (delete all)

> [!question]- Q8: Can you delete a hosted zone without deleting the domain?
> **Answer:**
> ==Yes.== The hosted zone and domain registration are separate. You can delete the hosted zone (stops the $0.50/month charge) while keeping the domain registered. However, without a hosted zone, DNS queries for your domain won't be answered by Route 53. You can recreate the hosted zone later.

> [!question]- Q9: What happens if you don't create a Default geolocation record?
> **Answer:**
> Users whose location doesn't match any geolocation rule will receive ==no DNS answer== (NXDOMAIN). This means the website won't load for those users. Always create a ==Default record== as a catch-all to ensure all users get a response.

> [!question]- Q10: Exam scenario — company bought domain on GoDaddy but wants to use Route 53 features. What should they do?
> **Answer:**
> 1. Create a ==public hosted zone== in Route 53 for the domain
> 2. Note the ==four NS records== assigned by Route 53
> 3. Go to GoDaddy and update the ==custom name servers== to Route 53's NS records
> 4. Create DNS records in Route 53 (A, Alias, routing policies, health checks, etc.)
> The domain stays registered with GoDaddy, but ==Route 53 handles all DNS==.
