---
title: Database Design & Optimization
tags: [mysql, mongodb, postgresql, database]
created: 2026-02-03
---

# Database Design & Optimization

## 🎯 MySQL

### Schema Design for ERP

```sql
-- Students table
CREATE TABLE students (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    school_id BIGINT UNSIGNED NOT NULL,
    admission_no VARCHAR(20) UNIQUE NOT NULL,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE,
    class_id BIGINT UNSIGNED NOT NULL,
    section CHAR(1),
    guardian_phone VARCHAR(15),
    status ENUM('active', 'inactive', 'graduated') DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    INDEX idx_school_class (school_id, class_id),
    INDEX idx_status (status),
    FOREIGN KEY (school_id) REFERENCES schools(id),
    FOREIGN KEY (class_id) REFERENCES classes(id)
) ENGINE=InnoDB;

-- Fees table
CREATE TABLE fees (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    student_id BIGINT UNSIGNED NOT NULL,
    fee_type ENUM('tuition', 'transport', 'exam', 'misc') NOT NULL,
    amount DECIMAL(10,2) NOT NULL,
    due_date DATE NOT NULL,
    paid_date DATE,
    status ENUM('pending', 'paid', 'overdue', 'waived') DEFAULT 'pending',
    payment_method VARCHAR(50),
    transaction_id VARCHAR(100),
    
    INDEX idx_student_status (student_id, status),
    INDEX idx_due_date (due_date),
    FOREIGN KEY (student_id) REFERENCES students(id)
) ENGINE=InnoDB;
```

### Query Optimization

```sql
-- Use EXPLAIN to analyze queries
EXPLAIN SELECT s.name, SUM(f.amount) as total_fees
FROM students s
JOIN fees f ON s.id = f.student_id
WHERE s.school_id = 1 AND f.status = 'pending'
GROUP BY s.id;

-- Optimized with proper indexes
CREATE INDEX idx_fees_student_status ON fees(student_id, status);

-- Avoid SELECT *
-- Bad
SELECT * FROM students WHERE class_id = 5;

-- Good
SELECT id, name, email FROM students WHERE class_id = 5;

-- Use LIMIT for pagination
SELECT id, name FROM students 
WHERE school_id = 1 
ORDER BY created_at DESC 
LIMIT 20 OFFSET 40;

-- Batch inserts
INSERT INTO attendance (student_id, date, status) VALUES
(1, '2026-02-03', 'present'),
(2, '2026-02-03', 'present'),
(3, '2026-02-03', 'absent');
```

### Transactions & Locking

```sql
-- Transaction for fee payment
START TRANSACTION;

UPDATE fees 
SET status = 'paid', paid_date = CURDATE(), transaction_id = 'TXN123'
WHERE id = 100 AND status = 'pending';

INSERT INTO payment_logs (fee_id, amount, payment_date)
SELECT id, amount, CURDATE() FROM fees WHERE id = 100;

COMMIT;

-- Row-level locking
SELECT * FROM fees WHERE id = 100 FOR UPDATE;
```

## 🍃 MongoDB

### Schema Design (Document Model)

```javascript
// Students collection
{
  _id: ObjectId("..."),
  admissionNo: "2026/001",
  name: "John Doe",
  email: "john@example.com",
  class: {
    id: ObjectId("..."),
    name: "Class 10",
    section: "A"
  },
  guardian: {
    name: "Jane Doe",
    phone: "9876543210",
    relation: "Mother"
  },
  fees: [
    {
      type: "tuition",
      amount: 5000,
      dueDate: ISODate("2026-03-01"),
      status: "pending"
    }
  ],
  attendance: {
    "2026-02": { present: 20, absent: 2, total: 22 }
  },
  createdAt: ISODate("2026-01-15"),
  updatedAt: ISODate("2026-02-03")
}

// Exams collection (separate for complex queries)
{
  _id: ObjectId("..."),
  name: "Mid-Term 2026",
  class: ObjectId("..."),
  subjects: [
    {
      name: "Mathematics",
      maxMarks: 100,
      passingMarks: 35
    }
  ],
  results: [
    {
      studentId: ObjectId("..."),
      marks: [
        { subject: "Mathematics", obtained: 85 }
      ],
      totalMarks: 450,
      percentage: 90,
      grade: "A+"
    }
  ]
}
```

### Aggregation Pipeline

```javascript
// Fee collection report
db.students.aggregate([
  { $match: { "class.name": "Class 10" } },
  { $unwind: "$fees" },
  { $match: { "fees.status": "pending" } },
  { $group: {
      _id: "$class.section",
      totalPending: { $sum: "$fees.amount" },
      studentCount: { $addToSet: "$_id" }
  }},
  { $project: {
      section: "$_id",
      totalPending: 1,
      studentCount: { $size: "$studentCount" }
  }},
  { $sort: { totalPending: -1 } }
]);

// Attendance report
db.students.aggregate([
  { $match: { "attendance.2026-02": { $exists: true } } },
  { $project: {
      name: 1,
      attendancePercent: {
        $multiply: [
          { $divide: ["$attendance.2026-02.present", "$attendance.2026-02.total"] },
          100
        ]
      }
  }},
  { $match: { attendancePercent: { $lt: 75 } } }
]);
```

### Indexing Strategies

```javascript
// Single field index
db.students.createIndex({ admissionNo: 1 }, { unique: true });

// Compound index
db.students.createIndex({ "class.id": 1, status: 1 });

// Text index for search
db.students.createIndex({ name: "text", email: "text" });

// TTL index for logs
db.auditLogs.createIndex({ createdAt: 1 }, { expireAfterSeconds: 7776000 }); // 90 days

// Partial index
db.fees.createIndex(
  { dueDate: 1 },
  { partialFilterExpression: { status: "pending" } }
);
```

## 🐘 PostgreSQL

### Advanced Features

```sql
-- JSON/JSONB columns
CREATE TABLE students (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    metadata JSONB DEFAULT '{}',
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Query JSONB
SELECT name, metadata->>'guardian_name' as guardian
FROM students
WHERE metadata @> '{"category": "scholarship"}';

-- GIN index for JSONB
CREATE INDEX idx_metadata ON students USING GIN (metadata);

-- Array columns
CREATE TABLE classes (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50),
    subject_ids INTEGER[]
);

-- Query arrays
SELECT * FROM classes WHERE 5 = ANY(subject_ids);

-- CTE (Common Table Expression)
WITH pending_fees AS (
    SELECT student_id, SUM(amount) as total
    FROM fees
    WHERE status = 'pending'
    GROUP BY student_id
)
SELECT s.name, pf.total
FROM students s
JOIN pending_fees pf ON s.id = pf.student_id
WHERE pf.total > 10000;

-- Window functions
SELECT 
    name,
    class_id,
    total_marks,
    RANK() OVER (PARTITION BY class_id ORDER BY total_marks DESC) as rank
FROM exam_results;
```

## 📊 Comparison Table

| Feature | MySQL | MongoDB | PostgreSQL |
|---------|-------|---------|------------|
| **Type** | Relational | Document | Relational |
| **Schema** | Fixed | Flexible | Fixed + JSONB |
| **Joins** | Yes | $lookup | Yes |
| **Transactions** | ACID | ACID (4.0+) | ACID |
| **Scaling** | Vertical | Horizontal | Vertical |
| **Best For** | Structured data | Flexible schemas | Complex queries |
| **ERP Use** | Core data | Logs, analytics | Advanced reporting |

## ❓ Interview Q&A

> [!question]- What is database normalization?
> Normalization organizes data to reduce redundancy:
> - **1NF**: Atomic values, no repeating groups
> - **2NF**: 1NF + no partial dependencies
> - **3NF**: 2NF + no transitive dependencies
> - **BCNF**: Every determinant is a candidate key
> 
> For ERP, typically normalize to 3NF but denormalize for reporting tables.

> [!question]- How do you optimize slow queries?
> 1. **Analyze**: Use `EXPLAIN` / `EXPLAIN ANALYZE`
> 2. **Index**: Add indexes on WHERE, JOIN, ORDER BY columns
> 3. **Query**: Avoid SELECT *, use LIMIT, avoid functions on indexed columns
> 4. **Schema**: Denormalize for read-heavy tables
> 5. **Cache**: Use Redis/Memcached for frequent queries
> 6. **Partition**: Split large tables by date/range

> [!question]- When to use MongoDB vs MySQL?
> **Use MongoDB when:**
> - Schema changes frequently
> - Hierarchical/nested data (student profiles)
> - Horizontal scaling needed
> - Logs, analytics, real-time data
> 
> **Use MySQL when:**
> - Strong relationships (fees, transactions)
> - ACID compliance critical
> - Complex joins needed
> - Reporting requirements

> [!question]- Explain database indexing
> ```
> Without Index:        With Index (B-Tree):
> ┌─────────────┐       ┌─────────────┐
> │ Full Table  │       │   Root      │
> │   Scan      │       │  ┌──┴──┐    │
> │ O(n)        │       │ Node  Node  │
> └─────────────┘       │ ┌┴┐  ┌┴┐    │
>                       │ L L  L L    │ O(log n)
>                       └─────────────┘
> ```
> - **B-Tree**: Default, good for range queries
> - **Hash**: Exact matches only
> - **GIN**: Full-text, JSONB, arrays
> - **Composite**: Multiple columns (order matters)

> [!question]- How do you handle database migrations in production?
> 1. **Version control**: All migrations in Git
> 2. **Backward compatible**: Add columns nullable first
> 3. **Blue-green**: Run migrations before switching
> 4. **Rollback plan**: Always have down migration
> 5. **Test**: Run on staging with production data copy
> ```php
> // Laravel migration
> public function up() {
>     Schema::table('students', function (Blueprint $table) {
>         $table->string('new_field')->nullable(); // Safe
>     });
> }
> 
> public function down() {
>     Schema::table('students', function (Blueprint $table) {
>         $table->dropColumn('new_field');
>     });
> }
> ```

> [!question]- What is sharding and when to use it?
> Sharding distributes data across multiple databases:
> ```
> ┌─────────────────────────────────────┐
> │           Application               │
> └──────────────┬──────────────────────┘
>                │ Shard Key (school_id)
>     ┌──────────┼──────────┐
>     ▼          ▼          ▼
> ┌───────┐ ┌───────┐ ┌───────┐
> │Shard 1│ │Shard 2│ │Shard 3│
> │School │ │School │ │School │
> │ 1-100 │ │101-200│ │201-300│
> └───────┘ └───────┘ └───────┘
> ```
> Use when: Single DB can't handle load, data isolation needed (multi-tenant ERP)

> [!question]- How do you implement soft deletes?
> ```sql
> -- MySQL
> ALTER TABLE students ADD deleted_at TIMESTAMP NULL;
> CREATE INDEX idx_deleted ON students(deleted_at);
> 
> -- Soft delete
> UPDATE students SET deleted_at = NOW() WHERE id = 1;
> 
> -- Query active only
> SELECT * FROM students WHERE deleted_at IS NULL;
> 
> -- Query with deleted
> SELECT * FROM students; -- All
> SELECT * FROM students WHERE deleted_at IS NOT NULL; -- Only deleted
> ```

## 🎯 ERP Database Patterns

### Multi-Tenant Architecture

```sql
-- Option 1: Shared database, tenant column
CREATE TABLE students (
    id BIGINT PRIMARY KEY,
    school_id BIGINT NOT NULL, -- Tenant identifier
    name VARCHAR(100),
    INDEX idx_school (school_id)
);

-- Option 2: Schema per tenant (PostgreSQL)
CREATE SCHEMA school_123;
CREATE TABLE school_123.students (...);

-- Option 3: Database per tenant
-- Separate database for each school
```

### Audit Trail Table

```sql
CREATE TABLE audit_logs (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    table_name VARCHAR(50) NOT NULL,
    record_id BIGINT NOT NULL,
    action ENUM('INSERT', 'UPDATE', 'DELETE') NOT NULL,
    old_values JSON,
    new_values JSON,
    user_id BIGINT,
    ip_address VARCHAR(45),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    INDEX idx_table_record (table_name, record_id),
    INDEX idx_created (created_at)
);
```

### Fee Ledger Pattern

```sql
-- Double-entry bookkeeping for finance module
CREATE TABLE ledger_entries (
    id BIGINT PRIMARY KEY,
    transaction_id VARCHAR(50) NOT NULL,
    account_id BIGINT NOT NULL,
    entry_type ENUM('debit', 'credit') NOT NULL,
    amount DECIMAL(12,2) NOT NULL,
    balance_after DECIMAL(12,2) NOT NULL,
    description TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    INDEX idx_account_date (account_id, created_at)
);

-- Ensure debit = credit for each transaction
SELECT transaction_id,
    SUM(CASE WHEN entry_type = 'debit' THEN amount ELSE 0 END) as debits,
    SUM(CASE WHEN entry_type = 'credit' THEN amount ELSE 0 END) as credits
FROM ledger_entries
GROUP BY transaction_id
HAVING debits != credits; -- Should return empty
```
