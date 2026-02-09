---
title: S3 Static Website Hosting
date: 2026-02-10
tags:
  - aws
  - s3
  - static-website
  - saa-c03
---

# S3 Static Website Hosting

## Overview

S3 can host ==static websites== and make them accessible on the internet. This is one of the most popular S3 features — you upload your HTML, CSS, JavaScript, and image files to a bucket, enable static website hosting, and S3 serves them as a website. No web server needed.

The key requirement is that the bucket must have ==public reads enabled== via a bucket policy. Without it, visitors will get a 403 Forbidden error.

## Website URL Format

When you enable static website hosting, S3 gives you a website endpoint URL. The format depends on the region:

```
http://<bucket-name>.s3-website-<region>.amazonaws.com
                     OR
http://<bucket-name>.s3-website.<region>.amazonaws.com
```

The only difference is a ==dash== (`-`) vs a ==dot== (`.`) before the region name. You don't need to memorize which regions use which format — just know that both exist.

> [!note] HTTP Only
> The S3 website endpoint only supports ==HTTP==, not HTTPS. If you need HTTPS, you must put ==CloudFront== (AWS CDN) in front of the S3 bucket.

## How It Works

```
┌──────────┐    HTTP GET     ┌──────────────────────────────────────┐
│  User's   │──────────────▶│  S3 Bucket (Website Enabled)          │
│  Browser  │               │                                       │
│           │◀──────────────│  ├── index.html  ← homepage           │
│           │   HTML page   │  ├── error.html  ← custom error page  │
│           │               │  ├── coffee.jpg  ← image asset        │
│           │               │  ├── beach.jpg   ← image asset        │
│           │               │  └── style.css   ← stylesheet         │
│           │               │                                       │
│           │               │  Bucket Policy: Allow * GetObject     │
│           │               │  Block Public Access: DISABLED        │
└──────────┘               └──────────────────────────────────────┘
```

When a user visits the website URL:
1. S3 looks for the ==index document== (e.g., `index.html`) and serves it
2. The HTML can reference other objects in the bucket (images, CSS, JS)
3. All referenced objects must also be publicly readable

## Requirements Checklist

| Step | What to Do | Why |
|------|-----------|-----|
| 1 | Upload HTML, images, CSS, JS files | These are your website content |
| 2 | Disable ==Block Public Access== | Otherwise the bucket can't be public |
| 3 | Attach a ==public bucket policy== | Allow `s3:GetObject` for `*` on `/*` |
| 4 | Enable ==Static Website Hosting== | Properties → Static Website Hosting → Enable |
| 5 | Set ==Index document== | The homepage file (e.g., `index.html`) |
| 6 | Optionally set ==Error document== | Custom error page (e.g., `error.html`) |

> [!warning] 403 Forbidden Error
> If you enable static website hosting but get a ==403 Forbidden== error when visiting the URL, it means your bucket is ==not public==. You need to:
> 1. Disable Block Public Access
> 2. Attach a bucket policy allowing public reads
>
> This is the #1 troubleshooting issue with S3 websites.

## Hands-On Walkthrough

### Setting Up the Website

1. Make sure your bucket already has a ==public bucket policy== (from the previous lecture)
2. Upload an additional file: `beach.jpg` (so you have `coffee.jpg` and `beach.jpg`)
3. Go to **Properties** tab → scroll all the way down to **Static website hosting**
4. Click **Edit** → ==Enable== static website hosting
5. Choose **Host a static website**
6. **Index document**: enter `index.html`
7. Notice the warning: "you must make all your content publicly readable" — we've already done this
8. Click **Save changes**

### Uploading the Index File

1. Go back to **Objects** tab
2. Click **Upload** → **Add files** → select `index.html`
3. Upload it

### Testing the Website

1. Go to **Properties** → scroll to **Static website hosting**
2. You'll see a ==Bucket website endpoint== URL
3. Copy and paste it into your browser
4. You should see your HTML page (e.g., "I love coffee" with the coffee image)
5. You can also directly access other objects: append `/beach.jpg` to the URL

### What the HTML Looks Like

The `index.html` file references objects in the same bucket:

```html
<html>
  <body>
    <h1>I love coffee</h1>
    <p>Hello world!</p>
    <img src="coffee.jpg" />
  </body>
</html>
```

Because `coffee.jpg` is in the same bucket and the bucket is public, the image loads correctly.

## S3 Object URL vs Website Endpoint URL

| Feature | Object URL | Website Endpoint URL |
|---------|-----------|---------------------|
| **Format** | `https://bucket.s3.region.amazonaws.com/key` | `http://bucket.s3-website-region.amazonaws.com` |
| **Protocol** | HTTPS | ==HTTP only== |
| **Index document** | No — must specify exact key | Yes — serves `index.html` at root |
| **Error pages** | Returns XML error | Serves custom ==error.html== |
| **Redirects** | No | Supports redirect rules |
| **Use case** | Direct object access | ==Website hosting== |

## Questions & Answers

> [!question]- Q1: What type of websites can S3 host?
> **Answer:**
> ==Static websites== only — HTML, CSS, JavaScript, images, videos, fonts. S3 cannot run server-side code like PHP, Node.js, Python, or Java. For dynamic websites, you need EC2, Lambda, or other compute services.

> [!question]- Q2: What are all the steps required to make an S3 static website work?
> **Answer:**
> 1. Upload your website files (HTML, CSS, JS, images) to the bucket
> 2. Disable ==Block Public Access== on the bucket
> 3. Attach a bucket policy allowing `s3:GetObject` for principal `*` on resource `arn:aws:s3:::bucket-name/*`
> 4. Enable ==Static Website Hosting== in bucket Properties
> 5. Specify an ==index document== (e.g., `index.html`)
> 6. Access the bucket website endpoint URL

> [!question]- Q3: What does a 403 Forbidden error mean when accessing an S3 website?
> **Answer:**
> It means the bucket is ==not public==. The most common causes:
> - Block Public Access is still enabled
> - No bucket policy allowing public reads
> - The bucket policy ARN is missing the `/*` suffix
>
> Fix: disable Block Public Access AND attach a proper public bucket policy.

> [!question]- Q4: What is the index document and what happens if it's missing?
> **Answer:**
> The index document is the ==default homepage== served when users visit the root URL of the website. Typically `index.html`. If the index document doesn't exist in the bucket, users will get a ==404 Not Found== error when visiting the root URL.

> [!question]- Q5: Can you use a custom domain with S3 static website hosting?
> **Answer:**
> Yes, using ==Route 53== with an Alias record pointing to the S3 website endpoint. Important: the ==bucket name must match the domain name== exactly. For example, if your domain is `www.example.com`, the bucket must be named `www.example.com`.

> [!question]- Q6: What is the difference between the S3 object URL and the website endpoint URL?
> **Answer:**
> - **Object URL** (`https://bucket.s3.region.amazonaws.com/key`): direct access to a specific object, supports HTTPS, returns XML errors
> - **Website URL** (`http://bucket.s3-website-region.amazonaws.com`): serves index.html at root, supports custom error pages, ==HTTP only==

> [!question]- Q7: Does S3 static website hosting support HTTPS?
> **Answer:**
> No. The S3 website endpoint only supports ==HTTP==. To serve your website over HTTPS, you need to put ==Amazon CloudFront== (a CDN) in front of the S3 bucket. CloudFront can serve content over HTTPS and also provides caching and edge locations for better performance.

> [!question]- Q8: Can you configure a custom error page?
> **Answer:**
> Yes. When enabling static website hosting, you can specify an ==error document== (e.g., `error.html`). S3 serves this page for 4xx errors (like 404 Not Found) instead of the default XML error response.

> [!question]- Q9: What happens if you reference an image in your HTML that doesn't exist in the bucket?
> **Answer:**
> The browser will show a ==broken image icon==. If you try to access the image URL directly, you'll get a ==404 Not Found== error. The rest of the page will still load — only the missing resource will fail.

> [!question]- Q10: Is S3 static website hosting free?
> **Answer:**
> There's no additional charge for enabling static website hosting. You pay standard S3 pricing:
> - ==Storage==: per GB stored
> - ==Requests==: per GET request (each page view, each image load)
> - ==Data transfer==: per GB transferred out to the internet
>
> For low-traffic websites, the cost is very minimal.
