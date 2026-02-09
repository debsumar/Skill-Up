---
title: AWS Budget Setup
date: 2026-02-03
tags:
  - aws
  - billing
  - budget
  - cost-management
  - saa-c03
---

# AWS Budget Setup

## Enabling Billing Access for IAM Users

By default, IAM users ==cannot access billing data== even with AdministratorAccess.

### Steps to Enable (Root Account Required)

1. Login as **Root User** (not IAM user)
2. Click account name → **Account**
3. Scroll to **IAM user and role access to Billing information**
4. Click **Edit** → **Activate IAM Access**
5. Save changes

```
Root Account Settings
└── Account
    └── IAM user and role access to Billing information
        └── Activate IAM Access ✓
```

> [!warning] Root Account Only
> This setting can ==only be changed by the root user==, not by IAM administrators.

## Billing Dashboard

Access: **Account Menu → Billing and Cost Management**

### Key Information Available

| Section | Information |
|---------|-------------|
| Home | Month-to-date cost, Forecasted cost, Last month total |
| Bills | Detailed breakdown by service and region |
| Free Tier | Current usage vs limits, Forecast alerts |
| Budgets | Custom spending alerts |

### Bills Breakdown

Navigate: **Bills → Select Month → Charges by Service**

```
Bills
└── December 2023
    └── Charges by Service
        ├── EC2 (EU Ireland): $43
        │   ├── NAT Gateway: $35
        │   ├── EBS: $5
        │   └── Elastic IP: $3
        ├── S3: $2
        └── Lambda: $0.50
```

## Free Tier Dashboard

Shows usage against free tier limits:

| Service | Free Tier Limit | Your Usage | Status |
|---------|-----------------|------------|--------|
| EC2 t2.micro | 750 hours/month | 200 hours | ✅ Green |
| S3 Storage | 5 GB | 3 GB | ✅ Green |
| Lambda | 1M requests | 1.2M | 🔴 Red (Over) |

> [!tip] Monitor Regularly
> If forecast shows ==red==, you will be billed. Turn off unused resources immediately.

## Creating Budgets

### Zero-Spend Budget

Alerts when you spend ==even $0.01==.

**Steps:**
1. Budgets → Create Budget
2. Template: **Zero spend budget**
3. Name: `My Zero Spend Budget`
4. Email: your-email@example.com
5. Create Budget

### Monthly Cost Budget

Set a spending limit with multiple alerts.

**Steps:**
1. Budgets → Create Budget
2. Template: **Monthly cost budget**
3. Budget amount: `$10`
4. Email: your-email@example.com
5. Create Budget

**Default Alerts:**
- 85% of budget reached (actual)
- 100% of budget reached (actual)
- 100% of budget forecasted

```
Budget: $10/month
├── Alert 1: $8.50 actual spend (85%)
├── Alert 2: $10.00 actual spend (100%)
└── Alert 3: $10.00 forecasted (100%)
```

## Best Practices

> [!important] Course Cost Management
> - Set up Zero-Spend Budget immediately
> - Set Monthly Budget to $10 as safety net
> - Check Free Tier dashboard regularly
> - Review Bills if unexpected charges appear
> - Delete resources after hands-on exercises

## Questions & Answers

> [!question]- Q1: Why can't my IAM admin user see billing data?
> **Answer:**
> By default, billing access is disabled for IAM users. The ==root account== must enable "IAM user and role access to Billing information" in Account settings. Even AdministratorAccess policy doesn't grant billing access without this setting.

> [!question]- Q2: How do I find which service is costing me money?
> **Answer:**
> Go to **Billing → Bills → Select Month → Charges by Service**. Expand each service to see regional breakdown and specific charges (e.g., NAT Gateway, EBS volumes, Elastic IPs).
