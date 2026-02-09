---
title: EC2 Instance Store
date: 2026-02-09
tags:
  - aws
  - ec2
  - instance-store
  - ephemeral
  - high-performance
  - saa-c03
---

# EC2 Instance Store

## What is EC2 Instance Store?

A ==hardware disk physically attached== to the server hosting your EC2 instance. Provides the highest possible I/O performance.

> [!danger] Ephemeral Storage
> Data is ==lost== when the instance is stopped, terminated, or the underlying hardware fails. Not for durable storage.

## EBS vs Instance Store

| Feature | EBS | Instance Store |
|---------|-----|----------------|
| **Connection** | Network | ==Physical hardware== |
| **Performance** | Up to 64K IOPS (io2) | Up to ==3.3 million IOPS== |
| **Persistence** | Survives stop/terminate | ==Lost on stop/terminate== |
| **Backup** | Snapshots | ==Your responsibility== |
| **Use case** | Databases, OS | Cache, buffer, temp data |

## Performance Example

```
Instance Store (i3 family):
├── Read IOPS:  up to 3,300,000
└── Write IOPS: up to 1,400,000

EBS gp2:
├── Max IOPS: 16,000
└── (200x less than instance store!)
```

## Use Cases

- ✅ Buffer / cache
- ✅ Scratch data / temporary content
- ✅ High-performance temporary storage
- ❌ Long-term storage (use EBS)
- ❌ Data that must survive instance stop

> [!warning] Exam Tip
> Anytime you see ==very high performance hardware-attached volume== for EC2, think ==EC2 Instance Store==.

## Questions & Answers

> [!question]- Q1: What is EC2 Instance Store?
> **Answer:**
> A ==physical hard drive attached to the server== hosting your EC2 instance. It provides the highest I/O performance available but is ephemeral — data is lost when the instance stops or terminates.

> [!question]- Q2: What happens to Instance Store data when you stop an EC2 instance?
> **Answer:**
> The data is ==completely lost==. Instance Store is ephemeral storage. If you need persistent data, use EBS volumes instead.

> [!question]- Q3: When should you use Instance Store over EBS?
> **Answer:**
> When you need ==extremely high I/O performance== for temporary data: buffers, caches, scratch data, or temporary content. Instance Store can deliver millions of IOPS vs EBS's thousands.

> [!question]- Q4: How much faster is Instance Store compared to EBS?
> **Answer:**
> Instance Store (i3 family) can reach ==3.3 million read IOPS==, while EBS gp2 maxes out at ==16,000 IOPS==. That's roughly 200x more performance.

> [!question]- Q5: Who is responsible for backing up Instance Store data?
> **Answer:**
> ==You are==. AWS does not back up Instance Store data. If you use it, you must implement your own backup and replication strategy.

> [!question]- Q6: What happens if the underlying server hardware fails?
> **Answer:**
> You ==lose all data== on the Instance Store because the physical disk is attached to that specific server. This is another reason it's called ephemeral storage.

> [!question]- Q7: Can all EC2 instance types use Instance Store?
> **Answer:**
> ==No==. Only specific instance types (like i3, d2, h1) come with Instance Store volumes. The availability depends on the instance family.

> [!question]- Q8: Can you detach an Instance Store volume and attach it to another instance?
> **Answer:**
> ==No==. Instance Store is ==physically attached== to the host server. It cannot be detached or moved. Only EBS volumes (network drives) can be detached and reattached.

> [!question]- Q9: Is Instance Store included in the EBS free tier?
> **Answer:**
> ==No==. Instance Store is not an EBS product. It's part of the EC2 instance itself. The cost is included in the EC2 instance pricing for instance types that support it.

> [!question]- Q10: What is the exam keyword for Instance Store?
> **Answer:**
> Look for: ==very high performance==, ==hardware-attached==, ==millions of IOPS==, ==ephemeral==, ==temporary storage==. Any of these should make you think EC2 Instance Store.
