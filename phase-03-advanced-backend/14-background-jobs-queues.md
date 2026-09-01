# Chapter 14 — Background Jobs & Queues

In the last chapter, we talked about caching.

Now imagine we have this endpoint:

```http
POST /api/v1/users
```

A user signs up.

After creating the account, our backend wants to:

```text
Create the user
Send welcome email
Send analytics event
Generate recommendations
Resize profile image
Notify another service
```

If we do everything inside the request:

```text
Client
  ↓
POST /users
  ↓
Create user
  ↓
Send email
  ↓
Generate recommendations
  ↓
Resize image
  ↓
Send analytics
  ↓
Response
```

The user might be waiting for several seconds.

And that's not even the biggest problem.

What happens if the email service is down?

Does user registration fail?

Probably not.

The account was created successfully.

The email can be sent later.

This is where **background jobs and queues** become useful.

---

# 1. The Basic Idea

Instead of doing everything immediately:

```text
Request
  ↓
Do 10 things
  ↓
Response
```

we can separate the work:

```text
Request
  ↓
Do important work
  ↓
Put background task into queue
  ↓
Response
```

Then another process handles the task:

```text
Queue
  ↓
Worker
  ↓
Do the work
```

So our system becomes:

```text
                ┌──────────────┐
                │    Client    │
                └──────┬───────┘
                       ↓
                ┌──────────────┐
                │  API Server  │
                └──────┬───────┘
                       ↓
                 Create User
                       ↓
                 Add Job
                       ↓
                  ┌────────┐
                  │ Queue  │
                  └───┬────┘
                      ↓
                  ┌───────┐
                  │Worker │
                  └───┬───┘
                      ↓
                 Send Email
```

The API doesn't have to wait for the worker.

---

# 2. What Is a Job?

A **job** is simply a piece of work that needs to be done.

For example:

```text
Send welcome email
```

is a job.

```text
Resize uploaded image
```

is a job.

```text
Generate monthly report
```

is a job.

```text
Process payment webhook
```

could be a job.

You can think of it as:

```text
Job = "Please do this work."
```

---

# 3. What Is a Queue?

A queue is a waiting line.

Imagine a restaurant:

```text
Customer 1
Customer 2
Customer 3
Customer 4
```

The kitchen processes them one by one.

A software queue works similarly:

```text
Job 1
Job 2
Job 3
Job 4
```

A worker picks jobs from the queue and processes them.

So:

```text
Producer
   ↓
Queue
   ↓
Worker
```

---

# 4. Producer and Consumer

You will hear these two words often.

### Producer

The thing that puts a job into the queue.

For us:

```text
API server
```

is the producer.

### Consumer

The thing that takes jobs from the queue and processes them.

For us:

```text
Worker
```

is the consumer.

So:

```text
API Server
   ↓
Producer
   ↓
Queue
   ↓
Consumer
   ↓
Worker
```

That's the basic architecture.

---

# 5. Why Not Just Use `setTimeout()`?

A beginner might think:

```js
setTimeout(() => {
  sendEmail();
}, 5000);
```

Problem solved?

Not really.

What happens if your Node.js server restarts?

The job disappears.

What if there are 10,000 emails?

What if you have multiple API servers?

What if sending the email fails?

What if you need to retry?

What if you want to know which jobs failed?

`setTimeout()` isn't a durable job queue.

For serious background work, you usually need a proper queue system.

---

# 6. Why Do We Need Background Jobs?

The main reason is:

> **Not every piece of work needs to happen before the user gets a response.**

For example:

```text
User signs up
      ↓
Create account
      ↓
Response
```

The user doesn't necessarily need to wait for:

```text
Send welcome email
```

So:

```text
Synchronous work
→ must happen before response

Background work
→ can happen after the request
```

This can make APIs faster and more resilient.

---

# 7. Synchronous vs Asynchronous Work

### Synchronous

The request waits.

```text
Request
  ↓
Task
  ↓
Task
  ↓
Task
  ↓
Response
```

### Background/asynchronous

The request schedules work.

```text
Request
  ↓
Create job
  ↓
Response

Queue
  ↓
Worker
  ↓
Task
```

The second approach is useful when the work can safely happen later.

---

# 8. But Don't Put Everything in a Queue

This is important.

Queues aren't automatically better.

Suppose:

```http
GET /users/42
```

You need to return the user.

Putting this into a background queue would make no sense:

```text
GET /users/42
 ↓
Queue
 ↓
Worker
 ↓
Get user
 ↓
???
```

The client needs the answer immediately.

Queues are useful when work can be:

```text
Delayed
Retried
Processed independently
Expensive
Bursty
```

and doesn't need to block the original request.

---

# 9. Good Background Job Examples

Common examples:

```text
Email sending
Image processing
Video processing
Report generation
Data exports
Notifications
Search indexing
Analytics processing
Webhook processing
Scheduled tasks
```

For example:

```text
User uploads video
        ↓
API stores video
        ↓
Queue "process-video"
        ↓
Worker
        ↓
Transcode video
        ↓
Store result
```

The upload endpoint doesn't have to keep the HTTP connection open while a 10-minute video is processed.

---

# 10. The Queue Gives You a Buffer

Imagine suddenly 100,000 users sign up.

Without a queue:

```text
100,000 requests
      ↓
100,000 email operations
      ↓
Email provider
      ↓
Overload
```

With a queue:

```text
100,000 signup events
        ↓
      Queue
        ↓
  Workers process
        ↓
    Email provider
```

The queue acts as a buffer.

The API can accept work faster than the worker can process it.

The workers catch up over time.

---

# 11. Queue Depth

Now we have an interesting metric:

> **How many jobs are waiting?**

For example:

```text
Queue:
1,000 jobs
```

Then:

```text
Queue:
10,000 jobs
```

Then:

```text
Queue:
100,000 jobs
```

This tells us that work is arriving faster than we're processing it.

That's useful operational information.

---

# 12. Workers

A worker is a process responsible for processing jobs.

For example:

```text
Email Queue
    ↓
Worker 1
Worker 2
Worker 3
```

If one worker can process:

```text
100 jobs/minute
```

then three workers might process roughly:

```text
300 jobs/minute
```

assuming the work can be parallelized and the downstream systems can handle the load.

This gives us a way to scale background processing.

---

# 13. Horizontal Scaling Workers

Suppose:

```text
Queue
 ↓
Worker
```

isn't fast enough.

We can run:

```text
Queue
 ├── Worker 1
 ├── Worker 2
 ├── Worker 3
 ├── Worker 4
 └── Worker 5
```

They can consume jobs concurrently.

This is another example of horizontal scaling.

Instead of making one worker infinitely powerful:

```text
Worker
→ bigger machine
```

we can often add more workers:

```text
Worker 1
Worker 2
Worker 3
...
```

But remember:

> More workers aren't always better.

If the downstream email provider can only handle a certain rate, adding 100 workers may just make things worse.

---

# 14. Concurrency

Workers can also process multiple jobs concurrently.

For example:

```text
Worker
  ↓
Job A
Job B
Job C
```

But concurrency needs limits.

Imagine each job makes an API request to a third-party service.

If you process:

```text
1 job at a time
```

you might be too slow.

If you process:

```text
10,000 at a time
```

you might get:

```text
429 Too Many Requests
```

from the external service.

So you need to choose a reasonable concurrency level.

---

# 15. BullMQ

Since we're using Node.js and Redis in this series, a good practical queue library to learn is:

> **BullMQ**

BullMQ uses Redis to manage queues and jobs.

Conceptually:

```text
Node.js API
    ↓
BullMQ
    ↓
Redis
    ↓
BullMQ Worker
```

Redis is storing the queue state.

BullMQ handles the queue mechanics.

---

# 16. Installing BullMQ

In a Node.js project:

```bash
npm install bullmq
```

You'll also need Redis running.

For local development, you might use Docker:

```bash
docker run --name redis -p 6379:6379 redis
```

Now you have:

```text
Node.js
   ↓
Redis
```

---

# 17. Creating a Queue

A simplified example:

```ts
import { Queue } from "bullmq";

const emailQueue = new Queue("email", {
  connection: {
    host: "localhost",
    port: 6379
  }
});
```

Now we have an email queue.

Think:

```text
emailQueue
```

as:

> "The waiting line for email jobs."

---

# 18. Adding a Job

Suppose a user registers.

We can add:

```ts
await emailQueue.add("welcome-email", {
  userId: user.id,
  email: user.email
});
```

Now the queue contains something like:

```text
Job
----------------
name: welcome-email

data:
userId: 42
email: rahul@example.com
```

The API doesn't have to send the email itself.

It just puts the job into the queue.

---

# 19. The Worker

Now create a worker:

```ts
import { Worker } from "bullmq";

const worker = new Worker(
  "email",
  async job => {
    console.log("Processing:", job.name);

    await sendWelcomeEmail(job.data.email);
  },
  {
    connection: {
      host: "localhost",
      port: 6379
    }
  }
);
```

Now:

```text
API
 ↓
emailQueue.add()
 ↓
Redis
 ↓
Worker
 ↓
sendWelcomeEmail()
```

That's our first real background job system.

---

# 20. The Important Part: The API Doesn't Do the Work

Our registration endpoint becomes:

```ts
const user = await prisma.user.create({
  data: {
    email,
    passwordHash
  }
});

await emailQueue.add("welcome-email", {
  userId: user.id,
  email: user.email
});

return user;
```

The API does the important work:

```text
Create user
```

Then schedules:

```text
Send email
```

The worker handles the rest.

---

# 21. What If the Worker Is Down?

This is where queues become much more useful than:

```js
setTimeout()
```

Suppose:

```text
API
 ↓
Queue
 ↓
Worker DOWN
```

The job can remain in the queue.

When the worker comes back:

```text
Queue
 ↓
Worker
 ↓
Process pending jobs
```

So the work doesn't necessarily disappear just because a worker process restarted.

Exactly how recovery works depends on the queue system and job state.

---

# 22. What If a Job Fails?

Suppose:

```text
Worker
 ↓
sendEmail()
 ↓
Email provider error
```

Should we just give up?

Usually no.

Some failures are temporary.

For example:

```text
Network error
Service temporarily unavailable
Timeout
Rate limit
```

A retry may succeed.

---

# 23. Retries

You can configure jobs to retry.

Conceptually:

```text
Job
 ↓
Attempt 1
 ↓
FAILED
 ↓
Attempt 2
 ↓
FAILED
 ↓
Attempt 3
 ↓
SUCCESS
```

This is one of the biggest advantages of a proper job queue.

---

# 24. But Don't Retry Forever

Imagine:

```text
Payment provider is permanently broken.
```

And your worker does:

```text
retry
retry
retry
retry
retry
retry
...
```

Forever.

That's not useful.

You normally configure a maximum number of attempts.

For example:

```text
attempts = 5
```

After that:

```text
Job
 ↓
Failed
 ↓
Stop retrying
```

---

# 25. Exponential Backoff

Suppose a service is temporarily unavailable.

Instead of:

```text
retry immediately
retry immediately
retry immediately
```

we can wait longer between attempts.

For example:

```text
Attempt 1
 ↓
wait 1 sec

Attempt 2
 ↓
wait 2 sec

Attempt 3
 ↓
wait 4 sec

Attempt 4
 ↓
wait 8 sec
```

This is called:

> **Exponential backoff**

The exact delays don't have to follow these exact numbers.

The idea is:

> Give the failing service some time to recover instead of hammering it continuously.

---

# 26. Add Jitter

There's another small improvement.

Imagine 10,000 jobs all fail at exactly the same time.

If all of them retry after exactly:

```text
10 seconds
```

you could get:

```text
10,000 requests
```

at the same moment.

That's another thundering herd problem.

You can add randomness to retry delays.

This is called:

> **Jitter**

So instead of:

```text
retry after exactly 10 seconds
```

you might get:

```text
9.2 seconds
10.7 seconds
11.1 seconds
9.8 seconds
```

The requests become more spread out.

---

# 27. Dead-Letter Queue

What happens after a job fails repeatedly?

You don't necessarily want to lose it.

You can move it into a place for investigation.

This is often called a:

> **Dead-letter queue (DLQ)**

Conceptually:

```text
Main Queue
    ↓
Worker
    ↓
FAILED
    ↓
Retry
    ↓
FAILED
    ↓
Retry
    ↓
FAILED
    ↓
Dead-Letter Queue
```

Now engineers can inspect failed jobs.

The exact DLQ implementation depends on the queue technology.

---

# 28. Failed Doesn't Always Mean the Work Didn't Happen

This is a very important backend problem.

Suppose the worker calls:

```text
Payment API
```

The payment provider successfully processes the payment.

But the response times out.

Our worker thinks:

```text
FAILED
```

and retries.

Now the payment could happen twice.

This is why background jobs need the same kind of thinking we discussed in Chapter 12.

---

# 29. Idempotency

A job should ideally be safe to retry.

Suppose:

```text
send invoice #123
```

is retried.

You don't want:

```text
Invoice sent
Invoice sent again
Invoice sent again
```

You want the operation to recognize:

```text
invoice:123
```

has already been processed.

This is **idempotency**.

---

# 30. A Simple Job Idempotency Example

Suppose the job is:

```text
process-payment
paymentId = 123
```

Before processing:

```text
Was payment 123 already processed?
```

If yes:

```text
Skip
```

If no:

```text
Process
Record successful processing
```

Conceptually:

```text
Job
 ↓
Check idempotency state
 ↓
Already done?
 ├── Yes → finish
 └── No  → process
```

Retries then become much safer.

---

# 31. Exactly Once Is Hard

You may hear:

> "We'll make sure the job runs exactly once."

Be careful.

In distributed systems, guaranteeing exactly-once effects is much harder than it sounds.

A worker can:

```text
perform external action
        ↓
crash before recording success
```

When it restarts, it doesn't know whether the action happened.

So instead of obsessing over:

```text
exactly once execution
```

many systems aim for:

> **at-least-once delivery + idempotent processing**

Meaning:

```text
The job may be delivered more than once.
The processing logic safely handles duplicates.
```

This is a very useful mental model for interviews.

---

# 32. At-Least-Once Delivery

Imagine:

```text
Queue
 ↓
Job
 ↓
Worker
```

The queue guarantees that the job won't be lost easily and may deliver it again if the worker fails before acknowledging/completing it.

So:

```text
Job
 ↓
Worker processes
 ↓
Worker crashes
```

The queue may eventually make the job available again.

This is why your job processing code should be designed with retries and duplicates in mind.

---

# 33. Job Timeout

What if a worker gets stuck?

For example:

```text
Worker
 ↓
Job
 ↓
external API
 ↓
waiting...
```

forever.

You need appropriate timeouts.

A job should not consume a worker forever.

For example:

```text
Job starts
 ↓
External call
 ↓
Timeout
 ↓
Job fails
 ↓
Retry / DLQ
```

Again, exact settings depend on the workload.

---

# 34. Delayed Jobs

Queues can also schedule work for later.

For example:

```text
Send reminder in 24 hours
```

Instead of:

```text
sendReminder()
```

immediately.

You create:

```text
Job
delay = 24 hours
```

Then:

```text
24 hours later
      ↓
Worker
      ↓
Send reminder
```

This is useful for:

```text
Email reminders
Subscription renewals
Scheduled notifications
Trial expiration
Follow-ups
```

---

# 35. Scheduled Jobs

Another common use:

```text
Every night at 2 AM
```

run:

```text
Generate reports
Clean old records
Sync external data
```

This is often called a:

> **Scheduled job / cron job**

For example:

```text
02:00
 ↓
Generate daily report
```

Queues and schedulers can work together.

---

# 36. Queue vs Cron

They're related, but not the same.

### Cron

Answers:

> **When should something start?**

Example:

```text
Every day at 2 AM
```

### Queue

Answers:

> **How should work be processed reliably?**

Example:

```text
Generate 10,000 reports
```

You might use:

```text
Cron
 ↓
Create jobs
 ↓
Queue
 ↓
Workers
```

This lets you schedule work while still getting retries and scalable processing.

---

# 37. One Queue or Multiple Queues?

Suppose we have:

```text
Email jobs
Image jobs
Report jobs
```

You could have:

```text
Queue
 ├── email
 ├── image
 └── reports
```

Or separate queues:

```text
Email Queue
Image Queue
Report Queue
```

Separate queues can be useful when jobs have very different characteristics.

For example:

```text
Email
→ fast

Video processing
→ expensive

Reports
→ CPU-heavy
```

You might want separate worker pools.

---

# 38. Why Separate Worker Types?

Imagine your video processing jobs become huge.

If everything shares one queue:

```text
Email
Video
Email
Email
Video
```

Video jobs could occupy workers for a long time.

Emails might get delayed.

Instead:

```text
Email Queue
   ↓
Email Workers

Video Queue
   ↓
Video Workers
```

Now each workload can scale independently.

This is a common production design.

---

# 39. Queue Backpressure

Suppose:

```text
Producer:
10,000 jobs/sec
```

but:

```text
Workers:
2,000 jobs/sec
```

Then:

```text
Queue size
keeps growing
```

Eventually:

```text
Memory
Storage
Downstream systems
```

can become problems.

This is called a **backpressure** problem.

You need to think about:

```text
Can producers slow down?
Can we reject work?
Can we add workers?
Can downstream services handle more?
Can we prioritize jobs?
```

---

# 40. Job Priority

Not every job is equally important.

Imagine:

```text
Password reset email
```

versus:

```text
Weekly analytics report
```

A password reset might need to happen quickly.

So some queue systems support priorities.

Conceptually:

```text
HIGH
→ password reset

NORMAL
→ welcome email

LOW
→ analytics report
```

Workers can process high-priority work first.

---

# 41. Queue Monitoring

Once queues are part of production, you want to monitor:

```text
Queue depth
Processing rate
Failed jobs
Retry count
Job latency
Oldest waiting job
Worker health
```

For example:

```text
Queue:
50,000 waiting jobs
```

is very different from:

```text
Queue:
5 waiting jobs
```

The queue itself becomes an important part of system health.

---

# 42. Don't Put Huge Payloads Into Jobs

Suppose you need to process a video.

Don't necessarily put the entire video inside the queue message.

Instead:

```text
Queue job:
{
  "videoId": 123
}
```

Then the worker can retrieve the actual file from storage.

Same idea for large objects.

Keep job payloads reasonably small.

---

# 43. Pass IDs Instead of Large Objects

Instead of:

```json
{
  "user": {
    "id": 123,
    "name": "...",
    "hundredsOfFields": "..."
  }
}
```

consider:

```json
{
  "userId": 123
}
```

Then the worker can fetch the required information.

This can reduce queue size and avoid putting stale snapshots into jobs.

But there is a tradeoff.

If the worker needs the exact state from the moment the job was created, you may intentionally include some data.

The right choice depends on the job.

---

# 44. What If the User Deletes the Data?

Imagine:

```text
User creates report
 ↓
Queue job created
```

Then:

```text
User deletes account
```

Worker later receives:

```text
generate-report(userId=123)
```

But user 123 no longer exists.

Your worker needs to handle that case.

This is another reason background jobs require defensive programming.

Jobs can run later, when the world has changed.

---

# 45. Jobs Are Not Requests

This is a useful mindset.

An HTTP request usually has:

```text
Client
 ↓
Request
 ↓
Response
```

A background job can have:

```text
Producer
 ↓
Queue
 ↓
Worker
 ↓
Success / Retry / Failure
```

There's no user waiting on the other end of the queue.

That means you can design the workflow differently.

---

# 46. Build an Email System

Let's put everything together.

Our application:

```text
POST /api/v1/users
```

User registers.

### Step 1

Create user:

```text
PostgreSQL
 ↓
User created
```

### Step 2

Add job:

```text
email queue
 ↓
welcome-email
```

### Step 3

Return:

```http
201 Created
```

### Step 4

Worker picks job:

```text
Queue
 ↓
Worker
```

### Step 5

Send email:

```text
Worker
 ↓
Email provider
```

### Step 6

If it fails:

```text
Retry
```

### Step 7

If it keeps failing:

```text
Dead-letter / failed jobs
```

That's a real backend workflow.

---

# 47. Project Structure

You could organize the project like:

```text
src/
├── server.ts
├── routes/
│   └── auth.routes.ts
├── controllers/
│   └── auth.controller.ts
├── queues/
│   └── email.queue.ts
├── workers/
│   └── email.worker.ts
├── services/
│   └── email.service.ts
└── lib/
    └── redis.ts
```

The exact structure isn't important.

The separation of responsibilities is.

---

# 48. API Server vs Worker

You don't necessarily want your API server and worker to be the exact same process.

You can run:

```text
API process
   ↓
handles HTTP requests
```

and separately:

```text
Worker process
   ↓
handles background jobs
```

For example:

```bash
npm run server
```

and:

```bash
npm run worker
```

Now they can scale independently.

---

# 49. Why This Is Better

Imagine:

```text
API:
100 requests/sec
```

and:

```text
Email:
10,000 jobs waiting
```

You can add more email workers without adding more API servers.

```text
API
 ↓
Queue
 ├── Worker 1
 ├── Worker 2
 ├── Worker 3
 └── Worker 4
```

The two workloads are separated.

That's a major advantage.

---

# 50. Common Mistakes

## Mistake 1 — Using `setTimeout()` as a job queue

It doesn't provide durable job management.

---

## Mistake 2 — Putting everything into a queue

Not every operation should be asynchronous.

If the client needs the result immediately, a queue may not make sense.

---

## Mistake 3 — No retry strategy

Temporary failures happen.

---

## Mistake 4 — Retrying forever

Permanent failures need a stopping point.

---

## Mistake 5 — Retrying non-idempotent operations blindly

Retries can create duplicate side effects.

---

## Mistake 6 — No timeout

A stuck job can occupy a worker indefinitely.

---

## Mistake 7 — Unlimited concurrency

More workers can overwhelm downstream services.

---

## Mistake 8 — Huge job payloads

Put references/IDs into jobs when appropriate instead of enormous objects.

---

## Mistake 9 — No monitoring

A queue growing from:

```text
100
→ 1,000
→ 10,000
→ 100,000
```

is an important production signal.

---

## Mistake 10 — Running workers inside the API process without thinking about scaling

It can work for simple applications, but separating API and worker processes often makes scaling and deployment much cleaner.

---

# 51. Interview Questions

## 1. Why do we use background jobs?

To move work that doesn't need to block the HTTP response out of the request path.

This can improve response latency, reliability, and scalability.

---

## 2. What is a queue?

A system that stores work until a worker can process it.

---

## 3. What is a worker?

A process that consumes jobs from a queue and performs the required work.

---

## 4. What is a producer?

The component that creates and adds jobs to a queue.

---

## 5. What is a consumer?

The component that retrieves and processes jobs from a queue.

---

## 6. Why use a queue for sending emails?

Email delivery can be slow or temporarily fail.

A queue allows the API to respond without waiting and gives the system a way to retry failed email jobs.

---

## 7. What happens if a worker crashes?

Depending on the queue system, an unfinished job can become available for processing again.

This is why job processing should be designed to safely handle retries.

---

## 8. What is exponential backoff?

A retry strategy where the delay between failed attempts increases over time.

---

## 9. What is a dead-letter queue?

A place where jobs that repeatedly fail can be moved for later investigation or manual handling.

---

## 10. Why is idempotency important for background jobs?

Because jobs may be retried or delivered more than once.

Idempotent processing prevents duplicate side effects.

---

## 11. What is backpressure?

When work arrives faster than the system can process it, causing the queue to grow.

---

## 12. Why separate API servers and workers?

They have different workloads and can be scaled, deployed, and monitored independently.

---

# 52. Interview Scenario

> Your API sends emails directly. Email delivery takes 2 seconds. The API currently responds in 2 seconds. How would you improve it?

Don't say:

> "Use Redis."

First identify the problem.

```text
HTTP request
   ↓
Create user
   ↓
Send email
   ↓
Wait 2 seconds
   ↓
Response
```

Move email sending into a background job:

```text
HTTP request
   ↓
Create user
   ↓
Add email job
   ↓
Response

Queue
   ↓
Worker
   ↓
Send email
```

Now the user doesn't have to wait for email delivery.

---

# 53. Interview Scenario

> Your email worker sometimes sends the same email twice. What could be happening?

Think about retries.

Maybe:

```text
Worker
 ↓
Send email
 ↓
Email provider succeeds
 ↓
Worker crashes before acknowledging job
```

The queue may deliver the job again.

Now:

```text
Worker
 ↓
Send email again
```

Possible solutions include designing the operation with idempotency or maintaining appropriate processing state.

The key lesson:

> **A retry can repeat side effects.**

---

# 54. Interview Scenario

> Your queue has 1 million jobs and keeps growing. You add 50 workers, but the queue still grows. What would you investigate?

Don't immediately add 500 more workers.

Ask:

```text
What is producing the jobs?
What is the processing rate?
What is slowing workers down?
Is a downstream service rate-limiting us?
Are jobs waiting on the database?
Are jobs failing and retrying?
Is there a bottleneck shared by all workers?
Can the workload actually be parallelized?
```

Maybe your email provider only allows:

```text
1,000 requests/minute
```

Adding more workers won't solve that.

It could make the rate limiting worse.

This is why:

> **Scaling workers is not the same as scaling the whole system.**

---

# 55. Interview Scenario

> You have image-processing jobs and email jobs in the same queue. Image processing takes 30 seconds while emails take 100ms. Emails are getting delayed. What would you do?

Separate the workloads:

```text
Image Queue
   ↓
Image Workers

Email Queue
   ↓
Email Workers
```

Now they can scale independently.

This is a common reason to create separate queues for different workloads.

---

# 56. The Mental Model

Don't think:

```text
Queue
=
Redis list
```

Think about the complete workflow:

```text
Producer
    ↓
Queue
    ↓
Worker
    ↓
Success
```

And when things go wrong:

```text
Worker
    ↓
Failure
    ↓
Retry
    ↓
Backoff
    ↓
Failure again
    ↓
Dead-letter / failed state
```

And always ask:

```text
What if the worker crashes?

What if the job runs twice?

What if the downstream service is down?

What if 1 million jobs arrive?

What if processing is slower than production?

What if the job takes forever?

What if the underlying data changes before the job runs?
```

These questions are much more valuable than memorizing BullMQ methods.

---

# 57. The Backend Architecture So Far

Our system is getting more interesting.

```text
                         Client
                           │
                           ↓
                         HTTPS
                           │
                           ↓
                    ┌──────────────┐
                    │ API Server   │
                    └──────┬───────┘
                           │
                 ┌─────────┴─────────┐
                 ↓                   ↓
              Redis              PostgreSQL
              Cache             Source of Truth
                 │
                 │
                 ↓
               Queue
                 │
                 ↓
              Workers
                 │
        ┌────────┼────────┐
        ↓        ↓        ↓
      Email    Images   Reports
```

We've now introduced another important idea:

> **Not every piece of backend work belongs inside an HTTP request.**

Some work can happen later.

Some work needs retries.

Some work needs its own workers.

Some work needs to survive a server restart.

That's what queues help us build.

---

# 58. What You Should Know After Chapter 14

You should be comfortable explaining:

```text
Job
→ A piece of work to be processed

Queue
→ Waiting line for work

Producer
→ Adds jobs

Consumer / Worker
→ Processes jobs

Background job
→ Work that doesn't need to block the request

Retry
→ Try failed work again

Backoff
→ Wait before retrying

Dead-letter queue
→ Failed jobs that need separate handling

Idempotency
→ Safe handling of repeated processing

Concurrency
→ Number of jobs processed at the same time

Backpressure
→ Work arriving faster than it can be processed
```

And you should be able to draw:

```text
API
 ↓
Queue
 ↓
Worker
 ↓
External service
```

and explain what happens when:

```text
Worker crashes
Job fails
Job runs twice
Queue grows
External service is unavailable
```

If you can do that, you've understood the important part.

You don't need to memorize every BullMQ option.

---

# 59. Mini Project — Background Email System

Now build it.

Start with:

```http
POST /api/v1/users
```

When a user registers:

```text
1. Create user in PostgreSQL
2. Add welcome-email job
3. Return 201
```

Then create:

```text
email.worker.ts
```

The worker should:

```text
1. Pick job
2. Read email/user information
3. Send email
4. Mark job successful
```

Then add:

```text
✓ Retry failed jobs
✓ Exponential backoff
✓ Maximum attempts
✓ Job timeout
✓ Failed-job handling
✓ Idempotency
✓ Queue monitoring
```

Finally, deliberately break things.

Stop the email service.

Restart the worker.

Create 100 jobs.

Kill the worker halfway through processing.

Make the email provider return errors.

Then ask:

> **What happened to my jobs?**

That's the kind of experiment that will teach you much more than simply reading about queues.

---

# 60. One Last Thing

There's a pattern starting to appear throughout this series.

Chapter 12 taught us:

```text
Retries can happen.
```

Chapter 13 taught us:

```text
Caches can become stale.
```

Chapter 14 teaches us:

```text
Jobs can fail and run more than once.
```

These are all examples of the same bigger backend reality:

> **Distributed systems are unreliable.**

Networks fail.

Servers restart.

Databases become slow.

External APIs timeout.

Messages get delayed.

Requests get retried.

Data changes while you're processing it.

Good backend engineering isn't about pretending these things won't happen.

It's about designing the system so that when they do happen, the system behaves predictably.

That's the mindset you should carry into the next chapters.

---

# Next — Chapter 15: WebSockets & Real-Time Systems

So far, almost everything we've built works like this:

```text
Client
  ↓
Request
  ↓
Server
  ↓
Response
```

But what if the server needs to tell the client something **right now**?

Imagine a chat application:

```text
Rahul sends message
        ↓
Server
        ↓
Aman should see it immediately
```

We don't want Aman constantly asking:

```text
"Anything new?"
"Anything new?"
"Anything new?"
"Anything new?"
```

every few seconds.

We'll learn how real-time communication works using:

```text
WebSockets
Socket.IO
Rooms
Connections
Broadcasting
Presence
Reconnection
Heartbeats
Scaling real-time servers
Redis Pub/Sub
```

And we'll answer a very common interview question:

> **"You built a WebSocket chat application. How would you scale it from one server to 100 servers?"**