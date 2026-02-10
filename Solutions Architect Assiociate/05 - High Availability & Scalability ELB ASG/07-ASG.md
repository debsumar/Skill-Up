---
title: Auto Scaling Groups (ASG)
date: 2026-02-10
tags:
  - aws
  - asg
  - auto-scaling
  - scaling-policies
  - cloudwatch
  - launch-template
  - saa-c03
---

# Auto Scaling Groups (ASG)

When you deploy a website or application, the load changes over time — more users visit, traffic spikes during promotions, and demand drops at night. In the cloud, you can create and terminate servers quickly. An ==Auto Scaling Group (ASG)== automates this process: it ==scales out== (adds instances) when load increases and ==scales in== (removes instances) when load decreases, ensuring you always have the right number of instances running.

> [!important] ASG is Free
> You pay ==nothing== for the ASG itself. You only pay for the ==EC2 instances== (and other resources) it creates underneath.

---

## How ASG Works — Capacity Settings

Every ASG has three capacity parameters that define its boundaries:

| Parameter | Description | Example |
|-----------|-------------|---------|
| **Minimum** | Lowest number of instances — ==always running==, never goes below this | 2 |
| **Desired** | Target number of instances the ASG tries to maintain right now | 4 |
| **Maximum** | Upper limit — ASG ==never exceeds== this, even under extreme load | 7 |

```
┌─────────────────────────────────────────────────────────────────────┐
│                     ASG CAPACITY MODEL                               │
│                                                                     │
│  Minimum          Desired Capacity           Maximum                │
│  Capacity         (current target)           Capacity               │
│     │                   │                       │                   │
│     ▼                   ▼                       ▼                   │
│  ┌──┐ ┌──┐          ┌──┐ ┌──┐              ┌──┐ ┌──┐ ┌──┐         │
│  │EC│ │EC│          │EC│ │EC│    scale     │EC│ │EC│ │EC│         │
│  │2 │ │2 │          │2 │ │2 │    out ──▶   │2 │ │2 │ │2 │         │
│  └──┘ └──┘          └──┘ └──┘              └──┘ └──┘ └──┘         │
│  ◀── 2 ──▶          ◀── 4 ──▶              ◀────── 7 ──────▶      │
│                                                                     │
│  "Never fewer       "I want this            "Never more             │
│   than 2"            many right now"          than 7"               │
│                                                                     │
│  Scale IN ◀──────────────────────────────────────▶ Scale OUT        │
│  (remove instances)                    (add instances)              │
└─────────────────────────────────────────────────────────────────────┘
```

When you change the desired capacity (manually or via a scaling policy), the ASG reacts:
- **Desired > current count** → ASG ==launches== new instances
- **Desired < current count** → ASG ==terminates== excess instances
- **Desired = current count** → no action

---

## ASG + ELB Integration

The real power of ASG comes when paired with an ==Elastic Load Balancer==. Together, they form the backbone of highly available, auto-scaling architectures on AWS.

```
┌──────────────────────────────────────────────────────────────────────┐
│                    ASG + ELB ARCHITECTURE                             │
│                                                                      │
│  ┌────────┐                                                          │
│  │ Users  │                                                          │
│  └───┬────┘                                                          │
│      │                                                               │
│      ▼                                                               │
│  ┌─────────┐    Health checks                                        │
│  │   ELB   │◀──────────────────────────────────────────┐            │
│  └────┬────┘                                           │            │
│       │         Auto-registered                        │            │
│       │         by ASG                                 │            │
│       ▼                                                │            │
│  ┌─────────────────────────────────────────────────────┴──┐         │
│  │                  AUTO SCALING GROUP                      │         │
│  │                                                         │         │
│  │  ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐                  │         │
│  │  │EC2-A│  │EC2-B│  │EC2-C│  │EC2-D│   ← Desired: 4   │         │
│  │  │  ✅ │  │  ✅ │  │  ✅ │  │  ❌ │   ← D is failing │         │
│  │  └─────┘  └─────┘  └─────┘  └──┬──┘      health check│         │
│  │                                 │                      │         │
│  │                          ASG terminates D              │         │
│  │                          and launches E                │         │
│  │                                 │                      │         │
│  │                            ┌────▼────┐                 │         │
│  │                            │  EC2-E  │ ← replacement   │         │
│  │                            │   ✅    │   auto-registered│         │
│  │                            └─────────┘   to ELB        │         │
│  └─────────────────────────────────────────────────────────┘         │
└──────────────────────────────────────────────────────────────────────┘
```

| Feature | Details |
|---------|---------|
| **Auto-register** | New instances are ==automatically added to the ELB target group== |
| **Health check types** | ==EC2 health checks== (default) — checks if the instance is running |
| | ==ELB health checks== (optional, recommended) — checks if the app responds with 200 OK |
| **Unhealthy replacement** | ASG ==terminates unhealthy instances and launches replacements== automatically |
| **Activity history** | Every scaling event is logged in the ASG ==Activity tab== — useful for debugging |

> [!warning] Common Hands-On Issue
> If your ASG keeps terminating and relaunching instances in a loop, it means the ELB health check is failing. Check two things:
> 1. ==Security group== — is port 80 open?
> 2. ==EC2 User Data script== — is the web server starting correctly?

---

## Launch Template

A ==launch template== defines ==how to launch EC2 instances== within the ASG. It contains all the same parameters you'd specify when launching an EC2 instance manually:

```
Launch Template Contents:
├── AMI (Amazon Machine Image)
├── Instance Type (e.g., t2.micro)
├── EC2 User Data (bootstrap script)
├── EBS Volumes (storage configuration)
├── Security Groups (firewall rules)
├── SSH Key Pair
├── IAM Instance Profile (role for EC2)
├── Network configuration
└── Tags
```

| Setting | Where Configured |
|---------|-----------------|
| AMI, instance type, user data, SGs, key pair, IAM role | ==Launch Template== |
| Subnets / Availability Zones | ==ASG itself== (not the launch template) |
| Load balancer / target group | ==ASG itself== |
| Min / Desired / Max capacity | ==ASG itself== |

> [!note] Launch Configurations vs Launch Templates
> ==Launch Configurations are deprecated==. Always use ==Launch Templates== — they support versioning, multiple instance types, and a mix of On-Demand + Spot instances.

### Advanced: Mixed Instance Types

When creating an ASG, you can override the launch template's instance type to specify:
- ==Multiple instance types== (e.g., t2.micro, t3.micro, m5.large)
- A ==mix of On-Demand and Spot instances== (e.g., 70% On-Demand, 30% Spot)
- ==Instance attributes== (e.g., "any instance with 2 vCPUs and 4 GB RAM")

This is configured in the ASG's "Instance type requirements" section, not in the launch template itself.

---

## Scaling Policies — The Four Types

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ASG SCALING POLICIES                              │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                    DYNAMIC SCALING                           │   │
│  │                                                             │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │   │
│  │  │   Target     │  │   Simple     │  │    Step      │     │   │
│  │  │  Tracking    │  │   Scaling    │  │   Scaling    │     │   │
│  │  │              │  │              │  │              │     │   │
│  │  │ "Keep CPU    │  │ "If alarm X  │  │ "If alarm    │     │   │
│  │  │  at 40%"     │  │  → add 2"   │  │  value > 80  │     │   │
│  │  │              │  │              │  │  → add 10    │     │   │
│  │  │ ==Simplest== │  │ Needs alarm  │  │  value > 60  │     │   │
│  │  │ ==Recommended│  │ pre-created  │  │  → add 2"   │     │   │
│  │  └──────────────┘  └──────────────┘  └──────────────┘     │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌──────────────────────┐  ┌──────────────────────┐               │
│  │  SCHEDULED SCALING   │  │  PREDICTIVE SCALING  │               │
│  │                      │  │                      │               │
│  │ "Every Friday 5 PM   │  │ ML analyzes history  │               │
│  │  set min = 10"       │  │ → generates forecast │               │
│  │                      │  │ → pre-schedules       │               │
│  │ You know the pattern │  │ scaling actions       │               │
│  │ in advance           │  │                      │               │
│  │                      │  │ ==Great for cyclical  │               │
│  │                      │  │   traffic patterns== │               │
│  └──────────────────────┘  └──────────────────────┘               │
└─────────────────────────────────────────────────────────────────────┘
```

### 1. Target Tracking Scaling (Dynamic — Recommended)

The ==simplest and most commonly used== scaling policy. You define a target metric value, and the ASG automatically adjusts capacity to maintain it.

| Setting | Example |
|---------|---------|
| **Metric** | Average CPU Utilization |
| **Target value** | ==40%== |
| **Behavior** | ASG adds instances when CPU > 40%, removes when CPU drops |

> [!tip] Behind the Scenes — Two CloudWatch Alarms
> When you create a target tracking policy, AWS ==automatically creates two CloudWatch alarms==:
>
> | Alarm | Trigger | Action |
> |-------|---------|--------|
> | **AlarmHigh** | CPU > ==40%== for 3 datapoints within 3 minutes | ==Scale OUT== (add instances) |
> | **AlarmLow** | CPU < ==28%== for 15 datapoints within 15 minutes | ==Scale IN== (remove instances) |
>
> The scale-in threshold (28%) is automatically calculated — it's lower than the target (40%) to prevent flapping (rapid scale out → scale in → scale out cycles).

```
CPU Utilization Over Time:
                                                    AlarmHigh
100% ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─
                    ╱╲
                   ╱  ╲  stress test
                  ╱    ╲
 40% ─ ─ ─ ─ ─ ╱─ ─ ─ ╲─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ Target
              ╱         ╲
 28% ─ ─ ─ ╱─ ─ ─ ─ ─ ─ ╲─ ─ ─ ─ ─ ─ ─ ─ ─ ─ AlarmLow
          ╱                ╲
  0% ────                   ──────────────────────
     │         │              │              │
     Normal    Scale OUT      Cooldown       Scale IN
               triggered      period         triggered
```

### 2. Simple Scaling (Dynamic)

Triggered by a ==pre-created CloudWatch alarm==. When the alarm fires, a single action occurs.

| Setting | Example |
|---------|---------|
| **Alarm** | "CPU > 70% for 5 minutes" (you create this in CloudWatch first) |
| **Action** | Add 2 capacity units / Remove 1 / Set to exact number |

### 3. Step Scaling (Dynamic)

Like simple scaling, but with ==multiple steps based on alarm severity==:

```
Step Scaling Example:
├── CPU 60-70%  → Add 1 instance
├── CPU 70-80%  → Add 3 instances
├── CPU 80-90%  → Add 5 instances
└── CPU > 90%   → Add 10 instances

The bigger the breach, the more aggressive the response.
```

### 4. Scheduled Scaling

For ==predictable, known patterns== that you can plan for in advance.

| Setting | Example |
|---------|---------|
| **Schedule** | Every Friday at 5:00 PM |
| **Recurrence** | Once / Weekly / Hourly / Cron expression |
| **Action** | Set desired = 10, min = 10 |
| **Start/End time** | Optional — when the schedule takes effect and expires |

**Use case**: You know a big promotion is happening next Saturday, so you schedule a scale-out in advance.

### 5. Predictive Scaling

==Machine learning-driven==. The ASG analyzes your ==historical load patterns==, generates a ==forecast==, and ==pre-schedules scaling actions== before the load arrives.

```
Predictive Scaling Timeline:
                                                    
Past Data (analyzed)          Forecast (generated)     Action (pre-scheduled)
─────────────────────────────────────────────────────────────────────▶
│ Mon Tue Wed Thu Fri Sat Sun │ Mon Tue Wed Thu Fri │                │
│ ▂▃▅▇▅▃▂ ▂▃▅▇▅▃▂ ▂▃▅▇▅▃▂  │ ▂▃▅▇▅▃▂ ▂▃▅▇▅▃▂  │ Scale out at   │
│                             │                     │ predicted peak │
│ "I see a weekly pattern"    │ "Next week will     │ "Adding 5      │
│                             │  look like this"    │  instances at   │
│                             │                     │  Friday 4 PM"  │
```

| Setting | Details |
|---------|---------|
| **Metric** | CPU utilization, Network In/Out, ALB request count, or custom |
| **Target** | e.g., 50% CPU utilization |
| **Requirement** | Needs ==at least 24 hours of historical data== to create initial forecasts; ==14 days recommended== for best accuracy |
| **Best for** | ==Cyclical, repeating traffic patterns== (daily peaks, weekly spikes) |

---

## Good Metrics to Scale On

Choosing the right metric is critical — it should reflect your application's ==actual bottleneck==:

| Metric | When to Use | Example |
|--------|-------------|---------|
| **CPU Utilization** | ==Most common== — compute-bound apps | Web servers, API backends |
| **RequestCountPerTarget** | You know the optimal requests per instance from load testing | "Each instance handles 1,000 concurrent requests optimally" |
| **Average Network In/Out** | ==Network-bound== apps with heavy uploads/downloads | File processing, media streaming |
| **Custom CloudWatch Metric** | Application-specific bottleneck | Queue depth, active connections, memory usage |

```
Choosing Your Scaling Metric:

Is your app CPU-bound?
├── Yes → CPUUtilization (most common)
│
Is your app request-bound?
├── Yes → RequestCountPerTarget (from ALB)
│         e.g., "3 requests per target currently,
│                optimal is 1000 per target"
│
Is your app network-bound?
├── Yes → NetworkIn / NetworkOut
│
None of the above?
└── Push a Custom CloudWatch Metric from your app
```

---

## Scaling Cooldown

After every scaling action (adding or removing instances), the ASG enters a ==cooldown period== where it ==refuses to take any further scaling actions==. This prevents rapid, oscillating scale-out/scale-in cycles.

| Parameter | Value |
|-----------|-------|
| **Default cooldown** | ==300 seconds== (5 minutes) |
| **Purpose** | Allow metrics to ==stabilize== after new instances come online |
| **During cooldown** | ==No additional launch or terminate actions== |
| **Can be customized** | Yes — per scaling policy |

```
Scaling Cooldown Flow:

Alarm triggers     New instance      Metrics         Next action
scale out          launches          stabilize       allowed
    │                  │                │                │
    ▼                  ▼                ▼                ▼
────┼──────────────────┼────────────────┼────────────────┼──────▶ Time
    │                  │                │                │
    │◀─────────── Cooldown (300s) ─────────────────────▶│
    │                  │                │                │
    │           During this period:                      │
    │           "Is cooldown in effect?"                 │
    │           ├── YES → Ignore scaling action          │
    │           └── NO  → Proceed with scaling           │
```

### Optimizing Cooldown

> [!tip] Use Ready-to-Use AMIs
> If your EC2 instances take 5 minutes to boot and configure (installing packages, pulling code), the cooldown must be at least that long. But if you use a ==pre-baked AMI== (Golden AMI) with everything pre-installed, instances serve traffic ==within seconds==. This lets you:
> - ==Reduce cooldown== to 60 seconds or less
> - Achieve ==more dynamic, responsive scaling==
> - React faster to traffic spikes

> [!important] Enable Detailed Monitoring
> By default, EC2 sends metrics to CloudWatch every ==5 minutes==. Enable ==detailed monitoring== to get metrics every ==1 minute==. This makes your scaling policies ==5x more responsive== — the ASG detects load changes faster and reacts sooner.

---

## Hands-On Walkthrough

### Creating an ASG

1. ==Terminate all existing EC2 instances== (start clean)
2. Go to ==EC2 Console → Auto Scaling Groups → Create==
3. Name: `DemoASG`
4. ==Create a Launch Template== first:
   - Name: `MyDemoTemplate`
   - AMI: Amazon Linux 2 (free tier)
   - Instance type: t2.micro
   - Key pair: your existing key
   - Security group: select existing (with port 80 open)
   - User Data: paste your web server bootstrap script
   - ==Do NOT specify subnets== in the launch template
5. Back in ASG creation, select the launch template
6. Choose ==VPC and subnets== (e.g., eu-west-1a, 1b, 1c)
7. ==Attach to existing load balancer== → select your ALB's target group
8. Enable ==ELB health checks== (in addition to EC2 health checks)
9. Set capacity: Desired = 1, Min = 1, Max = 1 (for now)
10. Skip scaling policies for now → Create

### Observing ASG Behavior

After creation, the ASG immediately launches one instance:
- ==Activity tab== shows: "Launching a new EC2 instance — desired capacity is 1"
- ==Instance Management tab== shows the new instance
- ==Target Group== shows the instance being registered (initially unhealthy → healthy after bootstrap)
- ==ALB DNS== returns "Hello World" once the instance passes health checks

### Manual Scaling Test

1. Edit ASG → set Desired = 2, Max = 2 → Update
2. Activity tab shows: "Launching a new EC2 instance — desired capacity changed from 1 to 2"
3. Both instances appear in the target group and ALB rotates between them
4. Edit ASG → set Desired = 1 → Update
5. Activity tab shows: "Terminating EC2 instance — desired capacity changed from 2 to 1"
6. ASG picks one instance to terminate and deregisters it from the target group

### Target Tracking Policy Demo

1. Edit ASG → set Max = 3
2. Go to ==Automatic Scaling → Create dynamic scaling policy==
3. Choose ==Target tracking==
4. Metric: ==Average CPU Utilization==, Target: ==40==
5. Create policy
6. SSH into the running instance and install `stress`:
   ```bash
   sudo amazon-linux-extras install epel -y
   sudo yum install stress -y
   stress -c 4    # Pins CPU to 100%
   ```
7. Wait ~3 minutes → ==AlarmHigh triggers== → ASG scales from 1 → 2 → 3
8. Stop the stress test (reboot the instance)
9. Wait ~15 minutes → ==AlarmLow triggers== → ASG scales from 3 → 2 → 1

> [!note] CloudWatch Alarms Created Automatically
> Go to ==CloudWatch → Alarms== to see the two alarms created by the target tracking policy:
> - **AlarmHigh**: CPU > 40% for 3 datapoints in 3 minutes → scale out
> - **AlarmLow**: CPU < 28% for 15 datapoints in 15 minutes → scale in

### Cleanup

1. Delete the scaling policy
2. Delete the ASG (this terminates all instances)
3. Delete the launch template
4. Delete the ALB and target group if no longer needed

---

## Summary

| Concept | Key Fact |
|---------|----------|
| **ASG cost** | ==Free== — pay only for EC2 instances |
| **Capacity** | Min ≤ Desired ≤ Max |
| **ELB integration** | Auto-register, ELB health checks, auto-replace unhealthy |
| **Launch Template** | Defines HOW to launch instances (AMI, type, SGs, user data) |
| **Target Tracking** | ==Simplest== — "keep CPU at 40%" — auto-creates 2 CloudWatch alarms |
| **Simple/Step** | Requires pre-created CloudWatch alarm |
| **Scheduled** | For known patterns — "Fridays at 5 PM, min = 10" |
| **Predictive** | ML-driven — analyzes history, forecasts, pre-schedules |
| **Cooldown** | ==300 seconds default== — no actions during this period |
| **Detailed monitoring** | ==1-minute metrics== (vs 5-min default) — more responsive scaling |

---

## Questions & Answers

> [!question]- Q1: What is an Auto Scaling Group and what does it cost?
> **Answer:**
> An ASG automatically ==scales EC2 instances out (add) or in (remove)== based on demand. You define minimum, desired, and maximum capacity. The ASG itself is ==completely free== — you only pay for the EC2 instances and other resources it creates.

> [!question]- Q2: What are the three capacity settings and how do they relate?
> **Answer:**
> - **Minimum**: Lowest number of instances — ASG ==never goes below== this
> - **Desired**: The target count the ASG tries to maintain right now
> - **Maximum**: Upper limit — ASG ==never exceeds== this
>
> Relationship: ==Min ≤ Desired ≤ Max==. Scaling policies adjust the desired capacity within the min/max bounds.

> [!question]- Q3: How does ASG integrate with ELB?
> **Answer:**
> New instances are ==automatically registered== with the ELB's target group. The ELB performs health checks, and if an instance is deemed unhealthy, the ASG ==terminates it and launches a replacement== that is automatically registered with the ELB. This creates a ==self-healing architecture==.

> [!question]- Q4: What is a launch template and what does it contain?
> **Answer:**
> A launch template defines ==how to launch EC2 instances==: AMI, instance type, user data, EBS volumes, security groups, key pair, IAM roles. ==Subnets and load balancer settings are configured in the ASG itself==, not the launch template. Launch templates replaced the deprecated ==launch configurations==.

> [!question]- Q5: What is target tracking scaling and what happens behind the scenes?
> **Answer:**
> You set a ==target metric value== (e.g., "keep CPU at 40%") and the ASG auto-adjusts. Behind the scenes, AWS creates ==two CloudWatch alarms==:
> - **AlarmHigh**: metric exceeds target → ==scale out==
> - **AlarmLow**: metric drops well below target → ==scale in==
>
> The scale-in threshold is automatically set lower than the target to prevent flapping.

> [!question]- Q6: What is the difference between simple, step, and target tracking scaling?
> **Answer:**
> - **Target tracking**: ==Simplest== — "keep metric at X%" — ASG handles everything automatically
> - **Simple scaling**: One CloudWatch alarm → one action (add N or remove N instances)
> - **Step scaling**: One alarm with ==multiple steps== based on severity (e.g., CPU 70% → add 1, CPU 90% → add 10)
>
> Target tracking is ==recommended for most use cases==.

> [!question]- Q7: What is predictive scaling and when should you use it?
> **Answer:**
> Predictive scaling uses ==machine learning== to analyze ==historical load patterns==, generate a ==forecast==, and ==pre-schedule scaling actions== before the load arrives. Use it for ==cyclical, repeating traffic patterns== (daily peaks, weekly spikes). It needs at least ==24 hours== of historical data to create initial forecasts, but ==14 days is recommended== for best accuracy.

> [!question]- Q8: What is the scaling cooldown and why does it exist?
> **Answer:**
> A ==300-second (default) period after any scaling action== during which the ASG ==refuses to take further scaling actions==. This allows new instances to come online and metrics to stabilize before the next decision. Without cooldown, the ASG could rapidly oscillate between scaling out and scaling in.

> [!question]- Q9: How can you make scaling more responsive?
> **Answer:**
> Two key optimizations:
> 1. Use a ==ready-to-use AMI (Golden AMI)== with pre-installed software → instances serve traffic faster → cooldown can be reduced
> 2. Enable ==detailed monitoring== on EC2 instances → metrics every ==1 minute== instead of 5 → ASG detects load changes 5x faster

> [!question]- Q10: What are the best metrics to scale on?
> **Answer:**
> | Metric | Best For |
> |--------|----------|
> | ==CPU Utilization== | Most common — compute-bound apps |
> | ==RequestCountPerTarget== | When you know optimal requests per instance from load testing |
> | ==Network In/Out== | Network-bound apps (uploads, downloads, streaming) |
> | ==Custom CloudWatch Metric== | Application-specific bottlenecks (queue depth, memory, connections) |
>
> Choose the metric that reflects your application's ==actual bottleneck==.
