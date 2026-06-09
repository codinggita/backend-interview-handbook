# Node.js Interview Handbook — Part 3 (Q131–Q190)

> Final part of the backend interview handbook. Covers Express advanced concepts, Authentication, Security, MongoDB deep dive, and more.

---

### 121. What is Role-Based Access Control (RBAC)?

## Definition

RBAC is an authorization model where access to resources is based on the roles assigned to users rather than individual permissions per user.

---

## How RBAC Works

```text
User Has A Role
      ↓

Role Has Permissions
      ↓

Access Granted Based On Role
```

---

## Common Roles

```text
admin    → Full Access
moderator → Limited Management
user     → Basic Access
guest    → Read Only
```

---

## Example Schema

```js
const userSchema = new Schema({
  name: String,
  email: String,
  role: {
    type: String,
    enum: ["admin", "user", "moderator"],
    default: "user"
  }
});
```

---

## RBAC Middleware

```js
const authorize = (...roles) => {
  return (req, res, next) => {
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({
        message: "Access Denied"
      });
    }
    next();
  };
};

// Usage
app.delete(
  "/users/:id",
  authenticate,
  authorize("admin"),
  deleteUser
);
```

---

## Benefits

### Easy to Manage

Add or remove access by changing a role.

### Scalable

New permissions don't require user-level changes.

### Secure

Principle of Least Privilege.

---

## Interview Answer (Ready to Speak)

RBAC is an authorization approach where users are assigned roles and each role has specific permissions. Instead of managing permissions per user, access is controlled at the role level. It is scalable, easy to manage, and follows the security principle of least privilege.

---

## Quick Revision

```text
Role Determines Access
admin, user, guest
Middleware Check
403 If Unauthorized
Least Privilege Principle
```

---

### 122. What is Access Token?

## Definition

An Access Token is a short-lived token issued after successful authentication that grants access to protected resources.

---

## Purpose

```text
After Login
     ↓

Server Issues Access Token
     ↓

Client Sends Token With Each Request
     ↓

Server Validates Token
     ↓

Access Granted
```

---

## Characteristics

### Short-Lived

Typically expires in 15 minutes to 1 hour.

---

### Stateless

No server-side storage needed.

---

### Used In Authorization Header

```http
Authorization: Bearer <access_token>
```

---

## Example

```js
const accessToken = jwt.sign(
  { id: user._id, role: user.role },
  process.env.JWT_SECRET,
  { expiresIn: "15m" }
);
```

---

## Benefits

### Fast Verification

No database lookup needed.

### Stateless

Works across multiple servers.

---

## Limitation

### Cannot Be Invalidated Before Expiry

Unless using a token blacklist.

---

## Interview Answer (Ready to Speak)

An access token is a short-lived JWT issued after authentication that allows users to access protected API routes. It is sent in the Authorization header and verified by the server. Short expiry times limit damage if the token is stolen.

---

## Quick Revision

```text
Short-Lived
Authorization Header
JWT Format
15 Minutes Typical
Cannot Be Revoked Early
```

---

### 123. What is Refresh Token?

## Definition

A Refresh Token is a long-lived token used to obtain a new access token when the access token expires, without requiring the user to log in again.

---

## Why Do We Need Refresh Tokens?

Short-lived access tokens expire quickly.

Without refresh tokens:

```text
User Must Log In Every 15 Minutes
```

With refresh tokens:

```text
Get New Access Token Automatically
User Stays Logged In
```

---

## How It Works

```text
Login
  ↓

Issue Access Token (15m) + Refresh Token (7d)
  ↓

Access Token Expires
  ↓

Client Sends Refresh Token
  ↓

Server Issues New Access Token
  ↓

User Continues Without Re-Login
```

---

## Storing Refresh Tokens

### In Database

Can be invalidated (logout).

---

### In HTTP-Only Cookie

Safer than localStorage.

---

## Access Token vs Refresh Token

| Feature | Access Token | Refresh Token |
|---------|-------------|---------------|
| Expiry | Short (15m-1h) | Long (7-30 days) |
| Stored On Server | No | Yes (optional) |
| Used For | API Access | Getting New Access Token |
| Sent In Header | Yes | No (Cookie) |

---

## Example

```js
// Generate Refresh Token
const refreshToken = jwt.sign(
  { id: user._id },
  process.env.REFRESH_SECRET,
  { expiresIn: "7d" }
);

// Store in database
await Token.create({
  user: user._id,
  token: refreshToken
});

// Set as HTTP-only cookie
res.cookie("refreshToken", refreshToken, {
  httpOnly: true,
  secure: true,
  maxAge: 7 * 24 * 60 * 60 * 1000
});
```

---

## Interview Answer (Ready to Speak)

A refresh token is a long-lived token stored securely on the client that allows obtaining a new access token after expiry. It enables users to stay logged in without re-authenticating frequently. Refresh tokens can be invalidated on logout by removing them from the database.

---

## Quick Revision

```text
Long-Lived Token
Gets New Access Token
HTTP-Only Cookie
Stored In Database
Can Be Revoked
```

---

### 124. What is JWT Authentication Middleware?

## Definition

JWT authentication middleware is a function that verifies the JWT token sent by the client and attaches the decoded user data to the request object.

---

## Purpose

Every protected route needs to verify:

```text
Is this request authenticated?
Who is making this request?
```

The middleware handles this automatically.

---

## Example

```js
const jwt = require("jsonwebtoken");

const authenticate = (req, res, next) => {
  const authHeader = req.headers.authorization;

  if (!authHeader || !authHeader.startsWith("Bearer ")) {
    return res.status(401).json({
      message: "No Token Provided"
    });
  }

  const token = authHeader.split(" ")[1];

  try {
    const decoded = jwt.verify(
      token,
      process.env.JWT_SECRET
    );

    req.user = decoded;
    next();

  } catch (err) {
    return res.status(401).json({
      message: "Invalid Token"
    });
  }
};
```

---

## Using The Middleware

```js
// Protect specific routes
app.get("/profile", authenticate, getProfile);

// Protect all routes in a router
router.use(authenticate);
```

---

## Internal Flow

```text
Request Arrives
      ↓

Extract Token From Header
      ↓

jwt.verify()
      ↓

Valid: req.user = decoded
      ↓

next() → Route Handler
      ↓

Invalid: 401 Response
```

---

## Benefits

### Reusable

Apply to any route.

### Centralized

One function handles all verification.

### Clean

Routes stay focused on business logic.

---

## Interview Answer (Ready to Speak)

JWT authentication middleware extracts the token from the Authorization header, verifies it using the secret key, and attaches the decoded payload to req.user. Protected routes apply this middleware to ensure only authenticated users can access them.

---

## Quick Revision

```text
Verify JWT Token
Extract From Authorization Header
req.user = decoded
Protect Routes
401 If Invalid
```

---

### 125. What is Password Hashing Best Practice?

## Definition

Password hashing best practices are security guidelines for storing and verifying passwords that protect users if the database is compromised.

---

## Key Principles

### Never Store Plain Text Passwords

```text
BAD:  password: "admin123"
GOOD: password: "$2b$10$xyz..."
```

---

### Always Use bcrypt

Not MD5, not SHA-256.

```js
const hashed = await bcrypt.hash(
  password,
  10
);
```

---

### Use 10+ Salt Rounds

```js
const SALT_ROUNDS = 12; // Production
```

---

### Never Return Password In Response

```js
const user = await User.findById(id).select("-password");
```

---

### Compare Safely With bcrypt.compare()

```js
const isMatch = await bcrypt.compare(
  inputPassword,
  storedHash
);
```

---

## Registration Flow

```text
User Enters Password
       ↓

Hash Password With bcrypt (10+ rounds)
       ↓

Store Hashed Password In Database
       ↓

Never Store Plain Text
```

---

## Login Flow

```text
User Enters Password
       ↓

Fetch User From Database
       ↓

bcrypt.compare(inputPwd, storedHash)
       ↓

Match: Issue JWT
No Match: 401 Unauthorized
```

---

## Interview Answer (Ready to Speak)

Passwords must always be hashed using bcrypt before storage. Plain-text passwords and weak algorithms like MD5 should never be used. bcrypt handles salting automatically and is intentionally slow to resist brute-force attacks. Passwords should be excluded from query results and compared only using bcrypt.compare().

---

## Quick Revision

```text
Always bcrypt
10+ Salt Rounds
Never Plain Text
.select("-password")
bcrypt.compare()
```

---

### 126. What is Input Validation?

## Definition

Input validation is the process of ensuring that data submitted by users meets expected format, type, and security requirements before processing it.

---

## Why Is It Critical?

Without validation:

```text
Attacker Submits: { "email": null, "password": "" }
```

Application crashes or behaves unexpectedly.

---

## What To Validate

- Required fields present
- Correct data types
- String length limits
- Email format
- Password strength
- Numeric ranges

---

## Example With express-validator

```bash
npm install express-validator
```

```js
const {
  body,
  validationResult
} = require("express-validator");

app.post(
  "/register",
  [
    body("name").notEmpty().trim(),
    body("email").isEmail().normalizeEmail(),
    body("password").isLength({ min: 8 })
  ],
  (req, res) => {
    const errors = validationResult(req);

    if (!errors.isEmpty()) {
      return res.status(400).json({
        errors: errors.array()
      });
    }

    // Proceed with valid data
  }
);
```

---

## Benefits

### Security

Prevents injection attacks.

### Data Integrity

Only valid data enters the database.

### Better Error Messages

Clear feedback to users.

---

## Interview Answer (Ready to Speak)

Input validation ensures that data submitted by users meets defined rules before processing. It prevents security vulnerabilities like injection attacks and maintains data integrity. In Express, the express-validator package provides convenient validation and sanitization utilities.

---

## Quick Revision

```text
Validate All Input
express-validator
Required, Type, Format
400 Bad Request If Invalid
Security Best Practice
```

---

### 127. What is MongoDB Connection Pooling?

## Definition

Connection pooling is a technique where MongoDB maintains a pool of reusable database connections instead of creating a new connection for every request.

---

## Why Is It Needed?

Creating a new database connection for every request is:

- Slow
- Resource-intensive
- Scalability-limiting

Connection pooling reuses existing connections, dramatically improving performance.

---

## Default Pool Size in Mongoose

Mongoose uses a default connection pool size of 5.

---

## Configuring Pool Size

```js
mongoose.connect(process.env.MONGO_URI, {
  maxPoolSize: 10
});
```

---

## How It Works

```text
Application Starts
      ↓

Connection Pool Created (5 by default)
      ↓

Request 1 → Uses Connection 1
Request 2 → Uses Connection 2
Request 3 → Uses Connection 3

All Requests Finish
      ↓

Connections Returned To Pool

New Requests Reuse Same Connections
```

---

## Benefits

### Faster Queries

No connection overhead.

### Better Scalability

Multiple requests use pool connections.

### Resource Efficiency

Fewer connections to MongoDB server.

---

## Interview Answer (Ready to Speak)

Connection pooling maintains a set of pre-established database connections that can be reused across requests. Mongoose uses a default pool size of 5 and allows configuration via maxPoolSize. It significantly improves performance under concurrent load by eliminating the overhead of creating new connections for each request.

---

## Quick Revision

```text
Reuse Connections
Default Pool Size = 5
maxPoolSize Option
Faster Queries
Better Scalability
```

---

### 128. What is MongoDB Transactions?

## Definition

MongoDB Transactions allow multiple read and write operations to execute atomically — either all succeed or all fail — ensuring data consistency.

---

## Why Do We Need Transactions?

Example: Transfer money between accounts.

```text
Debit Account A: -500
Credit Account B: +500
```

If step 1 succeeds but step 2 fails:

```text
Money Lost
Data Inconsistent
```

Transactions ensure either both happen or neither does.

---

## ACID Properties

### Atomicity

All or nothing.

### Consistency

Data remains valid.

### Isolation

Transactions don't interfere.

### Durability

Committed changes persist.

---

## Example in Mongoose

```js
const session = await mongoose.startSession();
session.startTransaction();

try {

  await Account.findByIdAndUpdate(
    fromId,
    { $inc: { balance: -500 } },
    { session }
  );

  await Account.findByIdAndUpdate(
    toId,
    { $inc: { balance: 500 } },
    { session }
  );

  await session.commitTransaction();

} catch (err) {

  await session.abortTransaction();

} finally {

  session.endSession();

}
```

---

## Requires

MongoDB 4.0+ and a replica set or sharded cluster.

---

## Benefits

### Data Integrity

### Atomic Multi-Document Operations

### Financial Systems Support

---

## Interview Answer (Ready to Speak)

MongoDB transactions enable multiple operations to execute atomically across documents and collections. They follow ACID principles, ensuring that either all operations succeed or none take effect. Transactions require MongoDB 4.0 and above with a replica set configuration.

---

## Quick Revision

```text
Atomic Operations
ACID
session.startTransaction()
commitTransaction()
abortTransaction()
```

---

### 129. What is the difference between findById() and findOne()?

## Definition

Both findById() and findOne() are Mongoose methods used to retrieve a single document from a collection, but they differ in how they query.

---

## findById()

Queries by the document's _id field.

```js
const user = await User.findById("65f1a2b3c4d5e6f7");
```

Equivalent to:

```js
User.findOne({ _id: "65f1a2b3c4d5e6f7" })
```

---

## findOne()

Queries by any field or combination of fields.

```js
const user = await User.findOne({
  email: "yogesh@example.com"
});

const user = await User.findOne({
  email: "yogesh@example.com",
  role: "admin"
});
```

---

## Comparison Table

| Feature | findById() | findOne() |
|---------|-----------|-----------|
| Query By | _id only | Any Field |
| Parameter | ID string | Query Object |
| Use Case | ID-based Lookup | Field-based Lookup |
| Returns | Document or null | Document or null |

---

## Handling null

Both return null if no document is found.

```js
const user = await User.findById("invalid-id");

if (!user) {
  return res.status(404).json({
    message: "User Not Found"
  });
}
```

---

## Interview Answer (Ready to Speak)

findById() retrieves a document by its _id field and is shorthand for findOne({ _id: id }). findOne() is more flexible and can query by any field or combination of fields. Both return the document or null if not found.

---

## Quick Revision

```text
findById() → By _id
findOne() → By Any Field
Both Return null If Not Found
findById Is Shorthand
More Flexible = findOne()
```

---

### 130. What are Mongoose Query Operators?

## Definition

Mongoose query operators are special MongoDB operators used inside queries to perform comparisons, logical operations, and other advanced filtering.

---

## Comparison Operators

### $eq — Equal

```js
User.find({ age: { $eq: 25 } })
```

---

### $ne — Not Equal

```js
User.find({ role: { $ne: "admin" } })
```

---

### $gt — Greater Than

```js
User.find({ price: { $gt: 100 } })
```

---

### $gte — Greater Than or Equal

```js
User.find({ price: { $gte: 100 } })
```

---

### $lt — Less Than

```js
User.find({ price: { $lt: 500 } })
```

---

### $lte — Less Than or Equal

```js
User.find({ price: { $lte: 500 } })
```

---

### $in — Matches Any Value In Array

```js
User.find({ role: { $in: ["admin", "moderator"] } })
```

---

### $nin — Not In Array

```js
User.find({ role: { $nin: ["guest"] } })
```

---

## Logical Operators

### $and

```js
User.find({
  $and: [
    { age: { $gte: 18 } },
    { role: "user" }
  ]
})
```

---

### $or

```js
User.find({
  $or: [
    { role: "admin" },
    { role: "moderator" }
  ]
})
```

---

### $not

```js
User.find({
  age: { $not: { $lt: 18 } }
})
```

---

## Element Operators

### $exists

```js
User.find({ phone: { $exists: true } })
```

---

## Real-World Use Case

```js
// Find products between 100-500 in Electronics
const products = await Product.find({
  price: { $gte: 100, $lte: 500 },
  category: "Electronics"
});
```

---

## Interview Answer (Ready to Speak)

MongoDB query operators allow advanced filtering beyond simple equality checks. Comparison operators like $gt, $lt, $in enable range and set-based queries. Logical operators like $and and $or combine multiple conditions. They are used inside find() and other query methods in Mongoose.

---

## Quick Revision

```text
$eq, $ne, $gt, $lt
$gte, $lte
$in, $nin
$and, $or
$exists
```

---

### 131. What is Environment Variable Management with dotenv?

## Definition

dotenv is an npm package that loads environment variables from a .env file into process.env, keeping configuration separate from source code.

---

## Why Use dotenv?

Hardcoding credentials in code is dangerous:

```js
// BAD
mongoose.connect("mongodb+srv://admin:password@cluster.mongodb.net");
```

With dotenv:

```js
// GOOD
mongoose.connect(process.env.MONGO_URI);
```

---

## Installing dotenv

```bash
npm install dotenv
```

---

## Creating .env File

```env
PORT=5000
MONGO_URI=mongodb://localhost:27017/mydb
JWT_SECRET=supersecretkey
NODE_ENV=development
```

---

## Loading dotenv

```js
require("dotenv").config();

// OR at top of index.js
import "dotenv/config"; // ESM
```

---

## Accessing Variables

```js
const port = process.env.PORT || 3000;
const mongoUri = process.env.MONGO_URI;
```

---

## .gitignore

Always add .env to .gitignore:

```text
.env
node_modules/
```

---

## .env.example

Provide a template without real values:

```env
PORT=
MONGO_URI=
JWT_SECRET=
```

---

## Interview Answer (Ready to Speak)

dotenv loads environment variables from a .env file into process.env. It keeps sensitive configuration like database URLs, API keys, and secrets out of source code. The .env file must be added to .gitignore to prevent accidental exposure. A .env.example file can document required variables without revealing actual values.

---

## Quick Revision

```text
npm install dotenv
.env File
process.env.VARIABLE
.gitignore .env
.env.example For Team
```

---

### 132. What is MongoDB Atlas?

## Definition

MongoDB Atlas is the official cloud-hosted database service by MongoDB that provides fully managed MongoDB clusters on AWS, Azure, or GCP.

---

## Why Use Atlas?

Self-hosting MongoDB requires:

- Server setup
- Security configuration
- Backup management
- Scaling management

Atlas handles all of this automatically.

---

## Key Features

### Fully Managed

No server administration.

---

### Automatic Backups

Point-in-time recovery.

---

### Global Clusters

Deploy in multiple regions.

---

### Built-In Security

Network isolation, encryption, IP whitelisting.

---

### Free Tier

512 MB free cluster for development.

---

## Connecting To Atlas

```js
mongoose.connect(
  "mongodb+srv://username:password@cluster.mongodb.net/dbname"
);
```

---

## Benefits

### Quick Setup

### Scalable

### Secure

### Monitoring Dashboard

---

## Interview Answer (Ready to Speak)

MongoDB Atlas is a cloud-hosted, fully managed MongoDB database service. It eliminates the need for manual server setup and maintenance, provides automatic backups, built-in security, and global deployment. It is the recommended choice for production MongoDB deployments.

---

## Quick Revision

```text
Cloud MongoDB
Fully Managed
Free Tier Available
Automatic Backups
mongodb+srv:// Connection String
```

---

### 133. What is Mongoose select() Method?

## Definition

The select() method in Mongoose allows you to specify which fields to include or exclude from query results.

---

## Why Use select()?

By default, all document fields are returned.

For security and performance:

- Exclude passwords from responses
- Return only needed fields
- Reduce data transferred

---

## Include Specific Fields

```js
const users = await User.find()
  .select("name email role");
```

Only name, email, and role are returned.

---

## Exclude Specific Fields

```js
const users = await User.find()
  .select("-password -__v");
```

Everything except password and __v is returned.

The minus (-) means exclude.

---

## In Schema

```js
const userSchema = new Schema({
  name: String,
  password: { type: String, select: false }
});
```

Password is never returned by default.

---

## Real-World Use Case

```js
const user = await User.findById(id)
  .select("-password -__v");

res.json({ user });
```

---

## Interview Answer (Ready to Speak)

The select() method controls which fields Mongoose returns from a query. Using a minus sign before a field name excludes it, while listing field names includes only those. It is commonly used to exclude sensitive fields like passwords from API responses.

---

## Quick Revision

```text
select("name email")
select("-password")
Exclude Sensitive Fields
Reduce Response Size
Schema-Level select:false
```

---

### 134. What is Mongoose lean() Method?

## Definition

The lean() method in Mongoose makes queries return plain JavaScript objects instead of full Mongoose Document instances, resulting in faster and lighter queries.

---

## Why Use lean()?

Mongoose documents come with many methods and properties attached:

```text
save()
validate()
toObject()
toJSON()
Virtual Properties
```

For read-only operations where you don't need these features, lean() is faster.

---

## Without lean()

```js
const users = await User.find();
// Returns full Mongoose Documents
```

---

## With lean()

```js
const users = await User.find().lean();
// Returns plain JavaScript objects
```

---

## Performance Difference

lean() queries can be up to 10x faster because less memory and processing is needed.

---

## When To Use lean()

### Good Use Cases

- GET endpoints that only read data
- Sending API responses
- Data reporting

---

### Do NOT Use lean() When

- You need to call .save()
- You need virtuals
- You need Mongoose hooks on the result

---

## Interview Answer (Ready to Speak)

The lean() method makes Mongoose return plain JavaScript objects instead of full Mongoose Document instances. This significantly improves query performance for read-only operations because the objects are lighter and require less memory. It should not be used when you need to call Mongoose methods like .save() on the results.

---

## Quick Revision

```text
Plain JS Objects
Faster Queries
No Mongoose Methods
Use For Read Operations
Not For .save()
```

---

### 135. What is the difference between PUT and PATCH?

## Definition

Both PUT and PATCH are HTTP methods used for updating resources, but they differ in how they apply the update.

---

## PUT

Replaces the entire resource with the new data.

If a field is not included in the request, it is removed.

---

### Example

Existing document:

```json
{
  "name": "Yogesh",
  "email": "yogesh@example.com",
  "role": "admin"
}
```

PUT request body:

```json
{
  "name": "Yogesh Patel"
}
```

Result:

```json
{
  "name": "Yogesh Patel"
}
```

Email and role are lost.

---

## PATCH

Updates only the provided fields.

Fields not included in the request remain unchanged.

---

### Example

Existing document:

```json
{
  "name": "Yogesh",
  "email": "yogesh@example.com",
  "role": "admin"
}
```

PATCH request body:

```json
{
  "name": "Yogesh Patel"
}
```

Result:

```json
{
  "name": "Yogesh Patel",
  "email": "yogesh@example.com",
  "role": "admin"
}
```

Only name was changed.

---

## Comparison Table

| Feature | PUT | PATCH |
|---------|-----|-------|
| Updates | Entire Resource | Specific Fields |
| Missing Fields | Removed | Unchanged |
| Use Case | Full Replacement | Partial Update |
| Idempotent | Yes | Yes |

---

## In Mongoose

```js
// PATCH — safe partial update
await User.findByIdAndUpdate(
  id,
  req.body,
  { new: true }
);
```

---

## Interview Answer (Ready to Speak)

PUT replaces the entire resource, meaning fields not included in the request body are removed. PATCH performs a partial update, only modifying the fields provided while leaving others unchanged. In practice, PATCH is more commonly used in REST APIs for updating resources.

---

## Quick Revision

```text
PUT = Full Replacement
PATCH = Partial Update
PATCH Preferred In APIs
Missing Fields Removed In PUT
findByIdAndUpdate for Both
```

---

### 136. What is Mongoose virtuals?

## Definition

Mongoose virtuals are document properties that are computed from other fields but are not stored in the database.

---

## Why Use Virtuals?

Sometimes you need derived data:

```text
firstName + lastName → fullName
```

Instead of storing fullName separately, compute it dynamically.

---

## Example

```js
const userSchema = new Schema({
  firstName: String,
  lastName: String
});

userSchema.virtual("fullName").get(function() {
  return `${this.firstName} ${this.lastName}`;
});

const User = model("User", userSchema);

const user = new User({
  firstName: "Yogesh",
  lastName: "Patel"
});

console.log(user.fullName);
// Yogesh Patel
```

---

## Including Virtuals In toJSON()

By default, virtuals are not included in JSON output.

```js
const userSchema = new Schema(
  { ... },
  {
    toJSON: { virtuals: true },
    toObject: { virtuals: true }
  }
);
```

---

## Benefits

### No Extra Storage

Not saved in MongoDB.

### Clean Data Model

Computed on the fly.

### Reusable Logic

---

## Common Use Cases

- Full name from first and last name
- Age from date of birth
- Formatted dates
- URL from slug

---

## Interview Answer (Ready to Speak)

Mongoose virtuals are computed properties derived from other schema fields that are not persisted in the database. They are useful for providing derived data like full names or formatted values. To include them in JSON responses, the toJSON: { virtuals: true } option must be set on the schema.

---

## Quick Revision

```text
Computed Properties
Not Stored In DB
.get() function
toJSON: { virtuals: true }
Derived Data
```

---

### 137. What is Error Handling in Node.js?

## Definition

Error handling in Node.js is the process of catching, managing, and responding to errors that occur during the execution of an application.

---

## Types of Errors

### Synchronous Errors

```js
try {
  throw new Error("Sync Error");
} catch (err) {
  console.log(err.message);
}
```

---

### Asynchronous Errors (Async/Await)

```js
try {
  const data = await fetchData();
} catch (err) {
  console.log(err.message);
}
```

---

### Unhandled Promise Rejections

```js
process.on(
  "unhandledRejection",
  (reason, promise) => {
    console.error("Unhandled Rejection:", reason);
    process.exit(1);
  }
);
```

---

### Uncaught Exceptions

```js
process.on("uncaughtException", (err) => {
  console.error("Uncaught Exception:", err);
  process.exit(1);
});
```

---

## Express Centralized Error Handling

```js
// Custom Error Class
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
  }
}

// Error Middleware
app.use((err, req, res, next) => {
  const statusCode = err.statusCode || 500;
  res.status(statusCode).json({
    success: false,
    message: err.message || "Server Error"
  });
});
```

---

## Best Practices

### Always Handle Errors In Async Routes

```js
app.get("/users", async (req, res, next) => {
  try {
    const users = await User.find();
    res.json(users);
  } catch (err) {
    next(err);
  }
});
```

---

### Never Expose Internal Errors To Clients

```js
// BAD
res.status(500).json({ error: err.stack });

// GOOD
res.status(500).json({ message: "Server Error" });
```

---

## Interview Answer (Ready to Speak)

Error handling in Node.js involves using try-catch for synchronous and async/await code, centralized error middleware in Express, and process-level handlers for uncaught exceptions and unhandled rejections. A custom AppError class provides structured error responses with appropriate HTTP status codes.

---

## Quick Revision

```text
try-catch
next(err)
Error Middleware
AppError Class
process.on("uncaughtException")
```

---

### 138. What is API Versioning?

## Definition

API versioning is the practice of maintaining multiple versions of an API to allow backward-compatible changes without breaking existing clients.

---

## Why Is It Needed?

```text
API v1 Released
Clients Use v1

Breaking Change Needed
     ↓

Release API v2
v1 Still Works For Old Clients
v2 Available For New Clients
```

---

## Versioning Strategies

### URL Versioning (Most Common)

```text
/api/v1/users
/api/v2/users
```

---

### Header Versioning

```http
API-Version: 2
```

---

### Query Parameter Versioning

```text
/api/users?version=2
```

---

## Example in Express

```js
const v1Router = require("./routes/v1");
const v2Router = require("./routes/v2");

app.use("/api/v1", v1Router);
app.use("/api/v2", v2Router);
```

---

## Benefits

### Backward Compatibility

### Gradual Migration

### Client Flexibility

---

## Interview Answer (Ready to Speak)

API versioning allows multiple versions of an API to coexist so that existing clients are not broken when new changes are introduced. The most common approach is URL versioning using prefixes like /api/v1 and /api/v2. It enables gradual migration and maintains backward compatibility.

---

## Quick Revision

```text
/api/v1 /api/v2
Backward Compatibility
URL Versioning Common
Separate Router Per Version
Gradual Migration
```

---

### 139. What is Pagination in APIs?

## Definition

Pagination is the technique of dividing large query results into smaller pages so that only a subset of data is returned per request.

---

## Why Is It Important?

Returning all 100,000 products at once:

```text
Slow Response
High Memory Usage
Bad User Experience
```

Pagination returns only what is needed.

---

## Types of Pagination

### Page-Based (Offset)

```text
/products?page=2&limit=10
```

---

### Cursor-Based

```text
/products?cursor=lastDocId&limit=10
```

---

## Example: Page-Based Pagination

```js
app.get("/products", async (req, res) => {
  const page = parseInt(req.query.page) || 1;
  const limit = parseInt(req.query.limit) || 10;
  const skip = (page - 1) * limit;

  const products = await Product.find()
    .skip(skip)
    .limit(limit);

  const total = await Product.countDocuments();

  res.json({
    products,
    currentPage: page,
    totalPages: Math.ceil(total / limit),
    totalCount: total
  });
});
```

---

## Response Format

```json
{
  "products": [...],
  "currentPage": 2,
  "totalPages": 10,
  "totalCount": 100
}
```

---

## Benefits

### Performance

### Lower Memory Usage

### Better User Experience

---

## Interview Answer (Ready to Speak)

Pagination splits large datasets into pages so that only a manageable subset is returned per request. In Mongoose, skip() and limit() implement page-based pagination. The response typically includes the current page, total pages, and total count alongside the data.

---

## Quick Revision

```text
skip and limit
page and limit Query Params
(page-1) * limit = skip
totalPages = Math.ceil(total/limit)
Performance Optimization
```

---

### 140. What is Sorting in MongoDB Queries?

## Definition

Sorting in MongoDB organizes query results in ascending or descending order based on one or more fields.

---

## Syntax

```js
Model.find().sort({ fieldName: 1 });  // Ascending
Model.find().sort({ fieldName: -1 }); // Descending
```

---

## Examples

### Sort By Price Ascending

```js
const products = await Product.find()
  .sort({ price: 1 });
```

---

### Sort By Price Descending

```js
const products = await Product.find()
  .sort({ price: -1 });
```

---

### Sort By Multiple Fields

```js
const products = await Product.find()
  .sort({ category: 1, price: -1 });
```

---

## Sort From Query Parameters

```js
app.get("/products", async (req, res) => {
  const { sort = "createdAt", order = "desc" } = req.query;

  const sortObj = {};
  sortObj[sort] = order === "asc" ? 1 : -1;

  const products = await Product.find()
    .sort(sortObj);

  res.json(products);
});
```

---

## With Pagination

```js
const products = await Product.find()
  .sort({ price: 1 })
  .skip(skip)
  .limit(limit);
```

---

## Interview Answer (Ready to Speak)

MongoDB and Mongoose support sorting using the sort() method with 1 for ascending and -1 for descending order. Sorting can be applied to multiple fields and combined with pagination for efficient data retrieval in production APIs.

---

## Quick Revision

```text
sort({ field: 1 }) = Ascending
sort({ field: -1 }) = Descending
Multiple Fields
Dynamic Sort From Query
Combined With Pagination
```

---

### 141. What is Filtering in MongoDB Queries?

## Definition

Filtering in MongoDB means querying only documents that match specific conditions using a query object.

---

## Basic Filtering

```js
// Find all users with role "admin"
const admins = await User.find({
  role: "admin"
});
```

---

## Multiple Conditions

```js
const users = await User.find({
  role: "admin",
  isActive: true
});
```

---

## Range Filtering

```js
const products = await Product.find({
  price: { $gte: 100, $lte: 500 }
});
```

---

## Dynamic Filtering From Query Params

```js
app.get("/products", async (req, res) => {
  const { category, minPrice, maxPrice } = req.query;

  const filter = {};

  if (category) filter.category = category;

  if (minPrice || maxPrice) {
    filter.price = {};
    if (minPrice) filter.price.$gte = Number(minPrice);
    if (maxPrice) filter.price.$lte = Number(maxPrice);
  }

  const products = await Product.find(filter);
  res.json(products);
});
```

---

## Text Search Filtering

```js
const results = await Product.find({
  $text: { $search: "laptop" }
});
```

Requires a text index on the collection.

---

## Interview Answer (Ready to Speak)

Filtering in MongoDB means passing a query object to methods like find() to retrieve only matching documents. Dynamic filtering from API query parameters allows clients to filter by multiple criteria. MongoDB operators like $gte, $lte, $in, and $text provide powerful filtering capabilities.

---

## Quick Revision

```text
find({ condition })
Dynamic Filter Object
$gte, $lte, $in
Text Search
Query Params To Filter
```

---

### 142. What is a RESTful API?

## Definition

A RESTful API (Representational State Transfer) is an architectural style for building APIs that uses HTTP methods and stateless communication to interact with resources.

---

## REST Principles

### Stateless

Every request must contain all information needed.

---

### Resource-Based

Everything is a resource.

```text
/users
/products
/orders
```

---

### HTTP Methods

```text
GET    → Read
POST   → Create
PUT    → Update (Full)
PATCH  → Update (Partial)
DELETE → Delete
```

---

### Uniform Interface

Consistent URL structure.

---

### JSON Responses

```json
{
  "success": true,
  "data": { ... }
}
```

---

## Standard REST API URLs

```text
GET    /users          → Get all users
GET    /users/:id      → Get one user
POST   /users          → Create user
PUT    /users/:id      → Replace user
PATCH  /users/:id      → Update user fields
DELETE /users/:id      → Delete user
```

---

## Standard Response Format

```json
{
  "success": true,
  "message": "User Created",
  "data": {
    "_id": "65f1...",
    "name": "Yogesh"
  }
}
```

---

## Benefits

### Simplicity

### Scalability

### Stateless

### Language Independent

### Widely Adopted

---

## Interview Answer (Ready to Speak)

A RESTful API follows the REST architectural style, using HTTP methods to perform operations on resources. It is stateless, resource-based, and uses standard HTTP status codes and JSON responses. REST APIs are the industry standard for building scalable web services.

---

## Quick Revision

```text
REST = Resource-Based
Stateless
HTTP Methods
JSON Responses
Standard URL Patterns
```

---

### 143. What is API Security Best Practices?

## Definition

API security best practices are guidelines that protect APIs from unauthorized access, data breaches, and malicious attacks.

---

## Key Security Practices

### 1. Always Use HTTPS

Encrypt all communication.

---

### 2. Use JWT Properly

Short expiry. HTTP-Only cookies for refresh tokens.

---

### 3. Validate All Input

```js
// express-validator
body("email").isEmail()
body("password").isLength({ min: 8 })
```

---

### 4. Hash Passwords With bcrypt

Never store plain-text passwords.

---

### 5. Rate Limiting

```js
app.use(rateLimit({ max: 100, windowMs: 15 * 60 * 1000 }));
```

---

### 6. Use Helmet.js

```js
app.use(helmet());
```

---

### 7. Sanitize Data

Prevent NoSQL Injection.

```bash
npm install express-mongo-sanitize
```

```js
app.use(mongoSanitize());
```

---

### 8. Use Environment Variables

Keep secrets out of code.

---

### 9. Proper Error Messages

Do not expose stack traces to clients.

---

### 10. CORS Configuration

Allow only trusted origins.

---

## Interview Answer (Ready to Speak)

API security best practices include using HTTPS, implementing JWT with short expiry, validating and sanitizing all input, hashing passwords with bcrypt, rate limiting, using Helmet.js for security headers, and proper CORS configuration. These practices collectively protect APIs from injection attacks, unauthorized access, and data leaks.

---

## Quick Revision

```text
HTTPS Always
bcrypt Passwords
Input Validation
Rate Limiting
Helmet.js
Environment Variables
```

---

### 144. What is NoSQL Injection and Prevention?

## Definition

NoSQL injection is an attack where malicious data is submitted to manipulate MongoDB queries, potentially exposing unauthorized data.

---

## Example Attack

```json
{
  "email": { "$ne": "" },
  "password": { "$ne": "" }
}
```

Without sanitization, this query matches ALL users.

---

## Prevention With express-mongo-sanitize

```bash
npm install express-mongo-sanitize
```

```js
const mongoSanitize = require("express-mongo-sanitize");

app.use(mongoSanitize());
```

This removes $ and . characters from request inputs.

---

## Manual Prevention

```js
if (typeof req.body.email !== "string") {
  return res.status(400).json({
    message: "Invalid Input"
  });
}
```

---

## Benefits

### Prevents Query Manipulation

### Data Protection

### Simple Integration

---

## Interview Answer (Ready to Speak)

NoSQL injection attacks exploit MongoDB's query operators by injecting operator objects like $ne into request data. The express-mongo-sanitize package strips these operators from request inputs, preventing the attack. Input type validation is also important to ensure expected data types.

---

## Quick Revision

```text
$ne, $gt Operator Injection
express-mongo-sanitize
Strips $ Operators
Type Validation
Critical Security Practice
```

---

### 145. What is the difference between SQL and NoSQL?

## Definition

SQL (Structured Query Language) databases use tables with a fixed schema, while NoSQL databases use flexible data models such as documents, key-value pairs, or graphs.

---

## SQL Databases

Examples: MySQL, PostgreSQL, SQLite.

```text
Table: users
id | name  | email          | role
1  | Yogesh| yogesh@mail.com| admin
```

---

## NoSQL Databases

Examples: MongoDB, Redis, Cassandra.

```json
{
  "_id": "65f1...",
  "name": "Yogesh",
  "email": "yogesh@mail.com",
  "role": "admin",
  "address": { ... }
}
```

---

## Comparison Table

| Feature | SQL | NoSQL |
|---------|-----|-------|
| Schema | Fixed | Flexible |
| Data Format | Tables/Rows | Documents/Key-Value |
| Scaling | Vertical | Horizontal |
| Transactions | Strong | Limited (improving) |
| Relationships | Joins | Embedded/Referenced |
| Use Case | Structured Data | Unstructured/Scalable |
| Examples | MySQL, PostgreSQL | MongoDB, Redis |

---

## When To Use SQL

- Financial systems
- Applications needing strong transactions
- Well-defined, rarely-changing schemas
- Complex relationships

---

## When To Use NoSQL

- High-volume, scalable applications
- Flexible or evolving data structures
- Real-time applications
- Content management systems

---

## Interview Answer (Ready to Speak)

SQL databases use structured tables with fixed schemas and are excellent for complex relationships and strong transactions. NoSQL databases like MongoDB use flexible document formats and scale horizontally, making them suitable for high-volume, schema-flexible applications. The choice depends on data structure, consistency requirements, and scalability needs.

---

## Quick Revision

```text
SQL = Tables, Fixed Schema
NoSQL = Documents, Flexible
SQL = Vertical Scaling
NoSQL = Horizontal Scaling
MongoDB = NoSQL
```

---

### 146. What is the $lookup Stage in MongoDB Aggregation?

## Definition

$lookup is a MongoDB aggregation stage that performs a left outer join between two collections, similar to SQL's JOIN.

---

## Why Is It Needed?

MongoDB stores data in separate collections:

```text
orders collection: { userId: ObjectId }
users collection: { _id, name, email }
```

$lookup fetches the related user data within an aggregation.

---

## Syntax

```js
{
  $lookup: {
    from: "collectionName",
    localField: "localField",
    foreignField: "foreignField",
    as: "outputArrayName"
  }
}
```

---

## Example

```js
const orders = await Order.aggregate([
  {
    $lookup: {
      from: "users",
      localField: "userId",
      foreignField: "_id",
      as: "userDetails"
    }
  },
  {
    $unwind: "$userDetails"
  }
]);
```

---

## $unwind

$lookup returns an array. $unwind deconstructs it into individual objects.

```js
{ $unwind: "$userDetails" }
```

---

## Full Pipeline Example

```js
const result = await Order.aggregate([
  { $match: { status: "completed" } },
  {
    $lookup: {
      from: "users",
      localField: "userId",
      foreignField: "_id",
      as: "user"
    }
  },
  { $unwind: "$user" },
  {
    $project: {
      orderId: "$_id",
      userName: "$user.name",
      amount: 1
    }
  }
]);
```

---

## Interview Answer (Ready to Speak)

$lookup is a MongoDB aggregation stage that joins documents from one collection with documents from another, similar to SQL JOIN. The result is added as an array field which can be deconstructed using $unwind. It is commonly used in aggregation pipelines when cross-collection data is needed.

---

## Quick Revision

```text
$lookup = JOIN
from, localField, foreignField, as
Returns Array
$unwind To Flatten
Used In Aggregation
```

---

### 147. What is the $group Stage in MongoDB Aggregation?

## Definition

The $group stage groups documents by a specified field and applies accumulator operations like sum, average, count, min, and max.

---

## Syntax

```js
{
  $group: {
    _id: "$fieldToGroupBy",
    result: { $accumulator: "$field" }
  }
}
```

---

## Accumulators

| Operator | Purpose |
|----------|---------|
| $sum | Total sum |
| $avg | Average |
| $count | Count documents |
| $min | Minimum value |
| $max | Maximum value |
| $push | Collect values into array |
| $first | First value in group |
| $last | Last value in group |

---

## Examples

### Count By Category

```js
{
  $group: {
    _id: "$category",
    count: { $sum: 1 }
  }
}
```

---

### Total Revenue Per Category

```js
{
  $group: {
    _id: "$category",
    totalRevenue: { $sum: "$price" },
    avgPrice: { $avg: "$price" },
    productCount: { $sum: 1 }
  }
}
```

---

### Count All Documents

```js
{
  $group: {
    _id: null,
    total: { $sum: 1 }
  }
}
```

---

## Full Example

```js
const stats = await Product.aggregate([
  { $match: { inStock: true } },
  {
    $group: {
      _id: "$category",
      totalProducts: { $sum: 1 },
      avgPrice: { $avg: "$price" },
      maxPrice: { $max: "$price" }
    }
  },
  { $sort: { totalProducts: -1 } }
]);
```

---

## Interview Answer (Ready to Speak)

The $group stage groups documents by a field and applies accumulators to compute aggregate values. Common accumulators include $sum for totals, $avg for averages, and $count or $sum: 1 for counting. It is the core aggregation stage for analytics and reporting.

---

## Quick Revision

```text
$group Groups Documents
_id = Group By Field
$sum, $avg, $min, $max
$sum: 1 = Count
Analytics and Reporting
```

---

### 148. What is the $project Stage in MongoDB Aggregation?

## Definition

The $project stage shapes the output of an aggregation pipeline by including, excluding, or transforming fields.

---

## Include Fields

```js
{ $project: { name: 1, price: 1 } }
```

Only name and price are included. _id is included by default.

---

## Exclude Fields

```js
{ $project: { password: 0, __v: 0 } }
```

---

## Exclude _id

```js
{ $project: { _id: 0, name: 1 } }
```

---

## Rename Fields

```js
{
  $project: {
    productName: "$name",
    productPrice: "$price"
  }
}
```

---

## Computed Fields

```js
{
  $project: {
    name: 1,
    discountedPrice: {
      $multiply: ["$price", 0.9]
    }
  }
}
```

---

## Interview Answer (Ready to Speak)

The $project stage controls which fields appear in aggregation output and allows field renaming and computation. It is similar to SELECT in SQL and is commonly used as the final stage to shape the response data in aggregation pipelines.

---

## Quick Revision

```text
1 = Include
0 = Exclude
Rename Fields
Compute New Fields
Like SQL SELECT
```

---

### 149. What is the difference between deleteOne() and deleteMany()?

## Definition

Both are Mongoose/MongoDB methods for deleting documents, but they differ in how many documents they delete.

---

## deleteOne()

Deletes the first matching document.

```js
await User.deleteOne({ email: "test@mail.com" });
```

---

## deleteMany()

Deletes all matching documents.

```js
// Delete all inactive users
await User.deleteMany({ isActive: false });
```

---

## findByIdAndDelete()

Deletes a document by _id and returns the deleted document.

```js
const deleted = await User.findByIdAndDelete(id);
```

---

## findOneAndDelete()

Deletes first matching document and returns it.

```js
const deleted = await User.findOneAndDelete({
  email: "test@mail.com"
});
```

---

## Comparison Table

| Method | Deletes | Returns Document |
|--------|---------|-----------------|
| deleteOne() | One | No |
| deleteMany() | Multiple | No |
| findByIdAndDelete() | One by _id | Yes |
| findOneAndDelete() | First Match | Yes |

---

## Interview Answer (Ready to Speak)

deleteOne() removes the first document matching the condition, while deleteMany() removes all matching documents. findByIdAndDelete() is preferred in REST APIs when deleting by ID because it returns the deleted document, confirming what was removed.

---

## Quick Revision

```text
deleteOne() = One Document
deleteMany() = All Matching
findByIdAndDelete() = By ID + Returns Doc
findOneAndDelete() = First Match + Returns Doc
```

---

### 150. What is the difference between updateOne() and findByIdAndUpdate()?

## Definition

Both methods update documents in MongoDB but differ in query type and whether they return the updated document.

---

## updateOne()

Updates the first matching document. Returns update result metadata, not the document.

```js
await User.updateOne(
  { email: "yogesh@example.com" },
  { $set: { role: "admin" } }
);
```

---

## updateMany()

Updates all matching documents.

```js
await User.updateMany(
  { isActive: false },
  { $set: { isActive: true } }
);
```

---

## findByIdAndUpdate()

Finds by _id and updates. Returns the document.

```js
const updated = await User.findByIdAndUpdate(
  id,
  { role: "admin" },
  { new: true }
);
```

The { new: true } option returns the updated document instead of the original.

---

## Comparison Table

| Method | Query | Returns |
|--------|-------|---------|
| updateOne() | Any Field | Metadata |
| updateMany() | Any Field | Metadata |
| findByIdAndUpdate() | _id | Document |
| findOneAndUpdate() | Any Field | Document |

---

## $set Operator

Prevents overwriting the entire document:

```js
// Without $set — replaces entire document
{ role: "admin" }

// With $set — updates only the role field
{ $set: { role: "admin" } }
```

---

## Interview Answer (Ready to Speak)

updateOne() finds the first matching document and updates it, returning metadata. findByIdAndUpdate() finds a document by _id and updates it, returning the document itself when new: true is set. findByIdAndUpdate() is preferred in REST APIs because it confirms the update by returning the modified document.

---

## Quick Revision

```text
updateOne() = Metadata Return
findByIdAndUpdate() = Document Return
new: true = Return Updated
$set = Partial Update
updateMany() = Update All Matching
```

---

### 151. What is express.urlencoded() Middleware?

## Definition

express.urlencoded() is a built-in Express middleware that parses incoming requests with URL-encoded payloads (form data) and makes the data available on req.body.

---

## Why Do We Need It?

HTML forms submit data in URL-encoded format:

```text
name=Yogesh&email=yogesh%40example.com
```

Without the middleware, req.body would be undefined.

---

## Usage

```js
app.use(express.urlencoded({ extended: true }));
```

---

## extended Option

### extended: true

Uses the qs library. Supports nested objects.

```text
user[name]=Yogesh&user[age]=25
```

---

### extended: false

Uses querystring library. Flat key-value pairs only.

---

## Difference From express.json()

| Feature | express.json() | express.urlencoded() |
|---------|---------------|---------------------|
| Content-Type | application/json | application/x-www-form-urlencoded |
| Use Case | REST APIs | HTML Forms |
| Data Format | JSON | Key=Value Pairs |

---

## Interview Answer (Ready to Speak)

express.urlencoded() parses form data submitted with the application/x-www-form-urlencoded content type and attaches it to req.body. The extended option controls whether nested objects are supported. Most REST APIs use express.json(), while express.urlencoded() is used when processing HTML form submissions.

---

## Quick Revision

```text
Parses Form Data
application/x-www-form-urlencoded
req.body
extended: true
HTML Forms
```

---

### 152. What is morgan Middleware?

## Definition

morgan is a popular HTTP request logger middleware for Node.js and Express that automatically logs details about each request.

---

## Why Use morgan?

Tracking requests in production is essential for:

- Debugging
- Performance monitoring
- Security auditing
- Error analysis

---

## Installing morgan

```bash
npm install morgan
```

---

## Usage

```js
const morgan = require("morgan");
app.use(morgan("dev"));
```

---

## Log Formats

### dev

Colored concise output for development.

```text
GET /users 200 5ms
```

---

### combined

Standard Apache combined log format for production.

```text
127.0.0.1 - user [date] "GET /users HTTP/1.1" 200 156
```

---

### tiny

Minimal output.

```text
GET /users 200 - 5ms
```

---

## Custom Format

```js
morgan(":method :url :status - :response-time ms")
```

---

## Benefits

### Request Logging

### Performance Tracking

### Debugging Aid

### Production Monitoring

---

## Interview Answer (Ready to Speak)

morgan is an HTTP request logger middleware for Express. It logs each request's method, URL, status code, and response time. The dev format is commonly used in development for colored output, while combined is used in production for comprehensive logging.

---

## Quick Revision

```text
HTTP Request Logger
npm install morgan
morgan("dev")
morgan("combined")
Debugging and Monitoring
```

---

### 153. What is the difference between req.body, req.params, and req.query?

## Definition

All three are properties of the Express request object that provide access to different parts of an incoming request.

---

## req.body

Contains data sent in the request body.

Used for POST, PUT, PATCH requests.

Requires express.json() or express.urlencoded().

```js
// POST /users
// Body: { "name": "Yogesh" }
console.log(req.body.name); // Yogesh
```

---

## req.params

Contains dynamic segments of the URL path.

Defined with : in the route.

```js
// GET /users/42
// Route: /users/:id
console.log(req.params.id); // 42
```

---

## req.query

Contains query string parameters.

Optional key-value pairs after ?.

```js
// GET /products?page=2&limit=10
console.log(req.query.page);  // 2
console.log(req.query.limit); // 10
```

---

## Comparison Table

| Feature | req.body | req.params | req.query |
|---------|----------|------------|-----------|
| Location | Request Body | URL Path | URL Query String |
| Requires Middleware | Yes | No | No |
| HTTP Methods | POST, PUT, PATCH | Any | Any |
| Example | { name: "Yogesh" } | id = 42 | page = 2 |

---

## Interview Answer (Ready to Speak)

req.body contains data from the request body, typically used in POST/PUT requests and requires body-parsing middleware. req.params contains dynamic URL path segments defined with a colon. req.query contains optional query string parameters after the question mark in the URL.

---

## Quick Revision

```text
req.body = Request Body (POST/PUT)
req.params = URL Segments (:id)
req.query = Query String (?key=value)
Body Needs Middleware
Params and Query = No Middleware
```

---

### 154. What is process.argv?

## Definition

process.argv is a Node.js array containing the command-line arguments passed when the Node.js process was launched.

---

## Structure

```text
process.argv[0] = node path
process.argv[1] = script path
process.argv[2+] = custom arguments
```

---

## Example

```bash
node app.js hello world
```

```js
console.log(process.argv);
// ["node", "/path/to/app.js", "hello", "world"]

console.log(process.argv[2]); // hello
console.log(process.argv[3]); // world
```

---

## Real-World Use Case

```js
const environment = process.argv[2] || "development";

if (environment === "production") {
  console.log("Production Mode");
}
```

---

## Benefits

### CLI Tools

### Script Configuration

### Dynamic Behavior

---

## Interview Answer (Ready to Speak)

process.argv is an array in Node.js that contains command-line arguments. The first two elements are the Node.js executable path and the script file path. Custom arguments start from index 2. It is commonly used in CLI tools and scripts to accept dynamic configuration at runtime.

---

## Quick Revision

```text
Command-Line Arguments
process.argv[0] = node
process.argv[1] = script
process.argv[2+] = custom args
CLI Tools
```

---

### 155. What is Nodemon?

## Definition

Nodemon is a development tool that automatically restarts the Node.js application whenever file changes are detected.

---

## Why Use Nodemon?

Without nodemon:

```text
Change Code
Stop Server (Ctrl+C)
Restart Server (node app.js)
```

With nodemon:

```text
Change Code
Server Restarts Automatically
```

---

## Installing Nodemon

```bash
npm install nodemon --save-dev
```

---

## Running With Nodemon

```bash
npx nodemon index.js
```

---

## In package.json Scripts

```json
{
  "scripts": {
    "dev": "nodemon index.js"
  }
}
```

```bash
npm run dev
```

---

## Configuration

```json
// nodemon.json
{
  "watch": ["src"],
  "ext": "js,json",
  "ignore": ["node_modules"]
}
```

---

## Benefits

### Faster Development

### No Manual Restarts

### File Watch Configuration

---

## Interview Answer (Ready to Speak)

Nodemon is a development utility that monitors file changes and automatically restarts the Node.js server. It significantly speeds up development by eliminating the need for manual restarts. It is installed as a dev dependency and typically configured as the dev script in package.json.

---

## Quick Revision

```text
Auto-Restart Server
Dev Dependency
nodemon index.js
--save-dev
npm run dev
```

---

### 156. What is the difference between synchronous and asynchronous code?

## Definition

Synchronous code executes line by line in order, blocking further execution until each operation completes. Asynchronous code initiates an operation and moves on without waiting for it to finish.

---

## Synchronous Code

```js
console.log("Step 1");
const data = fs.readFileSync("file.txt", "utf8");
console.log("Step 2");
console.log("Step 3");
```

Output:

```text
Step 1
(waits for file to be read)
Step 2
Step 3
```

---

## Asynchronous Code

```js
console.log("Step 1");

fs.readFile("file.txt", "utf8", (err, data) => {
  console.log("File Read");
});

console.log("Step 2");
```

Output:

```text
Step 1
Step 2
File Read
```

---

## Comparison

| Feature | Synchronous | Asynchronous |
|---------|-------------|--------------|
| Execution | Sequential | Non-sequential |
| Blocking | Yes | No |
| Performance | Lower | Higher |
| Server Use | Not Recommended | Recommended |

---

## Interview Answer (Ready to Speak)

Synchronous code blocks execution until each operation finishes, while asynchronous code allows the program to continue executing while waiting for operations to complete. Node.js is built around asynchronous execution to handle many concurrent operations efficiently without blocking the Event Loop.

---

## Quick Revision

```text
Sync = Blocking
Async = Non-Blocking
Async Preferred For Servers
Event Loop Handles Async
Callbacks, Promises, Async/Await
```

---

### 157. What is __dirname vs process.cwd()?

## Definition

Both provide directory paths but from different perspectives.

---

## __dirname

Returns the absolute path of the directory containing the CURRENTLY EXECUTING FILE.

```js
// In /project/src/utils/helper.js
console.log(__dirname);
// /project/src/utils
```

Does not change regardless of where node is executed from.

---

## process.cwd()

Returns the CURRENT WORKING DIRECTORY — the directory from which the Node.js process was launched.

```js
// Running from /project
node src/utils/helper.js

console.log(process.cwd());
// /project
```

Changes based on where you run the command.

---

## Comparison

| Feature | __dirname | process.cwd() |
|---------|-----------|----------------|
| Returns | File's directory | Execution directory |
| Changes | Never | Based on terminal location |
| Use For | File references relative to file | Config/startup paths |

---

## Interview Answer (Ready to Speak)

__dirname returns the directory of the currently executing file and is stable regardless of where Node.js is executed from. process.cwd() returns the current working directory which depends on where the node command was run. For file references within a module, __dirname is safer and more predictable.

---

## Quick Revision

```text
__dirname = File's Directory
process.cwd() = Terminal Directory
__dirname = Stable
process.cwd() = Changes
Use __dirname For File Paths
```

---

### 158. What is Buffer.from() vs Buffer.alloc()?

## Definition

Both create Buffers but from different starting points.

---

## Buffer.from()

Creates a Buffer from existing data.

```js
const buf = Buffer.from("Hello Node.js");
console.log(buf);
// <Buffer 48 65 6c 6c 6f...>

console.log(buf.toString());
// Hello Node.js
```

---

## Buffer.alloc()

Creates a fixed-size Buffer initialized with zeros.

```js
const buf = Buffer.alloc(10);
console.log(buf);
// <Buffer 00 00 00 00 00 00 00 00 00 00>
```

Safe. Memory is always zeroed.

---

## Buffer.allocUnsafe()

Creates a fixed-size Buffer WITHOUT initializing memory.

Faster but may contain old memory data.

```js
const buf = Buffer.allocUnsafe(10);
// May contain random data
```

---

## Comparison

| Method | From Existing Data | Zero-Filled | Fast |
|--------|-------------------|-------------|------|
| Buffer.from() | Yes | N/A | Yes |
| Buffer.alloc() | No | Yes | No |
| Buffer.allocUnsafe() | No | No | Yes |

---

## Interview Answer (Ready to Speak)

Buffer.from() creates a Buffer from existing data like a string or array. Buffer.alloc() creates a zero-filled Buffer of a specified size and is safe to use when the contents are unknown. Buffer.allocUnsafe() is faster but skips initialization, potentially exposing old memory contents.

---

## Quick Revision

```text
Buffer.from() = Existing Data
Buffer.alloc() = Safe Empty Buffer
Buffer.allocUnsafe() = Fast Unsafe
alloc() = Zero Filled
from() = Most Common
```

---

### 159. What is the Difference Between Streams and Buffers?

## Definition

Buffers and Streams are both mechanisms for handling binary data in Node.js, but they work differently.

---

## Buffers

Store a fixed-size chunk of binary data in memory.

```js
const buf = Buffer.from("Hello");
```

The entire data is available at once.

---

## Streams

Process data in small chunks over time without loading everything into memory.

```js
const readStream = fs.createReadStream("video.mp4");
readStream.on("data", chunk => {
  // Process each chunk
});
```

---

## Comparison

| Feature | Buffer | Stream |
|---------|--------|--------|
| Memory | Stores All Data | Chunk by Chunk |
| Suitable For | Small Data | Large Files |
| Processing | After Full Load | While Loading |
| Memory Usage | Higher | Lower |

---

## Real-World Analogy

Buffer:

```text
Fill Entire Bottle Before Drinking
```

Stream:

```text
Drink While Water Is Flowing
```

---

## Interview Answer (Ready to Speak)

Buffers hold a fixed amount of binary data in memory and are suitable for small data. Streams process data incrementally in chunks, making them memory-efficient for large files like videos or logs. Node.js applications commonly use both: streams for transport and buffers as the chunk containers within streams.

---

## Quick Revision

```text
Buffer = Fixed Size In Memory
Stream = Chunk by Chunk
Stream = Memory Efficient
Buffer = Small Data
Stream = Large Files
```

---

### 160. What is Node.js Best Practices Summary?

## Definition

Node.js best practices are proven guidelines for building production-ready, scalable, and secure Node.js applications.

---

## Architecture

### Use MVC Pattern

Separate models, controllers, and routes.

---

### Environment Variables

Use dotenv and never hardcode credentials.

---

### Centralized Error Handling

One error middleware in Express.

---

## Security

### Use Helmet.js

Set security headers automatically.

---

### Validate All Input

Use express-validator.

---

### Hash Passwords

Use bcrypt with 10+ rounds.

---

### Use HTTPS

Always in production.

---

### Rate Limiting

Protect APIs from abuse.

---

## Performance

### Use Async/Await

Never block the Event Loop.

---

### Use Streaming For Large Files

Avoid loading large files into memory.

---

### Pagination

Always paginate large datasets.

---

### Mongoose lean()

For read-only operations.

---

## Database

### Index Frequently Queried Fields

Improve MongoDB query performance.

---

### Use Connection Pooling

Default or custom pool size.

---

### Handle Errors In Database Operations

Always use try-catch with async operations.

---

## Code Quality

### Use ESLint

Enforce code standards.

---

### Write Meaningful Variable Names

---

### Handle Promise Rejections

Register process.on("unhandledRejection").

---

## Interview Answer (Ready to Speak)

Node.js best practices cover architecture (MVC pattern, environment variables), security (Helmet, input validation, bcrypt, HTTPS, rate limiting), performance (async/await, streaming, pagination, lean()), database (indexing, connection pooling), and code quality (error handling, ESLint). Following these ensures production-ready, scalable, and secure applications.

---

## Quick Revision

```text
MVC Architecture
dotenv + Environment Variables
Helmet + Validation + bcrypt
Async Always
Pagination + lean()
Centralized Error Handling
```

---

### 161. What is the difference between app.use() and app.get()?

## Definition

Both register middleware or route handlers in Express, but they differ in scope and HTTP method matching.

---

## app.use()

Registers middleware that runs for ALL HTTP methods and any matching URL prefix.

```js
// Runs for every request
app.use((req, res, next) => {
  console.log("Middleware");
  next();
});

// Runs for every request starting with /api
app.use("/api", router);
```

---

## app.get()

Registers a route handler that runs ONLY for GET requests at the exact specified path.

```js
app.get("/users", (req, res) => {
  res.json({ message: "Users" });
});
```

---

## Comparison

| Feature | app.use() | app.get() |
|---------|-----------|-----------|
| HTTP Methods | All | GET only |
| Path Matching | Prefix | Exact |
| Purpose | Middleware | Route Handler |

---

## Interview Answer (Ready to Speak)

app.use() registers middleware that applies to all HTTP methods and any path that starts with the specified prefix. app.get() registers a route handler that only executes for HTTP GET requests at the exact path. app.use() is for middleware; app.get() is for defining GET routes.

---

## Quick Revision

```text
app.use() = All Methods, Prefix Match
app.get() = GET Only, Exact Match
app.use() = Middleware
app.get() = Route Handler
```

---

### 162. What is the difference between res.send() and res.json()?

## Definition

Both send responses to the client but handle content type differently.

---

## res.send()

Sends various types of responses: string, buffer, or object.

Automatically sets Content-Type based on what is passed.

```js
res.send("Hello");          // text/html
res.send({ key: "value" }); // application/json
res.send(Buffer.alloc(10)); // application/octet-stream
```

---

## res.json()

Always sends a JSON response with Content-Type: application/json.

Explicitly converts the value to JSON.

```js
res.json({ message: "Success" });
res.json([1, 2, 3]);
res.json(null);
```

---

## Key Difference

res.json() is more explicit and always ensures proper JSON serialization and headers.

```js
res.send(null);  // Sends empty body
res.json(null);  // Sends "null" as JSON
```

---

## Best Practice

For REST APIs, always use res.json() for data responses.

---

## Interview Answer (Ready to Speak)

res.send() can send any type of response and automatically determines the Content-Type. res.json() always sends a JSON response with the correct Content-Type header and explicitly serializes the value. For REST APIs, res.json() is preferred because it ensures consistent JSON handling.

---

## Quick Revision

```text
res.send() = Any Content Type
res.json() = Always JSON
res.json() Preferred For APIs
res.json() Sets Content-Type Header
```

---

### 163. What is MongoDB $push and $pull Operator?

## Definition

$push adds elements to an array field in a document. $pull removes elements from an array field that match a condition.

---

## $push

```js
// Add a product to user's wishlist
await User.findByIdAndUpdate(
  userId,
  { $push: { wishlist: productId } }
);
```

---

## $addToSet

Like $push but prevents duplicates.

```js
await User.findByIdAndUpdate(
  userId,
  { $addToSet: { wishlist: productId } }
);
```

---

## $pull

```js
// Remove a product from user's wishlist
await User.findByIdAndUpdate(
  userId,
  { $pull: { wishlist: productId } }
);
```

---

## $pop

Removes the first (-1) or last (1) element.

```js
{ $pop: { items: 1 } }  // Remove last
{ $pop: { items: -1 } } // Remove first
```

---

## Real-World Use Case: Cart

```js
// Add to cart
await Cart.findByIdAndUpdate(
  cartId,
  { $push: { items: { product: productId, qty: 1 } } }
);

// Remove from cart
await Cart.findByIdAndUpdate(
  cartId,
  { $pull: { items: { product: productId } } }
);
```

---

## Interview Answer (Ready to Speak)

$push adds elements to array fields while $pull removes elements matching a condition. $addToSet is like $push but prevents duplicate values. These operators enable efficient array manipulation in MongoDB without fetching and re-saving the entire document.

---

## Quick Revision

```text
$push = Add To Array
$pull = Remove From Array
$addToSet = Add Without Duplicate
$pop = Remove First or Last
Array Field Operations
```

---

### 164. What is the $inc Operator in MongoDB?

## Definition

The $inc operator increments or decrements a numeric field in a MongoDB document by a specified amount.

---

## Syntax

```js
{ $inc: { field: amount } }
```

Positive amount = increment.

Negative amount = decrement.

---

## Example

```js
// Increment product views by 1
await Product.findByIdAndUpdate(
  productId,
  { $inc: { views: 1 } }
);

// Decrement stock by 5
await Product.findByIdAndUpdate(
  productId,
  { $inc: { stock: -5 } }
);
```

---

## Why Use $inc?

Without $inc:

```js
// 1. Read document
const product = await Product.findById(id);
// 2. Calculate
const newViews = product.views + 1;
// 3. Update
await Product.findByIdAndUpdate(id, { views: newViews });
```

Three operations. Risk of race conditions.

With $inc:

```js
// One atomic operation
await Product.findByIdAndUpdate(id, { $inc: { views: 1 } });
```

---

## Benefits

### Atomic

No race conditions.

### Single Operation

Faster than read-then-update.

---

## Interview Answer (Ready to Speak)

The $inc operator atomically increments or decrements a numeric field. It is safer and more efficient than reading a value and updating it separately, because it eliminates the race condition between the read and write operations.

---

## Quick Revision

```text
$inc = Increment/Decrement
Atomic Operation
$inc: { field: 1 }
$inc: { field: -1 }
Race Condition Safe
```

---

### 165. What is the $set and $unset Operator in MongoDB?

## Definition

$set updates the value of specific fields without affecting other fields. $unset removes fields from a document.

---

## $set

```js
// Update only the role field
await User.findByIdAndUpdate(
  userId,
  { $set: { role: "admin" } }
);
```

Other fields remain untouched.

---

## Without $set (Danger)

```js
// This REPLACES the entire document!
await User.findByIdAndUpdate(userId, { role: "admin" });
```

All other fields are removed.

---

## $unset

Removes a field from the document.

```js
// Remove the phone field
await User.findByIdAndUpdate(
  userId,
  { $unset: { phone: "" } }
);
```

The value passed to $unset does not matter; the field is removed regardless.

---

## Multiple Fields

```js
await User.findByIdAndUpdate(
  userId,
  {
    $set: {
      name: "Yogesh Patel",
      role: "admin"
    }
  },
  { new: true }
);
```

---

## Interview Answer (Ready to Speak)

$set updates specific fields in a document while leaving all other fields intact. $unset removes specified fields from a document. Always use $set when updating specific fields to avoid accidentally overwriting the entire document.

---

## Quick Revision

```text
$set = Update Specific Fields
$unset = Remove Fields
$set Preserves Other Fields
Always Use $set For Partial Update
findByIdAndUpdate Uses $set
```

---

### 166. What is MongoDB $match Stage?

## Definition

$match is an aggregation pipeline stage that filters documents to only pass those matching the specified condition, similar to find() or SQL's WHERE clause.

---

## Syntax

```js
{ $match: { condition } }
```

---

## Examples

```js
// Only completed orders
{ $match: { status: "completed" } }

// Orders above 1000
{ $match: { amount: { $gt: 1000 } } }

// Multiple conditions
{
  $match: {
    status: "completed",
    amount: { $gte: 500 }
  }
}
```

---

## Best Practice: $match First

Always place $match early in the pipeline to reduce the number of documents processed by subsequent stages.

```js
const result = await Order.aggregate([
  { $match: { status: "completed" } },  // ← Early filter
  { $group: { _id: "$userId", total: { $sum: "$amount" } } },
  { $sort: { total: -1 } }
]);
```

---

## With Index

$match can use indexes for better performance, but only when it is the first stage.

---

## Interview Answer (Ready to Speak)

The $match stage filters documents in an aggregation pipeline based on specified conditions. It should be placed as early as possible in the pipeline to reduce the number of documents processed by subsequent stages, improving performance. It supports all MongoDB query operators.

---

## Quick Revision

```text
Filter Documents
Like find() / WHERE
Place Early In Pipeline
Supports All Operators
Uses Indexes If First Stage
```

---

### 167. What is MongoDB $sort Stage?

## Definition

The $sort stage in the aggregation pipeline sorts documents by one or more fields.

---

## Syntax

```js
{ $sort: { field: 1 } }   // Ascending
{ $sort: { field: -1 } }  // Descending
```

---

## Example

```js
const result = await Product.aggregate([
  { $match: { inStock: true } },
  { $sort: { price: 1 } }  // Lowest price first
]);
```

---

## Multiple Sort Fields

```js
{
  $sort: {
    category: 1,
    price: -1
  }
}
```

Sort by category ascending, then price descending.

---

## After $group

```js
const result = await Order.aggregate([
  {
    $group: {
      _id: "$category",
      total: { $sum: "$amount" }
    }
  },
  { $sort: { total: -1 } }  // Highest revenue first
]);
```

---

## Interview Answer (Ready to Speak)

The $sort stage organizes aggregation pipeline documents in ascending (1) or descending (-1) order by specified fields. It is commonly placed after $group to rank results or before $limit to get top N records.

---

## Quick Revision

```text
1 = Ascending
-1 = Descending
After $group
Before $limit
Multiple Fields Supported
```

---

### 168. What is the $limit and $skip Stage?

## Definition

$limit restricts the number of documents passed to the next pipeline stage. $skip skips a specified number of documents from the beginning of the result.

---

## $limit

```js
{ $limit: 5 }
```

Returns only the first 5 documents.

---

## $skip

```js
{ $skip: 10 }
```

Skips the first 10 documents.

---

## Pagination With $skip and $limit

```js
const page = 2;
const limit = 10;
const skip = (page - 1) * limit; // 10

const result = await Product.aggregate([
  { $sort: { createdAt: -1 } },
  { $skip: skip },
  { $limit: limit }
]);
```

---

## Interview Answer (Ready to Speak)

$limit restricts the number of documents returned in an aggregation pipeline, while $skip bypasses a number of documents from the front. Used together with $sort, they implement pagination in aggregation queries.

---

## Quick Revision

```text
$limit = Max Documents
$skip = Skip First N
Combined = Pagination
$sort Before $skip
(page-1)*limit = skip value
```

---

### 169. What is the difference between Mongoose save() and create()?

## Definition

Both save documents to MongoDB, but they work differently.

---

## save()

Called on a Mongoose Document instance.

```js
const user = new User({
  name: "Yogesh",
  email: "yogesh@example.com"
});

await user.save();
```

Runs all schema validations and pre-save middleware.

---

## create()

A static method on the Model that creates and saves in one step.

```js
const user = await User.create({
  name: "Yogesh",
  email: "yogesh@example.com"
});
```

Internally calls new Model(data).save().

---

## Difference

| Feature | save() | create() |
|---------|--------|---------|
| Instance Needed | Yes | No |
| Steps | Two (new + save) | One |
| Bulk Insert | No | Yes (array) |
| Hooks | Yes | Yes |

---

## Bulk Insert With create()

```js
await User.create([
  { name: "Yogesh" },
  { name: "Raj" },
  { name: "Priya" }
]);
```

---

## Interview Answer (Ready to Speak)

save() is called on a Mongoose Document instance and is useful when you need to modify a document before saving. create() is a Model method that combines instantiation and saving in a single step and is preferred for straightforward insertions.

---

## Quick Revision

```text
save() = Instance Method
create() = Model Method
create() = One Step
create() Supports Array
Both Trigger Hooks
```

---

### 170. What is Node.js Child Process Module?

## Definition

The child_process module allows Node.js to spawn child processes, enabling the execution of external commands, scripts, or other Node.js files in separate processes.

---

## Why Do We Need It?

For CPU-intensive tasks like:

- Image compression
- PDF generation
- Running shell commands
- Database backups

We spawn a child process so the main Event Loop is not blocked.

---

## Methods

### exec()

Runs a shell command.

```js
const { exec } = require("child_process");

exec("ls -la", (err, stdout, stderr) => {
  if (err) console.error(err);
  console.log(stdout);
});
```

---

### spawn()

Spawns a new process. Better for large output.

```js
const { spawn } = require("child_process");

const ls = spawn("ls", ["-la"]);

ls.stdout.on("data", data => {
  console.log(data.toString());
});
```

---

### fork()

Creates a child Node.js process.

```js
const { fork } = require("child_process");

const child = fork("./worker.js");

child.on("message", msg => {
  console.log(msg);
});

child.send("Start");
```

---

## exec vs spawn

| Feature | exec | spawn |
|---------|------|-------|
| Output Buffering | Yes (limited size) | No (streaming) |
| Large Output | No | Yes |
| Shell Support | Yes | No (by default) |

---

## Interview Answer (Ready to Speak)

The child_process module allows Node.js to create child processes for running shell commands or CPU-intensive tasks without blocking the Event Loop. exec() is suitable for short commands, spawn() for long-running processes with large output, and fork() for spawning separate Node.js processes.

---

## Quick Revision

```text
exec() = Shell Commands
spawn() = Large Output
fork() = Node.js Child
Separate Process
Non-Blocking Main Thread
```

---

### 171. What is the difference between req.headers and res.setHeader()?

## Definition

req.headers reads the HTTP headers sent by the client. res.setHeader() sets HTTP response headers that the server sends back to the client.

---

## req.headers

Read-only. Contains the headers sent in the incoming request.

```js
// Client sends Authorization header
const token = req.headers.authorization;
const contentType = req.headers["content-type"];
```

---

## res.setHeader()

Sets a response header before sending the response.

```js
res.setHeader("Content-Type", "application/json");
res.setHeader("X-Custom-Header", "value");
```

---

## res.set() (Alias)

Express shorthand for setHeader.

```js
res.set("Content-Type", "application/json");

// Multiple headers
res.set({
  "Content-Type": "application/json",
  "X-Powered-By": "Node.js"
});
```

---

## Common Response Headers

```text
Content-Type
Authorization
Cache-Control
Set-Cookie
Access-Control-Allow-Origin
```

---

## Interview Answer (Ready to Speak)

req.headers contains the HTTP headers received from the client and is read-only. res.setHeader() is used to set HTTP headers on the server's response before it is sent. In Express, res.set() is a convenient alias that supports setting multiple headers at once.

---

## Quick Revision

```text
req.headers = Read Client Headers
res.setHeader() = Set Response Headers
res.set() = Express Alias
Read vs Write
Headers Sent Before Body
```

---

### 172. What is the difference between findByIdAndUpdate() with and without $set?

## Definition

Using findByIdAndUpdate() without $set replaces the entire document. Using $set updates only the specified fields.

---

## Without $set — Dangerous

```js
await User.findByIdAndUpdate(
  id,
  req.body  // Replaces entire document!
);
```

If req.body is { name: "Yogesh" }, ALL other fields are removed.

---

## With $set — Safe

```js
await User.findByIdAndUpdate(
  id,
  { $set: req.body },  // Only updates provided fields
  { new: true }
);
```

---

## Internal Difference

Without $set:

```text
MongoDB treats update as replacement.
```

With $set:

```text
MongoDB treats update as field update.
```

---

## Best Practice for REST API PATCH

```js
app.patch("/users/:id", async (req, res) => {
  const user = await User.findByIdAndUpdate(
    req.params.id,
    { $set: req.body },
    { new: true, runValidators: true }
  );
  res.json(user);
});
```

---

## runValidators

Runs schema validators on the update.

---

## Interview Answer (Ready to Speak)

Without $set, findByIdAndUpdate() can replace the entire document if the update object doesn't contain MongoDB update operators. Using $set ensures only the specified fields are modified. The { new: true } option returns the updated document, and runValidators: true ensures validators run on update.

---

## Quick Revision

```text
Without $set = Document Replacement Risk
With $set = Safe Partial Update
{ new: true } = Return Updated Doc
runValidators: true = Validate On Update
Always Use $set For Safety
```

---

### 173. What is the populate() Method With Select?

## Definition

The populate() method can be combined with select to retrieve only specific fields from the referenced document, improving performance.

---

## Basic Populate

```js
const orders = await Order
  .find()
  .populate("user");
```

Returns all fields of the user document.

---

## Populate With Select

```js
const orders = await Order
  .find()
  .populate("user", "name email");
```

Returns only name and email from the user.

---

## Exclude Fields In Populate

```js
const orders = await Order
  .find()
  .populate("user", "-password -__v");
```

---

## Populate Multiple Fields

```js
const orders = await Order
  .find()
  .populate("user", "name")
  .populate("product", "name price");
```

---

## Nested Populate

```js
const orders = await Order
  .find()
  .populate({
    path: "product",
    populate: {
      path: "category",
      select: "name"
    }
  });
```

---

## Benefits

### Performance

Fewer fields transferred from database.

### Security

Exclude sensitive fields from responses.

---

## Interview Answer (Ready to Speak)

The populate() method can accept a second argument to select specific fields from the referenced document. This reduces data transfer and prevents sensitive fields from being included. Multiple populations and nested populations are also supported for complex data relationships.

---

## Quick Revision

```text
.populate("field", "select fields")
"-field" = Exclude
Multiple .populate() Calls
Nested populate via path option
Performance + Security
```

---

### 174. What is Error Propagation in Express?

## Definition

Error propagation in Express is the process of passing errors from route handlers or middleware to the centralized error-handling middleware using next(err).

---

## Without Error Propagation

```js
app.get("/users", async (req, res) => {
  const users = await User.find(); // If this throws, server crashes
  res.json(users);
});
```

---

## With Error Propagation

```js
app.get("/users", async (req, res, next) => {
  try {
    const users = await User.find();
    res.json(users);
  } catch (err) {
    next(err); // Pass to error middleware
  }
});
```

---

## Centralized Handler

```js
app.use((err, req, res, next) => {
  res.status(err.statusCode || 500).json({
    success: false,
    message: err.message || "Server Error"
  });
});
```

---

## Using asyncHandler Wrapper

Many developers use a wrapper to avoid try-catch in every route:

```js
const asyncHandler = (fn) => {
  return (req, res, next) => {
    Promise.resolve(fn(req, res, next))
      .catch(next);
  };
};

// Usage
app.get("/users", asyncHandler(async (req, res) => {
  const users = await User.find();
  res.json(users);
}));
```

---

## Interview Answer (Ready to Speak)

Error propagation in Express passes errors through the middleware chain to the centralized error handler using next(err). The asyncHandler pattern wraps async route functions to automatically catch rejected promises and forward them to next(). This eliminates repetitive try-catch blocks in every route.

---

## Quick Revision

```text
next(err) = Propagate Error
Centralized Handler
asyncHandler Wrapper
Promise.resolve().catch(next)
Avoids Server Crashes
```

---

### 175. What is the Application Startup Pattern in Node.js?

## Definition

The application startup pattern defines the order in which an Express application initializes: configuration loading, database connection, middleware setup, routing, and server start.

---

## Recommended Startup Order

```js
// 1. Load Environment Variables
require("dotenv").config();

// 2. Import Modules
const express = require("express");
const mongoose = require("mongoose");
const cors = require("cors");
const helmet = require("helmet");
const morgan = require("morgan");

// 3. Create App
const app = express();

// 4. Security Middleware
app.use(helmet());
app.use(cors({ origin: process.env.CLIENT_URL }));

// 5. Logging
app.use(morgan("dev"));

// 6. Body Parsing
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// 7. Routes
const userRoutes = require("./routes/userRoutes");
app.use("/api/v1/users", userRoutes);

// 8. Error Handling
app.use((err, req, res, next) => {
  res.status(err.statusCode || 500).json({
    message: err.message || "Server Error"
  });
});

// 9. Connect To Database and Start Server
const startServer = async () => {
  await mongoose.connect(process.env.MONGO_URI);
  console.log("Database Connected");

  app.listen(process.env.PORT || 3000, () => {
    console.log("Server Running");
  });
};

startServer();
```

---

## Why This Order?

Security middleware must come before routes.

Body parsing middleware must come before routes that read req.body.

Error middleware must come after all routes.

---

## Interview Answer (Ready to Speak)

The application startup pattern in Node.js follows a specific order: environment setup, middleware (security first, then body parsing), routing, error handling, and finally database connection followed by server start. This order ensures security is applied before any requests are processed and errors are caught by the centralized handler.

---

## Quick Revision

```text
dotenv First
Security Middleware Before Routes
Body Parsing Before Routes
Routes
Error Handler Last
DB Connect Then Listen
```

---

### 176. What is BSON?

## Definition

BSON (Binary JSON) is the binary-encoded serialization format used by MongoDB to store documents. It extends JSON by adding additional data types.

---

## JSON vs BSON

| Feature | JSON | BSON |
|---------|------|------|
| Format | Text | Binary |
| Readable | Yes | No (binary) |
| Performance | Slower | Faster |
| Data Types | Limited | Extended |

---

## Additional BSON Data Types

### ObjectId

```text
_id: ObjectId("65f1a2b3c4d5e6f7a8b9c0d1")
```

---

### Date

```text
createdAt: ISODate("2024-01-01T00:00:00Z")
```

---

### Int32 / Int64

Explicit integer sizes.

---

### Binary

For binary data.

---

### Decimal128

High-precision decimal for financial data.

---

## ObjectId

MongoDB's default primary key.

12-byte unique identifier:

```text
4 bytes  = Timestamp
5 bytes  = Random Machine ID
3 bytes  = Incrementing Counter
```

---

## Interview Answer (Ready to Speak)

BSON is MongoDB's binary storage format that extends JSON with additional data types such as ObjectId, Date, Int32, Int64, and Decimal128. It is more compact and efficient to parse than JSON, enabling faster reads and writes. MongoDB uses BSON internally but exposes data in JSON format through drivers.

---

## Quick Revision

```text
Binary JSON
ObjectId, Date, Int32
Faster Than JSON
MongoDB Internal Format
Extended Data Types
```

---

### 177. What is the difference between MongoDB findOne() and find()?

## Definition

find() returns a cursor (array of documents) while findOne() returns a single document or null.

---

## find()

Returns all matching documents as an array.

```js
const products = await Product.find({ category: "Electronics" });
// Returns: [ {doc1}, {doc2}, ... ]
```

---

## findOne()

Returns the first matching document or null.

```js
const product = await Product.findOne({ name: "Laptop" });
// Returns: {doc} or null
```

---

## Comparison Table

| Feature | find() | findOne() |
|---------|--------|-----------|
| Returns | Array | Object or null |
| Multiple Docs | Yes | No |
| Speed | Slower (more docs) | Faster |
| Use Case | List All | Single Record |

---

## find() With Single Result

```js
const users = await User.find().limit(1);
const user = users[0];
```

vs more efficiently:

```js
const user = await User.findOne();
```

---

## Interview Answer (Ready to Speak)

find() returns an array of all matching documents, while findOne() returns only the first matching document as a plain object or null. findOne() is more efficient when you need a single document. find() is used when retrieving multiple records.

---

## Quick Revision

```text
find() = Array of Documents
findOne() = Single Document or null
findOne() = Faster For Single
find() = List Operations
Both Return null If Empty
```

---

### 178. What is Mongoose pre-save Middleware Use Cases?

## Definition

Pre-save middleware in Mongoose runs automatically before a document is saved, enabling automatic processing of data.

---

## Use Case 1: Password Hashing

```js
userSchema.pre("save", async function(next) {
  if (!this.isModified("password")) return next();

  this.password = await bcrypt.hash(this.password, 12);
  next();
});
```

Only hashes if password is new or modified.

---

## Use Case 2: Auto-Generate Slug

```js
productSchema.pre("save", function(next) {
  this.slug = this.name
    .toLowerCase()
    .replace(/\s+/g, "-");
  next();
});
```

---

## Use Case 3: Set Updated Timestamp

```js
userSchema.pre("save", function(next) {
  this.updatedAt = new Date();
  next();
});
```

---

## Use Case 4: Normalize Email

```js
userSchema.pre("save", function(next) {
  this.email = this.email.toLowerCase().trim();
  next();
});
```

---

## this.isModified()

Checks if a specific field was changed.

```js
if (!this.isModified("password")) return next();
```

This prevents re-hashing an already-hashed password.

---

## Interview Answer (Ready to Speak)

Pre-save middleware runs automatically before a Mongoose document is saved. Common use cases include password hashing, slug generation, email normalization, and automatic timestamp updates. The isModified() method checks if a field has changed, preventing unnecessary processing.

---

## Quick Revision

```text
pre("save") = Before Save
Password Hashing
Slug Generation
isModified("field")
Automatic Data Processing
```

---

### 179. What is MongoDB Compass?

## Definition

MongoDB Compass is the official GUI (Graphical User Interface) for MongoDB that allows developers to visually explore, query, and manage databases without writing command-line queries.

---

## Key Features

### Visual Query Builder

Build queries without writing code.

---

### Document Explorer

Browse collections and documents visually.

---

### Aggregation Builder

Build aggregation pipelines step by step.

---

### Index Management

View and create indexes visually.

---

### Schema Analysis

Understand field types and distributions.

---

### Performance Insights

Query execution plan visualization.

---

## Connection

Connect using a connection string:

```text
mongodb://localhost:27017
```

or MongoDB Atlas connection string.

---

## Benefits

### No Shell Required

### Visual Query Building

### Easy Database Management

### Team-Friendly

---

## Interview Answer (Ready to Speak)

MongoDB Compass is the official MongoDB GUI tool that allows developers to visually explore, query, and manage their databases. It features a document browser, visual query builder, aggregation pipeline editor, and index management. It is especially useful for developers learning MongoDB or inspecting production data.

---

## Quick Revision

```text
Official MongoDB GUI
Visual Query Builder
Aggregation Pipeline Editor
Schema Analysis
Index Management
```

---

### 180. What is Mongoose populate() vs $lookup?

## Definition

Both are used to join data from different collections, but they work at different levels.

---

## populate()

A Mongoose application-level operation.

Makes multiple database queries (one per reference).

```js
const orders = await Order.find()
  .populate("user", "name email");
```

Internally makes two queries:

1. Find orders
2. Find users by IDs

---

## $lookup

A MongoDB server-side aggregation stage.

Performs the join in a single database query on the server.

```js
const orders = await Order.aggregate([
  {
    $lookup: {
      from: "users",
      localField: "userId",
      foreignField: "_id",
      as: "user"
    }
  }
]);
```

---

## Comparison

| Feature | populate() | $lookup |
|---------|-----------|---------|
| Level | Application | Database |
| Queries | Multiple | Single |
| Flexibility | Less | More |
| Performance | Moderate | Better For Large Data |
| Syntax | Simpler | More Complex |

---

## When To Use Each

### populate()

Simple reference lookups. Small to medium datasets.

---

### $lookup

Complex reporting. Large datasets. When using aggregation pipeline.

---

## Interview Answer (Ready to Speak)

populate() is a Mongoose-level operation that makes multiple queries to replace ObjectId references with documents. $lookup is a MongoDB server-side aggregation stage that performs the join in a single query. $lookup is more efficient for large datasets and complex reporting, while populate() is simpler for straightforward use cases.

---

## Quick Revision

```text
populate() = Application Level
$lookup = Database Level
populate() = Multiple Queries
$lookup = Single Query
$lookup Better For Large Data
```

---

### 181. What is cookie-parser Middleware?

## Definition

cookie-parser is an Express middleware that parses the Cookie header from incoming requests and makes cookies accessible via req.cookies.

---

## Installing

```bash
npm install cookie-parser
```

---

## Usage

```js
const cookieParser = require("cookie-parser");
app.use(cookieParser());
```

---

## Setting A Cookie

```js
res.cookie("token", "jwt-value", {
  httpOnly: true,
  secure: true,
  maxAge: 7 * 24 * 60 * 60 * 1000 // 7 days
});
```

---

## Reading A Cookie

```js
const token = req.cookies.token;
console.log(token);
```

---

## Clearing A Cookie

```js
res.clearCookie("token");
```

---

## Cookie Security Options

### httpOnly

Prevents JavaScript access (XSS protection).

### secure

Only sent over HTTPS.

### sameSite

CSRF protection.

```js
res.cookie("token", value, {
  httpOnly: true,
  secure: process.env.NODE_ENV === "production",
  sameSite: "strict"
});
```

---

## Interview Answer (Ready to Speak)

cookie-parser is middleware that parses the Cookie header and populates req.cookies. Cookies are commonly used to store refresh tokens securely using httpOnly and secure flags, which prevent JavaScript access and ensure transmission only over HTTPS.

---

## Quick Revision

```text
npm install cookie-parser
app.use(cookieParser())
req.cookies.name
httpOnly + secure = Safe Storage
res.clearCookie() = Logout
```

---

### 182. What is multer Middleware?

## Definition

Multer is an Express middleware for handling multipart/form-data, which is the format used for file uploads.

---

## Installing

```bash
npm install multer
```

---

## Basic Setup

```js
const multer = require("multer");
const path = require("path");

const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, "uploads/");
  },
  filename: (req, file, cb) => {
    const uniqueName = Date.now() + path.extname(file.originalname);
    cb(null, uniqueName);
  }
});

const upload = multer({ storage });
```

---

## Single File Upload

```js
app.post(
  "/upload",
  upload.single("photo"),
  (req, res) => {
    console.log(req.file);
    res.json({ path: req.file.path });
  }
);
```

---

## Multiple Files

```js
upload.array("photos", 5)  // Max 5 files
```

---

## File Filter (Validation)

```js
const fileFilter = (req, file, cb) => {
  const allowed = ["image/jpeg", "image/png", "image/webp"];

  if (allowed.includes(file.mimetype)) {
    cb(null, true);
  } else {
    cb(new Error("Only images allowed"), false);
  }
};

const upload = multer({ storage, fileFilter });
```

---

## Interview Answer (Ready to Speak)

Multer is Express middleware for handling file uploads using multipart/form-data. It provides diskStorage and memoryStorage options, supports single and multiple file uploads, and allows file filtering by MIME type. Uploaded files are accessible via req.file or req.files.

---

## Quick Revision

```text
npm install multer
diskStorage
upload.single()
upload.array()
req.file / req.files
File Type Validation
```

---

### 183. What is the difference between authentication and session management?

## Definition

Authentication is the process of verifying identity. Session management is the process of maintaining that authenticated state across multiple requests.

---

## Authentication

Happens once per login.

```text
Verify credentials → Issue proof of identity
```

---

## Session Management

Happens on every subsequent request.

```text
Check proof of identity → Allow or deny access
```

---

## Session Management Approaches

### JWT-Based

State stored in the token (stateless).

```text
Client Stores JWT
Sends With Every Request
Server Verifies Without Database Lookup
```

---

### Session-Based

State stored on the server.

```text
Session Created In Database/Memory
Session ID Sent To Client As Cookie
Server Looks Up Session Every Request
```

---

## Comparison

| Feature | JWT | Sessions |
|---------|-----|----------|
| State Storage | Client-Side | Server-Side |
| Scalability | High | Lower |
| Revocation | Hard | Easy |
| Storage | localStorage/Cookie | Server Memory/DB |

---

## Interview Answer (Ready to Speak)

Authentication verifies identity once at login. Session management maintains that identity across requests. JWT-based systems store state in the token (stateless), while session-based systems store state on the server. JWT is more scalable but harder to revoke, while sessions are easier to invalidate but require server-side storage.

---

## Quick Revision

```text
Auth = Verify Identity Once
Session = Maintain Identity
JWT = Stateless
Sessions = Stateful
JWT = Hard To Revoke
Sessions = Easy To Revoke
```

---

### 184. What is load balancing in Node.js?

## Definition

Load balancing is the process of distributing incoming network traffic across multiple server instances to improve performance, reliability, and scalability.

---

## Why Is It Needed?

A single server has limits:

```text
CPU
Memory
Network Bandwidth
```

Load balancing distributes the work.

---

## Approaches

### Cluster Module (Single Machine)

```js
const cluster = require("cluster");
const os = require("os");

if (cluster.isMaster) {
  os.cpus().forEach(() => cluster.fork());
} else {
  app.listen(3000);
}
```

---

### Nginx Load Balancer (Multiple Machines)

```nginx
upstream backend {
  server server1:3000;
  server server2:3000;
  server server3:3000;
}

server {
  location / {
    proxy_pass http://backend;
  }
}
```

---

### PM2 (Process Manager)

```bash
pm2 start app.js -i max
```

Starts one instance per CPU core.

---

## Load Balancing Algorithms

### Round Robin

Requests distributed evenly.

---

### Least Connections

Request goes to the server with fewest active connections.

---

### IP Hash

Same client always goes to the same server.

---

## Interview Answer (Ready to Speak)

Load balancing distributes traffic across multiple server instances to improve performance and availability. Node.js supports it natively through the Cluster module on a single machine. For multi-server environments, Nginx or cloud load balancers distribute traffic. PM2 is a popular process manager that also provides clustering.

---

## Quick Revision

```text
Distribute Traffic
Cluster Module
Nginx
PM2
Round Robin, Least Connections
```

---

### 185. What is REST vs GraphQL?

## Definition

REST and GraphQL are two approaches for building APIs. REST uses fixed endpoints and HTTP methods, while GraphQL uses a single endpoint with flexible queries.

---

## REST

```text
GET /users         → All users
GET /users/:id     → One user
GET /users/:id/posts → User's posts
```

Multiple endpoints. Fixed response structure.

---

## GraphQL

```text
POST /graphql

Query: {
  user(id: "1") {
    name
    email
    posts {
      title
    }
  }
}
```

Single endpoint. Client specifies exactly what data is needed.

---

## Comparison Table

| Feature | REST | GraphQL |
|---------|------|---------|
| Endpoints | Multiple | Single |
| Response | Fixed | Flexible |
| Over-fetching | Common | Avoided |
| Under-fetching | Common | Avoided |
| Learning Curve | Low | Higher |
| Caching | Easy | Complex |

---

## Over-fetching in REST

```text
GET /users → Returns all 30 fields
But you only needed name and email
```

---

## Under-fetching in REST

```text
Need user + posts + comments
Requires 3 separate API calls
```

---

## GraphQL Solves Both

```graphql
query {
  user(id: "1") {
    name
    posts {
      title
      comments {
        text
      }
    }
  }
}
```

One request. Exactly the needed data.

---

## Interview Answer (Ready to Speak)

REST uses multiple fixed endpoints with predefined response structures, making it simple and widely supported. GraphQL uses a single endpoint where clients request exactly the data they need, solving over-fetching and under-fetching problems. REST is simpler to implement and cache; GraphQL is more flexible but has a steeper learning curve.

---

## Quick Revision

```text
REST = Multiple Endpoints
GraphQL = Single Endpoint
GraphQL = No Over/Under-fetching
REST = Simpler Caching
REST = Industry Standard
GraphQL = Flexible Queries
```

---

### 186. What is WebSocket and Socket.io?

## Definition

WebSocket is a protocol that provides full-duplex (bidirectional) communication channels over a single TCP connection. Socket.io is a library that simplifies WebSocket implementation in Node.js.

---

## Why WebSockets?

HTTP is request-response:

```text
Client → Request → Server → Response
```

One direction at a time.

WebSocket is bidirectional:

```text
Client ↔ Server (Both ways, anytime)
```

---

## Real-World Use Cases

- Chat Applications
- Live Notifications
- Online Games
- Stock Prices
- Collaborative Editing
- Live Sports Scores

---

## Socket.io Example

### Server

```bash
npm install socket.io
```

```js
const http = require("http");
const { Server } = require("socket.io");
const express = require("express");

const app = express();
const server = http.createServer(app);
const io = new Server(server);

io.on("connection", (socket) => {
  console.log("User connected:", socket.id);

  socket.on("message", (data) => {
    io.emit("message", data); // Broadcast to all
  });

  socket.on("disconnect", () => {
    console.log("User disconnected");
  });
});

server.listen(3000);
```

---

### Client

```html
<script src="/socket.io/socket.io.js"></script>
<script>
  const socket = io();
  socket.emit("message", "Hello!");
  socket.on("message", (data) => {
    console.log(data);
  });
</script>
```

---

## HTTP vs WebSocket

| Feature | HTTP | WebSocket |
|---------|------|-----------|
| Direction | Request-Response | Bidirectional |
| Connection | New Each Time | Persistent |
| Use Case | REST APIs | Real-Time |
| Overhead | Higher | Lower |

---

## Interview Answer (Ready to Speak)

WebSocket provides persistent bidirectional communication between client and server, unlike HTTP which requires a new connection for each request. Socket.io is a Node.js library that simplifies WebSocket implementation with fallback support and room-based messaging. It is the standard choice for real-time features like chat and notifications.

---

## Quick Revision

```text
WebSocket = Bidirectional
Persistent Connection
socket.io Library
io.emit() = Broadcast
Real-Time Applications
```

---

### 187. What is PM2?

## Definition

PM2 is a production process manager for Node.js applications that provides features like automatic restarts, clustering, log management, and monitoring.

---

## Installing PM2

```bash
npm install pm2 -g
```

---

## Starting Application

```bash
pm2 start app.js
```

---

## Cluster Mode (Use All CPU Cores)

```bash
pm2 start app.js -i max
```

---

## Restart On Crash

PM2 automatically restarts the application if it crashes.

---

## Startup Script

```bash
pm2 startup
pm2 save
```

Auto-restarts PM2 on server reboot.

---

## Useful Commands

```bash
pm2 list          # Show all processes
pm2 logs          # View logs
pm2 restart app   # Restart process
pm2 stop app      # Stop process
pm2 delete app    # Remove process
pm2 monit         # Real-time monitoring
```

---

## ecosystem.config.js

```js
module.exports = {
  apps: [{
    name: "my-app",
    script: "index.js",
    instances: "max",
    exec_mode: "cluster",
    env: {
      NODE_ENV: "development"
    },
    env_production: {
      NODE_ENV: "production"
    }
  }]
};
```

```bash
pm2 start ecosystem.config.js
```

---

## Benefits

### Automatic Restart

### Cluster Mode

### Log Management

### Monitoring

### Zero-Downtime Restart

---

## Interview Answer (Ready to Speak)

PM2 is a production process manager for Node.js that handles process lifecycle management. It automatically restarts applications on crash, provides cluster mode to utilize all CPU cores, manages logs, and offers real-time monitoring. It is the standard tool for deploying Node.js applications in production.

---

## Quick Revision

```text
Production Process Manager
pm2 start app.js
-i max = Cluster Mode
Auto Restart On Crash
pm2 monit = Monitoring
```

---

### 188. What is the difference between HTTP/1.1 and HTTP/2?

## Definition

HTTP/1.1 and HTTP/2 are different versions of the HTTP protocol with significant improvements in performance in HTTP/2.

---

## HTTP/1.1 Limitations

### Head-of-Line Blocking

Requests are processed one at a time per connection.

---

### Multiple Connections

Browsers open 6 connections per domain to workaround blocking.

---

### No Header Compression

Headers repeated in every request.

---

## HTTP/2 Improvements

### Multiplexing

Multiple requests over one connection simultaneously.

---

### Header Compression (HPACK)

Reduces header size significantly.

---

### Server Push

Server sends resources before client requests them.

---

### Binary Protocol

More efficient than text-based HTTP/1.1.

---

## Comparison

| Feature | HTTP/1.1 | HTTP/2 |
|---------|---------|--------|
| Multiplexing | No | Yes |
| Header Compression | No | Yes |
| Server Push | No | Yes |
| Protocol | Text | Binary |
| Connections | Multiple | Single |

---

## Node.js and HTTP/2

```js
const http2 = require("http2");
const fs = require("fs");

const server = http2.createSecureServer({
  key: fs.readFileSync("key.pem"),
  cert: fs.readFileSync("cert.pem")
});

server.on("stream", (stream, headers) => {
  stream.respond({ ":status": 200 });
  stream.end("Hello HTTP/2");
});

server.listen(443);
```

---

## Interview Answer (Ready to Speak)

HTTP/2 improves upon HTTP/1.1 by introducing multiplexing (multiple requests on one connection), header compression, server push, and a binary protocol. These improvements significantly reduce latency and improve page load performance. HTTP/2 requires HTTPS.

---

## Quick Revision

```text
HTTP/2 = Multiplexing
Header Compression
Server Push
Binary Protocol
Requires HTTPS
```

---

### 189. What is Caching in Node.js?

## Definition

Caching is storing the results of expensive operations in a faster storage layer so subsequent requests can be served quickly without recomputation.

---

## Why Cache?

Without caching:

```text
Every Request → Database Query
100 requests/sec → 100 DB queries/sec
```

With caching:

```text
First Request → DB Query → Store Result
Next 99 Requests → Serve From Cache
```

---

## Caching Levels

### In-Memory (Fastest)

```js
const cache = new Map();

app.get("/products", async (req, res) => {
  if (cache.has("products")) {
    return res.json(cache.get("products"));
  }

  const products = await Product.find();
  cache.set("products", products);

  res.json(products);
});
```

---

### Redis (Distributed Cache)

```bash
npm install ioredis
```

```js
const Redis = require("ioredis");
const redis = new Redis();

app.get("/products", async (req, res) => {
  const cached = await redis.get("products");

  if (cached) {
    return res.json(JSON.parse(cached));
  }

  const products = await Product.find();
  await redis.set("products", JSON.stringify(products), "EX", 3600);

  res.json(products);
});
```

EX 3600 = expires in 1 hour.

---

## Cache Invalidation

When data changes, the cache must be cleared:

```js
await redis.del("products");
```

---

## Benefits

### Faster Response Times

### Reduced Database Load

### Better Scalability

---

## Interview Answer (Ready to Speak)

Caching stores frequently accessed data in faster storage to reduce database queries. In Node.js, in-memory caching with Map or a distributed cache like Redis provides significant performance improvements. Cache invalidation must be handled when data changes to prevent stale responses.

---

## Quick Revision

```text
Store Computed Results
In-Memory = Map
Redis = Distributed
Cache Invalidation
EX = Expiry Time
```

---

### 190. What is the Full Backend Development Flow?

## Definition

The full backend development flow describes how a complete Node.js + Express + MongoDB backend application is built from project setup to production deployment.

---

## Step 1: Project Setup

```bash
mkdir my-api
cd my-api
npm init -y
npm install express mongoose dotenv bcrypt jsonwebtoken cors helmet morgan express-validator
npm install nodemon --save-dev
```

---

## Step 2: Folder Structure (MVC)

```text
my-api/
  controllers/
  models/
  routes/
  middleware/
  utils/
  config/
  .env
  .gitignore
  app.js
  index.js
```

---

## Step 3: Environment Configuration

```env
PORT=5000
MONGO_URI=mongodb://localhost:27017/mydb
JWT_SECRET=supersecret
JWT_EXPIRES_IN=15m
REFRESH_SECRET=refreshsecret
NODE_ENV=development
```

---

## Step 4: Database Connection

```js
// config/db.js
const mongoose = require("mongoose");

const connectDB = async () => {
  await mongoose.connect(process.env.MONGO_URI);
  console.log("MongoDB Connected");
};

module.exports = connectDB;
```

---

## Step 5: Models

```js
// models/userModel.js
const userSchema = new Schema({
  name: { type: String, required: true },
  email: { type: String, required: true, unique: true },
  password: { type: String, required: true, select: false },
  role: { type: String, enum: ["user", "admin"], default: "user" }
});

userSchema.pre("save", async function(next) {
  if (!this.isModified("password")) return next();
  this.password = await bcrypt.hash(this.password, 12);
  next();
});
```

---

## Step 6: Controllers

```js
// controllers/authController.js
const register = async (req, res, next) => {
  try {
    const user = await User.create(req.body);
    const token = jwt.sign({ id: user._id }, process.env.JWT_SECRET);
    res.status(201).json({ token });
  } catch (err) {
    next(err);
  }
};
```

---

## Step 7: Routes

```js
// routes/authRoutes.js
const router = express.Router();
router.post("/register", validate, register);
router.post("/login", validate, login);
module.exports = router;
```

---

## Step 8: Middleware Setup

```js
// app.js
app.use(helmet());
app.use(cors());
app.use(morgan("dev"));
app.use(express.json());
app.use("/api/v1/auth", authRoutes);
app.use(errorHandler);
```

---

## Step 9: Error Handler

```js
module.exports = (err, req, res, next) => {
  res.status(err.statusCode || 500).json({
    success: false,
    message: err.message || "Server Error"
  });
};
```

---

## Step 10: Start Server

```js
// index.js
const connectDB = require("./config/db");
connectDB().then(() => app.listen(5000));
```

---

## Step 11: Production

```bash
pm2 start ecosystem.config.js
```

---

## Interview Answer (Ready to Speak)

A complete Node.js backend follows the MVC pattern: environment setup with dotenv, database connection with Mongoose, schema-validated models, controllers for business logic, and routes with middleware for security, logging, and body parsing. Authentication uses bcrypt for password hashing and JWT for token issuance. The application is deployed using PM2 in cluster mode for production.

---

## Quick Revision

```text
Setup → MVC Structure → Environment
Models → Controllers → Routes
Middleware (Helmet, CORS, Morgan)
Auth (bcrypt + JWT)
Error Handler Last
PM2 For Production
```

---

*End of Node.js Interview Handbook — Part 3 (Q131–Q190)*

*This completes all 190 questions across all three parts.*
