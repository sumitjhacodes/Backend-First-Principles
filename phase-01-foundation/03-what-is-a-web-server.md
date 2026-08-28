# Chapter 3 — What Is a Web Server?

So far we've learned two important things.

In Chapter 1, we looked at how computers communicate:

```text
Client
   ↓
Internet
   ↓
Server
```

Then in Chapter 2, we looked at HTTP:

```text
Client
   ↓
HTTP Request
   ↓
Server
   ↓
HTTP Response
   ↓
Client
```

But there's still a big question.

**What exactly is the "server"?**

When we say:

> "The request goes to the server."

What is actually happening?

Is a server a computer?

Is Node.js a server?

Is Express a server?

Is your backend code the server?

If you're a beginner, these words can get mixed together very quickly.

Let's clear that up.

---

# 1. First: What Is a Server?

The simplest definition is:

> **A server is a system that provides something to another system, called a client.**

That's it.

For example:

```text
Your browser → Web server
Your frontend → API server
Your API → Database server
Your application → Redis server
```

The word "server" describes a **role**.

One computer can act as a server.

Another computer can act as a client.

But the same computer can also be both.

For example, on your laptop:

```text
Browser
   ↓
Node.js application
   ↓
PostgreSQL
```

Your browser is acting as a client.

Your Node.js application is acting as a server.

Your PostgreSQL process is also acting as a server.

All of them can be running on your own machine.

That's completely normal during development.

---

# 2. Server Is Not Necessarily a Huge Computer

When beginners hear "server", they sometimes imagine this:

```text
       SERVER
┌──────────────────────┐
│      Huge machine    │
│                      │
│      💻💻💻💻         │
│                      │
└──────────────────────┘
```

That's not really the important part.

A server can be:

* a physical computer
* a virtual machine
* a container
* a process running on your laptop
* a cloud instance
* a serverless function

What makes something a server is mainly **the service it provides**, not what the hardware looks like.

---

# 3. Client vs Server

Let's take a simple example.

You open:

```text
https://github.com
```

Your browser is the client.

GitHub's infrastructure is providing the service.

So:

```text
Browser
  |
  | Request
  ↓
GitHub server
  |
  | Response
  ↓
Browser
```

The client asks.

The server responds.

This is the basic client-server model.

---

# 4. Your Frontend Is Also a Client

This becomes very important once you start building full-stack applications.

Suppose you have:

```text
React Frontend
Node.js Backend
PostgreSQL Database
```

The architecture might look like:

```text
Browser
   |
   | HTTP
   ↓
Node.js API
   |
   | SQL
   ↓
PostgreSQL
```

From the Node.js API's point of view:

```text
Browser → Client
Node.js → Server
```

But from PostgreSQL's point of view:

```text
Node.js → Client
PostgreSQL → Server
```

So "client" and "server" are often **roles in a particular communication**.

That's a useful mental model.

---

# 5. So What Is a Web Server?

Now we can narrow the definition.

A **web server** is software that receives HTTP requests and sends HTTP responses.

For example:

```text
Browser
   |
   | GET /users
   ↓
Web Server
   |
   | 200 OK
   ↓
Browser
```

A web server usually:

1. listens for incoming network connections
2. receives HTTP requests
3. processes or forwards those requests
4. sends HTTP responses

Examples of web server software include:

```text
Nginx
Apache HTTP Server
Caddy
```

And Node.js can also be used to create HTTP servers.

---

# 6. Then What Is Node.js?

This is where things often become confusing.

Node.js is **not a programming language**.

JavaScript is the language.

Node.js is a **JavaScript runtime**.

A runtime is an environment that allows your JavaScript code to execute outside the browser.

For example, normally JavaScript runs inside a browser:

```text
Chrome
  ↓
JavaScript
```

Node.js lets you run JavaScript directly on your computer/server:

```text
Operating System
       ↓
    Node.js
       ↓
  JavaScript
```

Because Node.js provides APIs for networking, files, processes, streams, etc., we can use it to build backend applications.

---

# 7. Browser JavaScript vs Node.js

This distinction is worth understanding.

### Browser

JavaScript can interact with things like:

```text
DOM
window
document
localStorage
cookies
```

### Node.js

JavaScript can interact with things like:

```text
File system
Network
TCP
HTTP
Processes
Environment variables
Streams
```

So when you write:

```js
console.log("Hello");
```

Node.js can execute it without a browser.

And when you write:

```js
const http = require("http");
```

Node provides an HTTP API that lets your JavaScript application communicate over the network.

---

# 8. Your First Real Web Server

Let's build one.

Create:

```text
server.js
```

Add:

```js
const http = require("http");

const server = http.createServer((req, res) => {
  res.end("Hello from my server!");
});

server.listen(3000, () => {
  console.log("Server running on port 3000");
});
```

Run:

```bash
node server.js
```

Then open:

```text
http://localhost:3000
```

You'll see:

```text
Hello from my server!
```

Congratulations.

You have created a web server.

But let's understand what just happened instead of moving on immediately.

---

# 9. What Does `createServer()` Do?

This:

```js
http.createServer(...)
```

creates an HTTP server.

You give it a function:

```js
(req, res) => {
   ...
}
```

That function runs when a request arrives.

So:

```text
Browser
   |
   | HTTP Request
   ↓
Node.js HTTP Server
   |
   ↓
Your callback function
   |
   ↓
HTTP Response
```

The two important objects are:

```text
req
res
```

---

# 10. What Is `req`?

`req` means:

> **Request**

It contains information about the incoming HTTP request.

For example:

```js
req.method
```

might be:

```text
GET
```

And:

```js
req.url
```

might be:

```text
/users
```

You can inspect it:

```js
const http = require("http");

const server = http.createServer((req, res) => {
  console.log("Method:", req.method);
  console.log("URL:", req.url);

  res.end("Hello!");
});

server.listen(3000);
```

Now visit:

```text
http://localhost:3000/users
```

Your terminal might show:

```text
Method: GET
URL: /users
```

This is the same HTTP request we learned about in Chapter 2.

---

# 11. What Is `res`?

`res` means:

> **Response**

It's what you use to send something back to the client.

For example:

```js
res.end("Hello!");
```

means:

> Finish the response and send `"Hello!"` to the client.

You can also set a status code:

```js
res.statusCode = 200;
```

And headers:

```js
res.setHeader("Content-Type", "text/plain");
```

For JSON:

```js
res.setHeader("Content-Type", "application/json");

res.end(
  JSON.stringify({
    message: "Hello",
  })
);
```

Now the client knows:

> "The response body is JSON."

---

# 12. Let's Build a Tiny API

Let's make our server slightly more useful.

```js
const http = require("http");

const server = http.createServer((req, res) => {
  res.setHeader("Content-Type", "application/json");

  if (req.method === "GET" && req.url === "/") {
    res.statusCode = 200;

    res.end(
      JSON.stringify({
        message: "Welcome to my API",
      })
    );

    return;
  }

  if (req.method === "GET" && req.url === "/users") {
    res.statusCode = 200;

    res.end(
      JSON.stringify([
        {
          id: 1,
          name: "Rahul",
        },
        {
          id: 2,
          name: "Priya",
        },
      ])
    );

    return;
  }

  res.statusCode = 404;

  res.end(
    JSON.stringify({
      error: "Route not found",
    })
  );
});

server.listen(3000, () => {
  console.log("Server running at http://localhost:3000");
});
```

Now:

```text
GET /
```

returns:

```json
{
  "message": "Welcome to my API"
}
```

And:

```text
GET /users
```

returns:

```json
[
  {
    "id": 1,
    "name": "Rahul"
  },
  {
    "id": 2,
    "name": "Priya"
  }
]
```

We've now built a tiny API without Express.

That's intentional.

---

# 13. Why Did We Not Start With Express?

Because I don't want you to think this is magic:

```js
app.get("/users", handler);
```

When you use Express, it feels like:

> "I wrote this line and somehow HTTP works."

I want you to know what's underneath.

At a basic level, your application is still dealing with:

```text
HTTP request
     ↓
Node.js
     ↓
Your application
     ↓
HTTP response
```

Express simply gives you a much nicer way to work with this.

---

# 14. What Is Express Then?

Express is a **web framework for Node.js**.

It provides convenient abstractions for things like:

* routing
* middleware
* request handling
* response handling
* error handling

Instead of manually doing:

```js
if (req.method === "GET" && req.url === "/users") {
   ...
}
```

you can write:

```js
app.get("/users", (req, res) => {
  res.json([
    {
      id: 1,
      name: "Rahul",
    },
  ]);
});
```

Much easier.

---

# 15. Framework vs Runtime vs Language

Let's make this distinction very clear.

```text
JavaScript
   ↓
Programming language

Node.js
   ↓
Runtime

Express
   ↓
Web framework
```

These are different things.

You can use:

```text
JavaScript + Node.js
```

without Express.

You can use:

```text
TypeScript + Node.js
```

with Express.

You can also use:

```text
TypeScript + Node.js + Fastify
```

or another framework.

---

# 16. What Actually Happens When a Request Arrives?

This is one of the most important mental models in backend development.

Suppose you request:

```text
GET http://localhost:3000/users
```

Here's a simplified flow:

```text
Browser
   |
   | HTTP Request
   ↓
Operating System
   |
   ↓
Network stack
   |
   ↓
Port 3000
   |
   ↓
Node.js HTTP server
   |
   ↓
Your application
   |
   ↓
Route handling
   |
   ↓
Business logic
   |
   ↓
Response
   |
   ↓
Browser
```

Eventually, your callback runs:

```js
(req, res) => {
   // your code
}
```

That's the point where your application gets to decide what to do.

---

# 17. What Does "Listening on a Port" Mean?

When you write:

```js
server.listen(3000);
```

you're telling Node:

> "Listen for incoming connections on port 3000."

So:

```text
localhost:3000
```

can reach your application.

If you change it:

```js
server.listen(5000);
```

your server is now listening on:

```text
localhost:5000
```

If another application is already using port `3000`, you may get an error.

That's because two processes normally can't both bind to the exact same IP/port combination.

You might see something like:

```text
EADDRINUSE
```

which basically means:

> Address already in use.

This is one of those errors you'll probably encounter during development.

And now you'll know what it means.

---

# 18. What Is Middleware?

If you're going to use Express, this word is everywhere.

**Middleware** is code that runs during the request/response processing pipeline.

Imagine:

```text
Request
   ↓
Middleware 1
   ↓
Middleware 2
   ↓
Route handler
   ↓
Response
```

A middleware might:

* log requests
* authenticate users
* validate input
* parse JSON
* add information to the request
* handle errors

Think of an airport.

Before you reach the gate:

```text
Entrance
   ↓
Security
   ↓
Passport check
   ↓
Gate
```

Each step can inspect or process you before you reach the final destination.

Middleware is somewhat similar.

---

# 19. A Simple Express Middleware Example

Later we'll use more realistic middleware, but here's the basic idea:

```js
app.use((req, res, next) => {
  console.log(req.method, req.url);

  next();
});
```

The important part is:

```js
next();
```

It means:

> "I'm done. Continue to the next middleware/handler."

So:

```text
Request
   ↓
Logging middleware
   ↓
next()
   ↓
Route handler
   ↓
Response
```

If middleware doesn't call `next()` and doesn't send a response, the request may just sit there.

That's another common beginner mistake.

---

# 20. Why Do We Need Middleware?

Imagine every route needs authentication.

Without middleware, you might repeat:

```js
checkToken();
```

inside every route.

```js
app.get("/profile", checkToken, ...);
app.get("/orders", checkToken, ...);
app.get("/settings", checkToken, ...);
app.get("/payments", checkToken, ...);
```

Middleware gives us a reusable pipeline.

For example:

```text
Request
   ↓
CORS middleware
   ↓
Logging middleware
   ↓
Authentication middleware
   ↓
Validation middleware
   ↓
Route handler
   ↓
Response
```

This becomes extremely useful as your application grows.

---

# 21. The Request/Response Lifecycle

Let's make the complete picture.

A request comes in:

```text
GET /users
```

Then:

```text
1. Network connection
        ↓
2. Node receives request
        ↓
3. Middleware runs
        ↓
4. Router finds matching route
        ↓
5. Controller/handler runs
        ↓
6. Business logic executes
        ↓
7. Database may be queried
        ↓
8. Response is created
        ↓
9. Response sent to client
```

For example:

```text
Browser
   ↓
GET /users
   ↓
Logging
   ↓
Authentication
   ↓
Route
   ↓
Controller
   ↓
Service
   ↓
PostgreSQL
   ↓
Service
   ↓
Controller
   ↓
JSON response
   ↓
Browser
```

We'll build toward this architecture throughout the series.

---

# 22. Where Does the Database Fit?

A common beginner diagram is:

```text
Frontend
   ↓
Backend
   ↓
Database
```

That's fine as a starting point.

But the backend doesn't usually just blindly pass requests to the database.

Instead:

```text
Request
   ↓
Route
   ↓
Controller
   ↓
Business Logic
   ↓
Database
```

For example:

```text
POST /users
```

might become:

```text
Receive request
      ↓
Validate email
      ↓
Check authentication
      ↓
Hash password
      ↓
Create user
      ↓
Save to PostgreSQL
      ↓
Return 201
```

The backend is where your application's rules live.

---

# 23. What Is Business Logic?

This phrase will come up constantly.

Business logic is basically:

> **The rules that define how your application should behave.**

Suppose you're building a shopping application.

Rules might be:

```text
A product must have stock.

A user can't buy more items than are available.

A discount cannot exceed 50%.

A cancelled order cannot be shipped.

Only admins can delete products.
```

Those rules are business logic.

HTTP doesn't know any of this.

Node.js doesn't know any of this.

Express doesn't know any of this.

**You write these rules.**

---

# 24. Node.js and the Event Loop

Now we get into one of the most important Node.js concepts.

You've probably heard:

> "Node.js is single-threaded and uses an event loop."

This sentence gets repeated so much that many people memorize it without understanding it.

Let's fix that.

---

# 25. The Basic Problem

Suppose your server receives:

```text
GET /users
```

and your code needs to query a database.

Imagine the database takes:

```text
100ms
```

If Node simply stopped everything for 100ms:

```text
Request 1
   ↓
Database
   ↓
WAIT 100ms
   ↓
Response
```

then the server would waste time doing nothing.

And if many users made requests, this would become a problem.

Node's design is built around **non-blocking I/O**.

---

# 26. What Is I/O?

I/O means:

> **Input / Output**

It includes operations such as:

```text
Reading a file
Talking to a database
Calling another API
Reading from a network
Writing to a file
```

These operations often involve waiting for something outside your JavaScript code.

Node.js is designed so that your JavaScript code doesn't have to sit there doing nothing while these operations are in progress.

---

# 27. A Simple Example

Imagine:

```js
const result = await database.getUsers();
console.log(result);
```

Conceptually:

```text
JavaScript
    |
    | Ask database
    ↓
Database
    |
    | Working...
    |
    ↓
Result ready
    |
    ↓
JavaScript continues
```

While the database is doing its work, Node can handle other work.

This is one of the reasons Node.js is good for I/O-heavy applications.

---

# 28. The Event Loop

The event loop is one part of how Node.js manages asynchronous work.

A very simplified picture:

```text
             ┌──────────────┐
             │ Your JS Code │
             └──────┬───────┘
                    ↓
              Event Loop
                    ↓
          Async operations
                    ↓
          ┌─────────┴─────────┐
          ↓                   ↓
      Database             Network
          ↓                   ↓
          └─────────┬─────────┘
                    ↓
              Ready callback
                    ↓
              Event Loop
                    ↓
             Your JS Code
```

This is simplified on purpose.

Node's internals involve more components, including the operating system and libuv.

But for your first mental model:

> Node starts asynchronous work, doesn't unnecessarily block JavaScript while waiting, and later processes the result when it's ready.

That's the important idea.

---

# 29. Is Node.js Actually Single-Threaded?

This question is tricky.

You'll often hear:

> "Node.js is single-threaded."

A better explanation is:

> **Your JavaScript execution normally runs on a single main thread, while Node.js and the underlying system can use other threads/resources for certain operations.**

For example, Node uses the libuv library, which provides an event loop and a thread pool for certain tasks.

So don't say:

> "Node has only one thread."

That's misleading.

Instead say:

> "JavaScript execution in Node normally runs on a single main thread, while asynchronous I/O and some other operations can be handled outside that main JavaScript execution path."

That's a much better interview answer.

---

# 30. Why Is This Important?

Because blocking the main JavaScript thread can hurt your server.

For example:

```js
while (true) {
  // do something forever
}
```

This is terrible.

The event loop can't move on.

Your server effectively gets stuck.

Even something less obvious can cause trouble:

```js
function expensiveOperation() {
  // millions of CPU-heavy operations
}
```

If you run a huge CPU-heavy task directly on the main thread, other requests may have to wait.

That's why you should understand the difference between:

```text
I/O-bound work
```

and:

```text
CPU-bound work
```

We'll come back to this later.

---

# 31. A Realistic Node.js Request

Imagine 100 users send requests at roughly the same time:

```text
Request 1 → Database
Request 2 → Database
Request 3 → External API
Request 4 → Redis
Request 5 → Database
```

Node doesn't necessarily create one JavaScript thread per request.

Instead, it can efficiently coordinate many I/O operations while the main JavaScript execution thread continues processing work as operations complete.

That's a major reason Node.js became popular for APIs and real-time applications.

But don't turn this into:

> "Node is automatically faster."

It isn't.

A badly designed Node application can still be very slow.

---

# 32. Node.js Is Not Magic

Suppose your API does this:

```text
Request
 ↓
Run 10 million CPU-heavy operations
 ↓
Call database
 ↓
Call another API
 ↓
Generate huge file
 ↓
Response
```

You can't just say:

> "Node is asynchronous, so it's fine."

CPU-heavy work can still block the JavaScript thread.

Backend engineering is always about understanding **where the work happens and what you're waiting for**.

---

# 33. What Is a Reverse Proxy?

You may hear this term soon.

A reverse proxy sits in front of your application servers.

For example:

```text
Internet
   ↓
Nginx
   ↓
Node.js
```

The client talks to Nginx.

Nginx forwards the request to your Node.js application.

A reverse proxy can handle things such as:

* TLS termination
* routing
* load balancing
* compression
* static files
* connection handling

Don't worry about implementing one yet.

We'll cover this properly when we reach production and system design.

---

# 34. Web Server vs Application Server

People don't always use these terms consistently, so don't get obsessed with the terminology.

But conceptually:

```text
Nginx
↓
Web server / reverse proxy

Node.js application
↓
Application server
```

A production architecture might look like:

```text
Client
   ↓
Load Balancer
   ↓
Nginx / Reverse Proxy
   ↓
Node.js application
   ↓
PostgreSQL
```

Sometimes the terminology overlaps.

The important thing is understanding **what each component is responsible for**.

---

# 35. Why Do We Need a Framework?

At this point our raw Node server works.

But imagine your application has 100 routes.

You would end up writing a lot of code like:

```js
if (req.method === "GET" && req.url === "/users") {
   ...
}

if (req.method === "POST" && req.url === "/users") {
   ...
}

if (req.method === "GET" && req.url === "/products") {
   ...
}
```

And then you need:

```text
Routing
Middleware
Validation
Error handling
Authentication
Request parsing
Response formatting
```

That's where frameworks help.

They don't replace HTTP.

They make working with HTTP easier.

---

# 36. Express Example

With Express:

```js
const express = require("express");

const app = express();

app.use(express.json());

app.get("/users", (req, res) => {
  res.json([
    {
      id: 1,
      name: "Rahul",
    },
  ]);
});

app.post("/users", (req, res) => {
  console.log(req.body);

  res.status(201).json({
    message: "User created",
  });
});

app.listen(3000, () => {
  console.log("Server running on port 3000");
});
```

Compare this with the raw Node HTTP server.

Much cleaner.

And now you know what Express is doing for you.

---

# 37. The Architecture We're Building Toward

As this series progresses, we'll move from:

```text
HTTP
 ↓
Node.js
 ↓
Route
 ↓
Response
```

toward:

```text
Client
   ↓
HTTP
   ↓
Web Server / Reverse Proxy
   ↓
Node.js Application
   ↓
Middleware
   ↓
Router
   ↓
Controller
   ↓
Service
   ↓
Repository / ORM
   ↓
PostgreSQL
```

And eventually:

```text
                    ┌── Redis
                    │
Client
   ↓                ├── PostgreSQL
Load Balancer
   ↓                ├── Queue
Backend Servers
   ↓                ├── Object Storage
Services
                    └── External APIs
```

Don't try to understand the whole thing today.

We're going to build this mental model piece by piece.

---

# 38. Common Beginner Mistakes

## Mistake 1: "Node.js is a server."

Not exactly.

Node.js is a runtime.

You can use Node.js to create a server.

For example:

```js
http.createServer(...)
```

creates an HTTP server using Node's HTTP APIs.

---

## Mistake 2: "Express is Node.js."

No.

Express is a framework that runs on Node.js.

```text
Node.js
   ↓
Express
   ↓
Your application
```

---

## Mistake 3: "Backend = database."

No.

The backend can contain:

```text
Routing
Authentication
Business logic
Validation
Database access
Caching
Queues
External API calls
Logging
etc.
```

The database is one part of the system.

---

## Mistake 4: "Async means another JavaScript thread."

Not necessarily.

Asynchronous programming is about how work is scheduled and how your program handles waiting.

Don't equate:

```text
async
```

with:

```text
new thread
```

---

## Mistake 5: "Node.js can handle unlimited requests."

Absolutely not.

Node can efficiently handle many concurrent I/O operations, but your system still has limits:

```text
CPU
Memory
Database
Network
Connections
External services
```

We'll learn about scaling later.

---

# 39. Let's Build a Tiny Real API

Now let's make something slightly more realistic.

Create:

```text
server.js
```

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

app.listen(3000, () => {
  console.log("Todo API running on http://localhost:3000");
});
```

Now test:

```bash
curl http://localhost:3000/todos
```

Then:

```bash
curl -X POST http://localhost:3000/todos \
  -H "Content-Type: application/json" \
  -d '{"title":"Build something"}'
```

You now have:

```text
Client
   ↓
HTTP
   ↓
Express
   ↓
Route
   ↓
Your code
   ↓
Response
```

This is the beginning of our Todo API project.

We'll improve it as we continue through the series.

---

# 40. What We Just Built

It might look small.

That's okay.

Look at what's actually happening:

```text
curl/browser
      ↓
HTTP request
      ↓
Node.js
      ↓
Express
      ↓
Middleware
      ↓
Route
      ↓
Validation
      ↓
Application logic
      ↓
HTTP response
```

This is the foundation of a real backend.

Later, we'll replace:

```text
in-memory array
```

with:

```text
PostgreSQL
```

Then we'll add:

```text
Authentication
Caching
Queues
Testing
Logging
Docker
Deployment
```

The application will grow, but the basic request/response cycle won't disappear.

---

# 41. One Important Question: What Happens If My Server Crashes?

Suppose your Node.js process crashes.

Then:

```text
Node.js process
       X
```

Your API is no longer available.

That's why production systems usually don't depend on a single process.

You might have:

```text
             Load Balancer
              /        \
             ↓          ↓
         Node #1      Node #2
```

If one instance crashes, another can potentially continue serving requests.

We'll learn about this much later.

For now, understand:

> **Your Node.js process is just one running process.**

It isn't the entire internet.

---

# 42. A Better Mental Model of a Backend

When you hear:

> "We have a Node.js backend."

Don't imagine:

```text
Node.js
   ↓
Magic
```

Imagine:

```text
                   HTTP Request
                        ↓
              ┌─────────────────┐
              │ Node.js Process │
              │                 │
              │   Middleware    │
              │       ↓         │
              │     Router      │
              │       ↓         │
              │   Controller    │
              │       ↓         │
              │    Service      │
              │       ↓         │
              │   Database      │
              └─────────────────┘
                        ↓
                   HTTP Response
```

That's much closer to what you'll actually build.

---

# 43. Interview Questions

Before looking at the answers, try answering them yourself.

---

### 1. What is a web server?

A web server is software that accepts HTTP requests and sends HTTP responses. In broader usage, "server" can also refer to a machine or system providing a service.

---

### 2. What is Node.js?

Node.js is a JavaScript runtime that allows JavaScript to execute outside the browser and provides APIs for things such as networking, files, streams, and processes.

---

### 3. Is Node.js a programming language?

No.

JavaScript is the programming language.

Node.js is the runtime that executes JavaScript outside the browser.

---

### 4. Is Node.js a web server?

Not by itself.

Node.js provides APIs that allow you to create network and HTTP servers.

For example:

```js
http.createServer(...)
```

creates an HTTP server using Node's built-in HTTP module.

---

### 5. What is Express?

Express is a web framework for Node.js that provides abstractions for routing, middleware, request handling, and other common web application tasks.

---

### 6. What is middleware?

Middleware is code that runs during the request/response processing pipeline.

It can inspect or modify requests, perform authentication, logging, validation, and other tasks before passing control to the next middleware or route handler.

---

### 7. What does `next()` do in Express?

It passes control to the next middleware or matching handler in the pipeline.

---

### 8. Why is Node.js good for I/O-heavy applications?

Node's asynchronous, non-blocking I/O model allows the application to handle other work while waiting for operations such as network or database I/O to complete.

---

### 9. Is Node.js single-threaded?

JavaScript execution normally happens on a single main thread, but Node.js itself uses the operating system and mechanisms such as libuv's thread pool for certain asynchronous operations.

So saying "Node.js has only one thread" is an oversimplification.

---

### 10. What happens when a request reaches a Node.js server?

A simplified answer:

```text
Request arrives
     ↓
Node receives it
     ↓
Middleware runs
     ↓
Router finds handler
     ↓
Application logic executes
     ↓
Database/external services may be called
     ↓
Response is created
     ↓
Response is sent to client
```

---

# 44. Interview Scenario

Here's a more practical question.

> **Your Node.js API has 1,000 users making database requests at the same time. Does Node create 1,000 JavaScript threads?**

No.

Node's JavaScript execution normally happens on a single main thread.

It uses an event-driven, asynchronous model to coordinate many I/O operations.

So the application can have many requests **in progress concurrently** without needing one JavaScript thread per request.

But that doesn't mean all 1,000 database operations are magically free.

You can still run into:

```text
Database connection limits
CPU limits
Memory limits
Network limits
Connection pool limits
```

This is where backend engineering starts becoming interesting.

---

# 45. Another Interview Scenario

> **Your API usually responds in 50ms, but suddenly requests are taking 5 seconds. CPU is normal. What would you investigate?**

Don't immediately blame Node.

I'd start looking at:

```text
Database latency
External API latency
Network latency
Connection pool exhaustion
Slow queries
Lock contention
Queue delays
DNS issues
Cache misses
```

This is the kind of thinking you want to develop.

Backend engineering isn't just:

> "Which syntax do I use?"

It's:

> **"Where is the time going?"**

That mindset will matter much more as you become a better engineer.

---

# 46. Your Mini Challenge

Before moving to Chapter 4, build the Todo API yourself.

Don't copy the code above line by line.

Start with:

```text
GET /todos
POST /todos
```

Then add:

```text
GET /todos/:id
PATCH /todos/:id
DELETE /todos/:id
```

Your server should:

* return proper status codes
* return JSON
* validate basic input
* return `404` when a todo doesn't exist
* return `400` when the request is invalid

Keep the data in memory for now.

Don't add PostgreSQL yet.

We're intentionally building this application step by step.

---

# 47. What You Should Understand Before Moving On

You should now be able to explain the difference between:

```text
JavaScript
Node.js
Express
Web server
Backend application
Database
```

You should understand:

```text
Client
   ↓
HTTP request
   ↓
Node.js
   ↓
Middleware
   ↓
Route
   ↓
Application logic
   ↓
Database
   ↓
HTTP response
```

And you should have a basic understanding of:

```text
Request/response lifecycle
Middleware
Ports
Node.js runtime
Non-blocking I/O
Event loop
CPU-bound vs I/O-bound work
```

You don't need to master the event loop yet.

We'll revisit it when we start talking about performance and production.

---

# 48. Where We Go Next

We now know:

```text
How computers communicate
        ↓
How HTTP works
        ↓
How a server receives HTTP requests
```

But our URLs are still pretty basic.

We've written:

```text
GET /todos
POST /todos
```

What about:

```text
GET /todos/42
```

How do we get the `42`?

What about:

```text
GET /todos?completed=true
```

What is `completed=true`?

Why is this:

```text
/users/42
```

different from:

```text
/users?id=42
```

And how should we design URLs for something more complicated like:

```text
GET /users/42/orders/1001/items
```

This is where API design starts becoming important.

Next we'll learn:

# Chapter 4 — Routing & URL Design

We'll take what we've built so far and learn how to design APIs that are **predictable, readable, and actually pleasant to work with.**
