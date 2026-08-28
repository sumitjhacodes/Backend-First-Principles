# Chapter 6 — Postman, cURL & API Testing

We've built a small Todo API.

We know:

* what HTTP is
* how requests and responses work
* how routing works
* how JSON works
* how a server handles a request

But there's a problem.

How do we know our API actually works?

We could build a frontend for it.

But we don't need to.

As backend developers, we should be able to test an API **without depending on the frontend**.

That's where tools like **Postman** and **cURL** come in.

And more importantly, this chapter is not really about learning Postman.

It's about learning how to **think when testing an API**.

---

# 1. What Is API Testing?

Let's say we have:

```text
POST /todos
```

We expect it to:

1. receive a todo
2. validate the input
3. create the todo
4. return `201 Created`

API testing means checking whether the API actually behaves that way.

For example:

```text
Send request
     ↓
Server processes request
     ↓
Check response
     ↓
Did we get what we expected?
```

We're checking things like:

```text
Status code
Response body
Headers
Response time
Error handling
Validation
Authentication
Edge cases
```

---

# 2. Why Backend Developers Need This

Imagine you're building:

```text
POST /users
```

You don't need a React frontend just to test:

```json
{
  "name": "Sumit",
  "email": "sumit@example.com"
}
```

You can send that request directly to the backend.

This is useful because it separates two problems:

```text
Frontend problem
        ↓
Is the UI working?

Backend problem
        ↓
Is the API working?
```

When you're developing the backend, you want to test the backend independently.

That's why Postman and cURL are so useful.

---

# 3. What Is Postman?

Postman is a tool for making HTTP requests.

You can create:

```text
GET
POST
PUT
PATCH
DELETE
```

requests and see the response.

For example:

```text
POST http://localhost:3000/todos
```

with:

```json
{
  "title": "Learn backend"
}
```

Postman sends the request to your server.

Your server responds.

Postman shows you the result.

That's it.

It's basically a convenient UI for talking to APIs.

---

# 4. What Is cURL?

cURL is a command-line tool for making network requests.

Instead of clicking buttons in Postman, you can write:

```bash
curl http://localhost:3000/todos
```

That's it.

Your terminal sends the HTTP request.

The server responds.

You'll see the response directly in the terminal.

---

# 5. Postman vs cURL

Don't think:

> "Which one should I learn?"

Learn both.

### Postman

Good when:

* you're exploring an API
* you want a visual interface
* you have lots of requests
* you want saved collections
* you're debugging manually

### cURL

Good when:

* you're working in the terminal
* you're on a server
* you're debugging quickly
* you're writing shell scripts
* you're working with CI/CD
* you want to reproduce a request exactly

A senior engineer should be comfortable with both.

---

# 6. Our Todo API

Let's assume our server is running:

```bash
npm run dev
```

and listening on:

```text
http://localhost:3000
```

Our API:

```text
GET    /todos
POST   /todos
GET    /todos/:id
PATCH  /todos/:id
DELETE /todos/:id
```

Now let's test each one.

---

# 7. First Request — GET /todos

Open Postman.

Create a new request.

Method:

```text
GET
```

URL:

```text
http://localhost:3000/todos
```

Click:

```text
Send
```

You might receive:

```json
[]
```

That means:

> The request worked and there are currently no todos.

If you already have todos:

```json
[
  {
    "id": 1,
    "title": "Learn backend",
    "completed": false
  }
]
```

---

# 8. Test the Same Request with cURL

Open your terminal:

```bash
curl http://localhost:3000/todos
```

You might get:

```json
[]
```

That's your API response.

Notice something important.

The frontend isn't involved.

We're directly communicating with the backend.

---

# 9. POST /todos

Now let's create a Todo.

In Postman:

```text
Method:
POST

URL:
http://localhost:3000/todos
```

Go to:

```text
Body
→ raw
→ JSON
```

Send:

```json
{
  "title": "Learn backend"
}
```

Postman should send something like:

```http
POST /todos HTTP/1.1
Content-Type: application/json

{
  "title": "Learn backend"
}
```

If everything works, we expect:

```text
201 Created
```

and something like:

```json
{
  "id": 1,
  "title": "Learn backend",
  "completed": false
}
```

---

# 10. The cURL Version

The same request with cURL:

```bash
curl -X POST http://localhost:3000/todos \
  -H "Content-Type: application/json" \
  -d '{"title":"Learn backend"}'
```

Let's break this down.

### `curl`

Run cURL.

### `-X POST`

Use the POST method.

### `-H`

Add a request header.

We're adding:

```text
Content-Type: application/json
```

### `-d`

Send request data.

Our data is:

```json
{
  "title": "Learn backend"
}
```

So the entire command means:

> Send a POST request containing JSON to `/todos`.

---

# 11. Why Learn the Command Instead of Copy-Pasting It?

Because eventually you'll see something like this in production:

```text
"Can you reproduce this API request?"
```

Someone might give you:

```bash
curl -X POST \
  https://api.example.com/users \
  -H "Authorization: Bearer ..." \
  -H "Content-Type: application/json" \
  -d '{"name":"John"}'
```

If you understand cURL, you immediately understand:

```text
method
URL
headers
body
```

You don't need Postman to understand what's happening.

---

# 12. Path Parameters

Remember:

```text
GET /todos/:id
```

Suppose todo ID is:

```text
1
```

The actual URL becomes:

```text
GET /todos/1
```

In Postman:

```text
GET http://localhost:3000/todos/1
```

With cURL:

```bash
curl http://localhost:3000/todos/1
```

If it exists:

```json
{
  "id": 1,
  "title": "Learn backend",
  "completed": false
}
```

If it doesn't:

```text
404 Not Found
```

and perhaps:

```json
{
  "error": {
    "code": "TODO_NOT_FOUND",
    "message": "Todo not found"
  }
}
```

---

# 13. Query Parameters

Suppose later we add:

```text
GET /todos?completed=true
```

The `completed=true` part is a query parameter.

In Postman, you can put:

```text
Key: completed
Value: true
```

under the Params section.

The resulting URL:

```text
/todos?completed=true
```

With cURL:

```bash
curl "http://localhost:3000/todos?completed=true"
```

Notice the quotes.

They're useful when your shell might interpret special characters.

---

# 14. Headers

HTTP headers carry additional information about the request or response.

For example:

```http
Content-Type: application/json
```

Another common header:

```http
Authorization: Bearer <token>
```

We'll use that heavily when we reach authentication.

In Postman:

```text
Headers
```

lets you add them manually.

With cURL:

```bash
curl http://localhost:3000/users \
  -H "Authorization: Bearer my-token"
```

The important thing is not memorizing Postman's buttons.

Understand what is actually being sent over HTTP.

---

# 15. Status Codes

When testing APIs, don't only look at the response body.

Look at the status code.

For example:

```text
200 OK
```

means the request succeeded.

```text
201 Created
```

usually means something was successfully created.

```text
400 Bad Request
```

means the request is invalid.

```text
401 Unauthorized
```

usually means authentication is missing or invalid.

```text
403 Forbidden
```

means the client is authenticated but doesn't have permission.

```text
404 Not Found
```

means the requested resource doesn't exist.

```text
500 Internal Server Error
```

means something went wrong on the server.

You learned these in Chapter 2.

Now we're actually using them.

---

# 16. Don't Test Only the Happy Path

This is one of the biggest lessons of this chapter.

A beginner often tests:

```text
Valid request
     ↓
200/201
     ↓
"It works!"
```

That's not enough.

A real backend developer asks:

> What happens when things go wrong?

For our Todo API, test:

```text
Valid todo
Missing title
Empty title
Wrong data type
Missing body
Invalid JSON
Non-existing ID
Wrong HTTP method
Extra fields
Very long title
Malformed URL
```

That's API testing.

---

# 17. Test Case #1 — Valid Request

Send:

```json
{
  "title": "Learn backend"
}
```

Expected:

```text
201 Created
```

Response:

```json
{
  "id": 1,
  "title": "Learn backend",
  "completed": false
}
```

Good.

---

# 18. Test Case #2 — Missing Field

Send:

```json
{}
```

Your API should reject it.

Expected:

```text
400 Bad Request
```

Maybe:

```json
{
  "error": {
    "code": "INVALID_TITLE",
    "message": "Title must be a non-empty string"
  }
}
```

This tests validation.

---

# 19. Test Case #3 — Wrong Data Type

Send:

```json
{
  "title": 123
}
```

The JSON is valid.

But your application doesn't accept it.

Expected:

```text
400 Bad Request
```

This reminds us of the distinction from Chapter 5:

```text
Valid JSON
       ≠
Valid application input
```

---

# 20. Test Case #4 — Empty String

Send:

```json
{
  "title": ""
}
```

Should this be allowed?

Probably not.

Your API might reject it:

```text
400 Bad Request
```

This is where you start thinking like an engineer.

Don't only ask:

> "Does my code work?"

Ask:

> "What input should my system accept?"

---

# 21. Test Case #5 — Todo Doesn't Exist

Request:

```text
GET /todos/999999
```

If the todo doesn't exist:

```text
404 Not Found
```

Not:

```text
200 OK
```

with:

```json
{}
```

A clear API should communicate that the resource wasn't found.

---

# 22. Test Case #6 — Wrong Method

Try:

```text
PUT /todos
```

if your API doesn't support it.

What happens?

Maybe:

```text
404
```

or:

```text
405 Method Not Allowed
```

depending on how your application is implemented.

The important thing is to understand that HTTP methods are part of your API contract.

---

# 23. Test Case #7 — Invalid JSON

Send something broken:

```json
{
  "title": "Learn backend"
```

Missing:

```text
}
```

The JSON parser should reject it.

Your server shouldn't crash.

This is a useful production mindset:

> External input can always be malformed.

---

# 24. Test Case #8 — Huge Input

Try sending a very large title.

For example, thousands or millions of characters.

You don't need to manually type it.

The point is to think:

> What happens if someone sends much more data than expected?

This leads to concepts we'll learn later:

```text
Request body limits
Rate limiting
Input validation
Resource exhaustion
```

Don't implement everything now.

Just start noticing these problems.

---

# 25. API Testing Is About Contracts

Think of an API endpoint as having a contract.

For:

```text
POST /todos
```

the contract might be:

### Request

```json
{
  "title": "Learn backend"
}
```

### Requirements

```text
title must exist
title must be a string
title cannot be empty
```

### Success

```text
201 Created
```

### Response

```json
{
  "id": 1,
  "title": "Learn backend",
  "completed": false
}
```

### Errors

```text
400 → invalid input
404 → resource not found
500 → unexpected server error
```

When you test an API, you're checking whether the implementation follows its contract.

---

# 26. Postman Collections

Imagine your project has:

```text
50 API endpoints
```

You don't want to recreate each request every time.

Postman lets you organize requests into **collections**.

For example:

```text
Todo API
│
├── Todos
│   ├── Get all todos
│   ├── Get todo
│   ├── Create todo
│   ├── Update todo
│   └── Delete todo
│
└── Auth
    ├── Register
    ├── Login
    └── Logout
```

Now your API requests are saved.

This becomes very useful as your application grows.

---

# 27. Environment Variables

Suppose you're developing locally.

Your API:

```text
http://localhost:3000
```

But production is:

```text
https://api.example.com
```

You don't want to manually change every request.

Instead, you can create an environment variable such as:

```text
BASE_URL
```

Development:

```text
BASE_URL=http://localhost:3000
```

Production:

```text
BASE_URL=https://api.example.com
```

Then your request can use:

```text
{{BASE_URL}}/todos
```

This is one reason Postman becomes useful for real projects.

---

# 28. Don't Put Secrets in Postman Collections

This is important.

Suppose you have:

```text
Authorization: Bearer abc123...
```

Don't casually commit a real production token into Git.

The same rule applies to:

```text
Database passwords
API keys
JWT secrets
Cloud credentials
Private tokens
```

We'll cover proper environment and secret management later.

For now:

> Treat credentials as secrets, even during testing.

---

# 29. cURL Is Also Great for Debugging

Suppose someone reports:

> "The API isn't working."

You can reproduce the request manually.

For example:

```bash
curl -i http://localhost:3000/todos
```

The `-i` option includes response headers.

Now you might see:

```text
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 2

[]
```

Now you're seeing more than just:

```json
[]
```

You're seeing the HTTP response itself.

That's useful for debugging.

---

# 30. Useful cURL Options

You don't need to memorize hundreds of options.

Start with these.

### GET

```bash
curl http://localhost:3000/todos
```

### POST

```bash
curl -X POST http://localhost:3000/todos \
  -H "Content-Type: application/json" \
  -d '{"title":"Learn backend"}'
```

### Include headers

```bash
curl -i http://localhost:3000/todos
```

### Verbose mode

```bash
curl -v http://localhost:3000/todos
```

`-v` can show detailed information about the request and connection.

This becomes extremely useful when debugging networking problems.

---

# 31. What Does `-v` Actually Help With?

Imagine:

```bash
curl -v http://localhost:3000/todos
```

You may see information about:

```text
DNS
connection
request headers
response headers
HTTP version
TLS
```

For a local HTTP server, you'll see the connection and HTTP exchange.

When something isn't working, this can answer questions like:

> Did I actually connect to the server?

> What headers did I send?

> What status code did I receive?

That's much better than randomly changing code.

---

# 32. A Better Way to Debug APIs

Suppose:

```text
POST /todos
```

is returning:

```text
400 Bad Request
```

Don't immediately edit the backend.

Walk through the request.

### Step 1 — Check URL

```text
POST /todos
```

Correct?

### Step 2 — Check method

```text
POST
```

Correct?

### Step 3 — Check headers

```http
Content-Type: application/json
```

Present?

### Step 4 — Check body

```json
{
  "title": "Learn backend"
}
```

Valid JSON?

### Step 5 — Check server logs

Did the request reach the server?

### Step 6 — Check validation

Did your validation reject it?

This process is much more valuable than memorizing fixes.

---

# 33. Manual Testing vs Automated Testing

There's another important distinction.

Postman and cURL are great for **manual API testing**.

But imagine having:

```text
100 endpoints
```

and testing all of them manually after every code change.

That's painful.

That's where automated tests come in.

For example:

```text
Run tests
     ↓
POST /todos
     ↓
Check response
     ↓
GET /todos
     ↓
Check response
     ↓
DELETE /todos/1
     ↓
Check response
```

The computer does it for you.

We'll properly cover automated backend testing later.

For now, understand:

```text
Manual testing
→ human sends requests

Automated testing
→ code sends requests and checks results
```

---

# 34. Unit vs Integration Testing

You've probably heard these terms.

Don't worry about mastering them yet.

### Unit test

Tests a small piece of code in isolation.

For example:

```js
function calculateTotal(price, quantity) {
  return price * quantity;
}
```

You test:

```text
calculateTotal(100, 2)
→ 200
```

### Integration test

Tests multiple pieces working together.

For example:

```text
HTTP request
   ↓
Route
   ↓
Controller
   ↓
Database
   ↓
Response
```

We'll go deeper into this in Chapter 18.

For now, remember:

```text
Unit
→ small piece

Integration
→ multiple pieces working together
```

---

# 35. Build an API Test Checklist

For every endpoint you create, ask these questions.

### Request

```text
✓ Is the method correct?
✓ Is the URL correct?
✓ Are required headers present?
✓ Is the body valid?
```

### Success

```text
✓ Is the status code correct?
✓ Is the response body correct?
✓ Are the response headers correct?
```

### Failure

```text
✓ Missing input?
✓ Wrong input type?
✓ Invalid ID?
✓ Unauthorized request?
✓ Resource doesn't exist?
✓ Malformed request?
```

### Edge cases

```text
✓ Empty values?
✓ Very large values?
✓ Duplicate requests?
✓ Unexpected fields?
✓ Boundary values?
```

This checklist will save you a lot of time.

---

# 36. Let's Test Our Todo API Properly

Here's your first real testing exercise.

Start your server:

```bash
npm run dev
```

Then test these.

| # | Request                            | Expected          |
| - | ---------------------------------- | ----------------- |
| 1 | `GET /todos`                       | `200`             |
| 2 | `POST /todos` with valid title     | `201`             |
| 3 | `POST /todos` without title        | `400`             |
| 4 | `POST /todos` with number as title | `400`             |
| 5 | `GET /todos/1`                     | `200`             |
| 6 | `GET /todos/999`                   | `404`             |
| 7 | Invalid JSON                       | `400`             |
| 8 | Unsupported method                 | appropriate error |

Don't just check the status code.

Check the **response body too**.

---

# 37. Try to Break Your Own API

This is one habit I really want you to develop.

After implementing an endpoint, don't think:

> "Done."

Think:

> "How can I break this?"

For example:

```text
What if title is missing?

What if title is null?

What if title is a number?

What if title is an array?

What if the ID doesn't exist?

What if the request body is empty?

What if the JSON is malformed?

What if the same request comes twice?

What if someone sends a huge payload?
```

This mindset becomes incredibly useful when you start working on production systems.

---

# 38. One Important Thing: Postman Doesn't Test Everything

Postman can tell you:

```text
"I sent this request and got this response."
```

It doesn't automatically tell you:

```text
"Your API is secure."

"Your database won't fail."

"Your API can handle 10,000 requests."

"Your business logic is correct."

"Your API won't have race conditions."
```

Testing is much bigger than sending requests.

Postman is a tool.

**Testing is a mindset.**

---

# 39. What You'll Eventually Add

As your backend becomes more advanced, your testing will evolve.

You'll go from:

```text
Postman
cURL
```

to:

```text
Automated tests
      ↓
Integration tests
      ↓
Database tests
      ↓
Authentication tests
      ↓
Load testing
      ↓
CI/CD tests
```

Eventually, every important change should be checked automatically before it reaches production.

We'll get there.

---

# 40. Interview Questions

## 1. What is API testing?

API testing is testing an API by sending requests and verifying things like status codes, response bodies, headers, validation, errors, and expected behavior.

---

## 2. What is Postman?

Postman is a tool that allows developers to create, send, inspect, and organize HTTP/API requests.

---

## 3. What is cURL?

cURL is a command-line tool for making requests over protocols such as HTTP and HTTPS.

---

## 4. Postman vs cURL?

Postman provides a visual interface and features like collections and environments.

cURL is command-line based and is especially useful for quick testing, debugging, scripting, and server environments.

---

## 5. How would you test a POST endpoint?

I'd test at least:

```text
Valid request
Missing required fields
Invalid data types
Empty values
Malformed JSON
Extra fields
Boundary/large inputs
Authentication
Duplicate requests
Expected status codes
Expected response body
```

---

## 6. Why shouldn't you test only successful requests?

Because production users don't always send valid requests.

A robust API must behave predictably when input is missing, malformed, unauthorized, or otherwise invalid.

---

## 7. What is an API contract?

An API contract defines how an API should be used and what clients can expect.

It includes things like:

```text
Endpoint
HTTP method
Request format
Required fields
Response format
Status codes
Error behavior
```

---

## 8. What's the difference between manual and automated API testing?

Manual testing means a developer sends requests using tools such as Postman or cURL.

Automated testing uses code to repeatedly send requests and verify expected results.

---

## 9. How would you debug an API returning 400?

I'd inspect:

```text
URL
HTTP method
headers
request body
JSON format
server logs
validation rules
route handling
```

I'd first determine whether the problem is with the request or the server.

---

## 10. How would you test an API you've never seen before?

I'd first understand its contract.

Then I'd test:

```text
Basic successful request
Required fields
Invalid inputs
Authentication
Authorization
Resource-not-found cases
Boundary cases
Error responses
```

I'd also inspect the actual HTTP requests and responses rather than relying only on the frontend.

---

# 41. A More Real Interview Scenario

Imagine the interviewer says:

> "Our `POST /users` endpoint works from the frontend, but another developer says it doesn't work from cURL. How would you investigate?"

Don't say:

> "I'd check the code."

That's too vague.

Think about the request.

I'd compare the working frontend request with the failing cURL request.

Check:

```text
HTTP method
URL
query parameters
headers
Content-Type
Authorization
request body
cookies
```

Then I'd compare the server responses:

```text
status code
response headers
response body
```

If needed, I'd use:

```bash
curl -v ...
```

to inspect the request/connection details.

The key idea is:

> **Find the difference between the working request and the failing request.**

That's a real debugging approach.

---

# 42. Mini Project — API Test Suite for Todo

Before moving to the next chapter, create a Postman collection:

```text
Todo API
│
├── GET All Todos
├── GET Todo By ID
├── POST Create Todo
├── POST Invalid Todo
├── PATCH Update Todo
├── DELETE Todo
│
└── Error Cases
    ├── Missing title
    ├── Wrong title type
    ├── Invalid ID
    └── Invalid JSON
```

Then create an environment:

```text
BASE_URL=http://localhost:3000
```

Use:

```text
{{BASE_URL}}/todos
```

instead of hardcoding:

```text
http://localhost:3000/todos
```

Also write the equivalent cURL commands for your important endpoints.

You should be able to test the entire Todo API without opening the frontend.

---

# 43. The Bigger Picture

Look at what we've built across Phase 1.

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
API Testing
```

You can now explain what happens when someone makes an API request:

```text
Client
  ↓
HTTP request
  ↓
Server receives request
  ↓
Router finds endpoint
  ↓
JSON body is parsed
  ↓
Input is validated
  ↓
Business logic runs
  ↓
Server creates response
  ↓
HTTP response
  ↓
Client receives JSON
```

That's already a decent foundation.

But we're missing something huge.

Our Todo API currently stores data somewhere like:

```js
const todos = [];
```

Which means:

```text
Server restarts
      ↓
All todos disappear
```

Obviously, that's not how real applications work.

We need persistent storage.

And that's where the next phase begins.

---

# Phase 2 — Data & Storage

We'll move from:

```text
JavaScript array
```

to:

```text
PostgreSQL
```

and learn what actually happens when your backend talks to a database.

We'll start with:

**Chapter 7 — Databases 101: SQL vs NoSQL**

You'll learn:

```text
What is a database?
Why do we need one?
Tables
Rows
Columns
Primary keys
Foreign keys
SQL
NoSQL
PostgreSQL
MongoDB
Redis
ACID
Transactions
Consistency
```

And most importantly:

> **How do I decide which database to use for a real application?**

That's where backend development starts getting really interesting.