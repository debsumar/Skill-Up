---
title: IAM Roles for EC2
date: 2026-02-03
tags:
  - aws
  - ec2
  - iam
  - roles
  - security
  - saa-c03
---

# IAM Roles for EC2

## The Problem: AWS Credentials on EC2

> [!danger] Never Do This!
> ==NEVER== run `aws configure` on an EC2 instance to enter your personal IAM credentials.

### Why It's Dangerous

```
Bad Practice:
┌──────────────────────────────────────────────────────┐
│  EC2 Instance                                        │
│  ┌────────────────────────────────────────────────┐  │
│  │  ~/.aws/credentials                            │  │
│  │  aws_access_key_id = AKIA...                   │  │
│  │  aws_secret_access_key = wJalrXUtn...          │  │
│  └────────────────────────────────────────────────┘  │
│                                                      │
│  ⚠️ Anyone with EC2 access can retrieve these!      │
└──────────────────────────────────────────────────────┘
```

**Risks:**
- Other IAM users can SSH into the instance
- They can read your credentials
- Your credentials could be compromised
- Violates security best practices

## The Solution: IAM Roles

> [!tip] Best Practice
> Attach an ==IAM Role== to the EC2 instance instead of storing credentials.

```
Correct Approach:
┌──────────────┐      Attach      ┌──────────────┐
│   IAM Role   │─────────────────▶│ EC2 Instance │
│ (Permissions)│                  │              │
└──────────────┘                  └──────────────┘
                                        │
                                        │ Automatic
                                        │ Temporary
                                        │ Credentials
                                        ▼
                                  ┌──────────────┐
                                  │  AWS APIs    │
                                  │  (S3, IAM,   │
                                  │   DynamoDB)  │
                                  └──────────────┘
```

### How It Works

1. Create IAM Role with required permissions
2. Attach role to EC2 instance
3. Instance automatically receives temporary credentials
4. Credentials rotate automatically
5. No credentials stored on instance

## Demo: Attaching IAM Role

### Step 1: Create IAM Role (if not exists)

```
IAM Console → Roles → Create Role
├── Trusted entity: AWS Service
├── Use case: EC2
├── Permissions: IAMReadOnlyAccess
└── Role name: DemoRoleForEC2
```

### Step 2: Attach Role to EC2 Instance

```
EC2 Console → Instances → Select Instance
└── Actions → Security → Modify IAM Role
    └── Choose: DemoRoleForEC2
        └── Update IAM Role
```

### Step 3: Verify in Instance Details

```
Instance → Security Tab
└── IAM Role: DemoRoleForEC2 ✓
```

## Testing IAM Role

### Before Attaching Role

```bash
# SSH into instance
ssh -i key.pem ec2-user@<IP>

# Try AWS CLI command
aws iam list-users

# Result: Error
Unable to locate credentials. You can configure credentials 
by running "aws configure".
```

### After Attaching Role

```bash
# Same command now works!
aws iam list-users

# Result: Success
{
    "Users": [
        {
            "UserName": "stephane",
            "UserId": "AIDA...",
            ...
        }
    ]
}
```

## Role Permissions Demo

### With IAMReadOnlyAccess

```bash
aws iam list-users    # ✓ Works
aws iam list-roles    # ✓ Works
aws iam create-user   # ✗ Access Denied (read-only)
```

### After Removing Permission

```bash
# Detach IAMReadOnlyAccess from role
aws iam list-users    # ✗ Access Denied
```

> [!note] Propagation Delay
> Permission changes may take a few seconds to propagate. Retry if you get unexpected results.

## IAM Role vs IAM User Credentials

| Aspect | IAM Role | IAM User Credentials |
|--------|----------|---------------------|
| Storage | Not stored on instance | Stored in ~/.aws/credentials |
| Rotation | Automatic | Manual |
| Security | Secure | Risk of exposure |
| Best Practice | ✅ Yes | ❌ No |
| Scope | Instance-level | User-level |

## Common IAM Policies for EC2

| Policy | Use Case |
|--------|----------|
| `AmazonS3ReadOnlyAccess` | Read S3 buckets |
| `AmazonS3FullAccess` | Full S3 access |
| `IAMReadOnlyAccess` | Read IAM resources |
| `AmazonEC2ReadOnlyAccess` | Read EC2 resources |
| `CloudWatchAgentServerPolicy` | CloudWatch metrics/logs |
| `AmazonSSMManagedInstanceCore` | Systems Manager access |

## Instance Metadata Service (IMDS)

EC2 instances can retrieve role credentials via IMDS:

```bash
# Get temporary credentials (IMDSv1)
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/DemoRoleForEC2

# Response includes:
{
  "AccessKeyId": "ASIA...",
  "SecretAccessKey": "...",
  "Token": "...",
  "Expiration": "2024-01-15T12:00:00Z"
}
```

> [!warning] IMDSv2 Recommended
> AWS recommends IMDSv2 (requires session token) for better security.

## Questions & Answers

> [!question]- Q1: Why should you never use aws configure on EC2?
> **Answer:**
> Running `aws configure` stores your IAM credentials in plain text on the instance. Anyone with SSH access can retrieve them. This violates security best practices and risks credential exposure.

> [!question]- Q2: How do IAM Roles provide credentials to EC2?
> **Answer:**
> When you attach an IAM Role to EC2:
> 1. AWS provides temporary credentials via Instance Metadata Service
> 2. Credentials are automatically rotated
> 3. AWS CLI/SDKs automatically retrieve and use these credentials
> 4. No credentials are stored on the instance

> [!question]- Q3: How do you attach an IAM Role to a running EC2 instance?
> **Answer:**
> EC2 Console → Select Instance → Actions → Security → Modify IAM Role → Select Role → Update
> 
> You can attach/change roles on running instances without stopping them.

> [!question]- Q4: What happens if you remove a permission from an attached role?
> **Answer:**
> The EC2 instance immediately loses that permission. There may be a few seconds of propagation delay. The instance will get "Access Denied" for actions that require the removed permission.

> [!question]- Q5: Can an EC2 instance have multiple IAM Roles?
> **Answer:**
> ==No==. An EC2 instance can have only ==one IAM Role== attached at a time. However, that role can have multiple policies attached to it, granting various permissions.
