# 🖥️ Amazon EC2 Fundamentals

> **AWS Certified Study Notes** | Last Updated: January 2026

---

## 📋 Table of Contents

- [What is EC2?](#what-is-ec2)
- [EC2 Instance Types](#ec2-instance-types)
- [EC2 Pricing Models](#ec2-pricing-models)
- [Security Groups](#security-groups)
- [EC2 Storage Options](#ec2-storage-options)
- [AMI - Amazon Machine Images](#ami---amazon-machine-images)
- [EC2 Placement Groups](#ec2-placement-groups)
- [EC2 Hibernate](#ec2-hibernate)
- [Exam Tips](#exam-tips)

---

## 🎯 What is EC2?

**EC2 = Elastic Compute Cloud = Infrastructure as a Service (IaaS)**

| Feature | Description |
|---------|-------------|
| 🔄 **Elasticity** | Scale up/down based on demand |
| 💻 **Virtual Machines** | Rent virtual servers in AWS |
| 📦 **Storage** | EBS volumes for data storage |
| ⚖️ **Load Balancing** | Distribute traffic across instances |
| 📈 **Auto Scaling** | Automatic capacity management |

### EC2 Configuration Options

```
┌─────────────────────────────────────────────────────────────┐
│                    EC2 INSTANCE SETUP                       │
├─────────────────────────────────────────────────────────────┤
│  🖥️  Operating System    → Linux, Windows, Mac OS           │
│  ⚡  Compute Power       → CPU cores (vCPUs)                │
│  🧠  Memory              → RAM (GiB)                        │
│  💾  Storage             → EBS, Instance Store, EFS         │
│  🌐  Network             → Speed, Public IP                 │
│  🔒  Security Group      → Firewall rules                   │
│  📝  User Data           → Bootstrap script                 │
└─────────────────────────────────────────────────────────────┘
```

---

## 🏷️ EC2 Instance Types

> 💡 **Memory Trick**: `GFIRST` → General, Full memory, IO optimized, Real-time, Storage, Turbo compute

### Instance Type Naming Convention

```
    m   5   .   2xlarge
    │   │       │
    │   │       └── Size (nano, micro, small, medium, large, xlarge...)
    │   └────────── Generation (higher = newer)
    └────────────── Instance Class
```

### 📊 Instance Families

| Type | Use Case | Example | 🎯 Exam Focus |
|------|----------|---------|---------------|
| **General Purpose** | Web servers, code repos | `t2`, `t3`, `m5`, `m6i` | ✅ Balanced workloads |
| **Compute Optimized** | Batch processing, ML, gaming | `c5`, `c6i`, `c7g` | ✅ High CPU performance |
| **Memory Optimized** | In-memory databases, caching | `r5`, `r6i`, `x1`, `z1d` | ✅ Large datasets in RAM |
| **Storage Optimized** | Data warehousing, OLTP | `i3`, `d2`, `h1` | ✅ High sequential R/W |
| **Accelerated Computing** | GPU, ML inference | `p4`, `g4`, `inf1` | ✅ Hardware accelerators |

### 🔥 Common Instance Types to Remember

```
┌──────────────────────────────────────────────────────────────┐
│                     INSTANCE COMPARISON                      │
├────────────┬─────────────────────┬──────────────────────────┤
│  t3.micro  │  1 vCPU, 1 GiB RAM  │  Free Tier eligible ⭐   │
│  t3.small  │  2 vCPU, 2 GiB RAM  │  Development workloads   │
│  m5.large  │  2 vCPU, 8 GiB RAM  │  General purpose         │
│  c5.xlarge │  4 vCPU, 8 GiB RAM  │  Compute intensive       │
│  r5.large  │  2 vCPU, 16 GiB RAM │  Memory intensive        │
└────────────┴─────────────────────┴──────────────────────────┘
```

---

## 💰 EC2 Pricing Models

### Comparison Chart

| Model | Discount | Commitment | Best For |
|-------|----------|------------|----------|
| 🏷️ **On-Demand** | 0% | None | Short-term, unpredictable |
| 📦 **Reserved** | Up to 72% | 1-3 years | Steady-state usage |
| 💵 **Savings Plans** | Up to 72% | 1-3 years | Flexible usage |
| 🎯 **Spot Instances** | Up to 90% | None | Flexible, fault-tolerant |
| 🏢 **Dedicated Hosts** | Varies | None/Reserved | Compliance, licensing |
| 🖥️ **Dedicated Instances** | Varies | None | Hardware isolation |
| ⚡ **Capacity Reservations** | 0% | None | Guaranteed capacity |

### 🎯 Exam Scenarios

```
┌─────────────────────────────────────────────────────────────────┐
│                    PRICING DECISION TREE                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Need 24/7 steady workload? ──────────────────► RESERVED ✓     │
│           │                                                     │
│           ▼ NO                                                  │
│  Can handle interruptions? ───────────────────► SPOT ✓         │
│           │                                                     │
│           ▼ NO                                                  │
│  Short-term, unpredictable? ──────────────────► ON-DEMAND ✓    │
│           │                                                     │
│           ▼ NO                                                  │
│  Regulatory/Licensing needs? ─────────────────► DEDICATED ✓    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 📝 Reserved Instance Types

| Type | Flexibility | Discount |
|------|-------------|----------|
| **Standard RI** | Cannot change instance family | Higher (up to 72%) |
| **Convertible RI** | Can change instance family | Lower (up to 66%) |

### ⚠️ Spot Instance Key Points

> 🚨 **IMPORTANT FOR EXAM**: Spot instances can be **terminated with 2-minute warning**

- ✅ Great for: Batch jobs, data analysis, image processing
- ❌ NOT for: Critical applications, databases

---

## 🔒 Security Groups

> 🛡️ **Security Groups = Virtual Firewall for EC2 instances**

### Key Characteristics

```
┌─────────────────────────────────────────────────────────────┐
│              🔒 SECURITY GROUP RULES                        │
├─────────────────────────────────────────────────────────────┤
│  ✅ ALLOW rules only (no DENY rules)                        │
│  ✅ Stateful (return traffic auto-allowed)                  │
│  ✅ Can reference other SGs or IP addresses                 │
│  ✅ All inbound traffic BLOCKED by default                  │
│  ✅ All outbound traffic ALLOWED by default                 │
│  ✅ Can be attached to multiple instances                   │
│  ✅ Locked to Region/VPC combination                        │
└─────────────────────────────────────────────────────────────┘
```

### 🎯 Common Ports to Remember

| Port | Protocol | Service | 📝 Notes |
|------|----------|---------|----------|
| **22** | TCP | SSH | Linux instances |
| **21** | TCP | FTP | File Transfer |
| **22** | TCP | SFTP | Secure File Transfer |
| **80** | TCP | HTTP | Web traffic |
| **443** | TCP | HTTPS | Secure web traffic |
| **3389** | TCP | RDP | Windows instances |
| **3306** | TCP | MySQL/Aurora | Database |
| **5432** | TCP | PostgreSQL | Database |

### Security Group Rules Diagram

```
              ┌─────────────────────────┐
              │     INTERNET            │
              └───────────┬─────────────┘
                          │
                    ┌─────▼─────┐
                    │ Inbound   │
                    │ Rules     │
                    └─────┬─────┘
                          │ ✅ Allowed
              ┌───────────▼───────────┐
              │    🔒 Security       │
              │       Group          │
              │   ┌───────────┐      │
              │   │  EC2      │      │
              │   │ Instance  │      │
              │   └───────────┘      │
              └───────────┬───────────┘
                          │ ✅ Allowed (default)
                    ┌─────▼─────┐
                    │ Outbound  │
                    │ Rules     │
                    └───────────┘
```

---

## 💾 EC2 Storage Options

### Storage Types Comparison

| Storage | Persistence | Speed | Use Case |
|---------|-------------|-------|----------|
| **EBS** | ✅ Persists | ⚡ Fast | Boot volumes, databases |
| **Instance Store** | ❌ Ephemeral | ⚡⚡ Fastest | Temp data, cache |
| **EFS** | ✅ Persists | ⚡ Fast | Shared file storage |
| **S3** | ✅ Persists | 🔄 Variable | Object storage |

### 📦 EBS Volume Types

```
┌─────────────────────────────────────────────────────────────────┐
│                     EBS VOLUME TYPES                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  🔷 SSD-Based (IOPS performance)                               │
│     ├── gp2/gp3 (General Purpose)  → Boot volumes, dev/test    │
│     └── io1/io2 (Provisioned IOPS) → Mission-critical DBs      │
│                                                                 │
│  🔶 HDD-Based (Throughput performance)                         │
│     ├── st1 (Throughput Optimized) → Big data, log processing  │
│     └── sc1 (Cold HDD)             → Infrequent access data    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 🎯 EBS Quick Reference

| Type | IOPS | Throughput | Boot? | Use Case |
|------|------|------------|-------|----------|
| **gp3** | 16,000 | 1,000 MB/s | ✅ | Default choice |
| **gp2** | 16,000 | 250 MB/s | ✅ | Legacy workloads |
| **io2** | 64,000 | 1,000 MB/s | ✅ | High-performance DBs |
| **st1** | 500 | 500 MB/s | ❌ | Big data |
| **sc1** | 250 | 250 MB/s | ❌ | Archive |

> 💡 **Exam Tip**: Only **SSD volumes (gp2, gp3, io1, io2)** can be boot volumes!

---

## 🖼️ AMI - Amazon Machine Images

### What is an AMI?

> 📦 **AMI = Pre-configured template for EC2 instances**

```
┌─────────────────────────────────────────────────────────────┐
│                        AMI CONTAINS                         │
├─────────────────────────────────────────────────────────────┤
│  🖥️  Operating System                                       │
│  📦  Pre-installed software                                  │
│  🔧  Launch permissions                                      │
│  💾  Block device mapping (EBS volumes)                      │
└─────────────────────────────────────────────────────────────┘
```

### AMI Types

| Type | Description | Example |
|------|-------------|---------|
| **AWS Provided** | Official AWS AMIs | Amazon Linux 2, Windows Server |
| **Marketplace** | Third-party AMIs | Pre-configured apps |
| **Custom** | Your own AMIs | Golden images |
| **Community** | Shared public AMIs | Various |

### 🔄 AMI Lifecycle

```
     CREATE                 REGISTER               LAUNCH
        │                      │                     │
   ┌────▼────┐           ┌─────▼─────┐         ┌────▼────┐
   │ Running │──────────►│   AMI     │────────►│   New   │
   │ Instance│ Snapshot  │ (S3 bkd)  │  Launch │ Instance│
   └─────────┘           └───────────┘         └─────────┘
```

> ⚠️ **Exam Alert**: AMIs are **Region-specific**! To use in another region, you must **copy** it.

---

## 📍 EC2 Placement Groups

### Placement Strategies

```
┌─────────────────────────────────────────────────────────────────┐
│                    PLACEMENT GROUP TYPES                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  🏃 CLUSTER (Low Latency)                                      │
│     ┌─────┬─────┬─────┐                                        │
│     │ EC2 │ EC2 │ EC2 │  Same Rack, Same AZ                    │
│     └─────┴─────┴─────┘  ✅ Low latency ❌ Single point failure│
│                                                                 │
│  📊 SPREAD (High Availability)                                 │
│     ┌─────┐   ┌─────┐   ┌─────┐                                │
│     │ EC2 │   │ EC2 │   │ EC2 │  Different Racks               │
│     └──┬──┘   └──┬──┘   └──┬──┘  ✅ Isolated failures          │
│      Rack1    Rack2    Rack3     ⚠️ Max 7 instances per AZ    │
│                                                                 │
│  🗂️ PARTITION (Large Distributed)                              │
│     ┌─────────┐ ┌─────────┐ ┌─────────┐                        │
│     │Partition│ │Partition│ │Partition│  Logical partitions    │
│     │ EC2 EC2 │ │ EC2 EC2 │ │ EC2 EC2 │  ✅ HDFS, Kafka        │
│     └─────────┘ └─────────┘ └─────────┘                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Quick Comparison

| Strategy | Use Case | Limit | Risk |
|----------|----------|-------|------|
| **Cluster** | HPC, Low latency apps | Same AZ | High (same rack) |
| **Spread** | Critical individual instances | 7 per AZ | Low |
| **Partition** | HDFS, Kafka, Cassandra | 7 partitions per AZ | Medium |

---

## 😴 EC2 Hibernate

### How Hibernate Works

```
┌─────────────────────────────────────────────────────────────┐
│                     HIBERNATE PROCESS                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│    Running Instance                                         │
│    ┌───────────────┐                                        │
│    │   RAM State   │ ──────► Saved to EBS Root Volume       │
│    │   (In-memory) │                                        │
│    └───────────────┘                                        │
│           │                                                 │
│           ▼ Hibernate                                       │
│    ┌───────────────┐                                        │
│    │   Stopped     │ ◄────── Instance Stopped               │
│    │   (Hibernate) │         (EBS volume preserved)         │
│    └───────────────┘                                        │
│           │                                                 │
│           ▼ Start                                           │
│    ┌───────────────┐                                        │
│    │   Running     │ ◄────── RAM restored from EBS          │
│    │   (Resumed)   │         Boot much faster!              │
│    └───────────────┘                                        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Hibernate Requirements

> ✅ **Requirements Checklist**:

- [ ] Instance RAM ≤ 150 GB
- [ ] Root volume must be EBS (encrypted)
- [ ] Root volume must have enough space
- [ ] Supported instance families: C, I, M, R, T (most types)
- [ ] Available for On-Demand, Reserved, and Spot instances
- [ ] Max hibernation time: **60 days**

---

## 🎓 Exam Tips

### ⭐ Key Points to Remember

```
┌─────────────────────────────────────────────────────────────────┐
│                    🎯 EXAM MUST-KNOWS                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  📌 Security Groups are STATEFUL, NACLs are STATELESS          │
│                                                                 │
│  📌 Spot instances can be terminated with 2-min warning         │
│                                                                 │
│  📌 Reserved Instances: 1 or 3 years, up to 72% discount       │
│                                                                 │
│  📌 Instance Store = ephemeral, EBS = persistent               │
│                                                                 │
│  📌 Only SSD volumes can be boot volumes                        │
│                                                                 │
│  📌 AMIs are REGION-SPECIFIC                                   │
│                                                                 │
│  📌 Spread placement: max 7 instances per AZ                   │
│                                                                 │
│  📌 Hibernate: RAM saved to encrypted EBS, max 60 days         │
│                                                                 │
│  📌 User Data runs ONCE at first boot (as root)                │
│                                                                 │
│  📌 SSH = Port 22, RDP = Port 3389, HTTP = 80, HTTPS = 443     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 🧠 Memory Aids

| Concept | Memory Trick |
|---------|--------------|
| Instance Types | **GFIRST**: General, Full-memory, IO, Real-time, Storage, Turbo |
| Spot vs Reserved | **Spot = Spontaneous** (can disappear), **Reserved = Reliable** |
| gp2 vs io2 | **gp = general people**, **io = important operations** |
| Hibernate | Think "**Bear mode**" - sleeps but wakes up where it left off |

---

## 📝 Practice Questions Space

> 🖊️ **Add your own notes below:**

### Q1: 
_Your notes here..._

### Q2: 
_Your notes here..._

### Q3: 
_Your notes here..._

---

## 🔗 Quick Reference Links

- 📖 [AWS EC2 Documentation](https://docs.aws.amazon.com/ec2/)
- 💰 [EC2 Pricing Calculator](https://calculator.aws/)
- 📋 [Instance Type Comparison](https://aws.amazon.com/ec2/instance-types/)

---

<div align="center">

**Good luck with your AWS exam! 🚀**

*Created for AWS Certification Preparation*

</div>
