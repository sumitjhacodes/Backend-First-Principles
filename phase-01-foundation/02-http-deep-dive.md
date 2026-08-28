# Chapter 2 — HTTP Deep Dive

If you understood the previous chapter, you already know roughly this:

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

That's the basic communication pattern behind most web applications.

But now we need to slow down.

What exactly is inside an HTTP request?

Why do we have `GET`, `POST`, `PUT`, `PATCH`, and `DELETE`?

What are headers?

What's the difference between a request body and query parameters?

Why does an API return `200`, `201`, `400`, or `404`?

And what does it actually mean when someone says:

> "HTTP is stateless."

These aren't things you want to memorize just because they're asked in interviews.

You will use them **every single day** as a backend developer.

So let's understand HTTP properly.

---

# 1. What Is HTTP?

HTTP stands for:

> **HyperText Transfer Protocol**

The name sounds more complicated than it really is.

At its core:

> **HTTP is a set of rules that defines how a client and server communicate.**

A client sends a request.

A server sends a response.

For example:

```text
Client
  |
  | GET /users
  ↓
Server
  |
  | 200 OK
  | [users]
  ↓
Client
```

That's HTTP.

Of course, real requests contain much more information than just `GET /users`.

Let's look at one.

---

# 2. Anatomy of an HTTP Request

Here's a simplified HTTP request:

```http
POST /users HTTP/1.1
Host: api.example.com
Content-Type: application/json
Authorization: Bearer some-token

{
  "name": "Rahul",
  "email": "rahul@example.com"
}
```

There are four important parts here:

```text
Request
├── Method
├── URL
├── Headers
└── Body
```

Let's understand each one.

---

# 3. HTTP Method

The first thing in the request is:

```http
POST
```

This is the **HTTP method**.

It tells the server what kind of operation the client wants to perform.

Common HTTP methods are:

```text
GET
POST
PUT
PATCH
DELETE
```

You'll use these constantly when building REST APIs.

---

# 4. GET — "Give Me Something"

`GET` is generally used when the client wants to retrieve data.

For example:

```http
GET /users
```

Meaning:

> "Give me the users."

Or:

```http
GET /users/42
```

Meaning:

> "Give me user 42."

A typical API might respond:

```json
{
  "id": 42,
  "name": "Rahul",
  "email": "rahul@example.com"
}
```

A basic rule:

```text
GET → Read
```

---

## GET Usually Doesn't Have a Request Body

You might technically encounter GET requests with bodies in some systems, but don't build APIs around that.

For normal REST APIs, put filtering or identifying information in the URL:

```text
GET /users/42
```

or:

```text
GET /users?role=admin
```

We'll talk more about URLs in Chapter 4.

---

# 5. POST — "Create Something"

`POST` is commonly used to create a new resource or trigger an operation.

For example:

```http
POST /users
Content-Type: application/json

{
  "name": "Rahul",
  "email": "rahul@example.com"
}
```

The server might create a new user and respond:

```http
HTTP/1.1 201 Created
Content-Type: application/json

{
  "id": 42,
  "name": "Rahul",
  "email": "rahul@example.com"
}
```

The important part is:

```text
POST → Create / submit something
```

Don't make the mistake of thinking:

> POST always means database INSERT.

That's too simplistic.

POST can also be used for operations such as:

```text
POST /login
POST /payments
POST /orders
POST /users/42/reset-password
```

The method tells you the **general semantics of the request**, not exactly what your server implementation must do internally.

---

# 6. PUT — "Replace It"

This one causes confusion.

Suppose we have:

```text
User 42
```

with:

```json
{
  "name": "Rahul",
  "email": "rahul@example.com",
  "age": 25
}
```

A PUT request might be:

```http
PUT /users/42
```

with the complete representation:

```json
{
  "name": "Rahul",
  "email": "rahul@example.com",
  "age": 26
}
```

The general idea of PUT is:

> **Replace the resource representation with the one I'm sending.**

This is why PUT is often described as a **full update**.

---

# 7. PATCH — "Change Part of It"

Now suppose you only want to change the age.

You don't necessarily want to send the entire user again.

You can use:

```http
PATCH /users/42
Content-Type: application/json

{
  "age": 26
}
```

Meaning:

> "Change this part of the resource."

So the easy mental model is:

```text
PUT
→ Replace the resource

PATCH
→ Partially modify the resource
```

There are more precise semantic details, and real APIs don't always follow these conventions perfectly.

But this mental model is a good starting point.

---

# 8. DELETE — "Remove It"

Pretty straightforward:

```http
DELETE /users/42
```

Meaning:

> "Delete user 42."

The server might respond:

```http
204 No Content
```

or perhaps:

```http
200 OK
```

depending on the API design.

The common mental model:

```text
DELETE → Remove
```

---

# 9. The Five Methods Together

If you're building a simple users API:

```text
GET    /users
GET    /users/:id

POST   /users

PUT    /users/:id
PATCH  /users/:id

DELETE /users/:id
```

You can think:

```text
GET
↓
Read

POST
↓
Create / submit

PUT
↓
Replace

PATCH
↓
Partially update

DELETE
↓
Remove
```

Don't worry if this doesn't feel natural yet.

After building a few APIs, you'll stop thinking about it.

---

# 10. What Is a URL?

Look at this:

```text
https://api.example.com:3000/users/42?include=posts
```

A URL can contain several pieces.

A simplified breakdown:

```text
https://
   ↓
Scheme

api.example.com
   ↓
Host

:3000
   ↓
Port

/users/42
   ↓
Path

?include=posts
   ↓
Query string
```

We'll go deeper into URL design in Chapter 4.

For now, just understand that all of this information helps determine **where the request goes and what the client is asking for**.

---

# 11. What Are HTTP Headers?

Headers are metadata about the request or response.

Think of them as extra information attached to the message.

For example:

```http
Content-Type: application/json
Authorization: Bearer token
Accept: application/json
```

The actual request data might be:

```json
{
  "name": "Rahul"
}
```

And the headers tell the server things about that data.

For example:

```http
Content-Type: application/json
```

means:

> "The body I'm sending is JSON."

---

# 12. Important Request Headers

You don't need to memorize every HTTP header in existence.

There are hundreds.

Focus on the common ones.

### `Content-Type`

Tells the receiver what format the body is in.

Example:

```http
Content-Type: application/json
```

means:

> The body contains JSON.

Another example:

```http
Content-Type: application/x-www-form-urlencoded
```

is commonly used for form-encoded data.

---

### `Accept`

Tells the server what response formats the client prefers.

For example:

```http
Accept: application/json
```

means:

> "I'd like JSON back."

---

### `Authorization`

Used to send authentication credentials/token information.

For example:

```http
Authorization: Bearer eyJ...
```

We'll properly understand authentication later.

For now:

> Don't put passwords or secret tokens casually into URLs.

Use the appropriate secure mechanism and HTTPS.

---

### `User-Agent`

Tells the server something about the client software.

For example, a browser may send information identifying the browser.

---

### `Host`

Identifies the host being requested.

For example:

```http
Host: api.example.com
```

---

# 13. Request Body

The body contains data sent with the request.

For example:

```http
POST /users HTTP/1.1
Content-Type: application/json

{
  "name": "Rahul",
  "email": "rahul@example.com"
}
```

The JSON object is the request body.

You commonly see bodies with:

```text
POST
PUT
PATCH
```

For example, when creating a user:

```json
{
  "name": "Rahul",
  "email": "rahul@example.com"
}
```

The server reads that data and does something with it.

Maybe:

```text
Validate
   ↓
Business logic
   ↓
Database
   ↓
Response
```

---

# 14. Request vs Response

This distinction is extremely important.

### Request

Sent by the client.

```text
Client → Server
```

Contains things like:

```text
Method
URL
Headers
Body
```

### Response

Sent by the server.

```text
Server → Client
```

Contains things like:

```text
Status code
Headers
Body
```

So:

```text
             HTTP
              |
      ┌───────┴───────┐
      ↓               ↓
   REQUEST          RESPONSE
      ↓               ↓
   Client            Server
      ↓               ↓
   Server            Client
```

---

# 15. HTTP Status Codes

Now we get to one of the most recognizable parts of HTTP.

You make a request:

```http
GET /users/42
```

and the server responds:

```http
HTTP/1.1 200 OK
```

What's `200`?

It's a **status code**.

Status codes tell the client what happened with the request.

They are grouped into categories.

```text
1xx → Informational
2xx → Success
3xx → Redirection
4xx → Client error
5xx → Server error
```

You don't need to memorize every status code.

Learn the important ones first.

---

# 16. `200 OK`

This generally means:

> The request succeeded.

Example:

```http
GET /users/42

200 OK
```

Response:

```json
{
  "id": 42,
  "name": "Rahul"
}
```

---

# 17. `201 Created`

Used when a request successfully creates something.

For example:

```http
POST /users
```

Response:

```http
201 Created
```

This is often better than returning `200` when a new resource was created.

---

# 18. `204 No Content`

The operation succeeded, but there is no response body.

Common example:

```http
DELETE /users/42
```

Response:

```http
204 No Content
```

No JSON is necessary.

---

# 19. `400 Bad Request`

This generally means:

> The server couldn't process the request because the request was invalid.

For example, your API requires:

```json
{
  "email": "..."
}
```

but the client sends:

```json
{
  "name": "Rahul"
}
```

The server might respond:

```http
400 Bad Request
```

with:

```json
{
  "error": "email is required"
}
```

The exact choice between `400` and more specific client-error statuses depends on what went wrong and your API design.

---

# 20. `401 Unauthorized`

This one is commonly misunderstood.

`401` generally means:

> **Authentication is required or the provided authentication credentials aren't valid.**

For example:

```text
GET /profile
```

without a valid access token.

Response:

```http
401 Unauthorized
```

Think:

> "I don't know who you are / your authentication isn't valid."

---

# 21. `403 Forbidden`

Now imagine the user is authenticated.

The server knows who they are.

But they don't have permission.

For example:

```text
Normal user
   ↓
DELETE /admin/users/42
```

The server might return:

```http
403 Forbidden
```

Think:

```text
401 → Authentication problem
403 → Permission problem
```

A useful interview distinction.

---

# 22. `404 Not Found`

This usually means the requested resource wasn't found.

For example:

```http
GET /users/999999
```

If user `999999` doesn't exist:

```http
404 Not Found
```

It can also be used when a route/resource isn't available.

---

# 23. `409 Conflict`

This indicates a conflict with the current state of the resource.

A common example:

```text
POST /users
```

with an email that already exists.

The API might return:

```http
409 Conflict
```

For example:

```json
{
  "error": "email already exists"
}
```

Again, exact API conventions vary, but `409` is useful when there's a state conflict rather than simply malformed input.

---

# 24. `429 Too Many Requests`

This one becomes important when we learn rate limiting.

It means:

> The client has sent too many requests in a given period.

For example:

```text
100 requests/minute allowed
        ↓
Client sends 500
        ↓
429 Too Many Requests
```

We'll build this later.

---

# 25. `500 Internal Server Error`

This generally means:

> Something went wrong on the server.

For example:

```text
Request
   ↓
Backend
   ↓
Unexpected exception
   ↓
500
```

A common beginner mistake is returning `500` for everything.

Don't do this.

If the user sends invalid input, that's usually not a server failure.

For example:

```text
Missing email
```

shouldn't suddenly become:

```text
500 Internal Server Error
```

because the server is working correctly; the input is the problem.

---

# 26. Status Code Cheat Sheet

Keep this one nearby:

```text
200 → Success

201 → Created

204 → Success, no response body

400 → Bad request / invalid input

401 → Authentication required/failed

403 → Authenticated but not allowed

404 → Resource not found

409 → Conflict

429 → Too many requests

500 → Server-side failure
```

You don't need to know every status code before you start building.

Learn these first.

---

# 27. HTTP Is Stateless

This is one of those interview questions you'll almost definitely encounter.

> **"What does it mean that HTTP is stateless?"**

Simple answer:

> Each HTTP request is independent. HTTP itself does not automatically remember previous requests.

Imagine:

```text
Request 1
GET /profile
```

Then:

```text
Request 2
GET /orders
```

The HTTP protocol doesn't inherently say:

> "Oh, this is the same person who made Request 1."

If the server needs to know who you are, your application needs to provide some mechanism for that.

For example:

```text
Cookie
Session ID
Access token
```

This is why authentication exists.

---

# 28. "But Websites Remember Me"

Exactly.

This is where beginners get confused.

They hear:

> "HTTP is stateless."

Then they think:

> "But I log into websites and they remember me."

The distinction is:

```text
HTTP itself
↓
Stateless
```

But applications can build stateful behavior using mechanisms such as:

```text
Cookies
Sessions
Tokens
Databases
Caches
```

For example:

```text
Login
  ↓
Server creates session
  ↓
Browser stores session cookie
  ↓
Next request sends cookie
  ↓
Server identifies session
  ↓
Server knows the user
```

We'll explore this properly in the authentication chapter.

---

# 29. What Does Idempotent Mean?

Another interview favorite.

The word looks scary.

The idea isn't.

A request is **idempotent** if making the same request multiple times has the same intended effect on the server's resource state as making it once.

For example:

```http
PUT /users/42

{
  "name": "Rahul"
}
```

If you send it once:

```text
name = Rahul
```

Send it again:

```text
name = Rahul
```

Send it again:

```text
name = Rahul
```

The intended final state is the same.

That's the idea of idempotency.

---

# 30. What About POST?

Imagine:

```http
POST /orders
```

You send it once:

```text
Order #1001 created
```

Send it again:

```text
Order #1002 created
```

You may now have two orders.

So POST is generally **not idempotent**.

This matters a lot in real systems.

Imagine a payment request.

What happens if the client sends:

```text
POST /payments
```

and the network times out?

The client doesn't know whether the payment was processed.

If it blindly retries, you might charge the customer twice.

This is where **idempotency keys** become extremely useful.

We'll come back to this in the real-world APIs chapter.

---

# 31. Safe vs Idempotent

Don't mix these two concepts.

### Safe

A method is safe if it is intended not to modify server state.

For example:

```text
GET
```

### Idempotent

Repeated identical requests have the same intended effect as a single request.

For example:

```text
GET
PUT
DELETE
```

are defined as idempotent methods in HTTP semantics, while `POST` generally isn't.

Important detail:

> Idempotent doesn't mean the response must be identical every time.

It means the **intended effect on server state** is the same.

---

# 32. Let's Actually Build Something

Enough theory.

Let's create a tiny HTTP server.

We'll use Node.js because that's what we'll use throughout this backend series.

Create a folder:

```bash
mkdir http-demo
cd http-demo
```

Initialize a Node project:

```bash
npm init -y
```

Create:

```text
server.js
```

Add:

```js
const http = require("http");

const server = http.createServer((req, res) => {
  console.log(req.method, req.url);

  res.writeHead(200, {
    "Content-Type": "application/json",
  });

  res.end(
    JSON.stringify({
      message: "Hello from my server",
    })
  );
});

server.listen(3000, () => {
  console.log("Server running on http://localhost:3000");
});
```

Run:

```bash
node server.js
```

Now open:

```text
http://localhost:3000
```

You should receive:

```json
{
  "message": "Hello from my server"
}
```

You just created a server that understands HTTP requests.

---

# 33. Let's See the Request

Add this:

```js
console.log("Method:", req.method);
console.log("URL:", req.url);
console.log("Headers:", req.headers);
```

Now visit:

```text
http://localhost:3000/users
```

Your terminal should show something similar to:

```text
Method: GET
URL: /users
Headers: {
  host: 'localhost:3000',
  connection: 'keep-alive',
  ...
}
```

Now you can actually **see** the concepts we've been discussing.

---

# 34. Let's Handle Different Methods

We can do:

```js
const http = require("http");

const server = http.createServer((req, res) => {
  res.setHeader("Content-Type", "application/json");

  if (req.method === "GET" && req.url === "/users") {
    res.writeHead(200);

    res.end(
      JSON.stringify({
        users: [],
      })
    );

    return;
  }

  if (req.method === "POST" && req.url === "/users") {
    res.writeHead(201);

    res.end(
      JSON.stringify({
        message: "User created",
      })
    );

    return;
  }

  res.writeHead(404);

  res.end(
    JSON.stringify({
      error: "Route not found",
    })
  );
});

server.listen(3000, () => {
  console.log("Server running on http://localhost:3000");
});
```

Now:

```text
GET /users
```

returns:

```json
{
  "users": []
}
```

And:

```text
POST /users
```

returns:

```json
{
  "message": "User created"
}
```

This is already the beginning of an API.

Later, Express or Fastify will make this much cleaner.

But I actually recommend understanding the raw Node HTTP API at least once.

Otherwise frameworks can hide too much from you.

---

# 35. Test It With curl

You can test the GET endpoint:

```bash
curl http://localhost:3000/users
```

For POST:

```bash
curl -X POST http://localhost:3000/users
```

You can also inspect response headers:

```bash
curl -i http://localhost:3000/users
```

You might see:

```http
HTTP/1.1 200 OK
Content-Type: application/json
Date: ...
Connection: keep-alive
...
```

Now you're not just reading about HTTP.

You're actually sending HTTP requests.

---

# 36. What Happens Inside Your Server?

When you run:

```bash
node server.js
```

your application starts listening on:

```text
localhost:3000
```

Then you run:

```bash
curl http://localhost:3000/users
```

The flow is roughly:

```text
curl
 ↓
HTTP GET /users
 ↓
localhost:3000
 ↓
Node.js HTTP server
 ↓
Your callback executes
 ↓
req.method === "GET"
req.url === "/users"
 ↓
Create response
 ↓
200 OK
 ↓
JSON body
 ↓
curl receives response
```

This is the request/response cycle you'll spend a huge part of your backend career working with.

---

# 37. What About Request Bodies?

Now let's send data.

Suppose we want:

```http
POST /users
```

with:

```json
{
  "name": "Rahul"
}
```

With curl:

```bash
curl -X POST http://localhost:3000/users \
  -H "Content-Type: application/json" \
  -d '{"name":"Rahul"}'
```

Now there's something important happening.

The request body doesn't magically appear as a JavaScript object.

The server receives the incoming data as a stream of bytes/chunks.

With Node's HTTP API, you need to collect those chunks and then parse the JSON.

For example:

```js
const http = require("http");

const server = http.createServer((req, res) => {
  if (req.method === "POST" && req.url === "/users") {
    let body = "";

    req.on("data", (chunk) => {
      body += chunk;
    });

    req.on("end", () => {
      try {
        const data = JSON.parse(body);

        console.log(data);

        res.writeHead(201, {
          "Content-Type": "application/json",
        });

        res.end(
          JSON.stringify({
            message: "User created",
            user: data,
          })
        );
      } catch {
        res.writeHead(400, {
          "Content-Type": "application/json",
        });

        res.end(
          JSON.stringify({
            error: "Invalid JSON",
          })
        );
      }
    });

    return;
  }

  res.writeHead(404);

  res.end(
    JSON.stringify({
      error: "Not found",
    })
  );
});

server.listen(3000);
```

This is one reason frameworks are useful.

They give us cleaner abstractions over this low-level work.

---

# 38. What Frameworks Actually Give You

Later we'll use Express/Fastify.

You might write:

```js
app.post("/users", (req, res) => {
  const user = req.body;

  // business logic
});
```

Instead of manually handling:

```text
data chunks
JSON parsing
routing
headers
responses
```

Frameworks handle a lot of repetitive HTTP plumbing.

But now you know what they're hiding.

That's useful.

When something breaks, you're less likely to think:

> "Express is magic."

You'll know there's an HTTP request underneath it.

---

# 39. Common HTTP Mistakes

### Mistake 1: Using `POST` for everything

I've seen APIs like:

```text
POST /getUsers
POST /deleteUser
POST /updateUser
POST /createUser
```

Technically you can build an API this way.

But for a REST-style API, use HTTP methods meaningfully.

For example:

```text
GET    /users
POST   /users
PATCH  /users/:id
DELETE /users/:id
```

The URL identifies the resource.

The HTTP method describes the operation.

---

### Mistake 2: Returning `200` for every situation

Don't do:

```text
User doesn't exist → 200
Invalid input      → 200
Server crashed     → 200
Everything worked  → 200
```

Status codes communicate useful information to clients.

Use them intentionally.

---

### Mistake 3: Returning `500` for validation errors

This:

```text
Email is required
```

is generally a client/input problem.

It isn't automatically a server failure.

---

### Mistake 4: Confusing `401` and `403`

Remember:

```text
401
→ Authentication problem

403
→ Permission problem
```

---

### Mistake 5: Thinking PUT and PATCH are identical

They're related but have different intended semantics.

```text
PUT
→ Replace

PATCH
→ Partial modification
```

---

### Mistake 6: Putting sensitive information in URLs

Avoid things like:

```text
/login?password=123456
```

URLs can appear in browser history, logs, monitoring systems, proxies, and other places.

Use HTTPS and appropriate request mechanisms for sensitive data.

---

# 40. A Real Backend Example

Imagine you're building a shopping application.

You might have:

```text
GET    /products
GET    /products/:id

POST   /products

PATCH  /products/:id

DELETE /products/:id
```

Now imagine:

```text
POST /orders
```

The backend might:

```text
Receive request
      ↓
Authenticate user
      ↓
Validate input
      ↓
Check product availability
      ↓
Create order
      ↓
Save to database
      ↓
Return response
```

HTTP is only the communication layer.

Your actual backend logic sits behind it.

That's an important distinction.

---

# 41. HTTP Doesn't Tell You Your Business Logic

Suppose you send:

```http
POST /orders
```

HTTP doesn't know what an order is.

It doesn't know:

```text
inventory
payment
discount
shipping
database
```

Those are application-level concepts.

HTTP simply gives your application a standardized way to communicate.

Your backend decides what to do with the request.

---

# 42. A Better Mental Model

Think of an HTTP request as a package arriving at your backend.

It contains:

```text
┌──────────────────────────────┐
│ METHOD                       │
│ GET / POST / PATCH / DELETE  │
├──────────────────────────────┤
│ URL                          │
│ /users/42                    │
├──────────────────────────────┤
│ HEADERS                      │
│ Content-Type                 │
│ Authorization                │
│ Accept                       │
├──────────────────────────────┤
│ BODY                         │
│ {                            │
│   "name": "Rahul"            │
│ }                            │
└──────────────────────────────┘
```

Your backend receives it.

Then:

```text
Validate
   ↓
Authenticate
   ↓
Authorize
   ↓
Business Logic
   ↓
Database / External Services
   ↓
Create Response
```

Then sends:

```text
┌──────────────────────────────┐
│ STATUS                       │
│ 200 OK                       │
├──────────────────────────────┤
│ HEADERS                      │
│ Content-Type: application/json
├──────────────────────────────┤
│ BODY                         │
│ {                            │
│   "name": "Rahul"            │
│ }                            │
└──────────────────────────────┘
```

That is a backend engineer's everyday world.

---

# 43. Interview Questions

Don't just memorize the answers. Try answering them yourself first.

---

### 1. What is HTTP?

HTTP is an application-layer protocol that defines how clients and servers communicate using requests and responses.

---

### 2. What is the difference between GET and POST?

GET is generally used to retrieve data, while POST is generally used to submit data or create a resource.

---

### 3. PUT vs PATCH?

PUT is generally used to replace a resource representation, while PATCH is used for partial modifications.

---

### 4. What is an HTTP header?

A header is metadata attached to an HTTP request or response.

Examples include:

```text
Content-Type
Authorization
Accept
Cache-Control
```

---

### 5. What is a request body?

The body contains data sent as part of the request.

For example:

```json
{
  "name": "Rahul"
}
```

---

### 6. What does stateless mean in HTTP?

HTTP itself doesn't maintain client state between requests. Each request is treated independently unless the application uses mechanisms such as cookies, sessions, or tokens to associate requests with a client.

---

### 7. What is idempotency?

An operation is idempotent when repeating the same request has the same intended effect on server state as performing it once.

---

### 8. What is the difference between 401 and 403?

`401` indicates that authentication is missing or invalid.

`403` indicates that the client is authenticated but isn't allowed to perform the operation.

---

### 9. When would you return 201?

When a request successfully creates a new resource.

---

### 10. When would you return 204?

When an operation succeeds and there's intentionally no response body.

---

### 11. Why shouldn't you return 500 for invalid input?

Because `500` indicates a server-side failure. Invalid user input is generally a client/request problem and should normally result in an appropriate `4xx` response.

---

### 12. Is POST always used for creating data?

No.

POST is commonly used for creation, but it can also represent other operations where the client submits data to a resource or asks the server to perform an operation.

---

# 44. Interview Scenario

Here's a more realistic interview question.

> **Your frontend sends `POST /payments`. The server successfully charges the customer's card, but the response times out. The frontend retries the request. What problem can happen?**

Think about it.

The first request may have succeeded:

```text
Frontend
   ↓
POST /payments
   ↓
Payment server
   ↓
Charge succeeds
   ↓
Response gets lost
```

Frontend thinks:

```text
"Request failed."
```

So it retries:

```text
POST /payments
   ↓
Charge again
```

Now the customer could be charged twice.

This is why backend engineers care about:

> **Idempotency**

A payment API might accept an idempotency key:

```http
POST /payments
Idempotency-Key: order-123-payment
```

The server can recognize that the same logical payment request was already processed.

This is the kind of problem that HTTP concepts eventually lead us toward.

---

# 45. Mini Project — Build a Todo HTTP API

Now it's your turn.

Don't use Express yet.

Use Node's built-in HTTP module.

Build:

```text
GET    /todos
GET    /todos/:id
POST   /todos
PATCH  /todos/:id
DELETE /todos/:id
```

For now, keep the data in memory:

```js
const todos = [
  {
    id: 1,
    title: "Learn HTTP",
    completed: false
  }
];
```

Your API should support:

### Get all todos

```http
GET /todos
```

Response:

```json
[
  {
    "id": 1,
    "title": "Learn HTTP",
    "completed": false
  }
]
```

### Create a todo

```http
POST /todos
```

Body:

```json
{
  "title": "Build my API"
}
```

Return:

```http
201 Created
```

### Update a todo

```http
PATCH /todos/1
```

Body:

```json
{
  "completed": true
}
```

### Delete a todo

```http
DELETE /todos/1
```

Return:

```http
204 No Content
```

### Unknown route

Return:

```http
404 Not Found
```

---

# 46. Don't Copy the Solution Immediately

This is important.

Try building it yourself first.

You'll probably get stuck.

That's fine.

Actually, getting stuck here is useful.

You'll start asking questions like:

> How do I get the ID from `/todos/42`?

> How do I read the JSON body?

> What happens if the JSON is invalid?

> What if the todo doesn't exist?

> Should that be `400` or `404`?

> What if someone sends `PATCH /todos/999`?

Those questions are exactly what we want.

Because backend engineering isn't:

```text
memorize syntax
      ↓
write code
```

It's more like:

```text
Understand the request
        ↓
Understand possible states
        ↓
Handle valid cases
        ↓
Handle invalid cases
        ↓
Handle failures
        ↓
Return useful responses
```

---

# 47. What You Should Be Able to Explain Now

Before moving forward, you should be comfortable explaining:

```text
HTTP
 ↓
Request
 ↓
Method
 ↓
URL
 ↓
Headers
 ↓
Body
 ↓
Server processing
 ↓
Response
 ↓
Status code
 ↓
Headers
 ↓
Body
```

You should know the purpose of:

```text
GET
POST
PUT
PATCH
DELETE
```

And the important status codes:

```text
200
201
204
400
401
403
404
409
429
500
```

You should understand:

```text
Statelessness
Idempotency
Request vs response
Headers vs body
Authentication vs authorization
```

You don't need to know every HTTP specification detail.

If you understand these concepts and can use them while building an API, you're in a good place.

---

# 48. The Bigger Picture

We're slowly building your backend mental model.

Chapter 1:

```text
How computers communicate
```

Chapter 2:

```text
How applications communicate over HTTP
```

Next:

```text
How does a program actually receive
and handle those HTTP requests?
```

That's where Node.js comes in.

We'll look at:

```text
Web server
Node.js
Request/response lifecycle
Event loop
Blocking vs non-blocking work
Middleware
Express/Fastify
```

And we'll finally answer a question you've probably seen many times:

> **"What actually happens inside a Node.js backend when a request arrives?"**

That's **Chapter 3 — What Is a Web Server?**

And that's where we'll start turning these concepts into a real backend application.
