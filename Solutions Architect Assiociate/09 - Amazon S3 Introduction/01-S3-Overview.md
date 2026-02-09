---
title: S3 Overview
date: 2026-02-10
tags:
  - aws
  - s3
  - storage
  - saa-c03
---

# S3 Overview

## Overview

==Amazon S3== (Simple Storage Service) is infinitely scaling object storage and one of the main building blocks of AWS. A huge portion of the web relies on S3 — many websites use it as a backbone, and many AWS services integrate with it for storage. Think of S3 as the universal storage layer of the cloud.

## S3 Use Cases

S3 is incredibly versatile because at its core, it's just storage. Here are the primary use cases:

| Use Case | Description |
|----------|-------------|
| **Backup & Storage** | Store files, disk backups, application data |
| **Disaster Recovery** | Move data to another region so if one region goes down, your data is safe elsewhere |
| **Archival** | Archive files cheaply using Glacier classes and retrieve them later at much lower cost |
| **Hybrid Cloud Storage** | Extend on-premises storage into the cloud when you're running out of local capacity |
| **Application Hosting** | Host application assets, media files |
| **Media Hosting** | Store and serve video files, images, audio |
| **Data Lake** | Store massive amounts of data for big data analytics |
| **Software Delivery** | Distribute software updates and packages |
| **Static Websites** | Host HTML/CSS/JS websites directly from S3 |

> [!example] Real-World Examples
> - ==Nasdaq== stores 7 years of data in S3 Glacier for archival
> - ==Sysco== runs analytics on data stored in S3 to gain business insights

## S3 Buckets

S3 stores files into ==buckets==. Buckets are the top-level containers — think of them as root directories. The files inside buckets are called ==objects==.

| Property | Detail |
|----------|--------|
| **Name** | Must be ==globally unique== across all regions and all AWS accounts ever created |
| **Scope** | Defined at the ==region level== — you pick a specific region when creating a bucket |
| **Console View** | S3 console shows buckets from ==all regions== in one view, which makes it look global |

> [!warning] Common Exam Trap
> S3 ==looks like a global service== in the console because you see all buckets across all regions. But buckets are actually ==created in a specific region==. This is a very common mistake for beginners and a frequent exam question.

### Bucket Naming Rules

| Rule | Detail |
|------|--------|
| No uppercase letters | `MyBucket` ❌ → `mybucket` ✅ |
| No underscores | `my_bucket` ❌ → `my-bucket` ✅ |
| 3–63 characters long | Too short or too long won't work |
| Must not be an IP address | `192.168.1.1` ❌ |
| Must start with lowercase letter or number | `-mybucket` ❌ |
| Certain prefix restrictions | `xn--` and `sthree-` prefixes are reserved |

> [!tip] Practical Tip
> As long as you use ==lowercase letters, numbers, and hyphens==, you're good to go. Make your bucket name personal/unique (e.g., `yourname-demo-s3-v1`) to avoid conflicts.

## S3 Objects

Objects are the actual files you store in S3. Every object has a ==key==, which is the full path to the file within the bucket.

### Object Key Structure

```
s3://my-bucket/my_file.txt
               └── key: "my_file.txt"

s3://my-bucket/my_folder/another_folder/my_file.txt
               └── key: "my_folder/another_folder/my_file.txt"
                   ├── prefix: "my_folder/another_folder/"
                   └── object name: "my_file.txt"
```

The key is composed of a ==prefix== (the "folder" path) and the ==object name== (the file name). This is important to understand:

> [!important] No Real Directories
> S3 has ==no concept of directories==. Everything is a flat key with slashes in the name. When you see folders in the S3 console, that's just the UI creating a visual hierarchy based on the slashes in the key. Under the hood, `my_folder/another_folder/my_file.txt` is just one long key string.

### Object Properties

| Property | Detail |
|----------|--------|
| **Value** | The content of the body — the actual file data |
| **Max size** | ==5 TB== (5,000 GB) per object |
| **Multi-part upload** | ==Required== if file > 5 GB; recommended if > 100 MB |
| **Metadata** | List of key-value pairs — can be set by the system or by the user to describe the file |
| **Tags** | Up to ==10== Unicode key-value pairs — very useful for security rules and lifecycle policies |
| **Version ID** | Present if versioning is enabled on the bucket |

> [!note] Multi-Part Upload Math
> If you have a 5 TB file, you must upload it in parts. Each part can be up to 5 GB, so you'd need at least ==1,000 parts== to upload a 5 TB file. Multi-part upload also allows parallel uploads for better performance.

## Accessing Objects

There are different ways to access objects in S3, and understanding the difference is critical:

### Public URL vs Pre-Signed URL

```
Public URL (doesn't work by default):
https://stephane-demo-s3-v5.s3.eu-north-1.amazonaws.com/coffee.jpg
→ Returns 403 Access Denied

Pre-Signed URL (works — has credentials encoded):
https://stephane-demo-s3-v5.s3.eu-north-1.amazonaws.com/coffee.jpg
  ?X-Amz-Algorithm=AWS4-HMAC-SHA256
  &X-Amz-Credential=AKIA...
  &X-Amz-Signature=abc123...
  &X-Amz-Expires=300
→ Returns the image ✅
```

| Method | How It Works | When It Works |
|--------|-------------|---------------|
| **Public URL** | Simple URL with bucket + key | Only if bucket has a ==public bucket policy== |
| **Pre-signed URL** | URL with ==encoded credentials and signature== | Always works for the URL creator (temporary) |
| **Console "Open" button** | Generates a pre-signed URL behind the scenes | Always works for the logged-in user |

> [!tip] Why "Open" Works But the URL Doesn't
> When you click "Open" in the S3 console, AWS generates a ==pre-signed URL== that contains your credentials. That's why it works. But if you copy the simple public URL (the short one without query parameters), it returns ==403 Access Denied== because the bucket blocks public access by default.

## Hands-On Walkthrough

### Creating a Bucket

1. Go to S3 console → **Create bucket**
2. Choose your region (e.g., `eu-north-1`)
3. **Bucket type**: Choose ==General Purpose== (Directory buckets are not on the exam)
4. Enter a ==globally unique name== (e.g., `yourname-demo-s3-v1`)
5. **Object Ownership**: Leave as ==ACLs disabled== (recommended)
6. **Block Public Access**: Leave ==enabled== (maximum security by default)
7. **Bucket Versioning**: Disable for now (we'll enable it later)
8. **Default Encryption**: ==SSE-S3== (Amazon S3 managed keys) with bucket key enabled
9. Click **Create bucket**

### Uploading and Viewing Objects

1. Click into your bucket → **Upload** → **Add files** → choose a file (e.g., `coffee.jpg`)
2. The file appears in the Objects list with its size and type
3. Click on the object to see its properties:
   - **Object URL**: the public URL (won't work yet — 403)
   - Click **Open**: generates a pre-signed URL (works — shows the image)
4. The pre-signed URL is very long because it contains your ==encoded credentials and a signature==

### Creating Folders

1. In the bucket, click **Create folder** → name it (e.g., `images`)
2. Navigate into the folder → upload files there
3. The object key becomes `images/beach.jpg`
4. Remember: this "folder" is just a prefix in the key — not a real directory

### Deleting

1. Select a folder → **Delete** → type `permanently delete` → confirm
2. This deletes all objects with that prefix

## Questions & Answers

> [!question]- Q1: What makes S3 bucket names special compared to other AWS resources?
> **Answer:**
> Bucket names must be ==globally unique== across all AWS regions and all AWS accounts. This is the ==only resource in AWS== that requires global uniqueness. Even if someone in a completely different AWS account already has a bucket named "test", you cannot create a bucket with that name.

> [!question]- Q2: What is an S3 object key and how is it structured?
> **Answer:**
> The key is the ==full path== of the object within the bucket. For example: `my_folder/another_folder/my_file.txt`. It's composed of:
> - ==Prefix==: `my_folder/another_folder/` (the "directory" path)
> - ==Object name==: `my_file.txt` (the file name)
>
> S3 doesn't have real directories — the key is just a flat string with slashes.

> [!question]- Q3: Does S3 have real directories or folders?
> **Answer:**
> No. S3 has ==no concept of directories==. Keys are flat strings that contain slashes. The S3 console creates a folder-like visual experience by parsing the slashes in keys, but under the hood, `images/beach.jpg` is just a single key string, not a file inside a folder.

> [!question]- Q4: What is the maximum object size in S3 and what happens with large files?
> **Answer:**
> The maximum object size is ==5 TB== (5,000 GB). If you're uploading a file larger than ==5 GB==, you ==must== use multi-part upload. For a 5 TB file, you'd need at least 1,000 parts of 5 GB each. Multi-part upload is also recommended for files > 100 MB for better performance.

> [!question]- Q5: Why does the public URL return 403 Access Denied by default?
> **Answer:**
> By default, S3 buckets have ==Block Public Access enabled== and no bucket policy allowing public reads. The public URL only works if you:
> 1. Disable Block Public Access settings
> 2. Attach a bucket policy that allows `s3:GetObject` for principal `*`
>
> Without both steps, the public URL returns 403.

> [!question]- Q6: What is a pre-signed URL and why does the "Open" button work?
> **Answer:**
> A pre-signed URL contains ==encoded credentials and a digital signature== that verifies the requester's identity. When you click "Open" in the S3 console, AWS generates a pre-signed URL with your credentials, so S3 knows it's you making the request and allows access. The URL is temporary and only works for the person who generated it.

> [!question]- Q7: Are S3 buckets global or regional?
> **Answer:**
> Buckets are ==regional== — they are created in a specific AWS region. However, the S3 console shows buckets from ==all regions== in one unified view, which makes it look like a global service. The bucket name must be globally unique, but the bucket itself lives in one region.

> [!question]- Q8: What is multi-part upload and when should you use it?
> **Answer:**
> Multi-part upload splits a large file into smaller parts that can be uploaded ==in parallel==. It's:
> - ==Required== for files > 5 GB
> - ==Recommended== for files > 100 MB
> - Allows parallel uploads for better throughput
> - If one part fails, only that part needs to be re-uploaded

> [!question]- Q9: What metadata and tags can S3 objects have?
> **Answer:**
> - ==Metadata==: key-value pairs set by the system (content-type, last-modified) or by the user (custom metadata)
> - ==Tags==: up to 10 Unicode key-value pairs — very useful for security policies (e.g., only allow access to objects with tag `department=finance`) and lifecycle rules
> - ==Version ID==: automatically assigned if versioning is enabled on the bucket

> [!question]- Q10: What is the default encryption for S3 and what are bucket keys?
> **Answer:**
> The default encryption is ==SSE-S3== (Server-Side Encryption with Amazon S3 managed keys). Every object uploaded to S3 is automatically encrypted at rest. ==Bucket keys== are an optimization that reduces the number of API calls to AWS KMS, lowering encryption costs. Both are enabled by default when creating a new bucket.
