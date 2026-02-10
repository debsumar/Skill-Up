---
title: "S3 Encryption"
date: 2026-02-10
tags:
  - aws
  - s3
  - encryption
  - sse-s3
  - sse-kms
  - sse-c
  - kms
  - cloudtrail
  - bucket-key
  - saa-c03
---

# S3 Encryption

## Overview

You can encrypt objects in S3 buckets using one of ==four methods==. Three are **server-side encryption** (SSE) — where AWS performs the encryption after receiving the object — and one is **client-side encryption** — where you encrypt before uploading. At the exam, it's critical to understand ==which method fits which situation==, because the exam will present you with a scenario and expect you to pick the right one.

Here's the big picture before we dive deep:

| Method | Key Management | Header Required | Key Takeaway |
|--------|---------------|-----------------|-------------|
| ==SSE-S3== | AWS owns & manages key | `"x-amz-server-side-encryption": "AES256"` | Default, simplest, no extra cost |
| ==SSE-KMS== | You manage key in KMS | `"x-amz-server-side-encryption": "aws:kms"` | Audit trail via CloudTrail, KMS quota limits |
| ==SSE-C== | You provide key each request | Key in HTTP headers (HTTPS required) | AWS never stores your key |
| ==Client-Side== | You manage everything | None — AWS doesn't know it's encrypted | Full control, encrypt before upload |

> [!abstract] TL;DR for the Exam
> - ==SSE-S3== → default, no effort, no cost, no audit trail on key usage
> - ==SSE-KMS== → audit trail, user-controlled keys, but watch out for ==KMS API quotas==
> - ==SSE-C== → you own the key, must use HTTPS, CLI/SDK only
> - ==Client-Side== → you encrypt/decrypt everything yourself, AWS is just storage

---

## SSE-S3 (Amazon S3-Managed Keys)

This is the ==simplest and default== encryption method. The encryption key is ==handled, managed, and owned by AWS==. You never have access to this key, and you don't need to worry about key rotation or management — AWS does it all behind the scenes.

**Key characteristics:**
- Encryption type: ==AES-256==
- ==Enabled by default== for all new buckets and new objects (since January 5, 2023)
- To explicitly request it, set the header: `"x-amz-server-side-encryption": "AES256"`
- ==No additional cost== — included with S3 storage pricing
- No audit trail on key usage (unlike SSE-KMS)

### How SSE-S3 Works

```
┌──────────────────────────────────────────────────────────────┐
│                      SSE-S3 ENCRYPTION FLOW                   │
│                                                               │
│  ┌──────────┐                          ┌──────────────────┐  │
│  │          │   1. Upload object        │                  │  │
│  │   User   │ ──────────────────────▶   │   Amazon S3      │  │
│  │          │   + Header: AES256        │                  │  │
│  └──────────┘                          └────────┬─────────┘  │
│                                                  │            │
│                                          2. S3 pairs object  │
│                                             with S3-Owned    │
│                                             Key (you never   │
│                                             see this key)    │
│                                                  │            │
│                                                  ▼            │
│                                         ┌────────────────┐   │
│                                         │   Encrypted    │   │
│                                         │    Object      │   │
│                                         │  stored in S3  │   │
│                                         └────────────────┘   │
│                                                               │
│  To read: just have S3 GetObject permission — S3 handles     │
│  decryption automatically using the same S3-owned key        │
└──────────────────────────────────────────────────────────────┘
```

> [!note] When to Choose SSE-S3
> Choose SSE-S3 when you don't need to control or audit the encryption key. It's the ==path of least resistance== — zero configuration, zero cost, and it's already on by default. Most workloads that don't have specific compliance requirements around key management will use SSE-S3.

---

## SSE-KMS (AWS KMS-Managed Keys)

Instead of relying on the S3-owned key that you never see, SSE-KMS lets you ==manage your own keys== using the ==AWS Key Management Service (KMS)==. This gives you significantly more control and visibility.

**Advantages over SSE-S3:**
- ==User control== over the encryption key — you can create keys, define who can use them, rotate them, and even disable or delete them
- ==Full audit trail== — every single use of a KMS key is logged in ==CloudTrail==, so you know exactly who encrypted or decrypted what and when
- You specify which KMS key to use in the request header
- You can use the ==default `aws/s3` KMS key== (free) or create your own ==Customer Managed Key (CMK)== (costs ~$1/month per key)

**The critical security difference:** To read an SSE-KMS encrypted object, a user needs ==two things==:
1. ==S3 permissions== to access the object (e.g., `s3:GetObject`)
2. ==KMS permissions== to use the underlying KMS key (e.g., `kms:Decrypt`)

This ==dual-access requirement== is an extra layer of security that SSE-S3 doesn't provide. It's a ==common exam topic==.

### How SSE-KMS Works

```
┌──────────────────────────────────────────────────────────────┐
│                     SSE-KMS ENCRYPTION FLOW                   │
│                                                               │
│  ┌──────────┐                          ┌──────────────────┐  │
│  │          │   1. Upload object        │                  │  │
│  │   User   │ ──────────────────────▶   │   Amazon S3      │  │
│  │          │   + Header: aws:kms       │                  │  │
│  │          │   + KMS Key ID            │                  │  │
│  └──────────┘                          └────────┬─────────┘  │
│                                                  │            │
│                                          2. S3 calls KMS     │
│                                             GenerateDataKey  │
│                                             API              │
│                                                  │            │
│                              ┌───────────────────┘            │
│                              ▼                                │
│                    ┌──────────────────┐                       │
│                    │    AWS KMS       │                       │
│                    │  (your KMS key)  │                       │
│                    └────────┬─────────┘                       │
│                             │                                 │
│                     3. KMS returns a                          │
│                        data encryption key                    │
│                             │                                 │
│                             ▼                                 │
│                    ┌──────────────────┐                       │
│                    │   Encrypted      │                       │
│                    │    Object        │                       │
│                    │  stored in S3    │                       │
│                    └──────────────────┘                       │
│                                                               │
│  To read: need S3 GetObject permission AND KMS Decrypt       │
│  permission — both must be granted or access is denied       │
└──────────────────────────────────────────────────────────────┘
```

> [!warning] KMS API Quota Limitation — ==Heavily Tested on Exam==
> Every upload calls the KMS `GenerateDataKey` API. Every download calls the KMS `Decrypt` API. These API calls count against the ==KMS quota==:
> - ==5,500 to 30,000 requests/second== depending on region
> - Can be increased via the ==Service Quotas Console==
>
> **Exam scenario:** "A company has a very high-throughput S3 bucket encrypted with SSE-KMS and is experiencing throttling errors." → The answer involves either ==requesting a KMS quota increase== or ==enabling S3 Bucket Keys== to reduce KMS API calls.

> [!tip] S3 Bucket Keys — Reducing KMS Costs
> When using SSE-KMS, you can enable ==S3 Bucket Keys==. Instead of calling KMS for every single object operation, S3 generates a ==bucket-level data key== from your KMS key. This bucket-level key is then used to encrypt objects locally within S3, dramatically ==reducing the number of KMS API calls== (and therefore cost and throttling risk).
>
> S3 Bucket Keys are ==enabled by default== when you select SSE-KMS in the console.

### DSSE-KMS (Dual-Layer Server-Side Encryption)

DSSE-KMS was released in June 2023 and applies ==two independent layers of encryption== using KMS keys. Think of it as "double encryption based on KMS" — a stronger variant for workloads that require it.

- Appears as an option in the console alongside SSE-S3 and SSE-KMS
- ==Not commonly tested on the SAA-C03 exam== as of now
- If you see it at the exam, it's just "stronger KMS encryption"

> [!note] DSSE-KMS in Practice
> Unless you have a specific regulatory requirement for dual-layer encryption, ==SSE-KMS is sufficient== for most use cases. DSSE-KMS is there for organizations that need to meet standards like ==CNSSP 15== (Committee on National Security Systems Policy).

---

## SSE-C (Customer-Provided Keys)

With SSE-C, the encryption key is ==managed entirely outside of AWS==. You generate and store the key yourself. When you upload or download an object, you send the key along with the request. Amazon S3 uses the key to encrypt/decrypt, then ==immediately discards it== — S3 never stores your key.

**Key characteristics:**
- ==HTTPS is mandatory== — you're transmitting a secret key, so the connection must be encrypted
- The key must be provided in ==HTTP headers for every single request== (upload, download, copy, etc.)
- ==Not available in the AWS Console== — you can only use SSE-C via the ==CLI or SDK==
- If you lose the key, ==the data is permanently unrecoverable== — AWS cannot help you
- AWS stores a ==salted HMAC hash== of the key for validation purposes, but not the key itself

### How SSE-C Works

```
┌──────────────────────────────────────────────────────────────┐
│                      SSE-C ENCRYPTION FLOW                    │
│                                                               │
│  ┌──────────┐                          ┌──────────────────┐  │
│  │          │   1. Upload object        │                  │  │
│  │   User   │ ──────────────────────▶   │   Amazon S3      │  │
│  │ (manages │   + File data             │                  │  │
│  │  own key)│   + Encryption key        │                  │  │
│  │          │   (in HTTP headers)       │                  │  │
│  │          │   ⚠️ HTTPS REQUIRED       │                  │  │
│  └──────────┘                          └────────┬─────────┘  │
│                                                  │            │
│                                          2. S3 uses your key │
│                                             to encrypt the   │
│                                             object            │
│                                                  │            │
│                                          3. S3 DISCARDS      │
│                                             your key          │
│                                             immediately       │
│                                                  │            │
│                                                  ▼            │
│                                         ┌────────────────┐   │
│                                         │   Encrypted    │   │
│                                         │    Object      │   │
│                                         │  (key NOT      │   │
│                                         │   stored)      │   │
│                                         └────────────────┘   │
│                                                               │
│  To read: must provide the EXACT SAME KEY in the request     │
│  Lost key = lost data forever — AWS cannot recover it        │
└──────────────────────────────────────────────────────────────┘
```

> [!danger] Key Loss = Data Loss
> If you lose the encryption key used for SSE-C, ==the data is permanently unrecoverable==. Amazon S3 does not store your key — it's discarded after each operation. There is ==no recovery mechanism==. Treat SSE-C keys with the same care as private keys.

> [!important] SSE-C is CLI/SDK Only
> You ==cannot use SSE-C from the AWS Console==. The console doesn't have a way to pass custom encryption keys in HTTP headers. You must use the ==AWS CLI== or an ==AWS SDK== (Python Boto3, Java SDK, etc.).

---

## Client-Side Encryption

With client-side encryption, ==everything happens outside of AWS==. The client encrypts the data before sending it to S3 and decrypts it after retrieving it. Amazon S3 is just a storage service — it has ==no knowledge== that the data is encrypted, what algorithm was used, or what key was used.

**Key characteristics:**
- Use a client library like the ==Amazon S3 Client-Side Encryption Library== to simplify implementation
- The client ==fully manages== the keys and the entire encryption/decryption cycle
- AWS has ==zero visibility== into the encryption — no headers, no configuration, no audit trail on the AWS side
- You don't need to tell AWS anything about the encryption when uploading

### How Client-Side Encryption Works

```
┌──────────────────────────────────────────────────────────────┐
│                 CLIENT-SIDE ENCRYPTION FLOW                    │
│                                                               │
│  ┌──────────┐   ┌──────────┐   ┌──────────────┐             │
│  │          │   │ Client's │   │              │             │
│  │  File    │ + │   Key    │ = │  Encrypted   │             │
│  │ (plain)  │   │ (yours)  │   │    File      │             │
│  └──────────┘   └──────────┘   └──────┬───────┘             │
│                                        │                      │
│                  Encryption happens     │                      │
│                  on YOUR machine        │                      │
│                  BEFORE upload          │                      │
│                                        │                      │
│                                        ▼                      │
│                               ┌────────────────┐             │
│                               │   Amazon S3    │             │
│                               │                │             │
│                               │  Stores the    │             │
│                               │  encrypted     │             │
│                               │  blob as-is    │             │
│                               │                │             │
│                               │  Has NO idea   │             │
│                               │  it's encrypted│             │
│                               └────────────────┘             │
│                                                               │
│  To read: download the encrypted blob, decrypt on YOUR       │
│  machine using YOUR key. AWS is never involved in crypto.    │
└──────────────────────────────────────────────────────────────┘
```

> [!note] When to Choose Client-Side Encryption
> Choose client-side encryption when you have ==strict regulatory requirements== that mandate encryption keys never leave your environment, or when you don't trust the cloud provider with your encryption at all. It's the most secure option but also the most ==operationally complex== — you're responsible for key management, rotation, and backup.

---

## Encryption in Transit (SSL/TLS)

Everything we've discussed so far is ==encryption at rest== — protecting data stored on disk. But data also needs protection ==while it's moving== between your application and S3. This is called ==encryption in transit== (also known as ==encryption in flight==, ==SSL==, or ==TLS==).

Amazon S3 exposes two endpoints:

| Endpoint | Protocol | Encryption | Recommendation |
|----------|----------|-----------|----------------|
| `http://bucket.s3.amazonaws.com` | ==HTTP== | ❌ Not encrypted | ⚠️ Not recommended |
| `https://bucket.s3.amazonaws.com` | ==HTTPS== | ✅ Encrypted in flight (TLS) | ✅ ==Always use this== |

**Key points:**
- HTTPS is ==mandatory for SSE-C== — you're transmitting a secret key in the headers
- Most AWS clients, SDKs, and the console use ==HTTPS by default==, so in practice you rarely need to worry about this
- The green lock icon 🔒 in your browser means the connection is using TLS

### Forcing Encryption in Transit with a Bucket Policy

You can create a bucket policy that ==denies any request not using HTTPS==:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ForceHTTPS",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*",
      "Condition": {
        "Bool": {
          "aws:SecureTransport": "false"
        }
      }
    }
  ]
}
```

> [!tip] Exam Tip — `aws:SecureTransport`
> The condition key `aws:SecureTransport` is:
> - ==`true`== when the request uses HTTPS
> - ==`false`== when the request uses HTTP
>
> By setting a ==Deny== on `"false"`, you block all unencrypted HTTP connections. Users using HTTPS are unaffected. This is a ==common exam question== — "How do you force encryption in transit for S3?"

---

## Default Encryption

Since January 5, 2023, all S3 buckets have ==SSE-S3 default encryption enabled automatically==. This means every new object uploaded to any bucket is encrypted at rest, even if the uploader doesn't specify any encryption headers.

**You can customize the default:**
- Change the default encryption to ==SSE-KMS== or ==DSSE-KMS== in the bucket settings
- You ==cannot== set SSE-C as the default (it requires per-request key transmission)

**You can also enforce encryption via bucket policy** — by denying `PutObject` requests that don't include the correct encryption headers. This is ==stricter== than default encryption because it ==rejects non-compliant uploads== rather than silently applying a fallback.

> [!important] Evaluation Order — ==Exam Favorite==
> ==Bucket policies are always evaluated BEFORE default encryption settings.==
>
> If a bucket policy says "deny any PutObject without SSE-KMS headers," and the default encryption is SSE-S3, the upload is ==denied== — the default encryption never gets a chance to apply. The bucket policy wins.

### Example: Forcing SSE-KMS via Bucket Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyNonKMSEncryption",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::my-bucket/*",
      "Condition": {
        "Null": {
          "s3:x-amz-server-side-encryption-aws-kms-key-id": "true"
        }
      }
    }
  ]
}
```

This policy says: "If the `s3:x-amz-server-side-encryption-aws-kms-key-id` header is ==null== (missing), ==deny== the upload." This forces every upload to specify a KMS key.

### Example: Forcing SSE-C via Bucket Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyNonSSEC",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::my-bucket/*",
      "Condition": {
        "Null": {
          "s3:x-amz-server-side-encryption-customer-algorithm": "true"
        }
      }
    }
  ]
}
```

This denies uploads that don't include the SSE-C customer algorithm header.

---

## Hands-On Walkthrough

### Console Encryption Options

When creating a bucket or uploading objects in the AWS Console, you see three encryption options:

| Console Option | Encryption Type | Cost |
|---------------|----------------|------|
| ==Server-side encryption with Amazon S3 managed keys (SSE-S3)== | AES-256, AWS-managed key | Free |
| ==Server-side encryption with AWS KMS keys (SSE-KMS)== | KMS-managed key | Free with `aws/s3` default key; ~$1/month for custom CMK |
| ==Dual-layer server-side encryption with AWS KMS keys (DSSE-KMS)== | Double KMS encryption | Same as SSE-KMS but double the KMS API calls |

> [!note] What's Missing from the Console?
> - ==SSE-C== is ==not available in the console== — you can only use it via CLI or SDK because you need to pass the key in HTTP headers
> - ==Client-side encryption== doesn't appear because AWS doesn't need to know about it — you encrypt before uploading

### Step-by-Step: Creating an Encrypted Bucket

1. Go to ==S3 Console → Create Bucket==
2. Name your bucket and select a region
3. ==Enable Bucket Versioning== (recommended — changing encryption creates new versions)
4. Under ==Default Encryption==, choose your encryption type:
   - SSE-S3 (default) — no further configuration needed
   - SSE-KMS — select a KMS key (default `aws/s3` key is free)
   - DSSE-KMS — select a KMS key (double encryption)
5. If using SSE-KMS, the ==Bucket Key== option is enabled by default (reduces KMS API calls)
6. Click ==Create Bucket==

### Step-by-Step: Changing Encryption on an Existing Object

1. Navigate to the object in S3
2. Click on the object → scroll to ==Server-side encryption settings==
3. Click ==Edit==
4. Choose a new encryption type (e.g., switch from SSE-S3 to SSE-KMS)
5. If SSE-KMS: select the KMS key — the default `aws/s3` key is free, custom keys cost money
6. Click ==Save Changes==

> [!warning] Versioning Impact
> Changing encryption on an existing object ==creates a new version== of that object with the updated encryption settings. The old version retains its original encryption. This is why ==versioning should be enabled== before experimenting with encryption changes.

### KMS Key Options in the Console

When selecting SSE-KMS, you have two choices:

| Option | Description | Cost |
|--------|-------------|------|
| ==AWS managed key (`aws/s3`)== | Default KMS key for the S3 service, shared across your account | ==Free== — no monthly key charge |
| ==Customer managed key (CMK)== | A key you created in KMS with custom policies, rotation, etc. | ==~$1/month== per key + per-request charges |

### Bucket Default Encryption Settings

You can change the default encryption for the entire bucket:
1. Go to the bucket → ==Properties== → scroll to ==Default Encryption==
2. Click ==Edit==
3. Choose SSE-S3, SSE-KMS, or DSSE-KMS
4. If SSE-KMS: the ==Bucket Key== toggle is available (enabled by default)
5. Save changes

All new objects uploaded without explicit encryption headers will use this default.

---

## Encryption Decision Flowchart

```
┌─────────────────────────────────────────────────────────────────┐
│              WHICH ENCRYPTION METHOD SHOULD I USE?               │
│                                                                  │
│  Do you need to audit key usage in CloudTrail?                  │
│  ├── YES ──▶ Do you need to control the key yourself?           │
│  │           ├── YES ──▶ ==SSE-KMS with Customer Managed Key==  │
│  │           └── NO ───▶ ==SSE-KMS with aws/s3 default key==   │
│  │                                                               │
│  └── NO ──▶ Must the key never touch AWS?                       │
│             ├── YES ──▶ Must AWS handle encryption?              │
│             │           ├── YES ──▶ ==SSE-C==                   │
│             │           └── NO ───▶ ==Client-Side Encryption==  │
│             │                                                    │
│             └── NO ───▶ ==SSE-S3== (default, simplest)          │
└─────────────────────────────────────────────────────────────────┘
```

---

## Questions & Answers

> [!question]- Q1: What are the four encryption methods available for S3 objects?
> **Answer:**
> 1. ==SSE-S3== — server-side encryption with Amazon S3-managed keys (AES-256), enabled by default
> 2. ==SSE-KMS== — server-side encryption with AWS KMS-managed keys, provides audit trail via CloudTrail
> 3. ==SSE-C== — server-side encryption with customer-provided keys, AWS never stores the key
> 4. ==Client-Side Encryption== — encrypt before uploading, decrypt after downloading, AWS has zero visibility

> [!question]- Q2: Which encryption method is enabled by default for all new S3 buckets?
> **Answer:**
> ==SSE-S3== has been enabled by default for all new buckets and new objects since January 5, 2023. Objects are encrypted with AES-256 using Amazon S3-managed keys at no additional cost and with no impact on performance.

> [!question]- Q3: Why might SSE-KMS cause performance issues with high-throughput S3 buckets?
> **Answer:**
> Every upload calls the KMS `GenerateDataKey` API and every download calls the KMS `Decrypt` API. These count against the ==KMS quota== (5,500–30,000 requests/second per region). A very high-throughput bucket can hit throttling. Solutions:
> - Request a ==quota increase== via the Service Quotas Console
> - Enable ==S3 Bucket Keys== to reduce the number of KMS API calls

> [!question]- Q4: Why is HTTPS mandatory for SSE-C?
> **Answer:**
> With SSE-C, you transmit the encryption key in HTTP headers. Using HTTP would send the key ==in plaintext over the network==, which is a critical security risk. HTTPS ensures the key is encrypted during transit via TLS.

> [!question]- Q5: What happens if you lose the key used for SSE-C encryption?
> **Answer:**
> The data is ==permanently unrecoverable==. Amazon S3 discards the key immediately after each operation — it never stores it. There is no recovery mechanism. You must provide the exact same key to decrypt the object.

> [!question]- Q6: How do you force all S3 requests to use HTTPS?
> **Answer:**
> Attach a bucket policy with a ==Deny== statement that checks `"aws:SecureTransport": "false"`. This blocks any request made over HTTP while allowing HTTPS requests through.

> [!question]- Q7: What is the difference between default encryption and bucket policy enforcement?
> **Answer:**
> ==Default encryption== is a fallback — if no encryption is specified during upload, S3 applies the default. ==Bucket policy enforcement== is a gate — it ==rejects== uploads that don't meet encryption requirements. Bucket policies are ==evaluated first==, before default encryption applies. A bucket policy is stricter because it denies non-compliant uploads rather than silently fixing them.

> [!question]- Q8: What is the dual-access requirement for SSE-KMS?
> **Answer:**
> To read an SSE-KMS encrypted object, a user needs ==two permissions==:
> 1. ==S3 permission== — `s3:GetObject` to access the object
> 2. ==KMS permission== — `kms:Decrypt` to use the underlying KMS key
>
> If either permission is missing, access is denied. This dual requirement is an ==extra security layer== that SSE-S3 doesn't provide.

> [!question]- Q9: Can you change the encryption method of an existing S3 object?
> **Answer:**
> Yes, but it ==creates a new version== of the object with the updated encryption settings. The old version retains its original encryption. This requires ==versioning to be enabled== on the bucket.

> [!question]- Q10: What is an S3 Bucket Key and why would you use it?
> **Answer:**
> An S3 Bucket Key is a ==bucket-level data key== generated from your KMS key. Instead of calling KMS for every single object operation, S3 uses this bucket-level key locally to encrypt objects. This ==dramatically reduces KMS API calls==, lowering both cost and throttling risk. It's enabled by default when using SSE-KMS in the console.
