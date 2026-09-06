# Chapter 24 — System Design Basics

If you have followed this series from Chapter 1, you already know quite a lot about backend development.

You learned how HTTP works.

You built APIs.

You worked with databases.

You learned authentication, caching, queues, WebSockets, file uploads, logging, testing, Docker, database scaling, microservices, and CI/CD.

Now comes the question:

> **How do I put all of these pieces together to build a system that can handle real users?**

This is where **System Design** comes in.

And honestly, system design can look scary in the beginning.

You see diagrams with load balancers, Redis, Kafka, replicas, CDN, API gateways, queues, object storage, and suddenly it feels like:

> "I don't know any of this."

But you actually do.

You just need to learn how to connect the pieces.

---

# 1. What Is System Design?

System design is the process of deciding:

* What components does our system need?
* How do those components communicate?
* Where should data live?
* How does the system handle more users?
* What happens when something fails?
* How do we keep the system fast?
* How do we keep it secure?
* How do we avoid a single server becoming a bottleneck?

For example, imagine you build a simple todo application.

At the beginning:

```text
User
  |
  v
Backend Server
  |
  v
PostgreSQL
```

That's completely fine.

You might have:

```text
10 users
100 users
1,000 users
```

and one backend server could probably handle it.

But imagine your application suddenly gets:

```text
1 million users
```

Now you have different problems.

What if your server receives too many requests?

What if your database becomes slow?

What if your server crashes?

What if thousands of users upload files?

What if one API endpoint suddenly receives 100,000 requests per second?

System design is about thinking through these problems **before they become production incidents**.

---

# 2. System Design Is Not Just Drawing Boxes

One common mistake beginners make is thinking system design means drawing something like:

```text
              +-------------+
              | Load Balancer|
              +------+------+
                     |
        +------------+------------+
        |            |            |
        v            v            v
     Server 1     Server 2     Server 3
        |            |            |
        +------------+------------+
                     |
                   Redis
                     |
                  Database
```

That's not system design by itself.

The important question is:

> **Why are these components here?**

For example:

Why do we need a load balancer?

Because we have multiple backend servers and need to distribute traffic between them.

Why multiple servers?

Because one server may not be enough and we don't want one server failure to bring everything down.

Why Redis?

Maybe because some data is requested frequently and we don't want every request hitting the database.

Why replicas?

Maybe because read traffic is much larger than write traffic.

Good system design is not:

> "Add more technologies."

It is:

> **Identify the problem → understand the constraint → choose the simplest solution that solves it.**

---

# 3. Start With Requirements

Before designing anything, understand what you're building.

Suppose an interviewer says:

> "Design a URL shortener."

Don't immediately start drawing Redis and databases.

First ask questions.

For example:

### Functional requirements

What should users be able to do?

```text
Create a short URL
Open a short URL
Redirect to the original URL
```

Maybe users can also:

```text
Create custom aliases
Track clicks
Delete URLs
```

But don't assume everything.

Ask.

---

# 4. Functional vs Non-Functional Requirements

This distinction is extremely important in system design interviews.

## Functional requirements

These describe **what the system does**.

Example:

```text
User creates a short URL.

User opens the short URL.

System redirects them to the original URL.
```

## Non-functional requirements

These describe **how the system should behave**.

For example:

```text
Low latency
High availability
Scalability
Security
Reliability
Consistency
```

You might say:

> "The redirect API should respond quickly and the service should remain available even if one backend server fails."

Now we have actual design constraints.

---

# 5. Questions You Should Ask Before Designing

Before drawing architecture, try to understand:

### Users

```text
How many users?
How many active users?
```

### Traffic

```text
Requests per second?
Peak requests?
Read/write ratio?
```

### Data

```text
How much data?
How quickly does it grow?
How large is each record?
```

### Performance

```text
What latency is acceptable?
```

### Availability

```text
Can the system be temporarily unavailable?
```

### Consistency

```text
Must every user immediately see the latest data?
```

### Budget

```text
Are we optimizing for simplicity or massive scale?
```

You don't need perfect numbers.

You need reasonable assumptions.

---

# 6. Back-of-the-Envelope Estimation

You don't always need exact numbers.

But you should be able to estimate.

Suppose:

```text
10 million users
```

and:

```text
10% are active each day
```

That's:

```text
10,000,000 × 10%
= 1,000,000 daily active users
```

Now suppose each user makes:

```text
10 requests/day
```

Then:

```text
1,000,000 × 10
= 10,000,000 requests/day
```

Average requests per second:

```text
10,000,000 / 86,400
≈ 116 requests/sec
```

But traffic is rarely perfectly distributed.

Suppose peak traffic is 10x average:

```text
116 × 10
≈ 1,160 requests/sec
```

Now your architecture needs to handle roughly:

```text
~1,200 requests/sec at peak
```

This is much more useful than saying:

> "We have lots of users."

---

# 7. Storage Estimation

Suppose every URL record requires around:

```text
500 bytes
```

and we create:

```text
10 million URLs
```

Then:

```text
10,000,000 × 500 bytes
= 5,000,000,000 bytes
```

Approximately:

```text
5 GB
```

This is just an estimate.

Real systems also need:

* indexes
* metadata
* replication
* backups
* overhead

So actual storage will be higher.

The important skill is understanding **the order of magnitude**.

---

# 8. Bandwidth Estimation

You can do the same thing with bandwidth.

Suppose an API response is approximately:

```text
20 KB
```

and you receive:

```text
1,000 requests/sec
```

Then:

```text
20 KB × 1,000
= 20 MB/sec
```

Approximately:

```text
1.2 GB/minute
```

Again, these don't need to be perfect.

They help you understand whether the system is small, medium, or massive.

---

# 9. The Basic Architecture

A typical backend architecture might look like:

```text
                  Users
                    |
                    v
                  DNS
                    |
                    v
                   CDN
                    |
                    v
             Load Balancer
                    |
          +---------+---------+
          |         |         |
          v         v         v
       Server 1  Server 2  Server 3
          |         |         |
          +---------+---------+
                    |
          +---------+---------+
          |                   |
          v                   v
        Redis             PostgreSQL
                              |
                       +------+------+
                       |             |
                       v             v
                    Replica 1     Replica 2
```

And for asynchronous work:

```text
Backend
   |
   v
Queue
   |
   +------> Worker 1
   |
   +------> Worker 2
   |
   +------> Worker 3
```

For files:

```text
User
  |
  v
Backend
  |
  v
Object Storage
  |
  v
CDN
```

You don't always need every component.

That's important.

---

# 10. DNS

You learned about DNS in Chapter 1.

DNS translates a domain name into an IP address.

For example:

```text
api.example.com
       |
       v
    DNS
       |
       v
IP address
```

The user doesn't need to know the IP address of your server.

They use:

```text
api.example.com
```

DNS helps route them toward your infrastructure.

---

# 11. CDN

A CDN, or Content Delivery Network, stores and serves content from locations closer to users.

Imagine you have users in:

```text
India
USA
Europe
Australia
```

If every image has to come from one server in India, users far away may experience higher latency.

A CDN can cache static content closer to users.

For example:

```text
                Origin Server
                     |
              +------+------+
              |             |
              v             v
           CDN Edge       CDN Edge
             India          USA
              |              |
              v              v
           Users           Users
```

CDNs are especially useful for:

* images
* videos
* JavaScript
* CSS
* static files

---

# 12. Load Balancer

Suppose one server receives:

```text
100,000 requests
```

It may become overloaded.

Instead, we run multiple servers:

```text
                 Load Balancer
                 /     |      \
                /      |       \
               v       v        v
           Server 1 Server 2 Server 3
```

The load balancer distributes traffic.

For example:

```text
Request 1 → Server 1
Request 2 → Server 2
Request 3 → Server 3
Request 4 → Server 1
```

This is called **horizontal scaling**.

---

# 13. Vertical vs Horizontal Scaling

There are two basic approaches.

## Vertical scaling

Make one machine more powerful.

```text
8 GB RAM
   ↓
32 GB RAM
   ↓
64 GB RAM
```

and perhaps:

```text
4 CPU
 ↓
16 CPU
```

Simple, but there is a physical/financial limit.

---

## Horizontal scaling

Add more machines.

```text
Server 1
Server 2
Server 3
Server 4
```

This is usually more flexible for large distributed systems.

---

# 14. Stateless Backend Servers

Horizontal scaling becomes much easier when your backend servers are **stateless**.

Imagine:

```text
User → Server 1
```

The server stores important session information only in its own memory.

Later:

```text
User → Server 2
```

Server 2 doesn't know anything.

That's a problem.

Instead, shared state can live in something like:

```text
Redis
Database
Object Storage
```

Then:

```text
Server 1 ──┐
Server 2 ──┼──> Shared Storage
Server 3 ──┘
```

Now requests can go to any server.

This is one reason stateless application servers are useful.

---

# 15. Reverse Proxy

A reverse proxy sits between clients and backend servers.

```text
Client
  |
  v
Reverse Proxy
  |
  +----> Backend
  |
  +----> Backend
```

It can handle things like:

* routing
* TLS termination
* compression
* caching
* request filtering
* load balancing

Examples include Nginx and other proxy/load-balancing systems.

A reverse proxy and load balancer are related concepts, but they are not exactly the same thing.

---

# 16. API Gateway

In larger systems, you may have multiple backend services.

For example:

```text
User Service
Order Service
Payment Service
Notification Service
```

Instead of exposing everything directly:

```text
Client
   |
   +----> User Service
   +----> Order Service
   +----> Payment Service
```

you might have:

```text
Client
   |
   v
API Gateway
   |
   +----> User Service
   +----> Order Service
   +----> Payment Service
```

The gateway can handle common concerns such as:

* authentication
* routing
* rate limiting
* request validation
* logging

But again:

> Don't add an API gateway just because it looks impressive.

If you're building a small application, your normal backend server may be enough.

---

# 17. Caching

Suppose this endpoint receives thousands of requests:

```text
GET /products/popular
```

If every request hits PostgreSQL:

```text
User
 |
 v
API
 |
 v
PostgreSQL
```

the database can become a bottleneck.

Instead:

```text
User
 |
 v
API
 |
 v
Redis
 |
 +-- HIT → return data
 |
 +-- MISS → Database
```

This is the basic idea of caching.

Caching can improve:

* latency
* database load
* throughput

But it introduces another problem:

> **When does cached data become stale?**

You learned about cache invalidation earlier.

Remember:

> Caching is not free complexity.

---

# 18. Database Scaling

A single database may eventually become a bottleneck.

You can use:

```text
Primary
  |
  +----> Replica 1
  |
  +----> Replica 2
```

Writes go to the primary:

```text
POST /users
      |
      v
Primary
```

Reads can potentially go to replicas:

```text
GET /users
      |
      v
Replica
```

This can increase read capacity.

But there is an important problem:

### Replication lag

The replica may not immediately have the latest data.

So:

```text
Write → Primary
Read  → Replica
```

could temporarily return older data.

This is why system design isn't just:

> "Add replicas."

You must understand the tradeoff.

---

# 19. Connection Pooling

Imagine your backend has:

```text
20 application servers
```

and each server creates:

```text
50 database connections
```

Then:

```text
20 × 50
= 1,000 connections
```

Your database may not be able to handle that.

This is why connection pooling matters.

Instead of creating a new database connection for every request, applications reuse connections from a pool.

```text
Application
    |
    v
Connection Pool
    |
    +---- DB connection
    +---- DB connection
    +---- DB connection
```

More servers doesn't automatically mean more database capacity.

Sometimes the database becomes the bottleneck first.

---

# 20. Queues and Background Workers

Not every task needs to happen during the HTTP request.

Imagine a user signs up.

Your API might need to:

```text
Create account
Send welcome email
Generate analytics event
Create notification
Process profile image
```

Doing everything synchronously can make the request slow.

Instead:

```text
User
 |
 v
API
 |
 +----> Database
 |
 +----> Queue
          |
          +----> Email Worker
          |
          +----> Notification Worker
```

The API can respond quickly while workers process background jobs.

This also gives you:

* retries
* delayed processing
* controlled concurrency
* better failure isolation

---

# 21. Object Storage

Don't store large files directly inside your relational database unless you have a specific reason.

For things like:

```text
Images
Videos
PDFs
Documents
Backups
```

object storage is often a better fit.

The architecture might look like:

```text
User
 |
 v
Backend
 |
 v
Object Storage
 |
 v
CDN
 |
 v
User
```

The database stores metadata:

```text
id
user_id
file_name
file_url
file_size
content_type
created_at
```

while the actual file lives in object storage.

---

# 22. Rate Limiting

Imagine your login endpoint receives:

```text
10 requests
```

from a normal user.

Fine.

But someone sends:

```text
1,000,000 requests
```

against the same endpoint.

That's a problem.

Rate limiting controls how many requests a client can make during a period.

For example:

```text
100 requests/minute
```

You might enforce it using:

```text
IP address
User ID
API key
Token
Endpoint
```

A common architecture is:

```text
Client
  |
  v
Rate Limiter
  |
  v
API
```

Redis is often useful for distributed rate limiting.

---

# 23. CAP Theorem

CAP is one of those topics that sounds much harder than it actually is.

CAP stands for:

```text
C = Consistency
A = Availability
P = Partition Tolerance
```

The important idea is that in a distributed system, network partitions can happen.

When communication between parts of the system breaks, you have to make tradeoffs between consistency and availability.

### Consistency

Every successful read sees the appropriate latest value according to the system's consistency model.

### Availability

The system continues responding to requests.

### Partition tolerance

The system continues operating despite communication failures between distributed components.

The practical lesson is not:

> "Pick any two letters."

It's better to think:

> **When a network partition happens, what behavior do we want?**

For some systems:

```text
Consistency > Availability
```

For others:

```text
Availability > Immediate consistency
```

It depends on the product.

---

# 24. Strong vs Eventual Consistency

Suppose a user changes their name.

With strong consistency:

```text
Write
 ↓
Database
 ↓
Immediately read latest value
```

With eventual consistency:

```text
Write
 ↓
System propagates change
 ↓
Some readers may temporarily see old data
 ↓
Eventually everyone sees the new data
```

Eventual consistency can be acceptable for things like:

```text
Like counts
Analytics
Recommendations
Some feeds
Search indexes
```

But you probably don't want eventual consistency everywhere.

For example:

```text
Bank account balance
```

needs much stronger correctness guarantees.

---

# 25. Latency vs Consistency vs Availability

System design is full of tradeoffs.

You might want:

```text
Very low latency
+
Very high consistency
+
Very high availability
+
Very low cost
```

In practice, there are constraints.

Maybe stronger consistency requires more coordination.

Maybe more redundancy increases cost.

Maybe caching improves latency but introduces stale data.

Maybe asynchronous processing improves response time but delays some results.

A good engineer understands these tradeoffs instead of pretending there is one perfect architecture.

---

# 26. Single Point of Failure

A single point of failure is a component where:

> If this component goes down, the whole system goes down.

Imagine:

```text
             Load Balancer
                   |
                   v
               Server 1
                   |
                   v
               Database
```

If Server 1 fails:

```text
Everything is down.
```

Instead:

```text
             Load Balancer
              /          \
             v            v
         Server 1      Server 2
              \          /
               \        /
                v      v
                 Database
```

Now one backend server can fail without necessarily taking down the entire application.

But notice:

```text
              Database
```

could still be a single point of failure.

You need to identify these dependencies.

---

# 27. Redundancy and Failover

Redundancy means having additional capacity/components so the system can continue operating when something fails.

For example:

```text
Primary Database
       |
       v
Replica
```

If the primary fails, a replica may be promoted depending on the architecture.

Similarly:

```text
Server 1
Server 2
Server 3
```

means one server can fail while others continue serving traffic.

This is one of the basic ideas behind highly available systems.

---

# 28. Health Checks

How does a load balancer know whether a server is working?

Health checks.

For example:

```text
GET /health
```

The server might respond:

```json
{
  "status": "ok"
}
```

But in real systems, health checks can be more meaningful.

You might have:

```text
/health/live
/health/ready
```

### Liveness

> Is the application process alive?

### Readiness

> Is the application actually ready to receive traffic?

For example, a server may be alive but unable to connect to its database.

It might be better to temporarily remove it from traffic.

---

# 29. Graceful Degradation

Sometimes a dependency fails.

Imagine your shopping application depends on:

```text
Product Service
Recommendation Service
Review Service
Payment Service
```

Suppose recommendations are down.

Does the entire website need to fail?

Probably not.

Instead:

```text
Product page
   |
   +---- Product data ✓
   |
   +---- Reviews ✓
   |
   +---- Recommendations ✗
```

You could still show the product.

Maybe:

```text
"Recommendations temporarily unavailable."
```

The important functionality continues working.

This is called **graceful degradation**.

---

# 30. Idempotency

Imagine a user clicks:

```text
Pay ₹1,000
```

The request reaches your server.

But the network times out.

The client doesn't know whether the payment succeeded.

So it retries.

Now the server receives:

```text
Pay ₹1,000
```

again.

You definitely don't want:

```text
₹1,000 charged
₹1,000 charged again
```

Idempotency helps prevent duplicate operations.

A common approach is an idempotency key:

```text
Idempotency-Key: abc123
```

The server remembers:

```text
abc123 → payment already processed
```

If the same request arrives again:

```text
Don't create another payment.
Return the existing result.
```

This concept is extremely important in distributed systems.

---

# 31. Pagination

Imagine:

```text
GET /users
```

returns:

```text
10 million users
```

That's obviously a bad API design.

Instead:

```text
GET /users?page=1&limit=20
```

or cursor pagination:

```text
GET /users?cursor=abc123&limit=20
```

Pagination helps control:

* response size
* database work
* memory usage
* network bandwidth
* client processing

You already learned this in the API chapter.

Now you're seeing why it matters in system design.

---

# 32. API Versioning

Imagine you have:

```text
GET /api/v1/users
```

Thousands of clients depend on it.

You suddenly change the response format.

Old clients might break.

Versioning can help:

```text
/api/v1/users
/api/v2/users
```

This gives clients time to migrate.

Again, system design is not isolated knowledge.

Everything you learned earlier connects here.

---

# 33. Finding Bottlenecks

When designing a system, always ask:

> **What will probably break first?**

For example:

```text
Users
  |
  v
Load Balancer
  |
  v
API Servers
  |
  v
Redis
  |
  v
Database
```

Maybe the API servers can handle:

```text
10,000 req/sec
```

but the database can handle only:

```text
2,000 queries/sec
```

Then adding more API servers doesn't solve the problem.

You have simply created:

```text
More servers
     |
     v
More database pressure
     |
     v
Database becomes slower
```

So you need to identify the actual bottleneck.

---

# 34. A Practical System Design Process

When someone asks:

> "Design X."

Don't panic.

Use a repeatable process.

## Step 1 — Clarify requirements

Ask:

```text
What does the system need to do?
Who uses it?
How many users?
```

---

## Step 2 — Estimate scale

Think about:

```text
Requests/sec
Peak traffic
Storage
Bandwidth
Read/write ratio
```

You don't need perfect numbers.

---

## Step 3 — Define APIs

For example:

```text
POST /urls
GET /urls/:id
GET /:shortCode
```

Think about:

```text
Request
Response
Errors
Authentication
Pagination
```

---

## Step 4 — Define data model

For a URL shortener:

```text
URL
----------------
id
short_code
original_url
user_id
created_at
expires_at
```

---

## Step 5 — Draw the simplest architecture

Start with:

```text
Client
  |
  v
API
  |
  v
Database
```

Don't immediately add 15 services.

---

## Step 6 — Find bottlenecks

Ask:

```text
What happens if traffic increases?
What happens if database becomes slow?
What happens if one server dies?
```

---

## Step 7 — Add scaling solutions

Maybe:

```text
Load Balancer
Multiple API Servers
Redis
Read Replicas
Queue
CDN
Object Storage
```

But only where necessary.

---

## Step 8 — Think about failures

Ask:

```text
What if Redis is down?
What if database is down?
What if a worker crashes?
What if a request is retried?
What if a dependency becomes slow?
```

This is where good system design starts becoming production engineering.

---

# 35. Let's Design a URL Shortener

Let's put everything together.

Imagine we're building something like:

```text
tiny.example/abc123
```

which redirects to:

```text
https://example.com/very/long/url
```

## Requirements

Functional:

```text
Create short URL
Redirect short URL
```

Non-functional:

```text
Low redirect latency
High availability
Handle high read traffic
```

Assume:

```text
100 million URLs
10,000 redirects/sec
100 creates/sec
```

Notice something interesting:

```text
Reads = 10,000/sec
Writes = 100/sec
```

This is a read-heavy system.

That affects our architecture.

---

# 36. URL Shortener Data Model

We could have:

```text
urls
--------------------------------
id
short_code
original_url
created_at
expires_at
```

The important lookup is:

```text
short_code → original_url
```

So we need an index on:

```text
short_code
```

---

# 37. Basic Architecture

Start simple:

```text
             Client
                |
                v
              DNS
                |
                v
         Load Balancer
                |
       +--------+--------+
       |        |        |
       v        v        v
    API 1     API 2     API 3
       |        |        |
       +--------+--------+
                |
              Redis
                |
                v
            PostgreSQL
```

For redirects:

```text
GET /abc123
      |
      v
API Server
      |
      v
Redis
      |
   Cache Hit?
    /      \
  Yes       No
   |         |
   v         v
Return     PostgreSQL
URL          |
             v
          Redis
             |
             v
         Return URL
```

This works well because redirects are read-heavy.

---

# 38. Why Redis?

Suppose:

```text
abc123 → https://example.com/...
```

is requested thousands of times.

Without caching:

```text
Thousands of requests
        |
        v
   PostgreSQL
```

With caching:

```text
Thousands of requests
        |
        v
      Redis
        |
        v
Most requests stop here
```

The database handles fewer reads.

---

# 39. What If Redis Goes Down?

This is where beginners often make a mistake.

They think:

> "If Redis is down, everything is down."

Not necessarily.

If Redis is only a cache, the application can potentially fall back to PostgreSQL.

```text
Request
  |
  v
Redis
  |
  X unavailable
  |
  v
PostgreSQL
```

Performance may be worse, but the system can continue.

That's an example of graceful degradation.

However, this depends on what Redis is being used for. If Redis contains essential state and there is no fallback, the behavior is different.

---

# 40. What If PostgreSQL Becomes the Bottleneck?

Possible approaches:

```text
Better indexes
Connection pooling
Caching
Read replicas
Database optimization
Partitioning
Sharding
```

But don't jump directly to sharding.

First ask:

> "Have we actually exhausted the simpler options?"

This is one of the biggest lessons in system design.

---

# 41. What If Traffic Doubles?

Suppose:

```text
10,000 requests/sec
```

becomes:

```text
20,000 requests/sec
```

We could add more API servers:

```text
              Load Balancer
             /      |      \
            v       v       v
         Server  Server  Server
            |       |       |
            +-------+-------+
                    |
                  Redis
                    |
                 Database
```

This works well if the application servers are stateless and the bottleneck isn't elsewhere.

---

# 42. What If One API Server Dies?

Suppose:

```text
Server 2 ✗
```

The load balancer can stop sending traffic to it.

```text
              Load Balancer
              /          \
             v            v
         Server 1      Server 3
```

Users may not even notice.

That's the goal of redundancy.

---

# 43. What If the Database Dies?

This is much harder.

You might use:

```text
Primary
   |
   +----> Replica
   |
   +----> Replica
```

and a failover mechanism.

But now you need to think about:

```text
Replication lag
Failover time
Data loss
Consistency
Connection handling
Recovery
```

This is why databases are often one of the hardest parts of system design.

---

# 44. The Most Important System Design Lesson

You don't need to memorize architecture diagrams.

You need to understand the questions behind them.

When you see:

```text
Load Balancer
```

ask:

> What problem does it solve?

When you see:

```text
Redis
```

ask:

> What are we caching and why?

When you see:

```text
Queue
```

ask:

> Why doesn't this work need to happen synchronously?

When you see:

```text
Replica
```

ask:

> Is the workload read-heavy?

When you see:

```text
CDN
```

ask:

> What content can be served closer to users?

When you see:

```text
Sharding
```

ask:

> Has one database actually become too large or too busy?

That mindset is much more valuable than memorizing diagrams.

---

# 45. Common System Design Mistakes

## Mistake 1: Adding every technology

```text
Redis
Kafka
Kubernetes
Microservices
MongoDB
PostgreSQL
Elasticsearch
GraphQL
CDN
API Gateway
```

Looks impressive.

But maybe your application has:

```text
500 users
```

You don't need all of that.

---

## Mistake 2: Starting with architecture

Don't immediately draw:

```text
Load Balancer → 20 Services → Kafka → Redis → Sharding
```

First understand the requirements.

---

## Mistake 3: Ignoring numbers

Saying:

> "The system should handle a lot of traffic."

doesn't help.

Say:

```text
10,000 requests/sec
```

or whatever reasonable assumption you're making.

---

## Mistake 4: Ignoring failures

Always ask:

```text
What if the database fails?

What if Redis fails?

What if a server crashes?

What if the network is slow?

What if a request is duplicated?
```

---

## Mistake 5: Ignoring data consistency

Ask:

> "Is stale data acceptable?"

The answer affects your architecture.

---

## Mistake 6: Solving hypothetical scale

Don't optimize for:

```text
1 billion users
```

when you currently have:

```text
1,000 users
```

Design for the expected requirements and a sensible growth path.

---

## Mistake 7: Forgetting cost

Every component costs something.

More:

```text
Servers
Databases
Replicas
Caches
Queues
Storage
Bandwidth
```

means more operational complexity and usually more cost.

A good architecture is not just technically impressive.

It is economically sensible.

---

# 46. System Design Interview Questions

## Q1. What is system design?

**Answer:**

System design is the process of deciding how different components of a software system should work together to satisfy functional and non-functional requirements.

It includes things like architecture, databases, APIs, caching, scaling, reliability, security, and failure handling.

---

## Q2. What is horizontal scaling?

**Answer:**

Horizontal scaling means adding more machines or instances instead of making one machine more powerful.

For example:

```text
Server 1
Server 2
Server 3
```

A load balancer can distribute traffic between them.

---

## Q3. What is vertical scaling?

**Answer:**

Vertical scaling means increasing the resources of an existing machine.

For example:

```text
8 GB RAM → 32 GB RAM
```

It is simple but has physical and cost limits.

---

## Q4. Why do we use load balancers?

**Answer:**

A load balancer distributes incoming traffic across multiple servers.

It helps with:

* horizontal scaling
* availability
* failure handling
* traffic distribution

---

## Q5. Why do we cache data?

**Answer:**

Caching allows frequently accessed data to be served faster and can reduce load on the database.

The tradeoff is that cached data can become stale and introduces cache-management complexity.

---

## Q6. What is a single point of failure?

**Answer:**

A single point of failure is a component whose failure can bring down the entire system.

For example, if an application has only one backend server and that server crashes, the application becomes unavailable.

---

## Q7. What is eventual consistency?

**Answer:**

Eventual consistency means that after a successful update, different parts of a distributed system may temporarily see different values, but they eventually converge to the updated state.

---

## Q8. What is CAP theorem?

**Answer:**

CAP describes tradeoffs in distributed systems involving consistency, availability, and partition tolerance.

When a network partition occurs, a distributed system has to make a tradeoff between consistency and availability.

---

## Q9. Why should backend servers be stateless?

**Answer:**

Stateless servers make horizontal scaling easier because any server can handle a request without depending on memory stored on another server.

Shared state can be stored in systems such as databases, Redis, or object storage.

---

## Q10. Why use a queue?

**Answer:**

Queues allow work to be processed asynchronously.

They are useful when tasks are slow, retryable, or don't need to block the user's request.

Examples include:

```text
Email
Notifications
Image processing
Reports
Analytics
```

---

# 47. System Design Scenario Questions

Interviewers may also ask questions like:

### Scenario 1

> "Your API currently handles 1,000 requests/sec but traffic is expected to increase to 10,000. What would you do?"

Don't immediately answer:

> "Add Kubernetes."

Think:

```text
Can the API servers scale horizontally?
Are they stateless?
Where is the bottleneck?
Database?
CPU?
Network?
External API?
```

Then choose the solution.

---

### Scenario 2

> "Your database is receiving too many read queries. What can you do?"

Possible options:

```text
Caching
Indexes
Query optimization
Read replicas
Connection pooling
```

The correct answer depends on the actual bottleneck.

---

### Scenario 3

> "Your API is slow because it sends emails before responding."

Solution:

```text
API
 |
 v
Queue
 |
 v
Email Worker
```

Move email sending to a background job.

---

### Scenario 4

> "A user accidentally submits the same payment request twice."

Think:

```text
Idempotency
```

Use an idempotency key and make the operation safe to retry.

---

### Scenario 5

> "Your recommendation service is down. Should the entire product page fail?"

Not necessarily.

Think:

```text
Graceful degradation
```

Return the important product data and omit recommendations temporarily.

---

# 48. A Simple System Design Checklist

When you get a system design problem, walk through this list:

```text
1. Requirements
        ↓
2. Users and traffic
        ↓
3. Functional requirements
        ↓
4. Non-functional requirements
        ↓
5. API design
        ↓
6. Data model
        ↓
7. Basic architecture
        ↓
8. Scaling
        ↓
9. Caching
        ↓
10. Database scaling
        ↓
11. Async processing
        ↓
12. Failure handling
        ↓
13. Security
        ↓
14. Monitoring
        ↓
15. Tradeoffs
```

You don't have to discuss every item every time.

But this checklist prevents you from forgetting the important things.

---

# 49. The Backend Mental Model

After completing this series, you should be able to look at a backend system like this:

```text
                     USERS
                       |
                       v
                      DNS
                       |
                       v
                      CDN
                       |
                       v
                Load Balancer
                       |
          +------------+------------+
          |            |            |
          v            v            v
       API 1        API 2        API 3
          |            |            |
          +------------+------------+
                       |
          +------------+------------+
          |                         |
          v                         v
        Redis                    Queue
          |                         |
          |                    +----+----+
          |                    |         |
          |                    v         v
          |                 Worker 1  Worker 2
          |
          v
      PostgreSQL
          |
      +---+---+
      |       |
      v       v
   Replica  Replica
```

And you should be able to explain:

```text
DNS
↓
Finds the infrastructure

CDN
↓
Serves cacheable content closer to users

Load Balancer
↓
Distributes traffic

API Servers
↓
Handle business logic

Redis
↓
Handles frequently accessed/shared fast data

Queue
↓
Moves slow work outside the request

Workers
↓
Process background jobs

PostgreSQL
↓
Stores durable relational data

Replicas
↓
Provide additional read capacity/redundancy
```

But remember:

> **This is an example architecture, not a template you must use everywhere.**

---

# 50. What You Should Be Able to Do Now

After this chapter, you shouldn't expect yourself to design:

```text
YouTube
Netflix
Amazon
Google Search
```

perfectly.

That's not the goal.

You should be able to look at a backend problem and start asking the right questions.

For example:

```text
How many users?

How much traffic?

Read-heavy or write-heavy?

Where is the data stored?

Can we cache it?

Do we need asynchronous processing?

Can the backend scale horizontally?

What happens if one server fails?

What happens if the database becomes slow?

Do we need replicas?

Do we need object storage?

Is eventual consistency acceptable?

Where are the bottlenecks?

What is the simplest architecture that solves the problem?
```

That's the beginning of real system design thinking.

---

# 51. Mini Project — Design a Scalable URL Shortener

Now don't just read this chapter.

Build something.

Your project:

> **Build a production-minded URL shortener.**

Start with:

```text
POST /urls
```

Create a short URL.

```text
GET /:shortCode
```

Redirect the user.

Then progressively add:

### Level 1

```text
Node.js
Express
PostgreSQL
```

### Level 2

Add:

```text
Authentication
URL ownership
Expiration
Pagination
```

### Level 3

Add:

```text
Redis caching
Rate limiting
```

### Level 4

Add:

```text
Background click analytics
Queue
Worker
```

### Level 5

Dockerize everything.

### Level 6

Add:

```text
Logging
Health checks
Metrics
Graceful shutdown
```

### Level 7

Think about:

```text
Multiple API servers
Load balancing
Database replicas
Failure scenarios
```

Don't build all of this on day one.

Build it progressively.

The goal is to understand **why each piece exists**.

---

# 52. Final Interview Exercise

Take a blank piece of paper.

Write:

> **Design a URL Shortener**

Now, without looking at this chapter, try to answer:

```text
1. What are the requirements?

2. How many requests/sec?

3. What are the main APIs?

4. What does the database look like?

5. What should be indexed?

6. Is the workload read-heavy or write-heavy?

7. Where would you use caching?

8. How would you scale the backend?

9. What happens if Redis fails?

10. What happens if a server fails?

11. What happens if the database fails?

12. How would you prevent abuse?

13. How would you monitor the system?

14. What tradeoffs are you making?
```

If you can explain your decisions clearly, you're already doing system design.

---

# 53. Final Mental Model

System design is not about memorizing:

```text
Kafka
Redis
Kubernetes
Microservices
Sharding
CDN
Load Balancers
```

It's about understanding:

```text
Problem
   ↓
Requirements
   ↓
Constraints
   ↓
Simple Design
   ↓
Bottleneck
   ↓
Scale
   ↓
Failure
   ↓
Tradeoff
```

And this is probably the most important lesson from the entire Backend First Principles series:

> **Don't start with technology. Start with the problem.**

A good backend engineer doesn't say:

> "Let's use Redis."

They say:

> "This endpoint is read-heavy, the same data is requested frequently, and database latency is becoming a bottleneck. A cache could reduce database load."

That's the difference.

You stop choosing technologies because they're popular.

You start choosing them because they solve a specific problem.

---

# 54. Where You Are Now

If you've completed the previous chapters and actually practiced them, you now have a strong backend foundation.

You understand:

```text
Internet
HTTP
Web Servers
Routing
APIs
JSON
Databases
SQL
ORMs
Authentication
Password Security
API Design
Caching
Queues
WebSockets
File Uploads
Logging
Testing
Configuration
Docker
Database Scaling
Microservices
CI/CD
System Design
```

More importantly, you should start seeing how these concepts connect.

For example:

```text
HTTP
  ↓
API
  ↓
Authentication
  ↓
Database
  ↓
Cache
  ↓
Queue
  ↓
Worker
  ↓
Logging
  ↓
Monitoring
  ↓
Docker
  ↓
CI/CD
  ↓
Scaling
  ↓
System Design
```

That's the bigger picture.

You are no longer just learning:

> "How do I write an Express route?"

You're learning:

> **"How does a backend system actually work?"**

And that's what this entire repository was meant to teach.

---

# Final Challenge

Don't stop at reading.

Pick one project and build it properly.

For example:

```text
Todo API
Blog Platform
Chat Application
URL Shortener
```

Start simple.

Then ask:

```text
How would I handle 10 users?

How would I handle 1,000?

What changes at 100,000?

What breaks first?

How do I know it broke?

How do I recover?

How do I deploy it?

How do I scale it?
```

That loop is where system design becomes real.

**Build → Measure → Find the bottleneck → Improve → Repeat.**

That's how you move from learning backend concepts to thinking like a backend engineer.