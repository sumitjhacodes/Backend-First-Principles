# Chapter 4 — Routing & URL Design

In the previous chapter, we built a small HTTP server.

We had things like:

```text
GET  /todos
POST /todos
```

And our server looked at the request and decided what code to run.

But imagine your application grows.

You now have:

```text
/users
/products
/orders
/todos
/comments
/payments
/notifications
```

And each one might have 5, 10, or even 20 operations.

You don't want your entire backend to become a giant collection of:

```js
if (req.method === "GET" && req.url === "/users") {
  // ...
}

if (req.method === "POST" && req.url === "/users") {
  // ...
}

if (req.method === "GET" && req.url === "/products") {
  // ...
}
```

That's where **routing** comes in.

And once you start building real APIs, another question becomes important:

> **How should I design my URLs in the first place?**

That's what this chapter is about.

---

# 1. What Is Routing?

Let's start with the simplest definition.

> **Routing is the process of deciding which code should handle an incoming request.**

Suppose a request arrives:

```text
GET /users
```

Your application needs to figure out:

> "Which code should handle `GET /users`?"

Maybe:

```js
getUsers()
```

Then another request arrives:

```text
POST /users
```

That should probably run:

```js
createUser()
```

So routing is basically a mapping:

```text
HTTP Method + URL
        ↓
   Route Handler
```

For example:

```text
GET  /users       → getUsers
POST /users       → createUser
GET  /users/42    → getUser
PATCH /users/42   → updateUser
DELETE /users/42  → deleteUser
```

That's routing.

---

# 2. A Route Has More Than Just a URL

A common beginner mistake is thinking:

```text
/users
```

is the route.

Not exactly.

A route is usually identified by:

```text
HTTP method + path
```

So these are different routes:

```text
GET  /users
POST /users
PUT  /users
DELETE /users
```

Even though the path is the same.

For example:

```text
GET /users
```

might mean:

> Give me users.

While:

```text
POST /users
```

means:

> Create a new user.

Same URL.

Different HTTP method.

Different operation.

This is one reason HTTP methods matter so much.

---

# 3. Let's Use Express

In the previous chapter we briefly saw Express.

Now we'll use it properly.

A basic Express route looks like:

```js
app.get("/users", (req, res) => {
  res.json({
    message: "Get users",
  });
});
```

The important parts are:

```text
app.get()
   ↓
HTTP method

"/users"
   ↓
URL path

(req, res) => {}
   ↓
Handler
```

So:

```js
app.get("/users", handler);
```

basically says:

> "When a GET request comes to `/users`, run this handler."

---

# 4. Different HTTP Methods

You can define different routes for different methods:

```js
app.get("/users", (req, res) => {
  // Get users
});

app.post("/users", (req, res) => {
  // Create user
});

app.patch("/users/:id", (req, res) => {
  // Update user
});

app.delete("/users/:id", (req, res) => {
  // Delete user
});
```

Now our API has a clear structure:

```text
GET     /users
POST    /users
PATCH   /users/:id
DELETE  /users/:id
```

This is much easier to understand.

---

# 5. What Is an Endpoint?

You'll hear the word **endpoint** constantly.

An endpoint is basically a specific API entry point that clients can call.

For example:

```text
GET /users
```

is an endpoint.

So is:

```text
POST /users
```

And:

```text
GET /users/42
```

The URL alone isn't always enough to uniquely identify the operation.

Think:

```text
GET /users
```

and:

```text
POST /users
```

as two different endpoints because they perform different operations.

---

# 6. What Is a Resource?

This word is important when talking about REST APIs.

A **resource** is a thing your API manages.

For example, in a blogging application:

```text
Users
Posts
Comments
Likes
```

can all be resources.

In an e-commerce application:

```text
Products
Orders
Customers
Payments
```

can be resources.

Usually, we represent collections using plural nouns:

```text
/users
/products
/orders
/posts
```

Rather than:

```text
/getUsers
/createProduct
/deleteOrder
```

Why?

Because the HTTP method already tells us what action we're performing.

---

# 7. Don't Put Actions Everywhere

You might initially design an API like this:

```text
GET  /getUsers
POST /createUser
POST /deleteUser
POST /updateUser
```

It works.

But it's not a very clean REST-style design.

A more conventional design is:

```text
GET    /users
POST   /users
PATCH  /users/:id
DELETE /users/:id
```

The method describes the operation.

The URL describes the resource.

That's a useful rule:

> **Use nouns in URLs. Use HTTP methods to describe the operation.**

Not an absolute law—real APIs sometimes need action endpoints—but it's a very good default.

---

# 8. Collection vs Individual Resource

This is one of the most useful concepts when designing APIs.

Suppose we have users.

```text
/users
```

represents the collection of users.

For example:

```text
GET /users
```

means:

> Give me users.

But:

```text
/users/42
```

represents one specific user.

So:

```text
GET /users/42
```

means:

> Give me user 42.

Think of it like:

```text
/users
   ↓
All users

/users/42
   ↓
One specific user
```

---

# 9. Path Parameters

Now let's understand:

```text
/users/42
```

What is `42`?

It's a **path parameter**.

In Express:

```js
app.get("/users/:id", (req, res) => {
  console.log(req.params.id);
});
```

If the client requests:

```text
GET /users/42
```

then:

```js
req.params.id
```

will contain:

```text
42
```

The `:` tells Express:

> "This part of the URL is dynamic."

---

# 10. Why Do We Need Path Parameters?

Imagine we have 10 million users.

We obviously can't create routes like:

```text
/users/1
/users/2
/users/3
/users/4
...
```

Instead we create one route:

```text
/users/:id
```

And the value changes depending on the request.

```text
/users/1
/users/42
/users/982
/users/50000
```

All can be handled by:

```js
app.get("/users/:id", ...)
```

That's the point of path parameters.

---

# 11. Path Parameter Means "Which One?"

A useful mental shortcut:

> **Path parameters usually identify a specific resource.**

Examples:

```text
/users/42
/products/100
/orders/abc123
/posts/900
```

The value after the resource name identifies which resource you're talking about.

---

# 12. Multiple Path Parameters

You can have multiple parameters.

For example:

```text
/users/42/orders/1001
```

This could mean:

> Order `1001` belonging to user `42`.

Express:

```js
app.get("/users/:userId/orders/:orderId", (req, res) => {
  const { userId, orderId } = req.params;

  res.json({
    userId,
    orderId,
  });
});
```

Request:

```text
GET /users/42/orders/1001
```

Response:

```json
{
  "userId": "42",
  "orderId": "1001"
}
```

Notice something important:

> Express gives you route parameters as strings.

If you need a number:

```js
const userId = Number(req.params.userId);
```

And you should validate it.

Don't blindly trust input from URLs.

---

# 13. Query Parameters

Now consider this:

```text
/users?role=admin
```

What's `role=admin`?

That's a **query parameter**.

Query parameters come after:

```text
?
```

Example:

```text
/products?category=shoes
```

or:

```text
/products?category=shoes&sort=price
```

or:

```text
/products?page=2&limit=20
```

They are commonly used for:

* filtering
* sorting
* searching
* pagination
* optional configuration

---

# 14. Path Parameter vs Query Parameter

This is one of the most common interview questions.

Consider:

```text
/users/42
```

versus:

```text
/users?id=42
```

They're both technically possible.

But they communicate different ideas.

### Path parameter

```text
/users/42
```

usually means:

> I want this specific user.

### Query parameter

```text
/users?id=42
```

usually means:

> I'm querying the users collection with a filter.

That's a subtle but important difference.

A useful mental model:

```text
Path parameter
→ identity

Query parameter
→ filtering / options
```

---

# 15. Example: Product API

Imagine:

```text
GET /products
```

returns all products.

Now you want:

> Give me products under ₹1,000.

You could use:

```text
GET /products?maxPrice=1000
```

Or:

> Give me shoes.

```text
GET /products?category=shoes
```

Or both:

```text
GET /products?category=shoes&maxPrice=1000
```

The path stays:

```text
/products
```

because you're still asking about the product collection.

You're just changing the query.

---

# 16. Searching

Search is commonly represented with query parameters.

For example:

```text
GET /products?search=laptop
```

Or:

```text
GET /users?q=sumit
```

There isn't one universal parameter name.

You might see:

```text
?q=
?search=
?query=
```

The important thing is consistency within your API.

---

# 17. Sorting

Suppose:

```text
GET /products
```

returns products.

You want the cheapest products first.

You might design:

```text
GET /products?sort=price&order=asc
```

Or:

```text
GET /products?sort=price_asc
```

Both can work.

The important thing is that clients can understand and use your API consistently.

---

# 18. Pagination

Suppose you have:

```text
1,000,000 products
```

Would you return all of them?

No.

That would be a terrible idea.

Instead:

```text
GET /products?page=1&limit=20
```

means:

> Give me the first 20 products.

Then:

```text
GET /products?page=2&limit=20
```

means:

> Give me the next 20.

We'll go much deeper into pagination in Chapter 12.

For now, just understand that query parameters are commonly used to control how collections are returned.

---

# 19. A URL Can Have Both

You can combine path and query parameters.

For example:

```text
GET /users/42/orders?status=completed&limit=10
```

Here:

```text
/users/42/orders
```

is the path.

```text
42
```

is a path parameter.

And:

```text
status=completed
limit=10
```

are query parameters.

Express:

```js
app.get("/users/:userId/orders", (req, res) => {
  const { userId } = req.params;

  const { status, limit } = req.query;

  res.json({
    userId,
    status,
    limit,
  });
});
```

---

# 20. `req.params` vs `req.query`

Remember this.

### Path parameters

```js
req.params
```

Example:

```text
/users/42
```

```js
req.params.id
```

---

### Query parameters

```js
req.query
```

Example:

```text
/users?role=admin
```

```js
req.query.role
```

So:

```text
/users/:id
       ↓
req.params.id
```

and:

```text
/users?role=admin
       ↓
req.query.role
```

This distinction will become second nature with practice.

---

# 21. Nested Resources

Suppose users can have posts.

You might design:

```text
/users/42/posts
```

This means:

> Posts belonging to user 42.

Then:

```text
/users/42/posts/100
```

means:

> Post 100 belonging to user 42.

This can make relationships obvious.

But don't blindly nest everything.

We'll talk about that in a moment.

---

# 22. Don't Over-Nest URLs

Imagine:

```text
/companies/1/departments/2/teams/5/employees/10/projects/20
```

Technically possible.

But horrible to work with.

At some point, you should ask:

> Does the URL actually need to communicate all of these relationships?

Maybe:

```text
/employees/10
```

is enough.

And:

```text
?teamId=5
```

can handle filtering when necessary.

Good API design isn't about making URLs complicated.

It's about making them **clear**.

---

# 23. Singular or Plural?

Prefer consistency.

A common convention is:

```text
/users
/products
/orders
/posts
```

rather than:

```text
/user
/product
/order
/post
```

Why plural?

Because:

```text
/users
```

naturally represents a collection.

Then:

```text
/users/42
```

represents one member of that collection.

You don't have to treat this as a law of nature.

The important thing is:

> Pick a convention and stay consistent.

---

# 24. Trailing Slashes

You might see:

```text
/users
```

and:

```text
/users/
```

Some systems treat them differently.

Some normalize them.

Don't create inconsistent APIs where:

```text
/users
```

works but:

```text
/users/
```

behaves strangely.

Choose a convention and configure your framework appropriately.

For most APIs, you'll commonly see:

```text
/users
```

without a trailing slash.

---

# 25. URL Naming

Prefer readable names.

Good:

```text
/users
/user-profiles
/order-history
```

Avoid unnecessarily strange names:

```text
/getAllUsr
/doUserStuff
/fetch_everything
```

URLs are part of your API's interface.

Other developers will read them.

Your future self will read them.

Make them boring.

Boring APIs are often good APIs.

---

# 26. Don't Put Sensitive Data in URLs

This is important.

URLs can appear in:

* browser history
* server logs
* proxy logs
* monitoring systems
* analytics
* referrer information in some situations

So don't do something like:

```text
/reset-password?token=SECRET_TOKEN
```

unless you've deliberately designed and secured that flow with awareness of those risks.

And definitely don't put passwords in URLs:

```text
/login?email=sumit@example.com&password=123456
```

Never.

Credentials and sensitive data should be handled appropriately in the request body, headers, or secure mechanisms depending on the use case.

---

# 27. URL Design Is About Communication

This is something I want you to remember.

When you design:

```text
GET /users/42/orders
```

you're communicating something to another developer.

You're saying:

> "We're talking about orders associated with user 42."

When you design:

```text
GET /orders?userId=42
```

you're saying:

> "We're querying the orders collection and filtering by user."

Both can be valid.

The question is:

> **Which representation makes more sense for your API?**

This is why API design is not just syntax.

It's communication.

---

# 28. Let's Design a Todo API Properly

Let's return to the project we've been building.

We have a Todo resource.

Our API could look like this:

```text
GET     /todos
GET     /todos/:id

POST    /todos

PATCH   /todos/:id

DELETE  /todos/:id
```

Let's understand each one.

---

### Get all todos

```text
GET /todos
```

Meaning:

> Give me the todo collection.

---

### Get one todo

```text
GET /todos/42
```

Meaning:

> Give me todo 42.

---

### Create a todo

```text
POST /todos
```

The new todo is sent in the request body:

```json
{
  "title": "Learn backend"
}
```

---

### Update a todo

```text
PATCH /todos/42
```

Body:

```json
{
  "completed": true
}
```

---

### Delete a todo

```text
DELETE /todos/42
```

Meaning:

> Delete todo 42.

This is a clean API.

---

# 29. Let's Implement It

Here's a small Express implementation.

```js
const express = require("express");

const app = express();

app.use(express.json());

const todos = [
  {
    id: 1,
    title: "Learn backend",
    completed: false,
  },
];

app.get("/todos", (req, res) => {
  res.status(200).json(todos);
});

app.get("/todos/:id", (req, res) => {
  const id = Number(req.params.id);

  const todo = todos.find((todo) => todo.id === id);

  if (!todo) {
    return res.status(404).json({
      error: "Todo not found",
    });
  }

  res.status(200).json(todo);
});

app.post("/todos", (req, res) => {
  const { title } = req.body;

  if (!title) {
    return res.status(400).json({
      error: "Title is required",
    });
  }

  const todo = {
    id: todos.length + 1,
    title,
    completed: false,
  };

  todos.push(todo);

  res.status(201).json(todo);
});

app.patch("/todos/:id", (req, res) => {
  const id = Number(req.params.id);

  const todo = todos.find((todo) => todo.id === id);

  if (!todo) {
    return res.status(404).json({
      error: "Todo not found",
    });
  }

  const { title, completed } = req.body;

  if (title !== undefined) {
    todo.title = title;
  }

  if (completed !== undefined) {
    todo.completed = completed;
  }

  res.status(200).json(todo);
});

app.delete("/todos/:id", (req, res) => {
  const id = Number(req.params.id);

  const index = todos.findIndex((todo) => todo.id === id);

  if (index === -1) {
    return res.status(404).json({
      error: "Todo not found",
    });
  }

  todos.splice(index, 1);

  res.status(204).send();
});

app.listen(3000, () => {
  console.log("Todo API running on port 3000");
});
```

Don't worry if this looks like a lot.

Read it slowly.

The important thing is the pattern.

---

# 30. Notice How the Routes Tell a Story

Look at this:

```text
GET    /todos
GET    /todos/1
POST   /todos
PATCH  /todos/1
DELETE /todos/1
```

You can almost understand the API without reading the implementation.

That's what good API design should do.

A developer should be able to look at your routes and understand what your backend does.

---

# 31. What Happens When `/todos/42` Is Requested?

Suppose the client sends:

```text
GET /todos/42
```

Express checks the routes.

It sees:

```js
app.get("/todos/:id", ...)
```

It matches.

Then:

```js
req.params.id
```

is:

```text
"42"
```

We convert it:

```js
const id = Number(req.params.id);
```

Now:

```text
42
```

Then we find the todo:

```js
const todo = todos.find((todo) => todo.id === id);
```

If it exists:

```text
200 OK
```

If it doesn't:

```text
404 Not Found
```

This is routing + application logic working together.

---

# 32. Route Order Matters

Here's a subtle Express issue.

Suppose you have:

```js
app.get("/users/:id", (req, res) => {
  // ...
});

app.get("/users/me", (req, res) => {
  // ...
});
```

Now request:

```text
GET /users/me
```

Depending on route matching/order, `/users/:id` can capture `"me"` as the `id`.

So you might want:

```js
app.get("/users/me", ...);

app.get("/users/:id", ...);
```

The specific route comes first.

This is one reason you shouldn't blindly add routes without understanding how your router matches them.

---

# 33. Route Parameters Need Validation

Remember:

```text
GET /todos/abc
```

is possible.

Your code does:

```js
const id = Number(req.params.id);
```

Now:

```text
Number("abc")
```

becomes:

```text
NaN
```

So your API should validate the value.

For example:

```js
const id = Number(req.params.id);

if (!Number.isInteger(id)) {
  return res.status(400).json({
    error: "Invalid todo ID",
  });
}
```

Never assume that because your frontend normally sends valid data, the backend will only receive valid data.

Clients can be buggy.

Clients can be malicious.

Clients can be old versions of your application.

Your backend must protect itself.

---

# 34. Query Parameters Are Also Untrusted Input

Suppose:

```text
GET /products?limit=abc
```

Don't blindly do:

```js
const limit = Number(req.query.limit);
```

and assume everything is okay.

Validate:

```text
Is it a number?
Is it positive?
Is it within a reasonable maximum?
```

For example:

```text
limit=20
```

might be okay.

But:

```text
limit=999999999
```

probably shouldn't cause your server to attempt returning nearly a billion records.

This becomes very important when we build APIs for production.

---

# 35. What About `/search`?

Here's an interesting design question.

Would you do:

```text
GET /search/laptop
```

or:

```text
GET /search?q=laptop
```

Both can be valid.

But if search is really a query over products, you might prefer:

```text
GET /products?q=laptop
```

because you're saying:

> "Search the product collection."

While:

```text
/search/laptop
```

can make sense if search itself is treated as a distinct resource/operation.

There isn't always one correct answer.

Good engineers understand the tradeoff instead of memorizing one rule.

---

# 36. REST Is a Set of Principles, Not Magic

You've probably heard:

> "REST API"

A RESTful style generally encourages ideas such as:

* resources
* standard HTTP methods
* stateless requests
* meaningful URLs
* representations such as JSON
* using HTTP semantics appropriately

But don't fall into the trap of thinking:

> "If my URL isn't exactly `/users/:id`, my API isn't REST."

Real-world APIs aren't all perfectly RESTful.

You should understand the principles rather than trying to win a REST purity contest.

---

# 37. Common API Design Mistakes

### Mistake 1: Using verbs everywhere

Bad:

```text
GET /getUsers
POST /createUser
POST /deleteUser
```

Prefer:

```text
GET    /users
POST   /users
DELETE /users/:id
```

---

### Mistake 2: Mixing singular and plural

Avoid:

```text
/users
/products
/order
/customer
```

Pick a convention.

For example:

```text
/users
/products
/orders
/customers
```

---

### Mistake 3: Putting filters in the path

Instead of:

```text
/products/cheap
```

you might use:

```text
/products?maxPrice=1000
```

if "cheap" is really a filter.

---

### Mistake 4: Returning everything

Don't make:

```text
GET /users
```

return every user field and every related object by default.

We'll learn about pagination and response design later.

---

### Mistake 5: Ignoring status codes

Don't return:

```text
200 OK
```

for every possible situation.

Use HTTP semantics properly.

For example:

```text
200 → successful request
201 → resource created
204 → successful request with no response body
400 → invalid request
401 → unauthenticated
403 → forbidden
404 → resource not found
409 → conflict
500 → server error
```

We already covered status codes in Chapter 2.

Now we're using them in actual route design.

---

# 38. How Would You Design an E-Commerce API?

Let's practice.

Imagine you're building an e-commerce backend.

You have:

```text
Users
Products
Orders
Reviews
```

Start with the resources.

```text
/users
/products
/orders
/reviews
```

Now operations.

### Users

```text
GET    /users
GET    /users/:id
POST   /users
PATCH  /users/:id
DELETE /users/:id
```

### Products

```text
GET    /products
GET    /products/:id
POST   /products
PATCH  /products/:id
DELETE /products/:id
```

### Orders

```text
GET    /orders
GET    /orders/:id
POST   /orders
PATCH  /orders/:id
```

Notice something:

Maybe you shouldn't allow:

```text
DELETE /orders/:id
```

depending on your business rules.

An order usually isn't something you simply delete.

Maybe you instead support:

```text
PATCH /orders/:id
```

with:

```json
{
  "status": "cancelled"
}
```

That's where business requirements start influencing API design.

---

# 39. What About Authentication?

Suppose:

```text
GET /users
```

should only be available to administrators.

You don't necessarily change the URL to:

```text
GET /admin/getUsers
```

You can keep:

```text
GET /users
```

and protect it using authentication/authorization middleware.

Conceptually:

```text
Request
   ↓
Authentication
   ↓
Authorization
   ↓
GET /users
   ↓
Handler
```

We'll learn authentication properly in Chapter 10.

---

# 40. API Versioning

You might eventually see:

```text
/api/v1/users
/api/v2/users
```

Why?

Because APIs evolve.

Suppose version 1 returns:

```json
{
  "name": "Rahul"
}
```

And version 2 changes it to:

```json
{
  "firstName": "Rahul",
  "lastName": "Sharma"
}
```

If old clients depend on version 1, suddenly changing the response can break them.

Versioning gives you a way to evolve your API while keeping compatibility.

We'll cover API versioning properly later.

For now, just recognize:

```text
/v1
/v2
```

as one common approach.

---

# 41. A Practical API Design Checklist

When designing a new endpoint, ask yourself:

### 1. What resource am I working with?

```text
users
orders
products
posts
```

### 2. Am I working with a collection or one resource?

```text
/users
/users/:id
```

### 3. What HTTP method represents the operation?

```text
GET
POST
PATCH
DELETE
```

### 4. Is this identifying a resource or filtering a collection?

Identity:

```text
/users/42
```

Filtering:

```text
/users?role=admin
```

### 5. What should happen if the input is invalid?

```text
400
```

### 6. What if the resource doesn't exist?

```text
404
```

### 7. What if the operation isn't allowed?

Potentially:

```text
403
```

### 8. What happens if the same request is repeated?

This becomes important for idempotency and we'll revisit it later.

---

# 42. Interview Questions

## 1. What is routing?

Routing is the process of mapping an incoming request's HTTP method and URL path to the code that should handle it.

---

## 2. What is an endpoint?

An endpoint is a specific API entry point, typically identified by an HTTP method and URL path, that performs a particular operation.

For example:

```text
GET /users
POST /users
GET /users/:id
```

---

## 3. What is a path parameter?

A path parameter is a dynamic value embedded in the URL path, commonly used to identify a specific resource.

Example:

```text
/users/42
```

Here `42` is the parameter.

In Express:

```js
req.params.id
```

---

## 4. What is a query parameter?

A query parameter is a value provided after `?` in a URL.

Example:

```text
/products?category=shoes&sort=price
```

They're commonly used for filtering, searching, sorting, and pagination.

---

## 5. Path parameter vs query parameter?

A good interview answer:

> Path parameters are generally used to identify a specific resource, while query parameters are generally used for filtering, sorting, searching, pagination, or other optional query configuration.

Example:

```text
/users/42
```

identifies user 42.

```text
/users?role=admin
```

filters users.

---

## 6. Why use nouns instead of verbs in REST URLs?

Because the resource is represented by the URL while the HTTP method describes the operation.

Instead of:

```text
POST /createUser
```

we can use:

```text
POST /users
```

The method already tells us that we're creating something.

---

## 7. Should all APIs use plural resource names?

Not necessarily.

There is no universal law requiring plural names.

But plural resource names such as:

```text
/users
/products
/orders
```

are a common convention because they clearly represent collections.

Consistency matters more than arguing about singular vs plural.

---

## 8. What is nested routing?

Nested routing represents a relationship between resources.

Example:

```text
/users/42/orders
```

means orders associated with user 42.

But excessive nesting can make APIs difficult to use, so nesting should be kept reasonable.

---

## 9. Why should API input be validated?

Because the client cannot be trusted.

A client might send:

```text
/users/abc
```

or:

```text
/products?limit=999999999
```

or completely malformed data.

The backend must validate input before using it.

---

## 10. What happens if no route matches?

Typically the server returns:

```text
404 Not Found
```

Your application should have a consistent way to handle unknown routes.

For example:

```js
app.use((req, res) => {
  res.status(404).json({
    error: "Route not found",
  });
});
```

---

# 43. Practical Interview Scenario

Here's a question that tells me much more about your understanding than memorizing definitions.

> **Design the API for a blog application.**

You have:

```text
Users
Posts
Comments
```

A reasonable starting design:

```text
GET    /users
GET    /users/:id
POST   /users
PATCH  /users/:id

GET    /posts
GET    /posts/:id
POST   /posts
PATCH  /posts/:id
DELETE /posts/:id

GET    /posts/:postId/comments
POST   /posts/:postId/comments

GET    /comments/:id
PATCH  /comments/:id
DELETE /comments/:id
```

Then I'd ask:

> Why did you nest comments under posts?

Your answer could be:

> "Because comments are naturally associated with a post, and `/posts/:postId/comments` makes that relationship clear. But I would avoid nesting comments under users and posts simultaneously if that makes the URLs unnecessarily deep."

That's an engineer's answer.

Not:

> "Because REST says so."

---

# 44. Your Challenge

Don't just read this chapter.

Take your Todo API from Chapter 3 and redesign it yourself.

Your final routes should roughly look like:

```text
GET     /todos
GET     /todos/:id
POST    /todos
PATCH   /todos/:id
DELETE  /todos/:id
```

Then add filtering:

```text
GET /todos?completed=true
```

Add searching:

```text
GET /todos?search=backend
```

Add pagination:

```text
GET /todos?page=1&limit=10
```

And combine them:

```text
GET /todos?completed=false&search=backend&page=1&limit=10
```

Your job is to decide:

* how to validate each parameter
* what happens when `page` is invalid
* what happens when `limit` is too large
* what happens when a todo doesn't exist
* which status code each situation should return

Don't worry about the database yet.

Keep using the in-memory array.

---

# 45. One Last Mental Model

When you see:

```text
GET /users/42/orders?status=completed&limit=10
```

break it down immediately:

```text
GET
↓
HTTP method

/users/42/orders
↓
URL path

42
↓
Path parameter

status=completed
limit=10
↓
Query parameters
```

Then ask:

> What code should handle this?

Something like:

```js
app.get("/users/:userId/orders", handler);
```

Inside:

```js
req.params.userId
req.query.status
req.query.limit
```

That's routing.

And that's the skill I want you to develop—not memorizing Express syntax, but looking at a request and immediately understanding **what it means and how your backend should handle it**.

---

# What We've Learned So Far

Our backend journey now looks like:

```text
Chapter 1
How the Internet Works
        ↓
Chapter 2
HTTP
        ↓
Chapter 3
Web Server + Node.js
        ↓
Chapter 4
Routing + URL Design
```

And our Todo API has evolved from:

```text
"Hello World"
```

to:

```text
GET    /todos
GET    /todos/:id
POST   /todos
PATCH  /todos/:id
DELETE /todos/:id
```

That's not just theory anymore.

You're starting to build an actual API.

---

# Next: Chapter 5 — JSON & Data Formats

So now we have a problem.

Our client and server need to exchange data.

For example, when we create a todo:

```text
Client
   ↓
POST /todos

{
  "title": "Learn backend"
}
```

How does that object travel over HTTP?

Why do we use JSON?

What exactly happens when JSON is parsed?

What's the difference between:

```text
JavaScript object
JSON string
HTTP request body
```

What is serialization?

What happens if someone sends malformed JSON?

And why do APIs usually return:

```json
{
  "id": 1,
  "title": "Learn backend"
}
```

instead of some random format?

That's what we'll break down in **Chapter 5 — JSON & Data Formats**.