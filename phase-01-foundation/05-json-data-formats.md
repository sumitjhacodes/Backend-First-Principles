# Chapter 5 — JSON & Data Formats

So far, we've learned how a request reaches our server, how HTTP works, and how routing decides which code should handle the request.

But there's still one important question.

Suppose our frontend wants to create a todo.

It sends:

```http
POST /todos
```

But the server needs to know:

```text
What's the title?
Is it completed?
Who created it?
```

So we need a way to represent data and send it between systems.

This is where **JSON** comes in.

You've probably already written JSON hundreds of times.

But there's a difference between:

```js
const todo = {
  title: "Learn backend",
  completed: false
};
```

and:

```json
{
  "title": "Learn backend",
  "completed": false
}
```

They look almost identical.

They're not exactly the same thing.

Let's understand what is actually happening.

---

# 1. What Is JSON?

JSON stands for:

> **JavaScript Object Notation**

The name is a little misleading today because JSON isn't really "for JavaScript".

It's a **text-based data format** that almost every programming language can understand.

For example:

```json
{
  "name": "Sumit",
  "age": 22,
  "isDeveloper": true
}
```

A Node.js application can understand it.

A Python application can understand it.

A Java application can understand it.

A Go application can understand it.

That's why JSON became so popular for APIs.

---

# 2. JSON Is Just Text

This is one of the most important things to understand.

JSON looks like an object:

```json
{
  "name": "Sumit",
  "age": 22
}
```

But JSON itself is **text**.

Imagine this data:

```js
const user = {
  name: "Sumit",
  age: 22
};
```

Inside your JavaScript program, that's a JavaScript object.

If we convert it into JSON:

```js
const json = JSON.stringify(user);
```

Now `json` is a string:

```text
'{"name":"Sumit","age":22}'
```

You can verify it:

```js
console.log(typeof user);
// object

console.log(typeof json);
// string
```

That's the first big distinction:

```text
JavaScript Object
       ↓
     object

JSON
       ↓
     string/text
```

---

# 3. Why Do We Need JSON?

Because different systems need a common language for exchanging structured data.

Imagine:

```text
React frontend
       ↓
    Internet
       ↓
Node.js backend
       ↓
PostgreSQL
```

These systems don't necessarily represent data internally in exactly the same way.

Your React application might have:

```js
const user = {
  name: "Sumit",
  age: 22
};
```

Your backend might have a JavaScript object.

Your database stores data according to its own data types and structures.

We need a format that can travel across the network.

JSON is one of the most common choices.

So:

```text
Application
    ↓
Convert data to JSON
    ↓
HTTP
    ↓
Network
    ↓
Server
    ↓
Parse JSON
    ↓
Application data
```

---

# 4. Serialization

Here's another word you'll hear in backend development:

> **Serialization**

Don't let the word scare you.

Serialization basically means:

> **Converting data into a format that can be stored or transmitted.**

For example:

```js
const todo = {
  title: "Learn backend",
  completed: false
};
```

We can serialize it into JSON:

```js
const json = JSON.stringify(todo);
```

Result:

```text
'{"title":"Learn backend","completed":false}'
```

Now it's text that can be sent over HTTP.

Think:

```text
Object
   ↓
Serialization
   ↓
JSON text
   ↓
Network
```

---

# 5. Parsing

Now the server receives:

```text
'{"title":"Learn backend","completed":false}'
```

It needs to turn that text back into something the application can work with.

That's called **parsing**.

```js
const todo = JSON.parse(json);
```

Now:

```js
todo.title
```

works again.

So remember:

```text
Serialization
Object → JSON

Parsing
JSON → Object
```

A simple mental model:

```text
JavaScript object
       ↓
 JSON.stringify()
       ↓
    JSON text
       ↓
     HTTP
       ↓
 JSON.parse()
       ↓
JavaScript object
```

---

# 6. `JSON.stringify()`

JavaScript gives us:

```js
JSON.stringify()
```

to convert a JavaScript value into a JSON string.

Example:

```js
const user = {
  name: "Sumit",
  age: 22,
  active: true
};

const json = JSON.stringify(user);

console.log(json);
```

Output:

```json
{"name":"Sumit","age":22,"active":true}
```

And:

```js
console.log(typeof json);
```

gives:

```text
string
```

---

# 7. `JSON.parse()`

The opposite is:

```js
JSON.parse()
```

Example:

```js
const json = '{"name":"Sumit","age":22}';

const user = JSON.parse(json);

console.log(user.name);
```

Output:

```text
Sumit
```

And:

```js
console.log(typeof user);
```

gives:

```text
object
```

So:

```text
JSON.stringify()
→ object/value to JSON string

JSON.parse()
→ JSON string to JavaScript value
```

---

# 8. JSON Data Types

JSON supports a relatively small set of data types.

### String

```json
{
  "name": "Sumit"
}
```

### Number

```json
{
  "age": 22
}
```

### Boolean

```json
{
  "active": true
}
```

or:

```json
{
  "active": false
}
```

### Null

```json
{
  "middleName": null
}
```

### Object

```json
{
  "user": {
    "name": "Sumit",
    "age": 22
  }
}
```

### Array

```json
{
  "skills": [
    "JavaScript",
    "TypeScript",
    "Node.js"
  ]
}
```

That's basically it.

JSON does **not** have every JavaScript type.

---

# 9. JSON Does NOT Have `undefined`

This catches beginners sometimes.

JavaScript:

```js
const user = {
  name: "Sumit",
  age: undefined
};
```

But JSON doesn't have an `undefined` value.

For example:

```js
JSON.stringify({
  name: "Sumit",
  age: undefined
});
```

The `age` property gets omitted:

```json
{"name":"Sumit"}
```

This matters when designing API responses.

Don't assume every JavaScript value can be represented directly in JSON.

---

# 10. JSON Does NOT Have Functions

You can have this in JavaScript:

```js
const user = {
  name: "Sumit",
  sayHello: function () {
    console.log("Hello");
  }
};
```

But functions aren't valid JSON data.

JSON is meant to represent data, not executable code.

This is a good security property too.

Your API should exchange data—not executable JavaScript.

---

# 11. JSON Keys Use Double Quotes

Valid JSON:

```json
{
  "name": "Sumit"
}
```

Not valid JSON:

```js
{
  name: "Sumit"
}
```

That's a JavaScript object literal.

Also not valid JSON:

```js
{
  'name': 'Sumit'
}
```

JSON requires double quotes around property names.

This is a very common beginner mistake.

---

# 12. Trailing Commas

This is valid JavaScript:

```js
const user = {
  name: "Sumit",
  age: 22,
};
```

But this is **not valid JSON**:

```json
{
  "name": "Sumit",
  "age": 22,
}
```

No trailing comma.

JSON is intentionally more restrictive.

---

# 13. JavaScript Object vs JSON

Let's make this distinction very clear.

### JavaScript object

```js
const user = {
  name: "Sumit",
  age: 22
};
```

This is a JavaScript value.

### JSON

```json
{
  "name": "Sumit",
  "age": 22
}
```

This is JSON data represented as text.

They look almost identical.

That's why beginners often use the words interchangeably.

But technically:

```text
JavaScript object ≠ JSON
```

JSON is a format.

A JavaScript object is a JavaScript data structure.

---

# 14. Let's Connect This to HTTP

Now things become interesting.

Suppose your frontend wants to create a todo.

It sends:

```http
POST /todos
Content-Type: application/json
```

And the body contains:

```json
{
  "title": "Learn backend",
  "completed": false
}
```

The important header is:

```text
Content-Type: application/json
```

It tells the server:

> "The body I'm sending is JSON."

This is important because HTTP itself doesn't magically know what your body means.

The body is just bytes.

The `Content-Type` tells the receiver how those bytes should be interpreted.

---

# 15. What Is `Content-Type`?

`Content-Type` is an HTTP header that tells the receiver what format the request or response body is using.

For JSON:

```http
Content-Type: application/json
```

For HTML:

```http
Content-Type: text/html
```

For plain text:

```http
Content-Type: text/plain
```

For form data:

```http
Content-Type: application/x-www-form-urlencoded
```

For file uploads, you'll commonly see:

```http
Content-Type: multipart/form-data
```

We'll learn file uploads later.

For now, remember:

```text
Content-Type
      ↓
"What format is this body?"
```

---

# 16. `Accept` Is Different

You might also see:

```http
Accept: application/json
```

This is different from `Content-Type`.

### `Content-Type`

Tells you:

> What format is the body I'm sending?

### `Accept`

Tells the server:

> What response format can I accept?

For example:

```http
POST /todos
Content-Type: application/json
Accept: application/json
```

means:

> I'm sending JSON, and I'd like JSON back.

This distinction is small but important in interviews.

---

# 17. Express and JSON

Remember this from our Todo API?

```js
app.use(express.json());
```

Now you should understand what it actually does.

It adds middleware that can parse incoming JSON request bodies.

Suppose the client sends:

```json
{
  "title": "Learn backend"
}
```

With:

```js
app.use(express.json());
```

your route can access:

```js
req.body
```

So:

```js
app.post("/todos", (req, res) => {
  console.log(req.body);
});
```

might give:

```js
{
  title: "Learn backend"
}
```

Without JSON parsing middleware, Express won't automatically give your route a convenient parsed JavaScript object for a JSON request body.

---

# 18. Let's See the Complete Flow

Imagine a React frontend.

The user clicks:

> Add Todo

Frontend code:

```js
const todo = {
  title: "Learn backend",
  completed: false
};

fetch("/todos", {
  method: "POST",
  headers: {
    "Content-Type": "application/json"
  },
  body: JSON.stringify(todo)
});
```

Look carefully.

We start with:

```js
todo
```

which is a JavaScript object.

Then:

```js
JSON.stringify(todo)
```

turns it into JSON text.

Then HTTP sends that data.

The server receives it.

Express parses it.

Then:

```js
req.body
```

becomes a JavaScript object again.

So:

```text
Frontend JS object
       ↓
JSON.stringify()
       ↓
JSON text
       ↓
HTTP request
       ↓
Express JSON parser
       ↓
req.body
       ↓
Backend JS object
```

This is one of the most useful things to understand from this chapter.

---

# 19. Sending JSON Back

The same thing happens in the opposite direction.

Our server might have:

```js
const todo = {
  id: 1,
  title: "Learn backend",
  completed: false
};
```

We want to send it to the frontend.

With Express:

```js
res.json(todo);
```

Express handles the JSON response for us.

Conceptually:

```text
JavaScript object
       ↓
JSON serialization
       ↓
HTTP response
       ↓
Frontend
       ↓
JSON parsing
       ↓
JavaScript object
```

The response will look roughly like:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 1,
  "title": "Learn backend",
  "completed": false
}
```

---

# 20. Why `res.json()` Instead of `res.send()`?

Express gives you several ways to send a response.

For JSON:

```js
res.json(todo);
```

is explicit and convenient.

It communicates:

> "I'm sending JSON."

Express also handles appropriate response formatting and headers.

You can manually send strings, but for API responses, `res.json()` makes your intention clear.

---

# 21. JSON Arrays

APIs don't only return one object.

For example:

```json
[
  {
    "id": 1,
    "title": "Learn backend",
    "completed": false
  },
  {
    "id": 2,
    "title": "Build API",
    "completed": true
  }
]
```

This is a JSON array.

An endpoint like:

```text
GET /todos
```

could return:

```json
[
  {
    "id": 1,
    "title": "Learn backend",
    "completed": false
  },
  {
    "id": 2,
    "title": "Build API",
    "completed": true
  }
]
```

---

# 22. Objects Inside Arrays

You'll see this constantly in real APIs.

For example:

```json
{
  "users": [
    {
      "id": 1,
      "name": "Rahul"
    },
    {
      "id": 2,
      "name": "Aman"
    }
  ]
}
```

This is:

```text
Object
  ↓
users property
  ↓
Array
  ↓
Objects
```

JSON can be nested like this quite deeply.

But don't create unnecessarily complicated responses.

Simple is easier for clients to consume.

---

# 23. What Should an API Response Look Like?

There's no single universal structure.

For example, you could return:

```json
[
  {
    "id": 1,
    "title": "Learn backend"
  }
]
```

Or:

```json
{
  "data": [
    {
      "id": 1,
      "title": "Learn backend"
    }
  ]
}
```

Both are possible.

For a small project, the first might be perfectly fine.

For a larger API, you might eventually want a consistent response structure.

For example:

```json
{
  "data": {
    "id": 1,
    "title": "Learn backend"
  }
}
```

Or for lists:

```json
{
  "data": [
    {
      "id": 1,
      "title": "Learn backend"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20
  }
}
```

The important thing isn't copying someone else's format.

It's being **consistent**.

---

# 24. Error Responses Are Data Too

Suppose:

```text
GET /todos/999
```

doesn't exist.

We could return:

```json
{
  "error": "Todo not found"
}
```

with:

```http
404 Not Found
Content-Type: application/json
```

A more structured error might be:

```json
{
  "error": {
    "code": "TODO_NOT_FOUND",
    "message": "Todo not found"
  }
}
```

Again, there isn't one universal format.

What matters is that your API gives clients enough information to handle the error.

---

# 25. Don't Return Random Error Shapes

Avoid an API where one endpoint returns:

```json
{
  "error": "Something went wrong"
}
```

another returns:

```json
{
  "message": "Invalid request"
}
```

and another returns:

```json
{
  "errors": [
    "Email required"
  ]
}
```

You can technically do this.

But clients now have to handle three different formats.

A consistent API is much easier to work with.

For example, you might decide:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid request",
    "details": []
  }
}
```

and use that pattern consistently.

---

# 26. JSON and Dates

Here's an interesting problem.

JavaScript has a:

```js
Date
```

type.

JSON doesn't have a dedicated Date type.

So you typically send dates as strings.

For example:

```json
{
  "createdAt": "2026-08-28T10:30:00.000Z"
}
```

The client can then convert that string into a Date object if needed.

This is a good example of why:

> JSON doesn't contain every programming language's data type.

You need a representation that works across languages.

---

# 27. JSON and Big Numbers

Another subtle issue is numbers.

JSON has a number type, but different programming languages have different number representations and limits.

JavaScript's regular `number` has limitations for very large integers.

So if your system deals with values such as extremely large IDs or financial values with strict precision requirements, you need to think carefully about how they're represented.

For example, some APIs represent large identifiers as strings:

```json
{
  "id": "9223372036854775807"
}
```

instead of:

```json
{
  "id": 9223372036854775807
}
```

This isn't something you need to overthink for your Todo API.

But it's a good example of real-world backend engineering:

> **Data representation can become a compatibility problem.**

---

# 28. JSON vs XML

Before JSON became extremely common, APIs frequently used XML.

XML:

```xml
<user>
  <name>Sumit</name>
  <age>22</age>
</user>
```

JSON:

```json
{
  "name": "Sumit",
  "age": 22
}
```

JSON is generally easier to read and commonly maps naturally to objects used by modern web applications.

But XML isn't "dead".

You'll still encounter it in:

* older enterprise systems
* SOAP services
* some government systems
* configuration formats
* specific integrations

So don't say in an interview:

> "XML is useless."

That's not true.

A better answer is:

> "JSON is very common for modern web APIs because it's lightweight and easy for applications to work with, while XML is still used in many legacy and enterprise systems."

---

# 29. JSON vs Form Data

JSON isn't the only way to send data.

You might see:

```text
application/json
```

or:

```text
application/x-www-form-urlencoded
```

or:

```text
multipart/form-data
```

### JSON

Good for structured API data:

```json
{
  "name": "Sumit",
  "age": 22
}
```

### URL-encoded form

Looks roughly like:

```text
name=Sumit&age=22
```

### Multipart form

Common when sending:

```text
text fields + files
```

For example:

```text
profile picture
+
username
+
bio
```

We'll deal with `multipart/form-data` when we learn file uploads.

---

# 30. JSON Is Not a Database

This sounds obvious, but beginners sometimes mix these concepts.

JSON is a **data representation format**.

PostgreSQL is a **database system**.

Redis is a **data store**.

JSON might be:

```text
Database
   ↓
Backend object
   ↓
JSON
   ↓
HTTP
   ↓
Frontend
```

JSON doesn't replace your database.

It is often the format used when data travels between systems.

---

# 31. JSON Is Not an API

Another common misunderstanding:

> "Our backend is JSON."

No.

Your backend is an application/server.

An API is an interface through which systems communicate.

JSON is one possible data format used by that API.

Think:

```text
API
 ├── HTTP
 ├── URLs
 ├── Methods
 ├── Headers
 └── JSON data
```

JSON is one part of the whole system.

---

# 32. Malformed JSON

Now let's look at a real-world problem.

A client sends:

```json
{
  "title": "Learn backend"
```

Notice the missing:

```text
}
```

That's invalid JSON.

Your server needs to handle this safely.

With Express JSON parsing middleware:

```js
app.use(express.json());
```

malformed JSON can result in a client error rather than your application happily receiving a valid object.

The important lesson:

> **Never assume incoming data is valid.**

Your API receives data from outside your application.

Treat it as untrusted input.

---

# 33. JSON Parsing Is Not Validation

This distinction is extremely important.

Suppose the client sends valid JSON:

```json
{
  "title": 123
}
```

That's valid JSON.

But maybe your Todo API expects:

```text
title → string
```

JSON parsing succeeds.

Application validation should fail.

So:

```text
JSON parsing
    ↓
"Is this valid JSON?"

Validation
    ↓
"Does this JSON contain valid data for my application?"
```

They're two different jobs.

For example:

```json
{
  "title": ""
}
```

is valid JSON.

But your business rules might say:

> Todo title cannot be empty.

Again:

```text
Valid JSON
≠
Valid application input
```

Remember this.

---

# 34. A Real Request Flow

Let's put everything together.

User clicks:

```text
Create Todo
```

Frontend:

```js
const todo = {
  title: "Learn backend"
};
```

Then:

```js
JSON.stringify(todo)
```

produces:

```text
{"title":"Learn backend"}
```

The frontend sends:

```http
POST /todos HTTP/1.1
Content-Type: application/json

{"title":"Learn backend"}
```

The request travels through the network.

Our Node.js server receives it.

Express sees:

```text
Content-Type: application/json
```

and parses the body.

Our route:

```js
app.post("/todos", (req, res) => {
  console.log(req.body);
});
```

gets:

```js
{
  title: "Learn backend"
}
```

We validate it.

Then maybe save it to PostgreSQL.

Then return:

```http
HTTP/1.1 201 Created
Content-Type: application/json

{
  "id": 1,
  "title": "Learn backend",
  "completed": false
}
```

The frontend receives the JSON and parses it.

That's a real API request.

Not magic.

Just data being represented, transmitted, parsed, validated, and processed.

---

# 35. Let's Build Something

Let's upgrade our Todo API.

Start with:

```js
const express = require("express");

const app = express();

app.use(express.json());

const todos = [];

app.post("/todos", (req, res) => {
  const { title } = req.body;

  if (!title || typeof title !== "string") {
    return res.status(400).json({
      error: {
        code: "INVALID_TITLE",
        message: "Title must be a non-empty string",
      },
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
  console.log("Server running on port 3000");
});
```

Now test it.

### Valid request

```http
POST /todos
Content-Type: application/json
```

Body:

```json
{
  "title": "Learn backend"
}
```

Expected:

```http
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

---

# 36. Try Breaking Your API

This is where I want you to spend some time.

Don't just test the happy path.

Send:

### Missing title

```json
{}
```

What should happen?

```text
400 Bad Request
```

---

### Wrong type

```json
{
  "title": 123
}
```

Should fail validation.

---

### Empty string

```json
{
  "title": ""
}
```

Should probably fail.

---

### Extra field

```json
{
  "title": "Learn backend",
  "somethingRandom": "hello"
}
```

Should your API ignore it?

Reject it?

Store it?

This is a design decision.

Don't automatically assume there's one correct answer.

---

# 37. A Small Exercise

Create these three endpoints:

```text
GET /todos
POST /todos
GET /todos/:id
```

Then make sure:

### `POST /todos`

Accepts:

```json
{
  "title": "Learn HTTP"
}
```

Returns:

```json
{
  "id": 1,
  "title": "Learn HTTP",
  "completed": false
}
```

### `GET /todos`

Returns an array:

```json
[
  {
    "id": 1,
    "title": "Learn HTTP",
    "completed": false
  }
]
```

### `GET /todos/:id`

Returns one object:

```json
{
  "id": 1,
  "title": "Learn HTTP",
  "completed": false
}
```

And if it doesn't exist:

```json
{
  "error": {
    "code": "TODO_NOT_FOUND",
    "message": "Todo not found"
  }
}
```

with:

```text
404 Not Found
```

---

# 38. Common Beginner Mistakes

## Mistake 1: Thinking JSON and JavaScript objects are the same

They're similar in appearance but technically different.

```text
JavaScript object → in-memory value
JSON → text-based data format
```

---

## Mistake 2: Forgetting `Content-Type`

If you're sending JSON, tell the server:

```http
Content-Type: application/json
```

---

## Mistake 3: Forgetting `JSON.stringify()`

This:

```js
body: todo
```

isn't the normal way to send a JSON request body with `fetch`.

Use:

```js
body: JSON.stringify(todo)
```

---

## Mistake 4: Thinking parsing means validation

Valid JSON can still contain terrible application data.

```json
{
  "age": "banana"
}
```

Valid JSON.

Probably invalid user data.

---

## Mistake 5: Trusting the client

Never assume:

> "My React app only sends valid data."

Someone can call your API directly using:

```text
curl
Postman
another application
a script
```

The backend must validate input itself.

---

# 39. Interview Questions

## 1. What is JSON?

JSON is a lightweight, text-based data format commonly used for exchanging structured data between systems, especially in web APIs.

---

## 2. Is JSON a JavaScript object?

No.

A JavaScript object is a JavaScript data structure, while JSON is a text-based data format.

For example:

```js
const user = {
  name: "Sumit"
};
```

is a JavaScript object.

While:

```json
{
  "name": "Sumit"
}
```

is JSON.

---

## 3. What does `JSON.stringify()` do?

It converts a JavaScript value into a JSON string.

```js
const user = {
  name: "Sumit"
};

const json = JSON.stringify(user);
```

---

## 4. What does `JSON.parse()` do?

It converts a valid JSON string into a JavaScript value.

```js
const json = '{"name":"Sumit"}';

const user = JSON.parse(json);
```

---

## 5. What is serialization?

Serialization is converting data into a representation suitable for storage or transmission.

For example:

```text
JavaScript object
       ↓
JSON.stringify()
       ↓
JSON string
```

---

## 6. What is parsing?

Parsing is interpreting a serialized representation and converting it back into a usable data structure.

For example:

```text
JSON string
       ↓
JSON.parse()
       ↓
JavaScript object
```

---

## 7. What is `Content-Type: application/json`?

It tells the receiver that the HTTP message body contains JSON data.

---

## 8. What's the difference between `Content-Type` and `Accept`?

`Content-Type` describes the format of the body being sent.

`Accept` describes the response formats the client is willing to receive.

Example:

```http
Content-Type: application/json
Accept: application/json
```

---

## 9. Is valid JSON necessarily valid application data?

No.

For example:

```json
{
  "age": "hello"
}
```

is valid JSON.

But if your application expects `age` to be a number, it's invalid application input.

---

## 10. Why is JSON commonly used in APIs?

Because it's relatively simple, human-readable, widely supported across programming languages, and works well for representing structured data.

---

# 40. Interview Scenario

Here's a more realistic interview question.

> A frontend sends `POST /users` with JSON, but `req.body` is `undefined`. What would you check?

Don't immediately start changing random code.

Think through the request.

I'd check:

### 1. Is JSON parsing middleware enabled?

```js
app.use(express.json());
```

### 2. Is it registered before the route?

Correct:

```js
app.use(express.json());

app.post("/users", handler);
```

Not:

```js
app.post("/users", handler);

app.use(express.json());
```

because middleware order matters.

### 3. Is the client actually sending JSON?

Check:

```http
Content-Type: application/json
```

### 4. Is the body being serialized?

With `fetch`:

```js
body: JSON.stringify(data)
```

### 5. Is the request actually reaching the route you think it is?

Check the URL and HTTP method.

This is how you should approach backend debugging.

Don't memorize:

> "If req.body is undefined, add express.json()."

Understand the entire request flow.

---

# 41. The Mental Model I Want You to Keep

Whenever you see:

```js
fetch("/todos", {
  method: "POST",
  headers: {
    "Content-Type": "application/json"
  },
  body: JSON.stringify(todo)
});
```

your brain should automatically translate it into:

```text
JavaScript object
       ↓
JSON.stringify
       ↓
JSON text
       ↓
HTTP request body
       ↓
Content-Type tells server it's JSON
       ↓
Express parses JSON
       ↓
req.body
       ↓
Validation
       ↓
Business logic
       ↓
Database
       ↓
Response
       ↓
JSON
       ↓
Frontend
```

That's backend development.

There are just a lot of layers involved.

Once you understand what each layer is doing, backend stops feeling like magic.

---

# 42. Where We Are Now

Our learning path is:

```text
Chapter 1
How the Internet Works
        ↓
Chapter 2
HTTP Deep Dive
        ↓
Chapter 3
What Is a Web Server?
        ↓
Chapter 4
Routing & URL Design
        ↓
Chapter 5
JSON & Data Formats
```

And our Todo API now understands:

```text
HTTP
   ↓
Routes
   ↓
Path parameters
   ↓
Query parameters
   ↓
JSON request bodies
   ↓
JSON responses
   ↓
Validation
   ↓
HTTP status codes
```

We're getting somewhere.

---

# 43. Before Moving On

I don't want you to just read this chapter and move on.

Build the Todo API.

Then use Postman or `curl` and deliberately break it.

Try:

```text
Invalid JSON
Missing fields
Wrong data types
Unknown todo IDs
Huge limits
Unexpected fields
Empty strings
```

Watch what your server does.

When something breaks, don't immediately search for the answer.

Ask yourself:

> What did the client send?

> What did the server receive?

> Did JSON parsing happen?

> Did validation happen?

> Which route handled it?

> What response did the server return?

That habit will help you far more than memorizing another 50 interview questions.

---

# Next — Chapter 6: Postman, cURL & API Testing

We've built enough of our Todo API to start testing it properly.

Next we'll stop relying on:

> "I opened the browser and it seemed to work."

We'll learn how backend developers actually test APIs:

```text
GET
POST
PATCH
DELETE
Headers
JSON bodies
Query parameters
Path parameters
Authentication headers
Environment variables
```

We'll use both **Postman and cURL**, and we'll create a proper test checklist for our Todo API.

Because writing an API is only half the job.

**You need to know how to prove that it actually works.**