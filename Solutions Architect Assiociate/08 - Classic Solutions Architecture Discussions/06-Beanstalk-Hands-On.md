---
title: "Elastic Beanstalk - Hands On"
date: 2026-02-10
tags:
  - aws
  - elastic-beanstalk
  - hands-on
  - cloudformation
  - saa-c03
---

# Elastic Beanstalk - Hands On

## Overview

In this hands-on walkthrough, we create an Elastic Beanstalk application from scratch using the ==AWS Management Console==. The goal is to see how Beanstalk takes the complex architectures we've been building manually — ELB, ASG, EC2, security groups — and provisions them ==automatically== with just a few clicks.

This is a practical demonstration of the concepts from [[05-Beanstalk-Overview|Beanstalk Overview]]. We'll create a simple web application, observe what Beanstalk creates under the hood, and then ==clean everything up== to avoid charges.

> [!warning] Cost Awareness
> Even though Beanstalk itself is free, the resources it creates (EC2 instances, ELB, etc.) ==cost money==. Always delete your Beanstalk environment when you're done experimenting. We'll cover cleanup at the end of this walkthrough.

## Step 1: Navigate to Elastic Beanstalk

1. Open the ==AWS Management Console==
2. Search for "Elastic Beanstalk" in the search bar
3. Click on ==Elastic Beanstalk== to open the service console
4. Click ==Create Application==

```
┌─────────────────────────────────────────────────────────────────┐
│  AWS Console  ▶  Services  ▶  Elastic Beanstalk                │
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                                                           │  │
│  │   Welcome to AWS Elastic Beanstalk                       │  │
│  │                                                           │  │
│  │   [ Create Application ]                                  │  │
│  │                                                           │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

## Step 2: Configure the Application

### Application Information

| Setting | Value | Notes |
|---------|-------|-------|
| **Application name** | `MyApplication` | Any name you like |
| **Platform** | ==Node.js== (or any supported platform) | Beanstalk supports Go, Java, .NET, Node.js, PHP, Python, Ruby, Docker |
| **Platform branch** | Latest recommended version | Beanstalk manages the platform version |
| **Application code** | ==Sample application== | Beanstalk provides a sample "Hello World" app |

For this hands-on, we use the ==sample application== that Beanstalk provides. In a real deployment, you'd upload your own code as a zip file or connect to a code repository.

### Presets (Deployment Mode)

You'll be asked to choose a configuration preset:

```
┌─────────────────────────────────────────────────────────────────┐
│  Configuration Presets                                          │
│                                                                 │
│  ○  Single instance (free tier eligible)     ◀── Choose this   │
│     - 1 EC2 instance, no ELB                                   │
│     - Good for development                                      │
│     - Cheapest option                                           │
│                                                                 │
│  ○  Single instance (using Spot instances)                      │
│     - Same as above but with Spot pricing                       │
│                                                                 │
│  ○  High availability                                           │
│     - ELB + ASG + Multi-AZ                                      │
│     - Production-ready                                          │
│                                                                 │
│  ○  High availability (using Spot and On-Demand instances)      │
│     - Same as above with mixed instance types                   │
│                                                                 │
│  ○  Custom configuration                                        │
│     - Full control over all settings                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

For this demo, choose ==Single instance (free tier eligible)== to minimize costs. This creates one EC2 instance with an Elastic IP — no load balancer.

> [!tip] Free Tier
> The single instance preset uses a ==t2.micro== (or t3.micro) instance, which is eligible for the AWS Free Tier. If you're within your first 12 months of AWS, this won't cost anything (within Free Tier limits).

## Step 3: Configure Service Access

Beanstalk needs permissions to create and manage resources on your behalf:

| Setting | Value | Notes |
|---------|-------|-------|
| **Service role** | Create new or use existing | Beanstalk needs a role to manage resources |
| **EC2 key pair** | Optional | Only needed if you want to SSH into instances |
| **EC2 instance profile** | Create new or use existing | The IAM role attached to EC2 instances |

The ==service role== allows Beanstalk to call AWS APIs (create EC2 instances, ELBs, etc.). The ==instance profile== is the IAM role that the EC2 instances themselves assume — this controls what your application can access (S3, DynamoDB, etc.).

> [!note] IAM Roles
> Beanstalk typically creates two roles:
> - `aws-elasticbeanstalk-service-role` — for the Beanstalk service itself
> - `aws-elasticbeanstalk-ec2-role` — for the EC2 instances
> These are created automatically on first use. You can customize them later.

## Step 4: Review and Create

After configuring, review your settings and click ==Create Application==. Beanstalk now begins provisioning:

```
┌─────────────────────────────────────────────────────────────────┐
│  Creating environment: MyApplication-env                        │
│                                                                 │
│  ⏳ Creating CloudFormation stack...                            │
│  ⏳ Creating security group...                                  │
│  ⏳ Creating Elastic IP...                                      │
│  ⏳ Launching EC2 instance...                                   │
│  ⏳ Installing platform (Node.js)...                            │
│  ⏳ Deploying sample application...                             │
│  ⏳ Running health checks...                                    │
│  ✅ Environment health: OK                                      │
│                                                                 │
│  Total time: ~5-10 minutes                                      │
└─────────────────────────────────────────────────────────────────┘
```

## Step 5: Observe What Beanstalk Created

Once the environment is ready, explore what was created:

### Beanstalk Dashboard

The Beanstalk console shows you:
- ==Environment health== — Green (OK), Yellow (Warning), Red (Degraded), Grey (Unknown)
- ==Running version== — which application version is deployed
- ==Platform== — the runtime (e.g., Node.js 18 on Amazon Linux 2023)
- ==Environment URL== — the public URL to access your application

### The Environment URL

Beanstalk assigns a ==unique URL== to your environment:

```
http://MyApplication-env.eba-xxxxxxxxxx.us-east-1.elasticbeanstalk.com
```

Click this URL and you'll see the ==sample application== running — a simple web page confirming Beanstalk is working.

> [!tip] Custom Domain
> In production, you'd create a ==Route 53 alias record== pointing your domain (e.g., `www.myapp.com`) to the Beanstalk environment URL or the ELB DNS name. The Beanstalk URL itself is for development/testing.

### CloudFormation Stack

Navigate to ==CloudFormation== in the AWS Console. You'll see a stack created by Beanstalk:

```
┌─────────────────────────────────────────────────────────────────┐
│  CloudFormation  ▶  Stacks                                      │
│                                                                 │
│  Stack Name: awseb-e-xxxxxxxxxx-stack                           │
│  Status: CREATE_COMPLETE ✅                                     │
│                                                                 │
│  Resources created:                                             │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  Type                    │  Logical ID                    │  │
│  │─────────────────────────│────────────────────────────────│  │
│  │  AWS::EC2::Instance      │  AWSEBAutoScalingGroup         │  │
│  │  AWS::EC2::SecurityGroup │  AWSEBSecurityGroup            │  │
│  │  AWS::EC2::EIP           │  AWSEBElasticIP (single inst)  │  │
│  │  AWS::AutoScaling::ASG   │  AWSEBAutoScalingGroup         │  │
│  │  AWS::AutoScaling::LC    │  AWSEBAutoScalingLaunchConfig  │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
│  (High Availability mode would also show ELB resources)         │
└─────────────────────────────────────────────────────────────────┘
```

This is the key insight: ==Beanstalk uses CloudFormation under the hood==. Every resource is defined in a CloudFormation template, created as a stack, and managed as infrastructure as code.

### EC2 Instance

Navigate to ==EC2== in the console. You'll see the instance Beanstalk launched:
- Tagged with the Beanstalk environment name
- Running the platform you selected (Node.js on Amazon Linux)
- Security group allowing inbound HTTP (port 80)
- The sample application is running on this instance

## Step 6: Explore Beanstalk Features

While the environment is running, explore the Beanstalk console tabs:

| Tab | What It Shows |
|-----|---------------|
| **Configuration** | All environment settings — instances, capacity, load balancer, database, etc. |
| **Logs** | Application and platform logs from EC2 instances |
| **Health** | Detailed health status of each instance |
| **Monitoring** | CloudWatch metrics — CPU, network, latency, requests |
| **Events** | Timeline of all environment events (deployments, scaling, errors) |
| **Tags** | Resource tags for cost allocation and organization |

The ==Monitoring== tab is particularly useful — it shows the same CloudWatch metrics you'd normally configure manually, but Beanstalk sets them up automatically.

## Step 7: Clean Up (Critical!)

> [!danger] Always Clean Up!
> Beanstalk resources ==cost money==. Even a single t2.micro instance costs ~$8.50/month if left running. Always delete your environment when done experimenting.

### Delete the Environment

1. In the Beanstalk console, select your environment
2. Click ==Actions== → ==Terminate Environment==
3. Confirm by typing the environment name
4. Beanstalk will ==delete the CloudFormation stack== and all associated resources

```
┌─────────────────────────────────────────────────────────────────┐
│  Terminating environment: MyApplication-env                     │
│                                                                 │
│  ⏳ Deleting EC2 instance...                                    │
│  ⏳ Releasing Elastic IP...                                     │
│  ⏳ Deleting security group...                                  │
│  ⏳ Deleting CloudFormation stack...                            │
│  ✅ Environment terminated                                      │
│                                                                 │
│  Total time: ~5 minutes                                         │
└─────────────────────────────────────────────────────────────────┘
```

### Delete the Application (Optional)

After terminating the environment, you can also delete the application itself:
1. Go to ==Applications== in the Beanstalk console
2. Select your application
3. Click ==Actions== → ==Delete Application==

This removes the application and all stored versions. The application itself doesn't cost anything (versions are stored in S3), but it's good practice to clean up.

> [!tip] Verify Cleanup
> After deletion, check these services to make sure nothing was left behind:
> - ==EC2== — no running instances tagged with Beanstalk
> - ==CloudFormation== — no stacks in CREATE_COMPLETE or DELETE_FAILED state
> - ==Elastic IPs== — no unattached Elastic IPs (these cost money!)
> - ==Security Groups== — Beanstalk SGs should be deleted (may take a few minutes)

## What We Learned

This hands-on demonstrated the core Beanstalk workflow:

```
┌──────────┐   ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│  Create  │   │  Beanstalk   │   │  CloudForm-  │   │  Resources   │
│  App +   │──▶│  provisions  │──▶│  ation stack │──▶│  running     │
│  Environ │   │  everything  │   │  created     │   │  & healthy   │
└──────────┘   └──────────────┘   └──────────────┘   └──────────────┘
                                                              │
┌──────────┐   ┌──────────────┐   ┌──────────────┐           │
│  All     │◀──│  CloudForm-  │◀──│  Terminate   │◀──────────┘
│  clean   │   │  ation stack │   │  environment │
│          │   │  deleted     │   │              │
└──────────┘   └──────────────┘   └──────────────┘
```

| Step | What Happened | Key Takeaway |
|------|---------------|--------------|
| Create | Clicked a few buttons, chose platform | ==Minutes== vs days of manual setup |
| Provision | Beanstalk created CF stack → EC2, SG, EIP | ==CloudFormation under the hood== |
| Access | Got a public URL immediately | ==Environment URL== for testing |
| Monitor | Dashboard showed health, logs, metrics | ==Built-in monitoring== |
| Cleanup | Terminated environment → all resources deleted | ==Always clean up== to avoid charges |

## Questions & Answers

> [!question]- Q1: What resources does Beanstalk create for a single-instance environment?
> **Answer:**
> For a single-instance environment, Beanstalk creates:
> - ==One EC2 instance== running your application
> - An ==Elastic IP== (static public IP for the instance)
> - A ==Security Group== allowing inbound HTTP (port 80)
> - An ==Auto Scaling Group== (with min=1, max=1)
> - A ==CloudFormation stack== managing all these resources
> No ELB is created in single-instance mode — traffic goes directly to the EC2 instance via the Elastic IP.

> [!question]- Q2: How does Beanstalk use CloudFormation?
> **Answer:**
> When you create a Beanstalk environment, it generates a ==CloudFormation template== defining all required resources and creates a ==CloudFormation stack== from that template. CloudFormation then provisions the actual AWS resources (EC2, ELB, ASG, SGs, etc.). When you terminate the environment, Beanstalk deletes the CloudFormation stack, which ==automatically deletes all resources==. This is why you should always delete through Beanstalk, not manually.

> [!question]- Q3: What is the Beanstalk environment URL?
> **Answer:**
> Each Beanstalk environment gets a ==unique URL== in the format:
> `http://<env-name>.eba-<random>.region.elasticbeanstalk.com`
> This URL points to the ELB (in HA mode) or directly to the EC2 instance's Elastic IP (in single-instance mode). In production, you'd create a ==Route 53 alias record== pointing your custom domain to this URL.

> [!question]- Q4: What's the difference between the service role and instance profile?
> **Answer:**
> - ==Service role== (`aws-elasticbeanstalk-service-role`): Used by the Beanstalk service itself to call AWS APIs — create EC2 instances, manage ELBs, update ASGs, etc.
> - ==Instance profile== (`aws-elasticbeanstalk-ec2-role`): The IAM role attached to the EC2 instances. Controls what your ==application code== can access — S3 buckets, DynamoDB tables, SQS queues, etc.
> Both are created automatically on first use.

> [!question]- Q5: Why should you always delete through Beanstalk and not manually?
> **Answer:**
> Beanstalk manages resources through ==CloudFormation==. If you manually delete an EC2 instance or security group, CloudFormation's state becomes inconsistent — it thinks the resource still exists. This can cause ==stack drift==, failed updates, and orphaned resources. Always use ==Beanstalk's Terminate Environment== action, which properly deletes the CloudFormation stack and all resources in the correct order.

> [!question]- Q6: What does the Beanstalk health status mean?
> **Answer:**
> - ==Green==: Environment is healthy, all instances passing health checks
> - ==Yellow==: Warning — some health checks failing or metrics degraded
> - ==Red==: Degraded — significant issues, instances failing health checks
> - ==Grey==: Unknown — environment is updating or health data unavailable
> Beanstalk performs ==enhanced health monitoring== beyond basic EC2 status checks, including application-level health checks.

> [!question]- Q7: What monitoring does Beanstalk provide out of the box?
> **Answer:**
> Beanstalk automatically configures ==CloudWatch metrics== for:
> - CPU utilization, network I/O
> - Request count, latency (with ELB)
> - HTTP status codes (2xx, 3xx, 4xx, 5xx)
> - Environment health status
> - Instance health and count
> You can view these in the Beanstalk ==Monitoring tab== or in the CloudWatch console. No manual setup required.

> [!question]- Q8: Can you SSH into Beanstalk EC2 instances?
> **Answer:**
> Yes, if you configured an ==EC2 key pair== during environment creation. You can SSH into the instances just like any other EC2 instance. The security group may need to be updated to allow inbound SSH (port 22) from your IP. This is useful for debugging but ==not recommended for production== — use Beanstalk logs and monitoring instead.

> [!question]- Q9: What happens to uploaded application versions when you delete the application?
> **Answer:**
> Application versions are stored as ==zip files in an S3 bucket== that Beanstalk creates. When you delete the application, Beanstalk deletes the versions from S3. However, the S3 bucket itself (named `elasticbeanstalk-region-account-id`) may persist. Check S3 after cleanup to ensure no unwanted buckets remain.

> [!question]- Q10: What are the key things to verify after cleaning up a Beanstalk environment?
> **Answer:**
> After terminating an environment, verify:
> 1. ==EC2==: No running instances tagged with your Beanstalk environment
> 2. ==CloudFormation==: Stack is in `DELETE_COMPLETE` state (not `DELETE_FAILED`)
> 3. ==Elastic IPs==: No unattached EIPs (these cost ==~$3.65/month== if not associated)
> 4. ==Security Groups==: Beanstalk SGs should be deleted (may take a few minutes)
> 5. ==S3==: Check for leftover Beanstalk buckets with application versions
> If the CloudFormation stack is in `DELETE_FAILED`, check the events tab for the resource that failed to delete and remove it manually.
