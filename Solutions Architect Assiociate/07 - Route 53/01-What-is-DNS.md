---
title: "What is DNS?"
date: 2026-02-10
tags:
  - aws
  - dns
  - networking
  - route53
  - saa-c03
---

# What is DNS?

## Overview

Before diving into Amazon Route 53, we need to understand ==DNS (Domain Name System)== — the backbone of the internet. DNS is something you've been using behind the scenes every single day, but you may not know exactly how it works.

DNS translates ==human-friendly hostnames== (like `www.google.com`) into ==machine-readable IP addresses== (like `172.217.18.36`). Without DNS, you'd have to memorize IP addresses for every website you visit. DNS is what makes the internet usable for humans.

> [!tip] Why This Matters for the Exam
> Understanding DNS fundamentals is ==essential== for Route 53 questions. The exam tests your knowledge of record types, TTL behavior, and how DNS resolution works. This lecture provides the foundation for everything that follows.

## DNS Hierarchy

DNS uses a ==hierarchical naming structure==. Every domain name is built from right to left, from the most general to the most specific:

```
                    ┌─────────────────────────────────────────┐
                    │         DNS Hierarchy                    │
                    │                                         │
                    │              . (Root)                   │
                    │               │                         │
                    │         ┌─────┴─────┐                  │
                    │        .com        .org                 │
                    │         │                               │
                    │    example.com                          │
                    │         │                               │
                    │   www.example.com                       │
                    │         │                               │
                    │   api.www.example.com                   │
                    └─────────────────────────────────────────┘
```

## DNS Terminology

Understanding these terms is critical for Route 53:

| Term | Definition | Example |
|------|-----------|---------|
| **Domain Registrar** | Where you ==register/buy== domain names | Amazon Route 53, GoDaddy |
| **DNS Records** | Entries that map hostnames to IPs/other hostnames | A, AAAA, CNAME, NS |
| **Zone File** | Contains all DNS records for a domain | The file that maps hostnames to addresses |
| **Name Server** | Server that ==resolves DNS queries== | Route 53's name servers |
| **Top Level Domain (TLD)** | The last part of a domain | `.com`, `.org`, `.gov`, `.us`, `.in` |
| **Second Level Domain (SLD)** | The domain name itself | `amazon.com`, `google.com` |
| **FQDN** | ==Fully Qualified Domain Name== — the complete address | `api.www.example.com.` |
| **Subdomain** | A domain under the SLD | `www.example.com`, `api.example.com` |

### Anatomy of a URL

```
http://api.www.example.com.
│       │   │      │     │ │
│       │   │      │     │ └── Root (the final dot, usually hidden)
│       │   │      │     └──── TLD (Top Level Domain)
│       │   │      └────────── SLD (Second Level Domain)
│       │   └───────────────── Subdomain
│       └───────────────────── FQDN (Fully Qualified Domain Name)
└───────────────────────────── Protocol
```

> [!note] The Hidden Root Dot
> Every domain name technically ends with a dot (`.`) representing the root of the DNS hierarchy. When you type `www.google.com` in your browser, it's actually `www.google.com.` — the browser just hides the trailing dot. This root is managed by ==ICANN== (Internet Corporation for Assigned Names and Numbers).

## How DNS Resolution Works

When you type `example.com` in your browser, here's what happens behind the scenes — a ==recursive DNS resolution== process:

```
┌──────────┐                                                              ┌──────────┐
│  Your    │                                                              │   Web    │
│  Browser │                                                              │  Server  │
│          │                                                              │ 9.10.11.12│
└────┬─────┘                                                              └──────────┘
     │                                                                         ▲
     │ 1. "What is example.com?"                                               │
     ▼                                                                         │
┌──────────────┐                                                               │
│  Local DNS   │  (Assigned by your ISP or company)                           │
│  Server      │                                                               │
│  (Resolver)  │                                                               │
└──┬───┬───┬───┘                                                               │
   │   │   │                                                                   │
   │   │   │  2. "Do you know example.com?"                                   │
   │   │   ▼                                                                   │
   │   │  ┌──────────────┐                                                    │
   │   │  │  Root DNS    │  Managed by ICANN                                  │
   │   │  │  Server      │  "I don't know, but .com is at 1.2.3.4"           │
   │   │  └──────────────┘                                                    │
   │   │                                                                       │
   │   │  3. "Do you know example.com?"                                       │
   │   ▼                                                                       │
   │  ┌──────────────┐                                                        │
   │  │  TLD DNS     │  Managed by IANA                                       │
   │  │  Server      │  "I know .com! example.com NS is at 5.6.7.8"          │
   │  │  (.com)      │                                                        │
   │  └──────────────┘                                                        │
   │                                                                           │
   │  4. "Do you know example.com?"                                           │
   ▼                                                                           │
  ┌──────────────┐                                                            │
  │  SLD DNS     │  Managed by YOUR domain registrar (e.g., Route 53)        │
  │  Server      │  "Yes! example.com A record = 9.10.11.12"                 │
  │ (example.com)│                                                            │
  └──────────────┘                                                            │
                                                                               │
   5. Local DNS caches the answer (for TTL duration)                          │
   6. Returns IP 9.10.11.12 to your browser                                  │
   7. Browser connects to 9.10.11.12 ─────────────────────────────────────────┘
```

### Step-by-Step Breakdown

1. Your browser asks the ==Local DNS Server== (assigned by your ISP or company): "What is `example.com`?"
2. The Local DNS Server doesn't know, so it asks the ==Root DNS Server== (managed by ICANN). The Root says: "I don't know `example.com`, but I know `.com` — its name server is at `1.2.3.4`."
3. The Local DNS Server asks the ==TLD DNS Server== (`.com`, managed by IANA): "What about `example.com`?" The TLD says: "I know about `example.com` — its name server is at `5.6.7.8`."
4. The Local DNS Server asks the ==SLD DNS Server== (managed by your domain registrar, e.g., Route 53): "What is `example.com`?" The SLD says: "Yes! `example.com` is an A record pointing to `9.10.11.12`."
5. The Local DNS Server ==caches the answer== for the duration of the TTL (Time To Live).
6. The Local DNS Server returns the IP `9.10.11.12` to your browser.
7. Your browser connects directly to `9.10.11.12` and loads the website.

> [!important] Key Insight
> DNS does ==not route traffic==. It only ==resolves names to addresses==. The actual HTTP traffic goes directly from your browser to the web server — it never passes through the DNS servers. This is a critical distinction that the exam tests.

## Caching and TTL

After the Local DNS Server resolves a query, it ==caches the result== for the duration of the TTL. This means:
- Subsequent requests for the same domain are answered ==instantly== from cache
- No additional DNS queries are needed until the TTL expires
- This reduces load on DNS servers and speeds up browsing

We'll explore TTL in detail in [[03-Records-and-TTL|Records and TTL]].

## Why This Matters for Route 53

Route 53 acts as the ==SLD DNS Server== in the diagram above. When you register a domain with Route 53 (or point your domain's NS records to Route 53), it becomes the ==authoritative DNS== for your domain. This means:
- Route 53 is the ==source of truth== for your DNS records
- When someone queries your domain, Route 53 provides the answer
- You have ==full control== over what answers Route 53 gives

This is the foundation for everything we'll learn in this section — from simple A records to complex routing policies.

## Questions & Answers

> [!question]- Q1: What is DNS and what does it do?
> **Answer:**
> DNS (Domain Name System) translates ==human-friendly hostnames== (like `www.google.com`) into ==IP addresses== (like `172.217.18.36`) that computers use to communicate. It's the backbone of the internet — without DNS, you'd need to memorize IP addresses for every website.

> [!question]- Q2: What is the difference between a TLD and an SLD?
> **Answer:**
> - ==TLD (Top Level Domain)==: The last part of a domain name — `.com`, `.org`, `.gov`, `.us`
> - ==SLD (Second Level Domain)==: The domain name itself — `amazon.com`, `google.com`
> The TLD is managed by IANA, while the SLD is managed by your ==domain registrar== (like Route 53 or GoDaddy).

> [!question]- Q3: What is an FQDN?
> **Answer:**
> FQDN stands for ==Fully Qualified Domain Name==. It's the complete domain name that specifies the exact location in the DNS hierarchy. For example, `api.www.example.com.` is an FQDN. It includes the subdomain, SLD, TLD, and the root dot (usually hidden by browsers).

> [!question]- Q4: What is a domain registrar?
> **Answer:**
> A domain registrar is a service where you ==register (buy) domain names==. Examples include Amazon Route 53, GoDaddy, and Google Domains. The registrar manages the registration and ensures your domain is unique. Note: a domain registrar is ==different from a DNS service==, though most registrars provide both.

> [!question]- Q5: How does recursive DNS resolution work?
> **Answer:**
> When you query a domain, the Local DNS Server recursively asks:
> 1. ==Root DNS Server== → "I know `.com` is at this NS"
> 2. ==TLD DNS Server (.com)== → "I know `example.com` is at this NS"
> 3. ==SLD DNS Server (example.com)== → "Here's the A record: `9.10.11.12`"
> The Local DNS Server caches the result for the TTL duration and returns it to your browser.

> [!question]- Q6: What is the role of the Local DNS Server?
> **Answer:**
> The Local DNS Server (also called a ==resolver==) is assigned by your ISP or company. It's the first server your browser contacts for DNS resolution. It performs the recursive lookup by querying Root → TLD → SLD servers, then ==caches the result== so future queries for the same domain are answered instantly.

> [!question]- Q7: What are NS records?
> **Answer:**
> NS (Name Server) records specify which DNS servers are ==authoritative== for a domain. They tell the DNS system "to find records for this domain, ask these name servers." When you register a domain with Route 53, it creates NS records pointing to Route 53's name servers.

> [!question]- Q8: Does DNS route traffic?
> **Answer:**
> ==No.== DNS only resolves names to addresses. The actual HTTP/HTTPS traffic goes ==directly from your browser to the web server== — it never passes through DNS servers. DNS just tells your browser which IP address to connect to. This is a critical distinction tested on the exam.

> [!question]- Q9: What is the root of the DNS hierarchy?
> **Answer:**
> The root is represented by a ==dot (`.`)== at the end of every domain name (usually hidden by browsers). It's managed by ==ICANN==. The root DNS servers know the addresses of all TLD servers (`.com`, `.org`, `.gov`, etc.) and are the starting point for every DNS resolution.

> [!question]- Q10: What organizations manage the different levels of DNS?
> **Answer:**
> - ==Root DNS==: Managed by ==ICANN== (Internet Corporation for Assigned Names and Numbers)
> - ==TLD DNS== (`.com`, `.org`): Managed by ==IANA== (Internet Assigned Numbers Authority)
> - ==SLD DNS== (`example.com`): Managed by your ==domain registrar== (Route 53, GoDaddy, etc.)
> Each level delegates to the next, creating the hierarchical resolution chain.
