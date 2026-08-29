# Chapter 8 — SQL Deep Dive

In the last chapter, we finally gave our Todo API a proper place to store data.

Instead of:

```js
const todos = [];
```

we started thinking about:

```text
PostgreSQL
    ↓
todos table
    ↓
rows
```

But knowing that PostgreSQL exists isn't enough.

At some point, your backend has to ask the database questions.

For example:

> Give me all todos.

> Give me todo number 42.

> Give me all incomplete todos.

> Create a new todo.

> Mark this todo as completed.

> Delete this todo.

How do we ask the database to do these things?

We use **SQL**.

And honestly, this is one of the skills I don't recommend skipping.

Even if you're going to use Prisma or another ORM later, you should understand the SQL happening underneath.

---

# 1. What Is SQL?

SQL stands for:

> **Structured Query Language**

It's a language used to communicate with relational databases.

For example:

```sql
SELECT * FROM todos;
```

This basically says:

> "Give me everything from the `todos` table."

Or:

```sql
SELECT * FROM todos
WHERE completed = false;
```

This says:

> "Give me todos that aren't completed."

That's SQL.

Your backend sends SQL to the database, the database executes it, and then returns a result.

```text
Node.js
   ↓
SQL query
   ↓
PostgreSQL
   ↓
Result
   ↓
Node.js
   ↓
HTTP response
```

---

# 2. SQL Isn't Just SELECT

When people start learning SQL, they often think:

```text
SQL = SELECT
```

Not really.

You'll use SQL to:

```text
Create data
Read data
Update data
Delete data
```

These are commonly called **CRUD** operations.

```text
C → Create
R → Read
U → Update
D → Delete
```

The basic SQL commands are:

```sql
INSERT
SELECT
UPDATE
DELETE
```

We'll spend most of this chapter learning these properly.

---

# 3. Let's Create Our Example Database

We'll use a small forum application throughout this chapter.

Imagine something like a very simple Reddit.

We have:

```text
users
posts
comments
votes
```

The relationships look roughly like:

```text
User
 ├── Posts
 │     └── Comments
 │
 └── Votes
```

We'll start with the simplest table.

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username TEXT NOT NULL,
    email TEXT NOT NULL
);
```

Don't worry if some of this syntax is unfamiliar.

We'll break it down.

---

# 4. Understanding CREATE TABLE

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username TEXT NOT NULL,
    email TEXT NOT NULL
);
```

### `CREATE TABLE`

Create a new table.

### `users`

The table's name.

### `id`

A column called `id`.

### `SERIAL`

PostgreSQL can automatically generate increasing integer values for this older/common style of schema.

### `PRIMARY KEY`

This column uniquely identifies each row.

### `TEXT`

The column stores text.

### `NOT NULL`

This value must be provided.

So conceptually:

```text
users
--------------------------------
id
username
email
```

And our database has rules about those columns.

---

# 5. Insert Data

Now let's create a user.

```sql
INSERT INTO users (username, email)
VALUES ('rahul', 'rahul@example.com');
```

The database might create:

```text
id | username | email
---|----------|-------------------
1  | rahul    | rahul@example.com
```

Let's add another:

```sql
INSERT INTO users (username, email)
VALUES ('aman', 'aman@example.com');
```

Now:

```text
id | username | email
---|----------|-------------------
1  | rahul    | rahul@example.com
2  | aman     | aman@example.com
```

We've just performed a **Create** operation.

---

# 6. SELECT — Reading Data

Now let's get our users.

```sql
SELECT * FROM users;
```

The `*` means:

> All columns.

Result:

```text
id | username | email
---|----------|-------------------
1  | rahul    | rahul@example.com
2  | aman     | aman@example.com
```

But you don't always need every column.

You can ask for specific columns:

```sql
SELECT username, email
FROM users;
```

Result:

```text
username | email
---------|-------------------
rahul    | rahul@example.com
aman     | aman@example.com
```

This is generally better than blindly selecting everything when you only need a few columns.

---

# 7. WHERE — Filtering Data

Suppose we only want Rahul.

```sql
SELECT *
FROM users
WHERE username = 'rahul';
```

The database checks the rows and returns the matching one.

You can also filter by ID:

```sql
SELECT *
FROM users
WHERE id = 1;
```

This is something your backend will do constantly.

For:

```text
GET /users/1
```

your backend might eventually execute a query conceptually like:

```sql
SELECT *
FROM users
WHERE id = 1;
```

---

# 8. Comparison Operators

You can use different operators in conditions.

```sql
=
<>
>
<
>=
<=
```

For example:

```sql
SELECT *
FROM users
WHERE id > 1;
```

Or:

```sql
SELECT *
FROM posts
WHERE score >= 100;
```

You can combine conditions too.

---

# 9. AND and OR

Suppose we want users whose ID is greater than 1 **and** whose username is `"aman"`.

```sql
SELECT *
FROM users
WHERE id > 1
AND username = 'aman';
```

`AND` means both conditions must be true.

`OR` means either condition can be true.

For example:

```sql
SELECT *
FROM users
WHERE username = 'rahul'
OR username = 'aman';
```

---

# 10. NULL Is Different

This is a small detail that causes a lot of confusion.

Suppose a column contains:

```text
NULL
```

`NULL` doesn't mean:

```text
0
```

and it doesn't mean:

```text
""
```

It basically represents the absence/unknown nature of a value.

You don't write:

```sql
WHERE email = NULL
```

Instead:

```sql
WHERE email IS NULL;
```

And:

```sql
WHERE email IS NOT NULL;
```

You'll encounter this constantly when working with real databases.

---

# 11. ORDER BY

Suppose we have posts:

```text
id | title              | score
---|--------------------|------
1  | Hello              | 10
2  | PostgreSQL tips    | 50
3  | My first backend   | 20
```

We can sort them:

```sql
SELECT *
FROM posts
ORDER BY score DESC;
```

Result:

```text
PostgreSQL tips    50
My first backend   20
Hello              10
```

`DESC` means descending.

For ascending:

```sql
ORDER BY score ASC;
```

---

# 12. LIMIT

What if we only want five posts?

```sql
SELECT *
FROM posts
LIMIT 5;
```

This is useful when building APIs.

For example:

```text
GET /posts?limit=10
```

might eventually result in:

```sql
SELECT *
FROM posts
LIMIT 10;
```

But be careful.

Real pagination requires more thought than simply adding `LIMIT`.

We'll discuss proper pagination later.

---

# 13. OFFSET

You can also skip rows:

```sql
SELECT *
FROM posts
LIMIT 10
OFFSET 20;
```

This roughly means:

> Skip 20 rows, then give me 10.

It's useful for understanding traditional page-based pagination.

But `OFFSET` can become inefficient for very large datasets.

We'll come back to this when we discuss APIs and pagination.

---

# 14. UPDATE

Now suppose Rahul changes their username.

```sql
UPDATE users
SET username = 'rahul-dev'
WHERE id = 1;
```

The row becomes:

```text
id | username  | email
---|-----------|-------------------
1  | rahul-dev | rahul@example.com
```

Notice the `WHERE`.

This is extremely important.

Imagine accidentally writing:

```sql
UPDATE users
SET username = 'rahul-dev';
```

You didn't specify a `WHERE`.

You may update **every user**.

That's a very bad day.

Always understand which rows your query can affect.

---

# 15. DELETE

To delete a user:

```sql
DELETE FROM users
WHERE id = 1;
```

Again:

```text
WHERE
```

matters.

This:

```sql
DELETE FROM users;
```

means:

> Delete every row in the table.

Never casually run destructive queries against production.

---

# 16. CRUD in One Table

At this point:

```text
Create
↓
INSERT

Read
↓
SELECT

Update
↓
UPDATE

Delete
↓
DELETE
```

These four commands are the foundation of database work.

Your backend API is basically going to turn HTTP operations into these kinds of database operations.

For example:

```text
POST /users
      ↓
INSERT

GET /users
      ↓
SELECT

PATCH /users/1
      ↓
UPDATE

DELETE /users/1
      ↓
DELETE
```

This is the connection between the API layer and database layer.

---

# 17. Let's Create Posts

Now let's make our forum more realistic.

```sql
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    title TEXT NOT NULL,
    content TEXT NOT NULL,
    user_id INTEGER NOT NULL REFERENCES users(id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

Now we have:

```text
users
        ↓
      user_id
        ↓
posts
```

The important part is:

```sql
REFERENCES users(id)
```

That creates a foreign-key relationship.

A post belongs to a user.

---

# 18. Insert a Post

Assuming user `1` exists:

```sql
INSERT INTO posts (title, content, user_id)
VALUES (
    'Learning SQL',
    'Today I started learning PostgreSQL.',
    1
);
```

Now:

```text
posts

id | title        | content                  | user_id
---|--------------|--------------------------|--------
1  | Learning SQL | Today I started...       | 1
```

The `user_id` tells us who created the post.

---

# 19. Why Relationships Matter

Now imagine we want to answer:

> "Who wrote this post?"

The `posts` table has:

```text
user_id = 1
```

The `users` table has:

```text
id = 1
username = rahul
```

So:

```text
posts.user_id
      ↓
users.id
```

The database can connect these records.

This is where SQL becomes much more powerful.

---

# 20. JOINs

A **JOIN** lets you combine related rows from multiple tables.

For example:

```sql
SELECT
    posts.title,
    users.username
FROM posts
JOIN users
    ON posts.user_id = users.id;
```

You might get:

```text
title           | username
----------------|---------
Learning SQL    | rahul
My first post   | aman
```

Read this query slowly.

We're saying:

> Give me posts and users where the post's `user_id` matches the user's `id`.

That's a JOIN.

---

# 21. Why JOINs Are Important

Real applications have related data.

Imagine an e-commerce application:

```text
users
orders
order_items
products
```

You might want:

> Show me all orders placed by Rahul, including the products in each order.

The data isn't stored in one giant table.

It's split into related tables.

JOINs let you bring related data together when querying it.

---

# 22. INNER JOIN

The JOIN we just used is an `INNER JOIN`.

You can write it explicitly:

```sql
SELECT
    posts.title,
    users.username
FROM posts
INNER JOIN users
    ON posts.user_id = users.id;
```

An `INNER JOIN` returns rows where a matching record exists on both sides.

Conceptually:

```text
Posts       Users

  A    ←→     A
  B    ←→     B
  C           X
```

Only matching records are returned.

---

# 23. LEFT JOIN

Now imagine we want **all users**, even users who haven't created a post.

Use:

```sql
SELECT
    users.username,
    posts.title
FROM users
LEFT JOIN posts
    ON users.id = posts.user_id;
```

Now a user without posts can still appear.

You might get:

```text
username | title
---------|----------------
rahul    | Learning SQL
aman     | My first post
neha     | NULL
```

`neha` doesn't have a post, but she still appears because the `users` table is on the left side.

That's the basic idea behind `LEFT JOIN`.

---

# 24. INNER JOIN vs LEFT JOIN

This is a common interview question.

### INNER JOIN

> Give me rows where both sides have a match.

### LEFT JOIN

> Give me every row from the left table, and matching rows from the right table if they exist.

A simple mental model:

```text
INNER JOIN
→ only matches

LEFT JOIN
→ everything from left + matches from right
```

Don't memorize diagrams.

Think about what result you actually want.

---

# 25. Aliases

Queries can become annoying to read when table names are long.

You can give tables short names.

```sql
SELECT
    p.title,
    u.username
FROM posts AS p
JOIN users AS u
    ON p.user_id = u.id;
```

Now:

```text
p → posts
u → users
```

This is very common in real SQL.

---

# 26. COUNT

SQL can also calculate things.

For example:

> How many users do we have?

```sql
SELECT COUNT(*)
FROM users;
```

Or:

> How many posts does each user have?

```sql
SELECT
    user_id,
    COUNT(*)
FROM posts
GROUP BY user_id;
```

You might get:

```text
user_id | count
--------|------
1       | 5
2       | 3
3       | 0
```

Although note that users with zero posts won't appear in that particular query because we're grouping the `posts` table.

We'll later combine aggregation with joins.

---

# 27. GROUP BY

`GROUP BY` groups rows so aggregate functions can operate on each group.

For example:

```sql
SELECT
    user_id,
    COUNT(*)
FROM posts
GROUP BY user_id;
```

means:

> Group posts by user and count the posts in each group.

Other common aggregate functions include:

```text
COUNT()
SUM()
AVG()
MIN()
MAX()
```

For example:

```sql
SELECT AVG(score)
FROM posts;
```

---

# 28. HAVING

`WHERE` filters rows **before** grouping.

`HAVING` filters groups **after** grouping.

For example:

> Find users who have more than 10 posts.

```sql
SELECT
    user_id,
    COUNT(*) AS post_count
FROM posts
GROUP BY user_id
HAVING COUNT(*) > 10;
```

This distinction comes up often in SQL interviews.

A simple way to remember:

```text
WHERE
→ filter rows

HAVING
→ filter groups
```

---

# 29. Constraints

One of the reasons relational databases are powerful is that they can enforce rules.

We've already seen:

```text
PRIMARY KEY
NOT NULL
FOREIGN KEY
```

There are others.

### UNIQUE

For example:

```sql
email TEXT UNIQUE
```

This prevents duplicate email values.

### CHECK

You can enforce conditions:

```sql
score INTEGER CHECK (score >= 0)
```

Now negative scores are rejected by the database.

These rules are called **constraints**.

---

# 30. Why Constraints Matter

Suppose your application checks:

```text
email must be unique
```

but two requests arrive almost simultaneously.

```text
Request A → check email
Request B → check email
```

Both might see:

> "Email doesn't exist."

Then both try to create the user.

If uniqueness is only checked in application code, you can still get a race condition.

A database-level `UNIQUE` constraint can enforce the rule where the data lives.

That's a powerful idea:

> **Important data integrity rules should not exist only in application code.**

---

# 31. Indexes

Now let's talk about something every backend developer should understand.

Imagine:

```text
users
```

contains:

```text
10 rows
```

Finding:

```text
id = 5
```

is easy.

But imagine:

```text
10 million rows
```

and you're constantly doing:

```sql
SELECT *
FROM users
WHERE email = 'rahul@example.com';
```

The database needs an efficient way to find that record.

That's where an **index** comes in.

---

# 32. What Is an Index?

An index is a data structure that helps the database find rows efficiently for certain queries.

Think of a book.

Without an index:

```text
Search every page
```

With an index:

```text
Look up the term
     ↓
Find the page
```

A database index serves a similar purpose.

For example:

```sql
CREATE INDEX idx_users_email
ON users(email);
```

Now the database has an index on `email`.

---

# 33. Why Not Index Everything?

Because indexes aren't free.

They:

* consume storage
* need to be maintained
* can make writes more expensive
* can increase memory/storage pressure

Imagine:

```text
INSERT user
```

If you have many indexes, the database may need to update those indexes too.

So:

```text
More indexes
≠
Always better
```

You generally index columns based on actual query patterns and workload.

---

# 34. What Should You Index?

A common beginner mistake is:

> "I'll put an index on every column."

Don't.

Think about how the application queries the data.

If you frequently run:

```sql
SELECT *
FROM users
WHERE email = $1;
```

an index on `email` may be useful.

If you frequently run:

```sql
SELECT *
FROM posts
WHERE user_id = $1
ORDER BY created_at DESC;
```

you might eventually consider an index designed around that access pattern.

The exact index depends on the workload.

We'll go deeper into query performance later.

---

# 35. Composite Indexes

An index can involve multiple columns.

For example:

```sql
CREATE INDEX idx_posts_user_created
ON posts(user_id, created_at);
```

This can help queries that filter/order based on those columns in compatible ways.

Don't worry about memorizing index internals yet.

Just understand:

> Sometimes the best index matches the way your application commonly queries multiple columns together.

---

# 36. Transactions

Now we get to one of the most important database concepts.

Imagine transferring money:

```text
Account A → -₹1000
Account B → +₹1000
```

We need both operations to succeed.

We don't want:

```text
A → -₹1000
B → unchanged
```

A transaction lets us treat multiple database operations as one logical unit.

Conceptually:

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 1000
WHERE id = 1;

UPDATE accounts
SET balance = balance + 1000
WHERE id = 2;

COMMIT;
```

If something goes wrong:

```sql
ROLLBACK;
```

The transaction can undo the changes made within it.

---

# 37. Why Transactions Matter

Imagine an e-commerce checkout:

```text
1. Create order
2. Reduce inventory
3. Create payment record
```

If step 1 succeeds:

```text
Order created
```

but step 2 fails:

```text
Inventory wasn't reduced
```

you could end up with inconsistent application state.

Transactions let you define operations that need to succeed together.

Not every operation needs a transaction.

But when multiple changes represent one logical operation, transactions become extremely important.

---

# 38. SQL Injection

There's one security concept you absolutely need to know.

Never build SQL by directly concatenating untrusted user input.

Bad:

```js
const query =
  "SELECT * FROM users WHERE email = '" +
  email +
  "'";
```

Suppose someone sends malicious input.

You can accidentally turn user input into SQL code.

That's **SQL injection**.

---

# 39. Parameterized Queries

Instead, use parameters.

For example, with PostgreSQL's Node.js client:

```js
const result = await pool.query(
  "SELECT * FROM users WHERE email = $1",
  [email]
);
```

Now the user's email is treated as data rather than being directly inserted into the SQL string.

This is the basic idea behind parameterized queries.

ORMs also help with this, but you should still understand the underlying problem.

---

# 40. Don't Trust ORMs Blindly

Later we'll use Prisma.

It can make database operations much easier.

But if you don't understand SQL, you can end up writing code like:

```text
prisma.user.findMany(...)
```

without understanding:

```text
What query is this generating?

Is it fetching too much data?

Is it using an index?

Is it creating a huge JOIN?

Why is this request slow?
```

You don't need to write raw SQL for everything.

But you should be able to look underneath the ORM when necessary.

---

# 41. A Real API Example

Let's connect everything together.

Suppose the client sends:

```http
GET /posts/42
```

Your backend receives it.

The route might eventually execute:

```sql
SELECT
    p.id,
    p.title,
    p.content,
    u.username
FROM posts AS p
JOIN users AS u
    ON p.user_id = u.id
WHERE p.id = $1;
```

with:

```text
$1 = 42
```

PostgreSQL returns the row.

Node.js converts the result into JSON.

The client receives:

```json
{
  "id": 42,
  "title": "Learning SQL",
  "content": "I'm learning PostgreSQL.",
  "username": "rahul"
}
```

Look at the whole journey:

```text
Browser
   ↓
GET /posts/42
   ↓
Express route
   ↓
SQL query
   ↓
PostgreSQL
   ↓
JOIN users + posts
   ↓
Result
   ↓
Node.js
   ↓
JSON
   ↓
Browser
```

This is backend development coming together.

---

# 42. SQL vs Application Logic

Here's another important distinction.

Suppose you have:

```text
POST /orders
```

The backend might decide:

```text
Is the user authenticated?
Is the product available?
Is the order allowed?
```

That's application/business logic.

Then the database handles things like:

```text
Store order
Maintain constraints
Execute transaction
Return data
```

There's overlap, but don't confuse the responsibilities.

A good backend engineer knows what belongs where.

---

# 43. Common SQL Mistakes

## Mistake 1 — Forgetting WHERE

This:

```sql
UPDATE users
SET username = 'something';
```

updates every row.

This:

```sql
DELETE FROM users;
```

deletes every row.

Always understand the scope of your query.

---

## Mistake 2 — SELECT *

You might write:

```sql
SELECT *
FROM users;
```

while developing.

That's fine for exploration.

But don't blindly return every column from every API endpoint.

You may fetch unnecessary data or accidentally expose fields you didn't intend to return.

---

## Mistake 3 — No indexes on important queries

If your application constantly searches by email or foreign key and the table becomes large, poor indexing can become a performance problem.

---

## Mistake 4 — Too many indexes

Indexes improve some reads but add maintenance cost to writes and consume resources.

---

## Mistake 5 — Doing everything in one giant query

SQL is powerful.

That doesn't mean every piece of business logic should become a 200-line SQL query.

Keep your queries understandable.

---

## Mistake 6 — Ignoring transactions

If multiple database changes must succeed together, think about whether they belong in a transaction.

---

## Mistake 7 — Building SQL with strings

Never trust user input.

Use parameterized queries or safe database libraries/ORMs.

---

# 44. A SQL Mental Model

When you see a SQL query, don't panic.

Read it in pieces.

For:

```sql
SELECT
    p.title,
    u.username
FROM posts AS p
JOIN users AS u
    ON p.user_id = u.id
WHERE p.id = $1
ORDER BY p.created_at DESC
LIMIT 10;
```

Read it roughly as:

```text
SELECT
→ What do I want back?

FROM
→ Where is the data?

JOIN
→ What other data do I need?

ON
→ How are those tables related?

WHERE
→ Which rows do I want?

ORDER BY
→ How should they be sorted?

LIMIT
→ How many do I want?
```

That mental model will take you surprisingly far.

---

# 45. Mini Project — Build a Forum Database

Let's use the database concepts we've learned.

Create:

```text
users
posts
comments
votes
```

The relationships:

```text
User
 │
 ├──< Posts
 │       │
 │       └──< Comments
 │
 └──< Votes
```

The exact schema is up to you.

But aim for something like:

### users

```text
id
username
email
created_at
```

### posts

```text
id
title
content
user_id
created_at
```

### comments

```text
id
content
user_id
post_id
created_at
```

### votes

```text
id
user_id
post_id
value
```

Then practice:

### Create

```sql
INSERT
```

Create users, posts, comments and votes.

### Read

```sql
SELECT
```

Find posts.

### Filter

```sql
WHERE
```

Find posts belonging to a particular user.

### Sort

```sql
ORDER BY
```

Find newest posts.

### Join

Get:

```text
post title
+
author username
```

### Aggregate

Find:

```text
number of comments per post
```

### Update

Change a post title.

### Delete

Delete a comment.

### Transaction

Try to design a transaction for an operation that changes multiple related records.

---

# 46. Your SQL Challenge

Don't just copy the examples above.

Try answering these yourself.

### Challenge 1

Find all posts written by user `1`.

---

### Challenge 2

Find all posts with their author's username.

---

### Challenge 3

Find the 10 newest posts.

---

### Challenge 4

Find users who have created more than 5 posts.

---

### Challenge 5

Find the number of comments on each post.

---

### Challenge 6

Find posts that have no comments.

---

### Challenge 7

Find the top 5 posts by vote score.

---

### Challenge 8

Update a post only if the current user owns it.

Think carefully about where that ownership check should happen.

---

### Challenge 9

Prevent two users from registering with the same email.

What database constraint would help?

---

### Challenge 10

Imagine creating an order requires:

```text
create order
reduce inventory
create order items
```

Which operations would you consider putting into a transaction?

---

# 47. Interview Questions

## 1. What is SQL?

SQL is a language used to query and manipulate relational databases.

---

## 2. What is CRUD?

CRUD stands for:

```text
Create
Read
Update
Delete
```

Common SQL commands are:

```text
INSERT
SELECT
UPDATE
DELETE
```

---

## 3. What is a JOIN?

A JOIN combines related rows from multiple tables based on a condition.

---

## 4. INNER JOIN vs LEFT JOIN?

An `INNER JOIN` returns rows where matching records exist in both tables.

A `LEFT JOIN` returns all rows from the left table and matching rows from the right table when available.

---

## 5. What is an index?

An index is a data structure that can help the database locate rows more efficiently for certain queries.

---

## 6. Why not create indexes on every column?

Indexes consume storage and must be maintained when data changes, which can increase write costs. Indexes should be created based on actual query patterns and workload.

---

## 7. What is a transaction?

A transaction groups multiple database operations into one logical unit of work that can be committed or rolled back.

---

## 8. What happens if you forget WHERE in UPDATE?

You can update every row in the table.

For example:

```sql
UPDATE users
SET username = 'test';
```

can change every user's username.

---

## 9. What is SQL injection?

SQL injection is a security vulnerability where untrusted input is interpreted as part of a SQL statement.

Parameterized queries are a standard way to prevent this class of attack.

---

## 10. What is the difference between WHERE and HAVING?

`WHERE` filters rows before grouping.

`HAVING` filters groups after `GROUP BY`.

---

# 48. Interview Scenario

Here's one I'd expect a junior backend developer to be able to reason through.

> You have a `users` table with 10 million records. Your API searches users by email, and the endpoint has become slow. What would you investigate?

Don't immediately say:

> "Add an index."

That's probably part of the answer, but think first.

I'd investigate:

```text
What query is being executed?
        ↓
How long does it take?
        ↓
Is email indexed?
        ↓
What does the query plan show?
        ↓
How many rows are being scanned?
        ↓
Is the application fetching unnecessary data?
        ↓
Are there other bottlenecks?
```

Then, if the query pattern is something like:

```sql
SELECT *
FROM users
WHERE email = $1;
```

an index on `email` may be appropriate.

This is the mindset I want you to develop:

> **Don't blindly apply performance fixes. First understand the query and the workload.**

---

# 49. What You Should Be Able To Do Now

After this chapter, you should be comfortable looking at something like:

```sql
SELECT
    p.title,
    u.username,
    COUNT(c.id) AS comment_count
FROM posts AS p
JOIN users AS u
    ON p.user_id = u.id
LEFT JOIN comments AS c
    ON c.post_id = p.id
GROUP BY p.id, p.title, u.username
ORDER BY comment_count DESC;
```

and not think:

> "What the hell is this?"

You might not be able to write it from memory yet.

That's fine.

You should be able to read it:

```text
Get post title
+
author username
+
number of comments

from posts

join users
→ find author

left join comments
→ include posts even without comments

group
→ count comments per post

sort
→ most comments first
```

That's real progress.

---

# 50. One Last Thing About SQL

You don't need to become a database administrator to be a good backend engineer.

But you should know enough SQL that your database isn't a mystery.

When you're building an API, you should be able to think:

```text
HTTP request
     ↓
What data do I need?
     ↓
What query retrieves it?
     ↓
What tables are involved?
     ↓
Do I need a JOIN?
     ↓
Could this query become slow?
     ↓
Do I need an index?
     ↓
Does this operation need a transaction?
     ↓
What happens if it fails?
```

That's the skill we're trying to build.

---

# Next — Chapter 9: ORMs & Database Tools

We've now learned how to talk directly to PostgreSQL.

But writing raw SQL for every operation can become repetitive in a large application.

That's where **ORMs** come in.

We'll look at:

```text
What is an ORM?
Why do people use ORMs?
Prisma
Database schema
Migrations
Models
Relations
Raw SQL vs ORM
ORM advantages
ORM disadvantages
N+1 queries
Transactions
```

And we'll take our forum database and connect it to our Node.js backend.

But one rule stays the same:

> **Learn the database first. Use the ORM as a tool, not as a replacement for understanding SQL.**