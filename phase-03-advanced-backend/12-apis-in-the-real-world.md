# Chapter 12 — APIs in the Real World

So far, we've built APIs.

We've learned:

* HTTP
* routes
* JSON
* databases
* SQL
* Prisma
* authentication
* password security

At this point, you can probably build something like:

```text
GET    /posts
GET    /posts/:id
POST   /posts
PATCH  /posts/:id
DELETE /posts/:id
```

And for a small project, this might work perfectly.

But then your application grows.

You suddenly have:

```text
10 posts
        ↓
10,000 posts
        ↓
1,000,000 posts
        ↓
10,000,000 posts
```

And your API still has:

```text
GET /posts
```

Now we have a problem.

Are we really going to send 10 million posts to the browser?

Obviously not.

Then users start asking:

> "Can I search?"

> "Can I filter by location?"

> "Can I sort by salary?"

> "Can I get only the next 20 results?"

Your frontend team asks:

> "Can we depend on this response format?"

And six months later you realize:

> "Oops... we changed the API and broke the mobile app."

This chapter is about the things that start mattering when your API isn't just a demo anymore.

---

# 1. What Makes an API "Real World"?

A beginner API might look like:

```http
GET /users
```

and return:

```json
[
  {
    "id": 1,
    "name": "Rahul"
  },
  {
    "id": 2,
    "name": "Aman"
  }
]
```

Nothing wrong with that.

But real applications need to deal with:

```text
Large datasets
        ↓
Pagination

User requirements
        ↓
Filtering

Different ways of viewing data
        ↓
Sorting

Searching
        ↓
Search parameters

Changing requirements
        ↓
API versioning

Errors
        ↓
Consistent error responses

Multiple clients
        ↓
Stable API contracts
```

That's what we're going to learn.

---

# 2. Pagination

Let's start with one of the most important concepts.

Imagine:

```http
GET /jobs
```

Our database contains:

```text
5,000,000 jobs
```

Should the API return all of them?

No.

Instead, we split the results into smaller chunks.

That's **pagination**.

Think about Google search results.

You don't get every page on the internet.

You get a small set of results.

Then you ask for more.

That's pagination.

---

# 3. Why Do We Need Pagination?

Without pagination:

```text
Database
   ↓
5,000,000 rows
   ↓
Backend
   ↓
Huge JSON response
   ↓
Network
   ↓
Browser
```

That's bad for several reasons.

The server might need to:

* read a lot of data
* process a lot of data
* serialize a huge response

The network has to transfer it.

The client has to download it.

The client has to parse it.

And the user probably only sees:

```text
20 jobs
```

So instead:

```text
Database
   ↓
20 rows
   ↓
Backend
   ↓
20 results
   ↓
Client
```

Much more reasonable.

---

# 4. Offset Pagination

One simple approach is **offset pagination**.

You might design:

```http
GET /jobs?page=1&limit=20
```

Meaning:

```text
page = 1
limit = 20
```

Return the first 20 jobs.

Then:

```http
GET /jobs?page=2&limit=20
```

returns the next 20.

---

# 5. How Does Offset Work?

You can think about it like:

```text
page = 1
limit = 20

offset = (page - 1) × limit

offset = 0
```

So:

```sql
SELECT *
FROM jobs
LIMIT 20
OFFSET 0;
```

Page 2:

```text
offset = (2 - 1) × 20
       = 20
```

So:

```sql
SELECT *
FROM jobs
LIMIT 20
OFFSET 20;
```

Page 3:

```text
OFFSET 40
```

And so on.

---

# 6. A Simple API

You could have:

```http
GET /jobs?page=2&limit=20
```

and return:

```json
{
  "data": [
    {
      "id": 21,
      "title": "Backend Engineer"
    }
  ],
  "pagination": {
    "page": 2,
    "limit": 20,
    "total": 5000,
    "totalPages": 250
  }
}
```

This gives the frontend enough information to build:

```text
← Previous
1 2 3 4 5
Next →
```

---

# 7. Always Put Limits on Pagination

Here's a small security/performance mistake.

Suppose your API accepts:

```http
GET /jobs?limit=100000000
```

If you blindly trust the client, they might request an enormous amount of data.

Instead, define a maximum.

For example:

```text
default limit = 20
maximum limit = 100
```

Then:

```http
/jobs?limit=1000000
```

could effectively become:

```text
limit = 100
```

The exact number depends on your application.

The principle is:

> **Never let the client decide how much work your server must do without limits.**

---

# 8. Offset Pagination Problems

Offset pagination is simple.

But it has problems when datasets become large or change frequently.

Imagine:

```text
Page 1
A
B
C
D
```

Then someone inserts:

```text
X
```

at the beginning.

Now page 2 might shift.

You can end up seeing duplicates or missing items while moving between pages.

There can also be performance issues with very large offsets depending on the database and query.

For example:

```sql
OFFSET 9000000
```

isn't necessarily a great way to navigate through millions of rows.

This leads us to another approach.

---

# 9. Cursor Pagination

Instead of saying:

> "Give me page 50."

we say:

> "Give me the next 20 records after this record."

That's **cursor pagination**.

Imagine:

```text
First request:

GET /jobs?limit=20
```

The API returns:

```json
{
  "data": [
    ...
  ],
  "nextCursor": "abc123"
}
```

Then the client asks:

```http
GET /jobs?limit=20&cursor=abc123
```

The API returns the next set.

---

# 10. Why Is This Useful?

Think about scrolling through a feed.

Instead of:

```text
page 1
page 2
page 3
page 4
```

you can say:

```text
Give me the next 20 after this item.
```

Conceptually:

```text
A
B
C
D
E
F
G
H
```

The client received:

```text
A B C D
```

The cursor represents where it stopped.

Then:

```text
cursor = D
```

means:

> Give me the next records after D.

---

# 11. Cursor Pagination Example

Suppose jobs are ordered by ID.

You might have:

```http
GET /jobs?limit=20
```

Then the database query can conceptually look like:

```sql
SELECT *
FROM jobs
WHERE id > 100
ORDER BY id
LIMIT 20;
```

Where:

```text
100
```

represents the last item from the previous result.

This can be much more efficient than repeatedly skipping huge numbers of rows.

But cursor pagination has its own requirements.

---

# 12. Cursor Needs Stable Ordering

You can't just randomly sort data and expect cursor pagination to work correctly.

You need a stable ordering.

For example:

```sql
ORDER BY created_at DESC, id DESC
```

Using a unique tie-breaker such as `id` helps when multiple records have the same `created_at`.

The exact cursor design depends on your query.

This is one reason cursor pagination is more complicated than:

```text
?page=2
```

---

# 13. Offset vs Cursor

Here's the simple mental model.

| Offset                         | Cursor                                    |
| ------------------------------ | ----------------------------------------- |
| Easy to understand             | More complex                              |
| Easy page numbers              | Better for continuous feeds               |
| Good for many admin/table UIs  | Good for large/changing datasets          |
| Can struggle with huge offsets | Can scale better for sequential traversal |
| Can shift when data changes    | More stable when designed properly        |

Don't ask:

> "Which one is better?"

Ask:

> **"What does my application need?"**

---

# 14. Filtering

Now imagine our job API.

We have:

```text
10,000,000 jobs
```

A user doesn't want all of them.

They want:

```text
Backend jobs
in India
with TypeScript
remote
salary > ₹10 LPA
```

So we need **filtering**.

We might design:

```http
GET /jobs?location=India&remote=true
```

Or:

```http
GET /jobs?experience=2&remote=true
```

Query parameters are useful for optional filtering.

---

# 15. Path Parameters vs Query Parameters

Remember Chapter 4?

A path parameter usually identifies a specific resource:

```http
GET /users/42
```

Meaning:

> Get user 42.

A query parameter usually modifies the result:

```http
GET /users?role=admin
```

Meaning:

> Get users filtered by role.

Simple mental model:

```text
Path parameter
→ Which resource?

Query parameter
→ How should I search/filter/view it?
```

---

# 16. Combining Filters

A real API might have:

```http
GET /jobs?location=India&remote=true&experience=2
```

This means:

```text
location = India
remote = true
experience = 2
```

The backend converts those filters into a database query.

Conceptually:

```sql
SELECT *
FROM jobs
WHERE location = 'India'
  AND remote = true
  AND experience = 2;
```

Don't directly concatenate user input into SQL.

Use parameterized queries or your ORM's safe query mechanisms.

---

# 17. Optional Filters

Filters should generally be optional.

For example:

```http
GET /jobs
```

means:

> Give me jobs without additional filters.

While:

```http
GET /jobs?remote=true
```

means:

> Give me remote jobs.

And:

```http
GET /jobs?remote=true&location=India
```

means:

> Give me remote jobs in India.

Your backend builds the query based on which parameters are actually present.

---

# 18. Validate Query Parameters

Don't assume the client sends valid values.

For example:

```http
GET /jobs?limit=hello
```

or:

```http
GET /jobs?remote=maybe
```

or:

```http
GET /jobs?experience=-100
```

Your backend should validate these values.

For example:

```text
limit
→ integer
→ greater than 0
→ maximum 100

remote
→ boolean

experience
→ valid range
```

Never trust client input.

This rule applies to:

```text
Body
Path parameters
Query parameters
Headers
Cookies
```

---

# 19. Sorting

Users might want:

```text
Newest jobs
Highest salary
Lowest salary
Most popular
```

So we can support:

```http
GET /jobs?sort=salary&order=desc
```

or:

```http
GET /jobs?sort=createdAt&order=desc
```

But be careful.

Don't blindly pass a client-provided string into SQL.

For example, don't build:

```js
`ORDER BY ${req.query.sort}`
```

without validating it.

Instead use an allowlist.

Conceptually:

```text
Allowed sort fields:

createdAt
salary
title
```

If the client sends:

```text
sort=password
```

reject it.

---

# 20. Searching

Filtering and searching aren't always the same.

Filtering:

```text
location = India
```

Searching:

```text
query = backend engineer
```

A simple API might have:

```http
GET /jobs?search=backend
```

The backend might search:

```text
title
description
skills
company
```

At small scale, PostgreSQL search features may be enough.

At larger scale, dedicated search systems can become useful.

We'll encounter those kinds of architecture decisions later.

---

# 21. Pagination + Filtering + Sorting

Real APIs usually combine these.

For example:

```http
GET /jobs
    ?search=backend
    &location=India
    &remote=true
    &sort=salary
    &order=desc
    &page=2
    &limit=20
```

This looks ugly.

But it represents a very real API requirement.

The backend has to:

```text
Parse parameters
      ↓
Validate parameters
      ↓
Build database query
      ↓
Filter
      ↓
Sort
      ↓
Paginate
      ↓
Return response
```

This is where API design starts becoming real engineering.

---

# 22. API Versioning

Now let's talk about a problem that appears when your API has users.

Suppose your current API returns:

```json
{
  "name": "Rahul",
  "email": "rahul@example.com"
}
```

Your mobile app depends on:

```text
name
email
```

Six months later, you decide to completely change the response:

```json
{
  "fullName": "Rahul",
  "contact": {
    "email": "rahul@example.com"
  }
}
```

You deploy it.

Your old mobile application breaks.

Oops.

This is why API compatibility matters.

---

# 23. What Is API Versioning?

API versioning gives clients a stable contract while allowing the API to evolve.

A common approach:

```text
/v1/users
/v2/users
```

For example:

```http
GET /api/v1/users/42
```

and:

```http
GET /api/v2/users/42
```

Now you can maintain different contracts.

---

# 24. Why Version APIs?

Imagine:

```text
Mobile App
       ↓
v1
```

and:

```text
New Web App
       ↓
v2
```

You can gradually migrate clients.

Instead of:

```text
Change API
   ↓
Everything breaks
```

you can:

```text
v1 → existing clients
v2 → new clients
```

Then eventually retire v1 when appropriate.

---

# 25. Versioning Doesn't Mean "Version Every Tiny Change"

You don't need:

```text
v1.0.1
v1.0.2
v1.0.3
v1.0.4
```

for every little change.

The important question is:

> **Does this change break the existing API contract?**

Adding a new optional response field may not break clients.

Removing a field might.

Changing:

```text
string
```

into:

```text
object
```

could.

Changing the meaning of an existing field can also break clients even if the JSON shape looks the same.

---

# 26. Breaking vs Non-Breaking Changes

A useful mental model:

### Usually non-breaking

Adding an optional request field.

Adding a new endpoint.

Adding a new response field, assuming clients tolerate unknown fields.

### Potentially breaking

Removing a response field.

Renaming a field.

Changing a field's type.

Changing an existing field's meaning.

Changing authentication requirements.

Changing error semantics clients depend on.

The exact impact depends on how clients consume the API.

---

# 27. API Versioning Isn't Only `/v1`

You may also encounter versioning through headers.

For example:

```http
Accept: application/vnd.myapp.v2+json
```

Or other content-negotiation strategies.

There are multiple approaches.

For a beginner-friendly API, URL versioning:

```text
/api/v1
```

is often easier to understand.

The important thing isn't memorizing every versioning strategy.

Understand the problem:

> **APIs are contracts, and contracts need to evolve without unexpectedly breaking their consumers.**

---

# 28. API Response Design

Suppose we return:

```json
[
  {
    "id": 1,
    "title": "Backend Engineer"
  }
]
```

That's fine for a small endpoint.

But as APIs grow, consistent response structures become useful.

For example:

```json
{
  "data": [
    {
      "id": 1,
      "title": "Backend Engineer"
    }
  ],
  "pagination": {
    "nextCursor": "abc123"
  }
}
```

Now the client knows where the actual data is and where metadata lives.

Don't create complicated response wrappers just because someone on the internet said they're "best practice."

Consistency is more important than a specific shape.

---

# 29. Error Responses

Your API should also return predictable errors.

Bad:

```text
Something went wrong
```

Better:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid email address"
  }
}
```

For example:

```http
400 Bad Request
```

with:

```json
{
  "error": {
    "code": "INVALID_EMAIL",
    "message": "Please provide a valid email address."
  }
}
```

Now the frontend can understand what happened.

---

# 30. Don't Leak Internal Errors

Imagine PostgreSQL throws:

```text
relation "users_xyz" does not exist
```

Don't send that directly to the client.

Don't return:

```json
{
  "error": "PostgreSQL connection failed at 10.20.30.40"
}
```

Internal errors may reveal information attackers shouldn't have.

Instead:

```json
{
  "error": {
    "code": "INTERNAL_SERVER_ERROR",
    "message": "Something went wrong."
  }
}
```

And log the detailed error internally.

So:

```text
Client
→ safe error

Server logs
→ detailed diagnostic information
```

---

# 31. Status Codes Still Matter

Don't return:

```http
200 OK
```

for every situation.

Use appropriate status codes.

For example:

```text
200
→ successful request

201
→ resource created

204
→ successful request with no response body

400
→ invalid request

401
→ not authenticated

403
→ authenticated but not allowed

404
→ resource not found

409
→ conflict

422
→ semantically invalid input, where your API uses this convention

429
→ too many requests

500
→ unexpected server error
```

You don't need to memorize every HTTP status code.

Understand what they're communicating.

---

# 32. The Difference Between 400 and 422

You may see both.

The exact use varies between APIs.

A useful mental model:

```text
400
→ request is malformed or invalid

422
→ request structure is understandable,
   but the data doesn't satisfy the API's rules
```

For example:

```json
{
  "email": "not-an-email"
}
```

could be treated as a validation error.

The important thing is consistency within your API.

---

# 33. Idempotency in Real APIs

Remember Chapter 2?

Let's bring back **idempotency**.

Suppose a payment API receives:

```http
POST /payments
```

The client sends the request.

But the network times out.

The client doesn't know whether the payment succeeded.

So it retries.

Now we have:

```text
Request 1
→ payment created

Request 2
→ payment created again
```

That's a serious problem.

This is where **idempotency keys** can help.

---

# 34. Idempotency Keys

The client sends:

```http
POST /payments
Idempotency-Key: 8f7c...
```

The server remembers that key.

First request:

```text
key = 8f7c
→ process payment
→ store result
```

Retry:

```text
key = 8f7c
→ already processed
→ return previous result
```

Conceptually:

```text
Client
   ↓
Payment + idempotency key
   ↓
Server
   ↓
Have we seen this key?
   ├── No → process
   └── Yes → return previous result
```

This is extremely useful for operations where retries can cause duplicate side effects.

---

# 35. Don't Make Everything Idempotent

Not every endpoint needs an idempotency key.

For example:

```text
GET /users/42
```

is already designed to be safe to repeat.

But:

```text
POST /payments
```

may create a side effect.

Think about the consequences of retries.

---

# 36. API Timeouts

Here's another real-world issue.

Your API calls:

```text
Backend
   ↓
Payment provider
```

What if the payment provider takes:

```text
30 seconds
```

to respond?

Should your API wait forever?

No.

External calls should have appropriate timeouts.

Conceptually:

```text
Request
   ↓
External API
   ↓
wait...
   ↓
timeout
   ↓
handle failure
```

A timeout is not necessarily proof that the operation didn't happen.

This is especially important for payments and other side effects.

That's why idempotency and timeout handling often need to work together.

---

# 37. API Contracts

Think of your API as a contract between:

```text
Frontend
     ↕
   Backend
```

For example:

```http
GET /users/42
```

might promise:

```json
{
  "id": 42,
  "name": "Rahul"
}
```

The frontend builds against that contract.

So changing it carelessly can break another application.

This is why API design isn't just:

> "What JSON should I return?"

It's:

> **"What contract am I creating with my clients?"**

---

# 38. Documentation

If other developers use your API, they need to know:

```text
Endpoints
Methods
Parameters
Request body
Response body
Authentication
Errors
Pagination
Rate limits
Examples
```

That's why tools/specifications such as OpenAPI are useful.

You might eventually document:

```text
GET /api/v1/jobs
```

with:

```text
Query parameters
→ search
→ location
→ page
→ limit

Response
→ jobs
→ pagination metadata
```

Good API documentation saves a lot of back-and-forth.

---

# 39. API Design Example

Let's design our job API.

We start with:

```text
GET /api/v1/jobs
```

Then support:

```text
GET /api/v1/jobs
GET /api/v1/jobs/:id
```

Filtering:

```text
GET /api/v1/jobs?location=India
```

Search:

```text
GET /api/v1/jobs?search=backend
```

Sorting:

```text
GET /api/v1/jobs?sort=salary&order=desc
```

Pagination:

```text
GET /api/v1/jobs?page=2&limit=20
```

Combined:

```text
GET /api/v1/jobs
  ?search=backend
  &location=India
  &remote=true
  &sort=salary
  &order=desc
  &page=2
  &limit=20
```

Now we're getting somewhere.

---

# 40. But Don't Let Your API Become a Mess

You might eventually end up with:

```text
/jobs?foo=x&bar=y&sort=z&order=q&...
```

with 40 different parameters.

At some point, you need to ask:

> Is this API still understandable?

API design is about balancing flexibility with simplicity.

Don't expose every possible database filter just because you can.

Design around actual product requirements.

---

# 41. Validate Everything

Suppose the client sends:

```http
GET /jobs?page=-999&limit=999999&sort=DROP_TABLE
```

Your API should not blindly accept it.

Think:

```text
Request
  ↓
Parse
  ↓
Validate
  ↓
Normalize
  ↓
Build query
  ↓
Database
```

Not:

```text
Request
  ↓
Whatever client sent
  ↓
Database
```

---

# 42. Normalize Input

Sometimes different inputs should mean the same thing.

For example:

```text
remote=true
```

should become a boolean:

```text
true
```

instead of leaving everything as raw strings.

Similarly:

```text
page="2"
```

should become:

```text
2
```

after validation.

Your backend should convert external input into the types your application expects.

---

# 43. Don't Trust Client-Supplied Totals

Suppose the client sends:

```json
{
  "total": 1000000
}
```

That doesn't mean there are actually one million records.

The server/database determines authoritative data.

This sounds obvious, but the broader lesson is:

> **Client input is untrusted.**

Never let a client tell your backend:

```text
"userId = someone else"
"role = admin"
"price = 1"
"ownerId = another user"
```

without authorization and validation.

---

# 44. Large Datasets Change How You Think

At 100 records:

```text
SELECT * FROM jobs;
```

might seem fine.

At 10 million records:

```text
SELECT * FROM jobs;
```

is a completely different problem.

Now you think about:

```text
Indexes
Pagination
Query plans
Selected columns
Sorting
Filtering
Connection limits
Caching
Database load
```

This is the transition from:

> "Can I make it work?"

to:

> "Will this still work when the data grows?"

That's an important engineering mindset.

---

# 45. A Practical API Request Pipeline

A mature endpoint might look something like:

```text
Client
  ↓
HTTPS
  ↓
Authentication
  ↓
Authorization
  ↓
Parse request
  ↓
Validate input
  ↓
Apply defaults
  ↓
Business rules
  ↓
Database query
  ↓
Pagination
  ↓
Response formatting
  ↓
HTTP response
```

And around all of this:

```text
Logging
Monitoring
Rate limiting
Error handling
```

Now you can see how all the previous chapters connect.

---

# 46. Common Mistakes

## Mistake 1 — Returning everything

```http
GET /users
```

doesn't mean:

> Return every user in the database.

Use pagination.

---

## Mistake 2 — No maximum page size

Don't let:

```text
limit=999999999
```

destroy your database.

---

## Mistake 3 — Blindly trusting query parameters

Validate them.

---

## Mistake 4 — Allowing arbitrary sorting

Use an allowlist of sortable fields.

---

## Mistake 5 — Exposing database errors

Log detailed errors internally.

Return safe errors to clients.

---

## Mistake 6 — Breaking API clients

Treat API responses as contracts.

---

## Mistake 7 — Using pagination without stable ordering

Especially important for cursor pagination.

---

## Mistake 8 — Assuming timeout means failure

For operations with side effects, the server or downstream system may have completed the operation even if the response timed out.

---

## Mistake 9 — Making API responses unnecessarily complicated

Consistency matters more than having a fancy response format.

---

## Mistake 10 — Treating versioning as the solution to bad design

Versioning doesn't fix a poorly designed API.

Think carefully about the contract from the beginning.

---

# 47. Mini Project — Build a Real Job API

Now let's turn our simple CRUD API into something closer to a production-style API.

Create:

```text
GET /api/v1/jobs
GET /api/v1/jobs/:id
POST /api/v1/jobs
PATCH /api/v1/jobs/:id
DELETE /api/v1/jobs/:id
```

Add:

```text
✓ Pagination
✓ Filtering
✓ Sorting
✓ Search
✓ Validation
✓ Consistent errors
✓ API versioning
```

---

# 48. Requirements

Your:

```http
GET /api/v1/jobs
```

should support:

```text
?page=1
&limit=20
&location=India
&remote=true
&search=backend
&sort=createdAt
&order=desc
```

You don't need to implement every possible filter.

Start with:

```text
search
location
remote
```

and:

```text
page
limit
sort
order
```

---

# 49. Add Limits

For example:

```text
Default:
limit = 20

Maximum:
limit = 100
```

If someone sends:

```text
limit=5000
```

don't query 5,000 records just because they asked.

Return an error or clamp it according to your API design.

Be explicit and consistent.

---

# 50. Add Cursor Pagination

After implementing offset pagination, try cursor pagination.

For example:

```http
GET /api/v1/jobs?limit=20
```

Response:

```json
{
  "data": [
    ...
  ],
  "pagination": {
    "nextCursor": "eyJpZCI6MTAwfQ=="
  }
}
```

Then:

```http
GET /api/v1/jobs?limit=20&cursor=eyJpZCI6MTAwfQ==
```

The exact cursor format is up to you.

Don't put sensitive information into cursors unnecessarily.

---

# 51. Try Breaking Your Own API

This is important.

Don't just test:

```text
GET /jobs
```

Test:

```text
/jobs?limit=-1
/jobs?limit=999999
/jobs?page=-100
/jobs?sort=randomField
/jobs?order=random
/jobs?remote=hello
/jobs?search=
```

Ask:

> What does my API do?

A backend engineer should think about invalid input as much as valid input.

---

# 52. Interview Questions

## 1. Why do we need pagination?

To avoid fetching, processing, transferring, and returning unnecessarily large datasets.

It improves API performance and makes large collections manageable for clients.

---

## 2. Offset vs cursor pagination?

Offset pagination uses a page/offset to skip records.

Cursor pagination uses a position from the previous result to retrieve the next set.

Cursor pagination can be more suitable for large, frequently changing datasets, while offset pagination is often simpler and works well for many traditional list interfaces.

---

## 3. Why put filters in query parameters?

Filters usually modify how a collection is retrieved rather than identifying a specific resource.

For example:

```http
GET /jobs?location=India
```

---

## 4. Why should APIs validate query parameters?

Because client input is untrusted.

Invalid values can cause incorrect behavior, expensive queries, bugs, or security problems.

---

## 5. What is API versioning?

A way of allowing an API to evolve while maintaining compatibility for existing clients.

---

## 6. What is a breaking API change?

A change that can cause an existing client to stop working correctly.

Examples include removing a field, renaming a field, changing its type, or changing behavior clients depend on.

---

## 7. What is an idempotency key?

A unique key that allows a server to recognize retries of the same logical operation and avoid accidentally performing the same side effect multiple times.

---

## 8. Why are idempotency keys useful for payment APIs?

Because network failures can make a client unsure whether a payment request succeeded.

A retry without idempotency protection could create duplicate payments.

---

## 9. Why shouldn't we allow arbitrary sort fields from users?

Because blindly inserting user-controlled values into database queries can cause security and correctness problems.

Use an allowlist of valid sortable fields.

---

## 10. What happens if an API returns millions of records?

The API may consume excessive memory, CPU, database resources, and network bandwidth.

Use pagination, appropriate filtering, efficient queries, and only return the data the client actually needs.

---

# 53. Interview Scenario

> You have an API:

```http
GET /users
```

It works perfectly with 1,000 users.

Your company now has 20 million users.

What would you change?

Don't immediately say:

> "Add Redis."

That's not necessarily the first problem.

Think:

```text
1. Don't return all users
        ↓
2. Add pagination
        ↓
3. Add sensible maximum page size
        ↓
4. Select only required fields
        ↓
5. Add appropriate indexes
        ↓
6. Inspect query performance
        ↓
7. Consider cursor pagination if appropriate
        ↓
8. Consider caching only if measurements show it's useful
```

This is a much better engineering answer.

---

# 54. Another Interview Scenario

> Your frontend says users are seeing duplicate items when scrolling through a feed. The backend uses offset pagination and new posts are constantly being created. What could be happening?

Think about this:

```text
Initial data:

A
B
C
D
E
F
```

Client requests:

```text
page 1
```

gets:

```text
A B C
```

Then a new item appears:

```text
X
A
B
C
D
E
F
```

The client asks for:

```text
page 2
```

and might now get:

```text
C D E
```

The client saw:

```text
C
```

twice.

The changing dataset shifted the offset.

Possible approaches include cursor-based pagination with stable ordering.

The important thing isn't:

> "Cursor pagination is always better."

It's:

> **Understand how your pagination strategy behaves when the underlying data changes.**

---

# 55. Another Interview Scenario

> Your API supports:

```http
GET /jobs?sort=salary
```

A developer notices that users can send:

```http
GET /jobs?sort=some_internal_database_expression
```

What should you do?

Don't blindly insert the value into SQL.

Instead:

```text
Client
  ↓
sort=salary
  ↓
Validate against allowlist
  ↓
salary → safe internal mapping
  ↓
Database
```

For example:

```text
salary → jobs.salary
createdAt → jobs.created_at
title → jobs.title
```

The client chooses from allowed options.

It doesn't get to construct your SQL.

---

# 56. The Bigger Picture

Look at how much our backend has grown.

We started with:

```text
Client
   ↓
HTTP
   ↓
Server
   ↓
Database
```

Now:

```text
Client
   ↓
HTTPS
   ↓
Authentication
   ↓
Authorization
   ↓
Rate limiting
   ↓
Validation
   ↓
Routing
   ↓
Business logic
   ↓
Database
   ↓
Pagination
   ↓
Filtering
   ↓
Sorting
   ↓
Response
```

And around the system:

```text
Logging
Monitoring
Error handling
```

This is why backend engineering is much more than:

> "Create an Express route."

The route is only one small piece.

---

# 57. What You Should Know After This Chapter

You should now understand:

```text
Pagination
→ Don't return huge collections at once

Offset pagination
→ page + limit / offset

Cursor pagination
→ continue from a stable position

Filtering
→ narrow the result set

Sorting
→ control result ordering

Searching
→ find records based on query text

API versioning
→ evolve APIs without unexpectedly breaking clients

API contracts
→ clients depend on your request/response behavior

Validation
→ never trust client input

Idempotency
→ make retries safe for operations that can create side effects

Error responses
→ give clients useful, consistent information without leaking internals
```

But there's one concept I'd especially like you to remember:

> **An API is a contract.**

When you create:

```http
GET /api/v1/jobs
```

you're not just creating an endpoint.

You're creating something other developers and applications will depend on.

That's why API design deserves thought.

---

# 58. A Simple Checklist

Before shipping a collection endpoint, ask:

```text
□ Does it have pagination?

□ Is there a maximum page size?

□ Can users filter results?

□ Can users sort results?

□ Are query parameters validated?

□ Is the ordering stable?

□ Are database queries indexed appropriately?

□ Are we returning only the fields needed?

□ Are errors consistent?

□ Are internal errors hidden from clients?

□ Is the response format documented?

□ Could this API change break existing clients?

□ Do side-effecting operations need idempotency?

□ Are expensive requests protected?
```

You don't need every feature on every endpoint.

The point is to start asking the right questions.

---

# 59. Final Mental Model

When designing an API, don't start with:

> "What endpoint should I create?"

Start with:

```text
Who is consuming this API?
        ↓
What data do they need?
        ↓
How much data could exist?
        ↓
How will they search/filter it?
        ↓
How will they paginate it?
        ↓
How might the API evolve?
        ↓
What happens when something fails?
        ↓
What happens when requests are retried?
        ↓
How do I prevent expensive or invalid requests?
```

That's the shift from:

> **building an API**

to:

> **designing an API.**

And that's exactly the skill we're trying to build throughout this repository.

---

# What's Next?

We've now covered a lot of the application layer.

Our backend can:

```text
Receive HTTP requests
        ↓
Route them
        ↓
Authenticate users
        ↓
Authorize actions
        ↓
Validate input
        ↓
Query PostgreSQL
        ↓
Return useful API responses
```

But there's a problem.

Imagine:

```text
GET /jobs
```

is requested **100,000 times**.

Every request hits PostgreSQL.

The same expensive query runs again.

And again.

And again.

That's wasteful.

What if we could keep frequently requested data somewhere much faster?

Something like:

```text
Client
   ↓
Cache
   ↓
Found?
 ┌─┴─┐
Yes  No
 ↓    ↓
Return Database
       ↓
     Store in cache
```

That's where we're going next.

# Next — Chapter 13: Caching

We'll learn:

```text
What caching actually is
Why Redis is fast
Cache-aside
Write-through caching
TTL
Cache invalidation
Cache stampede
Stale data
Cache keys
When NOT to cache
```

And most importantly:

> **When does caching actually help, and when are you just adding another system for no reason?**