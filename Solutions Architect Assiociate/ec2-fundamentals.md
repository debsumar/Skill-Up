# 🖥️ EC2 Fundamentals - AWS Exam Notes

---

## 📌 What is EC2?
- **Elastic Compute Cloud** = Infrastructure as a Service (IaaS)
- Virtual servers in the cloud
- Full control over OS, storage, networking

---

## 🏗️ EC2 Components

| Component | Description |
|-----------|-------------|
| **AMI** | Amazon Machine Image - OS template |
| **Instance Type** | CPU, RAM, storage, network capacity |
| **EBS** | Elastic Block Store - persistent storage |
| **Security Group** | Virtual firewall (stateful) |
| **Key Pair** | SSH access (public/private keys) |

---

## 💡 Instance Types - Remember: **FGMCRAT**

```
┌─────────────────────────────────────────────────────┐
│  F - FPGA          │  Genomics, video processing   │
│  G - Graphics      │  ML, video rendering          │
│  M - General       │  Web servers, small DBs       │
│  C - Compute       │  Batch processing, gaming     │
│  R - RAM (Memory)  │  In-memory caching            │
│  A - Arm-based     │  Scale-out workloads          │
│  T - Burstable     │  Variable workloads           │
└─────────────────────────────────────────────────────┘
```

### 🎯 Naming Convention
```
m5.2xlarge
│ │  └── Size (nano → metal)
│ └──── Generation
└────── Family
```

---

## 💰 Purchasing Options

| Option | Use Case | Savings | ⚠️ Exam Tip |
|--------|----------|---------|-------------|
| **On-Demand** | Short, unpredictable | 0% | Pay per second (Linux) |
| **Reserved** | Steady-state apps | Up to 72% | 1 or 3 year commitment |
| **Savings Plans** | Flexible commitment | Up to 72% | $/hour commitment |
| **Spot** | Fault-tolerant jobs | Up to 90% | ❗ Can be interrupted |
| **Dedicated Host** | Compliance/licensing | - | Physical server for you |
| **Dedicated Instance** | Hardware isolation | - | May share hardware |
| **Capacity Reservation** | Guaranteed capacity | 0% | Reserve in specific AZ |

### 🔥 Spot Instance Key Points
```
⚡ Spot Request Types:
   ├── One-time: Request fulfilled once, then done
   └── Persistent: Auto-request when interrupted

❌ To terminate: Cancel request FIRST, then terminate instances
```

---

## 🔒 Security Groups

```
┌──────────────────────────────────────┐
│         SECURITY GROUP               │
│  ┌────────────────────────────────┐  │
│  │     ✅ INBOUND RULES           │  │
│  │     (What can come IN)         │  │
│  └────────────────────────────────┘  │
│  ┌────────────────────────────────┐  │
│  │     ✅ OUTBOUND RULES          │  │
│  │     (What can go OUT)          │  │
│  └────────────────────────────────┘  │
└──────────────────────────────────────┘

🎯 KEY POINTS:
• Stateful (return traffic auto-allowed)
• Only ALLOW rules (no deny)
• Can reference other SGs
• Locked to Region/VPC
```

---

## 🌐 Placement Groups

| Type | Diagram | Use Case |
|------|---------|----------|
| **Cluster** | `[EC2][EC2][EC2]` same rack | Low latency, HPC |
| **Spread** | `[EC2]...[EC2]...[EC2]` diff hardware | Critical apps, max 7/AZ |
| **Partition** | `[P1: EC2,EC2][P2: EC2,EC2]` | Hadoop, Kafka, Cassandra |

---

## 🔗 ENI vs ENA vs EFA

| Feature | ENI | ENA | EFA |
|---------|-----|-----|-----|
| **Full Name** | Elastic Network Interface | Enhanced Networking Adapter | Elastic Fabric Adapter |
| **Speed** | Basic | Up to 100 Gbps | Up to 100 Gbps |
| **Use Case** | Basic networking | High throughput | HPC, ML (OS-bypass) |

---

## 🚀 EC2 Hibernate

```
┌─────────────┐    Hibernate    ┌─────────────┐
│   RAM       │ ──────────────► │   EBS       │
│  (in-use)   │                 │  (saved)    │
└─────────────┘                 └─────────────┘
                   Resume
              ◄──────────────
              (much faster boot!)

📝 Requirements:
• Root EBS must be encrypted
• RAM < 150 GB
• Max 60 days hibernation
• Supported: On-Demand, Reserved, Spot
```

---

## 📝 User Data & Metadata

```bash
# User Data (bootstrap script) - runs ONCE at first boot
#!/bin/bash
yum update -y
yum install httpd -y

# Instance Metadata URL
curl http://169.254.169.254/latest/meta-data/
```

---

## ⚡ Quick Exam Tips

- [ ] **Timeout error** → Security Group issue
- [ ] **Connection refused** → App error or not running
- [ ] **Spot interruption** → 2-minute warning
- [ ] **Reserved Instance** → Can change AZ, instance size (same family)
- [ ] **Dedicated Host** → BYOL (Bring Your Own License)
- [ ] **T2/T3 Unlimited** → Pay extra for sustained high CPU

---

## 🎯 Practice Questions Checklist

- [ ] Difference between Dedicated Host vs Dedicated Instance?
- [ ] When to use Spot vs Reserved?
- [ ] What happens when Spot price exceeds your max?
- [ ] How to achieve lowest latency between instances?
- [ ] Security Group vs NACL differences?

---

*Last Updated: 2026-01-26*
