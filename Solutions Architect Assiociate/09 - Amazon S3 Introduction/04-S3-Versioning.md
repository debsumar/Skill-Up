---
title: S3 Versioning
date: 2026-02-10
tags:
  - aws
  - s3
  - versioning
  - saa-c03
---

# S3 Versioning

## Overview

S3 Versioning keeps ==multiple variants of an object== in the same bucket. Every time you upload a file with the same key (same name/path), instead of overwriting the previous file, S3 creates a new version. This is enabled at the ==bucket level== and is considered a ==best practice==.

Think of it like Git for your files — you can always go back to a previous version.

## How Versioning Works

```
Upload: index.html → Version 1 (ID: null or auto-generated)
Upload: index.html → Version 2 (new version ID)
Upload: index.html → Version 3 (new version ID)

Bucket view (Show versions ON):
┌──────────────────────────────────────────┐
│ index.html  │ Version 3  │ abc123  │ ← current
│ index.html  │ Version 2  │ def456  │
│ index.html  │ Version 1  │ null    │ ← uploaded before versioning
│ coffee.jpg  │            │ null    │ ← uploaded before versioning
│ beach.jpg   │            │ null    │ ← uploaded before versioning
└──────────────────────────────────────────┘
```

When versioning is enabled:
- Each upload of the same key creates a ==new version== with a unique version ID
- The latest version is served by default when someone requests the object
- All previous versions are retained and accessible

## Why Use Versioning?

| Benefit | How It Helps |
|---------|-------------|
| **Protect against unintended deletes** | Deleting an object just adds a ==delete marker== — the actual data is still there and can be restored |
| **Easy rollback** | Made a mistake? Roll back to any previous version instantly |
| **Audit trail** | See the complete history of changes to any object |
| **Required for Replication** | S3 Replication (CRR/SRR) requires versioning on both source and target buckets |

## Important Notes

| Scenario | What Happens |
|----------|-------------|
| Files uploaded ==before== enabling versioning | They get version ID = ==`null`== |
| Suspending versioning | ==Does NOT delete== previous versions — it's a safe operation |
| New uploads after suspending | Get version ID = `null` (no new versions created) |
| Re-enabling versioning | Previous versions are still there, new uploads get version IDs again |

> [!tip] Safe to Experiment
> Suspending versioning is completely safe. It doesn't delete any existing versions. You can re-enable it at any time and all your old versions will still be there.

## Delete Behavior — The Most Important Part

This is where versioning gets tricky and where exam questions focus. There are ==two completely different types of delete==:

### Type 1: Soft Delete (Delete Marker)

This happens when you delete an object ==without specifying a version ID== (i.e., with "Show versions" toggle OFF in the console).

```
Before:
┌─────────────────────────────────┐
│ coffee.jpg  │ v1  │ null        │ ← visible
└─────────────────────────────────┘

After "Delete" (soft):
┌─────────────────────────────────┐
│ coffee.jpg  │ Delete Marker │ xyz789 │ ← hides the object
│ coffee.jpg  │ v1            │ null   │ ← still exists!
└─────────────────────────────────┘

Result: coffee.jpg appears deleted (404 Not Found)
        But v1 is still stored and can be restored!
```

- The object ==appears deleted== to users (404 Not Found)
- But the actual data is ==still stored== behind the delete marker
- The console shows "delete" (not "permanently delete") as the confirmation text
- ==Reversible== — delete the delete marker to restore the object

### Type 2: Permanent Delete (Specific Version)

This happens when you delete a ==specific version ID== (i.e., with "Show versions" toggle ON in the console).

```
Before:
┌─────────────────────────────────┐
│ index.html  │ v3  │ abc123      │ ← current
│ index.html  │ v2  │ def456      │
│ index.html  │ v1  │ null        │
└─────────────────────────────────┘

After permanently deleting v3 (abc123):
┌─────────────────────────────────┐
│ index.html  │ v2  │ def456      │ ← now current
│ index.html  │ v1  │ null        │
└─────────────────────────────────┘

Result: v3 is GONE FOREVER. v2 becomes the current version.
```

- The specific version is ==permanently destroyed== — cannot be recovered
- The console shows "permanently delete" as the confirmation text
- ==Irreversible== — this is a destructive operation

> [!danger] Know the Difference
> - Delete ==without== version ID → ==delete marker== (reversible, soft delete)
> - Delete ==with== version ID → ==permanent delete== (irreversible, destructive)
>
> This distinction is heavily tested on the exam.

### Restoring a Soft-Deleted Object

To restore an object that was soft-deleted (has a delete marker):

```
Step 1: Enable "Show versions" toggle
Step 2: Find the Delete Marker for the object
Step 3: Select the Delete Marker → Delete it
Step 4: Confirm "permanently delete" (you're deleting the marker, not the object)
Step 5: The previous version is now restored and visible again
```

## Hands-On Walkthrough

### Enabling Versioning

1. Go to bucket **Properties** → **Bucket Versioning** → **Edit** → ==Enable==
2. Save changes
3. From now on, any file you upload will get a version ID

### Uploading a New Version

1. Edit your `index.html` locally (e.g., change "I love coffee" to "I REALLY love coffee")
2. Upload the modified `index.html` to the bucket
3. Visit your website URL → see the updated content
4. In the console, click **Show versions** toggle
5. You'll see ==two versions== of `index.html`:
   - One with version ID `null` (uploaded before versioning)
   - One with a real version ID (uploaded after versioning)

### Rolling Back

1. With "Show versions" ON, select the ==latest version== of `index.html`
2. Delete it (this is a permanent delete of that specific version)
3. The previous version becomes current
4. Refresh your website → see the old content ("I love coffee")

### Soft Delete and Restore

1. Turn "Show versions" ==OFF==
2. Select `coffee.jpg` → Delete → type "delete" → confirm
3. The file appears gone from the bucket
4. Turn "Show versions" ==ON== → you'll see a ==Delete Marker== on `coffee.jpg`
5. The original `coffee.jpg` (version `null`) is still there behind the marker
6. Select the Delete Marker → Delete it → type "permanently delete"
7. The `coffee.jpg` is ==restored== and visible again

## Questions & Answers

> [!question]- Q1: At what level is versioning enabled and what does it affect?
> **Answer:**
> Versioning is enabled at the ==bucket level==. Once enabled, ==all objects== in the bucket are versioned. You can't enable versioning for specific objects or prefixes — it's all or nothing for the entire bucket.

> [!question]- Q2: What happens to files that were uploaded before enabling versioning?
> **Answer:**
> They get a version ID of ==`null`==. They are not retroactively versioned — they just exist as a single version with a null ID. Once versioning is enabled, any new uploads (including re-uploads of the same key) will get real version IDs.

> [!question]- Q3: What is a delete marker and how does it work?
> **Answer:**
> When you delete an object in a versioned bucket ==without specifying a version ID==, S3 doesn't actually delete the data. Instead, it adds a ==delete marker== — a special zero-byte object that hides the previous versions. The object appears deleted (404 Not Found), but all previous versions still exist and can be restored by deleting the delete marker.

> [!question]- Q4: How do you restore a soft-deleted object?
> **Answer:**
> 1. Enable the "Show versions" toggle in the S3 console
> 2. Find the ==Delete Marker== for the object
> 3. Select the Delete Marker and delete it (permanently delete the marker)
> 4. The previous version is now restored and the object is visible again

> [!question]- Q5: What is the critical difference between soft delete and permanent delete?
> **Answer:**
> - ==Soft delete==: Delete without specifying a version ID → adds a delete marker → ==reversible== (delete the marker to restore)
> - ==Permanent delete==: Delete a specific version ID → that version is ==gone forever== → ==irreversible==
>
> The console helps distinguish: soft delete asks you to type "delete", permanent delete asks you to type "permanently delete".

> [!question]- Q6: Does suspending versioning delete previous versions?
> **Answer:**
> No. Suspending versioning is a ==safe operation== that does not delete any existing versions. It only stops creating new versions for future uploads. All existing versions remain accessible. You can re-enable versioning at any time.

> [!question]- Q7: How do you roll back to a previous version of a file?
> **Answer:**
> With "Show versions" enabled, ==permanently delete the current (latest) version==. The previous version automatically becomes the current version. Alternatively, you can download the old version and re-upload it.

> [!question]- Q8: Why is versioning considered a best practice?
> **Answer:**
> - Protects against ==accidental deletes== (delete markers are reversible)
> - Enables ==easy rollback== to any previous version
> - Provides an ==audit trail== of all changes
> - ==Required== for S3 Replication (CRR/SRR)
> - Suspending is safe — no data loss

> [!question]- Q9: Can you see version IDs in the S3 console?
> **Answer:**
> Yes, by enabling the ==Show versions== toggle in the Objects tab. This reveals all version IDs, delete markers, and the full version history of every object. Without this toggle, you only see the current (latest) version of each object.

> [!question]- Q10: What happens if you upload the same file name twice with versioning enabled?
> **Answer:**
> S3 creates a ==new version== with a unique version ID. Both versions are stored in the bucket. The latest version is served by default when someone requests the object. The previous version is still accessible by specifying its version ID. This is how versioning enables safe updates — you never lose the old version.
