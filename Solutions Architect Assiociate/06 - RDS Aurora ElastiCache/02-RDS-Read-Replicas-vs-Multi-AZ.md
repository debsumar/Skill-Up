---
title: RDS Read Replicas vs Multi AZ
date: 2026-02-10
tags:
  - aws
  - rds
  - read-replicas
  - multi-az
  - disaster-recovery
  - saa-c03
---

# RDS Read Replicas vs Multi AZ

> [!important] Exam Critical
> Understanding the difference between Read Replicas and Multi AZ is ==essential== for the exam. Many questions test this.

## Read Replicas — Scaling Reads

```
                    ┌──────────────┐
                    │  Application │
                    └──────┬───────┘
                     reads & writes
                           │
                    ┌──────▼───────┐
                    │  Main RDS DB │
                    └──┬───────┬───┘
            async      │       │      async
         replication   │       │   replication
                ┌──────▼──┐ ┌──▼──────┐
                │ Replica │ │ Replica │
                │   #1    │ │   #2    │
                └─────────┘ └─────────┘
                  (reads)     (reads)
```

| Feature | Detail |
|---------|--------|
| **Max replicas** | Up to ==15== |
| **Replication** | ==Asynchronous== (eventually consistent) |
| **Placement** | Same AZ, Cross AZ, or ==Cross Region== |
| **Promotion** | Can be promoted to ==independent database== |
| **Operations** | ==SELECT only== (no INSERT, UPDATE, DELETE) |
| **Connection** | App must update connection string to use replicas |

> [!tip] Classic Use Case
> A reporting/analytics team needs to run queries on production data. Instead of overloading the main DB, create a ==Read Replica== and point the reporting app to it. Production is ==unaffected==.

## Read Replica Network Cost

| Scenario | Cost |
|----------|------|
| Same Region, different AZ | ==Free== (managed service exception) |
| Cross Region | ==Paid== (network replication fee) |

```
Same Region (FREE):                Cross Region (PAID):
┌──────────┐    ┌──────────┐      ┌──────────┐    ┌──────────┐
│us-east-1a│───▶│us-east-1b│      │us-east-1 │───▶│eu-west-1 │
│  Main DB │    │ Replica  │      │  Main DB │    │ Replica  │
└──────────┘    └──────────┘      └──────────┘    └──────────┘
   FREE replication                  PAID replication
```

## Multi AZ — Disaster Recovery

```
                    ┌──────────────┐
                    │  Application │
                    └──────┬───────┘
                    One DNS name
                           │
                    ┌──────▼───────┐  SYNC   ┌──────────────┐
                    │   Master     │────────▶│   Standby    │
                    │   (AZ A)     │         │   (AZ B)     │
                    └──────────────┘         └──────────────┘
                     reads & writes           NO reads/writes
                                              (failover only)
```

| Feature | Detail |
|---------|--------|
| **Replication** | ==Synchronous== |
| **Purpose** | ==Disaster Recovery== only (not scaling) |
| **DNS** | ==One DNS name== — automatic failover |
| **Standby** | ==Not readable/writable== — only for failover |
| **Failover triggers** | AZ loss, network failure, instance/storage failure |
| **Manual intervention** | ==None required== — automatic |

> [!warning] Multi AZ ≠ Scaling
> The standby instance is ==not used for reads==. It's purely a failover target.

## Read Replicas as Multi AZ?

> [!success] Yes!
> You ==can== set up Read Replicas with Multi AZ for disaster recovery. This is a ==common exam question==.

## Single AZ → Multi AZ Conversion

| Aspect | Detail |
|--------|--------|
| **Downtime** | ==Zero downtime== operation |
| **Action** | Click "Modify" → Enable Multi AZ |
| **Internal process** | Snapshot taken → Restored to standby → Sync established |

```
Step 1: Snapshot          Step 2: Restore           Step 3: Sync
┌──────────┐             ┌──────────┐              ┌──────────┐
│  Main DB │──snapshot──▶│ Standby  │──sync────────│  Main DB │
│          │             │ (new)    │              │          │
└──────────┘             └──────────┘              └──────────┘
```

## Comparison Table

| Feature | Read Replicas | Multi AZ |
|---------|--------------|----------|
| **Purpose** | ==Read scaling== | ==Disaster Recovery== |
| **Replication** | Async | Sync |
| **Readable** | Yes (SELECT only) | No (standby) |
| **Max count** | 15 | 1 standby |
| **Cross Region** | Yes | No |
| **DNS** | Separate endpoints | Single DNS (auto failover) |
| **Promotion** | Can become independent DB | Standby becomes master on failure |
| **Network cost** | Free (same region) | N/A |

## Questions & Answers

> [!question]- Q1: What is the maximum number of RDS Read Replicas?
> **Answer:**
> Up to ==15== Read Replicas. They can be within the same AZ, cross AZ, or cross region.

> [!question]- Q2: What type of replication do Read Replicas use?
> **Answer:**
> ==Asynchronous== replication. This means reads are ==eventually consistent== — you may get stale data if the replica hasn't caught up yet.

> [!question]- Q3: Can you run INSERT/UPDATE/DELETE on a Read Replica?
> **Answer:**
> ==No==. Read Replicas only support ==SELECT== statements (read-only).

> [!question]- Q4: Is there a network cost for Read Replicas in the same region?
> **Answer:**
> ==No==. Cross-AZ replication within the same region is ==free== for RDS (managed service exception). Cross-region replication ==does== incur a fee.

> [!question]- Q5: What is the purpose of Multi AZ?
> **Answer:**
> ==Disaster Recovery==. It provides a synchronous standby in another AZ with automatic failover via a single DNS name. It is ==not== used for scaling reads.

> [!question]- Q6: Can the Multi AZ standby be used for reads?
> **Answer:**
> ==No==. The standby instance is purely for failover. No one can read from or write to it.

> [!question]- Q7: Is there downtime when converting from Single AZ to Multi AZ?
> **Answer:**
> ==No==. It's a ==zero downtime== operation. Just click "Modify" and enable Multi AZ. Internally, a snapshot is taken, restored to a new standby, and sync is established.

> [!question]- Q8: Can Read Replicas be set up as Multi AZ?
> **Answer:**
> ==Yes==. You can configure Read Replicas with Multi AZ for disaster recovery. This is a ==common exam question==.

> [!question]- Q9: What happens when the master fails in a Multi AZ setup?
> **Answer:**
> The standby is ==automatically promoted== to master. The DNS name stays the same, so no application changes are needed.

> [!question]- Q10: What is the difference between async and sync replication?
> **Answer:**
> - **Async** (Read Replicas): Write accepted on master first, then replicated. ==Eventually consistent==.
> - **Sync** (Multi AZ): Write must be replicated to standby ==before== being accepted. ==Strong consistency==.
