---
title: System Design for ERP
tags: [system-design, architecture, scalability]
created: 2026-02-03
---

# System Design for ERP

## 🎯 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                           CLIENTS                                    │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐               │
│  │   Web   │  │ Mobile  │  │  Admin  │  │   API   │               │
│  │   App   │  │   App   │  │  Panel  │  │ Clients │               │
│  └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘               │
└───────┼────────────┼────────────┼────────────┼──────────────────────┘
        │            │            │            │
        └────────────┴─────┬──────┴────────────┘
                           │
                    ┌──────┴──────┐
                    │    CDN      │ (Static assets)
                    │  CloudFront │
                    └──────┬──────┘
                           │
                    ┌──────┴──────┐
                    │Load Balancer│
                    │    (ALB)    │
                    └──────┬──────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
   ┌────┴────┐       ┌─────┴─────┐      ┌────┴────┐
   │ API     │       │ API       │      │ API     │
   │Server 1 │       │ Server 2  │      │Server 3 │
   └────┬────┘       └─────┬─────┘      └────┬────┘
        │                  │                  │
        └──────────────────┼──────────────────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
   ┌────┴────┐       ┌─────┴─────┐      ┌────┴────┐
   │  MySQL  │       │  MongoDB  │      │  Redis  │
   │ Primary │       │  Cluster  │      │ Cluster │
   └────┬────┘       └───────────┘      └─────────┘
        │
   ┌────┴────┐
   │  MySQL  │
   │ Replica │
   └─────────┘
```

## 🏗️ Component Design

### API Gateway Pattern

```
┌─────────────────────────────────────────────────────────┐
│                    API Gateway                           │
├─────────────────────────────────────────────────────────┤
│  • Authentication (JWT validation)                       │
│  • Rate limiting (per user/IP)                          │
│  • Request routing                                       │
│  • Response caching                                      │
│  • Request/Response transformation                       │
│  • Logging & monitoring                                  │
└────────────────────────┬────────────────────────────────┘
                         │
     ┌───────────────────┼───────────────────┐
     │                   │                   │
┌────┴────┐        ┌─────┴─────┐       ┌────┴────┐
│ Student │        │    Fee    │       │   HR    │
│ Service │        │  Service  │       │ Service │
└─────────┘        └───────────┘       └─────────┘
```

### Microservices vs Monolith

```
Monolith (Recommended for small-medium ERP):
┌─────────────────────────────────────┐
│           ERP Application           │
│  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐  │
│  │ HR  │ │ Fee │ │Exam │ │ LMS │  │
│  └─────┘ └─────┘ └─────┘ └─────┘  │
│         Shared Database             │
└─────────────────────────────────────┘

Microservices (Large scale):
┌─────────┐ ┌─────────┐ ┌─────────┐
│HR Service│ │Fee Svc  │ │Exam Svc │
│   DB    │ │   DB    │ │   DB    │
└────┬────┘ └────┬────┘ └────┬────┘
     │           │           │
     └───────────┼───────────┘
                 │
          Message Queue
```

### Database Design Strategy

```
┌─────────────────────────────────────────────────────────┐
│                  Database Strategy                       │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  MySQL (Primary - ACID transactions)                     │
│  ├── Students, Classes, Teachers                         │
│  ├── Fees, Payments, Invoices                           │
│  ├── HR, Payroll, Attendance                            │
│  └── Finance, Accounts, Transactions                     │
│                                                          │
│  MongoDB (Flexible schemas, high write)                  │
│  ├── Audit logs                                          │
│  ├── Notifications                                       │
│  ├── LMS content metadata                               │
│  └── Analytics events                                    │
│                                                          │
│  Redis (Caching, sessions)                              │
│  ├── Session data                                        │
│  ├── API response cache                                  │
│  ├── Rate limiting counters                             │
│  └── Real-time data (online users)                      │
│                                                          │
│  S3 (File storage)                                       │
│  ├── Student documents                                   │
│  ├── Assignment submissions                              │
│  └── Report exports                                      │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

## 📈 Scalability Patterns

### Horizontal Scaling

```
Before:                    After:
┌─────────┐               ┌─────────┐
│ Server  │               │   LB    │
│  (1x)   │               └────┬────┘
└─────────┘                    │
                    ┌──────────┼──────────┐
                    │          │          │
               ┌────┴───┐ ┌────┴───┐ ┌────┴───┐
               │Server 1│ │Server 2│ │Server 3│
               └────────┘ └────────┘ └────────┘
```

### Caching Strategy

```javascript
// Multi-level caching
class CacheService {
  constructor(redis, localCache) {
    this.redis = redis;
    this.local = localCache; // In-memory (node-cache)
  }
  
  async get(key) {
    // L1: Local cache (fastest)
    let value = this.local.get(key);
    if (value) return value;
    
    // L2: Redis (shared across instances)
    value = await this.redis.get(key);
    if (value) {
      this.local.set(key, value, 60); // Cache locally for 60s
      return JSON.parse(value);
    }
    
    return null;
  }
  
  async set(key, value, ttl = 3600) {
    await this.redis.setex(key, ttl, JSON.stringify(value));
    this.local.set(key, value, Math.min(ttl, 60));
  }
  
  async invalidate(pattern) {
    const keys = await this.redis.keys(pattern);
    if (keys.length) await this.redis.del(keys);
    this.local.flushAll();
  }
}

// Usage
const feeStructure = await cache.get(`fee:class:${classId}`);
if (!feeStructure) {
  const data = await FeeStructure.findOne({ classId });
  await cache.set(`fee:class:${classId}`, data);
}
```

### Queue-Based Processing

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   API       │────>│   Queue     │────>│   Worker    │
│  (Producer) │     │  (Redis/    │     │ (Consumer)  │
│             │     │   RabbitMQ) │     │             │
└─────────────┘     └─────────────┘     └─────────────┘

Use cases:
• Bulk fee generation
• Report generation
• Email/SMS notifications
• PDF generation
• Data exports
```

```javascript
// Queue setup
const Queue = require('bull');

const queues = {
  email: new Queue('email', REDIS_URL),
  reports: new Queue('reports', REDIS_URL),
  bulk: new Queue('bulk-operations', REDIS_URL)
};

// Producer
async function generateBulkFees(classId, feeData) {
  const students = await Student.find({ classId });
  
  // Queue individual fee creation
  const jobs = students.map(s => ({
    name: 'create-fee',
    data: { studentId: s._id, ...feeData }
  }));
  
  await queues.bulk.addBulk(jobs);
  
  return { queued: students.length };
}

// Consumer
queues.bulk.process('create-fee', 10, async (job) => {
  const { studentId, amount, dueDate } = job.data;
  await Fee.create({ studentId, amount, dueDate, status: 'pending' });
});
```

## 🔒 Security Architecture

```
┌─────────────────────────────────────────────────────────┐
│                  Security Layers                         │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  1. Network Security                                     │
│     • WAF (Web Application Firewall)                    │
│     • DDoS protection                                    │
│     • VPC with private subnets                          │
│                                                          │
│  2. Application Security                                 │
│     • JWT authentication                                 │
│     • Role-based access control (RBAC)                  │
│     • Input validation & sanitization                   │
│     • Rate limiting                                      │
│                                                          │
│  3. Data Security                                        │
│     • Encryption at rest (AES-256)                      │
│     • Encryption in transit (TLS 1.3)                   │
│     • PII masking in logs                               │
│     • Database access controls                          │
│                                                          │
│  4. Audit & Compliance                                   │
│     • Comprehensive audit logs                          │
│     • Data retention policies                           │
│     • GDPR/data privacy compliance                      │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### RBAC Implementation

```javascript
// Permission model
const permissions = {
  'student:read': ['admin', 'teacher', 'parent'],
  'student:write': ['admin'],
  'fee:read': ['admin', 'accountant', 'parent'],
  'fee:write': ['admin', 'accountant'],
  'exam:read': ['admin', 'teacher', 'student', 'parent'],
  'exam:write': ['admin', 'teacher']
};

// Middleware
const authorize = (...requiredPermissions) => {
  return (req, res, next) => {
    const userRole = req.user.role;
    
    const hasPermission = requiredPermissions.every(perm => 
      permissions[perm]?.includes(userRole)
    );
    
    if (!hasPermission) {
      return res.status(403).json({ error: 'Forbidden' });
    }
    
    next();
  };
};

// Usage
app.get('/api/students', authorize('student:read'), getStudents);
app.post('/api/students', authorize('student:write'), createStudent);
```

## ❓ Interview Q&A

> [!question]- How would you design a fee payment system for 10,000 concurrent users?
> ```
> Architecture:
> 1. Load balancer distributes traffic
> 2. Stateless API servers (auto-scale)
> 3. Redis for session & rate limiting
> 4. Database with read replicas
> 5. Queue for async processing
> 
> Key considerations:
> • Idempotency keys for payments
> • Database transactions with row locking
> • Optimistic locking for concurrent updates
> • Circuit breaker for payment gateway
> • Retry with exponential backoff
> ```

> [!question]- How do you handle report generation for large datasets?
> ```javascript
> // Async report generation
> async function generateReport(params) {
>   // 1. Create report job
>   const job = await Report.create({
>     type: 'fee-collection',
>     params,
>     status: 'queued'
>   });
>   
>   // 2. Queue for background processing
>   await reportQueue.add({ reportId: job._id });
>   
>   // 3. Return job ID for polling
>   return { jobId: job._id };
> }
> 
> // Worker: Stream data to avoid memory issues
> reportQueue.process(async (job) => {
>   const report = await Report.findById(job.data.reportId);
>   
>   const cursor = Fee.find(report.params).cursor();
>   const writeStream = createWriteStream(`/tmp/${report._id}.csv`);
>   
>   for await (const fee of cursor) {
>     writeStream.write(formatRow(fee));
>   }
>   
>   // Upload to S3
>   const url = await uploadToS3(`/tmp/${report._id}.csv`);
>   await Report.updateOne({ _id: report._id }, { status: 'completed', url });
> });
> ```

> [!question]- How do you ensure data consistency across modules?
> ```
> Strategies:
> 
> 1. Database transactions (same DB)
>    BEGIN → Update Student → Create Fee → COMMIT
> 
> 2. Saga pattern (distributed)
>    Student Created → Event → Fee Service → Create Fee
>                           ↓ (on failure)
>                    Compensating action
> 
> 3. Outbox pattern
>    ┌─────────────────────────────────┐
>    │ Transaction                      │
>    │  • Insert Student               │
>    │  • Insert to Outbox table       │
>    └─────────────────────────────────┘
>              ↓
>    Background job reads Outbox → Publishes event
> ```

> [!question]- How would you implement real-time notifications?
> ```
> ┌─────────┐     ┌─────────┐     ┌─────────┐
> │ Action  │────>│  Queue  │────>│ Worker  │
> │(Fee paid)│     │         │     │         │
> └─────────┘     └─────────┘     └────┬────┘
>                                      │
>                    ┌─────────────────┼─────────────────┐
>                    │                 │                 │
>               ┌────┴────┐      ┌─────┴─────┐     ┌────┴────┐
>               │  Email  │      │    SMS    │     │ WebSocket│
>               │ Service │      │  Service  │     │  Server  │
>               └─────────┘      └───────────┘     └─────────┘
> ```
> 
> ```javascript
> // WebSocket for real-time
> const io = require('socket.io')(server);
> 
> io.on('connection', (socket) => {
>   const userId = socket.handshake.auth.userId;
>   socket.join(`user:${userId}`);
> });
> 
> // Send notification
> function notifyUser(userId, notification) {
>   io.to(`user:${userId}`).emit('notification', notification);
> }
> ```

> [!question]- How do you handle database migrations in production?
> ```
> Strategy: Blue-Green with backward compatibility
> 
> 1. Add new column (nullable)
>    ALTER TABLE students ADD new_field VARCHAR(100);
> 
> 2. Deploy code that writes to both old & new
> 
> 3. Backfill data
>    UPDATE students SET new_field = old_field WHERE new_field IS NULL;
> 
> 4. Deploy code that reads from new
> 
> 5. Remove old column (after verification)
>    ALTER TABLE students DROP COLUMN old_field;
> 
> Tools: Flyway, Liquibase, Laravel migrations
> ```
