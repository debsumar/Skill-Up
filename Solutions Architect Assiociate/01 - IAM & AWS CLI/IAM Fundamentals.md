---
title: IAM & AWS CLI - Complete Guide
date: 2026-01-29
tags:
  - aws
  - iam
  - cli
  - security
  - saa-c03
---

# IAM & AWS CLI - Complete Guide

## Overview

==IAM (Identity and Access Management)== is a **global service** that manages authentication and authorization in AWS. When you create a user in IAM, it's available in all regions.

## Users and Groups

### Root User

- Created automatically when AWS account is created
- Has ==complete, unrestricted access== to all AWS services
- Should ==only be used for initial account setup==
- Never share root account credentials

> [!danger] Root User Best Practice
> Do not use the root account except when setting up your AWS account. Create an IAM user for daily tasks.

### IAM Users

- Represent ==one physical person== within your organization
- Have their own credentials (password for console, access keys for CLI/SDK)
- Can be grouped together for easier permission management

```
Organization
├── Developers Group
│   ├── Alice
│   ├── Bob
│   └── Charles
├── Operations Group
│   ├── David
│   └── Edward
└── Fred (no group - not best practice)
```

### IAM Groups

- Containers for users only
- ==Cannot contain other groups==
- Users can belong to ==multiple groups==
- No default group that includes all users

> [!tip] Why Groups?
> Assign permissions at the group level for easier security management. Users inherit all permissions from their groups.

## IAM Policies

### What are Policies?

JSON documents that define ==permissions== - what users/groups/roles are allowed or denied to do.

### Policy Inheritance

```
┌───────────────────┐          ┌───────────────────┐
│ Developers Group  │          │ Operations Group  │
│    (Policy A)     │          │    (Policy B)     │
└─────────┬─────────┘          └─────────┬─────────┘
          │                              │
    ┌─────┼─────┐                  ┌─────┴─────┐
    ▼     ▼     ▼                  ▼           ▼
┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐   ┌───────┐
│ Alice │ │  Bob  │ │Charles│ │ David │   │Edward │
└───────┘ └───────┘ └───┬───┘ └───┬───┘   └───────┘
                        │         │
                        └────┬────┘
                             ▼
                    ┌─────────────────┐
                    │   Audit Team    │
                    │   (Policy C)    │
                    └─────────────────┘

┌─────────────────┐
│  Inline Policy  │────▶ Fred (no group)
└─────────────────┘
```

- Users inherit policies from ==all groups== they belong to
- Users can have ==inline policies== attached directly
- Charles (in Developers + Audit) gets policies from both groups

### Policy Structure

```json
{
    "Version": "2012-10-17",
    "Id": "S3-Account-Permissions",
    "Statement": [
        {
            "Sid": "1",
            "Effect": "Allow",
            "Principal": {
                "AWS": ["arn:aws:iam::123456789012:root"]
            },
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": ["arn:aws:s3:::mybucket/*"],
            "Condition": {
                "StringEquals": {
                    "aws:RequestedRegion": "eu-west-1"
                }
            }
        }
    ]
}
```

| Element | Required | Description |
|---------|----------|-------------|
| `Version` | Yes | Policy language version (`2012-10-17`) |
| `Id` | No | Identifier for the policy |
| `Statement` | Yes | One or more permission statements |
| `Sid` | No | Statement identifier |
| `Effect` | Yes | ==Allow== or ==Deny== |
| `Principal` | Sometimes | Account/user/role to apply policy to |
| `Action` | Yes | List of API calls allowed/denied |
| `Resource` | Yes | Resources the actions apply to |
| `Condition` | No | When the policy is in effect |

> [!important] Exam Tip
> Understand Effect, Principal, Action, and Resource - these are critical for the exam.

### Policy Types

| Type | Description |
|------|-------------|
| AWS Managed | Created and managed by AWS (e.g., `AdministratorAccess`) |
| Customer Managed | Created by you, reusable across users/groups/roles |
| Inline | Embedded directly in a single user/group/role |

### Wildcards in Policies

```json
{
    "Effect": "Allow",
    "Action": [
        "iam:Get*",
        "iam:List*"
    ],
    "Resource": "*"
}
```

- `*` means ==anything==
- `iam:Get*` matches `GetUser`, `GetGroup`, `GetPolicy`, etc.
- `Resource: "*"` applies to all resources

## Password Policy

Configure password requirements for IAM users:

- Minimum password length
- Require specific character types:
  - Uppercase letters
  - Lowercase letters
  - Numbers
  - Non-alphanumeric characters
- Allow users to change their own password
- Password expiration (e.g., every 90 days)
- Prevent password reuse

> [!note] Security
> Strong password policies help protect against brute force attacks.

## Multi-Factor Authentication (MFA)

### Why MFA?

Adds a ==second layer of security==:
- Something you **know** (password)
- Something you **have** (MFA device)

Even if password is stolen/hacked, account remains protected without the physical device.

> [!warning] Important
> ==Always enable MFA for root account== and privileged IAM users.

### MFA Device Options

| Device | Description |
|--------|-------------|
| Virtual MFA Device | Google Authenticator, Authy - supports multiple tokens on single device |
| U2F Security Key | YubiKey (3rd party) - physical key, supports multiple root/IAM users |
| Hardware Key Fob | Gemalto (3rd party) |
| Hardware Key Fob (GovCloud) | SurePassID (3rd party) - for AWS GovCloud |

## AWS Access Methods

### Three Ways to Access AWS

| Method | Protection | Use Case |
|--------|------------|----------|
| Management Console | Password + MFA | Web interface |
| CLI (Command Line Interface) | Access Keys | Terminal commands |
| SDK (Software Development Kit) | Access Keys | Programmatic access in code |

### Access Keys

- Generated through Management Console
- Consist of:
  - **Access Key ID** (like username)
  - **Secret Access Key** (like password)
- ==Never share access keys== - treat them like passwords
- Users manage their own access keys

> [!danger] Security Warning
> Access keys are secret! Do not share them with colleagues. Each user should generate their own.

### AWS CLI

```bash
# Configure CLI with access keys
aws configure

# Example: List IAM users
aws iam list-users

# Example: Copy file to S3
aws s3 cp file.txt s3://mybucket/
```

- Direct access to public APIs of AWS services
- Develop scripts to automate tasks
- Open-source (available on GitHub)
- Built on AWS SDK for Python (Boto)

### AWS SDK

- Language-specific libraries (JavaScript, Python, PHP, .NET, Ruby, Java, Go, Node.js, C++)
- Mobile SDK (Android, iOS)
- IoT Device SDK
- Embed in your application code

### AWS CloudShell

- Browser-based terminal in AWS Console
- Pre-configured with AWS CLI
- ==Free to use==
- Files persist across sessions
- Not available in all regions

> [!tip] CloudShell Features
> - Default region is your current console region
> - Upload/download files
> - Multiple tabs and split view

## IAM Roles

### What are Roles?

IAM identities for ==AWS services== (not physical users) that need to perform actions on your behalf.

```
┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│ EC2 Instance │─────▶│   IAM Role   │─────▶│  Access S3   │
└──────────────┘      └──────┬───────┘      └──────────────┘
                  Assumes    │
                             │ Permissions
                             ▼
                      ┌──────────────┐
                      │Access DynamoDB│
                      └──────────────┘
```

### Common Role Use Cases

| Role Type | Description |
|-----------|-------------|
| EC2 Instance Roles | Allow EC2 to access other AWS services |
| Lambda Function Roles | Allow Lambda to access AWS resources |
| CloudFormation Roles | Allow CloudFormation to create resources |

### Creating a Role

1. Choose trusted entity type (AWS service)
2. Select the service (e.g., EC2)
3. Attach permission policies
4. Name the role

> [!note] Trust Policy
> Defines which entities can assume the role (e.g., `ec2.amazonaws.com`).

## IAM Security Tools

### IAM Credentials Report

- ==Account-level== report
- Lists all users and status of their credentials:
  - Password enabled/last used/last changed
  - MFA active
  - Access keys created/last used/last rotated
- Useful for security audits

### IAM Access Advisor

- ==User-level== tool
- Shows service permissions granted to a user
- Shows ==when services were last accessed==
- Helps implement ==least privilege principle==

> [!tip] Use Case
> If a user has access to 100+ services but only uses 5, use Access Advisor to identify and remove unnecessary permissions.

## IAM Best Practices

1. ==Don't use root account== except for initial setup
2. ==One physical user = One AWS user== (never share credentials)
3. Assign users to groups, manage permissions at group level
4. Create ==strong password policy==
5. ==Enable MFA== for root and privileged users
6. Use ==Roles== for AWS services (not access keys)
7. Use ==Access Keys== for CLI/SDK (keep them secret)
8. Audit permissions with:
   - IAM Credentials Report
   - IAM Access Advisor
9. ==Never share IAM users or access keys==

## Summary

| Component | Description |
|-----------|-------------|
| Users | Mapped to physical person, has password for console |
| Groups | Contains users only, for easier permission management |
| Policies | JSON documents defining permissions |
| Roles | Identities for AWS services (EC2, Lambda, etc.) |
| MFA | Multi-factor authentication for extra security |
| CLI | Command line access using access keys |
| SDK | Programmatic access using access keys |
| Credentials Report | Account-level audit of all users |
| Access Advisor | User-level service access audit |

## Questions & Answers

> [!question]- Q1: What is IAM and is it a regional or global service?
> **Answer:**
> IAM stands for Identity and Access Management. It is a ==global service==, meaning when you create a user in IAM, it will be available in all AWS regions.

> [!question]- Q2: What is the difference between IAM users and the root user?
> **Answer:**
> - **Root user**: Created automatically with AWS account, has complete unrestricted access to all services
> - **IAM users**: Created within the account for individual people/applications with specific permissions
> 
> Root should only be used for initial setup; IAM users for daily tasks.

> [!question]- Q3: Can IAM groups contain other groups?
> **Answer:**
> No, IAM groups can ==only contain users==, not other groups. However, users can belong to multiple groups.

> [!question]- Q4: What are the required elements in an IAM policy statement?
> **Answer:**
> Required elements:
> - `Version` - policy language version
> - `Statement` - one or more permission statements
> - `Effect` - Allow or Deny
> - `Action` - API calls allowed/denied
> - `Resource` - resources the actions apply to
> 
> Optional: `Sid`, `Id`, `Condition`. `Principal` is required for resource-based policies.

> [!question]- Q5: How does policy inheritance work when a user belongs to multiple groups?
> **Answer:**
> Users inherit ==all policies from all groups== they belong to. Additionally, users can have inline policies attached directly.
> 
> Final permissions = Union of all attached policies (group policies + inline policies)

> [!question]- Q6: What are the MFA device options available in AWS?
> **Answer:**
> Four options:
> 1. **Virtual MFA devices** - Google Authenticator, Authy
> 2. **U2F Security Keys** - YubiKey (3rd party)
> 3. **Hardware Key Fob** - Gemalto (3rd party)
> 4. **Hardware Key Fob for GovCloud** - SurePassID (3rd party)

> [!question]- Q7: What are the three ways to access AWS?
> **Answer:**
> | Method | Protection |
> |--------|------------|
> | Management Console | Password + MFA |
> | CLI | Access Keys |
> | SDK | Access Keys |

> [!question]- Q8: What is the purpose of IAM Roles and how do they differ from IAM Users?
> **Answer:**
> IAM Roles are identities for ==AWS services== (like EC2, Lambda) that need to perform actions on your behalf.
> 
> | Aspect | IAM User | IAM Role |
> |--------|----------|----------|
> | For | Physical people | AWS services |
> | Credentials | Long-term (password, keys) | Temporary (STS) |

> [!question]- Q9: What are the two IAM security tools and what level do they operate at?
> **Answer:**
> 1. **IAM Credentials Report** (Account-level)
>    - Lists all users and their credential status
> 2. **IAM Access Advisor** (User-level)
>    - Shows service permissions and when last accessed
>    - Helps implement least privilege principle

> [!question]- Q10: What is the principle of least privilege and how can you implement it?
> **Answer:**
> The principle of least privilege means giving users ==only the minimum permissions== they need.
> 
> Implementation:
> - Use IAM Access Advisor to identify unused permissions
> - Create specific policies instead of broad ones
> - Regularly audit with Credentials Report
