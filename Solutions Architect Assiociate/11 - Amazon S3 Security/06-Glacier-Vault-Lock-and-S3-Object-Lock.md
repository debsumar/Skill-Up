---
title: "Glacier Vault Lock & S3 Object Lock"
date: 2026-02-10
tags:
  - aws
  - s3
  - glacier
  - vault-lock
  - object-lock
  - worm
  - compliance
  - governance
  - legal-hold
  - retention
  - saa-c03
---

# Glacier Vault Lock & S3 Object Lock

## Overview

Both Glacier Vault Lock and S3 Object Lock implement the ==WORM model== — ==Write Once Read Many==. The idea is simple: you write an object once, and then it ==cannot be modified or deleted== for a specified period (or indefinitely). These features exist for ==compliance and data retention== — think financial records, healthcare data, legal evidence, and regulatory archives.

The exam ==heavily tests the differences== between these two features, especially the two retention modes of S3 Object Lock. Understanding the nuances will get you easy points.

---

## S3 Glacier Vault Lock

Glacier Vault Lock is the ==simpler of the two==. It applies a ==lock policy at the entire vault level==. Once you lock the policy, it becomes ==completely immutable== — nobody can change or delete it. Ever.

### How It Works

1. You create a ==Vault Lock Policy== on your S3 Glacier vault
2. You initiate the lock — this gives you a ==24-hour window== to test the policy
3. After you confirm, the policy is ==locked permanently==
4. Once locked: ==nobody can change it, nobody can delete it== — not the root user, not AWS administrators, not AWS support

```
┌──────────────────────────────────────────────────────────────┐
│                    GLACIER VAULT LOCK                          │
│                                                               │
│  Step 1: Create Vault Lock Policy                            │
│  ┌──────────────────────────────────────────────────────┐    │
│  │  "Deny deletion of any archive in this vault         │    │
│  │   that is less than 365 days old"                    │    │
│  └──────────────────────────────────────────────────────┘    │
│                          │                                    │
│                          ▼                                    │
│  Step 2: Lock the Policy (24-hour test window)               │
│                          │                                    │
│                          ▼                                    │
│  Step 3: Policy is PERMANENTLY LOCKED                        │
│  ┌──────────────────────────────────────────────────────┐    │
│  │  ❌ Cannot modify the policy                         │    │
│  │  ❌ Cannot delete the policy                         │    │
│  │  ❌ Cannot delete objects that violate the policy    │    │
│  │  ❌ Not even root user can override                  │    │
│  │  ❌ Not even AWS support can override                │    │
│  │                                                      │    │
│  │  ✅ Objects are protected according to the policy    │    │
│  │  ✅ Perfect for regulatory compliance                │    │
│  └──────────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────────┘
```

> [!important] Truly Immutable
> Once a Vault Lock Policy is locked, it ==can never be changed or deleted== — not by the root user, not by AWS support, not by anyone. This is ==by design== for regulatory compliance. If you lock the wrong policy, you're stuck with it. The 24-hour test window exists specifically to prevent this mistake.

> [!note] When to Use Glacier Vault Lock
> Use Glacier Vault Lock when you need ==vault-wide, immutable protection== for archived data. Common scenarios: financial records that must be retained for 7 years, healthcare records under HIPAA, legal documents under SEC Rule 17a-4.

---

## S3 Object Lock

S3 Object Lock is similar in concept but ==much more granular and flexible==. Instead of locking an entire vault with one policy, you can lock ==individual object versions== within an S3 bucket, each with its own retention settings.

### Prerequisites

- ==Versioning must be enabled== on the bucket (Object Lock works on specific version IDs)
- Object Lock can only be enabled ==when creating the bucket== — you can't add it to an existing bucket

### Two Retention Modes

This is where the exam focuses. S3 Object Lock offers ==two retention modes== with very different behaviors:

#### Compliance Mode — The Strictest Protection

In Compliance mode, a protected object version ==cannot be overwritten or deleted by ANY user==, including the root user. The retention mode itself cannot be changed, and the retention period cannot be shortened.

Think of it as: ==once set, absolutely nobody can touch it until the retention period expires==.

| What's Protected | Can Anyone Override? |
|-----------------|:-------------------:|
| Delete the object version | ❌ ==Nobody== — not even root |
| Overwrite the object version | ❌ ==Nobody== — not even root |
| Change the retention mode | ❌ ==Nobody== — not even root |
| Shorten the retention period | ❌ ==Nobody== — not even root |
| Extend the retention period | ✅ Yes — can be extended |

#### Governance Mode — Protection with Admin Override

In Governance mode, ==most users== cannot overwrite or delete a protected object version. However, ==some users with special IAM permissions== can bypass the protection.

Think of it as: ==protected by default, but admins have a master key==.

| What's Protected | Can Anyone Override? |
|-----------------|:-------------------:|
| Delete the object version | ❌ Most users / ✅ ==Admins with special IAM permissions== |
| Overwrite the object version | ❌ Most users / ✅ ==Admins with special IAM permissions== |
| Change the retention mode | ✅ ==Admins can change== |
| Shorten the retention period | ✅ ==Admins can shorten== |
| Extend the retention period | ✅ Yes — can be extended |

The special IAM permission needed to bypass Governance mode is: `s3:BypassGovernanceRetention`

```
┌──────────────────────────────────────────────────────────────────┐
│                    COMPLIANCE vs GOVERNANCE                        │
│                                                                   │
│  ┌─────────────────────────────┐  ┌─────────────────────────────┐│
│  │     COMPLIANCE MODE         │  │     GOVERNANCE MODE          ││
│  │                             │  │                              ││
│  │  🔒 Maximum protection      │  │  🔒 Protection with          ││
│  │                             │  │     admin override           ││
│  │  Who can delete?            │  │                              ││
│  │  → ==NOBODY==               │  │  Who can delete?             ││
│  │    Not even root user       │  │  → Most users: ❌ No         ││
│  │                             │  │  → Admins with               ││
│  │  Who can change settings?   │  │    BypassGovernance          ││
│  │  → ==NOBODY==               │  │    Retention: ✅ Yes         ││
│  │    Cannot change mode       │  │                              ││
│  │    Cannot shorten period    │  │  Who can change settings?    ││
│  │                             │  │  → Admins: ✅ Yes            ││
│  │  Use case:                  │  │                              ││
│  │  Regulatory compliance      │  │  Use case:                   ││
│  │  (SEC, HIPAA, FINRA)       │  │  Internal governance         ││
│  │                             │  │  with flexibility            ││
│  └─────────────────────────────┘  └─────────────────────────────┘│
│                                                                   │
│  Both modes require a ==RETENTION PERIOD==                       │
│  Both modes allow ==EXTENDING== the retention period             │
└──────────────────────────────────────────────────────────────────┘
```

### Retention Period

Both modes require you to set a ==retention period== — a specific duration during which the object is protected. Key rules:

| Action | Compliance Mode | Governance Mode |
|--------|:--------------:|:--------------:|
| Set a retention period | ✅ Required | ✅ Required |
| ==Extend== the period | ✅ Allowed | ✅ Allowed |
| ==Shorten== the period | ❌ ==Not allowed== | ✅ Admins can shorten |
| Object protected until period expires | ✅ Yes | ✅ Yes (unless admin overrides) |

---

### Legal Hold — Independent Protection

Legal Hold is a ==completely separate mechanism== from retention modes. It's an on/off switch that protects an object ==indefinitely==, regardless of any retention period.

**Key characteristics:**
- ==No expiration date== — stays until explicitly removed
- ==Independent of retention== — even if the retention period expires, the legal hold keeps the object protected
- Requires the `s3:PutObjectLegalHold` IAM permission to place or remove
- Can be placed on ==any object==, regardless of whether it has a retention period

**Use case:** A legal investigation requires preserving evidence. You place a legal hold on the relevant objects. Even if their retention periods expire, the objects remain protected until the legal team removes the hold.

```
┌──────────────────────────────────────────────────────────────┐
│                    LEGAL HOLD EXAMPLE                          │
│                                                               │
│  Object: financial-report-2023.pdf                           │
│                                                               │
│  Retention: Compliance Mode, 1 year (expires Dec 2024)       │
│  Legal Hold: ON (placed during fraud investigation)          │
│                                                               │
│  Timeline:                                                    │
│  ├── Jan 2024: Object created, retention starts              │
│  ├── Dec 2024: Retention period expires                      │
│  │             ⚠️ But Legal Hold is still ON                  │
│  │             ❌ Object CANNOT be deleted                    │
│  ├── Mar 2025: Investigation concludes                       │
│  │             Legal team removes Legal Hold                 │
│  │             (requires s3:PutObjectLegalHold)              │
│  └── Mar 2025: Object can now be deleted ✅                  │
│                                                               │
│  Key insight: Legal Hold is INDEPENDENT of retention.        │
│  It's an additional layer that overrides everything else.    │
└──────────────────────────────────────────────────────────────┘
```

> [!tip] Exam Tip — Legal Hold
> If the exam mentions ==indefinite protection== that's ==independent of retention periods== and can be ==placed and removed by authorized users==, the answer is ==Legal Hold==. Think of it as a "freeze" on the object for legal purposes.

---

## Glacier Vault Lock vs S3 Object Lock — Complete Comparison

| Feature | Glacier Vault Lock | S3 Object Lock |
|---------|-------------------|----------------|
| ==Scope== | Entire vault (all objects) | Individual object versions |
| ==Granularity== | One policy for everything | Per-object settings |
| ==Requires versioning== | N/A (Glacier concept) | ✅ Yes, must be enabled |
| ==Policy mutability== | ==Immutable== once locked — forever | Depends on retention mode |
| ==Retention modes== | Single vault-wide policy | ==Compliance== or ==Governance== |
| ==Admin override== | ❌ Never — nobody can override | ✅ Only in Governance mode |
| ==Legal Hold== | ❌ Not available | ✅ Available (independent of retention) |
| ==Root user override== | ❌ Not even root | ❌ Not in Compliance / ✅ In Governance (with IAM) |
| ==Use case== | Long-term archival compliance | Flexible per-object protection |

---

## Questions & Answers

> [!question]- Q1: What does WORM stand for and what does it mean?
> **Answer:**
> ==Write Once Read Many==. It means you can write (upload) an object once, and then it can be read many times but ==cannot be modified or deleted== for a specified period or indefinitely. This model is used for compliance and data retention requirements.

> [!question]- Q2: What is the key difference between Glacier Vault Lock and S3 Object Lock?
> **Answer:**
> Glacier Vault Lock applies at the ==entire vault level== with a single, ==permanently immutable== policy. S3 Object Lock applies at the ==individual object version level== within an S3 bucket and offers two retention modes (Compliance and Governance) with optional Legal Hold. S3 Object Lock is more granular and flexible.

> [!question]- Q3: Can anyone change a Glacier Vault Lock Policy once it's locked?
> **Answer:**
> No. Once locked, the policy is ==completely immutable==. Not the root user, not AWS administrators, not AWS support — ==nobody can change or delete it==. This is by design for regulatory compliance. The 24-hour test window before locking exists to prevent mistakes.

> [!question]- Q4: What is the difference between Compliance and Governance retention modes?
> **Answer:**
> ==Compliance mode==: nobody can delete, overwrite, change the mode, or shorten the retention period — ==not even the root user==. It's the strictest protection.
>
> ==Governance mode==: most users are blocked, but users with the `s3:BypassGovernanceRetention` IAM permission can delete objects, change retention settings, or shorten the period. It provides protection with an ==admin escape hatch==.

> [!question]- Q5: What is a Legal Hold in S3 Object Lock?
> **Answer:**
> A ==separate, independent protection== that locks an object ==indefinitely== regardless of any retention period. It has no expiration date and stays until someone with the `s3:PutObjectLegalHold` IAM permission explicitly removes it. It's used for preserving evidence during legal investigations.

> [!question]- Q6: Can you extend a retention period in Compliance mode?
> **Answer:**
> Yes. You can ==extend== the retention period in both Compliance and Governance modes. However, in Compliance mode you ==cannot shorten== it. This ensures that protection can only become stronger, never weaker.

> [!question]- Q7: What IAM permission is needed to place or remove a Legal Hold?
> **Answer:**
> `s3:PutObjectLegalHold` — this permission allows a user to place a legal hold on an object or remove an existing one. It's separate from the retention mode permissions.

> [!question]- Q8: A company needs to store financial records that cannot be deleted by anyone for 7 years, including administrators. Which feature should they use?
> **Answer:**
> ==S3 Object Lock in Compliance mode== with a 7-year retention period. In Compliance mode, nobody — not even the root user or AWS support — can delete the objects until the retention period expires. This meets regulatory requirements like SEC Rule 17a-4.

> [!question]- Q9: What is the prerequisite for enabling S3 Object Lock?
> **Answer:**
> ==Versioning must be enabled== on the bucket, and Object Lock must be enabled ==at bucket creation time== — you cannot add it to an existing bucket. Object Lock protects specific object versions from deletion.

> [!question]- Q10: An object has a 1-year retention period in Compliance mode and a Legal Hold. What happens after 1 year?
> **Answer:**
> The object is ==still protected== because the Legal Hold is ==independent== of the retention period. Even though the 1-year retention has expired, the Legal Hold keeps the object locked ==indefinitely== until someone with the `s3:PutObjectLegalHold` permission explicitly removes it. Both protections must be cleared before the object can be deleted.
