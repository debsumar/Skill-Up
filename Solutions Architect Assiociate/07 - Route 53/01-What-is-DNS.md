---
title: What is DNS?
date: 2026-02-10
tags:
  - aws
  - dns
  - networking
  - saa-c03
---

# What is DNS?

## Overview

==Domain Name System== — translates human-friendly hostnames into IP addresses.

Example: `www.google.com` → `172.217.18.36`

## DNS Terminology

| Term | Definition |
|------|------------|
| **Domain Registrar** | Where you register domain names (Route 53, GoDaddy) |
| **DNS Records** | A, AAAA, CNAME, NS, etc. |
| **Zone File** | Contains all DNS records (hostname → IP mappings) |
| **Name Server** | Servers that resolve DNS queries |
| **TLD** | Top Level Domain (.com, .org, .gov) |
| **SLD** | Second Level Domain (amazon.com, google.com) |
| **FQDN** | Fully Qualified Domain Name |

## Anatomy of a URL

```
http://api.www.example.com.
│       │    │      │     │
│       │    │      │     └── Root (invisible dot)
│       │    │      └── TLD (.com)
│       │    └── SLD (example.com)
│       └── Subdomain (www.example.com)
│
└── Protocol (http)

FQDN = api.www.example.com
```

## How DNS Works

```
┌──────────┐  1. example.com?   ┌───────────────┐
│   Web    │───────────────────▶│  Local DNS    │
│ Browser  │                    │  Server       │
└──────────┘                    └───┬───┬───┬───┘
     ▲                              │   │   │
     │ 5. IP: 9.10.11.12            │   │   │
     │                              │   │   │
     └──────────────────────────────┘   │   │
                                        │   │
         2. .com NS?  ┌─────────────┐   │   │
         ◀────────────│  Root DNS   │◀──┘   │
         1.2.3.4      │  (ICANN)    │       │
                      └─────────────┘       │
         3. example.com NS?  ┌──────────┐   │
         ◀───────────────────│  TLD DNS │◀──┘
         5.6.7.8             │  (IANA)  │
                             └──────────┘
         4. example.com A?   ┌──────────────┐
         ◀───────────────────│  SLD DNS     │
         9.10.11.12          │  (Route 53)  │
                             └──────────────┘
```

1. Browser asks ==Local DNS Server== for example.com
2. Local DNS asks ==Root DNS== → gets .com NS at 1.2.3.4
3. Local DNS asks ==TLD DNS== (.com) → gets example.com NS at 5.6.7.8
4. Local DNS asks ==SLD DNS== (Route 53) → gets A record 9.10.11.12
5. Local DNS ==caches== the result and returns IP to browser

## Questions & Answers

> [!question]- Q1: What does DNS stand for?
> **Answer:**
> ==Domain Name System==. It translates human-friendly hostnames (like google.com) into IP addresses that computers use.

> [!question]- Q2: What is an FQDN?
> **Answer:**
> ==Fully Qualified Domain Name==. Example: `api.www.example.com` — the complete domain name including all subdomains.

> [!question]- Q3: What is the difference between TLD and SLD?
> **Answer:**
> - **TLD**: Top Level Domain — `.com`, `.org`, `.gov`
> - **SLD**: Second Level Domain — `amazon.com`, `google.com`

> [!question]- Q4: What is a Name Server?
> **Answer:**
> A server that ==resolves DNS queries==. It looks up the DNS records and returns the corresponding IP address.

> [!question]- Q5: What role does the Local DNS Server play?
> **Answer:**
> It's the first server your browser contacts. It ==recursively queries== Root → TLD → SLD DNS servers, then ==caches== the result for future queries.

> [!question]- Q6: Who manages the Root DNS servers?
> **Answer:**
> ==ICANN== (Internet Corporation for Assigned Names and Numbers).

> [!question]- Q7: What is a Zone File?
> **Answer:**
> A file that contains ==all DNS records== for a domain — the mappings between hostnames and IP addresses.

> [!question]- Q8: What is a Domain Registrar?
> **Answer:**
> A service where you ==register domain names==. Examples: Amazon Route 53, GoDaddy, Google Domains.

> [!question]- Q9: Why does the Local DNS cache results?
> **Answer:**
> To avoid querying the DNS hierarchy repeatedly. If someone asks for the same domain again, the cached result is returned ==immediately==.

> [!question]- Q10: What are the steps in DNS resolution order?
> **Answer:**
> Browser → ==Local DNS== → ==Root DNS== → ==TLD DNS== → ==SLD DNS== (authoritative) → IP returned and cached.