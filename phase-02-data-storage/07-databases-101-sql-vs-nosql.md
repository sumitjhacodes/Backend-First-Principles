# Chapter 7 — Databases 101: SQL vs NoSQL

So far, our Todo API has been doing something that is fine for learning but completely useless for a real application.

We're storing our data in an array:

```js
const todos = [
  {
    id: 1,
    title: "Learn backend",
    completed: false
  }
];
```

It works.

You can create a todo.

You can fetch it.

You can update it.

You can delete it.

But restart the server:

```text
Server stops
     ↓
Server starts again
     ↓
todos = []
```

Everything is gone.

That's obviously a problem.

A real application needs somewhere to **persist data**.

That's where databases come in.

---

# 1. What Is a Database?

Let's start with a very simple definition.

> A database is a system that stores and manages data so that applications can reliably save, find, update, and delete it.

That's the basic idea.

For our Todo application, we might want to store:

```text
Todo
├── id
├── title
├── completed
└── createdAt
```

Instead of keeping that data in memory:

```text
Node.js process
      ↓
JavaScript array
```

we store it in a database:

```text
Node.js application
        ↓
     Database
        ↓
      Data
```

Now if our server restarts:

```text
Server restarts
      ↓
Connect to database
      ↓
Read existing data
      ↓
Todos are still there
```

That's the first problem databases solve.

---

# 2. Why Not Just Use a File?

A beginner might reasonably ask:

> "Why don't we just save everything in a JSON file?"

For a tiny project, you actually could.

For example:

```text
todos.json
```

containing:

```json
[
  {
    "id": 1,
    "title": "Learn backend",
    "completed": false
  }
]
```

Your application could read and write that file.

So why do we need a database?

Because real applications have much harder problems.

Imagine:

```text
10,000 users
1 million posts
5 million comments
100 requests/second
```

Now you need things like:

* efficient searching
* indexes
* concurrent access
* transactions
* relationships
* permissions
* backups
* recovery
* consistency
* connection management
* reliable writes

A JSON file isn't designed to solve these problems.

A database is.

---

# 3. What Does a Database Actually Do?

At a high level, your application asks the database questions.

For example:

> Give me todo with ID 42.

Or:

> Give me all incomplete todos.

Or:

> Create a new user.

Or:

> Update this order.

The database handles storing and retrieving that data.

So your application might look like:

```text
Client
   ↓
HTTP request
   ↓
Node.js API
   ↓
Database query
   ↓
Database
   ↓
Result
   ↓
Node.js
   ↓
HTTP response
   ↓
Client
```

The database isn't replacing your backend.

It's working **with** your backend.

---

# 4. Database vs Database Management System

You'll hear people casually say:

> "I'm using PostgreSQL."

Technically, PostgreSQL is a **database management system (DBMS)**.

A DBMS is software that manages databases.

Examples include:

```text
PostgreSQL
MySQL
MongoDB
Redis
SQLite
```

You don't need to get stuck on terminology here.

In everyday backend conversations, people commonly just say:

> "database"

when they mean the database system they're using.

---

# 5. The Two Big Families We'll Talk About

There are many types of databases.

But for this chapter, we'll focus on two broad categories:

```text
SQL / Relational databases

NoSQL databases
```

Examples:

### SQL / Relational

```text
PostgreSQL
MySQL
SQL Server
SQLite
```

### NoSQL

```text
MongoDB
Redis
DynamoDB
Cassandra
```

These aren't all interchangeable.

And "NoSQL" isn't one single database type.

That's an important point.

---

# 6. What Does SQL Mean?

SQL stands for:

> **Structured Query Language**

SQL is a language used to communicate with relational databases.

For example:

```sql
SELECT * FROM users;
```

means:

> Give me all rows from the `users` table.

Another example:

```sql
SELECT * FROM todos
WHERE completed = false;
```

means:

> Give me todos that aren't completed.

SQL is the language.

PostgreSQL is a database system that supports SQL.

So don't say:

> "SQL is a database."

Instead:

```text
SQL
↓
Query language

PostgreSQL
↓
Relational database system
```

---

# 7. What Is a Relational Database?

A relational database stores data in **tables** and represents relationships between that data.

Imagine an application with users and posts.

You might have:

### users

| id | name  | email                                         |
| -- | ----- | --------------------------------------------- |
| 1  | Rahul | [rahul@example.com](mailto:rahul@example.com) |
| 2  | Aman  | [aman@example.com](mailto:aman@example.com)   |

And:

### posts

| id  | title            | user_id |
| --- | ---------------- | ------- |
| 101 | My first post    | 1       |
| 102 | Learning backend | 2       |
| 103 | PostgreSQL notes | 1       |

The `user_id` connects a post to a user.

That's a relationship.

```text
users
  ↓
user_id
  ↓
posts
```

That's where the word **relational** comes from.

---

# 8. Tables, Rows and Columns

If you're new to SQL, these three words are everywhere.

Imagine:

```text
users
```

| id | name  | email                                         |
| -- | ----- | --------------------------------------------- |
| 1  | Rahul | [rahul@example.com](mailto:rahul@example.com) |
| 2  | Aman  | [aman@example.com](mailto:aman@example.com)   |

### Table

```text
users
```

The entire collection of related data.

### Row

```text
1 | Rahul | rahul@example.com
```

One record.

### Column

```text
id
name
email
```

A field describing something about each record.

So:

```text
Table
 ├── Column
 ├── Column
 └── Column

Rows
 ├── Record
 ├── Record
 └── Record
```

You'll use this vocabulary constantly.

---

# 9. What Is a Schema?

A **schema** describes the structure of your data.

For example, our users table might require:

```text
id       → integer
name     → string
email    → string
createdAt → timestamp
```

You're essentially saying:

> "This is what a user record should look like."

In a relational database, schemas are usually defined explicitly.

For example:

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT NOT NULL
);
```

Don't worry about understanding every piece yet.

We'll go deep into SQL in Chapter 8.

For now, understand the idea:

```text
Schema
↓
Structure / rules for your data
```

---

# 10. Why Do We Need Schemas?

Imagine your users table has no rules.

One user has:

```text
name = Rahul
```

Another:

```text
name = 42
```

Another:

```text
name = {}
```

Another:

```text
email = "hello"
```

Another doesn't have an email at all.

Your application would become difficult to reason about.

A database schema lets you define rules.

For example:

```text
name must exist
email must exist
email must be unique
id must be unique
```

The database can enforce some of these rules for you.

This is important:

> **Your database isn't just a place to dump data. It can enforce rules about that data.**

---

# 11. What Is a Primary Key?

Every row usually needs a way to be uniquely identified.

For example:

```text
users

id
---
1
2
3
```

The `id` can be the **primary key**.

A primary key uniquely identifies a row.

So:

```text
User 1
User 2
User 3
```

are distinguishable.

For our Todo API:

```text
todos

id
---
1
2
3
```

`id` could be the primary key.

We'll go deeper into keys in Chapter 8.

---

# 12. What Is a Foreign Key?

Now suppose:

```text
users
```

contains:

```text
id = 1
```

and:

```text
posts
```

contains:

```text
user_id = 1
```

`user_id` can reference the user's primary key.

That's a **foreign key**.

It creates a relationship between tables.

Conceptually:

```text
users
  |
  | id
  ↓
posts
  |
  | user_id
  ↓
users.id
```

This is one of the most important concepts in relational databases.

---

# 13. Relationships

Real applications contain relationships everywhere.

For example:

```text
User
 ↓
Posts
 ↓
Comments
 ↓
Likes
```

An e-commerce application:

```text
Customer
 ↓
Orders
 ↓
Order Items
 ↓
Products
```

A social application:

```text
User
 ↓
Followers
 ↓
Posts
 ↓
Comments
 ↓
Likes
```

Relational databases are particularly good at representing these kinds of structured relationships.

---

# 14. What Is NoSQL?

NoSQL generally refers to database systems that don't primarily use the traditional relational table model.

The name is sometimes interpreted as:

> "Not Only SQL"

rather than simply:

> "No SQL."

NoSQL databases come in different forms.

For example:

```text
Document databases
Key-value stores
Wide-column databases
Graph databases
```

So saying:

> "NoSQL means MongoDB."

is incorrect.

MongoDB is one type of NoSQL database.

---

# 15. MongoDB

MongoDB is a **document database**.

Instead of primarily thinking in terms of:

```text
tables
rows
columns
```

you often work with:

```text
collections
documents
fields
```

A MongoDB document might look like:

```json
{
  "_id": "123",
  "name": "Rahul",
  "email": "rahul@example.com"
}
```

Another document could contain different fields:

```json
{
  "_id": "456",
  "name": "Aman",
  "email": "aman@example.com",
  "github": "aman-dev"
}
```

The structure can be more flexible than a traditional relational schema.

---

# 16. SQL vs NoSQL — The Simple Difference

A rough mental model:

### Relational database

```text
Database
   ↓
Tables
   ↓
Rows
   ↓
Columns
```

### Document database

```text
Database
   ↓
Collections
   ↓
Documents
   ↓
Fields
```

For example:

### PostgreSQL

```text
users
--------------------------------
id | name  | email
1  | Rahul | rahul@example.com
2  | Aman  | aman@example.com
```

### MongoDB

```json
{
  "_id": 1,
  "name": "Rahul",
  "email": "rahul@example.com"
}
```

Again, this is only the starting point.

Real databases are much more nuanced.

---

# 17. "SQL vs NoSQL" Is Not a Competition

This is important.

You'll see questions like:

> "Which is better, SQL or NoSQL?"

That's usually the wrong question.

It's like asking:

> "Which is better, a car or a truck?"

Depends on the job.

Instead ask:

> **What kind of data do I have?**

> **How are the data related?**

> **How do I need to query it?**

> **What consistency guarantees do I need?**

> **How will the application scale?**

> **What operational requirements do I have?**

Then choose the appropriate technology.

---

# 18. When Relational Databases Are a Great Fit

Relational databases are often a strong choice when you have:

### Strong relationships

```text
users
orders
products
payments
```

### Structured data

You know what your records should look like.

### Transactions

You need multiple operations to succeed or fail together.

For example:

```text
Create order
+
Reduce inventory
+
Create payment record
```

You don't want half of that operation to succeed and half to fail.

### Complex queries

You might need to combine information from several tables.

For example:

> Give me all customers who placed an order in the last 30 days and spent more than ₹10,000.

SQL is very good at expressing this kind of query.

---

# 19. When a Document Database Can Be a Good Fit

A document database can make sense when:

* data is naturally document-shaped
* schema changes frequently
* you often retrieve an entire document together
* relationships are relatively simple
* horizontal scaling requirements favor the chosen technology
* the team's operational needs fit it

For example, imagine product catalog data where different products can have very different attributes.

A laptop:

```json
{
  "name": "Laptop",
  "ram": "16GB",
  "storage": "1TB"
}
```

A shoe:

```json
{
  "name": "Running Shoe",
  "size": 9,
  "material": "Mesh"
}
```

A document database can represent these varying fields naturally.

But again:

> Flexible schema does not automatically mean MongoDB is the right answer.

You still need to understand your access patterns.

---

# 20. Flexible Schema Doesn't Mean "No Structure"

This is a common misunderstanding.

People sometimes hear:

> MongoDB is schema-flexible.

and think:

> "I can throw anything into it."

Technically, you can make a mess.

Imagine documents like:

```json
{
  "name": "Rahul"
}
```

then:

```json
{
  "fullName": "Aman Sharma"
}
```

then:

```json
{
  "username": 123
}
```

then:

```json
{
  "name": ["John"]
}
```

Your database may allow a lot of this.

But your application becomes painful to maintain.

**Schema flexibility does not mean schema doesn't matter.**

You still need good data design.

---

# 21. What Is ACID?

Now we reach one of the most important database concepts.

You will hear:

> **ACID transactions**

ACID stands for:

```text
A → Atomicity
C → Consistency
I → Isolation
D → Durability
```

Don't try to memorize the letters without understanding them.

Let's use a bank transfer.

Suppose:

```text
Account A: ₹10,000
Account B: ₹5,000
```

You transfer:

```text
₹1,000
```

We need:

```text
A → -₹1,000
B → +₹1,000
```

We don't want:

```text
A → -₹1,000
B → nothing
```

That would be terrible.

---

# 22. Atomicity

Atomicity means:

> A transaction happens completely or not at all.

Our transfer:

```text
Subtract ₹1,000 from A
+
Add ₹1,000 to B
```

Both should succeed.

Or neither should happen.

Think:

```text
ALL
or
NOTHING
```

That's atomicity.

---

# 23. Consistency

Consistency means a transaction should move the database from one valid state to another valid state while respecting its defined rules.

Before:

```text
A = ₹10,000
B = ₹5,000
```

After:

```text
A = ₹9,000
B = ₹6,000
```

The database should still obey its constraints and rules.

Don't confuse this with the broader distributed-systems meaning of "consistency." We'll encounter that later.

---

# 24. Isolation

Imagine two transactions happen at the same time.

```text
Transaction A
Transaction B
```

They shouldn't interfere with each other in ways that produce invalid results.

Isolation controls how concurrent transactions interact.

This becomes especially important when many users are changing data simultaneously.

We'll spend much more time on this in later chapters.

---

# 25. Durability

Once the database confirms that a transaction has been committed, the data should survive things like a normal application restart.

Conceptually:

```text
Write data
   ↓
Commit
   ↓
Server restarts
   ↓
Data still exists
```

That's durability.

This is one reason a database is different from:

```js
let todos = [];
```

---

# 26. What Is a Transaction?

A transaction is a group of database operations treated as one logical unit.

For example:

```text
Begin transaction

Create order
Reduce inventory
Create payment record

Commit
```

If something fails:

```text
Rollback
```

The goal is to avoid ending up with half-finished state.

We'll properly implement transactions when we learn PostgreSQL.

For now, remember:

> **A transaction groups related database operations into one unit of work.**

---

# 27. What Is Eventual Consistency?

Here's another term you'll hear when discussing distributed systems and some NoSQL architectures.

**Eventual consistency** means that after an update, different copies of data may temporarily disagree, but assuming no further updates and successful propagation, they should eventually converge.

Imagine:

```text
Primary database
      ↓
Replica A
Replica B
```

You update:

```text
username = "sumit"
```

The primary knows immediately.

A replica might briefly still have:

```text
username = "old-name"
```

A little later:

```text
Replica A → sumit
Replica B → sumit
```

Now they're consistent.

That's the basic idea.

---

# 28. Don't Say "NoSQL = Eventually Consistent"

This is an important interview correction.

You might hear:

> "SQL is consistent and NoSQL is eventually consistent."

That's far too simplistic.

Modern databases can support different consistency models and configurations.

Some NoSQL systems provide strong consistency for certain operations.

Some relational systems can also participate in distributed architectures with replication and eventual consistency.

So don't memorize:

```text
SQL → consistency
NoSQL → eventual consistency
```

Instead understand:

> **Consistency is a property of a particular database/system and its configuration, not a simple SQL-vs-NoSQL switch.**

---

# 29. What Is Redis?

You've already seen Redis in the roadmap.

Redis is commonly described as an **in-memory data store**.

It is extremely fast because data is primarily kept in memory.

You can use it for things like:

```text
Caching
Sessions
Rate limiting
Queues
Counters
Leaderboards
Temporary data
```

For example:

```text
User requests popular product
          ↓
Check Redis
          ↓
Cache hit?
     ↙         ↘
   Yes          No
    ↓            ↓
Return       Query database
             ↓
          Store in Redis
```

We'll learn Redis properly in Chapter 13.

For now, just understand:

> Redis is another kind of data store, and it isn't simply "another PostgreSQL."

---

# 30. SQL vs NoSQL vs Redis

A useful high-level comparison:

|               | Relational    | Document NoSQL              | Redis                                        |
| ------------- | ------------- | --------------------------- | -------------------------------------------- |
| Example       | PostgreSQL    | MongoDB                     | Redis                                        |
| Main model    | Tables        | Documents                   | Key-value/data structures                    |
| Relationships | Strong        | Usually handled differently | Not its main strength                        |
| Schema        | Structured    | More flexible               | Depends on how you model data                |
| Typical use   | Business data | Document-oriented data      | Cache, sessions, queues, fast temporary data |
| Query style   | SQL           | Document queries            | Commands/data structures                     |

This table is intentionally simplified.

The point is to build your first mental model, not memorize a database comparison chart.

---

# 31. So Which Database Should You Choose?

Here's how I'd approach it.

Don't start with:

> "I know MongoDB, so I'll use MongoDB."

Start with the application.

Imagine you're building:

## Banking application

You have:

```text
accounts
transactions
customers
payments
```

Data relationships and transaction correctness are extremely important.

A relational database such as PostgreSQL is a very natural candidate.

---

## Blog

You have:

```text
users
posts
comments
likes
```

Again, lots of relationships.

PostgreSQL would be a very reasonable choice.

---

## Product catalog

You might have products with very different attributes.

A document database could potentially make sense depending on the access patterns.

But PostgreSQL could also handle many such applications very well.

The important thing is not:

> "Product catalog = MongoDB."

It's:

> "What data do we have and how will we access it?"

---

## Session storage

You might have:

```text
sessionId → session data
```

Redis could be a great fit.

---

# 32. Don't Choose a Database Because It's Popular

This happens constantly.

Someone says:

> "MongoDB is popular."

So they use MongoDB.

Someone else says:

> "PostgreSQL is what serious companies use."

So they use PostgreSQL.

Neither is a database selection strategy.

Ask:

```text
What data am I storing?

How is it related?

What queries will I run?

What consistency do I need?

Do I need transactions?

What is the expected traffic?

What are the latency requirements?

How will the system be operated?

What does my team already know?
```

Then choose.

---

# 33. One Application Can Use Multiple Data Stores

This is another important real-world concept.

You don't necessarily have to choose exactly one database for everything.

For example:

```text
                    ┌── PostgreSQL
                    │    Users
                    │    Orders
                    │    Payments
                    │
Backend API ────────┼── Redis
                    │    Cache
                    │    Sessions
                    │
                    └── Object Storage
                         Images
                         Videos
                         Files
```

Each system can have a different job.

But don't add technologies just because you can.

A beginner application doesn't need:

```text
PostgreSQL
+
MongoDB
+
Redis
+
Kafka
+
Elasticsearch
```

just to look impressive.

Every additional system adds complexity.

Start simple.

---

# 34. The Database Is Not Your Business Logic

This is another useful distinction.

Your backend contains business logic.

For example:

```text
Can this user cancel the order?

Is this product in stock?

Can this user edit this post?
```

The database stores and protects data.

So:

```text
Backend
↓
Business rules

Database
↓
Data storage + querying + constraints + transactions
```

They work together.

You don't want to put every piece of business logic into SQL.

And you don't want your application to ignore database capabilities either.

Good backend engineering uses both appropriately.

---

# 35. What Happens When Your API Talks to a Database?

Let's return to our Todo API.

Previously:

```text
POST /todos
       ↓
JavaScript array
```

Now:

```text
POST /todos
       ↓
Node.js
       ↓
SQL INSERT
       ↓
PostgreSQL
```

For example:

```sql
INSERT INTO todos (title, completed)
VALUES ('Learn PostgreSQL', false);
```

The database stores it.

Then:

```text
Database
   ↓
Result
   ↓
Node.js
   ↓
JSON response
```

The client might receive:

```json
{
  "id": 1,
  "title": "Learn PostgreSQL",
  "completed": false
}
```

Now we have a real persistence layer.

---

# 36. What If Two Users Create Data at the Same Time?

This is where things start getting interesting.

Imagine:

```text
User A
   ↓
POST /todos

User B
   ↓
POST /todos
```

at almost exactly the same time.

Who gets:

```text
id = 1
```

and who gets:

```text
id = 2
```

A proper database handles concurrency and ID generation safely.

This is one of many reasons you don't want to build your own database using:

```js
todos.length + 1
```

in a production application.

That might work for a tutorial.

It isn't a reliable concurrency strategy.

---

# 37. What About 1 Million Rows?

Suppose:

```text
todos = 1,000,000 rows
```

and you want:

```text
Find todo with id = 500000
```

A simple JavaScript array search might have to inspect many elements.

Databases have mechanisms such as **indexes** to make common lookups much faster.

We'll learn indexes properly in Chapter 8.

For now:

> Databases aren't just files with extra steps. They have specialized data structures and algorithms for efficiently storing and retrieving data.

---

# 38. What About Backups?

Another reason databases matter.

Imagine your application has:

```text
5 years of customer data
```

and the server's disk dies.

Without backups:

```text
Data → gone
```

Production databases are usually operated with:

* backups
* recovery strategies
* replication
* monitoring
* access controls

We'll learn the scaling and production side later.

For now, understand that persistence also means **thinking about what happens when things fail.**

---

# 39. Common Beginner Mistakes

## Mistake 1 — "MongoDB is NoSQL"

MongoDB is a type of NoSQL database, specifically a document database.

NoSQL is a broader category.

---

## Mistake 2 — "SQL is a database"

SQL is a query language.

PostgreSQL is a relational database system that uses SQL.

---

## Mistake 3 — "NoSQL means no schema"

No.

Many NoSQL systems are schema-flexible, but your application still needs a sensible data model.

---

## Mistake 4 — "NoSQL is always faster"

No.

Performance depends on:

* data model
* query patterns
* indexes
* workload
* hardware
* configuration
* architecture

There is no universal:

```text
NoSQL > SQL
```

rule.

---

## Mistake 5 — "Use MongoDB because the data is JSON"

Just because your API sends JSON doesn't mean you need MongoDB.

Your API can send JSON while storing data in PostgreSQL.

For example:

```text
PostgreSQL
    ↓
Node.js object
    ↓
JSON
    ↓
Frontend
```

The database choice and API response format are separate decisions.

---

## Mistake 6 — "Database handles everything"

Your database isn't your entire backend.

You still need:

```text
Validation
Business logic
Authorization
Error handling
API design
Security
```

---

# 40. Interview Questions

## 1. What is a database?

A database is a system used to reliably store, organize, retrieve, and manage application data.

---

## 2. What is SQL?

SQL stands for Structured Query Language and is used to interact with relational databases.

---

## 3. What is a relational database?

A relational database stores structured data in tables made up of rows and columns and allows relationships between tables.

Examples include PostgreSQL and MySQL.

---

## 4. What is NoSQL?

NoSQL is a broad category of non-relational database systems that use different data models, such as documents, key-value pairs, wide columns, or graphs.

---

## 5. SQL vs NoSQL?

A good interview answer:

> "Relational databases like PostgreSQL organize structured data into tables and are especially useful when relationships, transactions, and complex queries are important. NoSQL databases use different models such as documents or key-value structures and can be a good fit for certain flexible or high-scale workloads. I wouldn't choose one just based on SQL vs NoSQL; I'd look at the data model, access patterns, consistency requirements, transaction needs, and operational requirements."

That's much better than:

> "SQL is for structured data and NoSQL is for unstructured data."

---

## 6. What is a primary key?

A primary key uniquely identifies a row in a table.

For example:

```text
users

id
--
1
2
3
```

Here `id` could be the primary key.

---

## 7. What is a foreign key?

A foreign key is a column that references a key in another table, allowing the database to represent a relationship between the two tables.

---

## 8. What is ACID?

ACID represents:

```text
Atomicity
Consistency
Isolation
Durability
```

These properties help databases provide reliable transaction behavior.

---

## 9. What is a transaction?

A transaction groups multiple database operations into a single logical unit so they can be committed together or rolled back when necessary.

---

## 10. What is eventual consistency?

Eventual consistency means that distributed copies of data may temporarily differ after an update but are expected to converge to the same value once updates have propagated.

---

## 11. Is NoSQL always eventually consistent?

No.

Consistency behavior depends on the particular database and its configuration.

Don't reduce the SQL/NoSQL distinction to "consistent vs eventually consistent."

---

## 12. Why can't we just store everything in a JSON file?

A JSON file can work for tiny projects, but it doesn't provide the database capabilities needed for serious applications, such as efficient querying, concurrency control, transactions, indexes, reliability, and operational tooling.

---

# 41. Interview Scenario

Here's the question I'd actually like you to practice.

> **You're building an e-commerce application. Would you choose PostgreSQL or MongoDB? Why?**

Don't answer immediately.

First ask:

```text
What data do we have?

Users
Products
Orders
Payments
Inventory
Reviews
```

Then:

```text
Which data is strongly related?

What transactions are required?

How important is consistency?

What queries will we run?

How flexible are the product attributes?

How large is the workload?

What are the scaling requirements?
```

You might decide PostgreSQL is the better default because the application has many relationships and transaction-heavy workflows.

But the important part of the answer is **your reasoning**.

An interviewer is often more interested in how you arrive at the decision than whether you picked the database they expected.

---

# 42. Mini Project — Move Your Todo API to PostgreSQL

Now we're going to make the Todo API more real.

Previously:

```text
Todo API
   ↓
JavaScript array
```

Change it to:

```text
Todo API
   ↓
PostgreSQL
```

Your database should have a `todos` table with something like:

```text
todos
----------------------------
id
title
completed
created_at
```

Your API should still expose:

```text
GET    /todos
GET    /todos/:id
POST   /todos
PATCH  /todos/:id
DELETE /todos/:id
```

But now:

```text
POST /todos
      ↓
Node.js
      ↓
INSERT
      ↓
PostgreSQL
```

and:

```text
GET /todos
      ↓
Node.js
      ↓
SELECT
      ↓
PostgreSQL
      ↓
JSON response
```

Restart your server.

Your todos should still exist.

That's the whole point.

---

# 43. Your First Database Exercise

Before using an ORM, I want you to understand what the database is actually doing.

Create a PostgreSQL database and a `todos` table.

Then manually practice:

```sql
INSERT
SELECT
UPDATE
DELETE
```

For example:

```sql
INSERT INTO todos (title, completed)
VALUES ('Learn databases', false);
```

Then:

```sql
SELECT * FROM todos;
```

Then:

```sql
UPDATE todos
SET completed = true
WHERE id = 1;
```

Then:

```sql
DELETE FROM todos
WHERE id = 1;
```

Don't worry about Prisma yet.

**First understand SQL.**

We'll introduce Prisma after we understand the database itself.

---

# 44. What I Want You To Understand From This Chapter

If you remember only a few things, remember these:

### 1.

A database gives your application persistent storage.

```text
Application
     ↓
Database
     ↓
Persistent data
```

### 2.

SQL is a language.

PostgreSQL is a relational database system.

### 3.

Relational databases organize data into tables and relationships.

```text
users
orders
products
```

### 4.

NoSQL isn't one database.

It's a broad category containing different data models.

### 5.

Don't choose a database because it's popular.

Choose based on:

```text
Data
Queries
Relationships
Transactions
Consistency
Scale
Operational needs
```

### 6.

A database is more than storage.

It can provide:

```text
Constraints
Indexes
Transactions
Concurrency control
Durability
```

These are some of the reasons we use databases instead of plain files.

---

# 45. Where We Are Now

Look at how our backend has evolved:

```text
Chapter 1
Internet
      ↓
Chapter 2
HTTP
      ↓
Chapter 3
Web Server
      ↓
Chapter 4
Routing
      ↓
Chapter 5
JSON
      ↓
Chapter 6
API Testing
      ↓
Chapter 7
Database
```

Our mental model is now:

```text
Client
   ↓
HTTP Request
   ↓
Web Server
   ↓
Router
   ↓
Business Logic
   ↓
Database
   ↓
Response
   ↓
JSON
   ↓
Client
```

That's starting to look like a real backend.

But we haven't actually learned how to **work with SQL properly** yet.

We know what tables are.

We know what rows are.

We know what primary and foreign keys are.

But how do you actually query this data?

How do you combine users with their posts?

How does an index work?

What happens inside a transaction?

Why can one SQL query be fast and another painfully slow?

That's next.

---

# Next — Chapter 8: SQL Deep Dive

We're going to stop talking about databases only at the conceptual level.

We're going to actually use SQL.

We'll learn:

```text
SELECT
INSERT
UPDATE
DELETE
WHERE
ORDER BY
GROUP BY
JOIN
Indexes
Constraints
Transactions
```

And we'll build something more interesting than a Todo table.

We'll start modelling a small **Reddit-like forum** with:

```text
users
posts
comments
votes
```

Because once you understand how to model and query related data, databases start making a lot more sense.

**Don't skip SQL just because you'll eventually use Prisma.**

An ORM can help you write database code.

It can't replace understanding the database underneath.