---
title: "S3 CORS"
date: 2026-02-10
tags:
  - aws
  - s3
  - cors
  - cross-origin
  - security
  - static-website
  - preflight
  - access-control
  - saa-c03
---

# S3 CORS

## Overview

CORS stands for ==Cross-Origin Resource Sharing==. It's a ==web browser-based security mechanism== that allows or denies requests to other origins while visiting a main origin. This is something you need to know at the exam — it typically comes up as ==one question==, and understanding the concept deeply makes answering it trivial.

The core idea: when your web browser loads a page from Website A, and that page tries to fetch a resource (image, script, API call) from Website B, the browser ==blocks the request by default== unless Website B explicitly says "I allow requests from Website A." That permission is granted through ==CORS headers==.

---

## What is an Origin?

An origin is defined by ==three components==: the **scheme** (protocol), the **host** (domain), and the **port**.

For example, looking at `https://www.example.com`:
- ==Scheme==: `https`
- ==Host==: `www.example.com`
- ==Port==: `443` (implied for HTTPS, `80` for HTTP)

| Comparison | Same Origin? | Why |
|-----------|:----------:|-----|
| `https://www.example.com/page1` vs `https://www.example.com/page2` | ✅ Yes | Same scheme, host, port — only the path differs |
| `https://www.example.com` vs `https://other.example.com` | ❌ No | Different ==host== (subdomain counts as different) |
| `http://www.example.com` vs `https://www.example.com` | ❌ No | Different ==scheme== (HTTP vs HTTPS) |
| `https://www.example.com` vs `https://www.example.com:8443` | ❌ No | Different ==port== (443 vs 8443) |

> [!note] Path Doesn't Matter
> Two URLs with the same scheme, host, and port are the ==same origin== regardless of their path. `/page1` and `/page2` on the same domain are the same origin.

---

## How CORS Works — The Full Flow

When a web browser loads a page from one origin and that page needs to fetch resources from a ==different origin==, the browser enforces a security check. Here's the complete flow:

### Step 1: Browser Loads the Main Page

The browser requests `index.html` from your origin server (e.g., `https://www.example.com`). The server returns the HTML, which contains a reference to a resource on a ==different origin== (e.g., an image on `https://www.other.com`).

### Step 2: Browser Sends a Preflight Request

Before making the actual cross-origin request, the browser sends an ==OPTIONS request== (called a **preflight request**) to the cross-origin server. This preflight says:

> "Hey, I'm coming from `https://www.example.com`. Am I allowed to make requests to you?"

The preflight includes the `Origin` header with the requesting origin's URL.

### Step 3: Cross-Origin Server Responds with CORS Headers

If the cross-origin server is configured to allow requests from that origin, it responds with CORS headers:

- `Access-Control-Allow-Origin: https://www.example.com` — "Yes, I allow this origin"
- `Access-Control-Allow-Methods: GET, PUT, DELETE` — "These HTTP methods are allowed"

### Step 4: Browser Makes the Actual Request

If the browser is satisfied with the CORS headers (the origin matches, the method is allowed), it proceeds with the actual request and retrieves the resource.

```
┌──────────────────────────────────────────────────────────────────┐
│                    CORS FLOW — STEP BY STEP                       │
│                                                                   │
│  ┌──────────────┐                    ┌──────────────────────┐    │
│  │              │  1. GET index.html  │  Origin Server       │    │
│  │  Web Browser │ ─────────────────▶  │  www.example.com     │    │
│  │              │ ◀─────────────────  │                      │    │
│  │              │  Returns HTML that  └──────────────────────┘    │
│  │              │  says: "fetch image                             │
│  │              │  from www.other.com"                            │
│  │              │                                                 │
│  │              │  2. Preflight        ┌──────────────────────┐   │
│  │              │     OPTIONS request  │  Cross-Origin Server │   │
│  │              │ ─────────────────▶   │  www.other.com       │   │
│  │              │  Origin:             │                      │   │
│  │              │  www.example.com     │                      │   │
│  │              │                      │                      │   │
│  │              │ ◀─────────────────   │                      │   │
│  │              │  Access-Control-     │                      │   │
│  │              │  Allow-Origin:       │                      │   │
│  │              │  www.example.com     │                      │   │
│  │              │  Allow-Methods:      │                      │   │
│  │              │  GET, PUT, DELETE    │                      │   │
│  │              │                      │                      │   │
│  │              │  3. Actual GET       │                      │   │
│  │              │ ─────────────────▶   │                      │   │
│  │              │ ◀─────────────────   │                      │   │
│  │              │  (resource returned) └──────────────────────┘   │
│  └──────────────┘                                                 │
│                                                                   │
│  ⚠️ If CORS headers are missing or don't match → BLOCKED        │
│  ✅ If CORS headers match the origin → REQUEST ALLOWED           │
└──────────────────────────────────────────────────────────────────┘
```

> [!important] CORS is Browser-Side Security
> CORS is enforced ==entirely by the web browser==, not by the server. The server simply responds with CORS headers — the ==browser reads those headers and decides== whether to allow or block the request. Server-to-server requests (e.g., Lambda calling S3, EC2 calling an API) are ==not subject to CORS== because there's no browser involved.

---

## CORS with Amazon S3

This is where it gets relevant for the exam. If you host a ==static website on S3== and that website needs to load assets (images, scripts, HTML pages) from ==another S3 bucket==, you must ==enable CORS headers on the target bucket==.

### The Scenario

```
┌──────────────────────────────────────────────────────────────────┐
│                    S3 CORS SCENARIO                                │
│                                                                   │
│  ┌──────────────┐                    ┌──────────────────────┐    │
│  │              │  1. GET index.html  │  S3 Bucket           │    │
│  │  Web Browser │ ─────────────────▶  │  my-bucket-html      │    │
│  │              │ ◀─────────────────  │  (static website)    │    │
│  │              │  index.html says:   │  Origin: http://     │    │
│  │              │  "load image from   │  my-bucket-html.s3-  │    │
│  │              │   other bucket"     │  website.amazonaws   │    │
│  │              │                     │  .com                │    │
│  │              │                     └──────────────────────┘    │
│  │              │                                                 │
│  │              │  2. GET image       ┌──────────────────────┐   │
│  │              │ ─────────────────▶  │  S3 Bucket           │   │
│  │              │  Origin: bucket-    │  my-bucket-assets    │   │
│  │              │  html URL           │  (static website)    │   │
│  │              │                     │                      │   │
│  │              │  ❌ WITHOUT CORS:   │  ⚠️ Must configure   │   │
│  │              │  "Cross-Origin      │  CORS to allow       │   │
│  │              │   Request Blocked"  │  requests from       │   │
│  │              │                     │  bucket-html origin  │   │
│  │              │  ✅ WITH CORS:      │                      │   │
│  │              │  Image loads!       └──────────────────────┘   │
│  └──────────────┘                                                 │
└──────────────────────────────────────────────────────────────────┘
```

> [!tip] Exam Pattern
> If the exam mentions a static website on S3 that needs to load assets from ==another S3 bucket== and it's failing with a CORS error, the answer is almost always: ==enable CORS on the target bucket== (the one serving the assets) with the correct `AllowedOrigins` pointing to the first bucket's static website URL.

### Configuring CORS on S3

CORS is configured as a ==JSON document== in the bucket's Permissions settings. You can allow a ==specific origin== or use `*` to allow ==all origins==:

**Allow a specific origin (recommended for security):**

```json
[
  {
    "AllowedHeaders": ["*"],
    "AllowedMethods": ["GET"],
    "AllowedOrigins": [
      "http://my-bucket-html.s3-website-eu-west-1.amazonaws.com"
    ],
    "ExposeHeaders": []
  }
]
```

**Allow all origins (less secure, but quick):**

```json
[
  {
    "AllowedHeaders": ["*"],
    "AllowedMethods": ["GET"],
    "AllowedOrigins": ["*"],
    "ExposeHeaders": []
  }
]
```

### CORS Configuration Fields

| Field | Description | Example |
|-------|-------------|---------|
| ==AllowedOrigins== | Which origins can make requests | `["http://example.com"]` or `["*"]` |
| ==AllowedMethods== | Which HTTP methods are allowed | `["GET", "PUT", "POST", "DELETE"]` |
| ==AllowedHeaders== | Which request headers are allowed | `["*"]` (all headers) |
| ==ExposeHeaders== | Which response headers the browser can access | `["ETag", "x-amz-request-id"]` |
| ==MaxAgeSeconds== | How long the browser caches the preflight response | `3000` (50 minutes) |

---

## Hands-On Walkthrough

### Setup: Two S3 Buckets as Static Websites

**Bucket 1 — Origin bucket** (`my-bucket-html`):
1. Create bucket, ==unblock all public access==
2. Enable ==Static Website Hosting== (Properties → Static Website Hosting → Enable)
3. Set `index.html` as the index document
4. Add a ==bucket policy== to make it public:
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [{
       "Effect": "Allow",
       "Principal": "*",
       "Action": "s3:GetObject",
       "Resource": "arn:aws:s3:::my-bucket-html/*"
     }]
   }
   ```
5. Upload `index.html` that references a page on the second bucket

**Bucket 2 — Cross-origin bucket** (`my-bucket-assets`):
1. Create bucket in a ==different region== (to clearly show they're different servers)
2. ==Unblock all public access==, enable ==Static Website Hosting==
3. Add the same public bucket policy (with the correct bucket ARN)
4. Upload `extra-page.html`

### Testing Without CORS

1. Open the first bucket's static website URL in a browser
2. The `index.html` loads, but the content from the second bucket ==fails to load==
3. Open ==Chrome DevTools== (F12 or right-click → Inspect → Console tab)
4. You'll see a red error: ==`Cross-Origin Request Blocked: The Same Origin Policy disallows reading the remote resource. (Reason: CORS header 'Access-Control-Allow-Origin' missing)`==

> [!bug] The Error You'll See
> In the browser's developer console:
> ```
> Cross-Origin Request Blocked: The Same Origin Policy disallows
> reading the remote resource at http://my-bucket-assets...
> (Reason: CORS header 'Access-Control-Allow-Origin' missing)
> ```

### Fixing It: Adding CORS Configuration

1. Go to ==Bucket 2== (the cross-origin bucket) → ==Permissions== → scroll to ==CORS==
2. Click ==Edit== and paste the CORS JSON:
   ```json
   [
     {
       "AllowedHeaders": ["Authorization"],
       "AllowedMethods": ["GET"],
       "AllowedOrigins": [
         "http://my-bucket-html.s3-website-eu-west-1.amazonaws.com"
       ],
       "ExposeHeaders": [],
       "MaxAgeSeconds": 3000
     }
   ]
   ```
3. ==Save changes==

> [!warning] No Trailing Slash!
> The `AllowedOrigins` URL must ==not have a trailing slash==:
> - ✅ `http://my-bucket.s3-website.amazonaws.com`
> - ❌ `http://my-bucket.s3-website.amazonaws.com/`

### Verifying It Works

1. Refresh the first bucket's website in the browser
2. The cross-origin content now ==loads successfully==
3. In Chrome DevTools → ==Network tab== → click on the cross-origin request → ==Response Headers==:
   - `Access-Control-Allow-Methods: GET`
   - `Access-Control-Allow-Origin: http://my-bucket-html.s3-website-eu-west-1.amazonaws.com`

These headers confirm that CORS is working — the cross-origin server is telling the browser "yes, I allow requests from this origin."

---

## Questions & Answers

> [!question]- Q1: What does CORS stand for and what is it?
> **Answer:**
> ==Cross-Origin Resource Sharing==. It's a web browser-based security mechanism that controls whether a web page loaded from one origin can request resources from a different origin. The browser blocks cross-origin requests by default unless the target server explicitly allows them via CORS headers.

> [!question]- Q2: What defines an "origin" in the context of CORS?
> **Answer:**
> An origin is the combination of ==scheme (protocol) + host (domain) + port==. For example, `https://www.example.com:443`. Two URLs share the same origin only if ==all three components match exactly==. Even different subdomains (`www.example.com` vs `api.example.com`) are different origins.

> [!question]- Q3: What is a preflight request?
> **Answer:**
> Before making the actual cross-origin request, the browser sends an ==OPTIONS request== (the preflight) to the target server. It includes the `Origin` header saying "I'm coming from this origin." The server responds with CORS headers (`Access-Control-Allow-Origin`, `Access-Control-Allow-Methods`). If the browser is satisfied with the response, it proceeds with the actual request.

> [!question]- Q4: How do you enable CORS on an S3 bucket?
> **Answer:**
> Go to the bucket's ==Permissions → CORS configuration== and add a JSON document specifying `AllowedOrigins`, `AllowedMethods`, `AllowedHeaders`, and optionally `ExposeHeaders` and `MaxAgeSeconds`. You can allow specific origins or use `*` for all origins.

> [!question]- Q5: A static website on S3 Bucket A tries to load an image from S3 Bucket B and fails. What's the likely issue?
> **Answer:**
> Bucket B does not have ==CORS headers configured== to allow requests from Bucket A's origin. The fix is to add a CORS configuration on ==Bucket B== (the target) with Bucket A's static website URL in `AllowedOrigins`. Remember: CORS is configured on the ==target== bucket, not the source.

> [!question]- Q6: Is CORS enforced by the server or the browser?
> **Answer:**
> CORS is enforced ==entirely by the web browser==. The server only provides the CORS headers in its response — the browser reads those headers and decides whether to allow or block the cross-origin request. This is why server-to-server requests (Lambda, EC2, etc.) are not affected by CORS.

> [!question]- Q7: What happens if you set AllowedOrigins to "*"?
> **Answer:**
> It allows ==all origins== to make cross-origin requests to the bucket. This is less secure but useful for public assets (CDN content, public images) that should be accessible from any website. For sensitive data, always specify the exact allowed origins.

> [!question]- Q8: Can you have multiple CORS rules on a single S3 bucket?
> **Answer:**
> Yes. The CORS configuration is a ==JSON array== of rule objects. Each rule can specify different allowed origins, methods, and headers. S3 evaluates them in order and uses the ==first matching rule== for each request.

> [!question]- Q9: Does CORS apply to server-to-server requests?
> **Answer:**
> No. CORS is a ==browser-only== security mechanism. Server-to-server requests (e.g., Lambda calling S3, EC2 calling an API, backend services communicating) are ==not subject to CORS== restrictions because there's no browser enforcing the policy.

> [!question]- Q10: What is the Access-Control-Allow-Origin header?
> **Answer:**
> It's the most important CORS response header. It tells the browser ==which origins are allowed== to access the resource. If the requesting origin matches the value in this header (or the header is `*`), the browser allows the cross-origin request. If the header is missing or doesn't match, the browser ==blocks the request==.
