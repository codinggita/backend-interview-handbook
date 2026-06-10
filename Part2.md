# Node.js Interview Handbook — Part 2 (Q78–Q130)

> Continue from Part 1. Same format, same depth.

---

### 78. What is Hashing? (Continued & Completed)

## Definition

Hashing is the process of converting data of any size into a fixed-length value called a hash (or digest) using a mathematical algorithm known as a hash function.

The output is always a fixed size regardless of the input size.

---

## Why Was Hashing Introduced?

Imagine a website storing passwords like this:

```text
User: Yogesh
Password: admin123
```

If the database gets leaked:

```text
Hackers can see all passwords.
```

This is extremely dangerous.

Instead, we store:

```text
Password
   ↓
Hash Function
   ↓
ef92b778bafe771e89245b89ecbc08a44a4e166c06659911881f383d4473e94f
```

Even if the database leaks, the actual passwords are never exposed.

---

## How Hashing Works

```text
Input: "password123"
          ↓
   Hash Function (SHA-256)
          ↓
Output: ef92b778bafe77...
```

The same input always produces the same output.

A different input always produces a completely different output.

---

## One-Way Property

Hashing is a one-way process.

```text
Input → Hash ✓
Hash → Input ✗ (Not Possible)
```

You cannot reverse-engineer the original value from a hash.

---

## Example in Node.js

```js
const crypto = require("crypto");

const hash = crypto
  .createHash("sha256")
  .update("password123")
  .digest("hex");

console.log(hash);
```

Output:

```text
ef92b778bafe771e89245b89ecbc08a44a4e166c06659911881f383d4473e94f
```

---

## Popular Hashing Algorithms

### MD5

Fast but not secure for passwords.

---

### SHA-256

Highly secure. Used in Bitcoin and TLS.

---

### SHA-512

Even more secure. Used in sensitive applications.

---

### bcrypt

Specifically designed for password hashing. Includes salting.

---

## Comparison Table

| Algorithm | Speed | Security | Use Case |
|-----------|-------|----------|----------|
| MD5 | Fast | Low | Checksums |
| SHA-256 | Medium | High | Data Integrity |
| bcrypt | Slow | Very High | Passwords |

---

## Real-World Use Cases

- Password Storage
- Data Integrity Verification
- File Checksums
- Digital Signatures
- Blockchain Transactions

---

## Interview Answer (Ready to Speak)

Hashing is a one-way mathematical process that converts data of any size into a fixed-length string called a hash. It is irreversible, meaning you cannot get the original input back from the hash. Node.js provides the crypto module for hashing. Popular algorithms include SHA-256 for data integrity and bcrypt for password storage.

---

## Follow-Up Questions

### Is hashing reversible?

No.

### Which algorithm is best for passwords?

bcrypt.

### What module provides hashing in Node.js?

crypto.

---

## Common Mistakes

❌ Hashing is the same as encryption.

❌ MD5 is secure enough for passwords.

---

## Quick Revision

```text
One-Way Process
Fixed-Length Output
SHA-256
bcrypt
crypto Module
```

---

### 79. What is Cluster Module?

## Definition

The Cluster module is a built-in Node.js module that allows you to create multiple child processes (workers) that share the same server port to take advantage of multi-core processors.

---

## Why Do We Need Cluster Module?

Node.js runs on a single thread by default.

```text
Single Thread
     ↓
Uses Only One CPU Core
```

A modern server may have:

```text
4 Cores
8 Cores
16 Cores
```

Without clustering:

```text
15 Cores Idle
1 Core Working
```

This wastes resources.

---

## How Cluster Module Works

```text
Master Process
      ↓

Fork Workers
      ↓

Worker 1  Worker 2  Worker 3  Worker 4
```

Each worker runs on a separate CPU core.

---

## Example

```js
const cluster = require("cluster");
const http = require("http");
const os = require("os");

if (cluster.isMaster) {

  const numCPUs = os.cpus().length;

  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

} else {

  http.createServer((req, res) => {
    res.end("Hello from Worker");
  }).listen(3000);

}
```

---

## Internal Flow

```text
Master Process
      ↓

Forks N Workers
      ↓

Each Worker Listens On Port 3000
      ↓

OS Distributes Requests
```

---

## Real-World Analogy

Imagine a restaurant.

Single Worker:

```text
1 Waiter
Serves All Tables
```

With Cluster:

```text
4 Waiters
Each Serves Some Tables
```

More customers can be served simultaneously.

---

## Benefits

### Better CPU Utilization

Uses all available cores.

### Higher Throughput

Handles more requests.

### Built-In Load Balancing

OS distributes connections across workers.

---

## Limitations

### No Shared Memory

Workers cannot share variables.

### Restart Logic Required

If a worker crashes, it must be restarted manually.

---

## Real-World Use Cases

- High-Traffic APIs
- Production Servers
- Scalable Applications

---

## Interview Answer (Ready to Speak)

The Cluster module in Node.js allows a single application to create multiple worker processes, one for each CPU core. The master process forks the workers, and the operating system distributes incoming connections among them. This improves throughput and makes full use of multi-core processors in production environments.

---

## Follow-Up Questions

### Why use Cluster module?

To utilize multiple CPU cores.

### What does the master process do?

It forks worker processes.

### Do workers share memory?

No.

---

## Common Mistakes

❌ Cluster creates threads, not processes.

❌ Workers share state automatically.

---

## Quick Revision

```text
Multi-Core Utilization
Master Process
Worker Processes
Fork
Load Balancing
```

---

### 80. What are Worker Threads?

## Definition

Worker Threads are a Node.js feature that allows JavaScript code to run in parallel on separate threads within the same process.

---

## Why Are Worker Threads Needed?

Node.js is great for I/O-intensive tasks.

But for CPU-intensive tasks:

```text
Image Processing
Video Encoding
Machine Learning
Encryption
```

A single JavaScript thread would block the Event Loop.

Worker Threads allow these tasks to run in the background without blocking the main thread.

---

## Worker Threads vs Cluster Module

| Feature | Cluster | Worker Threads |
|---------|---------|----------------|
| Process Type | Separate Processes | Threads In Same Process |
| Memory Sharing | No | Yes (SharedArrayBuffer) |
| Use Case | Network I/O | CPU-Intensive Tasks |
| Communication | IPC | MessageChannel |

---

## Example

```js
const {
  Worker,
  isMainThread,
  parentPort
} = require("worker_threads");

if (isMainThread) {

  const worker = new Worker(__filename);

  worker.on("message", msg => {
    console.log(msg);
  });

} else {

  parentPort.postMessage(
    "Hello from Worker"
  );

}
```

Output:

```text
Hello from Worker
```

---

## Internal Flow

```text
Main Thread
     ↓

Spawn Worker Thread
     ↓

Worker Runs CPU Task
     ↓

Result Sent Via Message
     ↓

Main Thread Continues
```

---

## Real-World Analogy

Imagine a research team.

```text
Manager (Main Thread)
     ↓

Assigns Heavy Research Tasks
     ↓

Researchers (Worker Threads)
     ↓

Results Submitted Back
```

The manager never stops working while researchers do the heavy lifting.

---

## Benefits

### Non-Blocking CPU Work

Heavy tasks do not block the Event Loop.

### Shared Memory Support

Using SharedArrayBuffer.

### Better Performance

For compute-heavy applications.

---

## Real-World Use Cases

- Image Processing
- Data Analysis
- Video Transcoding
- Cryptography
- Machine Learning

---

## Interview Answer (Ready to Speak)

Worker Threads are a Node.js feature that allows JavaScript to run CPU-intensive operations in parallel on separate threads without blocking the Event Loop. Unlike the Cluster module which creates separate processes, Worker Threads run within the same process and can optionally share memory using SharedArrayBuffer.

---

## Follow-Up Questions

### When should Worker Threads be used?

For CPU-intensive operations.

### Can Worker Threads share memory?

Yes, using SharedArrayBuffer.

### Is the main thread blocked by Worker Threads?

No.

---

## Common Mistakes

❌ Worker Threads are the same as Cluster.

❌ Worker Threads are needed for every async operation.

---

## Quick Revision

```text
CPU-Intensive Tasks
Separate Threads
Same Process
SharedArrayBuffer
Non-Blocking
```

---

### 81. What is npm?

## Definition

npm (Node Package Manager) is the default package manager for Node.js that allows developers to install, share, and manage JavaScript packages and dependencies.

---

## Why Was npm Created?

As Node.js applications grew:

```text
Every Developer
Writing Same Code
```

npm solved this by creating a central registry where packages could be published and reused.

---

## What Does npm Provide?

### Package Registry

A cloud database of over 2 million public packages.

---

### Command-Line Tool

```bash
npm install express
```

---

### Dependency Management

Manages project dependencies automatically.

---

## How npm Works

```text
Developer Needs Package
        ↓

npm install express
        ↓

Downloads From Registry
        ↓

Saved To node_modules
```

---

## Example

```bash
npm install express
```

This:

1. Downloads express package
2. Saves to node_modules/
3. Updates package.json
4. Updates package-lock.json

---

## Common npm Commands

### Initialize Project

```bash
npm init
```

---

### Install Package

```bash
npm install express
```

---

### Install Dev Dependency

```bash
npm install nodemon --save-dev
```

---

### Run Script

```bash
npm run start
```

---

### Uninstall Package

```bash
npm uninstall express
```

---

## Real-World Analogy

Imagine an app store.

```text
Browse Packages
Install With One Command
Use In Project
```

npm is the app store for Node.js developers.

---

## Benefits

### Huge Ecosystem

Millions of packages available.

### Easy Dependency Management

### Version Control

### Community Support

---

## Interview Answer (Ready to Speak)

npm is the Node Package Manager and the default package manager for Node.js. It provides access to a vast registry of reusable JavaScript packages and handles dependency installation, versioning, and project scripts. It is an essential tool for any Node.js developer.

---

## Follow-Up Questions

### What does npm stand for?

Node Package Manager.

### What is the npm registry?

A cloud database of all published packages.

### Where are packages installed?

node_modules folder.

---

## Common Mistakes

❌ npm is a programming language.

❌ npm only works for Node.js backend.

---

## Quick Revision

```text
Node Package Manager
Package Registry
Install Dependencies
node_modules
package.json
```

---

### 82. What is package.json?

## Definition

package.json is a JSON file in a Node.js project that contains metadata about the project and manages its dependencies, scripts, and configuration.

---

## Why Is It Important?

Without package.json:

```text
No Record Of Dependencies
No Project Scripts
No Version Information
```

Every collaborator would need to manually install packages.

---

## Creating package.json

```bash
npm init
```

or with defaults:

```bash
npm init -y
```

---

## Structure of package.json

```json
{
  "name": "my-app",
  "version": "1.0.0",
  "description": "My Node.js App",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  },
  "devDependencies": {
    "nodemon": "^3.0.1"
  }
}
```

---

## Key Fields

### name

Project name.

---

### version

Current version.

---

### scripts

Runnable commands.

---

### dependencies

Packages needed in production.

---

### devDependencies

Packages only needed in development.

---

## Running Scripts

```bash
npm run start
```

---

## Internal Flow

```text
npm install
     ↓

Reads package.json
     ↓

Downloads Dependencies
     ↓

Creates node_modules
```

---

## Benefits

### Project Documentation

### Reproducible Installs

### Script Management

### Dependency Tracking

---

## Interview Answer (Ready to Speak)

package.json is the configuration file of a Node.js project. It contains metadata such as the project name, version, description, and most importantly the list of dependencies and scripts. It allows developers to share and reproduce project environments consistently.

---

## Follow-Up Questions

### Which command creates package.json?

```bash
npm init
```

### What section defines run commands?

scripts.

### What is the difference between dependencies and devDependencies?

dependencies are needed in production; devDependencies are only needed during development.

---

## Common Mistakes

❌ Manually editing versions without understanding semver.

❌ Committing node_modules to version control.

---

## Quick Revision

```text
Project Metadata
Dependencies
Scripts
npm init
Version Information
```

---

### 83. What is package-lock.json?

## Definition

package-lock.json is an automatically generated file that records the exact versions of every installed dependency and their sub-dependencies.

---

## Why Do We Need It?

package.json uses version ranges like:

```json
"express": "^4.18.2"
```

This means install any version >= 4.18.2.

Over time, this may install different versions for different developers.

package-lock.json solves this by locking exact versions:

```json
"express": {
  "version": "4.18.2"
}
```

---

## Internal Flow

```text
npm install
     ↓

Reads package.json
     ↓

Resolves Exact Versions
     ↓

Creates package-lock.json
     ↓

Installs node_modules
```

---

## Benefits

### Reproducible Builds

Every team member installs identical packages.

### Faster Installs

npm uses lock file to skip version resolution.

### Security

Prevents unexpected version upgrades.

---

## Should It Be Committed?

Yes.

Commit package-lock.json to version control.

---

## package.json vs package-lock.json

| Feature | package.json | package-lock.json |
|---------|-------------|-------------------|
| Manually Edited | Yes | No |
| Contains Version Ranges | Yes | No (Exact) |
| Committed to Git | Yes | Yes |
| Purpose | Project Config | Exact Dependency Tree |

---

## Interview Answer (Ready to Speak)

package-lock.json is an automatically generated file created by npm when dependencies are installed. It locks the exact versions of every package and sub-dependency to ensure consistent, reproducible installations across all environments and team members.

---

## Follow-Up Questions

### Should package-lock.json be committed?

Yes.

### Who generates package-lock.json?

npm generates it automatically.

### Can we manually edit package-lock.json?

No. It should only be updated by npm.

---

## Common Mistakes

❌ Deleting package-lock.json.

❌ Not committing package-lock.json to version control.

---

## Quick Revision

```text
Locks Exact Versions
Auto-Generated
Reproducible Builds
Commit To Git
Dependency Tree
```

---

### 84. What is npx?

## Definition

npx is a tool that comes with npm and allows you to execute npm packages without permanently installing them.

---

## Why Was npx Created?

Sometimes you need a package only once.

Example:

```text
Create a React App
Run a CLI Tool
```

Without npx:

```bash
npm install -g create-react-app
create-react-app my-app
```

This installs globally permanently.

With npx:

```bash
npx create-react-app my-app
```

Downloads, runs, and discards the package.

---

## How npx Works

```text
npx create-react-app
        ↓

Check If Package Is Installed
        ↓

If Not: Download Temporarily
        ↓

Execute Package
        ↓

Discard If Temporary
```

---

## npm vs npx

| Feature | npm | npx |
|---------|-----|-----|
| Purpose | Install Packages | Execute Packages |
| Permanent Install | Yes | Optional |
| Runs CLI Tools | No | Yes |
| Common Usage | Dependencies | One-Time Tools |

---

## Common npx Examples

```bash
npx create-react-app my-app

npx nodemon index.js

npx eslint src/
```

---

## Benefits

### No Global Pollution

Packages are not permanently installed.

### Always Latest Version

Downloads the current version each time.

### Convenient

Single command to run any CLI tool.

---

## Interview Answer (Ready to Speak)

npx is a package runner tool bundled with npm that allows developers to execute npm packages without permanently installing them. It is commonly used for running one-time CLI tools like create-react-app, avoiding the need for global installations.

---

## Follow-Up Questions

### What is the difference between npm and npx?

npm installs packages; npx runs them.

### Does npx install permanently?

Not by default.

### What version of package does npx use?

The latest available version unless specified.

---

## Common Mistakes

❌ npx and npm are the same.

❌ npx requires a separate installation.

---

## Quick Revision

```text
Execute Packages
No Permanent Install
Latest Version
CLI Tools
Bundled With npm
```

---

### 85. What is Semantic Versioning (SemVer)?

## Definition

Semantic Versioning (SemVer) is a versioning standard where a version number consists of three parts: MAJOR.MINOR.PATCH.

---

## Version Format

```text
MAJOR.MINOR.PATCH

Example: 4.18.2
```

---

## MAJOR

```text
4.18.2
^
```

Breaking changes that are not backward compatible.

---

## MINOR

```text
4.18.2
  ^
```

New features added in a backward-compatible manner.

---

## PATCH

```text
4.18.2
     ^
```

Bug fixes that do not change functionality.

---

## Real-World Analogy

Imagine a car model:

```text
Generation.Year.Bugfix

5.2023.1 = 5th Gen, 2023 Model, First Patch
```

---

## Version Range Symbols in package.json

### ^ (Caret)

Allows MINOR and PATCH updates.

```json
"express": "^4.18.2"
```

Can install: 4.18.x or 4.19.x but NOT 5.x.x

---

### ~ (Tilde)

Allows only PATCH updates.

```json
"express": "~4.18.2"
```

Can install: 4.18.x only.

---

### No Symbol

Exact version only.

```json
"express": "4.18.2"
```

Installs exactly 4.18.2.

---

## Benefits

### Clear Communication

### Safe Updates

### Dependency Management

---

## Interview Answer (Ready to Speak)

Semantic Versioning is a standard that uses a three-part version number: MAJOR for breaking changes, MINOR for new backward-compatible features, and PATCH for bug fixes. In package.json, the caret (^) allows compatible upgrades while the tilde (~) restricts updates to patch-level changes only.

---

## Follow-Up Questions

### What does MAJOR version increment mean?

Breaking changes.

### What does the ^ symbol allow?

Minor and patch updates.

### What does ~ allow?

Patch updates only.

---

## Common Mistakes

❌ Incrementing MAJOR for every change.

❌ ^ and ~ are the same.

---

## Quick Revision

```text
MAJOR.MINOR.PATCH
^ = Minor + Patch
~ = Patch Only
Breaking Changes = MAJOR
Bug Fixes = PATCH
```

---

### 86. What are npm Scripts?

## Definition

npm Scripts are custom commands defined in the scripts section of package.json that can be executed using npm run.

---

## Why Are They Useful?

Instead of typing long commands every time:

```bash
node --watch --env-file=.env src/index.js
```

We define a script:

```json
"scripts": {
  "dev": "node --watch --env-file=.env src/index.js"
}
```

And run:

```bash
npm run dev
```

---

## Example package.json Scripts

```json
{
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js",
    "test": "jest",
    "build": "webpack"
  }
}
```

---

## Running Scripts

```bash
npm run start

npm run dev

npm run test
```

---

## Special Scripts

### start

```bash
npm start
```

Runs without the run keyword.

---

### test

```bash
npm test
```

Also runs without run.

---

## Lifecycle Scripts

```text
prestart → start → poststart
```

npm automatically runs pre and post hooks.

---

## Benefits

### Consistency

### Shorter Commands

### Team Productivity

### CI/CD Integration

---

## Real-World Use Cases

- Starting the Server
- Running Tests
- Building for Production
- Linting Code
- Database Migrations

---

## Interview Answer (Ready to Speak)

npm Scripts are commands defined in the scripts section of package.json and executed via npm run. They allow teams to standardize common tasks such as starting the server, running tests, and building the application, reducing the need to remember long or complex commands.

---

## Follow-Up Questions

### Which section defines scripts?

scripts in package.json.

### How to run a script?

```bash
npm run <script-name>
```

### Which scripts don't need "run"?

start and test.

---

## Common Mistakes

❌ Writing scripts outside package.json.

❌ Forgetting to define the start script.

---

## Quick Revision

```text
Defined In package.json
npm run <name>
start and test = No "run"
Lifecycle Scripts
Team Consistency
```

---

### 87. What is node_modules Folder?

## Definition

The node_modules folder is a directory where npm installs all the packages and their dependencies required by a project.

---

## Why Does It Exist?

When you run:

```bash
npm install express
```

npm downloads express and all its dependencies into:

```text
node_modules/
  express/
  body-parser/
  ...
```

---

## Internal Flow

```text
npm install
     ↓

Reads package.json
     ↓

Downloads Packages
     ↓

Saves To node_modules
```

---

## Why Should node_modules Not Be Pushed to GitHub?

### It Is Huge

Can be hundreds of megabytes or gigabytes.

---

### It Can Be Recreated

```bash
npm install
```

Regenerates it from package.json.

---

### Different Platforms

node_modules may differ between Windows and Linux.

---

## .gitignore

Add to .gitignore:

```text
node_modules/
```

---

## Regenerating node_modules

```bash
npm install
```

Recreates node_modules from package.json.

---

## Benefits

### Centralized Dependencies

All packages in one location.

### Easy Management

npm handles installation automatically.

---

## Interview Answer (Ready to Speak)

The node_modules folder is where npm stores all installed packages and their dependencies. It should not be committed to version control because it can be very large and can be regenerated using npm install. The package.json and package-lock.json files contain all the information needed to recreate it.

---

## Follow-Up Questions

### Should node_modules be committed?

No.

### How to recreate node_modules?

```bash
npm install
```

### Where is it added to ignore?

.gitignore.

---

## Common Mistakes

❌ Pushing node_modules to GitHub.

❌ Manually editing files inside node_modules.

---

## Quick Revision

```text
Installed Packages
Do Not Commit
.gitignore
Recreate With npm install
Large Directory
```

---

### 88. What is Express.js?

## Definition

Express.js is a minimal, fast, and unopinionated web framework for Node.js that simplifies building web applications and APIs.

---

## Why Was Express.js Created?

The built-in http module is powerful but low-level.

Without Express:

```js
const server = http.createServer(
  (req, res) => {
    if (req.url === "/users" && req.method === "GET") {
      res.end("Users");
    }
  }
);
```

Routing, middleware, and error handling must all be written manually.

With Express:

```js
app.get("/users", (req, res) => {
  res.json({ message: "Users" });
});
```

Far simpler and more readable.

---

## Core Features of Express

### Routing

Define paths and HTTP methods clearly.

---

### Middleware

Process requests in a pipeline.

---

### Request / Response Handling

Simplified req and res objects.

---

### Error Handling

Centralized error management.

---

## Installing Express

```bash
npm install express
```

---

## Basic Server Example

```js
const express = require("express");
const app = express();

app.get("/", (req, res) => {
  res.send("Hello Express");
});

app.listen(3000, () => {
  console.log("Server Running");
});
```

---

## Internal Flow

```text
HTTP Request
      ↓

Express Router
      ↓

Middleware Pipeline
      ↓

Route Handler
      ↓

Response Sent
```

---

## Real-World Analogy

Imagine building a house.

Without Express:

```text
Mix Cement
Lay Bricks
Wire Electricity
Manually
```

With Express:

```text
Prefabricated Modules
Quick Assembly
```

Express provides ready-made tools.

---

## Benefits

### Minimal and Flexible

### Fast Development

### Large Ecosystem

### Middleware Support

### Industry Standard

---

## Real-World Use Cases

- REST APIs
- Authentication Systems
- E-Commerce Backends
- Admin Dashboards
- Microservices

---

## Interview Answer (Ready to Speak)

Express.js is a minimal and flexible web framework for Node.js. It provides routing, middleware support, and simplified request-response handling, making it much easier to build APIs and web applications compared to using the raw http module. Express is the most widely used Node.js framework in production.

---

## Follow-Up Questions

### Is Express a core module?

No. It must be installed via npm.

### Which language powers Express?

JavaScript on Node.js.

### What is Express mainly used for?

Building REST APIs and web applications.

---

## Common Mistakes

❌ Express is a programming language.

❌ Express replaces Node.js.

---

## Quick Revision

```text
Web Framework
Simplified Routing
Middleware Support
REST APIs
npm install express
```

---

### 89. What is Routing in Express.js?

## Definition

Routing in Express.js is the process of defining how an application responds to client requests based on the URL path and HTTP method.

---

## Why Is Routing Important?

Every application has multiple features.

Examples:

```text
GET /products      → Fetch Products
POST /products     → Create Product
GET /users         → Fetch Users
DELETE /users/:id  → Delete User
```

Routing maps URLs to specific handler functions.

---

## Basic Routing

```js
const express = require("express");
const app = express();

app.get("/", (req, res) => {
  res.send("Home Page");
});

app.get("/about", (req, res) => {
  res.send("About Page");
});
```

---

## HTTP Method Routing

```js
app.get("/products", (req, res) => {
  res.send("Get Products");
});

app.post("/products", (req, res) => {
  res.send("Create Product");
});

app.put("/products/:id", (req, res) => {
  res.send("Update Product");
});

app.delete("/products/:id", (req, res) => {
  res.send("Delete Product");
});
```

---

## Route Parameters

```js
app.get("/users/:id", (req, res) => {
  const id = req.params.id;
  res.send(`User: ${id}`);
});
```

Request: `/users/42`

Output: `User: 42`

---

## Query Parameters

```js
app.get("/products", (req, res) => {
  const category = req.query.category;
  res.send(`Category: ${category}`);
});
```

Request: `/products?category=mobile`

---

## Internal Flow

```text
Request URL
      ↓

Match Route
      ↓

Execute Handler
      ↓

Send Response
```

---

## Benefits

### Clean Code Structure

### Easy URL Management

### Supports All HTTP Methods

### Scalable Application Design

---

## Interview Answer (Ready to Speak)

Routing in Express.js defines how the application handles incoming requests based on the URL path and HTTP method. Routes are defined using methods like app.get(), app.post(), app.put(), and app.delete(). Each route has a handler function that processes the request and sends a response.

---

## Follow-Up Questions

### How is a GET route defined?

```js
app.get("/path", handler)
```

### How do we access route parameters?

```js
req.params.id
```

### How do we access query parameters?

```js
req.query.key
```

---

## Common Mistakes

❌ Defining routes after app.listen().

❌ Forgetting to send a response.

---

## Quick Revision

```text
URL + Method = Route
app.get()
app.post()
req.params
req.query
```

---

### 90. What is Middleware in Express.js?

## Definition

Middleware in Express.js is a function that has access to the request object (req), the response object (res), and the next() function in the application's request-response cycle.

---

## Why Do We Need Middleware?

Many operations need to happen before reaching a route handler:

- Authentication Check
- Logging
- Request Body Parsing
- CORS Headers
- Rate Limiting

Middleware allows these operations to be centralized and reusable.

---

## How Middleware Works

```text
Request
    ↓

Middleware 1
    ↓

Middleware 2
    ↓

Route Handler
    ↓

Response
```

---

## Simple Example

```js
const express = require("express");
const app = express();

// Middleware
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();
});

// Route
app.get("/", (req, res) => {
  res.send("Hello");
});
```

---

## What Does next() Do?

```js
next()
```

Passes control to the next middleware or route handler.

Without calling next(), the request hangs.

---

## Types of Middleware

### Application-Level

```js
app.use(middleware)
```

---

### Router-Level

```js
router.use(middleware)
```

---

### Error-Handling

```js
app.use((err, req, res, next) => {})
```

---

### Built-In

```js
express.json()
express.urlencoded()
express.static()
```

---

### Third-Party

```js
cors
helmet
morgan
```

---

## Real-World Analogy

Imagine airport security.

```text
Check-in
    ↓

Baggage Scan
    ↓

Passport Check
    ↓

Board Flight
```

Each step is a middleware. If any step fails, you don't proceed.

---

## Benefits

### Reusable Logic

### Centralized Operations

### Clean Code

### Modular Design

---

## Interview Answer (Ready to Speak)

Middleware in Express.js is a function that intercepts requests before they reach route handlers. It can inspect, modify, or terminate requests. Middleware functions receive req, res, and next. Calling next() passes control to the following middleware. Common uses include authentication, logging, request parsing, and error handling.

---

## Follow-Up Questions

### What are the three parameters of middleware?

req, res, next.

### What happens if next() is not called?

The request hangs and no response is sent.

### Name three built-in middleware.

express.json(), express.urlencoded(), express.static().

---

## Common Mistakes

❌ Forgetting to call next().

❌ Using res.send() without ending the middleware.

---

## Quick Revision

```text
Request Interceptor
req, res, next
next() = Continue
Reusable
Authentication, Logging, Parsing
```

---

### 91. What is next() in Express.js?

## Definition

next() is a function provided by Express.js that passes control from the current middleware to the next middleware function or route handler.

---

## Why Is next() Important?

Without next():

```text
Request Gets Stuck
No Response Is Sent
Connection Times Out
```

next() keeps the request flowing through the middleware pipeline.

---

## Example

```js
app.use((req, res, next) => {
  console.log("Middleware 1");
  next();
});

app.use((req, res, next) => {
  console.log("Middleware 2");
  next();
});

app.get("/", (req, res) => {
  res.send("Done");
});
```

Output:

```text
Middleware 1
Middleware 2
Done
```

---

## Passing Errors

```js
next(new Error("Something Failed"));
```

This skips regular middleware and goes to the error-handling middleware.

---

## Error-Handling Middleware

```js
app.use((err, req, res, next) => {
  res.status(500).json({
    message: err.message
  });
});
```

---

## Internal Flow

```text
Middleware 1
     ↓
next()
     ↓
Middleware 2
     ↓
next()
     ↓
Route Handler
```

---

## Benefits

### Flow Control

### Error Propagation

### Reusable Middleware Chains

---

## Interview Answer (Ready to Speak)

next() is an Express.js function that passes control to the next middleware or route handler. It is essential for middleware chaining. If next() is not called, the request-response cycle ends and the client receives no response. Passing an error to next() routes it to the error-handling middleware.

---

## Follow-Up Questions

### What happens when next(err) is called?

It routes to the error-handling middleware.

### Is next() required in every middleware?

Only if there are more middleware or routes to execute.

---

## Common Mistakes

❌ Calling next() after sending a response.

❌ Forgetting next() in authentication middleware.

---

## Quick Revision

```text
Passes Control
Middleware Chaining
next(err) = Error Handler
Required For Flow
Express Core Concept
```

---

### 92. What is express.json() Middleware?

## Definition

express.json() is a built-in Express.js middleware that parses incoming requests with a JSON body and makes the parsed data available on req.body.

---

## Why Do We Need It?

When a client sends a POST request with JSON:

```json
{
  "name": "Yogesh",
  "email": "yogesh@example.com"
}
```

Without express.json():

```js
console.log(req.body);
// undefined
```

With express.json():

```js
console.log(req.body);
// { name: "Yogesh", email: "yogesh@example.com" }
```

---

## How to Use

```js
const express = require("express");
const app = express();

app.use(express.json());

app.post("/users", (req, res) => {
  console.log(req.body);
  res.json({ received: true });
});
```

---

## Internal Flow

```text
POST Request With JSON Body
           ↓

express.json() Middleware
           ↓

Parses JSON String
           ↓

Attaches To req.body
           ↓

Route Handler Accesses Data
```

---

## Required Header

The client must send:

```http
Content-Type: application/json
```

---

## Benefits

### Automatic JSON Parsing

### Clean Access Via req.body

### Built-In — No Extra Package Needed

---

## Real-World Use Cases

- Login API
- Registration API
- Data Creation Endpoints
- Any POST/PUT/PATCH Route

---

## Interview Answer (Ready to Speak)

express.json() is a built-in middleware in Express that parses incoming requests with JSON payloads. It reads the request body, parses it, and attaches the result to req.body. Without it, req.body would be undefined when clients send JSON data.

---

## Follow-Up Questions

### What does express.json() do?

Parses JSON request bodies.

### What does it attach data to?

req.body.

### Is it required for all routes?

Only for routes that receive JSON bodies.

---

## Common Mistakes

❌ Forgetting to add express.json() before routes.

❌ Using it without the correct Content-Type header.

---

## Quick Revision

```text
Built-In Middleware
Parses JSON Body
Attaches To req.body
Required For POST Requests
Content-Type: application/json
```

---

### 93. What is express.Router()?

## Definition

express.Router() is a mini Express application that provides routing capabilities. It is used to create modular, mountable route handlers.

---

## Why Do We Need express.Router()?

As applications grow:

```js
// app.js — 500 Lines
app.get("/products", handler);
app.post("/products", handler);
app.get("/users", handler);
app.post("/users", handler);
```

This becomes difficult to manage.

With Router:

```js
// routes/products.js
router.get("/", handler);
router.post("/", handler);

// routes/users.js
router.get("/", handler);
router.post("/", handler);
```

Clean and modular.

---

## Example

### routes/productRoutes.js

```js
const express = require("express");
const router = express.Router();

router.get("/", (req, res) => {
  res.json({ message: "All Products" });
});

router.post("/", (req, res) => {
  res.json({ message: "Product Created" });
});

module.exports = router;
```

---

### app.js

```js
const express = require("express");
const productRoutes = require("./routes/productRoutes");

const app = express();

app.use(express.json());
app.use("/products", productRoutes);

app.listen(3000);
```

---

## Internal Flow

```text
Request: GET /products
         ↓

app.use("/products", productRoutes)
         ↓

productRoutes.get("/")
         ↓

Handler Executes
```

---

## Benefits

### Modular Architecture

### Separation of Concerns

### Cleaner app.js

### Scalable Codebase

---

## Interview Answer (Ready to Speak)

express.Router() creates a modular route handler that can be mounted on a path in the main Express application. It helps organize routes into separate files, keeping the codebase clean and maintainable as the application scales.

---

## Follow-Up Questions

### What does express.Router() return?

A mini router object.

### How is it used in app.js?

```js
app.use("/path", router)
```

### Why use Router over defining all routes in app.js?

Modularity and maintainability.

---

## Common Mistakes

❌ Forgetting to export the router.

❌ Adding wrong base path in app.use().

---

## Quick Revision

```text
Modular Routes
Mini Express App
app.use()
Separate Files
Scalable Architecture
```

---

### 94. What are Route Parameters in Express.js?

## Definition

Route parameters are dynamic segments of a URL path defined with a colon (:) that capture values from the URL.

---

## Why Do We Need Route Parameters?

Instead of defining separate routes for every user:

```text
/users/1
/users/2
/users/3
...
```

We define one dynamic route:

```text
/users/:id
```

---

## Example

```js
app.get("/users/:id", (req, res) => {
  const id = req.params.id;
  res.json({ userId: id });
});
```

Request: `GET /users/42`

Response:

```json
{ "userId": "42" }
```

---

## Multiple Parameters

```js
app.get(
  "/users/:userId/orders/:orderId",
  (req, res) => {
    const { userId, orderId } = req.params;
    res.json({ userId, orderId });
  }
);
```

Request: `GET /users/5/orders/100`

---

## Route Params vs Query Params

| Feature | Route Params | Query Params |
|---------|-------------|--------------|
| Location | In URL Path | After ? |
| Example | /users/:id | /users?id=5 |
| Required | Yes | Optional |
| Purpose | Identify Resource | Filter / Sort |

---

## Internal Flow

```text
Request: /users/42
         ↓

Express Matches /users/:id
         ↓

req.params.id = "42"
         ↓

Handler Executes
```

---

## Benefits

### Dynamic Routes

### Cleaner URLs

### RESTful Design

---

## Interview Answer (Ready to Speak)

Route parameters are dynamic values embedded in URL paths, defined using a colon followed by the parameter name. They are accessed via req.params and are commonly used to identify specific resources like users, products, or orders in RESTful APIs.

---

## Follow-Up Questions

### How to define a route parameter?

```text
/path/:paramName
```

### How to access it?

```js
req.params.paramName
```

### Are route parameters required?

Yes, unlike query parameters.

---

## Common Mistakes

❌ Confusing route params with query params.

❌ Forgetting the colon before the parameter name.

---

## Quick Revision

```text
Dynamic URL Segment
:paramName
req.params
RESTful APIs
Required Values
```

---

### 95. What are Query Parameters in Express.js?

## Definition

Query parameters are optional key-value pairs appended to a URL after the ? symbol, used to filter, sort, or paginate data.

---

## Example URL

```text
/products?category=mobile&page=2
```

---

## Accessing Query Parameters

```js
app.get("/products", (req, res) => {
  const category = req.query.category;
  const page = req.query.page;

  res.json({ category, page });
});
```

Request: `/products?category=mobile&page=2`

Response:

```json
{
  "category": "mobile",
  "page": "2"
}
```

---

## Internal Flow

```text
URL: /products?category=mobile
          ↓

Express Parses Query String
          ↓

req.query.category = "mobile"
          ↓

Handler Uses Data
```

---

## Common Use Cases

- Filtering Results
- Sorting Results
- Pagination
- Search Queries

---

## Real-World Example

```js
app.get("/products", (req, res) => {
  const {
    category,
    sort,
    page = 1,
    limit = 10
  } = req.query;

  res.json({ category, sort, page, limit });
});
```

---

## Interview Answer (Ready to Speak)

Query parameters are optional key-value pairs included in a URL after the ? symbol. In Express.js, they are accessed via req.query. They are commonly used for filtering, sorting, searching, and paginating data in REST APIs.

---

## Follow-Up Questions

### How to access query parameters?

```js
req.query.key
```

### Are query parameters required?

No. They are optional.

### Example of use?

Pagination: `/products?page=2&limit=10`

---

## Common Mistakes

❌ Confusing query params with route params.

❌ Sending sensitive data in query params.

---

## Quick Revision

```text
Optional Key-Value Pairs
After ? In URL
req.query
Filtering, Sorting, Pagination
Not For Sensitive Data
```

---

### 96. What is Request Object (req) in Express?

## Definition

The request object (req) in Express.js is an enhanced version of Node.js's http.IncomingMessage that provides convenient methods and properties to access all data sent by the client.

---

## Common Properties

### req.body

Parsed request body (requires express.json()).

```js
console.log(req.body);
```

---

### req.params

Route parameters.

```js
console.log(req.params.id);
```

---

### req.query

Query string parameters.

```js
console.log(req.query.page);
```

---

### req.headers

HTTP request headers.

```js
console.log(req.headers.authorization);
```

---

### req.method

HTTP method.

```js
console.log(req.method); // GET, POST, etc.
```

---

### req.url

Request URL.

```js
console.log(req.url);
```

---

### req.ip

Client's IP address.

```js
console.log(req.ip);
```

---

## Example

```js
app.post("/login", (req, res) => {
  const { email, password } = req.body;
  const token = req.headers.authorization;
  const page = req.query.page;

  console.log(email, token, page);

  res.json({ success: true });
});
```

---

## Internal Flow

```text
Incoming HTTP Request
         ↓

Express Enhances Request Object
         ↓

req.body, req.params, req.query, etc.
         ↓

Route Handler Reads Data
```

---

## Interview Answer (Ready to Speak)

The request object in Express.js provides access to all data sent by the client. Key properties include req.body for the request body, req.params for route parameters, req.query for query strings, req.headers for HTTP headers, and req.method for the HTTP method used.

---

## Common Mistakes

❌ Accessing req.body without express.json().

❌ Confusing req.params with req.query.

---

## Quick Revision

```text
req.body = Request Body
req.params = URL Params
req.query = Query Strings
req.headers = Headers
req.method = HTTP Method
```

---

### 97. What is Response Object (res) in Express?

## Definition

The response object (res) in Express.js is an enhanced version of Node.js's http.ServerResponse that provides convenient methods to send responses back to the client.

---

## Common Methods

### res.send()

Sends a response (string, buffer, object).

```js
res.send("Hello");
```

---

### res.json()

Sends a JSON response.

```js
res.json({ message: "Success" });
```

---

### res.status()

Sets the HTTP status code.

```js
res.status(201).json({ id: 1 });
```

---

### res.redirect()

Redirects to another URL.

```js
res.redirect("/dashboard");
```

---

### res.sendFile()

Sends a file as a response.

```js
res.sendFile("/path/to/file.html");
```

---

### res.set()

Sets response headers.

```js
res.set("Content-Type", "application/json");
```

---

## Example

```js
app.post("/users", (req, res) => {
  const user = req.body;

  // Create user...

  res.status(201).json({
    message: "User Created",
    data: user
  });
});
```

---

## Chaining

```js
res
  .status(200)
  .set("X-Custom-Header", "value")
  .json({ success: true });
```

---

## Interview Answer (Ready to Speak)

The response object in Express.js provides methods for sending data back to clients. The most commonly used methods are res.json() for sending JSON, res.send() for general responses, res.status() for setting HTTP status codes, and res.redirect() for URL redirections.

---

## Common Mistakes

❌ Sending multiple responses.

❌ Forgetting to set the correct status code.

---

## Quick Revision

```text
res.json() = JSON Response
res.send() = General Response
res.status() = HTTP Status Code
res.redirect() = Redirect
res.sendFile() = File Response
```

---

### 98. What is Error Handling Middleware in Express?

## Definition

Error-handling middleware in Express.js is a special middleware function with four parameters (err, req, res, next) that catches and handles errors passed via next(err).

---

## Why Do We Need It?

In production applications, errors are inevitable:

- Database failures
- Validation errors
- Authentication failures
- Server crashes

A centralized error handler sends a clean response instead of crashing the server.

---

## Standard Error Handler

```js
app.use((err, req, res, next) => {
  console.error(err.message);

  res.status(err.status || 500).json({
    success: false,
    message: err.message || "Server Error"
  });
});
```

---

## Triggering Error Handler

```js
app.get("/data", (req, res, next) => {
  try {

    throw new Error("Database Failed");

  } catch (err) {

    next(err);

  }
});
```

---

## Internal Flow

```text
Route Handler Throws Error
         ↓

next(err) Called
         ↓

Error-Handling Middleware
         ↓

Error Response Sent
```

---

## Important Rule

Error-handling middleware must be defined AFTER all routes.

```js
app.get("/path", handler);

// Error handler last
app.use((err, req, res, next) => {});
```

---

## Custom Error Class

```js
class AppError extends Error {
  constructor(message, status) {
    super(message);
    this.status = status;
  }
}

throw new AppError("Not Found", 404);
```

---

## Benefits

### Centralized Error Handling

### Clean Responses

### No Crashes

### Easy Debugging

---

## Interview Answer (Ready to Speak)

Error-handling middleware in Express.js is a special function with four parameters: err, req, res, and next. It is placed after all other routes and handles errors passed via next(err). It centralizes error management, ensuring consistent error responses and preventing uncaught exceptions from crashing the server.

---

## Follow-Up Questions

### How many parameters does error middleware have?

Four: err, req, res, next.

### Where should it be placed?

After all routes.

### How to trigger it?

```js
next(err)
```

---

## Common Mistakes

❌ Placing error middleware before routes.

❌ Forgetting the four-parameter signature.

---

## Quick Revision

```text
Four Parameters
err, req, res, next
After All Routes
next(err)
Centralized Handling
```

---

### 99. What is CORS?

## Definition

CORS (Cross-Origin Resource Sharing) is an HTTP mechanism that allows or restricts web applications running at one domain to make requests to a different domain.

---

## Why Is It Needed?

Browsers enforce the Same-Origin Policy.

This blocks frontend at:

```text
http://localhost:3000
```

from accessing backend at:

```text
http://localhost:5000
```

CORS allows the server to grant permission to specific origins.

---

## What Is an Origin?

```text
Protocol + Domain + Port

https://example.com:443
```

Two URLs with different protocol, domain, OR port are considered different origins.

---

## Without CORS

Browser blocks the request:

```text
Access to fetch at 'http://localhost:5000'
from origin 'http://localhost:3000'
has been blocked by CORS policy.
```

---

## Enabling CORS in Express

### Install cors package

```bash
npm install cors
```

---

### Enable For All Origins

```js
const cors = require("cors");
app.use(cors());
```

---

### Enable For Specific Origin

```js
app.use(cors({
  origin: "http://localhost:3000"
}));
```

---

## How It Works Internally

```text
Browser Sends Preflight Request (OPTIONS)
         ↓

Server Responds With CORS Headers
         ↓

Browser Sends Actual Request
```

---

## CORS Headers

```http
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, POST
```

---

## Benefits

### Secure Cross-Origin Communication

### Controlled Access

### Industry Standard

---

## Interview Answer (Ready to Speak)

CORS is a browser security feature that restricts cross-origin HTTP requests. When a frontend application on one origin tries to access a backend on a different origin, the browser blocks it unless the server explicitly allows it through CORS headers. In Express, the cors npm package provides easy configuration.

---

## Follow-Up Questions

### What does CORS stand for?

Cross-Origin Resource Sharing.

### Which package handles CORS in Express?

cors.

### What header allows all origins?

```http
Access-Control-Allow-Origin: *
```

---

## Common Mistakes

❌ Thinking CORS is a server security feature.

❌ Allowing all origins (*) in production.

---

## Quick Revision

```text
Cross-Origin Requests
Browser Security
cors Package
Access-Control Headers
Configure Specific Origins
```

---

### 100. What is express.static() Middleware?

## Definition

express.static() is a built-in Express middleware used to serve static files such as HTML, CSS, JavaScript, images, and fonts from a specified folder.

---

## Why Do We Need It?

Web applications often need to serve:

- HTML Files
- CSS Files
- JavaScript Files
- Images
- Fonts

Express provides express.static() for this purpose.

---

## Example

```js
const express = require("express");
const app = express();

app.use(express.static("public"));

app.listen(3000);
```

If public/index.html exists:

```text
http://localhost:3000/index.html
```

Serves the file automatically.

---

## Folder Structure

```text
project/
   public/
      index.html
      style.css
      script.js
      images/
         logo.png
```

---

## Accessing Files

```text
http://localhost:3000/index.html
http://localhost:3000/style.css
http://localhost:3000/images/logo.png
```

---

## Virtual Path Prefix

```js
app.use(
  "/static",
  express.static("public")
);
```

Now files are accessed at:

```text
http://localhost:3000/static/index.html
```

---

## Internal Flow

```text
GET /index.html
      ↓

express.static() Middleware
      ↓

Look In public/ Folder
      ↓

Serve File If Found
```

---

## Benefits

### Simple Static File Serving

### No Separate Server Needed

### Built-In — No Extra Package

---

## Real-World Use Cases

- Serving Frontend Files
- Image Delivery
- Document Downloads
- SPA (Single Page Application) Serving

---

## Interview Answer (Ready to Speak)

express.static() is a built-in middleware that serves static files such as HTML, CSS, JavaScript, and images from a folder. It eliminates the need for a separate static file server and is commonly used to serve frontend files alongside a backend API.

---

## Follow-Up Questions

### Which folder is commonly used?

public/.

### Can we use a virtual path?

Yes.

### Is it a third-party package?

No. It is built into Express.

---

## Common Mistakes

❌ Placing static middleware after routes.

❌ Not creating the public folder.

---

## Quick Revision

```text
Serve Static Files
HTML, CSS, JS, Images
express.static("folder")
Built-In Middleware
Virtual Path Optional
```

---

### 101. What is Authentication?

## Definition

Authentication is the process of verifying the identity of a user, device, or system attempting to access a resource.

---

## Simple Explanation

Authentication answers the question:

```text
Who Are You?
```

---

## Real-World Analogy

Imagine entering a building.

```text
Security Guard
     ↓
Shows ID Card
     ↓
Identity Verified
     ↓
Entry Granted
```

The ID check is authentication.

---

## Authentication vs Authorization

| Feature | Authentication | Authorization |
|---------|---------------|---------------|
| Question | Who are you? | What can you do? |
| Happens | Before Authorization | After Authentication |
| Example | Login | Admin Panel Access |

---

## Common Authentication Methods

### Username & Password

Most common approach.

---

### JWT (JSON Web Token)

Token-based authentication.

---

### OAuth

Third-party login (Google, GitHub).

---

### OTP

One-time password via email or SMS.

---

### Biometric

Fingerprint or face recognition.

---

## Authentication Flow

```text
User Sends Credentials
         ↓

Server Verifies
         ↓

Success: Issue Token or Session
         ↓

Client Uses Token For Future Requests
```

---

## Example in Express

```js
app.post("/login", async (req, res) => {
  const { email, password } = req.body;

  const user = await User.findOne({ email });

  if (!user) {
    return res.status(401).json({
      message: "Invalid Credentials"
    });
  }

  const isMatch = await bcrypt.compare(
    password,
    user.password
  );

  if (!isMatch) {
    return res.status(401).json({
      message: "Invalid Credentials"
    });
  }

  const token = jwt.sign(
    { id: user._id },
    process.env.JWT_SECRET,
    { expiresIn: "1h" }
  );

  res.json({ token });
});
```

---

## Benefits

### Security

### Identity Verification

### Access Control Foundation

---

## Interview Answer (Ready to Speak)

Authentication is the process of verifying who a user is. In web applications, it typically involves accepting credentials such as email and password, verifying them against stored records, and issuing a token or session upon success. Authentication is the first step before authorization can occur.

---

## Follow-Up Questions

### What is the difference between authentication and authorization?

Authentication verifies identity; authorization grants permissions.

### What is JWT used for?

Token-based authentication.

### Which npm package is used for password verification?

bcrypt.

---

## Common Mistakes

❌ Storing plain-text passwords.

❌ Confusing authentication with authorization.

---

## Quick Revision

```text
Verify Identity
Who Are You?
Before Authorization
JWT, bcrypt
Login Flow
```

---

### 102. What is Authorization?

## Definition

Authorization is the process of determining what permissions and access rights an authenticated user has within a system.

---

## Simple Explanation

Authorization answers the question:

```text
What Are You Allowed To Do?
```

---

## Real-World Analogy

After entering a building (authentication):

```text
Employee Badge
     ↓

Floor 1: Everyone Can Enter
Floor 5: Only Managers
Floor 10: Only Executives
```

The badge level determines access. That is authorization.

---

## Authorization Types

### Role-Based Access Control (RBAC)

Access based on roles.

```text
Admin → Full Access
User → Limited Access
Guest → Read Only
```

---

### Permission-Based

Specific permissions per action.

```text
can_create_post: true
can_delete_user: false
```

---

## Example in Express

```js
const checkRole = (role) => {
  return (req, res, next) => {
    if (req.user.role !== role) {
      return res.status(403).json({
        message: "Access Denied"
      });
    }
    next();
  };
};

app.delete(
  "/users/:id",
  authenticate,
  checkRole("admin"),
  deleteUser
);
```

---

## Authorization Flow

```text
User Authenticated
       ↓

Check Role/Permissions
       ↓

Allowed: Proceed
       ↓

Denied: 403 Forbidden
```

---

## HTTP Status Codes

### 401 Unauthorized

Not authenticated.

---

### 403 Forbidden

Authenticated but not authorized.

---

## Benefits

### Security

### Role Management

### Data Protection

---

## Interview Answer (Ready to Speak)

Authorization determines what actions an authenticated user is allowed to perform. It comes after authentication and uses roles or permissions to grant or deny access to resources. Common HTTP responses for authorization failures are 401 for unauthenticated requests and 403 for unauthorized actions.

---

## Follow-Up Questions

### What status code means access denied?

403 Forbidden.

### What does RBAC stand for?

Role-Based Access Control.

### Does authorization come before authentication?

No. Authentication comes first.

---

## Common Mistakes

❌ Returning 401 for authorization failures (should be 403).

❌ Skipping authorization checks on sensitive routes.

---

## Quick Revision

```text
What Can You Do?
After Authentication
RBAC
403 Forbidden
Permissions
```

---

### 103. What is JWT (JSON Web Token)?

## Definition

JWT (JSON Web Token) is an open standard for securely transmitting information as a JSON object between parties, typically used for authentication and information exchange.

---

## Why Was JWT Created?

Traditional session-based authentication stores sessions on the server.

With JWT:

```text
No Session Storage On Server
Token Contains All Information
Stateless Authentication
```

---

## JWT Structure

A JWT has three parts separated by dots:

```text
Header.Payload.Signature

eyJhbG.eyJzdW.SflKxw
```

---

### Part 1: Header

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

---

### Part 2: Payload

```json
{
  "id": "123",
  "role": "admin",
  "exp": 1714000000
}
```

---

### Part 3: Signature

```text
HMACSHA256(
  base64(header) + "." + base64(payload),
  secretKey
)
```

The signature verifies that the token was not tampered with.

---

## How JWT Works

```text
User Logs In
     ↓

Server Verifies Credentials
     ↓

Server Signs JWT With Secret
     ↓

Token Sent To Client
     ↓

Client Sends Token In Header
     ↓

Server Verifies Signature
     ↓

Access Granted
```

---

## Example in Node.js

### Generating JWT

```js
const jwt = require("jsonwebtoken");

const token = jwt.sign(
  { id: user._id, role: user.role },
  process.env.JWT_SECRET,
  { expiresIn: "1h" }
);
```

---

### Verifying JWT

```js
const decoded = jwt.verify(
  token,
  process.env.JWT_SECRET
);

console.log(decoded.id);
```

---

## Sending Token From Client

```http
Authorization: Bearer eyJhbG...
```

---

## Benefits

### Stateless

No server-side session storage needed.

### Scalable

Works across multiple servers.

### Self-Contained

All user data in the token.

---

## Limitations

### Cannot Be Invalidated Before Expiry

Unless using a blacklist.

### Token Theft

If stolen, attacker can use it.

---

## Interview Answer (Ready to Speak)

JWT is an open standard for creating signed tokens that carry user information. It consists of three Base64-encoded parts: header, payload, and signature. After login, the server signs a JWT with a secret key and sends it to the client. On subsequent requests, the client sends the token in the Authorization header and the server verifies it without needing to query a database.

---

## Follow-Up Questions

### How many parts does a JWT have?

Three.

### What is in the payload?

User data and expiry time.

### Which header carries JWT?

Authorization: Bearer.

---

## Common Mistakes

❌ Storing JWT in localStorage (vulnerable to XSS).

❌ Putting sensitive data in the payload (it is Base64, not encrypted).

---

## Quick Revision

```text
Header.Payload.Signature
Stateless
Authorization: Bearer
jsonwebtoken Package
jwt.sign() and jwt.verify()
```

---

### 104. What is bcrypt?

## Definition

bcrypt is a password hashing function designed specifically for securely storing passwords. It includes a built-in salt and is intentionally slow to resist brute-force attacks.

---

## Why Use bcrypt Over SHA-256?

SHA-256 is:

```text
Very Fast
```

This is a security risk for passwords because attackers can test millions of passwords per second.

bcrypt is:

```text
Intentionally Slow
Configurable Cost Factor
```

---

## How bcrypt Works

```text
Password
   ↓

Generate Salt
   ↓

Hash(Password + Salt)
   ↓

Hashed Password Stored
```

---

## Salt

A salt is a random value added to the password before hashing.

```text
Password: admin123
Salt: $2b$10$abcxyz
Hash: $2b$10$abcxyz...
```

Every hash is unique even for the same password.

---

## Cost Factor (Salt Rounds)

```text
10 rounds = 2^10 iterations = 1024
12 rounds = 2^12 iterations = 4096
```

Higher rounds = slower = more secure = higher CPU usage.

10 is the default and recommended starting point.

---

## Installing bcrypt

```bash
npm install bcrypt
```

---

## Hashing a Password

```js
const bcrypt = require("bcrypt");

const saltRounds = 10;
const hashed = await bcrypt.hash(
  "password123",
  saltRounds
);

console.log(hashed);
// $2b$10$xyz...
```

---

## Comparing a Password

```js
const isMatch = await bcrypt.compare(
  "password123",
  hashed
);

console.log(isMatch); // true
```

---

## Interview Answer (Ready to Speak)

bcrypt is a password hashing library specifically designed for security. It automatically handles salting and uses an adjustable cost factor to make hashing computationally expensive. This slows down brute-force attacks significantly. In Node.js, the bcrypt package provides hash() and compare() methods for password management.

---

## Follow-Up Questions

### Why is bcrypt slow?

By design, to prevent brute-force attacks.

### What is a salt in bcrypt?

A random value added to the password before hashing.

### Which method verifies a password?

```js
bcrypt.compare()
```

---

## Common Mistakes

❌ Using MD5 or SHA-256 for passwords.

❌ Storing plain-text passwords.

---

## Quick Revision

```text
Password Hashing
Built-In Salt
Cost Factor
bcrypt.hash()
bcrypt.compare()
Slow By Design
```

---

### 105. What is Session-Based Authentication?

## Definition

Session-based authentication is an approach where the server creates and stores a session after a user logs in, and the client receives a session ID stored in a cookie.

---

## How It Works

```text
User Logs In
     ↓

Server Verifies Credentials
     ↓

Server Creates Session
Stores In Memory or Database
     ↓

Session ID Sent To Client As Cookie
     ↓

Client Sends Cookie With Each Request
     ↓

Server Looks Up Session
     ↓

Access Granted
```

---

## Session vs JWT

| Feature | Session | JWT |
|---------|---------|-----|
| Storage | Server-Side | Client-Side |
| Scalability | Harder | Easier |
| Stateless | No | Yes |
| Invalidation | Easy | Harder |
| Common Use | Traditional Web Apps | REST APIs |

---

## Example with express-session

```bash
npm install express-session
```

```js
const session = require("express-session");

app.use(session({
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: { maxAge: 3600000 }
}));

app.post("/login", (req, res) => {
  req.session.userId = user._id;
  res.json({ message: "Logged In" });
});
```

---

## Benefits

### Easy Invalidation

Delete session to log out user immediately.

### Secure

Session ID is opaque, not the actual user data.

---

## Limitations

### Server Memory Usage

All sessions stored on server.

### Scalability Issues

Multiple servers need shared session storage (Redis).

---

## Interview Answer (Ready to Speak)

Session-based authentication stores a session on the server after a successful login and sends the session ID to the client as a cookie. On each subsequent request, the server retrieves the session using the ID. Unlike JWT, it is stateful and requires server-side storage, making invalidation easier but scaling more challenging.

---

## Common Mistakes

❌ Not securing session cookies (httpOnly, secure flags).

❌ Using in-memory sessions in production with multiple servers.

---

## Quick Revision

```text
Server-Side Storage
Session ID In Cookie
Stateful
Easy Invalidation
express-session Package
```

---

### 106. What is HTTPS?

## Definition

HTTPS (HyperText Transfer Protocol Secure) is the secure version of HTTP that uses TLS/SSL encryption to protect data transmitted between clients and servers.

---

## Why Is HTTPS Important?

HTTP sends data in plain text:

```text
Username: admin
Password: password123
```

Anyone intercepting the network can read it.

HTTPS encrypts this:

```text
xK9mN2pQ7vR3...
```

The data is unreadable without the decryption key.

---

## How HTTPS Works

```text
Client Hello
     ↓

Server Sends Certificate
     ↓

Client Verifies Certificate
     ↓

Key Exchange
     ↓

Encrypted Communication Begins
```

---

## TLS Certificate

Issued by a Certificate Authority (CA).

Proves the server's identity.

---

## HTTP vs HTTPS

| Feature | HTTP | HTTPS |
|---------|------|-------|
| Encrypted | No | Yes |
| Port | 80 | 443 |
| Secure | No | Yes |
| Recommended | No | Yes |

---

## In Node.js

```js
const https = require("https");
const fs = require("fs");

const options = {
  key: fs.readFileSync("key.pem"),
  cert: fs.readFileSync("cert.pem")
};

https.createServer(options, app)
  .listen(443);
```

---

## Real-World Use Cases

- All Production APIs
- Banking Applications
- E-Commerce Platforms
- Any Application Handling User Data

---

## Interview Answer (Ready to Speak)

HTTPS is the secure version of HTTP that encrypts all data transmitted between the client and server using TLS. It ensures confidentiality, data integrity, and server authentication. All production applications should use HTTPS to protect user data.

---

## Common Mistakes

❌ Using HTTP in production.

❌ Thinking HTTPS encrypts stored data.

---

## Quick Revision

```text
Encrypted HTTP
TLS/SSL
Port 443
Certificate Authority
Production Requirement
```

---

### 107. What is Rate Limiting?

## Definition

Rate limiting is a technique used to control how many requests a client can make to an API within a given time period.

---

## Why Is It Needed?

Without rate limiting:

```text
Attacker Sends
10,000 Requests Per Second
```

This causes:

- Server overload
- DoS/DDoS attacks
- Brute-force login attempts
- API abuse

---

## How Rate Limiting Works

```text
Request Counter Per Client IP
           ↓

If Under Limit: Allow Request
           ↓

If Over Limit: Return 429 Too Many Requests
```

---

## Example with express-rate-limit

```bash
npm install express-rate-limit
```

```js
const rateLimit = require("express-rate-limit");

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // max 100 requests per window
  message: "Too many requests"
});

app.use(limiter);
```

---

## Route-Specific Rate Limiting

```js
const loginLimiter = rateLimit({
  windowMs: 10 * 60 * 1000,
  max: 5,
  message: "Too many login attempts"
});

app.post("/login", loginLimiter, loginHandler);
```

---

## Benefits

### Prevents Brute Force Attacks

### Protects Server Resources

### Prevents API Abuse

### Improves Reliability

---

## HTTP Status Code

When rate limit exceeded:

```text
429 Too Many Requests
```

---

## Interview Answer (Ready to Speak)

Rate limiting restricts the number of requests a client can make to an API in a time window. It protects against brute-force attacks, DoS attacks, and API abuse. In Express, the express-rate-limit package provides easy configuration and returns a 429 status code when the limit is exceeded.

---

## Follow-Up Questions

### Which status code is returned when rate limit is exceeded?

429.

### Which package provides rate limiting in Express?

express-rate-limit.

### Why is rate limiting important on login endpoints?

To prevent brute-force password attacks.

---

## Quick Revision

```text
Request Limit Per Time Window
429 Too Many Requests
express-rate-limit
Brute Force Protection
DoS Prevention
```

---

### 108. What is Helmet.js?

## Definition

Helmet.js is a collection of middleware functions for Express.js that set various HTTP response headers to improve application security.

---

## Why Do We Need Helmet?

By default, Express does not set security headers.

Attackers can exploit missing headers for attacks like:

- XSS (Cross-Site Scripting)
- Clickjacking
- MIME type sniffing
- Protocol downgrade attacks

---

## Installing Helmet

```bash
npm install helmet
```

---

## Using Helmet

```js
const helmet = require("helmet");
app.use(helmet());
```

One line of code enables multiple security headers.

---

## Headers Set by Helmet

### X-Content-Type-Options

Prevents MIME type sniffing.

---

### X-Frame-Options

Prevents clickjacking.

---

### Strict-Transport-Security (HSTS)

Forces HTTPS.

---

### Content-Security-Policy

Prevents XSS attacks.

---

### X-XSS-Protection

Enables browser XSS filter.

---

## Benefits

### Multiple Security Headers At Once

### Production-Ready Security

### Easy Integration

### Industry Standard

---

## Interview Answer (Ready to Speak)

Helmet.js is an Express middleware package that improves security by automatically setting HTTP response headers. It protects against common vulnerabilities like XSS, clickjacking, and protocol downgrade attacks. It is a standard recommendation for any production Express application.

---

## Follow-Up Questions

### What does Helmet do?

Sets security-related HTTP headers.

### Is Helmet a single middleware?

No. It is a collection of middleware.

### Is it needed in production?

Yes, strongly recommended.

---

## Quick Revision

```text
Security Headers
npm install helmet
One Line Setup
XSS Protection
Clickjacking Prevention
```

---

### 109. What is XSS (Cross-Site Scripting)?

## Definition

XSS is a security vulnerability where attackers inject malicious scripts into web pages viewed by other users.

---

## How XSS Works

```text
Attacker Submits
<script>stealCookies()</script>

Server Stores It Without Sanitization

Other User Views The Page

Browser Executes The Script
```

---

## Types of XSS

### Stored XSS

Malicious script is stored in the database.

---

### Reflected XSS

Script is reflected in the URL or response.

---

## Prevention

### Sanitize Input

Remove or escape HTML tags from user input.

---

### Use Content Security Policy

```http
Content-Security-Policy: default-src 'self'
```

---

### Use Helmet.js

Automatically sets CSP headers.

---

### Avoid innerHTML

Use textContent instead.

---

## Interview Answer (Ready to Speak)

XSS is a web security vulnerability where attackers inject malicious scripts into a website that are then executed in the browsers of other users. Prevention involves sanitizing all user inputs, using Content Security Policy headers, and avoiding unsafe DOM manipulation methods.

---

## Quick Revision

```text
Malicious Scripts
Input Sanitization
CSP Headers
Helmet.js
Avoid innerHTML
```

---

### 110. What is CSRF (Cross-Site Request Forgery)?

## Definition

CSRF is an attack where a malicious website tricks an authenticated user's browser into making unwanted requests to another website where the user is logged in.

---

## How CSRF Works

```text
User Logged Into bank.com
       ↓

User Visits evil.com
       ↓

evil.com Sends Hidden Request To bank.com
       ↓

bank.com Thinks User Made The Request
       ↓

Money Transferred
```

---

## Prevention

### CSRF Tokens

A unique token is included in forms and verified on the server.

---

### SameSite Cookie Attribute

```http
Set-Cookie: session=xyz; SameSite=Strict
```

---

### Custom Headers

APIs should verify the origin header.

---

## CSRF vs XSS

| Feature | CSRF | XSS |
|---------|------|-----|
| Victim | Trusted User | Any User |
| Attack Vector | User's Browser | Malicious Script |
| Defense | CSRF Token | Input Sanitization |

---

## Interview Answer (Ready to Speak)

CSRF is an attack where an attacker tricks an authenticated user's browser into sending unauthorized requests to a server. Prevention includes using CSRF tokens, the SameSite cookie attribute, and custom header validation. REST APIs with JWT are generally less vulnerable to CSRF because they don't rely on cookies.

---

## Quick Revision

```text
Tricks Authenticated User
CSRF Token
SameSite Cookie
Origin Validation
REST APIs Less Vulnerable
```

---

### 111. What is MongoDB?

## Definition

MongoDB is a NoSQL, document-oriented database that stores data in flexible JSON-like documents called BSON (Binary JSON) instead of rows and columns.

---

## Why Was MongoDB Created?

Relational databases require a rigid schema:

```text
Table: users
id | name | email | phone
```

If requirements change:

```text
ALTER TABLE users ADD COLUMN age INT
```

MongoDB uses flexible documents:

```json
{
  "name": "Yogesh",
  "email": "yogesh@example.com",
  "phone": "9898989898",
  "age": 25
}
```

Fields can be added without schema changes.

---

## Key Concepts

### Database

Container for collections.

---

### Collection

Similar to a table. Groups related documents.

---

### Document

A single record stored as BSON.

---

## Example Document

```json
{
  "_id": "65f1a2b3c4d5e6f7a8b9c0d1",
  "name": "Yogesh Patel",
  "email": "yogesh@example.com",
  "role": "admin",
  "createdAt": "2024-01-01"
}
```

---

## MongoDB vs SQL

| Feature | MongoDB | SQL |
|---------|---------|-----|
| Schema | Flexible | Fixed |
| Data Format | BSON Documents | Rows & Columns |
| Relations | Embedded/Referenced | Joins |
| Scalability | Horizontal | Vertical |

---

## Benefits

### Flexible Schema

### High Scalability

### Fast Reads

### JSON-Like Documents

### Easy With JavaScript

---

## Interview Answer (Ready to Speak)

MongoDB is a NoSQL database that stores data in flexible BSON documents instead of fixed rows and columns. It is schema-free, allowing documents to vary in structure. MongoDB is especially well-suited for applications that handle large volumes of unstructured or semi-structured data and is commonly used with Node.js due to its JSON-like document format.

---

## Follow-Up Questions

### What format does MongoDB use?

BSON (Binary JSON).

### What is a collection?

A group of related documents, similar to a table.

### Is MongoDB relational?

No. It is a NoSQL database.

---

## Quick Revision

```text
NoSQL Database
BSON Documents
Collections
Flexible Schema
Horizontal Scalability
```

---

### 112. What is Mongoose?

## Definition

Mongoose is an Object Data Modeling (ODM) library for MongoDB and Node.js that provides a schema-based solution to model application data.

---

## Why Do We Need Mongoose?

MongoDB is schema-less.

Without Mongoose:

```js
db.collection("users").insertOne({
  name: "Yogesh",
  email: "yogesh@example.com"
});
```

No validation. Any data can be inserted.

With Mongoose:

```js
const userSchema = new Schema({
  name: { type: String, required: true },
  email: { type: String, required: true, unique: true }
});
```

Data is validated before insertion.

---

## Installing Mongoose

```bash
npm install mongoose
```

---

## Connecting to MongoDB

```js
const mongoose = require("mongoose");

mongoose.connect(process.env.MONGO_URI);
```

---

## Creating a Schema

```js
const { Schema, model } = require("mongoose");

const userSchema = new Schema({
  name: { type: String, required: true },
  email: { type: String, required: true },
  role: { type: String, default: "user" },
  createdAt: { type: Date, default: Date.now }
});
```

---

## Creating a Model

```js
const User = model("User", userSchema);
```

---

## CRUD With Mongoose

### Create

```js
const user = await User.create({
  name: "Yogesh",
  email: "yogesh@example.com"
});
```

---

### Read

```js
const users = await User.find();
const user = await User.findById(id);
```

---

### Update

```js
await User.findByIdAndUpdate(
  id,
  { name: "Updated Name" },
  { new: true }
);
```

---

### Delete

```js
await User.findByIdAndDelete(id);
```

---

## Benefits

### Schema Validation

### Type Casting

### Query Helpers

### Middleware Hooks

### Relationship Support

---

## Interview Answer (Ready to Speak)

Mongoose is an ODM library for MongoDB that provides schema-based data modeling. It validates data before saving, provides query helpers, supports middleware hooks, and makes working with MongoDB in Node.js significantly more structured and maintainable.

---

## Follow-Up Questions

### What does ODM stand for?

Object Data Modeling.

### What does a Mongoose Schema do?

Defines the structure and validation rules for documents.

### What does a Model do?

Provides an interface to interact with a MongoDB collection.

---

## Quick Revision

```text
ODM Library
Schema Validation
model()
CRUD Methods
Middleware Hooks
```

---

### 113. What is CRUD in MongoDB?

## Definition

CRUD stands for Create, Read, Update, and Delete — the four fundamental operations performed on data in any database.

---

## CRUD Mapping in MongoDB

| Operation | Mongoose Method |
|-----------|----------------|
| Create | Model.create() |
| Read | Model.find() |
| Update | Model.findByIdAndUpdate() |
| Delete | Model.findByIdAndDelete() |

---

## Create

```js
const product = await Product.create({
  name: "Laptop",
  price: 50000,
  category: "Electronics"
});
```

---

## Read — All Documents

```js
const products = await Product.find();
```

---

## Read — One Document

```js
const product = await Product.findById(id);
```

---

## Read — With Filter

```js
const laptops = await Product.find({
  category: "Electronics"
});
```

---

## Update

```js
const updated = await Product.findByIdAndUpdate(
  id,
  { price: 45000 },
  { new: true }
);
```

The { new: true } option returns the updated document.

---

## Delete

```js
await Product.findByIdAndDelete(id);
```

---

## Full CRUD in Express

```js
// Create
app.post("/products", async (req, res) => {
  const product = await Product.create(req.body);
  res.status(201).json(product);
});

// Read All
app.get("/products", async (req, res) => {
  const products = await Product.find();
  res.json(products);
});

// Read One
app.get("/products/:id", async (req, res) => {
  const product = await Product.findById(req.params.id);
  res.json(product);
});

// Update
app.put("/products/:id", async (req, res) => {
  const product = await Product.findByIdAndUpdate(
    req.params.id,
    req.body,
    { new: true }
  );
  res.json(product);
});

// Delete
app.delete("/products/:id", async (req, res) => {
  await Product.findByIdAndDelete(req.params.id);
  res.json({ message: "Deleted" });
});
```

---

## Interview Answer (Ready to Speak)

CRUD represents the four essential database operations: Create, Read, Update, and Delete. In Mongoose, these map to methods like Model.create(), Model.find(), Model.findByIdAndUpdate(), and Model.findByIdAndDelete(). Every REST API is built around these operations.

---

## Quick Revision

```text
Create → Model.create()
Read → Model.find()
Update → findByIdAndUpdate()
Delete → findByIdAndDelete()
Foundation of REST APIs
```

---

### 114. What is Mongoose Schema?

## Definition

A Mongoose Schema defines the structure, data types, and validation rules for documents stored in a MongoDB collection.

---

## Why Schemas?

MongoDB is schema-less, but real applications need consistency.

Without schema:

```js
// Any data passes
{ name: "Yogesh" }
{ age: 25 }
{ xyz: true }
```

With schema:

```js
// Only valid data passes
{ name: String, required: true }
```

---

## Creating a Schema

```js
const { Schema } = require("mongoose");

const productSchema = new Schema({
  name: {
    type: String,
    required: true,
    trim: true
  },
  price: {
    type: Number,
    required: true,
    min: 0
  },
  category: {
    type: String,
    enum: ["Electronics", "Fashion", "Food"]
  },
  inStock: {
    type: Boolean,
    default: true
  },
  createdAt: {
    type: Date,
    default: Date.now
  }
});
```

---

## Schema Data Types

```text
String
Number
Boolean
Date
Array
ObjectId
Mixed
Buffer
Map
```

---

## Validation Options

### required

Field must be present.

---

### unique

No duplicates allowed.

---

### min / max

For numbers.

---

### enum

Allowed values only.

---

### default

Default value if not provided.

---

### trim

Removes whitespace from strings.

---

## Interview Answer (Ready to Speak)

A Mongoose Schema defines the structure of documents in a MongoDB collection, including field types, default values, and validation rules. It brings order to MongoDB's schema-less nature, ensuring that only valid and consistent data is stored.

---

## Quick Revision

```text
Defines Document Structure
Data Types
Validation Rules
required, unique, default
Makes MongoDB Structured
```

---

### 115. What is Mongoose Model?

## Definition

A Mongoose Model is a class compiled from a schema that provides the interface to interact with a MongoDB collection.

---

## Creating a Model

```js
const mongoose = require("mongoose");
const { Schema, model } = mongoose;

const userSchema = new Schema({
  name: { type: String, required: true },
  email: { type: String, required: true }
});

const User = model("User", userSchema);
```

---

## What Does the Model Name Do?

```js
model("User", userSchema)
```

MongoDB collection name becomes:

```text
users
```

(Mongoose automatically lowercases and pluralizes.)

---

## Using the Model

```js
// Create
const user = await User.create({ name: "Yogesh" });

// Read
const users = await User.find();

// Update
const updated = await User.findByIdAndUpdate(id, data);

// Delete
await User.findByIdAndDelete(id);
```

---

## Model vs Schema

| Feature | Schema | Model |
|---------|--------|-------|
| Purpose | Define Structure | Interact With DB |
| Creates | Blueprint | Class |
| Used For | Validation Rules | CRUD Operations |

---

## Interview Answer (Ready to Speak)

A Mongoose Model is a compiled class based on a schema that provides methods to interact with a MongoDB collection. It handles CRUD operations, query building, and middleware execution. Each model maps to a specific MongoDB collection.

---

## Quick Revision

```text
Compiled From Schema
CRUD Interface
Collection Interaction
model("Name", schema)
Maps To MongoDB Collection
```

---

### 116. What is Mongoose Population?

## Definition

Population in Mongoose is the process of automatically replacing an ObjectId reference with the actual document it refers to, similar to a JOIN in SQL.

---

## Why Is It Needed?

MongoDB documents can reference other documents:

```js
const orderSchema = new Schema({
  user: { type: ObjectId, ref: "User" },
  product: { type: ObjectId, ref: "Product" }
});
```

Instead of returning:

```json
{ "user": "65f1a2b3c4d5e6f7" }
```

Population returns:

```json
{
  "user": {
    "_id": "65f1a2b3c4d5e6f7",
    "name": "Yogesh",
    "email": "yogesh@example.com"
  }
}
```

---

## Example

### Schema With Reference

```js
const orderSchema = new Schema({
  user: {
    type: Schema.Types.ObjectId,
    ref: "User"
  },
  totalAmount: Number
});
```

---

### Populating During Query

```js
const orders = await Order
  .find()
  .populate("user");
```

---

### Selecting Specific Fields

```js
const orders = await Order
  .find()
  .populate("user", "name email");
```

---

## Internal Flow

```text
Order Document Has user: ObjectId
          ↓

.populate("user")
          ↓

Mongoose Finds User Document By ID
          ↓

Replaces ObjectId With Full Document
```

---

## Benefits

### Avoids Multiple Queries

### Clean Data

### SQL-Like JOIN Behavior

---

## Interview Answer (Ready to Speak)

Mongoose population allows ObjectId references to be replaced with actual document data from related collections. It is used when documents reference other documents, providing a way to retrieve fully populated objects in a single query using the .populate() method.

---

## Quick Revision

```text
Replace ObjectId With Document
.populate("field")
ref In Schema
SQL-Like JOIN
Cross-Collection Data
```

---

### 117. What is MongoDB Indexing?

## Definition

Indexing in MongoDB is the process of creating data structures that allow the database to find documents faster without scanning the entire collection.

---

## Why Is Indexing Important?

Without an index:

```text
Search: Find user with email = "yogesh@example.com"
     ↓

Scan Every Document
     ↓

Slow For Large Collections
```

With an index:

```text
Jump Directly To Matching Document
     ↓

Fast
```

---

## Default Index

MongoDB automatically creates an index on:

```text
_id
```

---

## Creating Indexes in Mongoose

### Unique Index

```js
const userSchema = new Schema({
  email: {
    type: String,
    required: true,
    unique: true  // Creates unique index
  }
});
```

---

### Compound Index

```js
userSchema.index({
  firstName: 1,
  lastName: 1
});
```

---

### Text Index (Search)

```js
productSchema.index({
  name: "text",
  description: "text"
});
```

---

## Types of Indexes

### Single Field

Index on one field.

---

### Compound

Index on multiple fields.

---

### Unique

Prevents duplicate values.

---

### Text

Full-text search support.

---

### TTL (Time-To-Live)

Auto-deletes documents after a time period.

---

## Benefits

### Faster Queries

### Efficient Searches

### Database Performance

---

## Limitations

### Slower Writes

Each write updates indexes.

### Memory Usage

Indexes consume RAM.

---

## Interview Answer (Ready to Speak)

Indexing in MongoDB improves query performance by creating data structures that allow fast lookups without scanning every document. In Mongoose, indexes are created using the unique or index options in schemas. Common types include single-field, compound, text, and TTL indexes.

---

## Quick Revision

```text
Faster Queries
unique: true
.index()
Text Index
TTL Index
Slower Writes
```

---

### 118. What is MongoDB Aggregation?

## Definition

MongoDB Aggregation is a framework for processing and transforming documents in a collection through a pipeline of stages to produce computed results.

---

## Why Use Aggregation?

Simple queries find documents.

Aggregation allows:

- Grouping data
- Calculating totals, averages, counts
- Filtering and transforming
- Joining collections
- Complex reporting

---

## Aggregation Pipeline

Data flows through stages one by one:

```text
Input Documents
      ↓

$match (Filter)
      ↓

$group (Aggregate)
      ↓

$sort (Order)
      ↓

$project (Shape Output)
      ↓

Result
```

---

## Common Stages

### $match

Filter documents.

```js
{ $match: { category: "Electronics" } }
```

---

### $group

Group and aggregate.

```js
{
  $group: {
    _id: "$category",
    total: { $sum: "$price" },
    count: { $sum: 1 }
  }
}
```

---

### $sort

Sort documents.

```js
{ $sort: { total: -1 } }
```

---

### $project

Select and reshape fields.

```js
{ $project: { name: 1, price: 1 } }
```

---

### $limit and $skip

Pagination.

```js
{ $limit: 10 }
{ $skip: 20 }
```

---

### $lookup

Join with another collection.

```js
{
  $lookup: {
    from: "users",
    localField: "userId",
    foreignField: "_id",
    as: "userDetails"
  }
}
```

---

## Full Example

```js
const result = await Order.aggregate([
  { $match: { status: "completed" } },
  {
    $group: {
      _id: "$category",
      totalRevenue: { $sum: "$amount" },
      orderCount: { $sum: 1 }
    }
  },
  { $sort: { totalRevenue: -1 } }
]);
```

---

## Benefits

### Complex Data Analysis

### Efficient — Runs On Server

### Real-Time Reporting

### Replaces Multiple Queries

---

## Interview Answer (Ready to Speak)

MongoDB's Aggregation Framework processes documents through a pipeline of stages to compute analytics, perform transformations, and generate reports. Common stages include $match for filtering, $group for aggregation, $sort for ordering, $project for field selection, and $lookup for joining collections.

---

## Follow-Up Questions

### What is the aggregation pipeline?

A sequence of stages that transform documents.

### Which stage filters documents?

$match.

### Which stage performs joins?

$lookup.

---

## Quick Revision

```text
Pipeline Of Stages
$match
$group
$sort
$lookup
$project
```

---

### 119. What are Mongoose Middleware (Hooks)?

## Definition

Mongoose middleware, also called hooks, are functions that execute before or after certain Mongoose operations such as save, find, update, and delete.

---

## Why Use Middleware?

Common tasks that should run automatically:

- Hash password before saving
- Update timestamps automatically
- Log database operations
- Validate data before saving

---

## Types of Middleware

### Document Middleware

Runs on document operations.

```text
save
validate
remove
updateOne
```

---

### Query Middleware

Runs on query operations.

```text
find
findOne
findOneAndUpdate
deleteOne
```

---

## Pre Middleware (Before)

Runs before the operation.

```js
userSchema.pre("save", async function(next) {
  if (this.isModified("password")) {
    this.password = await bcrypt.hash(
      this.password,
      10
    );
  }
  next();
});
```

---

## Post Middleware (After)

Runs after the operation.

```js
userSchema.post("save", function(doc) {
  console.log(`User saved: ${doc.email}`);
});
```

---

## Real-World Use Cases

### Auto-Hash Password

```js
userSchema.pre("save", async function(next) {
  if (this.isModified("password")) {
    this.password = await bcrypt.hash(this.password, 10);
  }
  next();
});
```

---

### Auto-Update Timestamp

```js
userSchema.pre("findOneAndUpdate", function() {
  this.set({ updatedAt: new Date() });
});
```

---

## Benefits

### Automatic Operations

### Clean Code

### Single Responsibility

### Reusability

---

## Interview Answer (Ready to Speak)

Mongoose middleware are hooks that execute before (pre) or after (post) specific database operations. Pre middleware is commonly used for password hashing before saving, while post middleware handles logging or notifications after operations complete. They keep business logic centralized and schemas clean.

---

## Follow-Up Questions

### What are the two types of hooks?

pre and post.

### Name a common use case for pre("save").

Password hashing.

### Must next() be called in pre middleware?

Yes, in document middleware.

---

## Quick Revision

```text
pre = Before Operation
post = After Operation
save, find, update, delete
Password Hashing
Auto Timestamps
```

---

### 120. What is MVC Architecture?

## Definition

MVC (Model-View-Controller) is an architectural pattern that separates an application into three components: Model, View, and Controller.

---

## Why Use MVC?

Without architecture:

```text
All Code In One File
Routes
Database Queries
Business Logic
HTML Rendering
```

Difficult to maintain, scale, or test.

MVC separates concerns cleanly.

---

## Three Components

### Model

Handles data and database interactions.

```js
// models/userModel.js
const userSchema = new Schema({ ... });
const User = model("User", userSchema);
module.exports = User;
```

---

### View

Handles the presentation layer (HTML, templates, or JSON for APIs).

In REST APIs, the view is typically the JSON response.

---

### Controller

Handles request logic, interacts with Model, and sends response.

```js
// controllers/userController.js
const getUsers = async (req, res) => {
  const users = await User.find();
  res.json(users);
};
```

---

### Router

Routes requests to the correct controller.

```js
// routes/userRoutes.js
const router = express.Router();
router.get("/", getUsers);
```

---

## Folder Structure

```text
project/
  controllers/
    userController.js
    productController.js
  models/
    userModel.js
    productModel.js
  routes/
    userRoutes.js
    productRoutes.js
  app.js
```

---

## Request Flow

```text
HTTP Request
      ↓

Router (routes/)
      ↓

Controller (controllers/)
      ↓

Model (models/)
      ↓

Database
      ↓

Response
```

---

## Benefits

### Separation of Concerns

### Maintainability

### Testability

### Team Collaboration

### Scalability

---

## Real-World Analogy

Imagine a restaurant:

```text
Waiter = Controller
(Takes order, communicates)

Menu/Kitchen = Model
(Data and preparation)

Plate Presented = View
(Final output)
```

---

## Interview Answer (Ready to Speak)

MVC is an architectural pattern that divides an application into three parts: Model for data management, View for presentation, and Controller for business logic. In Node.js REST APIs, the pattern is implemented using separate folders for models, controllers, and routes. MVC improves code organization, maintainability, and scalability.

---

## Follow-Up Questions

### What does MVC stand for?

Model, View, Controller.

### What does the Controller do?

Handles request logic and communicates between Model and View.

### What does the Model do?

Manages data and database interactions.

---

## Quick Revision

```text
Model → Data
View → Presentation
Controller → Logic
Separate Concerns
Scalable Architecture
```

---
