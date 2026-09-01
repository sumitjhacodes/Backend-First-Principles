# Chapter 17 — Logging, Monitoring & Error Handling

Your backend is running.

Everything looks fine.

Then a user sends you a message:

> "The API is broken."

You check it locally.

Works.

You check again.

Still works.

Then another user says:

> "Sometimes the API takes 5 seconds."

Another:

> "My payment went through but the response failed."

Another:

> "The image processing never completed."

Now what?

You can't just SSH into the server and start guessing.

You need evidence.

You need to know:

```text
What happened?
When did it happen?
Which user was affected?
Which request caused it?
How long did it take?
Which service failed?
What was the error?
How often is it happening?
```

That's what logging, monitoring, and error handling help us answer.

---

# 1. The Three Things We Need

There are three related but different ideas:

```text
Logging
Monitoring
Error Handling
```

Think of them like this.

### Logging

> **"Tell me what happened."**

### Monitoring

> **"Tell me whether the system is healthy."**

### Error handling

> **"Tell me what to do when something goes wrong."**

They work together.

---

# 2. A Simple Example

Suppose this request happens:

```http
POST /api/orders
```

Something goes wrong while saving the order.

### Error handling

Your application catches the error:

```text
Database failed
```

and returns:

```http
500 Internal Server Error
```

### Logging

The server records:

```text
POST /api/orders
user=123
error=database_timeout
```

### Monitoring

Your monitoring system notices:

```text
5xx errors increased from 1% → 15%
```

and alerts you.

Now you have:

```text
Application
   ↓
Error handling
   ↓
Logging
   ↓
Monitoring
   ↓
Alert
```

That's the production feedback loop.

---

# 3. Why `console.log()` Isn't Enough

When you're learning Node.js, you might write:

```js
console.log("user created");
```

That's perfectly fine while learning.

But imagine production has:

```text
50 servers
100,000 requests/minute
```

Your logs might look like:

```text
user created
request received
something failed
user created
request received
something failed
```

Good luck figuring out which request caused which error.

Production logs need structure and context.

---

# 4. What Is a Log?

A log is simply a record of something that happened in your application.

For example:

```text
User 123 logged in
```

or:

```text
POST /api/orders completed in 120ms
```

or:

```text
Payment service returned 503
```

Logs help you reconstruct what happened.

---

# 5. Log Levels

Most logging systems have different levels.

Common ones are:

```text
DEBUG
INFO
WARN
ERROR
FATAL
```

Let's understand them.

---

# 6. DEBUG

Used for detailed information useful while debugging.

Example:

```text
DEBUG:
Checking cache for user 123
```

You might not want these logs enabled everywhere in production because they can become very noisy.

---

# 7. INFO

Normal application events.

For example:

```text
INFO:
Server started on port 3000
```

or:

```text
INFO:
Order created successfully
```

Think:

> "Everything is behaving normally, but this event is worth recording."

---

# 8. WARN

Something unusual happened, but the application can continue.

For example:

```text
WARN:
Redis unavailable, falling back to database
```

The system is still working.

But you want someone to know.

---

# 9. ERROR

Something failed.

For example:

```text
ERROR:
Failed to process payment
```

An error doesn't necessarily mean the entire application is down.

Maybe one request failed.

---

# 10. FATAL

A serious failure where the application cannot continue safely.

For example:

```text
FATAL:
Unable to initialize database connection
```

Your application might need to stop.

Not every application uses a `FATAL` level, but the concept is useful.

---

# 11. Structured Logging

This is one of the most useful production concepts.

Instead of:

```text
User 123 created order 456
```

you might log structured data:

```json
{
  "level": "info",
  "event": "order_created",
  "userId": "123",
  "orderId": "456"
}
```

Now machines can understand the log too.

You can search:

```text
orderId = 456
```

or:

```text
userId = 123
```

much more easily.

---

# 12. Why JSON Logs?

Imagine you have:

```text
10 million logs
```

You don't want humans manually reading all of them.

A logging system can search structured fields.

For example:

```text
level = "error"
service = "payment-service"
statusCode = 500
```

That's much easier when the data has a consistent structure.

---

# 13. What Should You Log?

This is where beginners often go wrong.

You don't need to log everything.

Useful things include:

```text
Request started
Request completed
Request duration
User/action context
Important business events
External service calls
Database errors
Authentication failures
Unexpected exceptions
Background job failures
```

For example:

```json
{
  "event": "order_created",
  "orderId": "ord_123",
  "userId": "user_456"
}
```

---

# 14. What Should You NOT Log?

Be very careful with sensitive information.

Don't casually log:

```text
Passwords
Access tokens
Refresh tokens
API keys
Credit card numbers
Private secrets
```

Bad:

```js
logger.info({
  password: req.body.password
});
```

Never do this.

Logs can end up in:

```text
Cloud systems
Log aggregators
Developer dashboards
Backups
```

Treat logs as sensitive infrastructure.

---

# 15. Logging Requests

A useful API log might look like:

```json
{
  "level": "info",
  "method": "GET",
  "path": "/api/users/123",
  "statusCode": 200,
  "durationMs": 82
}
```

Now you can answer:

```text
Which endpoint?
What status?
How long?
```

without guessing.

---

# 16. Request IDs

Now let's solve a bigger problem.

Imagine a request goes through:

```text
Client
 ↓
API Gateway
 ↓
Backend
 ↓
Redis
 ↓
Payment Service
 ↓
Database
```

That's a lot of logs.

How do we know which logs belong to the same request?

We can attach a:

> **Request ID**

For example:

```text
requestId = req_abc123
```

Then every related log includes:

```text
req_abc123
```

---

# 17. Request ID Example

API server:

```json
{
  "requestId": "req_abc123",
  "event": "request_started"
}
```

Database:

```json
{
  "requestId": "req_abc123",
  "event": "database_query_started"
}
```

Payment service:

```json
{
  "requestId": "req_abc123",
  "event": "payment_provider_called"
}
```

Payment response:

```json
{
  "requestId": "req_abc123",
  "event": "payment_provider_failed"
}
```

Now you can search:

```text
req_abc123
```

and reconstruct the request.

This becomes extremely useful in distributed systems.

---

# 18. Correlation IDs

You may also hear:

> **Correlation ID**

It is closely related to request IDs.

The general idea is:

> Give related operations an identifier so you can trace them across services.

For a simple backend:

```text
requestId
```

may be enough.

In a larger distributed system, you may have:

```text
traceId
spanId
correlationId
```

We'll get deeper into tracing later.

For now, understand the purpose:

```text
One request
 ↓
many operations
 ↓
one identifier
```

---

# 19. Error Handling

Let's talk about the other half.

Suppose this code runs:

```js
const user = await db.user.findUnique({
  where: { id }
});

console.log(user.name);
```

What if:

```text
user = null
```

Then:

```text
user.name
```

throws an error.

Your application needs to handle failures intentionally.

---

# 20. Expected vs Unexpected Errors

This distinction is useful.

### Expected error

User asks for:

```text
GET /users/999
```

but user doesn't exist.

That's not necessarily a server failure.

You might return:

```http
404 Not Found
```

### Unexpected error

Your database suddenly crashes.

That's a server-side failure.

You might return:

```http
500 Internal Server Error
```

Don't treat every error as the same thing.

---

# 21. HTTP Error Responses

Your API should return appropriate status codes.

For example:

```text
400 → Bad Request
401 → Unauthorized
403 → Forbidden
404 → Not Found
409 → Conflict
422 → Unprocessable Content
429 → Too Many Requests
500 → Internal Server Error
502 → Bad Gateway
503 → Service Unavailable
```

The exact choice depends on the situation.

But don't return:

```text
200 OK
```

for everything.

---

# 22. Don't Expose Internal Errors

Bad:

```json
{
  "error": "PrismaClientKnownRequestError: 
  database connection failed at /app/src/services/user.ts:42"
}
```

You're exposing internal implementation details.

Instead:

```json
{
  "error": {
    "code": "INTERNAL_ERROR",
    "message": "Something went wrong"
  }
}
```

Log the detailed error internally.

Return a safe message to the client.

---

# 23. One Error Format

Your API should ideally have a consistent error response.

For example:

```json
{
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "User not found"
  }
}
```

Another:

```json
{
  "error": {
    "code": "INVALID_EMAIL",
    "message": "Please provide a valid email"
  }
}
```

Consistency makes frontend development easier.

---

# 24. Custom Error Classes

In Node.js applications, you can create application-specific errors.

For example:

```js
class AppError extends Error {
  constructor(message, statusCode, code) {
    super(message);

    this.statusCode = statusCode;
    this.code = code;
  }
}
```

Then:

```js
throw new AppError(
  "User not found",
  404,
  "USER_NOT_FOUND"
);
```

Your error middleware can handle it.

---

# 25. Centralized Error Handling

Don't write this everywhere:

```js
try {
  ...
} catch (error) {
  res.status(500).json(...);
}
```

A better approach is to have centralized error handling.

Conceptually:

```text
Request
   ↓
Route
   ↓
Service
   ↓
Error
   ↓
Error Middleware
   ↓
HTTP Response
```

For Express:

```js
app.use((err, req, res, next) => {
  // handle error
});
```

Now your application has one place to deal with errors consistently.

---

# 26. Async Errors

Backend applications use asynchronous operations constantly:

```js
await database.query();
await redis.get();
await sendEmail();
await callPaymentAPI();
```

Any of these can fail.

So your application needs a consistent strategy for propagating asynchronous errors to your error handler.

Don't leave rejected promises unhandled.

---

# 27. Error Boundaries

A useful mental model:

```text
Controller
   ↓
Service
   ↓
Repository
   ↓
Database
```

Suppose the database throws:

```text
Connection timeout
```

You don't necessarily want every lower layer to send an HTTP response.

Instead:

```text
Database
   ↓
throws error
   ↓
Service
   ↓
propagates/handles
   ↓
Controller
   ↓
Error middleware
   ↓
HTTP response
```

Your lower layers should generally not need to know about HTTP.

This keeps responsibilities separated.

---

# 28. Logging the Error

When an error occurs, log useful context.

For example:

```json
{
  "level": "error",
  "event": "database_query_failed",
  "requestId": "req_123",
  "userId": "user_456",
  "operation": "create_order",
  "error": "connection_timeout"
}
```

Don't just log:

```text
ERROR!!!
```

Give yourself enough information to investigate.

---

# 29. Stack Traces

When an unexpected error happens:

```js
try {
  ...
} catch (error) {
  console.error(error);
}
```

you'll often get a stack trace.

Something like:

```text
Error: Database connection failed
    at createOrder (...)
    at processOrder (...)
    at ...
```

The stack trace helps you understand where the error originated.

In production error tracking systems, stack traces are extremely useful.

---

# 30. Error Tracking

Logging is useful.

But sometimes you want a dedicated system for application exceptions.

Tools such as:

> Sentry

can collect application errors and provide information such as:

```text
Error
Stack trace
Request information
Environment
Release/version
Frequency
Affected users
```

The goal isn't just:

> "An error happened."

It's:

> "This error started after deployment version X and affected 3.2% of requests."

---

# 31. Logs vs Error Tracking

They're related but not identical.

### Logs

Tell you:

```text
What happened
```

across normal and abnormal application events.

### Error tracking

Focuses heavily on:

```text
Exceptions
Stack traces
Affected users
Error frequency
```

You generally want both in a production system.

---

# 32. Monitoring

Now we get to:

> **Monitoring**

Logging tells you what happened.

Monitoring tells you whether the system is healthy.

For example:

```text
CPU
Memory
Request rate
Error rate
Latency
Database connections
Queue depth
Disk usage
```

You collect measurements and watch them over time.

---

# 33. Metrics

A metric is a numerical measurement over time.

For example:

```text
requests_per_second = 1,240
```

or:

```text
error_rate = 0.7%
```

or:

```text
average_latency = 120ms
```

Unlike logs, metrics are usually designed for aggregation and trend analysis.

---

# 34. Three Important Observability Signals

You'll often hear:

```text
Logs
Metrics
Traces
```

These are often called the three pillars of observability.

Very roughly:

### Logs

> What happened?

### Metrics

> How much/how often?

### Traces

> Where did this request spend its time?

---

# 35. What Is Observability?

Monitoring asks:

> "Is the system healthy?"

Observability is broader.

It means having enough information from system outputs to understand what's happening internally.

A simple mental model:

```text
System
 ↓
Logs
Metrics
Traces
 ↓
Understand behavior
```

Observability becomes especially important when systems become distributed.

---

# 36. Request Rate

One important metric:

```text
Requests per second
```

For example:

```text
100 req/sec
```

Then suddenly:

```text
5,000 req/sec
```

Maybe something changed.

Traffic could have increased.

A bot could be attacking you.

A client might have an accidental retry loop.

Monitoring helps you notice the change.

---

# 37. Error Rate

Suppose:

```text
10,000 requests
```

and:

```text
100 failed
```

Error rate:

```text
100 / 10,000 = 1%
```

If it suddenly becomes:

```text
15%
```

something is probably wrong.

This is often more useful than simply watching CPU.

---

# 38. Latency

Latency means:

> How long did the request take?

Example:

```text
GET /users
```

takes:

```text
80ms
```

That's latency.

But averages can hide problems.

Suppose:

```text
99 requests = 50ms
1 request = 10 seconds
```

Average latency might not tell the full story.

That's why percentiles matter.

---

# 39. P50, P95, P99

These are latency percentiles.

### P50

50% of requests are faster than this value.

Think:

> Typical request.

### P95

95% of requests are faster than this value.

The slowest 5% are slower.

### P99

99% of requests are faster than this value.

The slowest 1% are slower.

---

# 40. Why P99 Matters

Suppose your API has:

```text
P50 = 50ms
P95 = 100ms
P99 = 2 seconds
```

Most users are fine.

But 1% of requests are very slow.

At:

```text
1 million requests/day
```

1% means:

```text
10,000 requests
```

That's not a tiny number anymore.

This is why senior engineers don't look only at average latency.

---

# 41. The Famous Interview Scenario

Here's a common backend interview question:

> Your API normally responds in 80ms. P99 suddenly jumps to 2 seconds. CPU and memory are normal. What could be happening?

Don't say:

> "Increase the server size."

CPU and memory aren't the problem.

Think about dependencies.

Possibilities include:

```text
Database latency
Redis latency
External API latency
Network problems
Connection pool exhaustion
Lock contention
Slow queries
DNS issues
Queue delays
Load balancer issues
```

This is why monitoring needs more than CPU graphs.

---

# 42. Database Metrics

Your application may look healthy while the database is struggling.

Monitor things like:

```text
Query latency
Connection count
Connection pool usage
Slow queries
Locks
Deadlocks
CPU
Disk I/O
Replication lag
```

For example:

```text
API latency ↑
Database query latency ↑
```

Now you've narrowed down the problem.

---

# 43. Connection Pool Exhaustion

Remember connection pooling from Chapter 21?

Suppose your application has:

```text
Pool size = 20
```

and all 20 connections are busy.

New requests wait.

So you might see:

```text
CPU = normal
Memory = normal
Database = healthy
API latency = high
```

Why?

Because requests are waiting for a database connection.

This is a perfect example of why system metrics need context.

---

# 44. Queue Metrics

We learned queues in Chapter 14.

Monitor:

```text
Queue depth
Job processing time
Failed jobs
Retry count
Oldest job age
```

Suppose:

```text
Queue depth:
100
```

then:

```text
10,000
```

Your workers may not be processing jobs quickly enough.

The application might still accept HTTP requests normally.

But background processing is falling behind.

---

# 45. Health Checks

Your application can expose an endpoint such as:

```http
GET /health
```

which returns:

```json
{
  "status": "ok"
}
```

This can tell a load balancer or platform:

> "This application process is alive."

But be careful.

---

# 46. Liveness vs Readiness

Two useful concepts:

### Liveness

> Is the process alive?

Example:

```text
GET /health/live
```

### Readiness

> Is the process ready to receive traffic?

Example:

```text
GET /health/ready
```

Suppose:

```text
Node.js process = alive
Database connection = unavailable
```

The process may be alive but not ready to serve normal requests.

Separating these concepts can help orchestrators and load balancers make better decisions.

---

# 47. Don't Make Health Checks Too Complicated

A common mistake is:

```text
GET /health
 ↓
Database
 ↓
Redis
 ↓
Kafka
 ↓
External API
 ↓
Another service
```

Now your health check fails because one unrelated dependency is temporarily down.

Whether that's correct depends on what the health check is supposed to represent.

Keep the semantics clear:

```text
Liveness
=
Is this process alive?

Readiness
=
Can this instance safely receive traffic?
```

---

# 48. Alerts

Monitoring is useless if nobody reacts to it.

An:

> **Alert**

means:

> "Something crossed a threshold or condition that deserves attention."

Examples:

```text
Error rate > 5%
P99 latency > 1 second
Disk usage > 90%
Queue depth > 100,000
Database connections exhausted
```

---

# 49. Don't Alert on Everything

If your system sends:

```text
500 alerts/day
```

developers start ignoring them.

This is called:

> **Alert fatigue**

Good alerts should represent something that requires action.

Bad:

```text
CPU = 51%
```

when that has no operational consequence.

Better:

```text
API error rate > 10% for 5 minutes
```

if that actually indicates a problem.

---

# 50. Alert Severity

You can have:

```text
Warning
Critical
```

For example:

```text
Warning:
Queue growing

Critical:
Queue growing + workers failing
```

The goal is to distinguish:

```text
"Keep an eye on this."

from:

"Someone needs to investigate this now."
```

---

# 51. External Dependencies

Your application isn't alone.

You may depend on:

```text
Stripe
AWS
Google APIs
Email provider
Redis
PostgreSQL
Another internal service
```

If one dependency becomes slow:

```text
Your API
 ↓
External API
 ↓
slow
```

your API can become slow too.

Monitor dependency:

```text
latency
error rate
timeouts
availability
```

---

# 52. Timeouts

Never let external requests wait forever.

Bad:

```text
await paymentProvider.charge();
```

with no reasonable timeout strategy.

Imagine the provider never responds.

Your request may remain stuck.

Then:

```text
Requests accumulate
 ↓
Connections remain occupied
 ↓
Resources are exhausted
```

A timeout is a way of saying:

> "I won't wait forever."

---

# 53. Retries

Suppose an external API fails temporarily.

You might retry.

For example:

```text
Attempt 1 → failed
Attempt 2 → failed
Attempt 3 → success
```

But retries can also make things worse.

Imagine:

```text
External API
already overloaded
```

and your service sends:

```text
10,000 retries
```

Now you've increased the load.

Retries need:

```text
Limited attempts
Backoff
Jitter
Idempotency where needed
```

---

# 54. Exponential Backoff

Instead of:

```text
Retry immediately
Retry immediately
Retry immediately
```

you might wait:

```text
1 second
2 seconds
4 seconds
8 seconds
```

This is called:

> **Exponential backoff**

Usually some randomness ("jitter") is also added so many clients don't retry at exactly the same moment.

---

# 55. Retry the Right Errors

Not every error should be retried.

For example:

```text
401 Unauthorized
```

retrying the same request probably won't fix anything.

But:

```text
temporary network failure
```

might be retryable.

Always ask:

> Is this failure likely to be temporary?

And:

> Is retrying safe?

---

# 56. Idempotency and Retries

Remember idempotency from Chapter 2?

This becomes extremely important here.

Imagine:

```http
POST /payments
```

The server processes the payment.

But the response gets lost.

Client thinks:

```text
Payment failed
```

and retries.

Now you could accidentally charge the customer twice.

That's why payment APIs often use:

```text
Idempotency-Key
```

For example:

```text
Idempotency-Key: payment_abc123
```

The server can recognize:

> "I've already processed this operation."

This is a beautiful example of concepts from different chapters connecting together.

---

# 57. Circuit Breaker

Another production concept is:

> **Circuit breaker**

Imagine your service repeatedly calls:

```text
Payment Service
```

but the payment service is completely down.

Instead of sending thousands of requests that all fail, your system can temporarily stop calling it.

Conceptually:

```text
Normal
  ↓
Failures increase
  ↓
Circuit opens
  ↓
Requests fail fast
  ↓
After some time
  ↓
Try again
```

This prevents one failing dependency from consuming all your resources.

---

# 58. Graceful Degradation

Sometimes the entire application doesn't need to fail just because one feature is broken.

Example:

```text
Recommendation Service
```

is down.

Could your e-commerce site still show:

```text
Product page
Cart
Checkout
```

maybe with:

```text
"No recommendations available"
```

instead of:

```text
500 Internal Server Error
```

That's:

> **Graceful degradation**

Think:

> "What can still work if one dependency fails?"

---

# 59. Fail Fast

Sometimes failing quickly is better than waiting.

Suppose:

```text
External service timeout = 30 seconds
```

and you have:

```text
1,000 concurrent requests
```

You could end up with lots of requests stuck waiting.

A shorter timeout can allow the system to recover resources faster.

But don't pick timeouts randomly.

Base them on actual system behavior and requirements.

---

# 60. Distributed Tracing

Now let's return to:

```text
requestId
```

Imagine:

```text
Client
 ↓
API Gateway
 ↓
Order Service
 ↓
Payment Service
 ↓
Database
```

The entire operation takes:

```text
2 seconds
```

Where did the 2 seconds go?

Tracing helps answer this.

---

# 61. Trace and Spans

A:

> **Trace**

represents the journey of one request through a distributed system.

Inside it are:

> **Spans**

For example:

```text
Trace: req_123

├── API Gateway       20ms
├── Order Service     50ms
├── Payment Service   1,800ms
└── Database          100ms
```

Now it's obvious:

```text
Payment Service
```

is the bottleneck.

---

# 62. Logs + Metrics + Traces

These complement each other.

### Metrics

```text
P99 latency = 2 sec
```

tells you:

> Something is slow.

### Trace

```text
Payment Service = 1.8 sec
```

tells you:

> Where it's slow.

### Logs

```text
Payment provider timeout
```

tells you:

> What happened.

Together:

```text
Metrics
   ↓
Notice problem

Traces
   ↓
Locate problem

Logs
   ↓
Understand problem
```

That's a powerful debugging workflow.

---

# 63. Error Monitoring Doesn't Replace Logging

You might use an error tracking system.

But don't think:

```text
Sentry
=
all observability
```

You still need:

```text
Logs
Metrics
Traces
```

depending on your system.

Each gives you different information.

---

# 64. Production Debugging Example

Let's say someone reports:

> "Checkout is randomly slow."

You don't start changing code immediately.

First:

```text
1. Check error rate
2. Check request rate
3. Check P50/P95/P99 latency
4. Check database latency
5. Check external dependency latency
6. Check queue depth
7. Check connection pools
8. Inspect traces
9. Search logs using request IDs
10. Identify the bottleneck
```

This is a much better engineering process than:

```text
"Maybe Redis is slow?"
```

and changing random things.

---

# 65. A Practical Example

Suppose you discover:

```text
P99:
80ms → 2s
```

CPU:

```text
40%
```

Memory:

```text
50%
```

Database:

```text
normal
```

Redis:

```text
normal
```

Then tracing shows:

```text
Payment API
 ↓
1.8 seconds
```

Logs show:

```text
payment_provider_timeout
```

Now the problem is much clearer.

You might:

```text
Reduce timeout
Add controlled retries
Add circuit breaker
Investigate provider
Provide graceful failure
```

That's production debugging.

---

# 66. Monitoring a Background Worker

Suppose we have:

```text
API
 ↓
Queue
 ↓
Worker
 ↓
Email Provider
```

The API may be perfectly healthy.

But the queue grows:

```text
100
500
2,000
10,000
```

Monitoring should tell us:

```text
Queue depth ↑
Job processing time ↑
Failed jobs ↑
```

Now we know:

> Background processing is falling behind.

This is why monitoring should cover more than HTTP servers.

---

# 67. Monitoring WebSockets

From Chapter 15, we know WebSockets maintain persistent connections.

Useful metrics might include:

```text
Active connections
Connection rate
Disconnect rate
Reconnect rate
Messages/sec
Message errors
Connection duration
```

Imagine:

```text
Active connections:
100,000

Suddenly:
20,000 disconnects
```

That's worth investigating.

Maybe:

```text
Server restarted
Network issue
Load balancer issue
Deployment problem
```

---

# 68. Monitoring File Uploads

From Chapter 16:

```text
Upload
 ↓
S3
 ↓
Queue
 ↓
Worker
```

Monitor:

```text
Upload failures
Average upload duration
Processing duration
Queue depth
Failed processing jobs
Storage errors
```

Now if users say:

> "My image is stuck."

you can investigate:

```text
Upload succeeded?
Queue received event?
Worker picked job?
Processing succeeded?
Database status updated?
```

Observability lets you answer these questions.

---

# 69. Monitoring Isn't Just Graphs

A common beginner misunderstanding:

> "Monitoring means looking at dashboards."

Not exactly.

Monitoring is about measuring system behavior and detecting meaningful problems.

A dashboard is just one way to visualize the information.

You also have:

```text
Alerts
Logs
Metrics
Health checks
Traces
Error reports
```

---

# 70. The Golden Signals

A commonly used monitoring model is the:

> **Four Golden Signals**

They are:

```text
Latency
Traffic
Errors
Saturation
```

### Latency

How long requests take.

### Traffic

How much demand the system receives.

### Errors

How many requests fail.

### Saturation

How close the system is to its capacity.

For example:

```text
CPU
Memory
Disk
Connection pools
Queue capacity
```

These four give you a useful high-level view of system health.

---

# 71. Saturation

This is an important one.

Suppose:

```text
CPU = 50%
```

Looks fine.

But:

```text
Database connection pool = 100%
```

Now you're saturated on database connections.

Or:

```text
Queue workers = 100% busy
```

Your queue may start growing.

Saturation is about:

> **How close are we to running out of an important resource?**

---

# 72. SLI

You'll eventually hear:

> **SLI — Service Level Indicator**

It's a measurement of actual service behavior.

Example:

```text
Percentage of successful requests
```

or:

```text
Percentage of requests completed under 300ms
```

It's the thing you measure.

---

# 73. SLO

> **SLO — Service Level Objective**

This is the target.

For example:

```text
99.9% of API requests should succeed.
```

or:

```text
99% of requests should complete within 500ms.
```

So:

```text
SLI
=
what we measure

SLO
=
what target we want
```

---

# 74. SLA

You may also hear:

> **SLA — Service Level Agreement**

This is typically a formal commitment between a service provider and customer.

For example:

```text
99.9% availability
```

could be contractually promised.

For interviews, remember:

```text
SLI
→ measurement

SLO
→ target

SLA
→ agreement/commitment
```

---

# 75. Error Budget

If your SLO is:

```text
99.9% availability
```

then you're allowed:

```text
0.1%
```

of unavailability/error within that objective.

This is the basic idea of an:

> **Error budget**

Instead of saying:

> "Everything must be perfect."

you acknowledge that some amount of failure is acceptable within the reliability target.

This becomes useful when balancing:

```text
Reliability
vs
Shipping new features
```

---

# 76. Don't Catch Everything and Ignore It

Bad:

```js
try {
  await saveUser();
} catch (error) {
  console.log(error);
}
```

Now the application may continue as if nothing happened.

You might have silently lost data.

Better:

```text
Catch
 ↓
Understand whether recoverable
 ↓
Log
 ↓
Handle or propagate
 ↓
Return appropriate response
```

Don't swallow errors blindly.

---

# 77. Retry + Logging

If you retry something:

```text
Attempt 1 failed
Attempt 2 failed
Attempt 3 succeeded
```

don't necessarily log all three as `ERROR` if the final operation succeeded.

You might record:

```text
WARN:
Temporary failure, retrying
```

and:

```text
INFO:
Operation succeeded after retry
```

Your logs should reflect the actual severity.

---

# 78. Don't Log the Same Error 10 Times

Sometimes an error gets logged at multiple layers:

```text
Repository logs error
Service logs error
Controller logs error
Middleware logs error
```

Now one failure produces:

```text
4 identical error logs
```

This creates noise.

Choose where the error gets logged, especially for errors that are simply being propagated upward.

---

# 79. Operational Context Matters

A production log should ideally tell you things like:

```text
timestamp
service
environment
requestId
event
severity
user/context where appropriate
duration
error
```

For example:

```json
{
  "timestamp": "2026-09-01T10:30:00Z",
  "level": "error",
  "service": "order-api",
  "environment": "production",
  "requestId": "req_123",
  "event": "create_order_failed",
  "durationMs": 850,
  "errorCode": "DB_TIMEOUT"
}
```

You don't need every field for every log.

But the principle is:

> Give future-you enough context to investigate.

---

# 80. Development vs Production Logs

In development you might want:

```text
Pretty logs
Detailed stack traces
Debug information
```

In production:

```text
Structured logs
Consistent fields
Useful context
Reduced noise
Safe error messages
```

Don't accidentally expose internal debugging information to users.

---

# 81. Log Rotation

Imagine your application produces:

```text
10 GB logs/day
```

and you keep everything on the server forever.

Eventually:

```text
Disk = 100%
```

and your application can start failing.

Production logging usually involves:

```text
Retention
Rotation
Centralized log storage
Compression
Deletion policies
```

You generally don't want your application server's disk becoming your permanent log database.

---

# 82. Centralized Logging

With multiple servers:

```text
Server A → logs
Server B → logs
Server C → logs
```

you don't want to SSH into every server individually.

Instead:

```text
Server A ─┐
Server B ─┼→ Central Log System
Server C ─┘
```

Now you can search all application logs in one place.

---

# 83. Metrics Aggregation

Same idea for metrics:

```text
Server A ─┐
Server B ─┼→ Metrics System
Server C ─┘
```

You can then see:

```text
Total request rate
Total error rate
P99 latency
```

across the entire service.

---

# 84. A Simple Production Stack

You don't need all of this on day one.

A typical ecosystem might contain:

```text
Application
   ↓
Structured Logger
   ↓
Log Aggregation

Application
   ↓
Metrics
   ↓
Monitoring Dashboard

Application
   ↓
Tracing
   ↓
Trace Backend

Application
   ↓
Errors
   ↓
Error Tracking
```

Different companies use different tools.

The concepts matter more than the specific product.

---

# 85. A Beginner-Friendly Node.js Example

Imagine:

```js
app.get("/users/:id", async (req, res, next) => {
  try {
    const start = Date.now();

    const user = await getUser(req.params.id);

    logger.info({
      event: "get_user_success",
      userId: req.params.id,
      durationMs: Date.now() - start
    });

    res.json(user);
  } catch (error) {
    next(error);
  }
});
```

Then centralized error middleware:

```js
app.use((err, req, res, next) => {
  logger.error({
    event: "request_failed",
    error: err.message,
    requestId: req.id
  });

  res.status(500).json({
    error: {
      code: "INTERNAL_ERROR",
      message: "Something went wrong"
    }
  });
});
```

This is simplified, but the architecture is important.

---

# 86. Add Request IDs

Middleware:

```js
app.use((req, res, next) => {
  req.id = crypto.randomUUID();
  next();
});
```

Then logs can include:

```js
logger.info({
  requestId: req.id,
  event: "request_started",
  method: req.method,
  path: req.path
});
```

Now you can search all logs for:

```text
requestId = abc123
```

and follow the request.

---

# 87. What About Sensitive User Information?

Be careful with:

```js
logger.info({
  user: req.user
});
```

Maybe `req.user` contains:

```text
email
tokens
roles
internal IDs
```

Log only what you actually need.

For example:

```js
logger.info({
  userId: req.user.id,
  event: "profile_updated"
});
```

Minimal useful context is usually better than dumping entire objects.

---

# 88. Common Mistakes

## Mistake 1 — Logging everything

Too much noise makes important events harder to find.

---

## Mistake 2 — Logging secrets

Never casually log:

```text
Passwords
Tokens
API keys
Private credentials
```

---

## Mistake 3 — Only watching CPU

A system can be unhealthy while CPU is completely normal.

Check:

```text
Latency
Errors
Database
Queues
Connections
External services
```

---

## Mistake 4 — Returning raw errors to users

Internal stack traces shouldn't become your public API response.

---

## Mistake 5 — Swallowing errors

Don't do:

```js
catch (error) {
  console.log(error);
}
```

and pretend nothing happened.

---

## Mistake 6 — No request IDs

Debugging distributed systems becomes much harder.

---

## Mistake 7 — No timeouts

A dependency that never responds can consume resources indefinitely.

---

## Mistake 8 — Blind retries

Retries can amplify an outage.

---

## Mistake 9 — Alerting on everything

Too many alerts create alert fatigue.

---

## Mistake 10 — Only monitoring the API

Your:

```text
Queue
Database
Redis
Workers
WebSockets
External APIs
```

can fail independently.

---

# 89. Interview Questions

## 1. What is logging?

Recording events and information about what happened inside an application or system.

---

## 2. What is structured logging?

Logging data in a consistent machine-readable structure, often JSON, so it can be searched and analyzed easily.

---

## 3. What are common log levels?

```text
DEBUG
INFO
WARN
ERROR
FATAL
```

The exact levels vary by logging system.

---

## 4. What should you avoid logging?

Sensitive information such as:

```text
Passwords
Tokens
API keys
Secrets
Payment information
```

unless there is an extremely specific, secure reason and appropriate redaction.

---

## 5. Why are request IDs useful?

They allow you to associate logs belonging to the same request, especially when a request travels through multiple services.

---

## 6. What is centralized logging?

Collecting logs from multiple servers/services into a central system where they can be searched and analyzed.

---

## 7. What is monitoring?

Collecting and observing system measurements to determine whether the system is healthy and detect problems.

---

## 8. What are metrics?

Numerical measurements collected over time, such as:

```text
Request rate
Error rate
Latency
CPU usage
Queue depth
```

---

## 9. What is the difference between logs, metrics, and traces?

```text
Logs
→ What happened?

Metrics
→ How much/how often?

Traces
→ Where did this request spend its time?
```

---

## 10. What are P50, P95, and P99?

Latency percentiles.

```text
P50 → median
P95 → 95% of requests are faster
P99 → 99% of requests are faster
```

---

## 11. Why is P99 useful?

Because averages can hide slow requests. P99 exposes behavior among the slowest portion of requests.

---

## 12. What is a health check?

An endpoint or mechanism used to determine whether an application instance is alive or ready to serve traffic.

---

## 13. Liveness vs readiness?

```text
Liveness
→ Is the process alive?

Readiness
→ Is it ready to receive traffic?
```

---

## 14. What are the four golden signals?

```text
Latency
Traffic
Errors
Saturation
```

---

## 15. What is an SLI?

A measurement of actual service behavior.

---

## 16. What is an SLO?

A target for an SLI.

Example:

```text
99.9% successful requests
```

---

## 17. What is an SLA?

A formal service-level agreement or commitment, often between a provider and customer.

---

## 18. What is graceful degradation?

Allowing parts of an application to continue working when a non-critical dependency or feature fails.

---

## 19. Why are timeouts important?

They prevent requests from waiting indefinitely for slow or unavailable dependencies and help protect system resources.

---

## 20. Why can retries be dangerous?

Retries increase traffic to a dependency that may already be failing and can amplify an outage. They should be limited and use appropriate backoff.

---

# 90. Interview Scenario

> Your API's average latency is 100ms, but users complain that it sometimes takes 5 seconds. What would you investigate?

Don't stop at average latency.

Look at:

```text
P95
P99
```

Then:

```text
Database latency
External API latency
Redis latency
Connection pool usage
Queue delays
Network issues
Lock contention
Slow queries
```

Then use:

```text
Trace
 ↓
Find slow component
 ↓
Logs
 ↓
Understand why
```

---

# 91. Interview Scenario

> CPU is only 30%, memory is 40%, but your API response time increased from 100ms to 2 seconds. What could be wrong?

CPU and memory aren't the only resources.

Possibilities:

```text
Database connection pool exhausted
Database query became slow
External API became slow
Network latency increased
Redis is slow
Lock contention
Thread/event-loop blocking
DNS problems
Load balancer problems
```

This is why:

> **"CPU looks normal" does not mean "the system is healthy."**

---

# 92. Interview Scenario

> Your payment API request times out, but the payment provider actually charged the customer. The client retries. What could happen?

Potentially:

```text
First request
→ payment succeeds
→ response lost

Retry
→ payment succeeds again
```

Now the customer may be charged twice.

You need an idempotency strategy.

For example:

```text
Idempotency-Key
```

so the payment provider/application can recognize duplicate attempts.

This connects:

```text
HTTP
+
Error handling
+
Retries
+
Distributed systems
+
Payments
```

---

# 93. Interview Scenario

> Your queue has 1,000 jobs. Suddenly it grows to 100,000. What do you investigate?

Look at:

```text
Worker count
Worker failures
Processing time
External dependencies
Retry rate
Job production rate
Queue consumer health
```

Maybe:

```text
Job production:
1,000/sec

Worker capacity:
500/sec
```

Then:

```text
Queue grows continuously
```

The problem isn't necessarily Redis or the queue itself.

Your consumers may simply not have enough processing capacity.

---

# 94. Interview Scenario

> A production error happens once every 10,000 requests. How would you debug it?

This is where good observability pays off.

You want:

```text
Request ID
Timestamp
Endpoint
User/context
Version
Stack trace
Input/context where safe
Database/external service information
```

Search your logs/error tracker.

Then ask:

```text
Does it happen for one endpoint?
One deployment?
One region?
One dependency?
One type of request?
One user?
```

Rare errors require context.

---

# 95. Mini Project

Take one of your previous projects.

For example:

```text
Todo API
```

or:

```text
Blog API
```

Add production-style observability.

---

# 96. Step 1 — Create a Logger

Use a logging library such as:

```text
Pino
```

or another structured logging library.

Create:

```text
src/lib/logger.ts
```

Then standardize:

```text
logger.info()
logger.warn()
logger.error()
```

---

# 97. Step 2 — Add Request Logging

For every request record:

```text
requestId
method
path
statusCode
duration
```

Example:

```json
{
  "event": "http_request",
  "requestId": "req_123",
  "method": "GET",
  "path": "/api/tasks",
  "statusCode": 200,
  "durationMs": 43
}
```

---

# 98. Step 3 — Add Centralized Error Handling

Create one error middleware.

It should:

```text
Catch unexpected errors
Log useful context
Return safe response
Use correct status code
```

---

# 99. Step 4 — Create Error Types

Create errors such as:

```text
NotFoundError
ValidationError
UnauthorizedError
ForbiddenError
ConflictError
```

Then map them to HTTP status codes.

For example:

```text
NotFoundError
→ 404

UnauthorizedError
→ 401

ForbiddenError
→ 403

ConflictError
→ 409
```

---

# 100. Step 5 — Add Health Endpoints

Create:

```http
GET /health/live
GET /health/ready
```

Keep their purpose clear.

---

# 101. Step 6 — Measure Latency

Track:

```text
Request duration
```

Then look at:

```text
P50
P95
P99
```

You don't need a huge observability platform just to understand the concept.

Even logging request duration is a good start.

---

# 102. Step 7 — Add Error Tracking

Connect your project to an error-tracking system.

The goal is to see:

```text
What error happened?
How often?
Which endpoint?
Which release?
Which users?
What stack trace?
```

---

# 103. Step 8 — Simulate Failures

This is important.

Don't just test the happy path.

Break things intentionally.

For example:

```text
Stop PostgreSQL
Stop Redis
Make an external API return 500
Make a request invalid
Send duplicate requests
Make a worker fail
```

Then see:

```text
Does the API return the correct response?

Did we log the error?

Did monitoring detect it?

Did the application recover?

Did retries make things worse?
```

This is how you actually learn production engineering.

---

# 104. Your Final Observability Architecture

By the end, think about your backend like this:

```text
                         Client
                           │
                           ↓
                    ┌─────────────┐
                    │ API Server  │
                    └──────┬──────┘
                           │
              ┌────────────┼─────────────┐
              ↓            ↓             ↓
            Logs         Metrics       Traces
              │            │             │
              ↓            ↓             ↓
         Log System   Monitoring     Trace System
              │            │             │
              └────────────┼─────────────┘
                           ↓
                       Engineers
                           │
                           ↓
                         Alerts
```

Meanwhile:

```text
API
 ├── PostgreSQL
 ├── Redis
 ├── Queue
 ├── Workers
 ├── WebSockets
 ├── Object Storage
 └── External APIs
```

should all have useful signals where appropriate.

---

# 105. The Mental Model

When your application breaks, don't think:

> "Let me restart the server."

First think:

```text
What changed?

How many users are affected?

Is it all requests or one endpoint?

Did error rate increase?

Did latency increase?

Which percentile increased?

Is traffic unusual?

Is some resource saturated?

Is the database slow?

Is Redis slow?

Is an external service slow?

Are connections exhausted?

Are queues growing?

Did we deploy something recently?

What do the logs say?

What does the trace say?
```

Then form a hypothesis.

Then test it.

That's the mindset we're trying to build throughout this series.

---

# 106. What You Should Know After Chapter 17

You should now be comfortable explaining:

```text
✓ Logging
✓ Log levels
✓ Structured logging
✓ JSON logs
✓ Request IDs
✓ Correlation IDs
✓ Error handling
✓ Custom errors
✓ Centralized error middleware
✓ Safe error responses
✓ Stack traces
✓ Error tracking
✓ Metrics
✓ Monitoring
✓ Observability
✓ Logs vs metrics vs traces
✓ P50 / P95 / P99
✓ Health checks
✓ Liveness
✓ Readiness
✓ Alerts
✓ Alert fatigue
✓ Timeouts
✓ Retries
✓ Exponential backoff
✓ Circuit breakers
✓ Graceful degradation
✓ Golden signals
✓ SLI
✓ SLO
✓ SLA
✓ Error budgets
✓ Production debugging
```

But the bigger lesson is this:

A backend isn't finished when:

```text
npm run dev
```

works.

A real backend needs to tell you:

```text
when it's healthy,
when it's slow,
when it's failing,
why it's failing,
who is affected,
and where the problem actually is.
```

And when something does fail, you shouldn't be guessing.

You should have enough evidence to follow the trail.

That's what good logging, monitoring, and error handling give you.

---

# Next — Chapter 18: Testing Your Backend

We've now built quite a lot:

```text
HTTP
APIs
Databases
Authentication
Caching
Queues
WebSockets
File Storage
Logging
Monitoring
Error Handling
```

But there's one uncomfortable question:

> **How do you know your backend still works after you change the code?**

You don't want to manually open Postman every time and test:

```text
login
create user
create post
update post
delete post
```

And you definitely don't want to discover in production that:

```text
"small refactor"
```

broke authentication.

In Chapter 18 we'll learn:

```text
Unit tests
Integration tests
API tests
End-to-end tests
Mocks
Stubs
Fixtures
Test databases
Test isolation
Testing async code
Testing authentication
Testing external APIs
Test coverage
Testing failures
```

And most importantly:

> **How do you decide what should actually be tested instead of blindly trying to get 100% test coverage?**