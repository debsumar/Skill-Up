---
title: S3 Security & Bucket Policy
date: 2026-02-10
tags:
  - aws
  - s3
  - security
  - iam
  - bucket-policy
  - saa-c03
---

# S3 Security & Bucket Policy

## Overview

S3 security operates on two dimensions: ==user-based== (IAM policies) and ==resource-based== (bucket policies, ACLs). Understanding how these interact is critical for the exam. The most common and recommended approach today is using ==S3 Bucket Policies==.

## Security Mechanisms

### User-Based Security (IAM Policies)

As an IAM user, you can have IAM policies attached to you that define which S3 API calls you're allowed to make. For example, an IAM policy might say "this user can perform `s3:GetObject` and `s3:PutObject` on bucket X."

### Resource-Based Security

These are policies attached to the S3 resource itself (the bucket or object), rather than to the user:

| Mechanism | Scope | Status |
|-----------|-------|--------|
| **S3 Bucket Policies** | Bucket-wide rules (JSON) | ==Recommended== — most common approach |
| **Object ACL** | Per-object, finer grain | Legacy — ==can be disabled== |
| **Bucket ACL** | Per-bucket | Legacy — ==less common, can be disabled== |

### Encryption

A third layer of security: encrypt objects using encryption keys so even if someone gains access, the data is unreadable without the key.

## The Access Decision Rule

An IAM principal (user, role, or service) can access an S3 object if:

```
   ┌─────────────────────────────────────────────┐
   │  (IAM permissions ALLOW it)                 │
   │           OR                                │
   │  (Resource policy ALLOWS it)                │
   │                                             │
   │           AND                               │
   │                                             │
   │  (There is NO explicit DENY anywhere)       │
   └─────────────────────────────────────────────┘
```

This means access can be granted through ==either== the IAM policy ==or== the bucket policy — you don't need both. But an explicit deny in either one will always win.

## S3 Bucket Policy Deep Dive

Bucket policies are ==JSON documents== and they're the focus of S3 security for the exam. Here's what a typical public-read policy looks like:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicRead",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::example-bucket/*"
    }
  ]
}
```

### Breaking Down Each Field

| Field | What It Does | Example |
|-------|-------------|---------|
| **Resource** | Specifies ==which bucket and objects== the policy applies to | `arn:aws:s3:::example-bucket/*` — the `/*` means all objects inside |
| **Effect** | Whether to ==Allow or Deny== the action | `Allow` |
| **Action** | Which ==API calls== are covered | `s3:GetObject` (read), `s3:PutObject` (write) |
| **Principal** | ==Who== the policy applies to — which account, user, or everyone | `*` means anyone in the world |

> [!important] The `/*` Is Critical
> The `s3:GetObject` action applies to ==objects within the bucket==, not the bucket itself. So the resource must end with `/*` to match all objects. Without it, the policy won't work. This is a very common mistake in the hands-on and on the exam.

### Three Main Use Cases for Bucket Policies

1. **Grant public access** to the bucket (for static websites)
2. **Force encryption** at upload (deny unencrypted uploads)
3. **Grant cross-account access** (allow users from another AWS account)

## Access Scenarios Explained

### Scenario 1: Public Access (Website Visitors)

A website visitor on the internet wants to read files from your S3 bucket.

```
┌──────────────────┐                    ┌──────────────────────────────┐
│  Website Visitor  │──── HTTP GET ────▶│  S3 Bucket                   │
│  (anonymous)      │                   │                              │
│                   │                   │  Bucket Policy:              │
│                   │                   │  Allow * → s3:GetObject      │
│                   │                   │  on arn:aws:s3:::bucket/*    │
└──────────────────┘                    └──────────────────────────────┘
```

The bucket policy with `Principal: "*"` allows ==anyone== to read objects. This is how you make a bucket public for hosting a website.

### Scenario 2: IAM User in Same Account

An IAM user within your AWS account wants to access S3.

```
┌──────────────────┐    ┌────────────────────┐    ┌───────────┐
│  IAM User        │───▶│  IAM Policy        │───▶│ S3 Bucket │
│  (your account)  │    │  attached to user   │    │           │
│                  │    │  Allow s3:*         │    │           │
└──────────────────┘    └────────────────────┘    └───────────┘
```

The IAM policy attached to the user grants S3 permissions. No bucket policy needed — the IAM policy alone is sufficient because the access rule says "IAM permissions allow it ==OR== resource policy allows it."

### Scenario 3: EC2 Instance Accessing S3

An EC2 instance needs to read/write to S3 (e.g., to pull application config or store logs).

```
┌──────────────────┐    ┌────────────────────┐    ┌───────────┐
│  EC2 Instance    │───▶│  IAM Role          │───▶│ S3 Bucket │
│                  │    │  (Instance Profile) │    │           │
│                  │    │  Allow s3:GetObject │    │           │
└──────────────────┘    └────────────────────┘    └───────────┘
```

> [!warning] Never Use IAM Users for EC2
> For EC2 instances, you must use ==IAM Roles== (attached as an Instance Profile), not IAM user credentials. Putting access keys on an EC2 instance is a security anti-pattern.

### Scenario 4: Cross-Account Access

An IAM user in ==another AWS account== wants to access your S3 bucket.

```
┌──────────────────────┐    ┌──────────────────────────────┐
│  IAM User            │───▶│  S3 Bucket (your account)    │
│  (Account B)         │    │                              │
│                      │    │  Bucket Policy:              │
│                      │    │  Allow Account B user        │
│                      │    │  → s3:GetObject              │
└──────────────────────┘    └──────────────────────────────┘
```

For cross-account access, you ==must use a Bucket Policy==. The IAM policy in Account B alone is not enough — the bucket in your account needs to explicitly allow the other account.

## Block Public Access Settings

This is an ==extra layer of security== invented by AWS specifically to prevent company data leaks. It's a safety net that sits above bucket policies.

```
┌─────────────────────────────────────────────────┐
│  Block Public Access (ON by default)            │
│  ┌───────────────────────────────────────────┐  │
│  │  Bucket Policy: Allow * → s3:GetObject    │  │
│  │  (This would make it public, BUT...)      │  │
│  └───────────────────────────────────────────┘  │
│                                                 │
│  Block Public Access OVERRIDES the policy!      │
│  Bucket stays PRIVATE even with public policy.  │
└─────────────────────────────────────────────────┘
```

| Setting | Detail |
|---------|--------|
| **Default** | ==Enabled== — blocks all public access |
| **Override** | Even if bucket policy allows public access, this setting ==wins== |
| **Account-level** | Can be set for ==all buckets== in the account at once |
| **When to disable** | ==Only== if you intentionally want a public bucket (e.g., static website) |

> [!danger] Data Leak Risk
> If you put real company data on an S3 bucket and accidentally make it public, you have a data leak. Block Public Access is your safety net. If you know your bucket should ==never== be public, leave this setting on. If you know ==none== of your buckets should ever be public, set it at the account level.

## Hands-On: Making a Bucket Public with Policy Generator

### Step 1: Disable Block Public Access

1. Go to your bucket → **Permissions** tab
2. Under **Block public access** → click **Edit**
3. ==Untick== "Block all public access"
4. Type `confirm` to acknowledge this is a dangerous action
5. Save → Permissions overview now shows "Objects can be public"

### Step 2: Create a Bucket Policy Using the Policy Generator

1. Scroll down to **Bucket policy** → click **Edit**
2. Click **Policy generator** (opens AWS Policy Generator tool)
3. Fill in:
   - **Policy Type**: S3 Bucket Policy
   - **Effect**: Allow
   - **Principal**: `*` (anyone)
   - **AWS Service**: Amazon S3
   - **Actions**: `GetObject`
   - **ARN**: Copy your bucket ARN from the S3 console, then add ==`/*`== at the end
     - Example: `arn:aws:s3:::stephane-demo-s3-v5/*`
4. Click **Add Statement** → **Generate Policy**
5. Copy the JSON → paste it into the bucket policy editor
6. **Save changes**

### Step 3: Verify

1. Go to your object (e.g., `coffee.jpg`)
2. Copy the **Object URL** (the short public one)
3. Paste it in a new browser tab
4. The image now loads — ==public access is working==

## Questions & Answers

> [!question]- Q1: What are the two main types of S3 security?
> **Answer:**
> 1. ==User-based==: IAM policies attached to IAM users or roles — define which S3 API calls the user can make
> 2. ==Resource-based==: S3 Bucket Policies (JSON rules on the bucket), Object ACLs, and Bucket ACLs — define who can access the bucket/objects
>
> The most common and recommended approach today is S3 Bucket Policies. ACLs are legacy and can be disabled.

> [!question]- Q2: When can an IAM principal access an S3 object?
> **Answer:**
> When the IAM permissions allow it ==OR== the resource policy allows it, ==AND== there is no explicit deny. This means you can grant access through either mechanism — you don't need both. But an explicit deny in either one always wins.

> [!question]- Q3: What are the four fields in a bucket policy statement and what does each do?
> **Answer:**
> - **Resource**: which bucket/objects the policy applies to (e.g., `arn:aws:s3:::bucket/*`)
> - **Effect**: `Allow` or `Deny`
> - **Action**: which API calls (e.g., `s3:GetObject` for reads, `s3:PutObject` for writes)
> - **Principal**: who the policy applies to (`*` = anyone, or a specific account/user ARN)

> [!question]- Q4: Why must you add `/*` to the ARN when using GetObject in a bucket policy?
> **Answer:**
> The `s3:GetObject` action applies to ==objects within the bucket==, not the bucket itself. Objects are represented by the path after the bucket name. The `/*` is a wildcard that matches ==all objects== in the bucket. Without it, the policy won't match any objects and won't work.

> [!question]- Q5: How do you grant EC2 instances access to S3?
> **Answer:**
> Create an ==IAM Role== with the appropriate S3 permissions and attach it to the EC2 instance as an ==Instance Profile==. The EC2 instance assumes this role and gets temporary credentials automatically. Never put IAM user access keys on an EC2 instance — that's a security anti-pattern.

> [!question]- Q6: How does cross-account S3 access work?
> **Answer:**
> You must use an ==S3 Bucket Policy== on the target bucket that explicitly allows the IAM user or role from the other AWS account. The IAM policy in the other account alone is not sufficient — the bucket must explicitly grant access to the external account.

> [!question]- Q7: What is Block Public Access and why does it exist?
> **Answer:**
> Block Public Access is an ==extra safety layer== created by AWS to prevent company data leaks. It ==overrides== bucket policies — even if you set a bucket policy that allows public access, Block Public Access will prevent the bucket from being public. It was invented because too many companies accidentally made their S3 buckets public and leaked sensitive data.

> [!question]- Q8: What are the three main use cases for S3 bucket policies?
> **Answer:**
> 1. ==Grant public access== to the bucket (for static website hosting)
> 2. ==Force encryption== at upload (deny `s3:PutObject` if encryption header is missing)
> 3. ==Grant cross-account access== (allow users/roles from another AWS account)

> [!question]- Q9: What is the recommended security approach for S3 today?
> **Answer:**
> Use ==S3 Bucket Policies== for resource-based access control. ACLs (both Object ACL and Bucket ACL) are legacy and should be ==disabled==. Keep Block Public Access enabled unless you explicitly need a public bucket. Use IAM Roles for EC2 instances, not IAM users.

> [!question]- Q10: What happens if you set a public bucket policy but Block Public Access is still enabled?
> **Answer:**
> The bucket will ==NOT be public==. Block Public Access ==overrides== the bucket policy. You must first disable Block Public Access, then apply the public bucket policy. This two-step process is intentional — it forces you to consciously acknowledge that you're making the bucket public.
