# Chapter 13 — Caching

Let's say we have built our job API.

We have:

```http
GET /api/v1/jobs
```

Everything works.

Our database has:

```text
1,000,000 jobs
```

Now imagine this endpoint gets called:

```text
10,000 times
```

in a short period.

Every request goes:

```text
Client
  ↓
Backend
  ↓
PostgreSQL
  ↓
Query
  ↓
Response
```

But here's something interesting.

Maybe the first 9,000 requests are asking for almost exactly the same data.

Why should PostgreSQL do the same work 9,000 times?

What if we could keep a copy of frequently requested data somewhere much faster?

That's the basic idea behind **caching**.

---

# 1. What Is a Cache?

A cache is simply:

> **A place where we temporarily keep data that we expect to need again.**

Think about a restaurant.

Imagine the waiter has to walk to the kitchen every single time someone asks:

> "Can I get some water?"

That would be silly.

Instead, they keep water nearby.

```text
Kitchen
   ↓
Water
   ↓
Waiter
   ↓
Table
```

The next request is much faster.

That's basically what caching does.

Instead of:

```text
Request
   ↓
Database
   ↓
Expensive work
   ↓
Response
```

we can have:

```text
Request
   ↓
Cache
   ↓
Found?
 ┌─┴─┐
Yes  No
 ↓    ↓
Return Database
       ↓
      Store
       ↓
      Cache
```

---

# 2. Why Is Caching Useful?

Caching can help with:

```text
Lower latency
Less database load
Higher throughput
Better scalability
```

For example:

```text
Without cache:

10,000 requests
       ↓
10,000 database queries
```

With a good cache hit rate:

```text
10,000 requests
       ↓
8,000 cache hits
       ↓
2,000 database queries
```

Those numbers are just an example.

The actual benefit depends on the workload and cache design.

---

# 3. Cache Is Not Your Primary Database

This is one of the most important concepts.

Your database might contain:

```text
Users
Jobs
Posts
Orders
Payments
```

Your cache usually contains:

```text
temporary copies
```

For example:

```text
PostgreSQL
   ↓
source of truth

Redis
   ↓
cached copy
```

If Redis disappears, your application should generally be able to rebuild the cached data from the source of truth.

That's the basic idea.

---

# 4. Why Is Cache Faster?

A common caching system is:

> **Redis**

Redis is an in-memory data store.

The important word is:

> **Memory**

Accessing data from memory can be much faster than repeatedly performing database operations involving disk, query planning, indexes, locks, network round trips, and other work.

This doesn't mean:

```text
Redis = always faster for everything
```

It means:

> Redis is designed for very fast data access and can be useful for workloads where repeated access to the same data makes caching worthwhile.

---

# 5. What Is Redis?

Redis is an in-memory data store that supports several data structures.

You can use it for things such as:

```text
Caching
Sessions
Counters
Rate limiting
Queues
Pub/Sub
Distributed coordination
```

We'll use it primarily for caching here.

Later in this series, you'll see Redis again for queues and other backend patterns.

---

# 6. The Simplest Cache

Imagine:

```text
GET /jobs/123
```

Without caching:

```text
Client
  ↓
Backend
  ↓
PostgreSQL
  ↓
job 123
  ↓
Backend
  ↓
Client
```

With caching:

```text
Client
  ↓
Backend
  ↓
Redis
  ↓
job 123
  ↓
Client
```

If the job isn't in Redis:

```text
Client
  ↓
Backend
  ↓
Redis
  ↓
MISS
  ↓
PostgreSQL
  ↓
job 123
  ↓
Redis
  ↓
Client
```

That is the basic caching pattern.

---

# 7. Cache Hit

When the requested data is already in the cache:

```text
Request
   ↓
Redis
   ↓
Found
   ↓
Return data
```

That's a:

> **Cache hit**

For example:

```text
GET /jobs/123

Redis:
job:123 → found
```

We don't need to query PostgreSQL.

---

# 8. Cache Miss

When the requested data isn't in the cache:

```text
Request
   ↓
Redis
   ↓
Not found
   ↓
Database
```

That's a:

> **Cache miss**

For example:

```text
GET /jobs/123

Redis:
job:123 → not found

PostgreSQL:
job 123 → found
```

We then have an opportunity to put the result into the cache.

---

# 9. Cache Hit Ratio

Now you can measure how useful your cache actually is.

Suppose:

```text
1000 requests
```

and:

```text
800 cache hits
200 cache misses
```

Then:

```text
cache hit ratio = 80%
```

A higher hit ratio generally means the cache is serving more requests.

But don't blindly optimize for:

> "I need 99% cache hit rate."

It depends on the workload.

A cache that reduces expensive database work by 50% might already be extremely useful.

---

# 10. Cache-Aside Pattern

This is probably the most important caching pattern to understand first.

It's called:

> **Cache-aside**

The application controls the cache.

The flow is:

```text
Request
   ↓
Check cache
   ↓
Found?
 ┌─┴─┐
Yes  No
 ↓    ↓
Return Database
       ↓
      Store
       ↓
      Cache
       ↓
      Return
```

Let's walk through it.

---

# 11. Cache-Aside Example

Request:

```http
GET /jobs/123
```

Application checks:

```text
Redis:
job:123
```

If found:

```text
Return cached job
```

If not:

```text
Query PostgreSQL
      ↓
Get job
      ↓
Store job in Redis
      ↓
Return job
```

The next request can come directly from Redis.

---

# 12. Why Is It Called Cache-Aside?

Because the application goes:

```text
Database
```

and:

```text
Cache
```

separately.

The cache isn't automatically managing the database.

Your application decides:

```text
"Check cache first."

"If missing, go to database."

"Then put the result into cache."
```

That's the idea.

---

# 13. Simple Node.js Example

Imagine:

```ts
async function getJob(id: string) {
  const cached = await redis.get(`job:${id}`);

  if (cached) {
    return JSON.parse(cached);
  }

  const job = await prisma.job.findUnique({
    where: { id }
  });

  if (!job) {
    return null;
  }

  await redis.set(
    `job:${id}`,
    JSON.stringify(job),
    {
      EX: 60
    }
  );

  return job;
}
```

Don't worry about the Redis client syntax yet.

Focus on the flow:

```text
Check cache
    ↓
Hit?
    ↓
Return

Miss?
    ↓
Database
    ↓
Store in cache
    ↓
Return
```

That's the core pattern.

---

# 14. What Is TTL?

You don't want cached data to live forever.

Imagine:

```text
job:123
```

gets cached today.

Then the job changes tomorrow.

If Redis still returns yesterday's copy, users might see old data.

That's why we often use:

> **TTL — Time To Live**

TTL tells the cache:

> Keep this value for this amount of time, then expire it.

For example:

```text
job:123
TTL = 60 seconds
```

After 60 seconds:

```text
job:123
   ↓
expired
```

The next request becomes a cache miss and can fetch fresh data.

---

# 15. Why TTL Is Useful

Suppose we cache:

```text
Popular jobs
```

for:

```text
60 seconds
```

We accept that users might see data that's up to roughly a minute old.

In exchange:

```text
Less database traffic
+
Faster responses
```

This is a tradeoff.

Caching is often about deciding:

> **How stale can this data safely be?**

---

# 16. Cache Invalidation

Now we reach the famous problem.

> **Cache invalidation.**

Imagine:

```text
PostgreSQL

job:123
salary = ₹10 LPA
```

Redis contains:

```text
job:123
salary = ₹10 LPA
```

Then someone updates the job:

```http
PATCH /jobs/123
```

Database becomes:

```text
salary = ₹15 LPA
```

But Redis still says:

```text
salary = ₹10 LPA
```

Now your application can return stale data.

---

# 17. How Do We Handle Stale Cache?

One approach:

After updating the database:

```text
Update database
      ↓
Delete cache
```

So:

```text
PATCH /jobs/123
        ↓
PostgreSQL updated
        ↓
Redis DEL job:123
```

The next GET:

```text
GET /jobs/123
        ↓
Redis MISS
        ↓
PostgreSQL
        ↓
New data
        ↓
Redis
```

This is a very common cache-aside strategy.

---

# 18. Update vs Delete the Cache

Suppose:

```text
PATCH /jobs/123
```

changes:

```text
salary
title
location
```

You have two choices.

### Option 1 — Update the cache

```text
Database
   ↓
Update
   ↓
Update Redis
```

### Option 2 — Delete the cache

```text
Database
   ↓
Update
   ↓
Delete Redis value
```

Then the next read repopulates it.

For many applications, invalidating/deleting the cache after a successful database write is simpler.

But it isn't always the best design.

---

# 19. The Hard Part of Cache Invalidation

Imagine your application caches:

```text
job:123
```

but also:

```text
jobs:location:india
jobs:remote
jobs:company:google
jobs:salary:high
```

Now:

```text
PATCH /jobs/123
```

changes the job's location.

Which caches are now stale?

Potentially:

```text
job:123
jobs:location:india
jobs:location:usa
jobs:remote
...
```

This is where caching becomes difficult.

The problem isn't:

> "How do I store something in Redis?"

The difficult question is:

> **"Which cached data becomes invalid when the source data changes?"**

---

# 20. Cache Keys

A cache needs a key.

For example:

```text
job:123
```

or:

```text
user:42
```

or:

```text
jobs:location:india:page:1
```

A good cache key should be:

```text
Predictable
Unique
Consistent
```

For example:

```text
job:${jobId}
```

gives:

```text
job:123
job:124
job:125
```

---

# 21. Query Caching

Now imagine:

```http
GET /jobs?location=India&remote=true
```

You could create a cache key based on the query:

```text
jobs:location=India:remote=true
```

But be careful.

These requests:

```text
/jobs?location=India&remote=true
```

and:

```text
/jobs?remote=true&location=India
```

mean the same thing.

If you blindly construct keys from the raw query string, you might create two cache entries.

So cache keys sometimes need **normalization**.

For example:

```text
sort parameters
normalize values
remove irrelevant parameters
```

Then generate one consistent key.

---

# 22. Cache Serialization

Redis stores data in forms that need to be encoded/decoded when you're working with normal JavaScript objects.

For example:

```ts
const job = {
  id: 123,
  title: "Backend Engineer"
};
```

You might store:

```ts
JSON.stringify(job)
```

and later:

```ts
JSON.parse(cached);
```

So:

```text
JavaScript object
      ↓
JSON.stringify
      ↓
Redis
      ↓
JSON.parse
      ↓
JavaScript object
```

There are other Redis data structures and serialization approaches, but this is enough for our first caching example.

---

# 23. Don't Cache Everything

This is a mistake I see beginners make.

They learn Redis and immediately think:

> "Let's cache the entire database."

No.

Caching has a cost.

You now have:

```text
Application
+
PostgreSQL
+
Redis
```

instead of:

```text
Application
+
PostgreSQL
```

Now you have another system to operate.

You need to think about:

```text
Memory
Expiration
Invalidation
Consistency
Failures
Monitoring
Cache warming
```

Use caching when it solves an actual problem.

---

# 24. What Is Good to Cache?

Good candidates often include:

```text
Frequently requested data
Expensive-to-compute results
Data that doesn't change very often
Popular content
Configuration that changes rarely
Session data
Rate-limit counters
```

Examples:

```text
Popular blog posts
Product catalog
Public profiles
Job listings
Frequently requested API responses
```

---

# 25. What Should You Be Careful Caching?

Be careful with:

```text
Highly sensitive data
Frequently changing data
Data requiring strong consistency
Large objects
One-time data
```

Especially:

```text
Bank balances
Payment states
Inventory counts
```

depending on the application's consistency requirements.

Caching stale financial information can create serious problems.

---

# 26. Cache Consistency

This is a useful question:

> What happens if the database and cache disagree?

For example:

```text
Database:
balance = ₹10,000

Redis:
balance = ₹15,000
```

Which one is correct?

Usually:

```text
Database
→ source of truth
```

The cache is a copy.

This is why you need an explicit strategy for when cached data becomes invalid.

---

# 27. Cache Stampede

Here's an interesting problem.

Suppose:

```text
popular-posts
```

is cached.

Thousands of users request it.

Everything is great.

Then the TTL expires.

Now thousands of requests arrive at almost the same time:

```text
Request 1 → MISS
Request 2 → MISS
Request 3 → MISS
Request 4 → MISS
...
Request 5000 → MISS
```

All of them hit PostgreSQL.

You suddenly created:

```text
5000 database queries
```

at the same time.

This is sometimes called a:

> **Cache stampede**

or:

> **Thundering herd**

---

# 28. How Can We Reduce Cache Stampedes?

There are several techniques.

For example:

```text
Request
   ↓
Cache MISS
   ↓
One request refreshes data
   ↓
Other requests wait/reuse result
```

Other techniques include:

```text
Request coalescing
Locks
Jittered expiration
Background refresh
Stale-while-revalidate
```

You don't need to implement all of these now.

But you should know the problem exists.

---

# 29. TTL Jitter

Imagine 10,000 cache entries all expire at exactly:

```text
12:00:00
```

You could create a burst of database traffic.

One simple technique is adding some randomness to expiration.

Instead of:

```text
TTL = 60 seconds
```

you might use:

```text
60 + random amount
```

so expiration is spread out.

This is called:

> **TTL jitter**

Again, it's not something you need everywhere.

It's useful when synchronized expiration could cause a problem.

---

# 30. Cache Failure

Here's another important question:

> What happens if Redis goes down?

Your application shouldn't necessarily become completely unusable just because the cache is unavailable.

For many cache-aside systems:

```text
Redis unavailable
       ↓
Skip cache
       ↓
Read from database
```

This is sometimes called:

> **Failing open**

for the cache layer.

But whether this is safe depends on the workload.

If Redis is also being used for something more critical, such as distributed coordination or rate limiting, the failure behavior can be different.

The key lesson:

> **A cache is another dependency, and your application needs a failure strategy.**

---

# 31. Cache Eviction

Redis has finite memory.

What happens when it fills up?

Redis can be configured with eviction policies.

For example, the system may remove older/less useful keys depending on the configured policy.

This means:

```text
Cache
   ↓
Memory full
   ↓
Some keys removed
```

Your application should already be prepared for cache misses.

That's another reason cache-aside is convenient.

A missing cache entry isn't necessarily an error.

It's just:

```text
MISS
```

---

# 32. Cache-Aside Failure Scenario

Imagine:

```text
Database
   ↓
successful update
```

Then:

```text
Redis delete
   ↓
fails
```

Now:

```text
Database = new data
Redis = old data
```

What should happen?

You need to think about this.

Possible strategies depend on the system:

```text
Retry invalidation
Short TTL
Background repair
Versioning
Write-through approaches
```

There's no single universal answer.

The important thing is:

> **Cache failures can create consistency problems.**

---

# 33. Write-Through Cache

Another caching strategy is:

> **Write-through**

Instead of:

```text
Application
   ↓
Database
```

and separately updating the cache, writes go through the cache layer.

Conceptually:

```text
Application
     ↓
Cache
     ↓
Database
```

The cache is updated as part of the write process.

The exact implementation varies.

The benefit is that cache and database updates can be coordinated by the caching layer.

The downside is additional complexity and write-path overhead.

---

# 34. Write-Behind Cache

Another pattern is:

> **Write-behind**

The application writes to the cache first:

```text
Application
    ↓
Cache
    ↓
later
    ↓
Database
```

This can improve write latency in some systems.

But it introduces serious durability and consistency concerns.

If the cache fails before data reaches the database, you could lose data.

So this pattern is not something you should casually use for important application data.

---

# 35. Cache-Aside vs Write-Through

A simplified comparison:

|            | Cache-Aside                                  | Write-Through                            |
| ---------- | -------------------------------------------- | ---------------------------------------- |
| Read miss  | App reads DB and populates cache             | Cache layer handles read path            |
| Write      | App updates DB and invalidates/updates cache | Cache and DB updated through write path  |
| Simplicity | Usually simpler                              | More involved                            |
| Common use | Read-heavy workloads                         | Systems wanting coordinated cache writes |

Again:

> Don't choose a caching pattern because it sounds advanced.

Choose based on the consistency and workload requirements.

---

# 36. Cache Invalidation Strategies

There are several ways to keep cached data reasonably fresh.

### TTL

```text
Cache for 60 seconds
```

Simple.

---

### Delete on write

```text
Update DB
   ↓
Delete cache
```

Common with cache-aside.

---

### Update on write

```text
Update DB
   ↓
Update cache
```

Useful when you can reliably keep both synchronized.

---

### Combination

For example:

```text
Update DB
   ↓
Delete cache
   ↓
TTL as backup
```

The exact approach depends on the system.

---

# 37. Cache Key Namespacing

Imagine your Redis instance stores:

```text
user:123
job:123
post:123
```

Notice that IDs overlap.

That's okay because the key includes a namespace.

Instead of:

```text
123
```

use:

```text
user:123
job:123
post:123
```

This makes keys easier to reason about.

You might also use:

```text
prod:user:123
prod:job:123
```

depending on how environments share infrastructure.

---

# 38. Don't Put Unlimited Data Into Keys

This is a subtle issue.

Suppose someone sends:

```http
GET /jobs?search=<10MB string>
```

and your cache key contains the entire query.

You could end up creating enormous keys.

Again:

> Validate input.

Cache keys are generated from user input, so don't let unbounded input create operational problems.

---

# 39. Cache Security

Caching can accidentally leak data.

Imagine:

```text
GET /profile
```

returns:

```text
Rahul's profile
```

You cache it under:

```text
profile
```

Now Aman requests:

```text
GET /profile
```

and gets Rahul's cached profile.

That's a serious bug.

The cache key needs to include the identity when the response is user-specific.

For example:

```text
profile:user:42
```

and:

```text
profile:user:43
```

Caching private data requires careful key design and access-control reasoning.

---

# 40. Cache-Control and HTTP Caching

Caching isn't only Redis.

HTTP itself has caching mechanisms.

For example:

```http
Cache-Control: max-age=3600
```

tells a client or intermediary how long a response may be considered fresh under the specified caching rules.

You can also encounter:

```text
ETag
Last-Modified
If-None-Match
If-Modified-Since
```

These are part of HTTP caching and conditional requests.

So when someone says:

> "Caching"

don't automatically think:

> "Redis."

There are multiple layers.

---

# 41. Browser Cache vs CDN vs Redis

Caching can happen in different places.

```text
Browser
   ↓
CDN
   ↓
Reverse proxy
   ↓
Application cache / Redis
   ↓
Database
```

Each layer solves different problems.

### Browser cache

Reduces network requests from the user's device.

### CDN

Caches content closer to users around the world.

### Redis

Provides fast application-level data access.

### Database

Usually remains the source of truth.

This is why large systems often have multiple caching layers.

---

# 42. Cache the Right Layer

Imagine an image:

```text
company-logo.png
```

You probably don't need Redis to serve it from your backend.

A CDN/browser cache may be much more appropriate.

But:

```text
GET /jobs?location=India
```

might be a good application-level caching candidate.

Think:

> **Where is the expensive work happening, and where can a copy safely live?**

---

# 43. Caching Isn't Always About Speed

Caching can also reduce load.

Imagine:

```text
Database CPU = 90%
```

A cache can reduce repeated queries:

```text
Database CPU
90%
 ↓
50%
```

Again, those numbers are just illustrative.

The point is:

> Caching can increase capacity by preventing repeated work.

---

# 44. When Caching Makes Things Worse

Suppose your endpoint:

```text
GET /random-number
```

returns a different value every time.

Caching it might make no sense.

Or:

```text
GET /user-current-balance
```

may require fresh data depending on the application's requirements.

Adding Redis means:

```text
Application
+
Redis
+
Cache invalidation
+
Monitoring
+
More code
```

If your database can already handle the traffic, caching may simply add unnecessary complexity.

---

# 45. A Practical Rule

Before adding a cache, ask:

```text
1. Is this operation expensive?

2. Is it called frequently?

3. Is the result reused?

4. Can the data tolerate some staleness?

5. How will the cache be invalidated?

6. What happens if Redis goes down?

7. How much memory will it need?

8. Can cached data leak between users?

9. What cache key should we use?

10. How will we know whether caching actually helped?
```

If you can't answer these questions, you're probably not ready to add the cache.

---

# 46. Let's Add Redis to Our Job API

Suppose we have:

```http
GET /api/v1/jobs/:id
```

Without caching:

```text
Request
   ↓
Controller
   ↓
Prisma
   ↓
PostgreSQL
   ↓
Response
```

Now add Redis:

```text
Request
   ↓
Controller
   ↓
Redis
   ↓
Hit?
 ┌─┴─┐
Yes  No
 ↓    ↓
Return Prisma
       ↓
   PostgreSQL
       ↓
     Redis
       ↓
    Response
```

---

# 47. Cache Key

Use:

```text
job:${id}
```

So:

```text
job:1
job:2
job:3
```

Each job has its own cache entry.

---

# 48. Cache Read

Conceptually:

```ts
const cacheKey = `job:${id}`;

const cachedJob = await redis.get(cacheKey);

if (cachedJob) {
  return JSON.parse(cachedJob);
}
```

If it exists:

```text
Cache hit
```

Return it.

---

# 49. Cache Miss

If:

```ts
cachedJob === null
```

then:

```ts
const job = await prisma.job.findUnique({
  where: { id }
});
```

Then:

```ts
await redis.set(
  cacheKey,
  JSON.stringify(job),
  {
    EX: 60
  }
);
```

Now Redis contains the result for 60 seconds.

---

# 50. Update the Job

Suppose:

```http
PATCH /api/v1/jobs/123
```

updates the job.

Do:

```text
Update PostgreSQL
       ↓
Delete Redis key
       ↓
job:123
```

Then:

```http
GET /api/v1/jobs/123
```

becomes:

```text
Redis MISS
   ↓
PostgreSQL
   ↓
New job
   ↓
Redis
   ↓
Response
```

This is a very simple and useful cache-aside implementation.

---

# 51. What If Redis Is Down?

Don't blindly assume Redis will always work.

For this particular cache use case, a reasonable approach might be:

```text
Redis GET fails
     ↓
Log/monitor the failure
     ↓
Continue to database
```

Then:

```text
Database
   ↓
Response
```

The API might become slower, but it can still function.

Whether you should do this depends on your application's requirements.

---

# 52. What If PostgreSQL Is Down?

Now it's different.

If the cache has the requested data:

```text
Redis
   ↓
cached response
```

you might still be able to serve it.

But if it's a cache miss:

```text
Redis MISS
   ↓
PostgreSQL unavailable
```

you can't get fresh data.

Again, this is why the database is normally the source of truth.

---

# 53. Monitoring Your Cache

Once caching enters production, you should know:

```text
Cache hit rate
Cache miss rate
Redis memory usage
Evictions
Latency
Connection errors
Key counts
```

For example:

```text
Cache hit rate
→ 92%

Cache miss rate
→ 8%
```

This tells you whether the cache is actually doing useful work.

You shouldn't blindly assume:

> "We added Redis, so performance improved."

Measure it.

---

# 54. Common Mistakes

## Mistake 1 — Caching everything

More caching doesn't automatically mean better performance.

---

## Mistake 2 — No expiration

Some cached data should not live forever.

---

## Mistake 3 — No invalidation strategy

If the source data changes, what happens to the cached copy?

You should know the answer.

---

## Mistake 4 — Bad cache keys

Poor key design can cause collisions or stale data.

---

## Mistake 5 — Caching user-specific data under shared keys

This can leak one user's data to another.

---

## Mistake 6 — Treating Redis as the database

A cache is usually a copy, not your source of truth.

---

## Mistake 7 — Ignoring Redis failure

Your cache is another dependency.

Plan for failure.

---

## Mistake 8 — No limits on cached data

Redis has finite memory.

---

## Mistake 9 — Assuming cache hit = always correct

A cache hit can still be stale.

---

## Mistake 10 — Adding Redis before measuring

First understand the bottleneck.

Then optimize it.

---

# 55. Interview Questions

## 1. What is caching?

Caching stores frequently accessed or expensive-to-compute data temporarily so future requests can be served faster and/or with less load on the underlying system.

---

## 2. Why is Redis commonly used for caching?

Redis is an in-memory data store designed for very fast data access and supports useful data structures and expiration mechanisms.

---

## 3. What is a cache hit?

When requested data is found in the cache.

---

## 4. What is a cache miss?

When requested data isn't in the cache and the application needs to retrieve it from another source, such as the database.

---

## 5. What is TTL?

Time To Live.

It defines how long a cached value remains valid before it expires.

---

## 6. What is cache invalidation?

The process of making cached data no longer usable when the underlying source data changes.

---

## 7. Explain cache-aside.

The application checks the cache first.

If the data exists, return it.

If not, read from the database, store the result in the cache, and return it.

---

## 8. What is a cache stampede?

When many requests simultaneously miss or lose the same cached value and all hit the underlying database or expensive service at once.

---

## 9. Should Redis replace PostgreSQL?

Usually no.

For a typical cache-aside architecture, PostgreSQL remains the source of truth and Redis stores temporary copies.

---

## 10. What happens if Redis goes down?

It depends on the design.

For a non-critical cache, the application may fall back to the database, resulting in higher latency and database load.

---

# 56. Interview Scenario

> Your API normally responds in 50ms. You add Redis, but now responses are 70ms. What happened?

Don't immediately say:

> "Redis is slow."

Think.

Maybe:

```text
Before:

API
 ↓
PostgreSQL
 ↓
50ms
```

After:

```text
API
 ↓
Redis
 ↓
MISS
 ↓
PostgreSQL
 ↓
Redis SET
 ↓
70ms
```

If most requests are cache misses, you're doing extra work.

You added:

```text
Redis GET
+
Database query
+
Redis SET
```

instead of simply:

```text
Database query
```

This is why you measure **cache hit rate**.

Caching helps when enough requests can actually benefit from it.

---

# 57. Another Interview Scenario

> You cache a user's profile under the key `profile`. User A logs in and gets their profile. User B logs in and gets User A's profile. What's wrong?

The cache key doesn't include the user's identity.

You need something like:

```text
profile:user:123
```

instead of:

```text
profile
```

The general lesson:

> **Cache keys must represent the identity and parameters that affect the response.**

---

# 58. Another Interview Scenario

> Your product price is cached for 10 minutes. A product price changes in the database, but users still see the old price. Why?

Because the cache contains stale data.

Possible solutions:

```text
Delete cache when price changes
```

or:

```text
Update cache when price changes
```

or use an appropriate short TTL.

The correct choice depends on how fresh the data needs to be.

---

# 59. Another Interview Scenario

> Your most popular API response expires at exactly 12:00. At 12:00, your database suddenly receives thousands of requests. Why?

Likely a cache stampede.

Many requests saw the cache miss simultaneously and all attempted to rebuild the same cached result.

Possible solutions include:

```text
Request coalescing
Locks
TTL jitter
Background refresh
Stale-while-revalidate
```

You don't always need these techniques.

But you should recognize the problem.

---

# 60. The Mental Model I Want You to Remember

Don't think:

```text
Redis
=
fast database
```

Think:

```text
Source of truth
      ↓
PostgreSQL

Frequently needed copy
      ↓
Redis
```

And:

```text
Request
   ↓
Cache
   ↓
Hit?
 ┌─┴─┐
Yes  No
 ↓    ↓
Return Database
       ↓
     Cache
       ↓
     Return
```

Then ask:

```text
When does the cache expire?
        ↓
TTL

What happens when data changes?
        ↓
Invalidation

What if many requests miss?
        ↓
Cache stampede

What if Redis dies?
        ↓
Failure strategy

Who does this cached data belong to?
        ↓
Cache key

Is caching actually helping?
        ↓
Measure hit rate + latency + DB load
```

That's the real understanding.

---

# 61. The Bigger Backend Picture

Look at what we've built so far.

```text
Client
   ↓
HTTPS
   ↓
Authentication
   ↓
Authorization
   ↓
Validation
   ↓
Router
   ↓
Business Logic
   ↓
Redis Cache
   ↓
PostgreSQL
```

Now our application can avoid doing expensive database work repeatedly.

But there's another problem.

Some operations shouldn't happen while the user is waiting.

Imagine:

```text
POST /signup
```

After creating the account, we want to:

```text
Send welcome email
Send analytics event
Generate recommendation data
Resize profile image
Notify another service
```

Should the user wait for all of that?

Probably not.

Instead:

```text
Request
   ↓
Create account
   ↓
"Account created"
   ↓
Response
```

And the background work happens separately.

That's where queues and background jobs come in.

---

# Next — Chapter 14: Background Jobs & Queues

We'll learn:

```text
Why background jobs exist
Queues
Workers
Producers and consumers
Redis-backed queues
BullMQ
Retries
Failed jobs
Dead-letter queues
Job idempotency
Delayed jobs
Exponential backoff
```

And we'll answer a very real backend question:

> **What should happen when a user triggers work that takes 30 seconds, but your API shouldn't make them wait 30 seconds?**