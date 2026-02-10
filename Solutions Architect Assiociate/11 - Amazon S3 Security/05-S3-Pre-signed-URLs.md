---
title: "S3 Pre-signed URLs"
date: 2026-02-10
tags:
  - aws
  - s3
  - pre-signed-urls
  - presigned
  - temporary-access
  - security
  - saa-c03
---

# S3 Pre-signed URLs

## Overview

Pre-signed URLs are one of the most ==practical and elegant== S3 security features. They let you grant ==temporary, time-limited access== to a specific S3 object — without making it public, without changing bucket policies, and without requiring the recipient to have an AWS account.

The URL itself carries ==embedded credentials== from the user who generated it. Anyone who has the URL ==inherits that user's permissions== for that specific object — but only until the URL expires. After expiration, the URL becomes useless.

This is the go-to solution whenever you need to ==share a private file temporarily== or ==allow a temporary upload== to a specific location.

---

## How Pre-signed URLs Work

```
┌──────────────────────────────────────────────────────────────────┐
│                  PRE-SIGNED URL FLOW                              │
│                                                                   │
│  ┌──────────────┐                          ┌──────────────────┐  │
│  │              │  1. "Generate a           │                  │  │
│  │  Bucket      │     pre-signed URL        │   Amazon S3      │  │
│  │  Owner       │     for coffee.jpg"       │                  │  │
│  │  (IAM user   │ ──────────────────────▶   │                  │  │
│  │   or role)   │                           │                  │  │
│  │              │ ◀──────────────────────   │                  │  │
│  │              │  2. Returns URL:          │                  │  │
│  │              │     https://bucket.s3.    │                  │  │
│  │              │     amazonaws.com/        │                  │  │
│  │              │     coffee.jpg?           │                  │  │
│  │              │     X-Amz-Credential=... │                  │  │
│  │              │     &X-Amz-Expires=300   │                  │  │
│  │              │     &X-Amz-Signature=... │                  │  │
│  └──────┬───────┘                          └────────┬─────────┘  │
│         │                                           │            │
│         │ 3. Share URL via email,                    │            │
│         │    chat, API response, etc.               │            │
│         ▼                                           │            │
│  ┌──────────────┐                                   │            │
│  │              │  4. Click URL to                   │            │
│  │  Recipient   │     access the file               │            │
│  │  (no AWS     │ ──────────────────────────────────┘            │
│  │   account    │                                                │
│  │   needed!)   │  5. S3 validates the signature                 │
│  │              │     and expiration, then serves                │
│  │              │     the file                                   │
│  └──────────────┘                                                │
│                                                                   │
│  ⏰ After expiration: URL returns Access Denied                  │
└──────────────────────────────────────────────────────────────────┘
```

### URL Anatomy

A pre-signed URL looks like a normal S3 URL with ==query parameters appended==:

```
https://my-bucket.s3.amazonaws.com/coffee.jpg
  ?X-Amz-Algorithm=AWS4-HMAC-SHA256
  &X-Amz-Credential=AKIA.../20240210/eu-west-1/s3/aws4_request
  &X-Amz-Date=20240210T120000Z
  &X-Amz-Expires=300
  &X-Amz-SignedHeaders=host
  &X-Amz-Signature=abc123...
```

The key parameters:
- ==X-Amz-Credential== — identifies who generated the URL
- ==X-Amz-Expires== — how many seconds until the URL expires
- ==X-Amz-Signature== — cryptographic signature that validates the URL hasn't been tampered with

---

## URL Expiration Limits

| Generation Method | Maximum Expiration | Typical Use |
|-------------------|-------------------|-------------|
| ==S3 Console== | Up to ==12 hours== | Quick sharing via the UI |
| ==AWS CLI== | Up to ==168 hours== (7 days) | Programmatic generation for applications |
| ==AWS SDK== | Up to ==168 hours== (7 days) | Application-level URL generation |

> [!tip] Choosing Expiration Time
> Set the ==shortest expiration that meets your use case==. A 5-minute URL for a one-time download is more secure than a 7-day URL. The longer the URL is valid, the greater the risk if it's shared with unintended recipients.

---

## Key Characteristics

| Feature | Detail |
|---------|--------|
| ==Permission inheritance== | Recipient gets the ==same permissions== as the URL generator for that specific object |
| ==Works for GET and PUT== | Can be used for ==downloads== (GET) and ==uploads== (PUT) |
| ==Bucket stays private== | The bucket and object remain private — only the pre-signed URL grants access |
| ==Auto-revocation== | Once expired, the URL ==stops working immediately== — returns Access Denied |
| ==No AWS account needed== | The recipient doesn't need an AWS account, IAM user, or any AWS credentials |
| ==Specific to one object== | Each URL is for ==one specific object== — not a prefix, not a bucket |

---

## Use Cases

| Use Case | How It Works |
|----------|-------------|
| ==Premium content download== | A video streaming service generates pre-signed URLs for logged-in users to download premium videos. Non-logged-in users can't access the content because they don't have the URL. |
| ==Dynamic file sharing== | An application generates URLs on-the-fly for an ever-changing list of users. Each user gets a unique URL with a short expiration. |
| ==Temporary upload== | A mobile app needs to upload a photo to S3. The backend generates a pre-signed PUT URL and sends it to the app. The app uploads directly to S3 without needing AWS credentials. |
| ==Invoice/report sharing== | A SaaS application generates pre-signed URLs for customers to download their monthly invoices. URLs expire after 24 hours. |
| ==Secure file exchange== | Two parties need to exchange files through S3 without either having direct S3 access. Pre-signed URLs act as temporary, secure links. |

> [!tip] Exam Pattern
> If the exam asks about giving ==temporary access to a private S3 object== without:
> - Making the object public
> - Changing bucket policies
> - Creating IAM users for the recipients
>
> The answer is ==pre-signed URLs==. This is a ==very common exam question==.

---

## How the S3 Console "Open" Button Works

Here's something interesting: when you click the ==Open== button on an object in the S3 console, it doesn't use the public Object URL. Instead, it ==generates a pre-signed URL behind the scenes== using your console session credentials.

That's why:
- Clicking the ==Object URL== on a private object → ❌ ==Access Denied==
- Clicking the ==Open button== on the same private object → ✅ ==Works== (because it's a pre-signed URL)

```
┌──────────────────────────────────────────────────────────────┐
│              OBJECT URL vs OPEN BUTTON                        │
│                                                               │
│  Object URL (public):                                        │
│  https://my-bucket.s3.amazonaws.com/coffee.jpg               │
│  → ❌ Access Denied (bucket is private)                      │
│                                                               │
│  Open Button (pre-signed):                                   │
│  https://my-bucket.s3.amazonaws.com/coffee.jpg               │
│  ?X-Amz-Credential=...&X-Amz-Expires=...&X-Amz-Signature=. │
│  → ✅ Works! (carries your console session credentials)      │
└──────────────────────────────────────────────────────────────┘
```

---

## Hands-On Walkthrough

### Method 1: Generate via Console

1. Navigate to a ==private bucket== (one that's not public)
2. Click on an object (e.g., `coffee.jpg`)
3. Notice: clicking the ==Object URL== gives you ==Access Denied== (bucket is private)
4. Notice: clicking ==Open== works — because it generates a pre-signed URL with your credentials
5. To share: click ==Object Actions → Share with a pre-signed URL==
6. Set the expiration: choose ==minutes or hours== (up to 12 hours from the console)
7. Click ==Create pre-signed URL==
8. The URL is copied to your clipboard — share it with anyone
9. The recipient can paste the URL in their browser and ==access the file without an AWS account==

### Method 2: Generate via CLI

```bash
aws s3 presign s3://my-bucket/coffee.jpg \
  --expires-in 3600 \
  --region eu-west-1
```

This generates a pre-signed URL valid for ==3600 seconds (1 hour)==. The `--expires-in` parameter accepts values up to ==604800 seconds (168 hours / 7 days)==.

### Security Considerations

> [!warning] URL Sharing Risks
> Anyone with the pre-signed URL can access the object — there's no additional authentication. Treat pre-signed URLs like ==temporary passwords==:
> - Use the ==shortest possible expiration==
> - Share via ==secure channels== (encrypted email, HTTPS API responses)
> - Don't post pre-signed URLs in ==public forums or logs==

> [!tip] Revoking a Pre-signed URL Early
> You can't directly "revoke" a pre-signed URL. However, you can ==remove the permissions of the IAM user/role== that generated it. Since the URL inherits that user's permissions, removing the permissions effectively ==invalidates the URL== even before it expires.

---

## Questions & Answers

> [!question]- Q1: What is a pre-signed URL?
> **Answer:**
> A URL generated for a specific S3 object that includes ==embedded credentials, an expiration time, and a cryptographic signature==. Anyone with the URL can access the object (GET) or upload to it (PUT) until the URL expires, inheriting the permissions of the user who generated it. No AWS account is needed.

> [!question]- Q2: What is the maximum expiration time for a pre-signed URL?
> **Answer:**
> - S3 Console: up to ==12 hours==
> - AWS CLI / SDK: up to ==168 hours== (7 days)

> [!question]- Q3: Does the S3 bucket need to be public for pre-signed URLs to work?
> **Answer:**
> No. The bucket and object remain ==completely private==. The pre-signed URL grants temporary access by embedding the generator's credentials in the URL itself. This is the whole point — sharing private objects without making them public.

> [!question]- Q4: What permissions does the pre-signed URL recipient get?
> **Answer:**
> They ==inherit the permissions of the user who generated the URL== for that specific object. If the generator has `s3:GetObject` permission, the recipient can download. If the generator has `s3:PutObject` permission, the recipient can upload. The recipient cannot do anything the generator can't do.

> [!question]- Q5: Can pre-signed URLs be used for uploads?
> **Answer:**
> Yes. You can generate a pre-signed URL for a ==PUT operation==, allowing someone to upload a file to a specific location in your S3 bucket without having AWS credentials. This is commonly used in mobile apps and web applications where the client uploads directly to S3.

> [!question]- Q6: What happens when a pre-signed URL expires?
> **Answer:**
> The URL ==stops working immediately==. Any attempt to use it returns an ==Access Denied== error. The object remains in the bucket but is no longer accessible via that expired URL. A new pre-signed URL must be generated for further access.

> [!question]- Q7: Can you revoke a pre-signed URL before it expires?
> **Answer:**
> Not directly — there's no "revoke URL" API. However, you can ==remove or modify the permissions of the IAM user/role== that generated the URL. Since the URL inherits that user's permissions, removing the permissions effectively invalidates the URL even before its expiration time.

> [!question]- Q8: How does the S3 console "Open" button work for private objects?
> **Answer:**
> It generates a ==pre-signed URL behind the scenes== using your console session credentials. That's why clicking "Open" works on private objects while clicking the "Object URL" returns Access Denied. The Open button's URL contains embedded credentials and a signature.

> [!question]- Q9: A company wants to allow customers to download invoices from a private S3 bucket for 24 hours. What's the best approach?
> **Answer:**
> Generate ==pre-signed URLs via the CLI or SDK== (console max is 12 hours) with a ==24-hour expiration== for each invoice. Share the URL with the customer via email or the application. The bucket stays private, no IAM users need to be created for customers, and access is ==automatically revoked after 24 hours==.

> [!question]- Q10: What is the difference between pre-signed URLs and S3 bucket policies for granting access?
> **Answer:**
> | Feature | Pre-signed URLs | Bucket Policies |
> |---------|----------------|-----------------|
> | ==Duration== | Temporary (minutes to 7 days) | Persistent (until policy is changed) |
> | ==Scope== | One specific object | Entire bucket or prefix |
> | ==Recipient== | Anyone with the URL (no AWS account needed) | IAM principals, accounts, IP ranges |
> | ==Setup== | Generate on-the-fly | Configure once, applies to all matching requests |
> | ==Use case== | Temporary, one-off access | Ongoing access patterns |
>
> Use pre-signed URLs for ==temporary, ad-hoc sharing==. Use bucket policies for ==permanent access rules==.
