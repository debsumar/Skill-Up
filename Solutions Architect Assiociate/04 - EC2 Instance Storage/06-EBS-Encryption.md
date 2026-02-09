---
title: EBS Encryption
date: 2026-02-09
tags:
  - aws
  - ec2
  - ebs
  - encryption
  - kms
  - security
  - saa-c03
---

# EBS Encryption

## What You Get with Encrypted EBS

When you create an encrypted EBS volume, ==all of the following are encrypted==:

| What | Encrypted? |
|------|-----------|
| Data at rest inside the volume | ✅ |
| Data in flight (instance ↔ volume) | ✅ |
| All snapshots from the volume | ✅ |
| All volumes created from those snapshots | ✅ |

> [!tip] Transparent Encryption
> Encryption/decryption is handled ==transparently by EC2 and EBS==. You have nothing to do — minimal impact on latency.

## Key Facts

| Feature | Details |
|---------|---------|
| **Algorithm** | ==AES-256== |
| **Key management** | ==AWS KMS== (Key Management Service) |
| **Performance impact** | ==Minimal== (almost none) |
| **Recommendation** | ==Always use encryption== |

## How to Encrypt an Unencrypted EBS Volume

```
Step 1: Create snapshot of unencrypted volume
            │
            ▼
Step 2: Copy snapshot with encryption enabled
            │
            ▼
Step 3: Create new volume from encrypted snapshot
            │
            ▼
Step 4: Attach encrypted volume to instance
```

> [!note] Shortcut
> You can also create an encrypted volume ==directly from an unencrypted snapshot== by enabling encryption during the "Create Volume from Snapshot" step.

### Console Steps

```
Method 1 (Full):
  Snapshot → Actions → Copy Snapshot → Enable Encryption → Select KMS Key
  → Create Volume from encrypted snapshot

Method 2 (Shortcut):
  Unencrypted Snapshot → Actions → Create Volume from Snapshot
  → Enable Encryption → Select KMS Key → Create Volume
```

## Questions & Answers

> [!question]- Q1: What does EBS encryption protect?
> **Answer:**
> ==Everything==:
> - Data at rest inside the volume
> - Data in flight between instance and volume
> - All snapshots created from the volume
> - All volumes created from those snapshots

> [!question]- Q2: What encryption algorithm does EBS use?
> **Answer:**
> ==AES-256==, managed through ==AWS KMS== (Key Management Service). You can use the default `aws/ebs` key or a custom KMS key.

> [!question]- Q3: Does EBS encryption impact performance?
> **Answer:**
> ==Minimal impact==. The encryption/decryption is handled transparently by EC2 and EBS behind the scenes. AWS recommends always using encryption.

> [!question]- Q4: How do you encrypt an existing unencrypted EBS volume?
> **Answer:**
> 1. Create a ==snapshot== of the unencrypted volume
> 2. ==Copy the snapshot== with encryption enabled (or create volume directly with encryption)
> 3. Create a new ==encrypted volume== from the encrypted snapshot
> 4. Attach the encrypted volume to the instance

> [!question]- Q5: Can you directly encrypt an existing unencrypted EBS volume?
> **Answer:**
> ==No==. You cannot toggle encryption on an existing volume. You must go through the snapshot → copy/create with encryption → new volume process.

> [!question]- Q6: If a snapshot is encrypted, will volumes created from it be encrypted?
> **Answer:**
> ==Yes==. Volumes created from encrypted snapshots are ==automatically encrypted==. The encryption chain is maintained.

> [!question]- Q7: If a snapshot is unencrypted, can you create an encrypted volume from it?
> **Answer:**
> ==Yes==. When creating a volume from an unencrypted snapshot, you can ==enable encryption on the fly== and select a KMS key. This is the shortcut method.

> [!question]- Q8: What is the default KMS key for EBS encryption?
> **Answer:**
> ==aws/ebs== — the AWS-managed default key. You can also use your own customer-managed KMS keys for more control over key rotation and access policies.

> [!question]- Q9: Can you enable EBS encryption by default for a region?
> **Answer:**
> ==Yes==. In the EC2 console under EBS settings, you can enable ==encryption by default==. All new EBS volumes and snapshots in that region will be automatically encrypted.

> [!question]- Q10: Is an unencrypted snapshot from an unencrypted volume also unencrypted?
> **Answer:**
> ==Yes==. Snapshots inherit the encryption state of their source volume. An unencrypted volume produces unencrypted snapshots. To get an encrypted snapshot, you must ==copy it with encryption enabled==.
