---
title: Node.js Backend Development
tags: [nodejs, express, backend, async]
created: 2026-02-03
---

# Node.js Backend Development

## 🎯 Core Concepts

### Event Loop Architecture

```
   ┌───────────────────────────┐
┌─>│         timers            │ setTimeout, setInterval
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │ I/O callbacks
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │ internal use
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │          poll             │ retrieve new I/O events
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │          check            │ setImmediate
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
└──┤      close callbacks      │ socket.on('close')
   └───────────────────────────┘
```

### Async Patterns

```javascript
// 1. Callbacks (legacy)
fs.readFile('file.txt', (err, data) => {
  if (err) throw err;
  console.log(data);
});

// 2. Promises
fs.promises.readFile('file.txt')
  .then(data => console.log(data))
  .catch(err => console.error(err));

// 3. Async/Await (preferred)
async function readFile() {
  try {
    const data = await fs.promises.readFile('file.txt');
    return data;
  } catch (err) {
    throw err;
  }
}
```

### Streams

```javascript
const http = require('node:http');

const server = http.createServer((req, res) => {
  let body = '';
  req.setEncoding('utf8');
  
  req.on('data', (chunk) => { body += chunk; });
  
  req.on('end', () => {
    try {
      const data = JSON.parse(body);
      res.write(typeof data);
      res.end();
    } catch (er) {
      res.statusCode = 400;
      res.end(`error: ${er.message}`);
    }
  });
});
```

## 🚀 Express.js Essentials

### Basic Setup

```javascript
const express = require('express');
const app = express();

// Middleware
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Routes
app.get('/api/users', async (req, res) => {
  const users = await User.find();
  res.json(users);
});

// Error handling middleware
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: 'Something went wrong!' });
});

app.listen(3000);
```

### Middleware Pattern

```javascript
// Authentication middleware
const authMiddleware = async (req, res, next) => {
  try {
    const token = req.headers.authorization?.split(' ')[1];
    if (!token) throw new Error('No token');
    
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (err) {
    res.status(401).json({ error: 'Unauthorized' });
  }
};

// Usage
app.get('/api/protected', authMiddleware, (req, res) => {
  res.json({ user: req.user });
});
```

### Route Organization (ERP Pattern)

```javascript
// routes/index.js
const express = require('express');
const router = express.Router();

router.use('/hr', require('./hr.routes'));
router.use('/exams', require('./exam.routes'));
router.use('/fees', require('./fee.routes'));
router.use('/admissions', require('./admission.routes'));

module.exports = router;
```

## 📦 Common Packages for ERP

| Package | Purpose |
|---------|---------|
| `express` | Web framework |
| `mongoose` | MongoDB ODM |
| `sequelize` | SQL ORM |
| `jsonwebtoken` | JWT auth |
| `bcrypt` | Password hashing |
| `multer` | File uploads |
| `nodemailer` | Email sending |
| `bull` | Job queues |
| `winston` | Logging |
| `joi` / `zod` | Validation |

## 🔒 Security Best Practices

```javascript
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');
const cors = require('cors');

// Security headers
app.use(helmet());

// CORS
app.use(cors({
  origin: process.env.ALLOWED_ORIGINS.split(','),
  credentials: true
}));

// Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100 // limit each IP to 100 requests per windowMs
});
app.use('/api/', limiter);

// Input sanitization
const mongoSanitize = require('express-mongo-sanitize');
app.use(mongoSanitize());
```

## ❓ Interview Q&A

> [!question]- What is the Event Loop in Node.js?
> The event loop is what allows Node.js to perform non-blocking I/O operations despite JavaScript being single-threaded. It offloads operations to the system kernel whenever possible and uses a queue-based system to handle callbacks when operations complete.

> [!question]- Explain the difference between `process.nextTick()` and `setImmediate()`
> - `process.nextTick()`: Executes immediately after the current operation, before the event loop continues
> - `setImmediate()`: Executes in the check phase of the event loop, after I/O events
> - `nextTick` has higher priority and can starve I/O if overused

> [!question]- How do you handle errors in async/await?
> ```javascript
> // Method 1: try-catch
> async function getData() {
>   try {
>     const result = await fetchData();
>     return result;
>   } catch (err) {
>     logger.error(err);
>     throw new CustomError('Failed to fetch data');
>   }
> }
> 
> // Method 2: Error handling middleware
> const asyncHandler = fn => (req, res, next) =>
>   Promise.resolve(fn(req, res, next)).catch(next);
> ```

> [!question]- How would you implement caching in Node.js?
> ```javascript
> const Redis = require('ioredis');
> const redis = new Redis();
> 
> const cacheMiddleware = (duration) => async (req, res, next) => {
>   const key = `cache:${req.originalUrl}`;
>   const cached = await redis.get(key);
>   
>   if (cached) {
>     return res.json(JSON.parse(cached));
>   }
>   
>   res.sendResponse = res.json;
>   res.json = (body) => {
>     redis.setex(key, duration, JSON.stringify(body));
>     res.sendResponse(body);
>   };
>   next();
> };
> ```

> [!question]- How do you handle file uploads in an ERP system?
> ```javascript
> const multer = require('multer');
> const path = require('path');
> 
> const storage = multer.diskStorage({
>   destination: (req, file, cb) => {
>     cb(null, 'uploads/');
>   },
>   filename: (req, file, cb) => {
>     const uniqueSuffix = Date.now() + '-' + Math.round(Math.random() * 1E9);
>     cb(null, file.fieldname + '-' + uniqueSuffix + path.extname(file.originalname));
>   }
> });
> 
> const upload = multer({
>   storage,
>   limits: { fileSize: 5 * 1024 * 1024 }, // 5MB
>   fileFilter: (req, file, cb) => {
>     const allowed = ['.pdf', '.doc', '.docx', '.jpg', '.png'];
>     const ext = path.extname(file.originalname).toLowerCase();
>     cb(null, allowed.includes(ext));
>   }
> });
> ```

> [!question]- How do you implement job queues for background tasks?
> ```javascript
> const Queue = require('bull');
> const emailQueue = new Queue('email', process.env.REDIS_URL);
> 
> // Producer
> await emailQueue.add('welcome-email', {
>   to: user.email,
>   template: 'welcome',
>   data: { name: user.name }
> }, {
>   attempts: 3,
>   backoff: { type: 'exponential', delay: 2000 }
> });
> 
> // Consumer
> emailQueue.process('welcome-email', async (job) => {
>   await sendEmail(job.data);
> });
> ```

## 🎯 ERP-Specific Patterns

### Service Layer Pattern

```javascript
// services/fee.service.js
class FeeService {
  async calculateFee(studentId, academicYear) {
    const student = await Student.findById(studentId);
    const feeStructure = await FeeStructure.findOne({
      class: student.class,
      academicYear
    });
    
    const discounts = await this.calculateDiscounts(student);
    const totalFee = feeStructure.amount - discounts;
    
    return { feeStructure, discounts, totalFee };
  }
  
  async generateInvoice(studentId, feeData) {
    const invoice = await Invoice.create({
      student: studentId,
      ...feeData,
      invoiceNumber: await this.generateInvoiceNumber()
    });
    
    // Queue email notification
    await emailQueue.add('fee-invoice', { invoiceId: invoice._id });
    
    return invoice;
  }
}
```

### Transaction Handling

```javascript
const mongoose = require('mongoose');

async function transferFunds(fromAccount, toAccount, amount) {
  const session = await mongoose.startSession();
  session.startTransaction();
  
  try {
    await Account.updateOne(
      { _id: fromAccount },
      { $inc: { balance: -amount } },
      { session }
    );
    
    await Account.updateOne(
      { _id: toAccount },
      { $inc: { balance: amount } },
      { session }
    );
    
    await session.commitTransaction();
  } catch (error) {
    await session.abortTransaction();
    throw error;
  } finally {
    session.endSession();
  }
}
```
