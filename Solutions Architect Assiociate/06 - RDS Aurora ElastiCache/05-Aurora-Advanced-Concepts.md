---
title: Aurora Advanced Concepts
date: 2026-02-10
tags:
  - aws
  - aurora
  - serverless
  - global-database
  - auto-scaling
  - saa-c03
---

# Aurora Advanced Concepts

## Replica Auto Scaling

```
Before scaling:                    After scaling:
┌────────┐  Writer Endpoint        ┌────────┐  Writer Endpoint
│ Writer │◀────────────────        │ Writer │◀────────────────
└────────┘                         └────────┘
┌────────┐  Reader Endpoint        ┌────────┐
│Reader 1│◀────────────────        │Reader 1│
├────────┤  (high CPU!)            ├────────┤  Reader Endpoint
│Reader 2│                         │Reader 2│◀────────────────
└────────┘                         ├────────┤  (auto scaled)
                                   │Reader 3│
                                   ├────────┤
                                   │Reader 4│
                                   └────────┘
```

- Set up ==auto scaling policies== based on CPU utilization or connection count
- New replicas are ==automatically added== to the reader endpoint
- Scale from 1 to ==15 read replicas==

## Custom Endpoints

```
┌────────┐  Writer Endpoint
│ Writer │◀────────────────
└────────┘
┌──────────────┐
│ db.r3.large  │  ── Regular reader endpoint (not used after custom)
│ db.r3.large  │
└──────────────┘
┌──────────────────┐
│ db.r5.2xlarge    │  ── Custom Endpoint (analytical queries)
│ db.r5.2xlarge    │
└──────────────────┘
```

| Feature | Detail |
|---------|--------|
| **Purpose** | Route specific workloads to ==specific replica subsets== |
| **Use case** | Run analytical queries on ==larger, more powerful== instances |
| **Reader endpoint** | ==Not typically used== after defining custom endpoints |
| **Best practice** | Define ==multiple custom endpoints== for different workload types |

## Aurora Serverless

| Feature | Detail |
|---------|--------|
| **Scaling** | Automated instantiation + auto-scaling based on ==actual usage== |
| **Best for** | ==Infrequent, intermittent, or unpredictable== workloads |
| **Capacity planning** | ==Not needed== |
| **Billing** | Pay ==per second== of Aurora instance usage |
| **Architecture** | Client → ==Proxy Fleet== (managed by Aurora) → Aurora instances |

```
┌────────┐     ┌─────────────┐     ┌──────────────────┐
│ Client │────▶│ Proxy Fleet │────▶│ Aurora Instances  │
│        │     │ (managed)   │     │ (auto-created)    │
└────────┘     └─────────────┘     └──────────────────┘
```

> [!tip] When to Use Serverless
> When you ==can't predict== your database workload. No need to provision capacity in advance.

## Global Aurora

| Feature | Detail |
|---------|--------|
| **Primary region** | All reads and writes |
| **Secondary regions** | Up to ==5 read-only regions== |
| **Replication lag** | < ==1 second== cross-region |
| **Read replicas per secondary** | Up to ==16== |
| **Disaster recovery RTO** | < ==1 minute== |

```
┌─────────────────┐         ┌─────────────────┐
│   us-east-1     │  < 1s   │   eu-west-1     │
│   (Primary)     │────────▶│   (Secondary)   │
│  Read + Write   │  repl.  │   Read Only     │
│                 │         │  (up to 16      │
│                 │         │   replicas)     │
└─────────────────┘         └─────────────────┘
```

> [!important] Exam Hint
> If you see "==less than 1 second to replicate data across regions==" → think ==Global Aurora==.

## Aurora Machine Learning

| Service | Use Case |
|---------|----------|
| **SageMaker** | Any ML model (fraud detection, product recommendations) |
| **Amazon Comprehend** | Sentiment analysis |

```
┌─────────┐  SQL query   ┌────────┐  data    ┌──────────────┐
│   App   │─────────────▶│ Aurora │────────▶│ ML Service   │
│         │              │        │◀────────│ (SageMaker/  │
│         │◀─────────────│        │ predict │  Comprehend) │
│         │  results     │        │         └──────────────┘
└─────────┘              └────────┘
```

- No ML experience needed — use via ==SQL interface==
- Use cases: fraud detection, ads targeting, sentiment analysis, product recommendations

## Questions & Answers

> [!question]- Q1: What is Aurora Replica Auto Scaling?
> **Answer:**
> A feature that ==automatically adds read replicas== when CPU utilization or connection count exceeds a threshold. New replicas are automatically added to the reader endpoint.

> [!question]- Q2: What are Aurora Custom Endpoints?
> **Answer:**
> Endpoints that route traffic to a ==specific subset of Aurora replicas==. Used when you have different instance sizes and want to direct analytical queries to more powerful instances.

> [!question]- Q3: What happens to the reader endpoint after defining custom endpoints?
> **Answer:**
> The reader endpoint is ==generally not used anymore==. You would use multiple custom endpoints for different workload types instead.

> [!question]- Q4: What is Aurora Serverless?
> **Answer:**
> An Aurora mode with ==automated instantiation and auto-scaling== based on actual usage. Clients connect through a ==proxy fleet== managed by Aurora. Pay per second. No capacity planning needed.

> [!question]- Q5: When should you use Aurora Serverless?
> **Answer:**
> For ==infrequent, intermittent, or unpredictable== workloads where you can't predict database usage patterns.

> [!question]- Q6: How many secondary regions can Global Aurora have?
> **Answer:**
> Up to ==5 secondary read-only regions==, each with up to ==16 read replicas==.

> [!question]- Q7: What is the cross-region replication lag for Global Aurora?
> **Answer:**
> Less than ==1 second==. This is a key exam hint — if you see this phrase, think Global Aurora.

> [!question]- Q8: What is the RTO for Global Aurora disaster recovery?
> **Answer:**
> Less than ==1 minute==. You promote a secondary region to become the new primary.

> [!question]- Q9: What ML services does Aurora integrate with?
> **Answer:**
> - ==SageMaker== — any ML model (fraud detection, product recommendations)
> - ==Amazon Comprehend== — sentiment analysis
> All accessible via ==SQL queries==.

> [!question]- Q10: Do you need ML experience to use Aurora Machine Learning?
> **Answer:**
> ==No==. Aurora handles the integration. You just run SQL queries, and Aurora sends data to the ML service and returns predictions as query results.
