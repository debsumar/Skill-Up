---
title: Private vs Public vs Elastic IP
date: 2026-02-03
tags:
  - aws
  - ec2
  - networking
  - ip-addresses
  - elastic-ip
  - saa-c03
---

# Private vs Public vs Elastic IP

## IPv4 vs IPv6

| Feature | IPv4 | IPv6 |
|---------|------|------|
| Format | `192.168.1.1` (4 numbers, dots) | `2001:0db8:85a3::8a2e:0370:7334` |
| Addresses | ~3.7 billion | Virtually unlimited |
| Common Use | Most common online | IoT, future-proofing |
| AWS Support | ✅ Full | ✅ Full |

> [!note] Course Focus
> This course focuses on ==IPv4== as it's still the most common format used online.

## Public vs Private IP

```
┌─────────────────────────────────────────────────────────────┐
│                        INTERNET                              │
│                     (Public Network)                         │
└──────────────────────────┬──────────────────────────────────┘
                           │
              Public IP: 54.123.45.67
                           │
                    ┌──────┴──────┐
                    │   Internet  │
                    │   Gateway   │
                    └──────┬──────┘
                           │
┌──────────────────────────┴──────────────────────────────────┐
│                   PRIVATE NETWORK (VPC)                      │
│                                                              │
│   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐     │
│   │ EC2 Instance│    │ EC2 Instance│    │ EC2 Instance│     │
│   │ 10.0.1.10   │    │ 10.0.1.11   │    │ 10.0.1.12   │     │
│   └─────────────┘    └─────────────┘    └─────────────┘     │
│                                                              │
│              Private IPs: 10.0.0.0/16 range                  │
└──────────────────────────────────────────────────────────────┘
```

### Public IP Characteristics

| Feature | Description |
|---------|-------------|
| Uniqueness | ==Globally unique== across entire internet |
| Accessibility | Machine accessible from anywhere on internet |
| Geolocation | Can be looked up to find approximate location |
| Assignment | Assigned by AWS, changes on stop/start |

### Private IP Characteristics

| Feature | Description |
|---------|-------------|
| Uniqueness | Unique only ==within private network== |
| Accessibility | Only accessible within the VPC |
| Reuse | Same private IP can exist in different networks |
| Persistence | ==Does not change== on stop/start |

### Private IP Ranges (RFC 1918)

```
10.0.0.0    - 10.255.255.255   (10.0.0.0/8)     - Large networks
172.16.0.0  - 172.31.255.255   (172.16.0.0/12)  - Medium networks
192.168.0.0 - 192.168.255.255  (192.168.0.0/16) - Small networks
```

## EC2 Instance IP Behavior

### Default Configuration

Every EC2 instance gets:
- **Private IP**: For internal AWS network communication
- **Public IP**: For WWW access (if enabled)

### SSH Access

```
From Your Computer:
├── Can use Public IP ✓ (via Internet)
├── Cannot use Private IP ✗ (not on same network)
└── Exception: VPN connection to VPC
```

### Stop/Start Behavior

> [!warning] Public IP Changes!
> When you ==stop and start== an instance, the Public IP ==may change==.
> The Private IP ==stays the same==.

```
Before Stop:
├── Public IP:  54.123.45.67
└── Private IP: 10.0.1.10

After Start:
├── Public IP:  3.250.12.89   ← CHANGED!
└── Private IP: 10.0.1.10     ← Same
```

## Elastic IP

> [!tip] Static Public IP
> An Elastic IP is a ==static public IPv4== address that you own until you release it.

### Characteristics

| Feature | Details |
|---------|---------|
| Ownership | You own it until you delete/release it |
| Attachment | Can attach to ==one instance at a time== |
| Portability | Can move between instances for failover |
| Limit | ==5 per AWS account== (can request increase) |
| Cost | ==$0.005/hour== (~$3.60/month) whether used or not |

### Use Case: Failover

```
Normal Operation:
┌─────────────┐
│ Elastic IP  │──────▶ EC2 Instance A (Primary)
│ 54.1.2.3    │
└─────────────┘

After Failover:
┌─────────────┐
│ Elastic IP  │──────▶ EC2 Instance B (Backup)
│ 54.1.2.3    │        (Same IP, different instance)
└─────────────┘
```

### Elastic IP Workflow

```
1. Allocate Elastic IP
   EC2 Console → Elastic IPs → Allocate Elastic IP address

2. Associate with Instance
   Actions → Associate Elastic IP address → Select Instance

3. Verify
   Instance now shows Elastic IP as Public IPv4

4. Stop/Start Test
   Public IP remains the same! ✓

5. Cleanup (Important!)
   Disassociate → Release Elastic IP address
```

## Best Practices

> [!danger] Avoid Elastic IPs When Possible
> Elastic IPs are often considered ==poor architectural decisions==.

### Better Alternatives

| Instead of Elastic IP | Use |
|----------------------|-----|
| Static IP for web server | DNS name (Route 53) |
| Direct instance access | Load Balancer |
| Failover | Auto Scaling Group |

### Why Avoid Elastic IPs?

1. **Limited**: Only 5 per account
2. **Cost**: Charged even when not attached
3. **Not scalable**: One IP = one instance
4. **Better patterns exist**: DNS, Load Balancers

## Pricing (Since Feb 2024)

> [!warning] IPv4 Charges
> All Public IPv4 addresses now cost ==$0.005/hour== (~$3.60/month)

| Scenario | Cost |
|----------|------|
| EC2 with Public IP (running) | $0.005/hour |
| Elastic IP (attached, running) | $0.005/hour |
| Elastic IP (not attached) | $0.005/hour |
| EC2 Free Tier | 750 hours/month free (first 12 months) |

## Questions & Answers

> [!question]- Q1: What happens to the Public IP when you stop and start an EC2 instance?
> **Answer:**
> The Public IP ==may change== when you stop and start an instance. The Private IP ==stays the same==. To keep a static public IP, use an Elastic IP.

> [!question]- Q2: Can you SSH into an EC2 instance using its Private IP from your home computer?
> **Answer:**
> ==No==, unless you have a VPN connection to the VPC. Private IPs are only accessible within the private network. You must use the Public IP to SSH from the internet.

> [!question]- Q3: What is an Elastic IP and when would you use it?
> **Answer:**
> An Elastic IP is a ==static public IPv4== that you own. Use cases:
> - Need a fixed IP that doesn't change on stop/start
> - Quick failover by moving IP between instances
> 
> However, better alternatives exist (DNS, Load Balancers).

> [!question]- Q4: How many Elastic IPs can you have per AWS account?
> **Answer:**
> ==5 Elastic IPs== per region by default. You can request an increase from AWS, but it's rare to need more. Consider using DNS or Load Balancers instead.

> [!question]- Q5: Are Elastic IPs free if attached to a running instance?
> **Answer:**
> ==No==. Since February 2024, all Public IPv4 addresses (including Elastic IPs) cost ==$0.005/hour== whether attached or not. Always release unused Elastic IPs.

> [!question]- Q6: What is the difference between IPv4 and IPv6 in AWS?
> **Answer:**
> ==IPv4== uses 4 numbers separated by dots (e.g., `192.168.1.1`) with ~3.7 billion addresses. ==IPv6== uses a longer hex format and is virtually unlimited. AWS supports both, but IPv4 is still the most common. IPv6 is mainly used for IoT.

> [!question]- Q7: How do private machines connect to the internet?
> **Answer:**
> Machines on a private network connect to the internet through a ==NAT device== and an ==Internet Gateway== that acts as a proxy. The Internet Gateway provides a public-facing IP for outbound traffic.

> [!question]- Q8: Can two different companies have the same private IP addresses?
> **Answer:**
> ==Yes==. Private IPs only need to be unique within their own private network. Two separate VPCs or companies can use identical private IP ranges (e.g., both using `10.0.1.10`) without conflict.

> [!question]- Q9: What are the recommended alternatives to Elastic IPs?
> **Answer:**
> - Use a ==random public IP with a DNS name== (Route 53) — more scalable
> - Use a ==Load Balancer== and avoid public IPs on instances entirely — best pattern for AWS
> 
> Both approaches are more scalable and cost-effective than Elastic IPs.

> [!question]- Q10: What happens to an Elastic IP when you stop an instance it's attached to?
> **Answer:**
> The Elastic IP ==remains attached== to the stopped instance. The public IPv4 does not change. When you start the instance again, the same Elastic IP is still associated. You must ==disassociate and release== the Elastic IP manually to stop being charged.
