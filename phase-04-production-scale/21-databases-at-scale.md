# Chapter 21 — Databases at Scale

At the beginning of this series, we used a database pretty simply.

Our backend receives a request:

```text
Client
   ↓
API
   ↓
Database
   ↓
Response
```

For a small application, this works perfectly fine.

Maybe you have:

```text
100 users
1,000 users
10,000 users
```

and PostgreSQL handles everything without much trouble.

Then the application grows.

Suddenly you have:

```text
1 million users
100 million rows
thousands of requests/second
many database connections
large queries
heavy reports
background jobs
multiple API servers
```

Now the database can become one of the biggest bottlenecks in the system.

This chapter is about understanding what happens next.

---

# 1. What Does "Database at Scale" Actually Mean?

When people say:

> "We need to scale the database."

That sentence is actually too vague.

A database can struggle for different reasons.

For example:

```text
Too many connections
        ↓
Connection bottleneck

Too many reads
        ↓
Read bottleneck

Too many writes
        ↓
Write bottleneck

Huge tables
        ↓
Storage/query bottleneck

Expensive queries
        ↓
CPU bottleneck

Too much data
        ↓
Memory/storage bottleneck
```

So before choosing a scaling technique, ask:

> **What exactly is becoming the bottleneck?**

This is one of the most important habits in system design.

Don't jump directly to:

> "Let's add sharding."

Maybe your actual problem is a missing index.

---

# 2. Start With the Simplest Database

Suppose our application looks like this:

```text
                  API Server
                      │
                      ↓
                 PostgreSQL
```

And the database handles:

```text
reads
writes
users
posts
comments
payments
notifications
```

This is basically a single database architecture.

It's simple.

And that's a good thing.

A single database gives you:

* simple development
* simple transactions
* simple queries
* easy debugging
* fewer moving parts
* strong consistency within the database

You should not introduce distributed database architecture just because your application might become big someday.

Start simple.

Scale when you have a reason.

---

# 3. First Question: Is the Database Actually Slow?

Suppose users complain:

> "The application is slow."

Don't immediately blame PostgreSQL.

The request could be slow because of:

```text
Client
  ↓
Network
  ↓
Load Balancer
  ↓
API
  ↓
Cache
  ↓
Database
  ↓
External API
```

Maybe the database takes:

```text
20 ms
```

while an external API takes:

```text
800 ms
```

The database isn't the problem.

This is why monitoring from Chapter 17 matters.

You want to measure:

```text
Query latency
Connection usage
CPU
Memory
Disk I/O
Cache hit ratio
Lock contention
Rows scanned
Rows returned
Queries per second
```

Measure first.

Then optimize.

---

# 4. Database Scaling Usually Happens in Stages

A useful progression looks like this:

```text
1. Good schema
      ↓
2. Good queries
      ↓
3. Indexes
      ↓
4. Connection pooling
      ↓
5. Caching
      ↓
6. Read replicas
      ↓
7. Partitioning
      ↓
8. Vertical scaling
      ↓
9. Sharding
```

This isn't a strict rule.

Sometimes your architecture needs something earlier.

But the general idea is:

> **Exhaust the simple solutions before introducing distributed complexity.**

---

# 5. Vertical Scaling

The simplest way to make a database stronger is to give it more resources.

For example:

```text
Before

4 CPU
16 GB RAM
500 GB SSD


After

16 CPU
64 GB RAM
2 TB SSD
```

This is called **vertical scaling**.

Conceptually:

```text
Database
   ↓
More CPU
More RAM
Faster storage
```

It's simple.

You don't need to change your application architecture.

---

# 6. Horizontal Scaling

Horizontal scaling means adding more machines/instances.

Instead of:

```text
        Database
```

you might have:

```text
       Database
       /      \
      DB      DB
```

or:

```text
DB 1
DB 2
DB 3
DB 4
```

This is more complicated because now you need to decide:

> Which data goes where?

That's where replication, read replicas, partitioning, and sharding enter the picture.

---

# 7. What Is Database Replication?

Replication means maintaining copies of database data on multiple database instances.

A simple architecture:

```text
              Primary
                 │
          Replication
           /         \
          ↓           ↓
      Replica 1    Replica 2
```

The primary handles writes.

Replicas maintain copies of the data.

For example:

```text
INSERT
UPDATE
DELETE
    ↓
 Primary
    ↓
Replicas
```

The exact replication behavior depends on the database and configuration.

---

# 8. Primary vs Replica

A common setup is:

```text
             Primary
                │
          ┌─────┴─────┐
          ↓           ↓
      Replica 1    Replica 2
```

The primary generally handles writes:

```text
INSERT
UPDATE
DELETE
```

Replicas can handle reads:

```text
SELECT
```

This gives us a way to scale read-heavy workloads.

---

# 9. Read Replicas

Imagine our application receives:

```text
90,000 reads
10,000 writes
```

per second.

Sending everything to one database may become expensive.

We can use:

```text
                 Primary
                /       \
               ↓         ↓
          Replica 1   Replica 2
               ↑         ↑
               └────Reads┘
```

The application can route:

```text
Writes → Primary

Reads → Replicas
```

This is called **read/write splitting**.

---

# 10. Read/Write Splitting

Conceptually:

```text
                 API
                  │
          ┌───────┴───────┐
          ↓               ↓
       Write            Read
          ↓               ↓
      Primary        Read Replica
```

For example:

```text
POST /users
      ↓
Primary


GET /users/123
      ↓
Replica
```

But there is an important problem.

---

# 11. Replication Lag

Replication is not always instantaneous.

Suppose:

```text
User creates a post
        ↓
Write goes to primary
        ↓
Primary has the new post
        ↓
Replica hasn't received it yet
```

Now:

```text
POST /posts
    ↓
Primary

GET /posts
    ↓
Replica
```

The GET might temporarily not show the newly created post.

This is **replication lag**.

---

# 12. Why Replication Lag Matters

Imagine a user changes their email:

```text
PUT /profile
```

Then immediately loads:

```text
GET /profile
```

If the write goes to the primary and the read goes to a lagging replica:

```text
Update succeeded
        ↓
Read immediately
        ↓
Old data
```

The user might think:

> "My update didn't work."

It did.

The replica just hasn't caught up yet.

---

# 13. Read-After-Write Consistency

One solution is to route certain reads to the primary.

For example:

```text
After write
    ↓
Read from primary
```

for some period or workflow.

Another approach is to use consistency-aware routing.

The important lesson isn't one particular implementation.

It's:

> **Read replicas introduce consistency considerations.**

---

# 14. When Should You Use Read Replicas?

They're useful when:

```text
Reads >> Writes
```

and your database's read workload is becoming a bottleneck.

For example:

```text
90% reads
10% writes
```

can be a good candidate.

But read replicas don't solve heavy write workloads.

If your primary can't keep up with writes:

```text
10,000 writes/sec
```

adding more read replicas doesn't magically increase write capacity.

The primary is still receiving the writes.

---

# 15. Connection Pooling

This is one of the most important backend scaling concepts.

Imagine:

```text
100 API requests
```

and every request creates a brand-new database connection.

That's inefficient.

Creating database connections has overhead.

Instead, we use a **connection pool**.

Think of it like a pool of reusable connections:

```text
            Connection Pool
        ┌────┬────┬────┬────┐
        │ C1 │ C2 │ C3 │ C4 │
        └────┴────┴────┴────┘
             ↑    ↑
             │    │
          Requests
```

A request borrows a connection.

It performs its database work.

Then it returns the connection to the pool.

---

# 16. Without Connection Pooling

Conceptually:

```text
Request 1
   ↓
Create DB connection
   ↓
Query
   ↓
Close connection


Request 2
   ↓
Create DB connection
   ↓
Query
   ↓
Close connection
```

Repeated thousands of times.

Not ideal.

---

# 17. With Connection Pooling

```text
                Connection Pool
              ┌───┬───┬───┬───┐
              │ C1│ C2│ C3│ C4│
              └───┴───┴───┴───┘
                ↑   ↑   ↑
                │   │   │
             Requests
```

Connections are reused.

This reduces connection setup overhead.

---

# 18. But Bigger Pool Isn't Always Better

A common mistake is:

> "Our application is slow. Increase the database connection pool to 500."

That can make things worse.

Suppose PostgreSQL can comfortably handle:

```text
100 active connections
```

and suddenly you have:

```text
20 API servers
×
100 connections each
=
2,000 possible connections
```

Now the database can become overwhelmed.

Connection pooling needs to be sized based on:

```text
Database capacity
Application concurrency
Query duration
Number of application instances
Workload
```

The pool is shared capacity, not free performance.

---

# 19. A Scaling Trap With Multiple API Servers

Imagine:

```text
API 1 → 100 DB connections
API 2 → 100 DB connections
API 3 → 100 DB connections
API 4 → 100 DB connections
```

That's potentially:

```text
400 connections
```

to the database.

Now scale to:

```text
20 API servers
```

Potentially:

```text
2,000 connections
```

This is why database connection planning becomes important as the application scales horizontally.

---

# 20. Connection Pooling Mental Model

Think:

```text
Application instances
        │
        ↓
Connection Pools
        │
        ↓
Database
```

Every application instance can have its own pool.

So you need to think about the **total** number of connections.

---

# 21. Database Indexes at Scale

We discussed indexes in Chapter 8.

At scale, they become even more important.

Suppose:

```text
users
```

contains:

```text
100 million rows
```

and you run:

```sql
SELECT *
FROM users
WHERE email = 'user@example.com';
```

Without a useful index, the database may need to examine a huge amount of data.

With an index:

```text
email index
    ↓
Find matching row
```

The database can avoid scanning the entire table in many cases.

---

# 22. But Indexes Aren't Free

Indexes improve reads.

But they also have costs.

When you write:

```sql
INSERT
UPDATE
DELETE
```

the database may need to maintain relevant indexes.

Indexes also consume storage and memory.

So:

> Don't create an index on every column.

Ask:

```text
How is this column queried?
How selective is it?
What queries need it?
What is the write workload?
```

---

# 23. Composite Indexes

Sometimes queries filter by multiple columns.

For example:

```sql
SELECT *
FROM posts
WHERE author_id = 42
AND created_at > '2026-01-01';
```

A composite index might help:

```text
(author_id, created_at)
```

The order matters.

Database indexes aren't just:

> "Put an index on every WHERE column."

You need to understand your actual query patterns.

---

# 24. Query Performance Matters More Than Database Size

A database with:

```text
1 million rows
```

can have a terrible query.

A database with:

```text
500 million rows
```

can sometimes handle well-designed queries efficiently.

Don't think only in terms of row count.

Think:

```text
Query plan
Indexes
Data distribution
Access pattern
Rows scanned
Rows returned
Sorting
Joins
Locks
I/O
```

---

# 25. EXPLAIN

When a query is slow, one of the most useful tools in SQL is:

```sql
EXPLAIN
```

and often:

```sql
EXPLAIN ANALYZE
```

These help you understand how the database plans and executes a query.

For example:

```sql
EXPLAIN ANALYZE
SELECT *
FROM users
WHERE email = 'user@example.com';
```

You might discover:

```text
Sequential Scan
```

instead of an expected index-based lookup.

This gives you evidence rather than guessing.

---

# 26. Sequential Scan vs Index Scan

A simplified mental model:

### Sequential scan

```text
Row 1
 ↓
Row 2
 ↓
Row 3
 ↓
...
 ↓
Row 100,000,000
```

The database examines many rows.

### Index lookup

```text
Index
  ↓
Matching location
  ↓
Target row
```

The database can often find the relevant rows much more efficiently.

The optimizer decides the actual plan.

An index isn't guaranteed to be used just because you created it.

---

# 27. Caching Before More Database Scaling

Remember Chapter 13.

Sometimes the database is receiving the same read repeatedly.

For example:

```text
GET /products/123
```

might be requested thousands of times.

Instead of:

```text
Request
   ↓
Database
   ↓
Response
```

we can use:

```text
Request
   ↓
Redis
   │
   ├── Cache hit → Response
   │
   └── Cache miss
          ↓
       Database
          ↓
       Redis
          ↓
       Response
```

This can remove a significant amount of read traffic from the database.

---

# 28. Caching Isn't a Replacement for Database Scaling

Cache misses still hit the database.

Data needs invalidation.

Cached data can become stale.

And some workloads aren't cache-friendly.

So caching is another tool.

Don't turn:

> "We have a slow database"

into:

> "Let's cache everything."

Understand the access pattern first.

---

# 29. Partitioning

Now let's say one table becomes enormous.

For example:

```text
events
```

contains:

```text
5 billion rows
```

We may want to split the table into smaller logical pieces.

This is called **partitioning**.

Conceptually:

```text
events
  │
  ├── events_2024
  ├── events_2025
  └── events_2026
```

The database can use the partition key to target relevant partitions.

---

# 30. Why Partition?

Partitioning can help with:

* very large tables
* maintenance
* deleting old data
* query performance in appropriate workloads
* managing data by time/range/list

For example:

```text
events
    ↓
partition by month
```

Then:

```text
Query January 2026
        ↓
January partition
```

instead of considering every historical partition.

The actual performance depends on the database and query.

---

# 31. Partitioning Is Not Sharding

These terms are often confused.

### Partitioning

Splits data into partitions within a database system.

```text
One database
    │
    ├── Partition A
    ├── Partition B
    └── Partition C
```

### Sharding

Splits data across multiple database instances/nodes.

```text
              Application
             /     |     \
            ↓      ↓      ↓
         Shard 1 Shard 2 Shard 3
```

Sharding is a much bigger architectural decision.

---

# 32. What Is Sharding?

Sharding means distributing data across multiple database nodes.

Suppose we have users:

```text
User 1
User 2
...
User 100 million
```

Instead of keeping all users on one database:

```text
Database
```

we could distribute them:

```text
Shard 1
Users 1–25M

Shard 2
Users 25M–50M

Shard 3
Users 50M–75M

Shard 4
Users 75M–100M
```

Now each shard stores only part of the data.

---

# 33. Shard Key

The most important decision in sharding is often the **shard key**.

For example:

```text
user_id
```

might determine where a user's data lives.

Conceptually:

```text
user_id
   ↓
Shard calculation
   ↓
Shard 1 / 2 / 3 / 4
```

A common strategy is hashing:

```text
hash(user_id) → shard
```

Another approach could be ranges.

The right strategy depends heavily on workload and data distribution.

---

# 34. Why Sharding Is Hard

Suppose we have:

```text
Shard 1
Users A–M

Shard 2
Users N–Z
```

Now you run:

```sql
SELECT *
FROM users
WHERE country = 'India';
```

Which shard contains the data?

Potentially all of them.

The application or database infrastructure may need to query multiple shards.

This is called a **scatter-gather** pattern.

```text
             Query
            /  |  \
           ↓   ↓   ↓
       Shard 1 2  3
           \   |  /
            \  | /
             ↓
          Combine
```

That adds complexity.

---

# 35. Cross-Shard Queries

Imagine:

```text
users
orders
```

are sharded by:

```text
user_id
```

A query joining data across users and orders can become complicated if related data doesn't live together.

Distributed joins are much harder than normal database joins.

You may need:

* denormalization
* application-side aggregation
* careful shard-key design
* colocating related data

This is why sharding shouldn't be your first optimization.

---

# 36. Choosing a Good Shard Key

A good shard key should generally:

* distribute data reasonably evenly
* avoid hot spots
* support common access patterns
* avoid concentrating most traffic on one shard
* remain reasonably stable

A bad shard key can create:

```text
Shard 1 → 90% traffic
Shard 2 → 5%
Shard 3 → 5%
```

You've technically sharded the database.

But you haven't solved the bottleneck.

---

# 37. Hot Partitions and Hot Shards

Imagine we shard by:

```text
country
```

and one country has most of our users.

Then:

```text
India → 80% traffic
USA   → 10%
UK    → 5%
Other → 5%
```

The India shard becomes overloaded.

This is a **hot shard** problem.

The key lesson:

> Distribution of data isn't enough. You also need distribution of traffic.

---

# 38. Replication vs Sharding

This is a common interview question.

### Replication

Creates copies of the same data.

```text
Primary
   ↓
Replica
   ↓
Replica
```

Useful for:

* read scaling
* redundancy
* availability

### Sharding

Splits different data across nodes.

```text
Shard 1 → Part A
Shard 2 → Part B
Shard 3 → Part C
```

Useful for:

* distributing very large datasets
* distributing write/read workload
* scaling beyond one database's capacity

You can use both.

For example:

```text
Shard 1
 ├── Primary
 ├── Replica
 └── Replica

Shard 2
 ├── Primary
 ├── Replica
 └── Replica
```

Now things get much more interesting.

---

# 39. Database Failover

What happens if the primary database crashes?

Without redundancy:

```text
Application
     ↓
Primary
     X
```

Your application may stop working.

With replicas:

```text
             Primary
                X
                │
        ┌───────┴───────┐
        ↓               ↓
     Replica 1       Replica 2
```

A failover system can promote an eligible replica.

Conceptually:

```text
Primary fails
     ↓
Choose replica
     ↓
Promote replica
     ↓
Redirect writes
```

The exact mechanism depends on your database infrastructure.

---

# 40. Backups Are Not Replication

This distinction is extremely important.

Replication:

```text
Primary
   ↓
Replica
```

is mainly about keeping copies available.

Backups:

```text
Database
   ↓
Backup storage
```

are about recovery.

If bad application code runs:

```sql
DELETE FROM users;
```

the deletion can potentially replicate to your replicas.

So replicas aren't a substitute for backups.

You need proper backup and restore strategies.

---

# 41. Point-in-Time Recovery

For production databases, backups often need to support more than:

> "Restore yesterday's backup."

You may need:

> "Restore the database to a specific point before the accidental deletion."

This is the idea behind **point-in-time recovery**.

The exact implementation depends on the database system.

The important concept:

```text
Backup
+
Transaction/log history
        ↓
Restore to a chosen point in time
```

---

# 42. Database Transactions at Scale

Transactions become more interesting as systems grow.

A transaction might be:

```text
Create order
    +
Reduce inventory
    +
Create payment record
```

Within one database, a transaction can provide atomicity.

But what if these operations happen across different services/databases?

Now you have a distributed transaction problem.

For example:

```text
Order Service
     ↓
Database A

Payment Service
     ↓
Database B
```

A single normal database transaction cannot simply span both databases in the same way.

This is one reason distributed systems become harder.

We'll explore this more in system design and microservices.

---

# 43. Database Connection Pooling + Read Replicas

Let's combine two concepts.

Suppose we have:

```text
              Load Balancer
              /     |     \
             ↓      ↓      ↓
           API 1  API 2  API 3
             │      │      │
             └──────┼──────┘
                    │
             Connection Pools
                    │
          ┌─────────┴─────────┐
          ↓                   ↓
       Primary            Replicas
          ↑               /       \
          │              ↓         ↓
        Writes         Read 1    Read 2
```

Now our application can:

```text
Writes → Primary
Reads  → Replicas
```

while each API server uses a connection pool.

This is already a much more production-like architecture.

---

# 44. Database Scaling Isn't Just About More Hardware

Suppose your query is:

```sql
SELECT *
FROM orders
ORDER BY created_at DESC;
```

and there are:

```text
500 million orders
```

Adding CPU might help.

But you should first ask:

```text
Do we have the right index?
Do we need all columns?
Do we need all rows?
Can we paginate?
Can we cache?
Can we partition?
```

Good backend engineers don't only scale hardware.

They improve the workload.

---

# 45. Pagination Becomes Critical

This connects to Chapter 12.

Don't do:

```sql
SELECT *
FROM posts
LIMIT 1000000;
```

and send a million records to the client.

Instead:

```text
Client
 ↓
Request page
 ↓
Database
 ↓
Small result
```

For very large datasets, cursor/keyset pagination can often be more efficient than large offsets.

For example:

```text
GET /posts?cursor=abc
```

instead of repeatedly asking for huge offsets.

---

# 46. Avoid SELECT *

At scale, this can matter.

Instead of:

```sql
SELECT *
FROM users
WHERE id = 123;
```

consider:

```sql
SELECT id, name, email
FROM users
WHERE id = 123;
```

Why?

Because the application may not need:

```text
password_hash
large metadata
large JSON fields
internal columns
```

Reducing unnecessary data can reduce:

* database work
* network transfer
* application memory
* serialization cost

Again:

> Optimize based on actual workload.

---

# 47. Database Locks

When multiple transactions operate on the same data, they may need locks.

For example:

```text
Transaction A
    ↓
Update row

Transaction B
    ↓
Wants same row
```

Transaction B may have to wait.

At high traffic, lock contention can become a bottleneck.

So when diagnosing slow database performance, don't only look at CPU.

Also consider:

```text
Locks
Deadlocks
Long-running transactions
Connection waits
```

---

# 48. Long-Running Transactions

Suppose someone starts a transaction and keeps it open for minutes.

That can cause problems depending on the database and workload.

For example:

```text
BEGIN

UPDATE ...

... application waits ...

... user does something ...

COMMIT
```

Long transactions can hold locks or prevent cleanup/progress mechanisms.

Transactions should generally be kept as short as practical.

---

# 49. Deadlocks

A deadlock happens when transactions wait for each other.

For example:

```text
Transaction A
locks Row 1
waits for Row 2


Transaction B
locks Row 2
waits for Row 1
```

```text
A → waits for B
B → waits for A
```

Neither can continue.

Databases can detect and abort one transaction.

Your application should be prepared to handle transient transaction failures where appropriate.

---

# 50. Database Monitoring at Scale

Once your database becomes critical infrastructure, monitor it carefully.

Useful metrics include:

```text
CPU utilization
Memory usage
Disk usage
Disk I/O
Connections
Active queries
Query latency
Slow queries
Locks
Deadlocks
Replication lag
Cache hit ratio
Transactions/sec
Reads/sec
Writes/sec
Connection pool saturation
```

You want to answer:

> "Why is the database slow?"

with actual evidence.

Not:

> "Maybe PostgreSQL is having a bad day."

---

# 51. Database Scaling Decision Tree

When the database is slow, think like this:

```text
Database slow?
      │
      ↓
Measure first
      │
      ↓
What is the bottleneck?
      │
 ┌────┼─────────────┐
 ↓    ↓             ↓
Query CPU          Connections
 │     │             │
 ↓     ↓             ↓
Indexes Optimize   Pooling
Query   query      limits
 │
 ↓
Still insufficient?
      │
      ↓
Can caching help?
      │
      ↓
Read-heavy?
      │
      ↓
Read replicas
      │
      ↓
Huge tables?
      │
      ↓
Partitioning
      │
      ↓
Beyond one DB?
      │
      ↓
Sharding
```

This is a much better approach than:

```text
Database slow
     ↓
SHARD EVERYTHING
```

---

# 52. A Realistic Growth Story

Let's imagine our Todo application becomes popular.

### Stage 1

```text
10,000 users

API
 ↓
PostgreSQL
```

Simple.

---

### Stage 2

```text
100,000 users

API
 ↓
PostgreSQL
```

We improve:

```text
Indexes
Query performance
Connection pooling
```

---

### Stage 3

```text
1 million users

API
 ↓
Redis
 ↓
PostgreSQL
```

Add caching where useful.

---

### Stage 4

Read traffic grows:

```text
API
 │
 ├── Writes → Primary
 │
 └── Reads → Replicas
```

Add read replicas.

---

### Stage 5

A table becomes enormous:

```text
events
  ↓
Partition by time
```

---

### Stage 6

The dataset and workload exceed what one database cluster can reasonably handle:

```text
             Application
           /      |      \
          ↓       ↓       ↓
       Shard 1  Shard 2  Shard 3
```

Now we consider sharding.

Notice how we didn't start with sharding.

We earned the complexity.

---

# 53. Common Mistakes

## Mistake 1 — Sharding Too Early

You have:

```text
50,000 users
```

and you're already designing:

```text
12 database shards
```

Probably unnecessary.

Start simple.

---

## Mistake 2 — Increasing Connections Without Thinking

More connections can increase contention and resource usage.

Understand the database's capacity first.

---

## Mistake 3 — Adding Indexes Everywhere

Indexes have storage and write-maintenance costs.

Index based on real query patterns.

---

## Mistake 4 — Treating Replicas as Backups

They aren't.

You still need backups and recovery procedures.

---

## Mistake 5 — Assuming Replicas Are Immediately Consistent

Replication lag can exist.

Design read paths accordingly.

---

## Mistake 6 — Ignoring Slow Queries

A single inefficient query executed thousands of times can cause major problems.

Find your expensive queries.

---

## Mistake 7 — Selecting Huge Amounts of Data

Don't make the database and network move millions of rows when the client needs 20.

Use:

```text
Pagination
Filtering
Projection
```

---

## Mistake 8 — Ignoring Lock Contention

Sometimes CPU isn't the problem.

Your queries may simply be waiting on locks.

---

# 54. Interview Questions

## Q1. What is vertical scaling?

Vertical scaling means increasing the resources of an existing database server.

For example:

```text
More CPU
More RAM
Faster storage
```

---

## Q2. What is horizontal scaling?

Horizontal scaling means adding multiple database instances/nodes and distributing workload or data among them.

---

## Q3. What is database replication?

Replication means maintaining copies of database data on multiple database instances.

A common setup has one primary and one or more replicas.

---

## Q4. What is a read replica?

A read replica is a database instance that maintains a copy of the primary's data and can be used to serve read workloads.

It is especially useful for read-heavy systems.

---

## Q5. What is replication lag?

Replication lag is the delay between a change being committed on the primary and that change becoming available on a replica.

---

## Q6. What is read/write splitting?

It's routing writes to a primary database while routing eligible reads to read replicas.

```text
Write → Primary
Read  → Replica
```

---

## Q7. What is connection pooling?

Connection pooling maintains reusable database connections so applications don't have to create a new connection for every request.

---

## Q8. Why can too many database connections be a problem?

Each connection consumes resources.

Too many connections can increase memory usage, contention, context switching, and overall database pressure.

---

## Q9. What is partitioning?

Partitioning divides a large logical table into smaller partitions based on a partitioning strategy.

For example:

```text
events
 ↓
2024
2025
2026
```

---

## Q10. What is sharding?

Sharding distributes different subsets of data across multiple database nodes.

```text
Shard 1 → Data A
Shard 2 → Data B
Shard 3 → Data C
```

---

## Q11. What's the difference between replication and sharding?

Replication creates copies of the same data.

Sharding distributes different data across nodes.

```text
Replication
Same data → multiple nodes

Sharding
Different data → different nodes
```

---

## Q12. Why is sharding difficult?

Because queries, transactions, joins, migrations, rebalancing, and operational tasks can become distributed-system problems.

---

## Q13. What is a shard key?

A shard key is the value used to determine which shard stores a particular piece of data.

A poor shard key can cause uneven data or traffic distribution.

---

## Q14. Are read replicas always strongly consistent?

Not necessarily.

Depending on the replication setup, replicas can lag behind the primary.

---

## Q15. Are replicas backups?

No.

Replication improves availability and can help with read scaling, but it does not replace backups and disaster recovery.

---

# 55. Scenario-Based Interview Questions

## Scenario 1

> "Our PostgreSQL database has high CPU usage."

What would you do?

I wouldn't immediately increase the server size.

I'd first inspect:

```text
Slow queries
Query plans
Missing indexes
Expensive joins
Sorting
Aggregation
Queries executed most frequently
```

Then optimize the highest-impact queries.

If the workload is still too large, I'd consider scaling the database infrastructure.

---

## Scenario 2

> "Our application has 20 API servers and PostgreSQL is overwhelmed by connections."

I'd calculate the total possible connections:

```text
20 servers × pool size
```

Then I'd inspect actual connection usage and database capacity.

I'd tune pool sizes rather than blindly increasing them.

Depending on the infrastructure, I might also use an external connection pooler.

---

## Scenario 3

> "Our application is read-heavy."

I'd consider:

```text
Caching
Indexes
Query optimization
Read replicas
```

The right solution depends on what the measurements show.

---

## Scenario 4

> "Users sometimes don't see data immediately after creating it."

I'd investigate whether reads are being served from replicas.

If so, replication lag could be responsible.

I'd consider routing read-after-write operations appropriately or designing the consistency model around the product requirements.

---

## Scenario 5

> "Our orders table has billions of rows."

I wouldn't immediately shard it.

I'd first investigate:

```text
Indexes
Query patterns
Pagination
Archival
Partitioning
Data retention
Storage
Slow queries
```

If one database system still can't reasonably handle the workload, then I'd evaluate sharding or another architectural approach.

---

# 56. Mini Project — Scale the Todo API

Let's take the Todo API we've been building throughout this series.

Current architecture:

```text
Client
  ↓
API
  ↓
PostgreSQL
```

Now improve it gradually.

### Step 1 — Add Connection Pooling

Configure your database client/ORM properly so connections are reused.

Understand:

```text
API instances
    ↓
Connection pool
    ↓
PostgreSQL
```

---

### Step 2 — Analyze Queries

Find your most important endpoints:

```text
GET /todos
GET /todos/:id
POST /todos
PATCH /todos/:id
DELETE /todos/:id
```

Check:

```text
Which queries are executed?
Which are slow?
Which columns are filtered?
Which columns are sorted?
```

Add indexes based on those actual patterns.

---

### Step 3 — Add Pagination

Don't return every Todo.

Implement:

```text
GET /todos?limit=20
```

Then consider cursor-based pagination for larger datasets.

---

### Step 4 — Add Redis

Cache an appropriate read-heavy endpoint.

For example:

```text
GET /todos
```

Flow:

```text
Request
   ↓
Redis
   │
   ├── Hit → Response
   │
   └── Miss
         ↓
      PostgreSQL
         ↓
       Redis
         ↓
      Response
```

Then think carefully about invalidation.

---

### Step 5 — Add a Read Replica Conceptually

You don't need a production cluster just to understand the architecture.

Draw:

```text
              API
           /       \
       Writes      Reads
          ↓          ↓
       Primary    Replica
```

Then identify:

```text
Which endpoints can tolerate replica lag?
Which cannot?
```

---

### Step 6 — Add Partitioning Conceptually

Imagine your Todo API also records:

```text
todo_events
```

with billions of rows.

Design a partitioning strategy based on:

```text
created_at
```

For example:

```text
todo_events
     │
     ├── 2025
     ├── 2026
     └── 2027
```

Think about how old data could be archived.

---

# 57. Your Database Scaling Checklist

Before saying:

> "We need to scale the database."

check:

### Query

```text
Are queries efficient?
```

### Indexes

```text
Do important queries have appropriate indexes?
```

### Data

```text
Are we retrieving more data than necessary?
```

### Pagination

```text
Are large result sets paginated?
```

### Caching

```text
Can repeated reads be cached?
```

### Connections

```text
Is the connection pool correctly sized?
```

### Reads

```text
Are reads the actual bottleneck?
```

### Replicas

```text
Would read replicas help?
```

### Large tables

```text
Would partitioning help?
```

### Writes

```text
Is write throughput the bottleneck?
```

### Sharding

```text
Has the workload actually outgrown one database system?
```

This checklist can save you from introducing unnecessary complexity.

---

# 58. The Most Important Lesson

Database scaling isn't:

```text
Small DB
   ↓
Big DB
   ↓
Bigger DB
   ↓
Sharding
```

It's more like:

```text
Measure
  ↓
Understand workload
  ↓
Fix bad queries
  ↓
Add indexes
  ↓
Pool connections
  ↓
Cache where useful
  ↓
Scale vertically
  ↓
Add replicas for read scaling
  ↓
Partition large datasets
  ↓
Shard when necessary
```

Not every application reaches the last step.

Many applications can handle a lot of traffic without sharding.

---

# 59. Final Mental Model

Keep these concepts separate.

```text
Vertical Scaling
        ↓
Bigger database machine


Replication
        ↓
Copies of data
        ↓
Read scaling / availability


Connection Pooling
        ↓
Reuse DB connections


Caching
        ↓
Reduce database reads


Partitioning
        ↓
Split large tables logically


Sharding
        ↓
Split data across database nodes
```

And the bigger picture:

```text
                         Application
                              │
                    ┌─────────┴─────────┐
                    ↓                   ↓
                  Cache                DB
                    │                   │
                    │          ┌────────┴────────┐
                    │          ↓                 ↓
                    │       Primary           Replicas
                    │          │
                    │          ↓
                    │      Partitions
                    │
                    ↓
                 Response
```

The key idea:

> **Scale the database based on the actual bottleneck, not because the architecture diagram looks more impressive with more boxes.**

A good backend engineer should be comfortable with a single PostgreSQL database when that's all the system needs—and equally comfortable explaining what changes when that database becomes the bottleneck.

---

# What You Should Know After This Chapter

You should now be able to explain:

```text
Vertical vs horizontal scaling
Primary vs replica
Database replication
Replication lag
Read replicas
Read/write splitting
Connection pooling
Indexes at scale
Query optimization
EXPLAIN
Partitioning
Sharding
Shard keys
Hot shards
Database failover
Backups vs replication
Point-in-time recovery
Locks and deadlocks
Database monitoring
```

More importantly, you should be able to answer:

> **"Our database is slow. What would you do?"**

with something better than:

> "Add Redis and shard it."

You should start with:

> **"I'd first measure where the database is spending its time and identify whether the bottleneck is queries, CPU, I/O, connections, locks, reads, or writes. Then I'd choose the simplest solution that addresses that bottleneck."**

That's the kind of answer that shows actual engineering thinking.

---

# What's Next?

We've spent the last few chapters making our backend more production-ready:

```text
Testing
   ↓
Configuration
   ↓
Docker
   ↓
Database Scaling
```

But there's another architectural question that comes up when applications become larger:

> **Should everything live inside one backend application, or should we split the system into multiple services?**

For example:

```text
                 Backend
                    │
       ┌────────────┼────────────┐
       ↓            ↓            ↓
    Users        Payments      Orders
```

Should these remain one application?

Or should they become:

```text
User Service
Payment Service
Order Service
Notification Service
```

That's where the trade-offs between **monoliths and microservices** become important.

# Chapter 22 — Microservices vs Monolith