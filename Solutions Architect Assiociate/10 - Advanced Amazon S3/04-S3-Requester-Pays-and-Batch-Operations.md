---
title: "S3 Requester Pays & Batch Operations"
date: 2026-02-10
tags:
  - aws
  - s3
  - requester-pays
  - batch-operations
  - s3-inventory
  - athena
  - saa-c03
---

# S3 Requester Pays & Batch Operations

## S3 Requester Pays

### The Problem: Who Pays for Downloads?

By default, the ==bucket owner pays for everything== — both storage costs AND data transfer costs (networking) when someone downloads objects from the bucket. This is fine for most use cases, but what if you're sharing ==large datasets== with other AWS accounts? The data transfer costs can become enormous.

Consider a research organization that hosts a 10 TB public dataset. Every time another organization downloads the dataset, the hosting organization pays for the data transfer. With hundreds of downloads, the transfer costs could dwarf the storage costs.

```
┌──────────────────────────────────────────────────────────────┐
│              DEFAULT: BUCKET OWNER PAYS EVERYTHING            │
│                                                               │
│  ┌──────────────┐                    ┌──────────────┐        │
│  │  Bucket      │   Download 100 GB  │  Requester   │        │
│  │  Owner       │◀══════════════════│  (other      │        │
│  │              │                    │   account)   │        │
│  │  💰 Pays:   │                    │              │        │
│  │  • Storage   │                    │  Pays:       │        │
│  │  • Transfer  │ ← expensive!      │  • Nothing   │        │
│  │  • Requests  │                    │              │        │
│  └──────────────┘                    └──────────────┘        │
│                                                               │
│  Problem: Owner bears ALL costs, even for others' downloads  │
└──────────────────────────────────────────────────────────────┘
```

### The Solution: Requester Pays

With ==Requester Pays== enabled on the bucket, the ==requester pays for the data transfer== (networking) costs instead of the bucket owner. The bucket owner still pays for storage — that doesn't change.

```
┌──────────────────────────────────────────────────────────────┐
│              REQUESTER PAYS ENABLED                           │
│                                                               │
│  ┌──────────────┐                    ┌──────────────┐        │
│  │  Bucket      │   Download 100 GB  │  Requester   │        │
│  │  Owner       │◀══════════════════│  (other      │        │
│  │              │                    │   account)   │        │
│  │  💰 Pays:   │                    │              │        │
│  │  • Storage   │                    │  💰 Pays:   │        │
│  │    only      │                    │  • Transfer  │        │
│  │              │                    │  • Requests  │        │
│  └──────────────┘                    └──────────────┘        │
│                                                               │
│  Owner only pays storage; requester pays for their downloads │
└──────────────────────────────────────────────────────────────┘
```

### How It Works

When Requester Pays is enabled:

1. The ==bucket owner== continues to pay for ==storage costs== (storing the objects in S3)
2. The ==requester== pays for ==data transfer costs== (downloading objects) and ==request costs== (GET, HEAD, etc.)
3. The requester must include `x-amz-request-payer: requester` in their API requests to acknowledge they'll pay
4. AWS bills the data transfer to the ==requester's AWS account==

### Key Rules

| Rule | Detail |
|------|--------|
| **Bucket owner pays** | ==Storage costs only== (per GB stored per month) |
| **Requester pays** | ==Data transfer (networking) costs== + ==request costs== (per GET, PUT, etc.) |
| **Requester must be authenticated** | ==Anonymous requests are NOT allowed== — AWS needs to know who to bill |
| **Use case** | Sharing ==large datasets== with other AWS accounts (research data, public datasets, media libraries) |
| **Enabling** | Bucket-level setting — applies to all objects in the bucket |

> [!warning] No Anonymous Access
> Requester Pays buckets ==cannot serve anonymous requests==. The requester must be ==authenticated with AWS== so that AWS knows which account to bill for the data transfer. This means:
> - ✅ Other AWS accounts can download (they're authenticated)
> - ❌ Public/anonymous users cannot download (no account to bill)
> - ❌ Cannot be used for public websites or unauthenticated APIs

### Real-World Use Cases

| Use Case | Why Requester Pays |
|----------|-------------------|
| ==Public research datasets== | Research institutions share large genomics/climate datasets; downloaders pay their own transfer costs |
| ==Media asset libraries== | Media companies share large video/image libraries with partners; each partner pays for what they download |
| ==Data marketplace== | Organizations sell or share data; buyers pay for the transfer |
| ==Cross-account data sharing== | Large enterprises share data between business units; each unit pays for their own consumption |

> [!tip] Exam Tip
> If the exam mentions "sharing large datasets with other accounts" and "the bucket owner doesn't want to pay for data transfer," the answer is ==S3 Requester Pays==. The key phrase is "requester must be authenticated" — this eliminates anonymous/public access scenarios.

---

## S3 Batch Operations

### The Problem: Bulk Operations on Existing Objects

What if you need to:
- ==Encrypt all unencrypted objects== in a bucket with millions of objects?
- ==Copy millions of objects== between buckets for migration?
- ==Modify metadata or tags== on thousands of objects?
- ==Restore objects from Glacier== in bulk?
- ==Invoke a Lambda function== on every object for custom processing?

Doing this one object at a time with scripts is slow, error-prone, and hard to track. You'd need to handle retries, track progress, generate reports, and manage failures yourself. ==S3 Batch Operations== solves all of this as a managed service.

### What S3 Batch Operations Does

S3 Batch Operations performs ==bulk operations on existing S3 objects== with a single request. You provide a list of objects, an action, and parameters — S3 handles the rest:

```
┌──────────────────────────────────────────────────────────────┐
│              S3 BATCH OPERATIONS PIPELINE                     │
│                                                               │
│  Step 1: Generate Object List                                │
│  ┌──────────────┐                                            │
│  │  S3 Inventory│──▶ CSV/ORC/Parquet report of ALL objects   │
│  │  (scheduled) │    in your bucket (size, class, encryption │
│  └──────────────┘    status, replication status, etc.)       │
│         │                                                     │
│         ▼                                                     │
│  Step 2: Filter Objects                                      │
│  ┌──────────────┐                                            │
│  │   Amazon     │──▶ SQL query to filter objects             │
│  │   Athena     │    e.g., "WHERE encryption = 'NOT_SSE'"   │
│  └──────────────┘                                            │
│         │                                                     │
│         ▼                                                     │
│  Step 3: Execute Batch Operation                             │
│  ┌──────────────┐                                            │
│  │  S3 Batch    │──▶ Process all filtered objects            │
│  │  Operations  │    with the specified action               │
│  │              │                                            │
│  │  • Object    │    Automatic retries on failure            │
│  │    list      │    Progress tracking                       │
│  │  • Action    │    Completion notifications                │
│  │  • Params    │    Detailed reports                        │
│  └──────────────┘                                            │
└──────────────────────────────────────────────────────────────┘
```

### Supported Operations

| Operation | Use Case | Example |
|-----------|----------|---------|
| ==Modify metadata & properties== | Update content-type, cache-control, etc. on many objects | Change all `.html` files to `text/html` content-type |
| ==Copy objects== | Copy between buckets (migration, backup, compliance) | Copy all objects from `us-east-1` bucket to `eu-west-1` bucket |
| ==Encrypt objects== | Encrypt all unencrypted objects with SSE-S3 or SSE-KMS | Encrypt 10 million unencrypted objects after enabling default encryption |
| ==Modify ACLs or tags== | Add/remove tags or change access control lists | Add `compliance:gdpr` tag to all objects in `eu-data/` prefix |
| ==Restore from Glacier== | Bulk restore archived objects | Restore 100,000 objects from Glacier Flexible for a data migration |
| ==Invoke Lambda function== | Run custom processing on each object | Transcode video files, extract metadata, watermark images |

### Job Components

Every Batch Operations job consists of three components:

| Component | Description | Source |
|-----------|-------------|--------|
| ==Object list (manifest)== | The list of objects to process | S3 Inventory report, CSV manifest file, or S3 Inventory filtered by Athena |
| ==Action== | What to do with each object | Copy, encrypt, tag, restore, invoke Lambda, etc. |
| ==Parameters== | Action-specific settings | Encryption type, destination bucket, Lambda ARN, tag key-value pairs |

### Why S3 Batch Operations Instead of Custom Scripts?

| Feature | Custom Script | S3 Batch Operations |
|---------|--------------|-------------------|
| **Retries** | Manual — you code retry logic | ==Automatic== — built-in retry with exponential backoff |
| **Progress tracking** | Manual — you build a tracking system | ==Built-in== — real-time progress in the console |
| **Completion notifications** | Manual — you set up SNS/email | ==Built-in== — automatic notifications when job completes |
| **Reports** | Manual — you generate your own | ==Generated automatically== — success/failure per object |
| **Scalability** | Limited by your compute (EC2, Lambda concurrency) | ==Managed by AWS== — scales to billions of objects |
| **Cost** | EC2/Lambda compute costs + your development time | Per-job + per-object pricing (often cheaper at scale) |
| **Error handling** | Manual — you handle each failure | ==Managed== — failed objects tracked in completion report |

### The Pipeline: S3 Inventory → Athena → Batch Operations

This is the ==most common exam pattern== for S3 Batch Operations:

```
┌──────────────────────────────────────────────────────────────┐
│  EXAMPLE: Encrypt All Unencrypted Objects                    │
│                                                               │
│  1. S3 Inventory (runs daily/weekly)                         │
│     ┌────────────────────────────────────────────┐           │
│     │ Object Key    │ Size  │ Encryption │ Class │           │
│     │ file1.jpg     │ 5 MB  │ SSE-S3     │ STD   │           │
│     │ file2.pdf     │ 10 MB │ NOT_SSE    │ STD   │  ← target│
│     │ file3.csv     │ 2 MB  │ NOT_SSE    │ IA    │  ← target│
│     │ file4.png     │ 8 MB  │ SSE-KMS    │ STD   │           │
│     └────────────────────────────────────────────┘           │
│                          │                                    │
│                          ▼                                    │
│  2. Athena Query                                             │
│     SELECT key FROM inventory                                │
│     WHERE encryption_status = 'NOT_SSE'                      │
│     → Returns: file2.pdf, file3.csv                          │
│                          │                                    │
│                          ▼                                    │
│  3. S3 Batch Operations                                      │
│     Action: PUT object copy (with SSE-S3 encryption)         │
│     Objects: file2.pdf, file3.csv                            │
│     → Both objects now encrypted ✅                          │
└──────────────────────────────────────────────────────────────┘
```

### S3 Inventory Details

S3 Inventory is the ==starting point== for most Batch Operations workflows:

| Feature | Detail |
|---------|--------|
| **What it does** | Generates a ==scheduled report== of all objects in your bucket |
| **Frequency** | ==Daily or weekly== |
| **Output formats** | ==CSV, ORC, or Apache Parquet== |
| **Destination** | Another S3 bucket (can be same or different account) |
| **Metadata included** | Object key, size, last modified, storage class, ==encryption status==, replication status, ETag, and more |
| **Scope** | Entire bucket or filtered by prefix |

The inventory report is what makes Batch Operations practical at scale — without it, you'd need to list millions of objects yourself using the S3 API (which is paginated and slow).

### Lambda Invocation in Batch Operations

When you choose "Invoke Lambda function" as the action, S3 Batch Operations invokes your Lambda function ==once per object== in the manifest:

```
┌──────────────────────────────────────────────────────────────┐
│  BATCH OPERATIONS + LAMBDA                                   │
│                                                               │
│  Manifest: [obj1, obj2, obj3, ..., obj1000]                  │
│                                                               │
│  S3 Batch ──▶ Lambda(obj1)  → custom processing             │
│           ──▶ Lambda(obj2)  → custom processing              │
│           ──▶ Lambda(obj3)  → custom processing              │
│           ──▶ ...                                            │
│           ──▶ Lambda(obj1000) → custom processing            │
│                                                               │
│  Lambda receives: bucket name, object key, version ID        │
│  Lambda can: transcode, watermark, extract metadata, etc.    │
└──────────────────────────────────────────────────────────────┘
```

The Lambda function receives the ==bucket name, object key, and version ID== as input. It can perform any custom processing — transcode video, extract metadata, apply watermarks, convert file formats, run ML inference, etc.

> [!tip] Key Exam Scenario
> "You need to encrypt all unencrypted objects in an S3 bucket with millions of objects." The answer:
> 1. Use ==S3 Inventory== to get a list of all objects with their encryption status
> 2. Use ==Amazon Athena== to query the inventory and filter for objects where `encryption_status = 'NOT_SSE'`
> 3. Use ==S3 Batch Operations== to encrypt them all at once with SSE-S3 or SSE-KMS
> This is the ==S3 Inventory → Athena → Batch Operations== pipeline — memorize it for the exam.

## Questions & Answers

> [!question]- Q1: What is S3 Requester Pays?
> **Answer:**
> A bucket-level setting where the ==requester pays for data transfer (networking) costs== instead of the bucket owner. The bucket owner still pays for storage. The requester must be ==authenticated with AWS== (no anonymous access allowed) so AWS knows which account to bill. Primary use case: sharing large datasets with other AWS accounts without bearing the transfer costs yourself.

> [!question]- Q2: Can anonymous users access a Requester Pays bucket?
> **Answer:**
> ==No.== The requester must be ==authenticated with AWS== so that AWS knows which account to bill for the data transfer. Anonymous requests are rejected. This means Requester Pays is ==not suitable== for public websites, unauthenticated APIs, or any scenario where the downloader doesn't have an AWS account. The requester must include `x-amz-request-payer: requester` in their API requests.

> [!question]- Q3: What is S3 Batch Operations?
> **Answer:**
> A managed service that performs ==bulk operations on existing S3 objects== with a single request. It supports operations like encrypting objects, copying between buckets, modifying tags/metadata/ACLs, restoring from Glacier, and invoking Lambda functions for custom processing. It provides ==automatic retries==, ==progress tracking==, ==completion notifications==, and ==detailed reports== — all managed by AWS.

> [!question]- Q4: How do you generate the object list for S3 Batch Operations?
> **Answer:**
> The recommended pipeline:
> 1. ==S3 Inventory== — generates a scheduled report (daily/weekly) of all objects in your bucket, including metadata like size, storage class, and encryption status. Output as CSV, ORC, or Parquet.
> 2. ==Amazon Athena== — query the inventory report with SQL to filter objects (e.g., `WHERE encryption_status = 'NOT_SSE'`)
> 3. The filtered list becomes the ==manifest== for S3 Batch Operations
> You can also provide a simple CSV manifest file directly if you already know which objects to process.

> [!question]- Q5: What is the most common exam scenario for S3 Batch Operations?
> **Answer:**
> =="Encrypt all unencrypted objects in an S3 bucket."== The solution:
> 1. ==S3 Inventory== → list all objects with encryption status
> 2. ==Athena== → filter for unencrypted objects (`encryption_status = 'NOT_SSE'`)
> 3. ==S3 Batch Operations== → encrypt them all with SSE-S3 or SSE-KMS
> This is the ==S3 Inventory → Athena → Batch Operations== pipeline. Memorize it.

> [!question]- Q6: Why use S3 Batch Operations instead of a custom script?
> **Answer:**
> S3 Batch Operations provides managed capabilities that would take significant effort to build yourself:
> - ==Automatic retries== with exponential backoff for failed operations
> - ==Real-time progress tracking== in the console
> - ==Completion notifications== when the job finishes
> - ==Automatic report generation== with success/failure per object
> - ==Managed scalability== — AWS handles the compute, scales to billions of objects
> A custom script requires you to build all of this yourself, plus manage the compute infrastructure.

> [!question]- Q7: Can S3 Batch Operations invoke Lambda functions?
> **Answer:**
> ==Yes.== You can invoke a Lambda function ==once per object== in the manifest. The Lambda function receives the ==bucket name, object key, and version ID== as input. This allows ==custom processing== — for example, transcoding video files, extracting metadata, applying watermarks, converting file formats, or running ML inference on each object. S3 Batch handles the orchestration, retries, and reporting.

> [!question]- Q8: What is S3 Inventory and what formats does it support?
> **Answer:**
> S3 Inventory generates a ==scheduled report== (daily or weekly) of all objects in your bucket. The report includes metadata like object key, size, last modified date, storage class, ==encryption status==, replication status, and more. Output formats: ==CSV, ORC, or Apache Parquet==. The report is delivered to another S3 bucket. It's the essential first step in the S3 Batch Operations pipeline — without it, you'd need to list millions of objects via the API.

> [!question]- Q9: Who pays for storage in a Requester Pays bucket?
> **Answer:**
> The ==bucket owner always pays for storage== — this never changes regardless of the Requester Pays setting. Requester Pays only shifts the ==data transfer (networking) costs== and ==request costs== (GET, HEAD, etc.) to the requester. The bucket owner bears the cost of storing the objects; the requester bears the cost of downloading them.

> [!question]- Q10: Can S3 Batch Operations copy objects between buckets in different regions?
> **Answer:**
> ==Yes.== S3 Batch Operations supports ==cross-region copies==. You specify the source objects (via manifest) and the destination bucket (which can be in a different region and even a different AWS account). This is useful for:
> - ==Data migration== — moving data to a new region
> - ==Backup== — creating cross-region copies for disaster recovery
> - ==Compliance== — ensuring data exists in specific regions for regulatory requirements
> S3 Batch handles the cross-region transfer, retries, and reporting automatically.
