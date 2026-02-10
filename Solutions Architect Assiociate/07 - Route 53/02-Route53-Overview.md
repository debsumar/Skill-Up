---
title: "Amazon Route 53 - Overview"
date: 2026-02-10
tags:
  - aws
  - route53
  - dns
  - hosted-zones
  - saa-c03
---

# Amazon Route 53 - Overview

## Overview

Amazon Route 53 is a ==highly available, scalable, and fully managed authoritative DNS== service. The name "Route 53" comes from the fact that ==DNS uses port 53== — that's the traditional port for DNS traffic.

What does "authoritative" mean? It means ==you (the customer) have full control== over the DNS records. You can update, create, and delete records as you see fit. Route 53 is the ==source of truth== for your domain's DNS.

Route 53 is also a ==domain registrar== — you can buy domain names directly through it (like `example.com`). And it's the ==only AWS service that provides a 100% availability SLA==.

```
┌──────────┐   DNS Query:          ┌──────────────┐   A Record:        ┌──────────────┐
│          │   "example.com?"      │              │   54.22.33.44      │              │
│  Client  │──────────────────────▶│   Route 53   │──────────────────▶│  EC2 Instance│
│          │◀──────────────────────│   (DNS)      │                    │  54.22.33.44 │
│          │   "54.22.33.44"       │              │                    │              │
└──────────┘                       └──────────────┘                    └──────────────┘
```

## DNS Records in Route 53

Each DNS record in Route 53 contains:

| Field | Description | Example |
|-------|-------------|---------|
| **Domain/Subdomain** | The hostname being resolved | `example.com`, `api.example.com` |
| **Record Type** | The type of DNS record | A, AAAA, CNAME, NS |
| **Value** | The answer to the DNS query | `12.34.56.78` |
| **Routing Policy** | How Route 53 responds to queries | Simple, Weighted, Latency, etc. |
| **TTL** | Time To Live — how long the result is cached | 300 seconds |

### Record Types You Must Know

| Type | Maps To | Example |
|------|---------|---------|
| ==A== | Hostname → ==IPv4== address | `example.com` → `1.2.3.4` |
| ==AAAA== | Hostname → ==IPv6== address | `example.com` → `2001:db8::1` |
| ==CNAME== | Hostname → ==another hostname== | `app.example.com` → `lb.amazonaws.com` |
| ==NS== | ==Name servers== for the hosted zone | Controls which DNS servers answer queries |

> [!warning] CNAME Restriction
> You ==cannot create a CNAME== for the top node of a DNS namespace (the ==Zone Apex==). For example, you cannot create a CNAME for `example.com` — only for subdomains like `www.example.com`. This is a standard DNS restriction, not specific to AWS. To point the zone apex to another hostname, use an ==Alias record== (covered in [[04-CNAME-vs-Alias|CNAME vs Alias]]).

## Hosted Zones

A ==hosted zone== is a container for DNS records. It defines how traffic is routed for a domain and its subdomains. There are two types:

### Public Hosted Zone

```
┌──────────────────────────────────────────────────────────────┐
│                    PUBLIC HOSTED ZONE                         │
│                    mypublicdomain.com                         │
│                                                               │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │  Records:                                               │ │
│  │  app1.mypublicdomain.com  →  A  →  54.22.33.44        │ │
│  │  api.mypublicdomain.com   →  A  →  54.22.33.55        │ │
│  │  www.mypublicdomain.com   →  CNAME → app1.myp...      │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                               │
│  Accessible from: ==The entire internet==                    │
└──────────────────────────────────────────────────────────────┘
         ▲
         │  DNS Query
┌────────┴────────┐
│  Public Client  │  (anyone on the internet)
│  (web browser)  │
└─────────────────┘
```

- Answers queries from ==public clients== (anyone on the internet)
- Used for public-facing websites and APIs
- Created when you register a domain or manually create one

### Private Hosted Zone

```
┌──────────────────────────────────────────────────────────────┐
│                    PRIVATE HOSTED ZONE                        │
│                    company.internal                           │
│                                                               │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │  Records:                                               │ │
│  │  webapp.company.internal   →  A  →  10.0.0.5           │ │
│  │  api.company.internal      →  A  →  10.0.0.10          │ │
│  │  database.company.internal →  A  →  10.0.0.20          │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                               │
│  Accessible from: ==Only within your VPC(s)==                │
└──────────────────────────────────────────────────────────────┘
         ▲
         │  DNS Query (private)
┌────────┴────────┐
│  EC2 Instance   │  (inside your VPC)
│  10.0.0.5       │
└─────────────────┘
```

- Answers queries ==only from within your VPC(s)==
- Used for internal service discovery (microservices, databases, etc.)
- Domain names like `company.internal`, `myapp.private`
- Not accessible from the public internet

> [!tip] Private Hosted Zones in Practice
> If you've worked at a company with internal URLs like `jira.company.internal` or `wiki.corp.net` that only work on the corporate network — those are backed by private DNS records. On AWS, private hosted zones serve the same purpose within your VPC.

### Hosted Zone Costs

| Item | Cost |
|------|------|
| Hosted zone | ==$0.50/month== per hosted zone |
| DNS queries | $0.40 per million queries (standard) |
| Domain registration | ==$12+/year== depending on TLD |

> [!note] Not Free Tier
> Route 53 is ==not part of the AWS Free Tier==. You'll pay $0.50/month for each hosted zone and $12+/year for domain registration. If you're following along with the course, be aware of these costs.

## Registering a Domain (Hands-On)

To get started with Route 53, you need to register a domain:

1. Go to ==Route 53 Console== → ==Register Domains==
2. Search for an available domain name
3. Choose duration (1 year minimum) and auto-renew preference
4. Enter contact information (enable ==privacy protection== to hide your personal info from WHOIS)
5. Review and submit — costs ==$12-13/year== for `.com` domains
6. Wait for registration (can take minutes to hours)

After registration, Route 53 automatically creates:
- A ==public hosted zone== for your domain
- ==NS records== pointing to Route 53's name servers
- An ==SOA record== (Start of Authority)

### Creating Your First Record

Once the hosted zone exists:

1. Go to ==Hosted Zones== → click your domain
2. Click ==Create Record==
3. Enter:
   - Record name: `test` (creates `test.yourdomain.com`)
   - Record type: ==A==
   - Value: an IP address (e.g., `11.22.33.44`)
   - TTL: 300 seconds
   - Routing policy: Simple
4. Click ==Create Record==

### Verifying with CLI

You can verify your record using command-line tools:

```bash
# Using nslookup (Windows/Mac/Linux)
nslookup test.yourdomain.com

# Using dig (Mac/Linux — more detailed output)
dig test.yourdomain.com

# Install dig on Amazon Linux if needed
sudo yum install -y bind-utils
```

The `dig` command shows the ==answer section== with the record type, TTL countdown, and IP value — very useful for debugging DNS issues.

## EC2 Setup for Route 53 Testing

For the hands-on exercises in this section, the course sets up:

- ==Three EC2 instances== in different regions (e.g., Frankfurt, N. Virginia, Singapore)
- Each running a simple web server that displays "Hello World from [AZ]"
- ==One ALB== in one region (e.g., Frankfurt) pointing to the local EC2 instance
- User Data script that shows the instance's AZ in the response

```
┌──────────────────────────────────────────────────────────────┐
│                    Test Infrastructure                        │
│                                                               │
│  Region: eu-central-1 (Frankfurt)                            │
│  ┌──────────┐  ┌──────────┐                                 │
│  │ EC2      │  │   ALB    │                                  │
│  │ Instance │◀─│          │                                  │
│  └──────────┘  └──────────┘                                  │
│                                                               │
│  Region: us-east-1 (N. Virginia)                             │
│  ┌──────────┐                                                │
│  │ EC2      │                                                │
│  │ Instance │                                                │
│  └──────────┘                                                │
│                                                               │
│  Region: ap-southeast-1 (Singapore)                          │
│  ┌──────────┐                                                │
│  │ EC2      │                                                │
│  │ Instance │                                                │
│  └──────────┘                                                │
└──────────────────────────────────────────────────────────────┘
```

This setup allows testing different routing policies — latency-based routing will direct to the closest region, weighted routing will distribute traffic by percentage, etc.

> [!warning] Cost Reminder
> These EC2 instances and the ALB ==cost money==. Remember to terminate all instances and delete the ALB when done with the section. The cleanup lecture at the end covers this.

## Questions & Answers

> [!question]- Q1: What is Amazon Route 53?
> **Answer:**
> Route 53 is a ==highly available, scalable, fully managed, and authoritative DNS== service from AWS. "Authoritative" means you have full control over DNS records. It's also a ==domain registrar== (you can buy domains). The name comes from ==DNS port 53==. It's the only AWS service with a ==100% availability SLA==.

> [!question]- Q2: What are the four DNS record types you must know?
> **Answer:**
> - ==A==: Maps hostname to IPv4 address (`example.com` → `1.2.3.4`)
> - ==AAAA==: Maps hostname to IPv6 address
> - ==CNAME==: Maps hostname to another hostname (cannot be used at zone apex)
> - ==NS==: Name Server records — specify which DNS servers are authoritative for the zone

> [!question]- Q3: What is a hosted zone?
> **Answer:**
> A hosted zone is a ==container for DNS records== that defines how traffic is routed for a domain and its subdomains. There are two types:
> - ==Public hosted zone==: Answers queries from the public internet
> - ==Private hosted zone==: Answers queries only from within your VPC(s)
> Each hosted zone costs ==$0.50/month==.

> [!question]- Q4: What is the difference between a public and private hosted zone?
> **Answer:**
> - ==Public==: Accessible from the entire internet. Used for public websites (`www.mysite.com`).
> - ==Private==: Accessible ==only from within your VPC(s)==. Used for internal service discovery (`api.company.internal`, `db.corp.net`).
> Both work the same way — they contain DNS records that map hostnames to addresses. The difference is ==who can query them==.

> [!question]- Q5: Why can't you create a CNAME for the zone apex?
> **Answer:**
> This is a ==standard DNS restriction== (RFC), not specific to AWS. The zone apex (e.g., `example.com` without any subdomain prefix) must have an A or AAAA record — it cannot be a CNAME. AWS solves this with ==Alias records==, which can point the zone apex to AWS resources like ELBs, CloudFront, etc.

> [!question]- Q6: How much does Route 53 cost?
> **Answer:**
> - ==Hosted zone==: $0.50/month per zone
> - ==DNS queries==: $0.40 per million queries (standard), $0.60 for latency-based
> - ==Domain registration==: $12+/year depending on TLD
> - ==Health checks==: $0.50/month per health check (basic)
> Route 53 is ==not part of the Free Tier==.

> [!question]- Q7: What records are automatically created when you register a domain?
> **Answer:**
> When you register a domain with Route 53, it automatically creates:
> - A ==public hosted zone== for your domain
> - ==NS records== pointing to four Route 53 name servers (these tell the internet where to find your DNS records)
> - An ==SOA record== (Start of Authority) with metadata about the zone

> [!question]- Q8: What is the difference between a domain registrar and a DNS service?
> **Answer:**
> - ==Domain registrar==: Where you buy/register domain names (pay annual fees)
> - ==DNS service==: Where you manage DNS records (hosted zones, A records, etc.)
> Most registrars provide both, but they're ==separate functions==. You can buy a domain on GoDaddy and use Route 53 as your DNS service (covered in [[08-Third-Party-Domains|Third-Party Domains]]).

> [!question]- Q9: How do you verify a DNS record is working?
> **Answer:**
> Use command-line tools:
> - `nslookup test.yourdomain.com` — shows the resolved IP address
> - `dig test.yourdomain.com` — shows detailed info including ==record type, TTL, and value==
> The `dig` command is preferred because it shows the TTL countdown, which is useful for understanding caching behavior.

> [!question]- Q10: Why is Route 53 the only AWS service with 100% SLA?
> **Answer:**
> DNS is ==critical infrastructure== — if DNS goes down, nothing works. Users can't resolve any hostnames, so all websites and APIs become unreachable. AWS guarantees ==100% availability== for Route 53 because DNS failure would cascade to every other service. This makes Route 53 one of the most reliable services in all of AWS.
