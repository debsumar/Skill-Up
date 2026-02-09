---
title: SSH Access Methods
date: 2026-02-03
tags:
  - aws
  - ec2
  - ssh
  - putty
  - instance-connect
  - saa-c03
---

# SSH Access Methods

## Overview

SSH (Secure Shell) allows you to ==control a remote machine using the command line==.

```
┌──────────────┐                    ┌──────────────┐
│  Your        │      SSH (22)      │  EC2         │
│  Computer    │───────────────────▶│  Instance    │
│              │    Public IP       │  (Linux)     │
└──────────────┘                    └──────────────┘
```

## Prerequisites

1. **Security Group**: Port 22 (SSH) open
2. **Key Pair**: `.pem` or `.ppk` file
3. **Public IP**: Instance must have public IP
4. **Instance State**: Running

## SSH Methods by Operating System

| Your OS | Method | Tool |
|---------|--------|------|
| Mac | SSH command | Terminal |
| Linux | SSH command | Terminal |
| Windows 10+ | SSH command | PowerShell/CMD |
| Windows 7/8 | PuTTY | PuTTY application |
| Any | EC2 Instance Connect | Web browser |

> [!tip] Recommendation
> If one method works, you're good! EC2 Instance Connect is easiest for beginners.

## Method 1: SSH on Mac/Linux

### Step 1: Prepare Key File

```bash
# Navigate to key location
cd ~/Downloads  # or wherever your key is

# Verify key exists
ls -la EC2Tutorial.pem

# Fix permissions (required!)
chmod 0400 EC2Tutorial.pem
```

> [!warning] Permission Error
> If you skip `chmod 0400`, you'll get "unprotected private key file" error.

### Step 2: Connect

```bash
ssh -i EC2Tutorial.pem ec2-user@<PUBLIC_IP>
```

**Breakdown:**
- `ssh` - SSH command
- `-i EC2Tutorial.pem` - Identity file (private key)
- `ec2-user` - Default username for Amazon Linux 2
- `@<PUBLIC_IP>` - Instance public IP address

### Step 3: First Connection

```
The authenticity of host '54.123.45.67' can't be established.
Are you sure you want to continue connecting (yes/no)? yes
```

Type `yes` to trust the host.

### Step 4: Verify Connection

```bash
# You should see:
[ec2-user@ip-172-31-33-135 ~]$

# Test commands
whoami          # Returns: ec2-user
ping google.com # Test internet connectivity
```

### Step 5: Exit

```bash
exit
# or press Ctrl+D
```

## Method 2: SSH on Windows 10+

### Using PowerShell

```powershell
# Open PowerShell
# Navigate to key location
cd C:\Users\YourName\Desktop

# Verify SSH is available
ssh

# Connect
ssh -i EC2Tutorial.pem ec2-user@<PUBLIC_IP>
```

### Fixing Permission Issues (Windows)

If you get permission errors:

1. Right-click `.pem` file → **Properties**
2. **Security** tab → **Advanced**
3. **Disable inheritance** → Remove all inherited permissions
4. **Add** → Select your username
5. Grant **Full control**
6. **Owner**: Change to your username

```
File Properties → Security → Advanced
├── Owner: YourUsername
├── Disable inheritance
└── Permission entries:
    └── YourUsername: Full control
```

## Method 3: PuTTY (Windows 7/8)

### Step 1: Install PuTTY

Download from: https://www.putty.org/

### Step 2: Convert Key (PuTTYgen)

If you have `.pem` file:

1. Open **PuTTYgen**
2. Click **Load** → Select `.pem` file (show all files)
3. Click **Save private key**
4. Save as `EC2Tutorial.ppk`

### Step 3: Configure PuTTY

1. Open **PuTTY**
2. **Session**:
   - Host Name: `ec2-user@<PUBLIC_IP>`
   - Port: 22
   - Connection type: SSH
3. **Connection → SSH → Auth → Credentials**:
   - Private key file: Browse to `.ppk` file
4. **Session**:
   - Saved Sessions: `EC2 Instance`
   - Click **Save**
5. Click **Open**

### Step 4: Accept Host Key

Click **Accept** when prompted about host key.

## Method 4: EC2 Instance Connect (Browser)

> [!tip] Easiest Method
> No key management, works from any OS, browser-based.

### Steps

1. Go to **EC2 Console → Instances**
2. Select your instance
3. Click **Connect**
4. Choose **EC2 Instance Connect** tab
5. Username: `ec2-user` (auto-filled)
6. Click **Connect**

```
EC2 Console
└── Instances
    └── Select Instance
        └── Connect
            └── EC2 Instance Connect
                └── Connect (opens browser terminal)
```

### Requirements

- Instance must have **public IP**
- Security Group must allow **SSH (22)** from AWS IP ranges
- Works with **Amazon Linux 2** (and Amazon Linux 2023)

> [!warning] Still Needs Port 22
> EC2 Instance Connect uses SSH behind the scenes. Port 22 must be open in Security Group.

### Troubleshooting Instance Connect

If connection fails:

1. Check Security Group has SSH (22) open
2. Try adding both IPv4 and IPv6 rules:
   - SSH from `0.0.0.0/0` (IPv4)
   - SSH from `::/0` (IPv6)

## Default Usernames by AMI

| AMI | Default Username |
|-----|------------------|
| Amazon Linux 2 | `ec2-user` |
| Amazon Linux 2023 | `ec2-user` |
| Ubuntu | `ubuntu` |
| CentOS | `centos` |
| Debian | `admin` |
| RHEL | `ec2-user` |
| SUSE | `ec2-user` |

## Common SSH Commands

```bash
# Check who you are
whoami

# Check hostname
hostname

# Test internet
ping google.com

# View system info
uname -a

# Check disk space
df -h

# Exit session
exit
```

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| Timeout | Security Group | Open port 22 |
| Permission denied | Wrong key | Use correct .pem file |
| Unprotected key file | Permissions too open | `chmod 0400 key.pem` |
| Connection refused | SSH not running | Check instance state |
| Host key verification failed | IP changed | Remove old entry from known_hosts |

## Questions & Answers

> [!question]- Q1: What's the default username for Amazon Linux 2?
> **Answer:**
> `ec2-user` is the default username for Amazon Linux 2 and Amazon Linux 2023. Other AMIs have different defaults (ubuntu for Ubuntu, centos for CentOS, etc.).

> [!question]- Q2: Why do I need to run chmod 0400 on the key file?
> **Answer:**
> SSH requires private keys to have restricted permissions. `chmod 0400` makes the file readable only by the owner. Without this, SSH refuses to use the key for security reasons.

> [!question]- Q3: What's the difference between .pem and .ppk files?
> **Answer:**
> - `.pem` - OpenSSH format, used by Mac/Linux/Windows 10+ SSH command
> - `.ppk` - PuTTY Private Key format, used by PuTTY on Windows
> 
> Use PuTTYgen to convert between formats.

> [!question]- Q4: Does EC2 Instance Connect require a key pair?
> **Answer:**
> ==No==. EC2 Instance Connect uploads a temporary SSH key automatically. You don't need to manage keys. However, port 22 must still be open in the Security Group.

> [!question]- Q5: Why does my Public IP change after stop/start?
> **Answer:**
> AWS may assign a different Public IP when you start an instance. Update your SSH command with the new IP. The Private IP stays the same. Use Elastic IP for a static public IP.
