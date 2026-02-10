---
title: "S3 Performance"
date: 2026-02-10
tags:
  - aws
  - s3
  - performance
  - transfer-acceleration
  - multipart-upload
  - byte-range-fetches
  - saa-c03
---

# S3 Performance

## Overview

Amazon S3 is ==extremely high performance== out of the box. It automatically scales to handle massive request rates with very low latency — no provisioning, no capacity planning. But understanding the baseline numbers, the concept of prefixes, and the optimization techniques (multi-part upload, transfer acceleration, byte-range fetches) is ==critical for the exam==.

The key takeaway: S3 performance is ==per prefix==, not per bucket. Since there's no limit on the number of prefixes, you can achieve virtually ==unlimited throughput== by distributing objects across multiple prefixes. Combined with multi-part upload for large files and transfer acceleration for cross-region transfers, S3 can handle any workload.

## Baseline Performance

| Metric | Value |
|--------|-------|
| **First byte latency** | ==100-200 milliseconds== |
| **PUT/COPY/POST/DELETE** | ==3,500 requests/second/prefix== |
| **GET/HEAD** | ==5,500 requests/second/prefix== |
| **Prefix limit** | ==No limit== on the number of prefixes in a bucket |

These numbers are ==per prefix==, which is the key to understanding S3 performance. If you need more throughput, you don't need to create more buckets — you just spread your objects across more prefixes.

> [!important] Per Prefix, Not Per Bucket
> The request limits are ==per prefix==, not per bucket. A single bucket with 100 prefixes can handle 100 × 5,500 = ==550,000 GET requests per second==. There's no limit on the number of prefixes, so S3 throughput is effectively unlimited if you design your key structure well.

## Understanding Prefixes

The ==prefix== is everything between the bucket name and the object name — essentially the "folder path." Each unique prefix gets its own independent throughput allocation:

```
┌─────────────────────────────────────────────────────────────────┐
│              S3 PREFIXES EXPLAINED                              │
│                                                                 │
│  Object Key                          Prefix                    │
│  ─────────────────────────────────   ──────────────────────    │
│  bucket/folder1/sub1/file            /folder1/sub1/            │
│  bucket/folder1/sub2/file            /folder1/sub2/            │
│  bucket/folder2/sub1/file            /folder2/sub1/            │
│  bucket/folder3/file                 /folder3/                 │
│                                                                 │
│  Each prefix gets its OWN:                                     │
│    • 3,500 PUT/COPY/POST/DELETE per second                     │
│    • 5,500 GET/HEAD per second                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Scaling with Prefixes

If you spread reads across ==4 prefixes== evenly:
- 4 × 5,500 = ==22,000 GET requests/second==

If you spread writes across ==4 prefixes== evenly:
- 4 × 3,500 = ==14,000 PUT requests/second==

If you have ==10 prefixes==:
- 10 × 5,500 = ==55,000 GET requests/second==
- 10 × 3,500 = ==35,000 PUT requests/second==

The math is simple: ==more prefixes = more throughput==. And since there's no limit on prefixes, you can scale to any level.

> [!tip] Exam Tip
> If the exam asks about S3 performance limits, remember: ==3,500 PUT and 5,500 GET per second per prefix==. If you need more throughput, ==spread objects across multiple prefixes==. The exam may give you a scenario with 4 prefixes and ask you to calculate the maximum GET throughput (answer: 22,000/second).

### Prefix Design Best Practices

| Pattern | Example | Throughput |
|---------|---------|-----------|
| ==Date-based== | `logs/2026/02/10/file.log` | Each day gets its own prefix — good for time-series data |
| ==User-based== | `users/user123/photos/file.jpg` | Each user gets their own prefix — good for multi-tenant apps |
| ==Random prefix== | `a1b2c3/file.dat` | Random prefixes maximize distribution — good for high-throughput workloads |
| ==Single prefix== | `data/file.dat` | ⚠️ All objects share one prefix — bottleneck at 3,500 PUT/5,500 GET |

## Optimizing Uploads: Multi-Part Upload

For large files, ==multi-part upload== splits the file into smaller parts and uploads them ==in parallel==, maximizing your network bandwidth:

```
┌──────────────────────────────────────────────────────────────┐
│              MULTI-PART UPLOAD                                 │
│                                                               │
│  Big File (e.g., 5 GB)                                       │
│  ┌──────┬──────┬──────┬──────┬──────┐                       │
│  │Part 1│Part 2│Part 3│Part 4│Part 5│                       │
│  └──┬───┘└──┬──┘└──┬──┘└──┬──┘└──┬──┘                       │
│     │       │      │      │      │                           │
│     │  ┌────┘      │      └────┐ │    PARALLEL UPLOAD        │
│     ▼  ▼           ▼          ▼ ▼    (all parts at once)     │
│  ┌─────────────────────────────────┐                         │
│  │         Amazon S3               │                         │
│  │                                 │                         │
│  │  S3 reassembles all parts      │                         │
│  │  into the complete file once   │                         │
│  │  all parts are uploaded        │                         │
│  └─────────────────────────────────┘                         │
└──────────────────────────────────────────────────────────────┘
```

| Rule | Detail |
|------|--------|
| ==Recommended== | For files > ==100 MB== |
| ==Required== | For files > ==5 GB== (S3 won't accept a single PUT for files > 5 GB) |
| **How it works** | File split into parts (each 5 MB to 5 GB), uploaded in parallel, S3 reassembles |
| **Benefit** | ==Maximizes bandwidth== — uses all available network capacity |
| **Resilience** | If one part fails, ==only that part== needs to be retried, not the entire file |
| **Part count** | Up to ==10,000 parts== per upload |

### How Multi-Part Upload Works

1. ==Initiate==: Client sends `CreateMultipartUpload` request → S3 returns an upload ID
2. ==Upload parts==: Client splits the file and uploads each part in parallel, referencing the upload ID
3. ==Complete==: Client sends `CompleteMultipartUpload` with the list of parts → S3 assembles the final object
4. ==Abort== (if needed): Client can abort the upload → S3 deletes all uploaded parts

> [!warning] Incomplete Multi-Part Uploads
> If a multi-part upload is initiated but never completed (client crash, network failure), the uploaded parts ==remain in S3 and consume storage==. These parts are invisible in the console's normal view. Use a ==lifecycle rule== to automatically delete incomplete multi-part uploads after a specified number of days (e.g., 7 days). See [[01-S3-Lifecycle-Rules|S3 Lifecycle Rules]] for details.

## Optimizing Uploads: S3 Transfer Acceleration

For ==cross-region uploads==, Transfer Acceleration uses AWS ==edge locations== (200+ worldwide) to speed up the transfer by minimizing the distance data travels over the public internet:

```
┌──────────────────────────────────────────────────────────────┐
│              S3 TRANSFER ACCELERATION                         │
│                                                               │
│  WITHOUT Transfer Acceleration:                              │
│  ┌──────┐                                        ┌────────┐ │
│  │ User │═══════ Public Internet (slow) ════════▶│S3 Bucket│ │
│  │ (USA)│        (long distance)                 │(Sydney) │ │
│  └──────┘                                        └────────┘ │
│                                                               │
│  WITH Transfer Acceleration:                                 │
│  ┌──────┐  Public    ┌───────────┐  AWS Private  ┌────────┐ │
│  │ User │══Internet═▶│   Edge    │══Network═════▶│S3 Bucket│ │
│  │ (USA)│  (short)   │  Location │  (fast, long) │(Sydney) │ │
│  └──────┘            │  (USA)    │               └────────┘ │
│                       └───────────┘                          │
│                                                               │
│  Key: Minimize public internet, maximize AWS private network │
└──────────────────────────────────────────────────────────────┘
```

| Feature | Detail |
|---------|--------|
| **How it works** | Upload to nearest ==edge location== (200+ worldwide), then transfer over ==fast AWS private network== to S3 bucket in target region |
| **Why it's faster** | ==Minimizes public internet== traversal (slow, variable latency) and ==maximizes AWS backbone== usage (fast, consistent) |
| **Compatible with** | ==Multi-part upload== — combine both for maximum speed on large cross-region uploads |
| **Use case** | Uploading files to a bucket in a ==distant region== (e.g., USA user → Sydney bucket) |
| **Cost** | Additional charge per GB transferred — but AWS won't charge if acceleration doesn't improve speed |
| **Edge locations** | ==200+== worldwide (far more than the ~30 AWS regions) |

### Transfer Acceleration vs CloudFront

| Aspect | Transfer Acceleration | CloudFront |
|--------|----------------------|------------|
| **Primary purpose** | Speed up ==uploads to S3== | Speed up ==downloads/content delivery== |
| **Direction** | Optimized for ==ingress== (upload) | Optimized for ==egress== (download) with caching |
| **Caching** | ==No caching== — every request goes to S3 | ==Caches content== at edge locations |
| **Best for** | Large file uploads from distant users | Serving static content to many users |
| **S3-specific** | ==Yes== — only works with S3 | General CDN — works with any origin |

> [!note] Edge Locations vs Regions
> There are ==200+ edge locations== worldwide but only ~30 AWS regions. Edge locations are much closer to end users, so the "public internet" portion of the transfer is very short. The long-distance transfer happens over AWS's ==fast, private backbone network== with consistent low latency.

## Optimizing Downloads: S3 Byte-Range Fetches

For large file downloads, ==byte-range fetches== let you download specific byte ranges of a file ==in parallel==, similar to how multi-part upload parallelizes uploads:

```
┌──────────────────────────────────────────────────────────────┐
│              S3 BYTE-RANGE FETCHES                            │
│                                                               │
│  Large File in S3 (e.g., 1 GB)                              │
│  ┌──────────────────────────────────────────────┐            │
│  │ Bytes 0-249MB │ Bytes 250-499MB │ Bytes 500MB+│           │
│  └──────┬────────┘└───────┬────────┘└──────┬─────┘           │
│         │                 │                │                  │
│         ▼                 ▼                ▼                  │
│    GET Range:       GET Range:        GET Range:             │
│    bytes=0-249MB    bytes=250-499MB   bytes=500MB-           │
│         │                 │                │                  │
│         │    (all three requests in parallel)                 │
│         │                 │                │                  │
│         └─────────────────┼────────────────┘                  │
│                           ▼                                   │
│                   Client reassembles                          │
│                   the complete file                           │
│                   from all byte ranges                        │
└──────────────────────────────────────────────────────────────┘
```

### Two Use Cases for Byte-Range Fetches

| Use Case | How | Benefit | Example |
|----------|-----|---------|---------|
| ==Speed up downloads== | Request multiple byte ranges in parallel | Parallelize GET requests, ==maximize bandwidth== | Download a 5 GB video file in 10 parallel chunks |
| ==Partial file retrieval== | Request only specific bytes | Get file metadata ==without downloading the entire file== | Read first 50 bytes (file header) to determine file type |

### Resilience

If a byte-range fetch fails, you only need to ==retry that specific range==, not the entire file. This makes downloads much more resilient to network issues — especially important for large files over unreliable connections.

```
┌──────────────────────────────────────────────────────────────┐
│  BYTE-RANGE FETCH RESILIENCE                                 │
│                                                               │
│  Request 1: bytes=0-99MB      ✅ Success                    │
│  Request 2: bytes=100-199MB   ❌ Failed → Retry only this   │
│  Request 3: bytes=200-299MB   ✅ Success                    │
│  Request 4: bytes=300-399MB   ✅ Success                    │
│                                                               │
│  Only Request 2 needs retry — not the entire 400 MB file    │
└──────────────────────────────────────────────────────────────┘
```

### Partial Retrieval Example

If you know that the first 50 bytes of a file contain a header with metadata (file type, creation date, etc.), you can issue a single byte-range request for just those 50 bytes:

```
GET /my-file.dat HTTP/1.1
Host: my-bucket.s3.amazonaws.com
Range: bytes=0-49
```

This returns ==only 50 bytes== instead of the entire file — extremely fast and cost-effective when you just need to inspect file metadata.

## KMS Encryption Performance Consideration

If your S3 objects are encrypted with ==SSE-KMS==, there's an additional performance consideration: every upload and download requires a call to the KMS API (GenerateDataKey for uploads, Decrypt for downloads).

| KMS Quota | Value |
|-----------|-------|
| **Default** | ==5,500 to 30,000 requests/second== (varies by region) |
| **Adjustable** | Yes — you can request a quota increase |

If you're doing high-throughput S3 operations with SSE-KMS encryption, the ==KMS quota can become a bottleneck==. Each S3 GET/PUT triggers a KMS API call, so at high request rates, you may hit the KMS limit before the S3 limit.

> [!warning] Exam Tip — KMS Throttling
> If the exam describes a scenario where S3 uploads/downloads are being ==throttled== and the objects use ==SSE-KMS encryption==, the answer is likely that you're hitting the ==KMS request quota==. Solutions:
> - Request a ==KMS quota increase== via Service Quotas
> - Use ==SSE-S3== encryption instead (no KMS API calls)
> - Use ==S3 Bucket Keys== to reduce KMS API calls (reduces cost and throttling)

## Performance Optimization Summary

```
┌─────────────────────────────────────────────────────────────────┐
│              S3 PERFORMANCE OPTIMIZATION CHEAT SHEET            │
│                                                                 │
│  UPLOADS:                                                      │
│  ┌─────────────────────────┐  ┌───────────────────────────┐   │
│  │  Multi-Part Upload      │  │  Transfer Acceleration    │   │
│  │  • Split into parts     │  │  • Edge location → AWS    │   │
│  │  • Parallel upload      │  │    private network        │   │
│  │  • Recommended > 100 MB │  │  • Cross-region uploads   │   │
│  │  • Required > 5 GB      │  │  • 200+ edge locations    │   │
│  │  • Up to 10,000 parts   │  │  • Compatible w/ multi-   │   │
│  │  • Retry individual     │  │    part upload            │   │
│  │    parts on failure     │  │  • Extra cost per GB      │   │
│  └─────────────────────────┘  └───────────────────────────┘   │
│                                                                 │
│  DOWNLOADS:                                                    │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Byte-Range Fetches                                     │   │
│  │  • Parallel GET requests for specific byte ranges       │   │
│  │  • Speed up downloads OR retrieve partial files         │   │
│  │  • Better resilience (retry only failed ranges)         │   │
│  │  • Great for reading file headers without full download │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  THROUGHPUT:                                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Spread across prefixes                                 │   │
│  │  • 3,500 PUT + 5,500 GET per prefix per second         │   │
│  │  • No limit on number of prefixes                       │   │
│  │  • 4 prefixes = 22,000 GET/sec, 14,000 PUT/sec        │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ENCRYPTION:                                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  SSE-KMS can be a bottleneck                            │   │
│  │  • Each S3 request = 1 KMS API call                     │   │
│  │  • KMS quota: 5,500-30,000 req/sec (region-dependent)  │   │
│  │  • Use S3 Bucket Keys to reduce KMS calls              │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

## Questions & Answers

> [!question]- Q1: What is the baseline S3 performance?
> **Answer:**
> - ==First byte latency==: 100-200 milliseconds
> - ==PUT/COPY/POST/DELETE==: 3,500 requests/second ==per prefix==
> - ==GET/HEAD==: 5,500 requests/second ==per prefix==
> - No limit on the number of prefixes in a bucket
> S3 automatically scales to these numbers — no provisioning needed. The limits are per prefix, so total bucket throughput scales linearly with the number of prefixes.

> [!question]- Q2: What is an S3 prefix and why does it matter for performance?
> **Answer:**
> The prefix is the "folder path" between the bucket name and the object name. For `bucket/folder1/sub1/file`, the prefix is `/folder1/sub1/`. Each unique prefix gets its own ==independent throughput allocation== of 3,500 PUT and 5,500 GET per second. Since there's no limit on prefixes, spreading objects across multiple prefixes multiplies your throughput. For example, 4 prefixes = 22,000 GET/second.

> [!question]- Q3: When is multi-part upload recommended vs required?
> **Answer:**
> - ==Recommended== for files over ==100 MB== — parallelization significantly speeds up the upload
> - ==Required== for files over ==5 GB== — S3 won't accept a single PUT request for files larger than 5 GB
> Multi-part upload splits the file into parts (5 MB to 5 GB each, up to 10,000 parts), uploads them in parallel, and S3 reassembles them. If one part fails, only that part needs to be retried.

> [!question]- Q4: What is S3 Transfer Acceleration and when should you use it?
> **Answer:**
> Transfer Acceleration speeds up cross-region uploads by routing through the nearest ==AWS edge location== (200+ worldwide). The file travels over the ==public internet== to the edge location (short distance), then over the ==fast AWS private network== to the S3 bucket (long distance). Use it when users are uploading to a bucket in a ==distant region==. It's compatible with multi-part upload for maximum speed. AWS charges per GB but won't charge if acceleration doesn't improve speed.

> [!question]- Q5: What are the two use cases for S3 Byte-Range Fetches?
> **Answer:**
> 1. ==Speed up downloads== — request multiple byte ranges simultaneously, download in parallel, client reassembles the file. Similar to multi-part upload but for downloads.
> 2. ==Partial retrieval== — get only specific bytes of a file without downloading the entire thing. Example: read the first 50 bytes (file header) to determine file type or metadata.
> Both use cases benefit from ==resilience== — if a range fails, only that range needs to be retried.

> [!question]- Q6: How can you achieve 22,000 GET requests per second on S3?
> **Answer:**
> Spread your reads across ==4 prefixes==. Each prefix supports 5,500 GET/second, so 4 × 5,500 = ==22,000 GET/second==. Since there's no limit on the number of prefixes, you can scale throughput virtually without limit by distributing objects across more prefixes. With 10 prefixes, you'd get 55,000 GET/second.

> [!question]- Q7: Can you combine Transfer Acceleration with multi-part upload?
> **Answer:**
> ==Yes.== They're compatible and complementary:
> - ==Multi-part upload== parallelizes the upload into parts (maximizes bandwidth utilization)
> - ==Transfer Acceleration== routes each part through the nearest edge location over the AWS private network (minimizes latency)
> Combining both gives you ==maximum upload speed== for large files to distant regions. This is the recommended approach for uploading large files cross-region.

> [!question]- Q8: What happens if S3 uploads are throttled with SSE-KMS encryption?
> **Answer:**
> Each S3 upload/download with SSE-KMS triggers a ==KMS API call== (GenerateDataKey for uploads, Decrypt for downloads). The KMS quota is ==5,500 to 30,000 requests/second== depending on the region. At high throughput, you can hit the KMS limit before the S3 limit. Solutions:
> - Request a ==KMS quota increase==
> - Use ==S3 Bucket Keys== to reduce KMS API calls
> - Switch to ==SSE-S3== encryption (no KMS calls)

> [!question]- Q9: What is the difference between Transfer Acceleration and CloudFront for S3?
> **Answer:**
> - ==Transfer Acceleration==: Optimizes ==uploads to S3== using edge locations → AWS private network. No caching. S3-specific.
> - ==CloudFront==: Optimizes ==downloads/content delivery== by ==caching content== at edge locations. General CDN for any origin.
> For uploading large files to S3, use Transfer Acceleration. For serving static content to many users, use CloudFront.

> [!question]- Q10: Exam scenario — users worldwide uploading large files to a single S3 bucket. How to optimize?
> **Answer:**
> Combine all three techniques:
> 1. ==Multi-part upload== — split large files into parts, upload in parallel (maximizes bandwidth)
> 2. ==S3 Transfer Acceleration== — route through nearest edge location, use AWS private network (minimizes latency)
> 3. ==Multiple prefixes== — spread objects across prefixes for higher per-prefix throughput
> For downloads, add ==byte-range fetches== to parallelize GET requests. If using SSE-KMS, enable ==S3 Bucket Keys== to avoid KMS throttling.
