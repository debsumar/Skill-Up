---
title: AMI (Amazon Machine Image)
date: 2026-02-09
tags:
  - aws
  - ec2
  - ami
  - image
  - saa-c03
---

# AMI (Amazon Machine Image)

## What is an AMI?

An AMI is a ==customization of an EC2 instance== — it packages the OS, software, configuration, and data into a reusable image.

> [!tip] Key Benefit
> Custom AMIs give ==faster boot times== because all software is pre-installed. No need to run User Data scripts for setup.

## AMI Characteristics

| Feature | Details |
|---------|---------|
| **Contains** | OS, software, configuration, monitoring tools |
| **Region** | Built for a ==specific region== |
| **Cross-Region** | Can be ==copied across regions== |
| **Boot speed** | ==Faster== than installing via User Data |
| **Behind the scenes** | Creates ==EBS snapshots== |

## AMI Sources

| Source | Description |
|--------|-------------|
| **Public AMI** | Provided by AWS (e.g., Amazon Linux 2) |
| **Custom AMI** | Created and maintained by you |
| **Marketplace AMI** | Made/sold by vendors on AWS Marketplace |

## AMI Creation Process

```
1. Launch EC2 Instance
   └── Start with a base AMI (e.g., Amazon Linux 2)

2. Customize Instance
   └── Install software, configure settings

3. Stop Instance
   └── Ensures data integrity

4. Create AMI
   └── Right-click → Image and Templates → Create Image
   └── Creates EBS snapshots behind the scenes

5. Launch from Custom AMI
   └── New instances boot with all software pre-installed
```

```
us-east-1a                              us-east-1b
┌──────────┐     ┌──────────┐          ┌──────────┐
│   EC2    │────▶│ Custom   │─────────▶│   EC2    │
│ (source) │     │   AMI    │  Launch  │  (copy)  │
│ + HTTPD  │     │          │          │ + HTTPD  │
└──────────┘     └──────────┘          └──────────┘
  Customize        Create AMI           Instant boot!
  & stop                                No install needed
```

## Why Use Custom AMIs?

| Without AMI | With AMI |
|-------------|----------|
| Launch instance | Launch from AMI |
| Run User Data (install HTTPD, etc.) | Software already installed |
| Wait 2-3 minutes for setup | ==Boot in seconds== |
| Repeat for every instance | Consistent across all instances |

## Questions & Answers

> [!question]- Q1: What is an AMI?
> **Answer:**
> An AMI (Amazon Machine Image) is a ==template for EC2 instances== that contains the OS, software, configuration, and data. It allows you to launch pre-configured instances quickly.

> [!question]- Q2: Why would you create a custom AMI?
> **Answer:**
> For ==faster boot times==. All software is pre-installed in the AMI, so you don't need to run installation scripts (User Data) on every launch. This saves minutes per instance.

> [!question]- Q3: Are AMIs region-specific?
> **Answer:**
> ==Yes==. AMIs are built for a specific region. To use an AMI in another region, you must ==copy it== to that region first.

> [!question]- Q4: What happens behind the scenes when you create an AMI?
> **Answer:**
> AWS creates ==EBS snapshots== of the instance's volumes. The AMI references these snapshots, and when you launch a new instance from the AMI, new EBS volumes are created from those snapshots.

> [!question]- Q5: What are the three sources of AMIs?
> **Answer:**
> 1. **Public AMIs** — provided by AWS (e.g., Amazon Linux 2)
> 2. **Custom AMIs** — created and maintained by you
> 3. **Marketplace AMIs** — made/sold by third-party vendors

> [!question]- Q6: Should you stop an instance before creating an AMI?
> **Answer:**
> It's ==recommended== to stop the instance first to ensure ==data integrity==. While you can create an AMI from a running instance, stopping it guarantees a consistent snapshot.

> [!question]- Q7: Can you sell your own AMIs?
> **Answer:**
> ==Yes==. You can publish and sell AMIs through the ==AWS Marketplace==. Some businesses create AMIs with pre-configured software and sell them to other AWS users.

> [!question]- Q8: How do you launch an instance from a custom AMI?
> **Answer:**
> Launch Instance → My AMIs tab → Select your custom AMI → Configure and launch. The instance will boot with all pre-installed software from the AMI.

> [!question]- Q9: Can you copy an AMI to another region?
> **Answer:**
> ==Yes==. You can copy AMIs across regions to leverage AWS global infrastructure. This is useful for deploying the same application in multiple regions.

> [!question]- Q10: What is the difference between User Data and a custom AMI?
> **Answer:**
> - **User Data**: Script that runs on ==every boot== to install/configure software. Takes time.
> - **Custom AMI**: Software is ==pre-baked== into the image. Instant availability on boot.
> 
> Best practice: Use AMI for base setup, User Data for final customization (e.g., writing config files).
