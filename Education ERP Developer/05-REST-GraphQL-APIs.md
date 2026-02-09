---
title: REST & GraphQL APIs
tags: [api, rest, graphql, integration]
created: 2026-02-03
---

# REST & GraphQL APIs

## 🎯 REST API Design

### RESTful Principles

```
┌─────────────────────────────────────────────────────────┐
│                    REST Constraints                      │
├─────────────────────────────────────────────────────────┤
│ 1. Client-Server    │ Separation of concerns            │
│ 2. Stateless        │ No session state on server        │
│ 3. Cacheable        │ Responses must define cacheability│
│ 4. Uniform Interface│ Standard HTTP methods             │
│ 5. Layered System   │ Client can't tell if direct       │
│ 6. Code on Demand   │ Optional executable code          │
└─────────────────────────────────────────────────────────┘
```

### HTTP Methods & Status Codes

| Method | Purpose | Idempotent | Safe |
|--------|---------|------------|------|
| GET | Retrieve resource | Yes | Yes |
| POST | Create resource | No | No |
| PUT | Replace resource | Yes | No |
| PATCH | Partial update | No | No |
| DELETE | Remove resource | Yes | No |

| Status | Meaning | Use Case |
|--------|---------|----------|
| 200 | OK | Successful GET/PUT/PATCH |
| 201 | Created | Successful POST |
| 204 | No Content | Successful DELETE |
| 400 | Bad Request | Validation error |
| 401 | Unauthorized | Missing/invalid auth |
| 403 | Forbidden | No permission |
| 404 | Not Found | Resource doesn't exist |
| 422 | Unprocessable | Semantic errors |
| 500 | Server Error | Unexpected error |

### API Endpoint Design

```
# Good REST endpoints for Education ERP

# Students
GET    /api/v1/students              # List all
GET    /api/v1/students/:id          # Get one
POST   /api/v1/students              # Create
PUT    /api/v1/students/:id          # Replace
PATCH  /api/v1/students/:id          # Update
DELETE /api/v1/students/:id          # Delete

# Nested resources
GET    /api/v1/students/:id/fees     # Student's fees
POST   /api/v1/students/:id/fees     # Add fee to student
GET    /api/v1/classes/:id/students  # Students in class

# Filtering, sorting, pagination
GET    /api/v1/students?class_id=5&status=active
GET    /api/v1/students?sort=-created_at
GET    /api/v1/students?page=2&per_page=20

# Actions (when CRUD doesn't fit)
POST   /api/v1/fees/:id/pay          # Pay a fee
POST   /api/v1/students/:id/promote  # Promote student
```

### Request/Response Format

```javascript
// POST /api/v1/students
// Request
{
  "name": "John Doe",
  "email": "john@example.com",
  "class_id": 5,
  "guardian": {
    "name": "Jane Doe",
    "phone": "9876543210"
  }
}

// Response (201 Created)
{
  "data": {
    "id": 123,
    "name": "John Doe",
    "email": "john@example.com",
    "class": {
      "id": 5,
      "name": "Class 10-A"
    },
    "created_at": "2026-02-03T10:30:00Z"
  }
}

// Error Response (422)
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed",
    "details": [
      { "field": "email", "message": "Email already exists" },
      { "field": "class_id", "message": "Class not found" }
    ]
  }
}

// Paginated Response
{
  "data": [...],
  "meta": {
    "current_page": 2,
    "per_page": 20,
    "total": 150,
    "total_pages": 8
  },
  "links": {
    "first": "/api/v1/students?page=1",
    "prev": "/api/v1/students?page=1",
    "next": "/api/v1/students?page=3",
    "last": "/api/v1/students?page=8"
  }
}
```

### Authentication

```javascript
// JWT Authentication
// Request header
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

// Token structure
{
  "header": { "alg": "HS256", "typ": "JWT" },
  "payload": {
    "sub": "user_123",
    "role": "admin",
    "school_id": 1,
    "exp": 1738600000
  },
  "signature": "..."
}

// Refresh token flow
POST /api/v1/auth/login
// Response: { access_token, refresh_token }

POST /api/v1/auth/refresh
// Request: { refresh_token }
// Response: { access_token }
```

## 📊 GraphQL

### Schema Definition

```graphql
# Types
type Student {
  id: ID!
  name: String!
  email: String
  class: Class!
  fees: [Fee!]!
  createdAt: DateTime!
}

type Class {
  id: ID!
  name: String!
  section: String
  students: [Student!]!
}

type Fee {
  id: ID!
  amount: Float!
  type: FeeType!
  status: FeeStatus!
  dueDate: Date!
}

enum FeeType {
  TUITION
  TRANSPORT
  EXAM
  MISC
}

enum FeeStatus {
  PENDING
  PAID
  OVERDUE
  WAIVED
}

# Queries
type Query {
  student(id: ID!): Student
  students(classId: ID, status: String, page: Int, limit: Int): StudentConnection!
  classes: [Class!]!
}

# Mutations
type Mutation {
  createStudent(input: CreateStudentInput!): Student!
  updateStudent(id: ID!, input: UpdateStudentInput!): Student!
  deleteStudent(id: ID!): Boolean!
  payFee(feeId: ID!, paymentMethod: String!): Fee!
}

# Input types
input CreateStudentInput {
  name: String!
  email: String
  classId: ID!
  guardianName: String
  guardianPhone: String
}

# Pagination
type StudentConnection {
  edges: [StudentEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type StudentEdge {
  node: Student!
  cursor: String!
}

type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}
```

### Resolvers (Node.js)

```javascript
const resolvers = {
  Query: {
    student: async (_, { id }, { dataSources }) => {
      return dataSources.studentAPI.getById(id);
    },
    
    students: async (_, { classId, status, page = 1, limit = 20 }, { dataSources }) => {
      return dataSources.studentAPI.getAll({ classId, status, page, limit });
    }
  },
  
  Mutation: {
    createStudent: async (_, { input }, { dataSources, user }) => {
      if (!user.can('create:student')) {
        throw new ForbiddenError('Not authorized');
      }
      return dataSources.studentAPI.create(input);
    },
    
    payFee: async (_, { feeId, paymentMethod }, { dataSources }) => {
      return dataSources.feeAPI.processPayment(feeId, paymentMethod);
    }
  },
  
  // Field resolvers
  Student: {
    class: async (student, _, { dataSources }) => {
      return dataSources.classAPI.getById(student.classId);
    },
    
    fees: async (student, _, { dataSources }) => {
      return dataSources.feeAPI.getByStudentId(student.id);
    }
  }
};
```

### GraphQL Queries

```graphql
# Get student with related data (single request)
query GetStudentDetails($id: ID!) {
  student(id: $id) {
    id
    name
    email
    class {
      name
      section
    }
    fees(status: PENDING) {
      amount
      dueDate
      type
    }
  }
}

# Mutation with variables
mutation CreateStudent($input: CreateStudentInput!) {
  createStudent(input: $input) {
    id
    name
    class {
      name
    }
  }
}

# Variables
{
  "input": {
    "name": "John Doe",
    "email": "john@example.com",
    "classId": "5"
  }
}
```

## 🔌 Third-Party Integrations

### Payment Gateway (Razorpay Example)

```javascript
// Create order
const Razorpay = require('razorpay');
const razorpay = new Razorpay({
  key_id: process.env.RAZORPAY_KEY,
  key_secret: process.env.RAZORPAY_SECRET
});

async function createPaymentOrder(fee) {
  const order = await razorpay.orders.create({
    amount: fee.amount * 100, // in paise
    currency: 'INR',
    receipt: `fee_${fee.id}`,
    notes: {
      student_id: fee.studentId,
      fee_type: fee.type
    }
  });
  return order;
}

// Verify payment (webhook)
app.post('/webhooks/razorpay', (req, res) => {
  const signature = req.headers['x-razorpay-signature'];
  const isValid = Razorpay.validateWebhookSignature(
    JSON.stringify(req.body),
    signature,
    process.env.RAZORPAY_WEBHOOK_SECRET
  );
  
  if (isValid && req.body.event === 'payment.captured') {
    const { order_id, id: payment_id } = req.body.payload.payment.entity;
    updateFeeStatus(order_id, payment_id);
  }
  
  res.status(200).json({ status: 'ok' });
});
```

### SMS Integration (Twilio/MSG91)

```javascript
// SMS service
class SMSService {
  async sendFeeReminder(student, fee) {
    const message = `Dear Parent, fee of ₹${fee.amount} for ${student.name} is due on ${fee.dueDate}. Please pay to avoid late fee.`;
    
    await this.send(student.guardianPhone, message);
  }
  
  async send(phone, message) {
    // MSG91 example
    const response = await fetch('https://api.msg91.com/api/v5/flow/', {
      method: 'POST',
      headers: {
        'authkey': process.env.MSG91_AUTH_KEY,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        template_id: process.env.MSG91_TEMPLATE_ID,
        recipients: [{ mobiles: `91${phone}`, message }]
      })
    });
    return response.json();
  }
}
```

### Email Integration (Nodemailer)

```javascript
const nodemailer = require('nodemailer');

const transporter = nodemailer.createTransport({
  host: process.env.SMTP_HOST,
  port: 587,
  auth: {
    user: process.env.SMTP_USER,
    pass: process.env.SMTP_PASS
  }
});

async function sendFeeInvoice(student, fee, invoicePdf) {
  await transporter.sendMail({
    from: '"School ERP" <noreply@school.com>',
    to: student.email,
    subject: `Fee Invoice - ${fee.invoiceNumber}`,
    html: `
      <h2>Fee Invoice</h2>
      <p>Dear ${student.name},</p>
      <p>Please find attached your fee invoice.</p>
      <p>Amount: ₹${fee.amount}</p>
      <p>Due Date: ${fee.dueDate}</p>
    `,
    attachments: [{
      filename: `invoice_${fee.invoiceNumber}.pdf`,
      content: invoicePdf
    }]
  });
}
```

## ❓ Interview Q&A

> [!question]- REST vs GraphQL - When to use which?
> | Aspect | REST | GraphQL |
> |--------|------|---------|
> | **Use when** | Simple CRUD, caching important | Complex nested data, mobile apps |
> | **Over-fetching** | Common | Solved (request what you need) |
> | **Under-fetching** | Multiple requests needed | Single request |
> | **Caching** | HTTP caching built-in | Requires custom solution |
> | **Learning curve** | Lower | Higher |
> | **File uploads** | Native support | Requires workarounds |

> [!question]- How do you handle API versioning?
> ```
> 1. URL versioning (recommended)
>    /api/v1/students
>    /api/v2/students
> 
> 2. Header versioning
>    Accept: application/vnd.api+json; version=1
> 
> 3. Query parameter
>    /api/students?version=1
> ```
> URL versioning is most common and cache-friendly.

> [!question]- How do you secure APIs?
> 1. **Authentication**: JWT, OAuth 2.0
> 2. **Authorization**: Role-based access control (RBAC)
> 3. **Rate limiting**: Prevent abuse
> 4. **Input validation**: Sanitize all inputs
> 5. **HTTPS**: Encrypt in transit
> 6. **CORS**: Restrict origins
> 7. **API keys**: For third-party access

> [!question]- What is the N+1 problem in GraphQL?
> When resolving nested fields, each parent triggers a separate query for children.
> ```javascript
> // Problem: 1 query for students + N queries for classes
> students {
>   name
>   class { name }  // Separate query per student
> }
> 
> // Solution: DataLoader (batching)
> const classLoader = new DataLoader(async (classIds) => {
>   const classes = await Class.find({ _id: { $in: classIds } });
>   return classIds.map(id => classes.find(c => c.id === id));
> });
> ```

> [!question]- How do you handle API rate limiting?
> ```javascript
> const rateLimit = require('express-rate-limit');
> 
> // Global limiter
> const limiter = rateLimit({
>   windowMs: 15 * 60 * 1000, // 15 minutes
>   max: 100, // 100 requests per window
>   message: { error: 'Too many requests' },
>   standardHeaders: true,
>   legacyHeaders: false
> });
> 
> // Stricter for auth endpoints
> const authLimiter = rateLimit({
>   windowMs: 60 * 60 * 1000, // 1 hour
>   max: 5, // 5 login attempts
>   skipSuccessfulRequests: true
> });
> 
> app.use('/api/', limiter);
> app.use('/api/auth/login', authLimiter);
> ```

> [!question]- How do you handle webhook security?
> ```javascript
> // 1. Verify signature
> function verifyWebhook(payload, signature, secret) {
>   const expected = crypto
>     .createHmac('sha256', secret)
>     .update(payload)
>     .digest('hex');
>   return crypto.timingSafeEqual(
>     Buffer.from(signature),
>     Buffer.from(expected)
>   );
> }
> 
> // 2. Idempotency - store processed webhook IDs
> // 3. Timeout handling - respond quickly, process async
> // 4. Retry logic - handle duplicate deliveries
> ```

## 🎯 ERP API Patterns

### Bulk Operations

```javascript
// Bulk fee generation
app.post('/api/v1/fees/bulk', async (req, res) => {
  const { classId, feeType, amount, dueDate } = req.body;
  
  const students = await Student.find({ classId, status: 'active' });
  
  const fees = students.map(s => ({
    studentId: s._id,
    type: feeType,
    amount,
    dueDate,
    status: 'pending'
  }));
  
  const result = await Fee.insertMany(fees);
  
  // Queue notifications
  for (const fee of result) {
    await notificationQueue.add('fee-created', { feeId: fee._id });
  }
  
  res.status(201).json({
    message: `Created ${result.length} fee records`,
    count: result.length
  });
});
```

### Report Generation API

```javascript
// Async report generation
app.post('/api/v1/reports/fee-collection', async (req, res) => {
  const { startDate, endDate, classId } = req.body;
  
  // Create report job
  const job = await Report.create({
    type: 'fee-collection',
    params: { startDate, endDate, classId },
    status: 'processing',
    requestedBy: req.user.id
  });
  
  // Queue for background processing
  await reportQueue.add('generate', { reportId: job._id });
  
  res.status(202).json({
    message: 'Report generation started',
    reportId: job._id,
    statusUrl: `/api/v1/reports/${job._id}/status`
  });
});

// Check status
app.get('/api/v1/reports/:id/status', async (req, res) => {
  const report = await Report.findById(req.params.id);
  res.json({
    status: report.status,
    downloadUrl: report.status === 'completed' ? report.fileUrl : null
  });
});
```
