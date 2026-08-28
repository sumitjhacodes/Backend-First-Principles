# Backend First Principles

> Learn backend development from the ground up — understand how things actually work, build real projects, and become confident enough to face backend interviews.

**Backend First Principles** is a beginner-friendly, practical backend engineering learning repository.

I created this because when you're starting backend development, it's very easy to learn *how to use* a framework without really understanding what's happening underneath.

You learn Express.

You learn `app.get()`.

You connect MongoDB.

You create some APIs.

And then an interviewer asks:

> "What actually happens when a request reaches your server?"

And suddenly things get confusing.

This repo is my attempt to build a learning path that starts from the **actual fundamentals** and slowly moves toward the things backend engineers deal with in real applications.

No need to know everything before starting.

No jumping straight into microservices.

No trying to memorize hundreds of definitions.

Just learn one concept, understand why it exists, build something with it, and then move to the next one.

---

## Who Is This For?

This repo is mainly for:

* Someone starting backend development
* Freshers preparing for backend/full-stack interviews
* Junior developers who want stronger fundamentals
* Frontend developers moving toward backend
* Anyone who can write some code but doesn't understand what happens behind an API
* Developers who want to learn backend by building, not just watching tutorials

You don't need to be a senior developer to follow this.

The goal is to **become one step better after every chapter.**

---

# What Will You Learn?

The roadmap is divided into four phases.

```text
Phase 1 → Foundation
Phase 2 → Data & Storage
Phase 3 → Advanced Backend
Phase 4 → Production & Scale
```

The idea is to build the knowledge in layers.

You shouldn't learn database scaling before understanding what a database actually is.

You shouldn't learn microservices before understanding a backend application.

And you shouldn't learn system design by memorizing architecture diagrams.

So we'll build the foundation first.

---

# Phase 1 — Foundation

**Folder:** `phase-01-foundation/`

This phase answers a very basic question:

> **What actually happens when a client talks to a backend?**

### 01 — How the Internet Actually Works

You'll learn:

* DNS
* IP addresses
* TCP / UDP
* Ports
* Packets
* Client and server
* What happens when you type a URL
* How your request travels across the internet

The goal isn't to become a networking engineer.

The goal is to understand enough networking to stop treating the internet like magic.

---

### 02 — HTTP Deep Dive

You'll learn:

* HTTP requests and responses
* HTTP methods
* Status codes
* Headers
* Request body
* Response body
* Statelessness
* Idempotency
* PUT vs PATCH
* HTTP vs HTTPS

You'll start understanding what is actually happening when you call an API.

---

### 03 — What Is a Web Server?

You'll learn:

* What a web server actually does
* Node.js
* Express
* Request/response lifecycle
* Middleware
* Event loop basics
* How a server receives and processes requests
* How to build a simple API server

---

### 04 — Routing & URL Design

You'll learn:

* Routes
* Endpoints
* Path parameters
* Query parameters
* REST conventions
* Resource-oriented URLs
* Nested resources
* How to design cleaner APIs

You'll start thinking about APIs as systems instead of just writing random endpoints.

---

### 05 — JSON & Data Formats

You'll learn:

* JSON
* Objects and arrays
* Serialization
* Parsing
* Content types
* JSON vs XML
* Valid vs invalid JSON
* How APIs exchange data

---

### 06 — Postman, cURL & API Testing

You'll learn:

* How to test APIs without a frontend
* Postman
* cURL
* Headers
* Request bodies
* Query parameters
* Path parameters
* Testing success cases
* Testing failure cases
* API testing mindset
* Basic automated API testing

You'll also learn an important backend habit:

> Don't just test whether your API works. Try to break it.

---

## Phase 1 Project

By the end of Phase 1, you'll build and test a simple:

### Todo API

You'll work with endpoints such as:

```text
GET    /todos
GET    /todos/:id
POST   /todos
PATCH  /todos/:id
DELETE /todos/:id
```

The purpose isn't to build the world's greatest Todo app.

It's to take everything you've learned and connect it together.

```text
Internet
   ↓
HTTP
   ↓
Web Server
   ↓
Routing
   ↓
JSON
   ↓
API
   ↓
Testing
```

---

# Phase 2 — Data & Storage

**Folder:** `phase-02-data-storage/`

Once we know how an API works, we need somewhere to actually store data.

This phase will cover:

### 07 — Databases 101

* What is a database?
* SQL vs NoSQL
* PostgreSQL
* MongoDB
* Redis
* Tables and documents
* Schemas
* ACID
* Consistency

### 08 — SQL Deep Dive

* CRUD
* SELECT
* INSERT
* UPDATE
* DELETE
* JOINs
* Indexes
* Constraints
* Transactions
* Relationships

### 09 — ORMs & Database Tools

* What is an ORM?
* Prisma
* Migrations
* Raw SQL vs ORM
* Database schema management

### 10 — Authentication & Authorization

* Authentication vs authorization
* Password hashing
* Sessions
* JWT
* OAuth
* Protected routes
* Access control

### 11 — Password Security & Best Practices

* Hashing
* Salt
* Password storage
* Rate limiting
* Brute-force protection
* Security mistakes developers commonly make

---

# Phase 3 — Advanced Backend

**Folder:** `phase-03-advanced-backend/`

Now we'll start dealing with problems that appear in real applications.

Topics include:

### 12 — APIs in the Real World

* Pagination
* Filtering
* Sorting
* API versioning
* Search
* API design

### 13 — Caching

* Why caching exists
* Redis
* Cache-aside
* TTL
* Cache invalidation
* Cache hit/miss

### 14 — Background Jobs & Queues

* Queues
* Workers
* Background processing
* Retries
* Failed jobs
* Async workflows

### 15 — WebSockets & Real-Time Systems

* WebSockets
* Socket.io
* Real-time communication
* Rooms
* Notifications
* Scaling real-time systems

### 16 — File Uploads & Storage

* File uploads
* Object storage
* S3
* Presigned URLs
* CDNs
* Large file handling

### 17 — Logging, Monitoring & Error Handling

* Error handling
* Structured logging
* Logs
* Metrics
* Traces
* Monitoring
* Alerting
* Debugging production problems

### 18 — Testing Your Backend

* Unit testing
* Integration testing
* Mocking
* Test isolation
* Test databases
* Coverage
* Testing strategy

---

# Phase 4 — Production & Scale

**Folder:** `phase-04-production-scale/`

This is where we'll take the things we've learned and start thinking about production systems.

### 19 — Environment & Configuration

* Environment variables
* Configuration
* Secrets
* Development vs production
* 12-factor principles

### 20 — Docker & Containerization

* Images
* Containers
* Dockerfiles
* Docker Compose
* Container networking
* Running backend + database + Redis together

### 21 — Databases at Scale

* Connection pooling
* Replication
* Read replicas
* Sharding
* Database bottlenecks
* Scaling databases

### 22 — Monolith vs Microservices

* Monoliths
* Microservices
* Service boundaries
* API gateways
* Inter-service communication
* REST vs gRPC vs messaging
* When *not* to use microservices

### 23 — CI/CD & Deployment

* Continuous Integration
* Continuous Deployment
* GitHub Actions
* Deployment pipelines
* Zero-downtime deployments
* Production environments

### 24 — System Design Basics

* Load balancers
* Reverse proxies
* Horizontal scaling
* Vertical scaling
* Rate limiting
* CAP theorem
* Availability
* Consistency
* Designing systems from requirements

---

# How Each Chapter Works

I'm trying to keep every chapter practical.

Instead of:

> Definition → definition → definition → next topic

the chapters will generally follow this pattern:

### 1. Human Explanation

First understand the concept without drowning in jargon.

### 2. Technical Explanation

Then learn the proper terminology and how it actually works.

### 3. Why Does This Matter?

Where would you encounter this in a real application?

### 4. Let's Build Something

Use the concept instead of just reading about it.

### 5. Common Mistakes

Things beginners commonly misunderstand or implement incorrectly.

### 6. Interview Questions

Questions you might actually encounter in a junior/fresher backend interview.

### 7. Mini Project / Exercise

Something small enough to build but useful enough to remember.

---

# The Learning Philosophy

This repository is not meant to teach you how to memorize backend definitions.

It's meant to help you develop a **backend engineering mindset**.

For every concept, try to answer:

```text
What is it?

Why does it exist?

What problem does it solve?

How does it work?

When would I use it?

When would I NOT use it?

What can go wrong?

How would I debug it?

How would I explain it in an interview?
```

If you can answer those questions, you probably understand the concept.

---

# Projects

The repository will also contain practical projects.

```text
projects/
├── todo-api/
├── blog-platform/
└── chat-app/
```

The projects will grow as the concepts grow.

### Todo API

Foundation:

```text
HTTP
REST
Routing
JSON
API testing
```

### Blog Platform

Data & authentication:

```text
PostgreSQL
SQL
ORM
Authentication
Authorization
Relationships
```

### Chat App

Advanced backend:

```text
WebSockets
Redis
Queues
Background jobs
Real-time notifications
```

The idea is to avoid building completely unrelated projects after every chapter.

Instead, the projects should become more realistic as your knowledge grows.

---

# Tech Stack

The examples and projects will primarily use:

```text
Node.js
TypeScript / JavaScript
Express.js
PostgreSQL
Prisma
Redis
Docker
GitHub Actions
```

The goal isn't to teach one framework.

The goal is to understand the backend concepts that sit underneath the framework.

Once you understand the concepts, switching frameworks becomes much easier.

---

# What You Should Be Able To Do After This

If you complete the whole roadmap properly, you should be comfortable with things like:

```text
Build REST APIs
        ↓
Design routes
        ↓
Work with databases
        ↓
Implement authentication
        ↓
Validate requests
        ↓
Handle errors
        ↓
Cache expensive operations
        ↓
Process background jobs
        ↓
Build real-time features
        ↓
Test your backend
        ↓
Dockerize your application
        ↓
Deploy it
        ↓
Understand basic scaling
        ↓
Discuss backend system design
```

You won't magically become a senior engineer after reading a repository.

That's not how it works.

But you should have a much stronger foundation for **building real backend applications and answering the "why" behind the code.**

---

# For Interview Preparation

This repo is also designed with junior/fresher backend interviews in mind.

You'll encounter questions around:

* HTTP
* REST APIs
* Node.js
* Express
* Databases
* SQL
* PostgreSQL
* Authentication
* JWT
* Redis
* Queues
* WebSockets
* Docker
* Testing
* Deployment
* Basic system design

But the goal isn't to memorize one-line answers.

Instead, you should be able to explain:

> **what it is → why it's used → how it works → where you'd use it → what can go wrong.**

That's a much better way to prepare for an interview.

---

# Repository Structure

```text
Backend-First-Principles/
│
├── README.md
│
├── phase-01-foundation/
│   ├── 01-how-the-internet-works.md
│   ├── 02-http-deep-dive.md
│   ├── 03-what-is-a-web-server.md
│   ├── 04-routing-url-design.md
│   ├── 05-json-data-formats.md
│   └── 06-postman-curl-api-testing.md
│
├── phase-02-data-storage/
│
├── phase-03-advanced-backend/
│
├── phase-04-production-scale/
│
└── projects/
    ├── todo-api/
    ├── blog-platform/
    └── chat-app/
```

The repository is being built progressively, so some sections may be incomplete while others are already available.

---

# How To Use This Repo

Don't try to finish everything in one weekend.

For each chapter:

```text
Read
 ↓
Understand
 ↓
Code
 ↓
Break it
 ↓
Fix it
 ↓
Answer the interview questions
 ↓
Build the exercise
 ↓
Move on
```

If something doesn't make sense, don't just memorize it.

Go back one step.

Backend concepts are connected.

Usually, if something feels confusing, there is a simpler concept underneath that you haven't fully understood yet.

---

# A Note From Me

I'm building this repository because I wanted a place where backend development could be learned from the **first principles**, without assuming that everyone already knows how everything works.

I'm also learning and refining this as I build it.

So this won't be a perfect textbook.

There will probably be mistakes, improvements, and things that change over time.

If you find something incorrect or have a better explanation, feel free to open an issue or pull request.

If this helps you learn something, that's honestly the main reason I'm building it.

---

## Start Here

If you're completely new to backend development, start with:

**[01 — How the Internet Actually Works](./phase-01-foundation/01-how-the-internet-works.md)**

Then go chapter by chapter.

Don't skip the fundamentals.

They'll make everything that comes later much easier.

---

## ⭐ If This Helps You

If you're learning backend development from this repository and find it useful, consider giving it a ⭐ on GitHub.

And if you find something that can be improved, contributions are welcome.

**Learn the fundamentals. Build things. Break things. Fix them. Repeat.**

That's the whole idea behind **Backend First Principles**.