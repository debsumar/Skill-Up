---
title: Security Groups
date: 2026-02-03
tags:
  - aws
  - ec2
  - security-groups
  - firewall
  - networking
  - saa-c03
---

# Security Groups

## What are Security Groups?

==Security Groups== are virtual firewalls that control inbound and outbound traffic for EC2 instances.

```
┌─────────────────────────────────────────────────────────┐
│                    SECURITY GROUP                        │
│  ┌─────────────────────────────────────────────────┐    │
│  │                                                 │    │
│  │              EC2 INSTANCE                       │    │
│  │                                                 │    │
│  └─────────────────────────────────────────────────┘    │
│                                                         │
│  INBOUND RULES              OUTBOUND RULES              │
│  ─────────────              ──────────────              │
│  What can come IN           What can go OUT             │
│  (Default: DENY ALL)        (Default: ALLOW ALL)        │
└─────────────────────────────────────────────────────────┘
```

## Key Characteristics

| Feature | Description |
|---------|-------------|
| **Allow Rules Only** | Can only specify ALLOW rules (no DENY) |
| **Stateful** | Return traffic automatically allowed |
| **Reference Options** | By IP address (CIDR) or other Security Groups |
| **Scope** | Locked to ==Region/VPC== combination |
| **Location** | Lives ==outside== EC2 (blocked traffic never reaches instance) |
| **Multiple Attachment** | One SG → Many instances, One instance → Many SGs |

## Inbound vs Outbound Rules

### Inbound Rules (Incoming Traffic)

Controls what traffic can ==enter== the instance.

```
Internet ────▶ Security Group ────▶ EC2 Instance
              (Inbound Rules)
              
Example:
├── Allow SSH (22) from 0.0.0.0/0
├── Allow HTTP (80) from 0.0.0.0/0
└── Allow HTTPS (443) from 0.0.0.0/0
```

**Default:** ==All inbound traffic BLOCKED==

### Outbound Rules (Outgoing Traffic)

Controls what traffic can ==leave== the instance.

```
EC2 Instance ────▶ Security Group ────▶ Internet
                  (Outbound Rules)
                  
Default:
└── Allow ALL traffic to 0.0.0.0/0
```

**Default:** ==All outbound traffic ALLOWED==

## Rule Components

| Component | Description | Example |
|-----------|-------------|---------|
| **Type** | Protocol preset | SSH, HTTP, HTTPS, Custom TCP |
| **Protocol** | TCP, UDP, ICMP | TCP |
| **Port Range** | Single or range | 22, 80, 443, 1024-65535 |
| **Source/Destination** | IP range or SG | 0.0.0.0/0, sg-12345, 10.0.0.0/8 |

### CIDR Notation

| CIDR | Meaning |
|------|---------|
| `0.0.0.0/0` | All IPv4 addresses (Anywhere) |
| `::/0` | All IPv6 addresses (Anywhere) |
| `192.168.1.0/24` | 192.168.1.0 - 192.168.1.255 |
| `10.0.0.5/32` | Single IP: 10.0.0.5 only |

## Classic Ports to Remember

> [!important] Exam Essential
> Memorize these ports for the SAA-C03 exam.

| Port | Protocol | Use |
|------|----------|-----|
| **22** | SSH | Secure Shell - Linux instance login |
| **21** | FTP | File Transfer Protocol |
| **22** | SFTP | Secure FTP (uses SSH) |
| **80** | HTTP | Unsecured web traffic |
| **443** | HTTPS | Secured web traffic (TLS/SSL) |
| **3389** | RDP | Remote Desktop - Windows instance login |

```
Linux Instance Access:  Port 22 (SSH)
Windows Instance Access: Port 3389 (RDP)
Web Server:             Port 80 (HTTP) / 443 (HTTPS)
```

## Security Group Referencing

> [!tip] Advanced Feature
> Instead of IP addresses, reference other Security Groups for dynamic access control.

```
┌──────────────────┐         ┌──────────────────┐
│  EC2 Instance A  │         │  EC2 Instance B  │
│     (SG-1)       │◀───────▶│     (SG-2)       │
└──────────────────┘         └──────────────────┘
         │
         │ SG-1 Inbound Rule:
         │ "Allow traffic from SG-2"
         │
         ▼
   No IP management needed!
   Any instance with SG-2 can connect.
```

### Use Case: Load Balancer to EC2

```
┌─────────────────┐      ┌─────────────────┐
│  Load Balancer  │─────▶│   EC2 Instance  │
│    (SG-LB)      │      │    (SG-App)     │
└─────────────────┘      └─────────────────┘

SG-App Inbound Rule:
├── Allow HTTP (80) from SG-LB
└── (Not from 0.0.0.0/0 - only from LB!)
```

## Troubleshooting

### Timeout vs Connection Refused

> [!danger] Critical Distinction
> This is a ==common exam question==.

| Symptom | Cause | Solution |
|---------|-------|----------|
| **Timeout** (hangs forever) | Security Group blocking | Check inbound rules |
| **Connection Refused** | Application issue | SG is fine, check app |

```
Timeout:
┌────────┐     ✗     ┌────────────────┐
│ Client │───────────│ Security Group │  Traffic blocked
└────────┘           └────────────────┘  before reaching EC2

Connection Refused:
┌────────┐     ✓     ┌────────────────┐     ✗     ┌─────────┐
│ Client │───────────│ Security Group │───────────│ EC2 App │
└────────┘           └────────────────┘           └─────────┘
                     Traffic passed               App not running
```

> [!warning] Remember
> ==Timeout = 100% Security Group issue==
> Check that the required port is open in inbound rules.

## Best Practices

1. **Separate SSH Security Group**
   - Create dedicated SG for SSH access
   - Easier to manage and audit

2. **Principle of Least Privilege**
   - Only open required ports
   - Restrict source IPs when possible

3. **Avoid 0.0.0.0/0 for SSH/RDP**
   - Use "My IP" option
   - Or specific IP ranges

4. **Use Security Group References**
   - For internal communication
   - Avoids IP management

## Hands-On: Verifying Security Groups

### Test HTTP Access

1. Remove HTTP (80) rule from Security Group
2. Try accessing `http://<PUBLIC_IP>`
3. Result: ==Timeout== (infinite loading)
4. Add HTTP (80) rule back
5. Refresh: Page loads successfully

### Security Group Settings

```
Inbound Rules:
├── SSH (22) ────────▶ Source: 0.0.0.0/0
└── HTTP (80) ───────▶ Source: 0.0.0.0/0

Outbound Rules:
└── All Traffic ─────▶ Destination: 0.0.0.0/0
```

## Questions & Answers

> [!question]- Q1: What does a timeout when connecting to EC2 indicate?
> **Answer:**
> A timeout ==always== indicates a Security Group issue. The traffic is being blocked by the firewall before reaching the instance. Check that the required port is open in the inbound rules.

> [!question]- Q2: Can Security Groups have DENY rules?
> **Answer:**
> ==No==. Security Groups can only have ALLOW rules. All traffic is denied by default, and you explicitly allow what you need. For DENY rules, use Network ACLs (NACLs).

> [!question]- Q3: What's the default behavior for inbound and outbound traffic?
> **Answer:**
> - **Inbound**: All traffic ==BLOCKED== by default
> - **Outbound**: All traffic ==ALLOWED== by default
> 
> You must explicitly add inbound rules to allow traffic.

> [!question]- Q4: Can one Security Group be used by multiple instances?
> **Answer:**
> ==Yes==. Security Groups have a many-to-many relationship:
> - One SG can be attached to multiple instances
> - One instance can have multiple SGs attached
> 
> Rules from all attached SGs are combined.

> [!question]- Q5: What ports are used for SSH and RDP?
> **Answer:**
> - **SSH (Linux)**: Port ==22==
> - **RDP (Windows)**: Port ==3389==
> - **HTTP**: Port ==80==
> - **HTTPS**: Port ==443==
