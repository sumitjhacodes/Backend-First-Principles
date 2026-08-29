# Chapter 9 — ORMs & Database Tools

In Chapter 8, we wrote SQL ourselves.

We did things like:

```sql
SELECT * FROM users;
```

```sql
INSERT INTO posts (title, content, user_id)
VALUES ('Learning SQL', 'This is my first post.', 1);
```

```sql
UPDATE posts
SET title = 'Learning PostgreSQL'
WHERE id = 1;
```

And that's great.

Actually, I **want** you to know SQL.

But imagine your backend has hundreds of database operations.

You might end up writing SQL everywhere:

```js
const result = await db.query(
  "SELECT * FROM users WHERE id = $1",
  [userId]
);
```

Then another query:

```js
const result = await db.query(
  "SELECT * FROM posts WHERE user_id = $1",
  [userId]
);
```

Then another:

```js
const result = await db.query(
  "INSERT INTO posts (...) VALUES (...)",
  [...]
);
```

And another.

Raw SQL is powerful, but application code can become repetitive and harder to maintain.

This is one reason developers use **ORMs**.

---

# 1. What Is an ORM?

ORM stands for:

> **Object-Relational Mapping**

That's a scary name for a fairly simple idea.

An ORM helps you work with database data using the programming language you're already writing your application in.

Without an ORM:

```text
TypeScript
    ↓
SQL
    ↓
PostgreSQL
```

With an ORM:

```text
TypeScript
    ↓
ORM
    ↓
SQL
    ↓
PostgreSQL
```

The ORM sits between your application and the database.

---

# 2. A Simple Example

Suppose we want to find a user.

With SQL:

```sql
SELECT *
FROM users
WHERE id = 1;
```

With Prisma, you can write:

```ts
const user = await prisma.user.findUnique({
  where: {
    id: 1
  }
});
```

You don't write the SQL yourself.

Prisma handles the database communication.

Conceptually:

```text
Your TypeScript
      ↓
prisma.user.findUnique()
      ↓
Prisma
      ↓
SQL
      ↓
PostgreSQL
```

The database is still doing the actual database work.

The ORM isn't replacing PostgreSQL.

---

# 3. Why Do Developers Use ORMs?

There are several reasons.

## Less repetitive code

Instead of writing SQL strings everywhere, you can work with a programming-language API.

## Type safety

With TypeScript and a good ORM, your editor can often catch mistakes before you even run the application.

## Database schema management

ORMs commonly provide tools for defining and changing your database schema.

## Migrations

You can track database structure changes as your application evolves.

## Relationships

ORMs can make common relationships easier to query.

## Developer experience

Autocomplete, generated types, validation, and familiar application code can make database work more comfortable.

But remember:

> **Convenient doesn't automatically mean better.**

There are situations where raw SQL is still the right tool.

---

# 4. ORM Is Not a Database

This is one of the first things I want you to remember.

Prisma isn't a database.

```text
PostgreSQL
→ Database system

Prisma
→ ORM / database toolkit

SQL
→ Query language
```

Think of it like this:

```text
Your application
       ↓
     Prisma
       ↓
   PostgreSQL
```

If PostgreSQL is down, Prisma can't magically store your data somewhere.

Prisma needs a database underneath it.

---

# 5. Popular ORMs and Database Tools

You'll encounter different tools depending on the programming language.

For example:

### JavaScript / TypeScript

```text
Prisma
Drizzle
Sequelize
TypeORM
```

### Python

```text
SQLAlchemy
Django ORM
```

### Java

```text
Hibernate
```

### C#

```text
Entity Framework
```

The exact tool isn't the most important thing.

The concept is.

Once you understand:

```text
database
SQL
schema
relationships
transactions
indexes
```

learning another ORM becomes much easier.

---

# 6. Why We're Using Prisma

For this repository, we'll use **Prisma** with PostgreSQL.

Why?

Mainly because it gives us a nice developer experience with TypeScript and makes it easier to understand how our application code maps to the database.

But there's an important rule:

> **Don't use Prisma to avoid learning SQL.**

We already learned the SQL fundamentals in Chapter 8.

That's intentional.

Now we can use Prisma while still understanding what's happening underneath.

---

# 7. Setting Up Prisma

Assuming you already have a Node.js + TypeScript project:

Install Prisma:

```bash
npm install prisma @prisma/client
```

Then initialize it:

```bash
npx prisma init
```

You should get something roughly like:

```text
project/
├── prisma/
│   └── schema.prisma
├── src/
├── package.json
└── .env
```

The exact project structure can vary.

The important files are:

```text
schema.prisma
.env
```

---

# 8. The Database Connection

Your `.env` file will contain a database connection string.

For example:

```env
DATABASE_URL="postgresql://username:password@localhost:5432/myapp"
```

Don't commit real database passwords or secrets to Git.

That's why connection information commonly lives in environment variables.

We'll go much deeper into configuration and secrets in Chapter 19.

For now:

```text
.env
↓
Database connection information
```

---

# 9. The Prisma Schema

Open:

```text
prisma/schema.prisma
```

This is where we define how Prisma understands our database models.

For example:

```prisma
model User {
  id       Int    @id @default(autoincrement())
  username String
  email    String @unique

  posts    Post[]
}

model Post {
  id        Int      @id @default(autoincrement())
  title     String
  content   String
  userId    Int
  createdAt DateTime @default(now())

  user      User     @relation(fields: [userId], references: [id])
}
```

At first this can look strange.

Let's break it down.

---

# 10. What Is a Model?

This:

```prisma
model User {
  ...
}
```

describes a `User` model.

And:

```prisma
model Post {
  ...
}
```

describes a `Post` model.

Conceptually:

```text
Prisma model
      ↓
Database table
```

So:

```text
User
↓
users

Post
↓
posts
```

The exact database table naming can depend on your Prisma setup, but the basic mental model is:

> A model represents a type of data your application works with.

---

# 11. Fields

Inside:

```prisma
model User {
  id       Int
  username String
  email    String
}
```

we have fields:

```text
id
username
email
```

and their types:

```text
id       → Int
username → String
email    → String
```

This looks very similar to defining TypeScript types.

But remember:

> A Prisma model is describing database data, not just a TypeScript object.

---

# 12. Primary Keys in Prisma

We wrote:

```prisma
id Int @id @default(autoincrement())
```

The important part:

```text
@id
```

means the field is the primary key.

And:

```text
@default(autoincrement())
```

means new records get automatically generated IDs.

Conceptually:

```text
User 1
User 2
User 3
```

just like we saw with PostgreSQL.

---

# 13. Unique Fields

We wrote:

```prisma
email String @unique
```

This tells Prisma/database that email values should be unique.

So:

```text
rahul@example.com
```

can't be used by two users if the corresponding database uniqueness constraint is applied.

This is an important distinction:

> Your application can check uniqueness, but the database should also enforce important uniqueness rules.

---

# 14. Relationships in Prisma

Remember our SQL relationship?

```text
users
   ↓
posts
```

A user can create many posts.

In Prisma:

```prisma
model User {
  id    Int    @id @default(autoincrement())
  posts Post[]
}
```

And:

```prisma
model Post {
  id     Int  @id @default(autoincrement())
  userId Int

  user User @relation(fields: [userId], references: [id])
}
```

Now Prisma knows that:

```text
One User
   ↓
Many Posts
```

This is called a **one-to-many relationship**.

---

# 15. Relationship Types

You'll commonly see:

### One-to-one

```text
User
 ↓
Profile
```

One user has one profile.

### One-to-many

```text
User
 ↓
Posts
```

One user can have many posts.

### Many-to-many

```text
Students
   ↕
Courses
```

A student can take many courses.

A course can have many students.

These relationships become very important once you build real applications.

---

# 16. Migration — The Important Part

Here's a problem.

You wrote this in Prisma:

```prisma
model User {
  id       Int    @id @default(autoincrement())
  username String
  email    String @unique
}
```

But PostgreSQL doesn't automatically know about it.

You need to apply the schema to the database.

That's where **migrations** come in.

A migration is basically a tracked change to your database structure.

Think of it like Git, but for your database schema.

---

# 17. Why Do We Need Migrations?

Imagine today your database looks like:

```text
users
------
id
username
```

Tomorrow you need:

```text
users
------
id
username
email
```

You changed the schema.

You need a reliable way to tell the database:

> "Add this column."

In a team, you also need other developers and environments to apply the same change.

Migrations give you a history of those changes.

Conceptually:

```text
Migration 001
Create users

        ↓

Migration 002
Add email

        ↓

Migration 003
Create posts

        ↓

Migration 004
Add created_at
```

Now the database structure can evolve along with the application.

---

# 18. Creating a Migration

In development, you can use:

```bash
npx prisma migrate dev --name init
```

Prisma will create a migration based on your schema.

You'll typically see something like:

```text
prisma/
├── schema.prisma
└── migrations/
    └── 2026..._init/
        └── migration.sql
```

Notice something interesting.

There's a SQL file.

That's a good reminder:

> Prisma isn't making SQL disappear.

It's generating/using SQL underneath.

---

# 19. Read the Migration

Open the generated migration.

You may see SQL similar to:

```sql
CREATE TABLE "User" (
    "id" SERIAL NOT NULL,
    "username" TEXT NOT NULL,
    "email" TEXT NOT NULL,

    CONSTRAINT "User_pkey" PRIMARY KEY ("id")
);
```

Now compare this with what we learned in Chapter 8.

It should look familiar.

That's exactly why learning SQL first was useful.

You can now look at Prisma and understand what it's doing.

---

# 20. Prisma Client

Prisma can generate a client you use from your application code.

For example:

```ts
import { PrismaClient } from "@prisma/client";

const prisma = new PrismaClient();
```

Now you can query the database through Prisma.

For example:

```ts
const users = await prisma.user.findMany();
```

Conceptually:

```text
prisma.user.findMany()
        ↓
Prisma Client
        ↓
SQL
        ↓
PostgreSQL
```

---

# 21. Reading Data

Get all users:

```ts
const users = await prisma.user.findMany();
```

Find one user:

```ts
const user = await prisma.user.findUnique({
  where: {
    id: 1
  }
});
```

Find a user by email:

```ts
const user = await prisma.user.findUnique({
  where: {
    email: "rahul@example.com"
  }
});
```

The syntax is easier to work with than manually constructing SQL strings.

---

# 22. Creating Data

Create a user:

```ts
const user = await prisma.user.create({
  data: {
    username: "rahul",
    email: "rahul@example.com"
  }
});
```

Conceptually, Prisma handles the equivalent database operation:

```sql
INSERT INTO users (...)
VALUES (...);
```

You don't have to write that SQL yourself.

But you should understand what's happening.

---

# 23. Updating Data

```ts
const user = await prisma.user.update({
  where: {
    id: 1
  },
  data: {
    username: "rahul-dev"
  }
});
```

Conceptually:

```sql
UPDATE users
SET username = 'rahul-dev'
WHERE id = 1;
```

Again:

```text
Prisma
↓
SQL
↓
Database
```

---

# 24. Deleting Data

```ts
await prisma.user.delete({
  where: {
    id: 1
  }
});
```

Conceptually:

```sql
DELETE FROM users
WHERE id = 1;
```

CRUD becomes very straightforward.

---

# 25. Filtering

Suppose we want all posts created by user `1`.

You can write:

```ts
const posts = await prisma.post.findMany({
  where: {
    userId: 1
  }
});
```

Conceptually, that's similar to:

```sql
SELECT *
FROM posts
WHERE user_id = 1;
```

---

# 26. Sorting

Suppose we want newest posts first:

```ts
const posts = await prisma.post.findMany({
  orderBy: {
    createdAt: "desc"
  }
});
```

Conceptually:

```sql
ORDER BY created_at DESC
```

This should look familiar.

---

# 27. Limiting Results

For example:

```ts
const posts = await prisma.post.findMany({
  take: 10
});
```

This is conceptually similar to:

```sql
LIMIT 10
```

Again, Prisma is providing a programming-language interface over database operations.

---

# 28. Selecting Only What You Need

You don't always want every column.

For example:

```ts
const users = await prisma.user.findMany({
  select: {
    id: true,
    username: true
  }
});
```

Now you're asking for only those fields.

This is useful because:

```text
Less data fetched
        ↓
Less data transferred
        ↓
Less unnecessary work
```

It can also help avoid accidentally returning sensitive fields.

---

# 29. Working With Relationships

Remember:

```text
User
 ↓
Posts
```

You can ask Prisma to include related posts.

```ts
const user = await prisma.user.findUnique({
  where: {
    id: 1
  },
  include: {
    posts: true
  }
});
```

The result can look conceptually like:

```json
{
  "id": 1,
  "username": "rahul",
  "email": "rahul@example.com",
  "posts": [
    {
      "id": 1,
      "title": "Learning SQL"
    },
    {
      "id": 2,
      "title": "My backend journey"
    }
  ]
}
```

This is convenient.

But there's an important warning.

Convenience can hide database work.

---

# 30. The N+1 Query Problem

This is one of those problems you should know early.

Imagine you have:

```text
100 users
```

and for every user you separately query their posts.

You might accidentally do:

```text
1 query
→ get 100 users

100 queries
→ get posts for each user
```

Total:

```text
101 database queries
```

That's called the **N+1 query problem**.

It can become a serious performance issue.

The ORM isn't automatically bad here.

The problem is how we're using it.

You need to understand what database queries your application is actually producing.

---

# 31. Why ORMs Can Hide Performance Problems

This is one of the biggest reasons I don't want you to blindly trust an ORM.

This code:

```ts
await prisma.user.findMany({
  include: {
    posts: true
  }
});
```

looks simple.

But underneath, the database still has to retrieve related data.

Depending on the query, schema, database, and ORM behavior, that can involve more complex SQL than the code suggests.

So when something becomes slow:

```text
Don't just stare at Prisma code.
        ↓
Understand the generated SQL.
        ↓
Look at the query plan.
        ↓
Check indexes.
        ↓
Measure the actual problem.
```

We'll get much deeper into database performance later.

---

# 32. Raw SQL Still Has a Place

Sometimes you have a complicated query.

Maybe you need a PostgreSQL feature or query that's awkward to express through your ORM.

You can use raw SQL when appropriate.

With Prisma, for example, there are raw-query APIs.

Conceptually:

```ts
const users = await prisma.$queryRaw`
  SELECT *
  FROM users
  WHERE email = ${email}
`;
```

The important part isn't memorizing the Prisma API.

It's understanding:

> **An ORM is a tool. You're still allowed to use SQL when SQL is the better tool.**

---

# 33. ORM vs Raw SQL

Let's compare them.

|                        | ORM                                  | Raw SQL                  |
| ---------------------- | ------------------------------------ | ------------------------ |
| Developer experience   | Usually easier                       | More manual              |
| Type safety            | Often strong with TypeScript tooling | Depends on your setup    |
| Complex queries        | Can become awkward                   | Very powerful            |
| SQL knowledge required | Less to get started                  | More                     |
| Control                | ORM abstracts details                | Maximum control          |
| Portability            | Can help in some cases               | SQL is database-specific |
| Debugging              | Need to understand generated queries | Query is explicit        |

Neither one is universally better.

A good backend engineer knows when to use each.

---

# 34. A Common Beginner Mistake

Someone learns Prisma and says:

> "I don't need SQL anymore."

That's exactly backwards.

You should be thinking:

```text
I know SQL
       +
I know how my ORM works
       ↓
I can choose the right tool
```

not:

```text
I know Prisma
       ↓
I don't care what the database does
```

When an ORM works well, great.

When a query becomes slow or complicated, your SQL knowledge becomes extremely valuable.

---

# 35. Database Schema vs Application Types

Another thing that confuses beginners:

```ts
type User = {
  id: number;
  username: string;
  email: string;
};
```

and:

```prisma
model User {
  id       Int    @id
  username String
  email    String
}
```

They may look similar.

But they serve different purposes.

The TypeScript type describes what your application expects.

The Prisma model describes database structure and relationships.

They're connected, but they aren't the same thing.

---

# 36. What Happens When the Schema Changes?

Suppose we start with:

```prisma
model User {
  id       Int    @id @default(autoincrement())
  username String
}
```

Later we decide users need:

```text
email
```

We change the schema:

```prisma
model User {
  id       Int    @id @default(autoincrement())
  username String
  email    String
}
```

Then create a migration:

```bash
npx prisma migrate dev --name add-user-email
```

Now we have:

```text
Old database
      ↓
Migration
      ↓
New database
```

This is much safer than manually changing every developer's database and hoping everyone remembers what they changed.

---

# 37. Migrations Are Version Control for Database Structure

This is the mental model I want you to remember.

Git tracks:

```text
Application code changes
```

Migrations track:

```text
Database structure changes
```

So your project might evolve like:

```text
Commit 1
Create users

Commit 2
Add posts

Commit 3
Add comments

Commit 4
Add email to users
```

and your migrations might evolve alongside them.

This becomes extremely useful when working with a team.

---

# 38. Development vs Production Migrations

Be careful here.

During development, you might use:

```bash
npx prisma migrate dev
```

For production, you typically want to apply already-created migrations rather than generating new schema changes interactively.

For example:

```bash
npx prisma migrate deploy
```

The exact deployment process depends on your project and environment.

The important principle:

> **Don't casually experiment with your production database.**

Production databases contain real data.

Database changes need to be planned and tested.

---

# 39. Transactions With Prisma

Remember the bank transfer example from Chapter 8.

You can also perform transactions through Prisma.

Conceptually:

```ts
await prisma.$transaction(async (tx) => {
  await tx.account.update({
    where: { id: 1 },
    data: {
      balance: {
        decrement: 1000
      }
    }
  });

  await tx.account.update({
    where: { id: 2 },
    data: {
      balance: {
        increment: 1000
      }
    }
  });
});
```

The important idea isn't the exact syntax.

It's:

```text
Operation A
+
Operation B
       ↓
Transaction
       ↓
Both succeed
OR
changes are rolled back
```

The ORM is giving us a convenient interface.

The database still provides the transactional guarantees.

---

# 40. Connection Management

Here's something that beginners often don't think about.

Your backend needs a connection to PostgreSQL.

But you don't want every request to create a completely new database connection.

Imagine:

```text
Request 1 → new connection
Request 2 → new connection
Request 3 → new connection
...
Request 10,000 → new connection
```

That can become a serious problem.

Databases have limits.

Instead, applications commonly use **connection pooling**.

Conceptually:

```text
             ┌── connection
             ├── connection
Node.js ─────┼── connection
             ├── connection
             └── connection
                    ↓
                PostgreSQL
```

Connections can be reused.

We'll go much deeper into connection pooling when we discuss production database scaling.

For now:

> Don't create a brand-new database connection for every request.

---

# 41. Don't Create PrismaClient Everywhere

A related practical issue.

Avoid doing this inside every request handler:

```ts
app.get("/users", async (req, res) => {
  const prisma = new PrismaClient();

  // ...
});
```

You don't want to repeatedly create database clients/connections unnecessarily.

A common pattern is to create a shared Prisma client for your application.

For example:

```ts
import { PrismaClient } from "@prisma/client";

export const prisma = new PrismaClient();
```

Then your services/routes can use that shared client.

The exact setup can become more nuanced depending on development environments, serverless deployments, and application architecture.

But the core idea is:

> **Database clients/connections need lifecycle management.**

---

# 42. Where Should Database Code Live?

You could technically write this inside a route:

```ts
app.get("/users/:id", async (req, res) => {
  const user = await prisma.user.findUnique({
    where: {
      id: Number(req.params.id)
    }
  });

  res.json(user);
});
```

It works.

But as the application grows, you don't want every route becoming a giant mix of:

```text
HTTP handling
validation
business logic
database queries
error handling
```

A common approach is to separate responsibilities.

For example:

```text
src/
├── routes/
├── controllers/
├── services/
├── repositories/
└── lib/
```

You don't need this architecture for a tiny Todo API.

But as applications grow, separation becomes useful.

---

# 43. ORM Doesn't Mean You Need a Huge Architecture

Another beginner trap:

> "I'm using Prisma, so I need repositories, services, factories, interfaces, dependency injection, and 25 folders."

No.

Start simple.

For a small application:

```text
route
  ↓
service
  ↓
Prisma
  ↓
PostgreSQL
```

can be perfectly fine.

Architecture should solve problems.

Don't create abstraction just because you've seen it in a large company codebase.

---

# 44. Common ORM Mistakes

## Mistake 1 — Thinking ORM replaces SQL

It doesn't.

Learn SQL.

---

## Mistake 2 — Fetching everything

Don't retrieve every column and every relationship just because your ORM makes it easy.

Ask for what you actually need.

---

## Mistake 3 — Ignoring generated queries

If something is slow, investigate what SQL is actually being executed.

---

## Mistake 4 — Creating too many database connections

Connection management matters.

---

## Mistake 5 — Treating migrations casually

Database schema changes affect real data.

Treat migrations as code.

Review them.

Test them.

---

## Mistake 6 — Putting all business logic inside ORM calls

This:

```text
prisma...
prisma...
prisma...
prisma...
```

shouldn't become your entire application's architecture.

Your backend still needs business logic.

---

## Mistake 7 — Using an ORM for everything

Sometimes raw SQL is clearer or more appropriate.

Use the tool that fits the problem.

---

# 45. Mini Project — Move the Forum API to Prisma

We already designed our forum database in Chapter 8.

Now let's connect it to our backend.

We have:

```text
users
posts
comments
votes
```

And relationships:

```text
User
 ├── Posts
 │     └── Comments
 │
 └── Votes
```

Create corresponding Prisma models.

For example:

```prisma
model User {
  id        Int       @id @default(autoincrement())
  username  String
  email     String    @unique
  createdAt DateTime  @default(now())

  posts     Post[]
  comments  Comment[]
}

model Post {
  id        Int       @id @default(autoincrement())
  title     String
  content   String
  userId    Int
  createdAt DateTime  @default(now())

  user      User      @relation(fields: [userId], references: [id])
  comments  Comment[]
}
```

Then add comments and votes.

Don't worry if your schema differs slightly.

The important part is understanding the relationships.

---

# 46. Build These API Endpoints

Your forum API should eventually support something like:

```text
POST   /users
GET    /users/:id

POST   /posts
GET    /posts
GET    /posts/:id
PATCH  /posts/:id
DELETE /posts/:id

POST   /posts/:id/comments
GET    /posts/:id/comments
```

Then connect those routes to Prisma.

For example:

```text
POST /posts
     ↓
Validate request
     ↓
Service/business logic
     ↓
Prisma
     ↓
PostgreSQL
     ↓
Result
     ↓
JSON response
```

Now you're building a backend with a real database layer.

---

# 47. Your Exercise

Don't copy everything.

Try building these yourself.

### Exercise 1

Create a user.

### Exercise 2

Create a post belonging to that user.

### Exercise 3

Fetch a post and its author.

### Exercise 4

Fetch a user and their posts.

### Exercise 5

Update a post.

### Exercise 6

Delete a post.

### Exercise 7

Try to create two users with the same email.

What happens?

Why?

---

### Exercise 8

Write the Prisma query.

Then write the equivalent SQL.

For example:

```ts
await prisma.user.findUnique({
  where: {
    id: 1
  }
});
```

Then ask yourself:

> What SQL would this roughly represent?

You don't need to reproduce Prisma's exact generated SQL.

The goal is to connect the two worlds.

---

# 48. The Skill I Want You To Build

Here's the progression we're aiming for.

At first:

```text
"How do I get a user?"
```

Then:

```text
"I can use Prisma."
```

Better:

```text
"I know Prisma is querying PostgreSQL."
```

Even better:

```text
"I understand the SQL this operation represents."
```

And eventually:

```text
"I can tell whether this query is appropriate,
what indexes it may need,
whether it could become expensive,
and how I would investigate it."
```

That's the backend engineer mindset.

---

# 49. Interview Questions

## 1. What is an ORM?

An ORM, or Object-Relational Mapping tool, provides an abstraction for interacting with relational databases using application-language objects and APIs instead of writing every SQL query manually.

---

## 2. Is Prisma a database?

No.

Prisma is an ORM/database toolkit. PostgreSQL, MySQL, and similar systems are databases/database management systems.

---

## 3. Why use an ORM?

ORMS can reduce repetitive database code, provide better developer experience and type safety, simplify relationships, and provide schema/migration tooling.

---

## 4. Does an ORM replace SQL?

No.

An ORM abstracts database operations, but understanding SQL remains important for debugging, performance, complex queries, and making good database decisions.

---

## 5. What is a migration?

A migration is a versioned change to a database schema.

It allows teams and environments to apply database structure changes consistently.

---

## 6. Why are migrations useful?

They provide a history of schema changes and make it easier to reproduce the same database structure across development, testing, and production environments.

---

## 7. What is the N+1 query problem?

It's a performance problem where an application executes one query to fetch a set of records and then performs an additional query for each individual record.

For example:

```text
1 query → fetch 100 users
100 queries → fetch posts for each user

Total = 101 queries
```

---

## 8. ORM vs raw SQL?

A good answer:

> "I use an ORM for common database operations because it improves developer productivity and type safety. But I still understand SQL because I may need complex queries, performance tuning, debugging, or database-specific features where raw SQL is a better fit."

---

## 9. What is connection pooling?

Connection pooling maintains a set of reusable database connections so the application doesn't need to establish a new database connection for every request.

---

## 10. Why shouldn't we create a database connection for every request?

Creating connections repeatedly adds overhead and can exhaust the database's connection limit under load. Reusing connections through a pool is more efficient.

---

# 50. Interview Scenario

Here's a practical one.

> Your API was fast when it had 100 users. Now it has 100,000 users and one endpoint became very slow. You're using Prisma. What do you do?

Don't say:

> "I'll replace Prisma."

That's not debugging.

Think:

```text
Measure the endpoint
       ↓
Find the slow operation
       ↓
Inspect database query
       ↓
Look at generated SQL
       ↓
Check query plan
       ↓
Check indexes
       ↓
Check how much data is fetched
       ↓
Look for N+1 queries
       ↓
Check connection pool
       ↓
Optimize based on evidence
```

The ORM isn't automatically the problem.

The query, schema, indexes, data volume, connection management, or something completely outside the database could be responsible.

---

# 51. ORM Mental Model

Whenever you see:

```ts
prisma.user.findMany(...)
```

don't think:

```text
Magic
```

Think:

```text
TypeScript
    ↓
Prisma
    ↓
Database query
    ↓
PostgreSQL
    ↓
Rows
    ↓
Prisma
    ↓
JavaScript/TypeScript objects
```

That's all we're doing.

The ORM is an abstraction layer.

---

# 52. What You Should Be Able To Do Now

After this chapter, you should understand:

```text
What an ORM is
        ↓
Why ORMs exist
        ↓
What Prisma does
        ↓
What a Prisma model is
        ↓
What migrations are
        ↓
How CRUD works through Prisma
        ↓
How relationships work
        ↓
Why N+1 queries are dangerous
        ↓
Why SQL still matters
        ↓
Why connection management matters
```

And more importantly:

You should be able to look at:

```ts
const posts = await prisma.post.findMany({
  where: {
    userId: 1
  },
  orderBy: {
    createdAt: "desc"
  }
});
```

and understand that this isn't some special Prisma magic.

You're essentially asking the database:

> "Give me posts belonging to this user, ordered by newest first."

---

# 53. Where We Are Now

Our backend has evolved again.

```text
Client
   ↓
HTTP
   ↓
Web Server
   ↓
Router
   ↓
Business Logic
   ↓
Prisma
   ↓
PostgreSQL
   ↓
Data
```

And now we understand both sides:

```text
Application
     ↓
Prisma
     ↓
SQL
     ↓
PostgreSQL
```

That's a very useful mental model.

But there's one major part of backend development we haven't covered yet.

How do we make sure users are actually **who they claim to be?**

How do we store passwords?

How do we protect private routes?

How does "Login with Google" work?

What exactly is a JWT?

And what's the difference between:

> **Authentication**

and

> **Authorization?**

That's where we're going next.

---

# Next — Chapter 10: Authentication & Authorization

We'll build a proper mental model for:

```text
Authentication
Authorization
Passwords
Hashing
Sessions
JWT
Cookies
Access tokens
Refresh tokens
OAuth
Protected routes
Roles & permissions
```

And we'll build authentication into our backend instead of just copying a JWT tutorial.

The goal isn't to memorize:

> "JWT has three parts."

The goal is to understand:

> **Who are you?**

> **How did you prove it?**

> **What are you allowed to do?**

That's the real problem authentication systems are solving.