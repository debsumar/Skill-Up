---
title: EC2 Basics & Instance Launch
date: 2026-02-03
tags:
  - aws
  - ec2
  - compute
  - user-data
  - saa-c03
---

# EC2 Basics & Instance Launch

## What is Amazon EC2?

==EC2 (Elastic Compute Cloud)== is AWS's Infrastructure as a Service (IaaS) offering.

```
Amazon EC2 Capabilities
├── Rent Virtual Machines ──────────▶ EC2 Instances
├── Store Data on Virtual Drives ───▶ EBS (Elastic Block Store)
├── Distribute Load ────────────────▶ ELB (Elastic Load Balancer)
└── Scale Services ─────────────────▶ ASG (Auto Scaling Group)
```

> [!important] Fundamental AWS Service
> Knowing EC2 is ==fundamental to understanding how the cloud works==. It's the ability to rent compute on demand.

## EC2 Instance Configuration Options

| Component | Options | Notes |
|-----------|---------|-------|
| **Operating System** | Linux, Windows, Mac OS | Linux most popular |
| **CPU** | Number of vCPUs/cores | Compute power |
| **RAM** | Amount of memory | Random Access Memory |
| **Storage** | Network (EBS/EFS) or Hardware (Instance Store) | See storage section |
| **Network** | Speed, Public IP | Network card type |
| **Firewall** | Security Groups | Control traffic |
| **Bootstrap** | EC2 User Data | First-launch script |

## EC2 User Data (Bootstrap Script)

> [!tip] Key Concept
> EC2 User Data runs ==only once== at ==first instance launch== with ==root user== privileges (sudo).

### Purpose

Automate boot tasks:
- Install updates
- Install software
- Download files from internet
- Configure the instance

### Example: Web Server Setup

```bash
#!/bin/bash
# Update packages
yum update -y
# Install Apache web server
yum install -y httpd
# Start and enable httpd
systemctl start httpd
systemctl enable httpd
# Create web page
echo "Hello World from $(hostname -f)" > /var/www/html/index.html
```

> [!warning] Boot Time Impact
> More commands in User Data = longer boot time. Keep scripts efficient.

## EC2 Instance Types Overview

### Free Tier: t2.micro

| Spec | Value |
|------|-------|
| vCPU | 1 |
| Memory | 1 GB |
| Storage | EBS only |
| Network | Low to Moderate |
| Free Tier | 750 hours/month |

### Instance Type Examples

| Type | vCPU | Memory | Network | Use Case |
|------|------|--------|---------|----------|
| t2.micro | 1 | 1 GB | Low-Moderate | Free tier, testing |
| t2.xlarge | 4 | 16 GB | Moderate | Small apps |
| c5d.4xlarge | 16 | 32 GB | Up to 10 Gbps | Compute-intensive |
| r5.16xlarge | 64 | 512 GB | 20 Gbps | Memory-intensive |

## Launching Your First EC2 Instance

### Step 1: Name and Tags

```
Name: My First Instance
(Creates a "Name" tag automatically)
```

### Step 2: Choose AMI (Amazon Machine Image)

- **Quick Start**: Amazon Linux 2 (Free tier eligible)
- **Architecture**: 64-bit (x86)

> [!tip] Amazon Linux 2
> Recommended for learning - comes with AWS CLI pre-installed.

### Step 3: Instance Type

- Select **t2.micro** (Free tier eligible)
- Compare types: Click "Compare instance types" link

### Step 4: Key Pair (for SSH)

**Create new key pair:**
- Name: `EC2Tutorial`
- Type: RSA
- Format:
  - `.pem` → Mac, Linux, Windows 10+
  - `.ppk` → Windows 7/8 (PuTTY)

> [!warning] Download Immediately
> Key pair downloads ==only once==. Store securely - you cannot retrieve it later.

### Step 5: Network Settings

**Security Group (Firewall):**
```
launch-wizard-1 (auto-created)
├── Inbound Rules
│   ├── SSH (Port 22) ──────▶ From: 0.0.0.0/0 (Anywhere)
│   └── HTTP (Port 80) ─────▶ From: 0.0.0.0/0 (Anywhere)
└── Outbound Rules
    └── All Traffic ────────▶ To: 0.0.0.0/0 (Anywhere)
```

- ✅ Allow SSH traffic from Anywhere
- ✅ Allow HTTP traffic from the internet
- ❌ HTTPS (not needed for this demo)

### Step 6: Storage Configuration

```
Root Volume
├── Size: 8 GB
├── Type: gp2 (General Purpose SSD)
└── Delete on termination: Yes ✓
```

> [!important] Delete on Termination
> Default is ==Yes== - volume deleted when instance terminates. Change to No if you need persistent data.

### Step 7: Advanced Details - User Data

Paste the web server script in User Data field:

```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "Hello World from $(hostname -f)" > /var/www/html/index.html
```

### Step 8: Launch

- Review summary
- Click **Launch Instance**
- Instance state: Pending → Running (~10-15 seconds)

## Accessing Your Instance

### Web Server Access

```
http://<PUBLIC_IPv4_ADDRESS>
```

> [!danger] Use HTTP, Not HTTPS
> Use `http://` (port 80), NOT `https://` (port 443). HTTPS will timeout since we didn't configure it.

### Instance Details

| Property | Example | Notes |
|----------|---------|-------|
| Instance ID | i-0abc123def456 | Unique identifier |
| Public IPv4 | 54.123.45.67 | For external access |
| Private IPv4 | 172.31.33.135 | Internal AWS network |
| Instance State | Running | Current status |

## Instance Lifecycle

### States

```
┌─────────┐    Launch    ┌─────────┐
│ Stopped │─────────────▶│ Pending │
└─────────┘              └────┬────┘
     ▲                        │
     │ Stop                   ▼
     │                   ┌─────────┐
     └───────────────────│ Running │
                         └────┬────┘
                              │ Terminate
                              ▼
                         ┌────────────┐
                         │ Terminated │
                         └────────────┘
```

### Actions

| Action | Effect | Billing |
|--------|--------|---------|
| **Stop** | Shutdown, keep EBS | No compute charge |
| **Start** | Boot from EBS | Resume billing |
| **Terminate** | Delete instance + EBS (if delete on term) | Stop billing |

## IP Address Behavior

> [!warning] Public IP Changes on Stop/Start
> When you stop and start an instance, the ==Public IPv4 may change==.
> The ==Private IPv4 stays the same==.

```
Before Stop:
├── Public IP: 54.123.45.67
└── Private IP: 172.31.33.135

After Start:
├── Public IP: 3.250.12.89  ← CHANGED!
└── Private IP: 172.31.33.135  ← Same
```

## Questions & Answers

> [!question]- Q1: When does EC2 User Data script run?
> **Answer:**
> EC2 User Data runs ==only once== at the ==first launch== of the instance. It never runs again, even after stop/start. It runs with ==root user== privileges.

> [!question]- Q2: What happens to the Public IP when you stop and start an instance?
> **Answer:**
> The Public IPv4 address ==may change== when you stop and start an instance. The Private IPv4 address ==stays the same==. Use Elastic IP if you need a static public IP.

> [!question]- Q3: What's the difference between Stop and Terminate?
> **Answer:**
> - **Stop**: Instance shuts down, EBS volume preserved, no compute charges, can restart later
> - **Terminate**: Instance deleted, EBS deleted (if delete on termination = yes), cannot recover

> [!question]- Q4: Why is my web server showing timeout instead of the page?
> **Answer:**
> Check:
> 1. Using `http://` not `https://`
> 2. Security group has port 80 (HTTP) open
> 3. Wait 2-5 minutes for User Data script to complete
> 4. Instance is in "Running" state
