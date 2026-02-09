---
title: EC2 Hibernate
date: 2026-02-03
tags:
  - aws
  - ec2
  - hibernate
  - performance
  - saa-c03
---

# EC2 Hibernate

## Instance States Review

| Action | RAM | EBS | Boot Time |
|--------|-----|-----|-----------|
| **Stop** | Lost | Preserved | Full boot (OS + apps) |
| **Terminate** | Lost | Deleted (if configured) | N/A |
| **Hibernate** | ==Preserved== | Preserved | ==Fast== (RAM restored) |

## What is EC2 Hibernate?

> [!tip] Key Concept
> Hibernate ==preserves the in-memory (RAM) state== by saving it to the root EBS volume. When started, the RAM is restored, making boot ==much faster==.

```
┌─────────────────────────────────────────────────────────────────┐
│                     HIBERNATE PROCESS                            │
│                                                                 │
│  RUNNING                 STOPPING                 STOPPED       │
│  ┌─────────┐            ┌─────────┐             ┌─────────┐    │
│  │   EC2   │            │   EC2   │             │   EC2   │    │
│  │         │  Hibernate │         │   Complete  │         │    │
│  │  ┌───┐  │ ─────────▶ │  ┌───┐  │ ─────────▶ │         │    │
│  │  │RAM│  │            │  │RAM│──┼──┐         │         │    │
│  │  └───┘  │            │  └───┘  │  │         │         │    │
│  │         │            │         │  │         │         │    │
│  │  ┌───┐  │            │  ┌───┐  │  │         │  ┌───┐  │    │
│  │  │EBS│  │            │  │EBS│◀─┼──┘ Dump    │  │EBS│  │    │
│  │  └───┘  │            │  └───┘  │    RAM     │  │RAM│  │    │
│  └─────────┘            └─────────┘             │  │dump│ │    │
│                                                 │  └───┘  │    │
│                                                 └─────────┘    │
│                                                                 │
│  STARTING                                       RUNNING         │
│  ┌─────────┐                                   ┌─────────┐     │
│  │   EC2   │            Load RAM               │   EC2   │     │
│  │         │ ◀──────────────────────────────── │         │     │
│  │  ┌───┐  │            from EBS               │  ┌───┐  │     │
│  │  │RAM│◀─┼────────────────────────────────── │  │RAM│  │     │
│  │  └───┘  │                                   │  └───┘  │     │
│  │         │                                   │         │     │
│  │  ┌───┐  │                                   │  ┌───┐  │     │
│  │  │EBS│  │                                   │  │EBS│  │     │
│  │  │RAM│  │                                   │  └───┘  │     │
│  │  │dump│ │                                   │         │     │
│  │  └───┘  │                                   └─────────┘     │
│  └─────────┘                                                   │
│                                                                 │
│  OS never stopped/restarted - just "frozen" and "unfrozen"     │
└─────────────────────────────────────────────────────────────────┘
```

## Why Use Hibernate?

| Benefit | Description |
|---------|-------------|
| **Fast Boot** | OS not restarted, just resumed |
| **Preserve State** | RAM contents maintained |
| **Long-running Processes** | Don't lose progress |
| **Service Initialization** | Skip slow startup |

### Use Cases

- ✅ Long-running processes you don't want to restart
- ✅ Services with slow initialization
- ✅ Save RAM state between sessions
- ✅ Quick resume for development environments

## Requirements

> [!warning] Must Meet All Requirements

| Requirement | Details |
|-------------|---------|
| **RAM Size** | Must be ==< 150 GB== |
| **Root Volume** | Must be ==EBS== (not instance store) |
| **Encryption** | Root EBS must be ==encrypted== |
| **Volume Size** | Must have space for RAM dump |
| **Instance Types** | Most types supported (not bare metal) |
| **OS** | Amazon Linux 2, Ubuntu, Windows |
| **Duration** | Max ==60 days== hibernation |

## Supported Instance Types

- General Purpose: M3, M4, M5, M6i, T2, T3
- Compute Optimized: C3, C4, C5, C6i
- Memory Optimized: R3, R4, R5, R6i
- Storage Optimized: I3

> [!note] Not Supported
> Bare metal instances cannot hibernate.

## Hands-On: Enable Hibernate

### Step 1: Launch Instance with Hibernate

```
Launch Instance
├── AMI: Amazon Linux 2
├── Instance Type: t2.micro
├── Storage: Configure
│   └── Advanced
│       └── Encrypted: Yes (required!)
│       └── KMS Key: aws/ebs (default)
└── Advanced Details
    └── Stop - Hibernate behavior: Enable ✓
```

### Step 2: Verify Hibernate Works

```bash
# SSH into instance
ssh -i key.pem ec2-user@<IP>

# Check uptime
uptime
# Output: up 1 minute

# Wait a minute, check again
uptime
# Output: up 2 minutes
```

### Step 3: Hibernate Instance

```
EC2 Console → Select Instance → Instance State → Hibernate Instance
```

### Step 4: Start and Verify

```bash
# After starting, SSH in again
ssh -i key.pem ec2-user@<IP>

# Check uptime - should NOT be 0!
uptime
# Output: up 3 minutes (continues from before!)
```

> [!tip] Proof of Hibernate
> If `uptime` shows continuous time (not reset to 0), hibernate worked! The OS was never actually stopped.

## Stop vs Hibernate Comparison

| Aspect | Stop | Hibernate |
|--------|------|-----------|
| RAM | Lost | ==Preserved== |
| Boot Time | Full OS boot | ==Fast resume== |
| User Data Script | Runs again | ==Does not run== |
| Processes | Terminated | ==Preserved== |
| uptime command | Resets to 0 | ==Continues== |
| EBS Root | Preserved | Preserved + RAM dump |

## Billing

| State | Compute Charge | Storage Charge |
|-------|----------------|----------------|
| Running | Yes | Yes |
| Stopped | No | Yes (EBS) |
| Hibernated | No | Yes (EBS + RAM dump) |

> [!note] Storage Cost
> Hibernated instances use more EBS storage (RAM dump), so storage costs may be slightly higher.

## Limitations

1. **60-day limit**: Cannot hibernate for more than 60 days
2. **RAM size**: Must be less than 150 GB
3. **Encrypted root**: Root volume must be encrypted
4. **No bare metal**: Not supported on bare metal instances
5. **Instance store**: Not supported with instance store root

## Questions & Answers

> [!question]- Q1: What is the main benefit of EC2 Hibernate?
> **Answer:**
> ==Fast boot time==. The RAM state is preserved by dumping it to the encrypted EBS root volume. When started, the RAM is restored, so the OS doesn't need to boot from scratch - it just resumes where it left off.

> [!question]- Q2: What are the requirements for EC2 Hibernate?
> **Answer:**
> - RAM must be ==< 150 GB==
> - Root volume must be ==EBS== (not instance store)
> - Root EBS must be ==encrypted==
> - EBS must have enough space for RAM dump
> - Cannot hibernate for more than ==60 days==

> [!question]- Q3: How can you prove that hibernate worked?
> **Answer:**
> Use the `uptime` command. After a normal stop/start, uptime resets to 0. After hibernate/start, uptime ==continues from where it was== because the OS was never actually stopped - just frozen and resumed.

> [!question]- Q4: Does the EC2 User Data script run after resuming from hibernate?
> **Answer:**
> ==No==. User Data only runs on first boot. Since hibernate preserves the OS state (it's not a fresh boot), User Data does not run when resuming from hibernation.

> [!question]- Q5: Can you hibernate an instance with instance store root volume?
> **Answer:**
> ==No==. Hibernate requires an ==EBS root volume== because the RAM contents must be written to persistent storage. Instance store is ephemeral and would lose the RAM dump when stopped.

> [!question]- Q6: What happens to the RAM data during the hibernate process?
> **Answer:**
> When you hibernate, the instance enters a ==stopping state== and the RAM contents are ==dumped to the root EBS volume==. The instance then stops and RAM is cleared. On start, the RAM dump is ==loaded back from EBS into memory==, restoring the exact previous state.

> [!question]- Q7: What is the maximum duration an EC2 instance can remain hibernated?
> **Answer:**
> ==60 days==. After 60 days, the instance should be started or terminated. This limit may change over time, but it's the current restriction.

> [!question]- Q8: Which instance purchase types support hibernate?
> **Answer:**
> Hibernate is available for:
> - ==On-Demand== instances
> - ==Reserved== instances
> - ==Spot== instances
> 
> All three purchase types support hibernation.

> [!question]- Q9: Why must the root EBS volume be encrypted for hibernate?
> **Answer:**
> Because the ==RAM contents are written to disk==, and RAM may contain sensitive data (passwords, keys, application state). ==Encryption ensures this data is protected at rest== on the EBS volume.

> [!question]- Q10: What is the difference between stopping and hibernating an EC2 instance?
> **Answer:**
> 
> | Aspect | Stop | Hibernate |
> |--------|------|-----------|
> | RAM | ==Lost== | ==Preserved== (dumped to EBS) |
> | Boot | Full OS boot | ==Fast resume== |
> | Processes | Terminated | ==Preserved== |
> | `uptime` | Resets to 0 | ==Continues== |
> | User Data | Runs again | Does not run |
