# Chapter 22 — Microservices vs Monolith

At this point, we've built quite a lot.

We have talked about:

```text
HTTP
APIs
Databases
Authentication
Caching
Queues
WebSockets
File storage
Logging
Testing
Configuration
Docker
Database scaling
```

Now imagine our application keeps growing.

Maybe we're building a SaaS product.

At first, everything is inside one backend:

```text
                 Backend
                    │
       ┌────────────┼────────────┐
       ↓            ↓            ↓
     Users        Orders       Payments
       ↓            ↓            ↓
    Database     Database     Database
```

And honestly?

That's completely fine.

But eventually the team might start asking:

> "Should we split this backend into multiple services?"

And that's where we hear a word that gets thrown around a lot:

> **Microservices**

Microservices sound impressive.

People sometimes say:

> "We're building a scalable system, so we're using microservices."

But that's not really how good architecture decisions work.

The better question is:

> **What problem are microservices solving for us?**

That's what this chapter is about.

---

# 1. What Is a Monolith?

A **monolith** is an application where multiple parts of the system are deployed as one application.

For example:

```text
                 Backend Application
                        │
       ┌────────────────┼────────────────┐
       ↓                ↓                ↓
    Users             Orders          Payments
       │                │                │
       └────────────────┼────────────────┘
                        ↓
                    Database
```

Everything may exist in one codebase and one deployable application.

You might have:

```text
src/
├── modules/
│   ├── users/
│   ├── orders/
│   ├── payments/
│   └── notifications/
│
├── middleware/
├── database/
└── server.ts
```

You deploy:

```text
backend-app:v1
```

And the entire application runs together.

---

# 2. Monolith Doesn't Mean Bad Code

This is important.

A monolith can be:

```text
well structured
modular
tested
scalable
maintainable
```

A monolith can also be:

```text
messy
tightly coupled
hard to test
hard to deploy
```

Those are different problems.

Don't confuse:

> Monolith

with:

> Bad architecture.

A well-designed modular monolith can be a very good architecture.

---

# 3. What Is a Modular Monolith?

This is a useful middle ground.

Instead of putting everything into one giant folder:

```text
src/
├── users.ts
├── orders.ts
├── payments.ts
├── randomStuff.ts
└── ...
```

we organize the application into clear modules.

```text
src/
├── users/
│   ├── controller.ts
│   ├── service.ts
│   ├── repository.ts
│   └── routes.ts
│
├── orders/
│   ├── controller.ts
│   ├── service.ts
│   ├── repository.ts
│   └── routes.ts
│
└── payments/
    ├── controller.ts
    ├── service.ts
    ├── repository.ts
    └── routes.ts
```

The application is still deployed as one unit.

But internally, the boundaries are clear.

This can give you many benefits of good service boundaries without immediately introducing distributed-system complexity.

---

# 4. What Is a Microservice?

A microservice is a small, independently deployable service that owns a specific business capability.

Instead of:

```text
One Backend
│
├── Users
├── Orders
├── Payments
└── Notifications
```

we might have:

```text
User Service
Order Service
Payment Service
Notification Service
```

Each service can potentially be:

```text
developed independently
deployed independently
scaled independently
owned by a team
```

Conceptually:

```text
                     API Gateway
                          │
          ┌───────────────┼───────────────┐
          ↓               ↓               ↓
     User Service    Order Service   Payment Service
          │               │               │
          ↓               ↓               ↓
      User DB         Order DB        Payment DB
```

Now we have multiple applications instead of one.

And that changes a lot.

---

# 5. Monolith vs Microservices

Let's put them side by side.

### Monolith

```text
                 Application
                     │
        ┌────────────┼────────────┐
        ↓            ↓            ↓
      Users        Orders       Payments
        │            │            │
        └────────────┼────────────┘
                     ↓
                  Database
```

### Microservices

```text
                 API Gateway
               /      |       \
              ↓       ↓        ↓
           Users    Orders    Payments
             │        │          │
             ↓        ↓          ↓
           DB       DB          DB
```

The second architecture looks more sophisticated.

But sophistication isn't automatically an advantage.

---

# 6. Why Do Teams Move Toward Microservices?

There are several reasons.

## Independent deployments

Imagine:

```text
Payments team
```

needs to release a change.

With a monolith:

```text
Change payment code
      ↓
Deploy entire application
```

With microservices:

```text
Change Payment Service
      ↓
Deploy Payment Service
```

Other services don't necessarily need to be redeployed.

---

# 7. Independent Scaling

Suppose:

```text
Users → 1,000 requests/sec
Orders → 5,000 requests/sec
Payments → 200 requests/sec
```

In a monolith, you may scale the whole application:

```text
API × 10
```

even though only Orders really needs the extra capacity.

With separate services:

```text
User Service      × 3
Order Service     × 10
Payment Service   × 2
```

You can scale according to workload.

That's one of the strongest arguments for microservices.

---

# 8. Independent Technology Choices

A microservice architecture can allow different services to use different technologies.

For example:

```text
User Service
→ TypeScript

Recommendation Service
→ Python

Payment Service
→ Java
```

This can be useful in some organizations.

But don't interpret this as:

> "Every service should use a different language."

That creates unnecessary complexity.

If your entire team is productive with TypeScript, using TypeScript everywhere may be a much better decision.

---

# 9. Team Ownership

Microservices often become useful when organizations become large.

Imagine:

```text
Team A → Users
Team B → Orders
Team C → Payments
Team D → Notifications
```

Each team can own a service.

This can reduce coordination between teams.

Instead of:

```text
20 developers
   ↓
One giant codebase
   ↓
Everyone changes everything
```

you can have clearer ownership boundaries.

But there's a catch.

---

# 10. Microservices Create Distributed Systems

This is the part beginners often miss.

With a monolith:

```text
Function A
   ↓
Function B
   ↓
Database
```

Everything may happen inside one process.

With microservices:

```text
Service A
   ↓
Network
   ↓
Service B
   ↓
Network
   ↓
Service C
```

Now the network is involved.

And networks fail.

That's a huge architectural difference.

---

# 11. Network Calls Are Not Function Calls

Inside a monolith:

```typescript
const order = createOrder(userId);
```

The function call happens in the same application process.

With microservices:

```text
Order Service
      ↓
HTTP request
      ↓
Payment Service
```

Now you have:

```text
latency
timeouts
network failures
retries
duplicate requests
authentication
service discovery
```

This is much more complicated.

---

# 12. Partial Failure

Imagine:

```text
Order Service
      ↓
Payment Service
      ↓
Notification Service
```

What happens if Notification Service is down?

The entire order should probably not fail.

But if Payment Service is down?

Maybe the order shouldn't be completed.

Now we have to think about failure boundaries.

```text
Order
  │
  ├── Payment → Critical
  │
  └── Email   → Can happen later
```

This is why the business requirements matter.

---

# 13. Distributed Failure

In a monolith:

```text
App
 ↓
Database
```

Maybe the database goes down.

That's already a problem.

In microservices:

```text
Service A
   ↓
Service B
   ↓
Service C
   ↓
Service D
   ↓
Database
```

Now any of these can fail.

```text
A fails
B fails
C times out
D returns 500
Database is slow
Network is unavailable
```

You have many more failure modes.

---

# 14. Latency

Suppose a request needs:

```text
API
 ↓
User Service
 ↓
Order Service
 ↓
Payment Service
```

Every network call adds latency.

For example, conceptually:

```text
API → User = 20ms
User → Order = 30ms
Order → Payment = 50ms
```

Now the total request time includes those network interactions.

In a real system, parallelism, connection reuse, caching, and many other factors matter, so don't simply add numbers blindly.

The important lesson is:

> **Network calls have cost.**

---

# 15. Service-to-Service Communication

Microservices need ways to communicate.

Common options include:

```text
HTTP/REST
gRPC
Message queues
Events
```

For example:

```text
Order Service
     │
     │ HTTP
     ↓
Payment Service
```

or:

```text
Order Service
     │
     │ Event
     ↓
Message Broker
     │
     ↓
Notification Service
```

Different communication patterns solve different problems.

---

# 16. Synchronous Communication

Suppose Order Service calls Payment Service:

```text
Order Service
      │
      │ POST /payments
      ↓
Payment Service
      │
      ↓
Response
      │
      ↓
Order Service
```

The Order Service waits for the Payment Service.

This is synchronous communication.

It's straightforward.

But now Payment Service availability affects Order Service.

---

# 17. Asynchronous Communication

Now imagine sending an event:

```text
Order Service
      │
      │ OrderCreated
      ↓
Message Broker
      │
      ├──────────────→ Notification Service
      │
      └──────────────→ Analytics Service
```

The Order Service doesn't need to wait for every consumer.

This is asynchronous communication.

It can improve decoupling.

But it introduces its own complexity:

```text
event delivery
duplicates
ordering
retries
dead-letter queues
idempotency
eventual consistency
```

We've already discussed several of these in the background jobs chapter.

---

# 18. What Is an API Gateway?

With many microservices, you might not want clients calling every service directly.

Instead:

```text
Client
   ↓
API Gateway
   ↓
┌──────┬──────┬──────┐
↓      ↓      ↓
Users Orders Payments
```

The API Gateway can provide a common entry point.

It may handle things such as:

```text
routing
authentication
rate limiting
request logging
TLS termination
response aggregation
```

But don't put every piece of business logic into the gateway.

It should not become a giant new monolith.

---

# 19. Database per Service

One of the important microservice ideas is service ownership of data.

For example:

```text
User Service
     ↓
User Database

Order Service
     ↓
Order Database

Payment Service
     ↓
Payment Database
```

Why?

Because if every service directly modifies the same database tables:

```text
User Service ─┐
Order Service ├──→ Same Database
Payment ──────┘
```

then the services aren't really independent.

They are tightly coupled through the database.

---

# 20. Shared Database Problem

Suppose:

```text
User Service
Order Service
Payment Service
```

all directly access:

```text
production_db.users
production_db.orders
production_db.payments
```

Now the Order Service can modify tables owned by Payments.

That's dangerous.

You can end up with:

```text
Service boundary
      X
      ↓
Shared database coupling
```

A service should ideally own its data.

---

# 21. But Database Per Service Doesn't Mean One Database Server Per Service

This is an important distinction.

You don't necessarily need:

```text
Service A → Server A
Service B → Server B
Service C → Server C
```

You could have different databases or schemas depending on the architecture and operational requirements.

The important concept is:

> **Ownership of data should be clear.**

Physical infrastructure and logical ownership are separate decisions.

---

# 22. Distributed Transactions

Now suppose creating an order requires:

```text
Order Service
      ↓
Create order

Payment Service
      ↓
Charge customer

Inventory Service
      ↓
Reserve product
```

In a monolith using one database, this might potentially be handled inside one transaction.

With microservices:

```text
Database A
Database B
Database C
```

You can't simply use one normal local database transaction across all of them.

Now you need distributed workflow patterns.

---

# 23. Eventual Consistency

Microservices often accept that different services may temporarily have different views of the world.

For example:

```text
Order created
    ↓
Order Service knows immediately

Event published
    ↓
Inventory Service receives event
    ↓
Inventory updated
```

For a short period:

```text
Order = created
Inventory = not updated yet
```

Eventually they become consistent.

This is **eventual consistency**.

It can be completely acceptable for some workflows.

For others, it may not be.

---

# 24. Saga Pattern

One approach to distributed business workflows is the **Saga pattern**.

Imagine:

```text
Create Order
    ↓
Reserve Inventory
    ↓
Charge Payment
```

If payment fails:

```text
Payment failed
    ↓
Release inventory
    ↓
Cancel order
```

Instead of one giant database transaction, we have a sequence of local transactions with compensating actions.

Conceptually:

```text
Order Created
     ↓
Inventory Reserved
     ↓
Payment Failed
     ↓
Release Inventory
     ↓
Cancel Order
```

This is powerful.

But it's also more complicated than a normal transaction.

---

# 25. Don't Use Microservices Just for Scaling

This is a very common misconception.

Suppose your application has:

```text
100 users
```

and someone says:

> "Let's create 15 microservices so we can scale."

That's probably not solving a real problem.

You can often scale a well-designed monolith horizontally:

```text
Load Balancer
   /   |   \
  ↓    ↓    ↓
 API  API  API
   \   |   /
    Database
```

You don't need microservices just because you need more API instances.

---

# 26. Monoliths Can Scale

This is worth emphasizing.

A monolith can be deployed as:

```text
Load Balancer
   │
   ├── API 1
   ├── API 2
   ├── API 3
   └── API 4
```

All instances run the same application.

You can still use:

```text
Redis
PostgreSQL replicas
CDN
Queues
Object storage
Load balancing
```

A monolith can handle significant traffic.

Architecture isn't simply:

```text
Monolith = small
Microservices = large
```

Real systems aren't that simple.

---

# 27. The Distributed Monolith

Here's a funny but very real architecture.

You split your monolith into services:

```text
User Service
Order Service
Payment Service
```

But every request requires:

```text
User → Order → Payment → Notification → Analytics
```

and every service must be deployed together.

Now you have:

```text
Multiple services
      +
Tight coupling
```

This is sometimes called a **distributed monolith**.

You've gained the complexity of microservices without getting their main benefits.

---

# 28. Signs of a Distributed Monolith

You may have a problem if:

```text
Every deployment requires every service
Every service depends on every other service
Services share the same database tables
Services cannot work independently
One service outage takes down everything
```

You basically moved the monolith over the network.

---

# 29. Service Boundaries Matter More Than Service Count

Don't ask:

> "How many microservices should we have?"

Ask:

> "Where are the business boundaries?"

For an e-commerce system:

```text
Users
Orders
Payments
Inventory
Shipping
Notifications
```

might be meaningful domains.

But perhaps:

```text
UserNameService
UserEmailService
UserAvatarService
```

are unnecessarily fragmented.

A service should generally represent a meaningful capability, not a random collection of endpoints.

---

# 30. The Two-Pizza Rule

You may hear the phrase:

> "Two-pizza team."

It's a popular idea associated with keeping teams small enough that they can be fed by two pizzas.

The exact team size isn't a technical law.

The useful lesson is:

> Service boundaries often make more sense when they align with team ownership and business responsibilities.

Don't turn a slogan into an architecture rule.

---

# 31. Microservices and Deployment

With a monolith:

```text id="y8c9c1"
Code
 ↓
Build
 ↓
One application
 ↓
Deploy
```

With microservices:

```text id="q2c3ef"
User Service
   ↓
Build
   ↓
Deploy

Order Service
   ↓
Build
   ↓
Deploy

Payment Service
   ↓
Build
   ↓
Deploy
```

Now CI/CD becomes much more important.

You need to manage:

```text
versions
deployments
rollbacks
compatibility
configuration
secrets
service discovery
monitoring
```

---

# 32. Version Compatibility

Imagine Order Service expects:

```text
Payment API v2
```

but Payment Service is deployed with:

```text
Payment API v1
```

Now things break.

In a monolith, changing a function signature can often be caught during the same build/test process.

With microservices, compatibility crosses deployment boundaries.

This means APIs need careful evolution.

---

# 33. Backward-Compatible APIs

Suppose Payment Service currently exposes:

```text
POST /payments
```

and you need to change its response.

Instead of immediately breaking all clients, you might introduce:

```text
v1
v2
```

or make the change backward compatible.

This is another reason API design from Chapter 12 becomes important.

---

# 34. Service Discovery

If services communicate with each other, they need to know where other services are.

For example:

```text
Order Service
     ↓
Where is Payment Service?
```

In a simple Docker Compose environment, service names can be enough:

```text
payment:3000
```

In larger production environments, you may have service discovery through your infrastructure/orchestration platform.

The important idea:

> Services need a reliable way to locate each other.

---

# 35. Timeouts

Imagine:

```text
Order Service
     ↓
Payment Service
```

Payment Service stops responding.

If Order Service waits forever:

```text
Request 1 → waiting
Request 2 → waiting
Request 3 → waiting
...
```

Soon your Order Service may run out of resources.

This is why network calls need timeouts.

For example:

```text
Request
  ↓
Wait 2 seconds
  ↓
No response
  ↓
Timeout
```

Exact timeout values depend on the operation.

---

# 36. Retries

If a network request fails temporarily, you may retry.

```text
Request
   ↓
Failure
   ↓
Retry
   ↓
Success
```

But retries can be dangerous.

Suppose you send:

```text
Charge customer $100
```

The request reaches the payment service.

Payment succeeds.

But the response is lost.

Your client thinks:

```text
Request failed
```

and retries.

Now you might charge the customer twice.

This is why retries and **idempotency** go together.

---

# 37. Idempotency in Microservices

For operations that can be retried safely, use an idempotency mechanism.

For example:

```text
Idempotency-Key: abc123
```

Payment Service can remember:

```text
abc123 → Payment already processed
```

Then a duplicate request doesn't create a second payment.

This concept becomes extremely important in distributed systems.

---

# 38. Circuit Breaker

Imagine:

```text
Order Service
     ↓
Payment Service
```

Payment Service is completely down.

Instead of making thousands of requests that all fail:

```text
Request
 ↓
Payment
 ↓
Timeout
 ↓
Retry
 ↓
Timeout
 ↓
Retry
```

a circuit breaker can temporarily stop sending requests.

Conceptually:

```text
Healthy
  ↓
Calls allowed

Repeated failures
  ↓
Circuit opens
  ↓
Calls rejected quickly
```

Later, the system can test whether the dependency has recovered.

This protects services from cascading failures.

---

# 39. Bulkheads

Another useful reliability idea is the **bulkhead pattern**.

Imagine your application has:

```text
Payment requests
Search requests
Notification requests
```

If notification traffic suddenly explodes, you don't want it consuming all resources needed for payments.

You can isolate resources:

```text
Payment capacity
      │
      └── protected

Notification capacity
      │
      └── separate
```

This reduces the chance that one workload takes down everything else.

---

# 40. Observability Becomes More Important

In a monolith, you might investigate:

```text
Request
 ↓
Application logs
 ↓
Database
```

In microservices:

```text
Request
 ↓
API Gateway
 ↓
User Service
 ↓
Order Service
 ↓
Payment Service
 ↓
Database
```

If the request takes 5 seconds:

> Which service caused the delay?

Without proper observability, this can be painful.

That's why we need:

```text
logs
metrics
traces
request IDs
correlation IDs
```

from Chapter 17.

---

# 41. Distributed Tracing

A trace can follow one request across services.

Conceptually:

```text
Trace ID: abc123

API Gateway
   │
   ├── User Service
   │
   ├── Order Service
   │      │
   │      └── Database
   │
   └── Payment Service
```

Now you can see:

```text
Gateway       20ms
User Service  30ms
Order Service 800ms
Payment       40ms
```

You immediately have a much better idea where the latency came from.

---

# 42. Security Between Services

Don't assume:

> "It's internal traffic, so it's trusted."

Internal services still need protection.

Depending on the environment, you may need:

```text
authentication
authorization
TLS
service identities
network policies
secret management
```

For example:

```text
Order Service
     ↓
authenticated request
     ↓
Payment Service
```

Payment Service should still verify that the caller is allowed to perform the operation.

---

# 43. Monolith Advantages

Let's be fair.

Monoliths have many advantages.

### Simpler development

```text
One codebase
One application
```

### Easier local development

You don't need to start:

```text
15 services
```

just to test one feature.

### Easier debugging

The entire call path may exist in one process.

### Simpler transactions

A single database can make transactional workflows easier.

### Lower operational overhead

Fewer deployments.

Fewer services.

Fewer dashboards.

Fewer network boundaries.

---

# 44. Monolith Disadvantages

Monoliths can become difficult when they grow without discipline.

Potential problems:

```text
Large codebase
Tight coupling
Long build times
Large deployments
Difficult team ownership
Harder independent scaling
```

But many of these problems can be reduced with good modular design.

---

# 45. Microservices Advantages

Microservices can provide:

```text
Independent deployments
Independent scaling
Clear ownership boundaries
Fault isolation
Team autonomy
Potential technology flexibility
```

For large organizations, these can be very valuable.

---

# 46. Microservices Disadvantages

The cost is significant.

You now need to deal with:

```text
Network failures
Service discovery
Distributed tracing
Retries
Timeouts
API compatibility
Distributed transactions
Eventual consistency
Multiple deployments
Multiple databases
More infrastructure
More monitoring
More operational complexity
```

That's a lot.

---

# 47. A Simple Comparison

| Area                 | Monolith             | Microservices                    |
| -------------------- | -------------------- | -------------------------------- |
| Codebase             | Usually one          | Multiple                         |
| Deployment           | One unit             | Independent services             |
| Local development    | Easier               | More complex                     |
| Debugging            | Simpler              | Distributed                      |
| Scaling              | Often whole app      | Per service                      |
| Transactions         | Simpler              | More difficult                   |
| Network calls        | Fewer internal calls | Many                             |
| Team ownership       | Can become shared    | Clearer boundaries               |
| Infrastructure       | Simpler              | More complex                     |
| Operational overhead | Lower                | Higher                           |
| Failure modes        | Fewer                | More                             |
| Best for             | Many teams/stages    | Larger/distributed organizations |

There isn't one universally correct choice.

---

# 48. When Should You Use a Monolith?

A monolith is often a good choice when:

```text
You're starting a product
Small team
Unclear requirements
Small-to-medium system
Simple deployment needs
One main database
```

Especially when you're still discovering the product.

You don't know yet which boundaries will actually make sense.

That's a good reason not to split too early.

---

# 49. When Should You Consider Microservices?

Microservices become more attractive when you have real problems such as:

```text
Large engineering organization
Clear domain boundaries
Different scaling requirements
Independent deployment requirements
Strong team ownership
Very different reliability requirements
```

Notice the word:

> **real**

Don't adopt microservices because someone on YouTube said:

> "Every serious company uses microservices."

Architecture should follow problems.

---

# 50. The Modular Monolith → Microservices Path

One practical approach is:

```text
Start
  ↓
Monolith
  ↓
Modular Monolith
  ↓
Identify strong boundaries
  ↓
Extract one service
  ↓
Measure
  ↓
Extract another if necessary
```

For example:

```text
Initial:

Monolith
├── Users
├── Orders
├── Payments
└── Notifications
```

Later:

```text
Monolith
├── Users
├── Orders
└── Notifications

Payment Service
```

Later:

```text
Monolith
├── Users
└── Notifications

Order Service
Payment Service
```

You don't have to split everything at once.

---

# 51. Strangler Pattern

A common migration strategy is to gradually move functionality out of an existing monolith.

Conceptually:

```text
                 Client
                    ↓
               Entry Layer
                /       \
               ↓         ↓
          Old Monolith   New Service
```

Over time:

```text
More traffic
     ↓
New Service
```

and eventually:

```text
Old functionality
     ↓
Removed
```

This is often safer than trying to rewrite the entire application in one giant migration.

---

# 52. Don't Rewrite Everything

Suppose your monolith works.

Someone proposes:

> "Let's rewrite everything as microservices."

That's a huge risk.

You may introduce:

```text
new bugs
new infrastructure
new deployment problems
new distributed-system failures
```

without actually solving the existing business problem.

A better approach is usually:

> Identify a specific pain point and solve that pain point.

---

# 53. A Real Example

Imagine our application is an e-commerce platform.

Initially:

```text
                 Monolith
                    │
       ┌────────────┼────────────┐
       ↓            ↓            ↓
     Users        Orders       Payments
       │            │            │
       └────────────┼────────────┘
                    ↓
                 Postgres
```

The team grows.

Payments now need:

```text
strong isolation
different deployment schedule
strict access control
different scaling requirements
```

We might extract:

```text
                 Monolith
              /           \
             ↓             ↓
          Users          Orders
                            │
                            ↓
                     Payment Service
                            │
                            ↓
                       Payment DB
```

Now we have a clear reason.

Not:

> "Microservices are cool."

But:

> "Payments have different operational and security requirements."

That's an engineering decision.

---

# 54. Microservices Don't Automatically Improve Performance

This is another myth.

Splitting:

```text
One application
```

into:

```text
10 services
```

doesn't automatically make it faster.

You might actually make it slower because of:

```text
network calls
serialization
deserialization
extra infrastructure
service coordination
```

Performance comes from architecture and workload optimization, not from the number of repositories.

---

# 55. Microservices Don't Automatically Improve Reliability

You might think:

```text
Many services
   ↓
If one fails, everything else works
```

Sometimes that's true.

But bad service dependencies can create:

```text
Service A
 ↓
Service B
 ↓
Service C
 ↓
Service D
```

If C fails, A may fail too.

Microservices can improve fault isolation.

They can also create cascading failures.

It depends on how they're designed.

---

# 56. Interview Questions

## Q1. What is a monolith?

A monolith is an application where multiple business capabilities are developed and deployed as one application unit.

---

## Q2. What is a microservice?

A microservice is an independently deployable service focused on a specific business capability.

---

## Q3. Are monoliths bad?

No.

A well-structured monolith can be simple, maintainable, scalable, and appropriate for many systems.

---

## Q4. What is a modular monolith?

A modular monolith is a single deployable application organized into clearly separated internal modules with defined boundaries.

---

## Q5. What are the advantages of microservices?

Common advantages include:

```text
Independent deployment
Independent scaling
Team ownership
Fault isolation
Clear business boundaries
```

---

## Q6. What are the disadvantages?

Common disadvantages include:

```text
Network failures
Distributed transactions
Operational complexity
Monitoring complexity
Service discovery
API compatibility
Eventual consistency
```

---

## Q7. Why shouldn't a startup immediately use microservices?

Because early in a product's life, requirements and boundaries are uncertain.

Microservices introduce significant operational and distributed-system complexity before you necessarily have a problem that requires it.

---

## Q8. Can a monolith scale?

Yes.

A monolithic application can be horizontally scaled by running multiple instances behind a load balancer and using appropriate shared infrastructure such as databases, caches, and queues.

---

## Q9. What is a distributed monolith?

A distributed monolith is a system split into multiple services but with strong coupling between them.

For example:

```text
Every service
   ↓
Depends on every other service
```

and:

```text
Every deployment
   ↓
Requires all services
```

You get microservice complexity without much of the independence benefit.

---

## Q10. Why is a network call different from a function call?

A function call within a process is generally local and predictable.

A network call can fail, timeout, experience latency, be duplicated, or return an error.

Distributed systems must explicitly handle these failure modes.

---

## Q11. What is eventual consistency?

Eventual consistency means different parts of a distributed system may temporarily have different states, but the system is designed for them to converge over time.

---

## Q12. What is service discovery?

Service discovery is the mechanism that allows services to find the network location of other services they need to communicate with.

---

## Q13. What is an API Gateway?

An API Gateway is an entry point between clients and backend services.

It can handle concerns such as routing, authentication, rate limiting, and request aggregation.

---

## Q14. Why is database-per-service commonly recommended?

It creates clearer data ownership and reduces direct coupling between services through shared database tables.

---

## Q15. Does database-per-service mean every service must have its own physical database server?

No.

The important concept is logical ownership and controlled access. Physical infrastructure can vary depending on operational requirements.

---

# 57. Scenario-Based Interview Questions

## Scenario 1

> "We're a five-person startup. Should we build 15 microservices?"

I'd probably start with a modular monolith unless there is a strong specific reason to separate services.

The team needs to optimize for:

```text
speed
simplicity
learning
product iteration
```

Microservices would add significant operational overhead.

---

## Scenario 2

> "Our Orders module needs to scale much more than the rest of our application."

I'd first investigate whether the monolith can be horizontally scaled and whether the actual bottleneck is elsewhere.

If Orders has a genuinely different scaling requirement and a clear boundary, extracting it into a service could eventually make sense.

---

## Scenario 3

> "Payment Service is down. Should Order Service immediately fail?"

It depends on the business requirement.

If payment confirmation is required before an order can be finalized, the order may need to remain pending.

If some other operation is non-critical, it might be asynchronous.

The important part is defining:

```text
critical dependency
vs
non-critical dependency
```

---

## Scenario 4

> "Our microservices are slow because every request calls five other services."

I'd investigate:

```text
Service dependency graph
Network latency
Sequential calls
Parallelization opportunities
Caching
Unnecessary service boundaries
Timeouts
Database queries
```

Maybe the architecture is too fragmented.

---

## Scenario 5

> "Every service shares the same PostgreSQL database."

I'd ask why.

If every service can directly access every table, the services are strongly coupled.

I'd work toward clearer ownership boundaries and controlled communication.

I wouldn't blindly create separate databases overnight; database migration is a significant change that needs a careful plan.

---

# 58. Mini Project — Start With a Modular Monolith

Instead of immediately building 10 microservices, let's practice good boundaries first.

Take the Todo API.

Start with:

```text
todo-api/
│
├── users/
├── todos/
├── notifications/
└── auth/
```

Keep it as one application.

But define boundaries.

For example:

```text
users/
    service.ts

todos/
    service.ts

notifications/
    service.ts
```

Try not to let every module directly modify another module's internals.

---

# 59. Then Identify a Service Candidate

Imagine notifications become much more complicated.

Now:

```text
Todo API
    │
    └── Notification Module
```

becomes:

```text
Todo API
    │
    │ event
    ↓
Notification Service
    │
    ↓
Email / Push
```

For example:

```text
Todo completed
      ↓
Todo Service
      ↓
TodoCompleted event
      ↓
Message Queue
      ↓
Notification Service
```

Now you have a reason for the boundary.

---

# 60. Think About Failure

Once you extract the service, intentionally break it.

Ask:

```text
What if Notification Service is down?
What if the queue is down?
What if the event is delivered twice?
What if notification processing is slow?
What if the service times out?
```

Then design the system accordingly.

This is where you start thinking like someone designing distributed systems rather than simply writing endpoints.

---

# 61. What You Should Actually Learn From This Chapter

Don't memorize:

```text
Microservices = good
Monolith = bad
```

That's not engineering.

Instead remember:

```text
Monolith
    ↓
Simple
    ↓
Low operational complexity
    ↓
Great starting point

Microservices
    ↓
Independent boundaries
    ↓
Independent deployment/scaling
    ↓
Higher distributed-system complexity
```

And:

> **The architecture should follow the problem.**

---

# 62. Final Decision Framework

When deciding between monolith and microservices, ask:

```text
1. How big is the team?
2. Are the business boundaries clear?
3. Do different parts need independent scaling?
4. Do they need independent deployments?
5. Do teams need independent ownership?
6. Can we handle distributed-system complexity?
7. Do we actually need separate data ownership?
8. Can a modular monolith solve the problem?
```

If the answer to most of these is:

```text
No
```

a monolith may be the better choice.

If several answers are:

```text
Yes
```

microservices may be worth considering.

---

# 63. Final Mental Model

Keep this picture in your head.

### Monolith

```text
             One Application
                   │
       ┌───────────┼───────────┐
       ↓           ↓           ↓
     Users       Orders      Payments
       │           │           │
       └───────────┼───────────┘
                   ↓
                Database
```

Simple.

---

### Modular Monolith

```text
             One Application
                   │
       ┌───────────┼───────────┐
       ↓           ↓           ↓
    Users       Orders      Payments
   Module       Module       Module
```

Still one deployment.

But with clear boundaries.

---

### Microservices

```text
                 API Gateway
                      │
       ┌──────────────┼──────────────┐
       ↓              ↓              ↓
   User Service   Order Service  Payment Service
       │              │              │
       ↓              ↓              ↓
      DB             DB             DB
```

Independent services.

But now:

```text
Network
Retries
Timeouts
Failures
Events
Tracing
Distributed data
```

become part of your engineering problem.

---

# The Big Lesson

If there is one thing I want you to remember from this chapter, it's this:

> **Don't use microservices because they look like a scalable architecture. Use them when independent boundaries, scaling, ownership, or deployment requirements justify the complexity.**

A clean monolith is often better than a badly designed microservice system.

And a well-designed microservice architecture can be extremely powerful when the organization and product actually need it.

The goal isn't to have more services.

The goal is to build a system that is **easy enough to change, reliable enough to operate, and scalable enough for the workload it actually has.**

---

# What's Next?

We've now talked about:

```text
Application
    ↓
Database
    ↓
Cache
    ↓
Queues
    ↓
Docker
    ↓
Database scaling
    ↓
Architecture boundaries
```

But none of this matters if we can't reliably get our code from:

```text
Developer Laptop
```

to:

```text
Production
```

without manually SSH-ing into servers and running commands every time.

We need:

```text
Git
   ↓
Tests
   ↓
Build
   ↓
Docker Image
   ↓
Registry
   ↓
Deployment
   ↓
Production
```

And that brings us to:

# Chapter 23 — CI/CD & Deployment