# Chapter 23 — CI/CD & Deployment

So far, we've mostly talked about how to **build** a backend.

Now let's talk about something just as important:

> **How do you actually get that backend into production?**

Imagine you finish a feature.

You push your code to Git.

Then what?

Maybe someone manually:

```text
SSH into server
Pull latest code
Install dependencies
Run tests
Build application
Restart server
Check logs
Hope everything works
```

This might work for a small project.

But imagine doing this:

```text
10 times a day
50 times a day
100 times a day
```

Very quickly, it becomes painful.

Worse, humans make mistakes.

Maybe someone forgets to run the tests.

Maybe they deploy the wrong branch.

Maybe they restart the application before the new build is ready.

Maybe the environment variable is missing.

Maybe production is now broken.

This is where **CI/CD** comes in.

---

# 1. What Is CI/CD?

CI/CD is a collection of practices and automation around building, testing, and delivering software.

The two main ideas are:

```text
CI
Continuous Integration

CD
Continuous Delivery / Continuous Deployment
```

Let's understand them separately.

---

# 2. What Is Continuous Integration?

Continuous Integration means developers frequently integrate their changes into a shared codebase and use automated checks to catch problems early.

A simplified flow:

```text
Developer
    ↓
git push
    ↓
CI starts
    ↓
Install dependencies
    ↓
Run tests
    ↓
Run lint/type checks
    ↓
Build application
    ↓
Pass / Fail
```

The important idea:

> **Every change gets automatically checked.**

Instead of discovering a problem after deployment, we try to catch it before it reaches production.

---

# 3. Why CI Matters

Imagine three developers work on the same backend.

Developer A changes authentication.

Developer B changes payments.

Developer C changes database code.

Without automated checks:

```text
Developer A → merge
Developer B → merge
Developer C → merge
        ↓
Production
        ↓
💥 Something breaks
```

With CI:

```text
Developer
   ↓
Pull Request
   ↓
CI
   ├── Tests
   ├── TypeScript
   ├── Lint
   └── Build
        ↓
    Pass / Fail
```

The team gets feedback much earlier.

---

# 4. What Is Continuous Delivery?

Continuous Delivery means keeping the application in a state where it can be released reliably, usually through an automated pipeline.

A simplified flow:

```text
Code
 ↓
CI
 ↓
Tests
 ↓
Build
 ↓
Artifact
 ↓
Ready to deploy
```

The deployment may still require a human to approve it.

For example:

```text
Production deployment
        ↓
Manual approval
        ↓
Deploy
```

---

# 5. What Is Continuous Deployment?

Continuous Deployment goes one step further.

When the pipeline passes its checks, the application can automatically be deployed.

```text
Git Push
   ↓
Tests
   ↓
Build
   ↓
Deploy
   ↓
Production
```

No manual production approval is required for every change.

So:

```text
Continuous Delivery
→ Ready to deploy

Continuous Deployment
→ Automatically deploy
```

People sometimes use "CD" for both.

The exact process depends on the organization.

---

# 6. CI/CD Is Not Just "Run Tests"

A beginner might think:

> CI/CD = GitHub Actions running npm test.

That's only one small part.

A real deployment pipeline can involve:

```text
Source control
      ↓
CI
      ↓
Tests
      ↓
Security checks
      ↓
Build
      ↓
Docker image
      ↓
Container registry
      ↓
Deployment
      ↓
Health checks
      ↓
Monitoring
```

Now connect this to Chapter 20.

Docker gives us a consistent artifact:

```text
Docker Image
```

CI/CD can build and deliver that artifact.

---

# 7. The Modern Backend Deployment Flow

A realistic simplified flow:

```text
Developer
    │
    ↓
Git Push
    │
    ↓
Pull Request
    │
    ↓
CI
    │
    ├── Tests
    ├── Type Check
    ├── Lint
    └── Build
    │
    ↓
Docker Image
    │
    ↓
Container Registry
    │
    ↓
Deploy
    │
    ↓
Production
    │
    ↓
Health Check
    │
    ↓
Traffic
```

This is the picture I want you to understand.

---

# 8. What Is a Deployment?

Deployment simply means making a new version of your application available in an environment.

For example:

```text
Version 1
    ↓
Production

Version 2
    ↓
Production
```

Deployment can happen in many ways.

You might deploy to:

```text
Virtual machine
Container platform
Kubernetes
Managed cloud service
Serverless platform
Bare metal
```

The infrastructure can change.

The core process remains similar:

```text
Build
→ Release
→ Run
→ Verify
```

---

# 9. Build vs Release vs Deploy

These terms can be confusing.

### Build

Turn source code into something runnable.

For example:

```text
TypeScript
   ↓
JavaScript
   ↓
Docker Image
```

### Release

Identify a specific version that is ready to be deployed.

For example:

```text
backend:v1.8.2
```

### Deploy

Actually make that version run in an environment.

```text
backend:v1.8.2
        ↓
Production
```

Keeping these concepts separate makes deployment systems easier to reason about.

---

# 10. Why Versioned Artifacts Matter

Imagine production is running:

```text
backend:v1.4.0
```

You deploy:

```text
backend:v1.5.0
```

Something breaks.

If the old version still exists, you can potentially roll back:

```text
v1.5.0
   ↓
Problem

Rollback

v1.4.0
   ↓
Working version
```

This is one reason immutable, versioned artifacts are so useful.

Don't deploy:

```text
latest
```

and then lose track of exactly what production is running.

Use meaningful, traceable versions such as:

```text
commit SHA
release version
build number
```

---

# 11. What Is a Deployment Pipeline?

A pipeline is a sequence of automated steps.

For example:

```text
Push
 ↓
Install
 ↓
Test
 ↓
Lint
 ↓
Type Check
 ↓
Build
 ↓
Docker Build
 ↓
Push Image
 ↓
Deploy
 ↓
Health Check
```

Each step should have a clear purpose.

If a critical step fails:

```text
Pipeline
   ↓
FAIL
   ↓
Don't continue
```

This is much safer than:

```text
Tests failed
   ↓
Deploy anyway
```

---

# 12. Example CI Pipeline

A simple Node.js backend pipeline might look like:

```yaml
steps:
  - checkout

  - install dependencies

  - run tests

  - run type check

  - run lint

  - build application
```

The exact YAML depends on your CI platform.

The important thing is the process.

---

# 13. Pull Requests and CI

A common workflow is:

```text
Developer creates branch
        ↓
Makes changes
        ↓
Pushes branch
        ↓
Opens Pull Request
        ↓
CI runs
        ↓
Tests pass
        ↓
Code review
        ↓
Merge
```

This gives teams a safety gate before code enters the main branch.

For example:

```text
main
  │
  ├── feature/auth
  ├── feature/payments
  └── fix/pagination
```

Each branch can be tested independently.

---

# 14. What Should CI Check?

A backend CI pipeline might include:

### Tests

```text
Unit tests
Integration tests
API tests
```

### Type checking

```bash
tsc --noEmit
```

### Linting

```text
ESLint
```

### Build

```text
npm run build
```

### Security checks

Depending on the project:

```text
dependency scanning
secret scanning
container scanning
```

You don't need every possible check.

The goal is to catch meaningful problems automatically.

---

# 15. Why Run the Build in CI?

Suppose developers run:

```bash
npm test
```

and tests pass.

But:

```bash
npm run build
```

fails.

If CI doesn't run the build, the broken code can still be merged.

So your CI should verify the artifact you actually intend to deploy.

For a TypeScript backend:

```text
Source
 ↓
Type check
 ↓
Build
 ↓
Runnable application
```

---

# 16. Environment Differences

Remember Chapter 19.

You might have:

```text
Development
Staging
Production
```

The application code should ideally be the same artifact.

For example:

```text
Same Docker Image
      │
      ├── Staging configuration
      │
      └── Production configuration
```

Don't build a completely different application just because you're deploying to production.

Configuration changes.

The application artifact should ideally remain consistent.

---

# 17. Staging Environment

A staging environment is an environment used to test the application before production.

Conceptually:

```text
Developer
   ↓
CI
   ↓
Staging
   ↓
Verification
   ↓
Production
```

Staging should resemble production enough to catch important deployment/configuration problems.

It doesn't have to be an exact copy.

That can be expensive.

The goal is to make it representative of important production behavior.

---

# 18. Why Staging Helps

Imagine your code works locally:

```text
localhost
```

but production uses:

```text
HTTPS
PostgreSQL
Redis
Docker
different environment variables
external services
```

A staging environment can catch problems like:

```text
missing environment variable
database migration failure
container startup failure
wrong configuration
service communication issue
```

before production.

---

# 19. Database Migrations in CI/CD

This is one of the most important deployment concerns for backend developers.

Suppose you add:

```text
phone_number
```

to the users table.

Your code expects:

```text
users.phone_number
```

but production's database doesn't have that column yet.

You deploy the code.

Now:

```text
Application
   ↓
Database
   X
```

This can break production.

So database migrations need to be part of your deployment strategy.

---

# 20. Migration Ordering

A dangerous deployment can look like:

```text
Deploy new code
      ↓
New code expects new column
      ↓
Database hasn't been migrated
      ↓
💥
```

A safer approach often involves backward-compatible changes.

For example:

```text
Step 1
Add nullable column

Step 2
Deploy code that can work with old + new schema

Step 3
Backfill data

Step 4
Start requiring the new field

Step 5
Remove old structure later
```

This is often called an **expand-and-contract** style migration.

---

# 21. Expand and Contract

Suppose we want to rename:

```text
username
```

to:

```text
display_name
```

Don't necessarily do:

```text
Rename column
   ↓
Immediately deploy new code
```

Instead:

```text
Phase 1
Add display_name
        ↓
Phase 2
Application supports both
        ↓
Phase 3
Backfill display_name
        ↓
Phase 4
Application stops using username
        ↓
Phase 5
Remove username
```

This makes rolling deployments safer.

---

# 22. Zero-Downtime Deployment

Suppose your production server is running:

```text
Version 1
```

and you need to deploy:

```text
Version 2
```

A naive deployment might be:

```text
Stop V1
   ↓
Deploy V2
   ↓
Start V2
```

During this time:

```text
Users
  ↓
💥 downtime
```

A zero-downtime approach tries to keep capacity available while the new version is starting.

---

# 23. Rolling Deployment

Suppose we have:

```text
Load Balancer
   │
   ├── V1
   ├── V1
   └── V1
```

We want:

```text
V2
```

A rolling deployment might do:

```text
Replace one instance

V2
V1
V1
```

Then:

```text
V2
V2
V1
```

Then:

```text
V2
V2
V2
```

The load balancer continues sending traffic to healthy instances.

Conceptually:

```text
Before
V1 V1 V1

During
V2 V1 V1
V2 V2 V1

After
V2 V2 V2
```

---

# 24. Blue-Green Deployment

Another strategy is blue-green deployment.

You have:

```text
Blue
→ Current production
```

and:

```text
Green
→ New version
```

Conceptually:

```text
             Load Balancer
                  │
             ┌────┴────┐
             ↓         ↓
           Blue      Green
            V1         V2
```

You deploy and test Green.

Then switch traffic:

```text
Before:

Traffic → Blue


After:

Traffic → Green
```

If something goes wrong, you can potentially switch back:

```text
Traffic → Blue
```

This can make rollback fast, although it may require additional infrastructure.

---

# 25. Canary Deployment

Canary deployment sends a small amount of traffic to the new version first.

For example:

```text
Traffic
  │
  ├── 95% → V1
  └── 5%  → V2
```

Monitor:

```text
error rate
latency
CPU
business metrics
```

If everything looks good:

```text
5%
 ↓
25%
 ↓
50%
 ↓
100%
```

If V2 starts failing:

```text
Stop rollout
 ↓
Send traffic back to V1
```

This reduces blast radius.

---

# 26. What Is a Health Check?

After deploying a new version, you shouldn't immediately assume:

> "Container started, therefore application works."

You need to verify it.

For example:

```text
GET /health
```

might return:

```json
{
  "status": "ok"
}
```

But a good health system often distinguishes between different kinds of checks.

---

# 27. Liveness vs Readiness

### Liveness

Asks:

> "Is this application process alive?"

If not, the platform may restart it.

### Readiness

Asks:

> "Is this instance ready to receive traffic?"

An application can be alive but not ready.

For example:

```text
Application process
      ↓
Running
      ↓
Database connection unavailable
      ↓
Not ready
```

The load balancer shouldn't necessarily send normal traffic to it.

This connects directly with Chapter 17 and Docker health checks.

---

# 28. Graceful Shutdown

Imagine your server receives:

```text
SIGTERM
```

because it is being replaced.

You don't want to immediately kill it while requests are running.

Instead:

```text
SIGTERM
  ↓
Stop accepting new requests
  ↓
Finish active requests
  ↓
Close DB connections
  ↓
Close Redis connection
  ↓
Stop workers
  ↓
Exit
```

This is called **graceful shutdown**.

It's extremely useful during deployments.

---

# 29. Why Graceful Shutdown Matters

Without it:

```text
Deployment
   ↓
Kill container
   ↓
Request interrupted
   ↓
User receives error
```

With graceful shutdown:

```text
Deployment
   ↓
Stop new traffic
   ↓
Finish active work
   ↓
Shutdown cleanly
```

Not every request can always finish, but graceful shutdown reduces avoidable failures.

---

# 30. Rollbacks

Deployment isn't complete just because version 2 was released.

You need to know:

> "What happens if version 2 is broken?"

A rollback means returning to a previous known-good version.

```text
V1
 ↓
V2
 ↓
Problem
 ↓
Rollback
 ↓
V1
```

This is why versioned artifacts matter.

If you can identify exactly what was deployed, rollback becomes much easier.

---

# 31. Rollback Isn't Always Simple

Application rollback can be easy:

```text
V2 → V1
```

Database rollback can be much harder.

Suppose V2 changes the schema:

```text
Add new column
Remove old column
```

If V2 partially changes production data, simply running V1 may not work.

This is another reason backward-compatible migrations are important.

---

# 32. Feature Flags

Sometimes you deploy code without immediately enabling the feature.

For example:

```text
Feature
   ↓
Code deployed
   ↓
Flag OFF
```

Later:

```text
Flag ON
   ↓
10% users
   ↓
50%
   ↓
100%
```

This separates:

```text
Deployment
```

from:

```text
Feature release
```

That's a powerful production technique.

---

# 33. Why Feature Flags Are Useful

Suppose you deploy:

```text
New checkout system
```

but something goes wrong.

Instead of rolling back the entire application, you can potentially:

```text
Disable checkout_v2
```

and keep the application running.

Feature flags can also help with:

```text
gradual rollout
A/B testing
internal testing
emergency disablement
```

But remember:

> Feature flags are configuration/state and need lifecycle management.

Don't let your codebase fill up with flags nobody remembers.

---

# 34. Secrets in CI/CD

Your pipeline may need access to:

```text
database credentials
cloud credentials
registry credentials
API keys
signing keys
```

Do not put these directly into source code.

A better pattern is:

```text
CI Secret Store
       ↓
Pipeline
       ↓
Deployment
```

Secrets should be:

```text
protected
limited
rotatable
audited
```

And use least privilege.

For example:

> A CI job that only needs to push a container image shouldn't have unrestricted access to every production system.

---

# 35. Container Registry

From Chapter 20:

```text
Dockerfile
    ↓
Docker Image
```

Now CI can:

```text
docker build
    ↓
Tag image
    ↓
Push image
    ↓
Container Registry
```

For example:

```text
backend:commit-abc123
```

Production can then pull that exact image.

This creates a clean artifact flow:

```text
Git Commit
    ↓
Build
    ↓
Docker Image
    ↓
Registry
    ↓
Production
```

---

# 36. Why Don't We Build Separately on the Production Server?

Imagine:

```text
Developer
   ↓
Source code
   ↓
Production server
   ↓
npm install
   ↓
npm build
```

Now production is responsible for building your application.

That can make deployments less predictable.

Instead:

```text
CI
 ↓
Build
 ↓
Test
 ↓
Docker Image
 ↓
Registry
 ↓
Production
```

Production receives an already-built artifact.

This is a cleaner deployment model.

---

# 37. Immutable Deployments

The idea is:

> Build an artifact once and deploy that exact artifact.

For example:

```text
Commit abc123
     ↓
Docker Image abc123
     ↓
Staging
     ↓
Production
```

You don't rebuild a slightly different version for production.

This improves reproducibility.

If staging ran:

```text
image abc123
```

production should ideally run:

```text
image abc123
```

not:

```text
"the same source code, rebuilt somehow"
```

---

# 38. Deployment Environments

A typical setup might be:

```text
Development
     ↓
CI
     ↓
Staging
     ↓
Production
```

Each environment can have different:

```text
configuration
secrets
database
traffic
resources
```

But the application artifact can remain the same.

---

# 39. Production Is Different From Local Development

Locally you might run:

```bash
npm run dev
```

Production might run:

```text
Docker container
      ↓
Node.js
      ↓
Compiled application
```

Local development may use:

```text
hot reload
debug logging
local PostgreSQL
local Redis
```

Production might use:

```text
managed database
managed Redis
centralized logs
monitoring
alerts
multiple API instances
```

Don't confuse:

> "It runs locally"

with:

> "It's production-ready."

---

# 40. What Happens When a Deployment Fails?

Let's say:

```text
CI
 ↓
Tests pass
 ↓
Build succeeds
 ↓
Deploy
 ↓
Health check fails
```

The pipeline should not blindly send all traffic to the broken version.

A good deployment system might:

```text
Deployment
   ↓
New version starts
   ↓
Health check fails
   ↓
Stop rollout
   ↓
Keep old version
```

Or if traffic has already moved:

```text
Detect failure
   ↓
Rollback
```

The exact behavior depends on the platform.

---

# 41. Observability During Deployment

You should watch:

```text
Error rate
Latency
Request volume
CPU
Memory
Database connections
Database latency
Queue depth
```

Suppose after deployment:

```text
Before deployment
5xx → 0.2%

After deployment
5xx → 8%
```

That's a strong signal that something went wrong.

Deployment systems and monitoring systems should work together.

---

# 42. Deployment Metrics

Some useful deployment metrics include:

### Deployment frequency

How often do you deploy?

### Lead time for changes

How long does it take for a change to go from code to production?

### Change failure rate

How often do deployments cause failures or require remediation?

### Time to restore

How quickly can you recover from a failed deployment?

These metrics help teams understand the health of their delivery process.

Don't optimize them blindly.

The goal is reliable delivery, not:

> "Deploy 200 times a day just to say we do."

---

# 43. Common CI/CD Mistakes

## Mistake 1 — Deploying Without Tests

```text
git push
 ↓
production
```

That's risky.

---

## Mistake 2 — Using `latest` Everywhere

You don't always know exactly what version is running.

Prefer immutable/versioned image references.

---

## Mistake 3 — Building Differently in Every Environment

```text
Staging → build A
Production → build B
```

Now you don't know whether you've actually tested the production artifact.

---

## Mistake 4 — Ignoring Database Migrations

Code and schema must evolve together.

---

## Mistake 5 — No Rollback Plan

Before deploying, know:

```text
How do we go back?
```

---

## Mistake 6 — No Health Checks

A process being alive doesn't necessarily mean the application is ready.

---

## Mistake 7 — No Monitoring After Deployment

Deployment succeeded technically.

But users are receiving:

```text
500 errors
```

You need to monitor the actual system.

---

## Mistake 8 — Giving CI Too Much Access

CI credentials should have only the permissions they need.

---

## Mistake 9 — Secrets in Git

Never commit:

```text
.env
private keys
production passwords
cloud credentials
```

---

## Mistake 10 — Giant Pipelines

Don't create:

```text
50 unnecessary checks
```

that make every deployment take an hour.

Automate what provides meaningful safety.

---

# 44. A Practical CI/CD Pipeline for Our Todo API

Let's apply this to our project.

We currently have:

```text
Todo API
   │
   ├── PostgreSQL
   ├── Redis
   └── Docker
```

A realistic pipeline could be:

```text
Developer
    ↓
Push code
    ↓
Pull Request
    ↓
CI
    ├── npm ci
    ├── type check
    ├── lint
    ├── unit tests
    ├── integration tests
    └── build
    ↓
Merge
    ↓
Build Docker Image
    ↓
Push Image
    ↓
Deploy Staging
    ↓
Health Check
    ↓
Deploy Production
    ↓
Monitor
```

This is already a very useful production workflow.

---

# 45. Example GitHub Actions Concept

You may eventually use a CI platform such as GitHub Actions.

A simplified workflow might look like:

```yaml
name: CI

on:
  pull_request:
  push:
    branches:
      - main

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 22

      - run: npm ci

      - run: npm run typecheck

      - run: npm run lint

      - run: npm test

      - run: npm run build
```

Don't focus on memorizing the YAML.

Understand the flow:

```text
Checkout
   ↓
Install
   ↓
Validate
   ↓
Test
   ↓
Build
```

The exact CI configuration will depend on the project and platform.

---

# 46. Add Docker to the Pipeline

After tests pass:

```text
Tests
 ↓
Docker Build
 ↓
Image
 ↓
Registry
```

For example:

```bash
docker build -t todo-api:$GIT_SHA .
```

Then:

```text
todo-api:abc123
```

can represent the exact commit that produced it.

This makes debugging much easier.

---

# 47. Deployment Flow With Docker

Conceptually:

```text
Git Commit
    ↓
CI
    ↓
Tests
    ↓
Docker Build
    ↓
todo-api:abc123
    ↓
Container Registry
    ↓
Production
    ↓
Pull abc123
    ↓
Start container
    ↓
Health check
    ↓
Traffic
```

Now Docker and CI/CD work together.

---

# 48. What If the New Container Crashes?

Imagine:

```text
New container
     ↓
Starts
     ↓
Crashes
```

A good platform should detect this.

Possible actions:

```text
Don't route traffic
Restart container
Mark deployment unhealthy
Stop rollout
Rollback
Alert engineers
```

This is why health checks and observability matter.

---

# 49. Deployment and Background Workers

Remember Chapter 14.

Our system may have:

```text
API
Worker
Redis
Queue
```

Deploying the API doesn't necessarily mean deploying the worker at exactly the same time.

You need to think about:

```text
API version
Worker version
Job format
Backward compatibility
```

For example, if V2 of the API puts a new job format onto the queue, an old worker may not understand it.

Again:

> Distributed systems require compatibility thinking.

---

# 50. Deployment and WebSockets

Remember Chapter 15.

If you have WebSocket connections:

```text
Client
  ↓
API Server
```

and you deploy that server, active connections may be disconnected.

Your architecture might need:

```text
reconnection
connection draining
load balancing
shared pub/sub
```

Deployment isn't just:

> Start new container.

You need to consider the behavior of long-lived connections.

---

# 51. Deployment and File Uploads

Remember Chapter 16.

If users upload files to the local container filesystem:

```text
Upload
 ↓
Container disk
```

and the container is replaced:

```text
Old container
   ↓
Deleted
```

the files may disappear.

Production systems should generally use persistent object storage for user-generated files.

This is another example of why containers should be treated as disposable.

---

# 52. Deployment and Databases

From Chapter 21:

Your production architecture might be:

```text
              API
               │
        ┌──────┴──────┐
        ↓             ↓
      Redis       PostgreSQL
                     │
                ┌────┴────┐
                ↓         ↓
             Primary    Replica
```

Your CI/CD pipeline should not casually recreate your production database.

Application containers can be disposable.

Production database data is not.

This distinction is critical.

---

# 53. Deployment Architecture

Putting everything together:

```text
                         Developer
                             │
                             ↓
                          Git Push
                             │
                             ↓
                            CI
                   ┌─────────┼─────────┐
                   ↓         ↓         ↓
                 Tests      Build     Security
                   │         │
                   └────┬────┘
                        ↓
                   Docker Image
                        │
                        ↓
                Container Registry
                        │
                  ┌─────┴─────┐
                  ↓           ↓
               Staging     Production
                  │           │
                  ↓           ↓
              Health       Health
               Check        Check
                              │
                              ↓
                           Traffic
```

This is a production-minded backend delivery pipeline.

---

# 54. Interview Questions

## Q1. What is CI?

Continuous Integration is the practice of frequently integrating code changes and automatically validating them through checks such as tests, linting, type checking, and builds.

---

## Q2. What is CD?

CD can mean Continuous Delivery or Continuous Deployment.

Continuous Delivery keeps software ready for release.

Continuous Deployment automatically deploys validated changes.

---

## Q3. What is the difference between Continuous Delivery and Continuous Deployment?

```text
Continuous Delivery
→ Ready to deploy

Continuous Deployment
→ Automatically deploy
```

The main difference is whether production deployment requires a manual approval step.

---

## Q4. Why do we run tests in CI?

To catch problems automatically before code is merged or deployed.

---

## Q5. Why should we build the Docker image in CI?

To produce a consistent artifact that can be tested and then deployed.

Ideally, the same artifact moves through environments.

---

## Q6. Why are immutable artifacts useful?

They make deployments reproducible and make it easier to identify exactly what version is running.

---

## Q7. What is a rolling deployment?

A deployment strategy where instances are gradually replaced with a new version instead of replacing all instances at once.

---

## Q8. What is blue-green deployment?

A deployment strategy where two environments/versions exist and traffic is switched from the old version to the new version after validation.

---

## Q9. What is canary deployment?

A deployment strategy where a small percentage of traffic is initially sent to the new version while monitoring its behavior before increasing traffic.

---

## Q10. What is a rollback?

Returning production to a previous known-good application version after detecting a problem.

---

## Q11. Why are database migrations dangerous during deployment?

Because application code and database schema must remain compatible during the transition.

A deployment can fail if new code expects schema changes that haven't happened yet.

---

## Q12. What is graceful shutdown?

Gracefully stopping an application by preventing new work, finishing active work where possible, and closing resources cleanly before exiting.

---

## Q13. What is the difference between liveness and readiness?

Liveness asks whether the process is alive.

Readiness asks whether the instance is ready to receive traffic.

---

## Q14. Why shouldn't we use `latest` as our only Docker image reference?

Because it doesn't clearly identify the exact artifact being deployed.

A version or immutable identifier such as a commit SHA provides better traceability.

---

## Q15. Why should CI/CD credentials follow least privilege?

Because if a CI credential is compromised, limiting its permissions reduces the potential damage.

---

# 55. Scenario-Based Interview Questions

## Scenario 1

> "The deployment succeeded, but users are getting 500 errors."

What would you do?

I'd check:

```text
Deployment logs
Application logs
Error rate
Recent code changes
Environment variables
Database migrations
External dependencies
Health checks
```

Then compare:

```text
Before deployment
vs
After deployment
```

I wouldn't immediately roll back without understanding the failure unless the incident required immediate mitigation.

If the new version is clearly causing widespread impact, rollback may be appropriate while investigating.

---

## Scenario 2

> "How would you deploy a new version without downtime?"

I'd use an approach such as:

```text
Rolling deployment
Blue-green deployment
Canary deployment
```

depending on the infrastructure.

I'd make sure the new version passes health checks before routing production traffic to it.

I'd also consider graceful shutdown for the old instances.

---

## Scenario 3

> "The new application version requires a new database column."

I'd avoid a deployment where the new code immediately requires a schema that doesn't exist.

I'd use a backward-compatible migration strategy such as:

```text
Add column
   ↓
Deploy compatible code
   ↓
Backfill
   ↓
Start using column
   ↓
Remove old structure later
```

---

## Scenario 4

> "The application is running but the health check fails."

I'd investigate:

```text
Application startup
Environment variables
Database connectivity
Redis connectivity
Health endpoint
Network configuration
Logs
```

I wouldn't route normal traffic to an instance that isn't ready.

---

## Scenario 5

> "The production deployment broke the application. How do you recover?"

First, stabilize the system.

If the new version is clearly responsible:

```text
Stop rollout
   ↓
Rollback to known-good version
```

Then investigate:

```text
logs
metrics
traces
deployment diff
database changes
```

After the incident, add tests or safeguards to prevent the same class of failure.

---

# 56. Mini Project — Add CI/CD to the Todo API

Let's make our Todo API more production-like.

Current system:

```text
Todo API
   │
   ├── PostgreSQL
   ├── Redis
   └── Docker
```

Now build this pipeline:

```text
Pull Request
      ↓
CI
      ↓
Type Check
      ↓
Lint
      ↓
Tests
      ↓
Build
```

After merging:

```text
main
 ↓
Build Docker Image
 ↓
Tag with commit SHA
 ↓
Push to Registry
 ↓
Deploy Staging
 ↓
Health Check
```

Then:

```text
Staging passes
      ↓
Production deployment
      ↓
Health Check
      ↓
Monitor
```

---

# 57. Add a Health Endpoint

Create:

```text
GET /health
```

Response:

```json
{
  "status": "ok"
}
```

Then improve it later.

For example, you might check:

```text
Application
Database
Redis
```

But be careful.

A health endpoint that checks every dependency can itself become complicated.

Think about what the platform actually needs to know.

---

# 58. Add Graceful Shutdown

Your Node.js application should handle termination signals.

Conceptually:

```text
SIGTERM
   ↓
Stop accepting traffic
   ↓
Finish active requests
   ↓
Close database
   ↓
Close Redis
   ↓
Stop worker
   ↓
Exit
```

Test it.

Don't just write it and assume it works.

---

# 59. Simulate a Bad Deployment

This is one of the best exercises.

Intentionally introduce:

```text
bad environment variable
```

or:

```text
broken database migration
```

or:

```text
application startup failure
```

Then see:

```text
What does CI do?
What does deployment do?
Does health check fail?
Does traffic move?
Can you rollback?
Can you identify the problem from logs?
```

This is how you start understanding production engineering.

---

# 60. Make Your Deployment Boring

This sounds strange.

But good deployments should become boring.

You don't want:

```text
Friday 11 PM

"Okay everyone, I'm going to SSH into production."

"Wait, which server?"

"Did we pull main?"

"Did anyone run the migration?"

"Why is Redis down?"

"Who changed this environment variable?"
```

You want:

```text
Merge
 ↓
CI
 ↓
Build
 ↓
Deploy
 ↓
Health check
 ↓
Monitor
```

Automation removes unnecessary human steps.

---

# 61. The Bigger Picture

Look at the journey now.

We started with:

```text
HTTP
```

Then:

```text
APIs
 ↓
Databases
 ↓
Authentication
 ↓
Caching
 ↓
Queues
 ↓
WebSockets
 ↓
File Storage
 ↓
Logging
 ↓
Testing
 ↓
Configuration
 ↓
Docker
 ↓
Database Scaling
 ↓
Microservices
 ↓
CI/CD
```

We're no longer just learning:

> "How to write an Express route."

We're learning:

> **How a backend system is built, tested, packaged, deployed, and operated.**

That's the bigger picture.

---

# 62. Final Mental Model

Keep this in your head:

```text
                 Developer
                     │
                     ↓
                   Git
                     │
                     ↓
                    CI
             ┌───────┼───────┐
             ↓       ↓       ↓
           Tests    Build   Checks
             │       │
             └───┬───┘
                 ↓
             Docker Image
                 │
                 ↓
          Container Registry
                 │
          ┌──────┴──────┐
          ↓             ↓
       Staging       Production
          │             │
          ↓             ↓
      Health Check  Health Check
                        │
                        ↓
                     Traffic
                        │
                        ↓
                  Monitoring
```

And when something goes wrong:

```text
Production
    ↓
Monitoring detects problem
    ↓
Stop rollout / rollback
    ↓
Known-good version
    ↓
Investigate
    ↓
Fix
    ↓
Test
    ↓
Deploy again
```

That's the mindset.

---

# 63. What You Should Know After This Chapter

You should be comfortable explaining:

```text
CI
Continuous Delivery
Continuous Deployment
CI pipelines
Pull request checks
Build artifacts
Docker images in CI/CD
Container registries
Staging
Production
Database migrations
Expand-and-contract migrations
Rolling deployments
Blue-green deployments
Canary deployments
Health checks
Liveness
Readiness
Graceful shutdown
Rollbacks
Feature flags
Secrets in CI/CD
Immutable deployments
Deployment monitoring
```

But more importantly, you should be able to answer:

> **"How would you safely deploy a new backend version to production?"**

A solid answer should sound something like:

> "I'd first run automated tests, type checks, linting and a production build in CI. Then I'd build a versioned Docker image and deploy that same artifact to staging. After health checks and verification pass, I'd roll it out to production using something like a rolling or canary deployment. I'd monitor errors and latency during the rollout, and I'd have a rollback plan if the new version causes problems. For database changes, I'd make migrations backward-compatible so old and new application versions can coexist during deployment."

That's a much stronger answer than:

> "We push to GitHub and deploy."

---

# What's Next?

We've now covered how to:

```text
Build
Test
Containerize
Scale
Split services
Deploy
```

But we're still missing something important.

When traffic comes into a production system, **who receives it first?**

How does the system decide which server should handle a request?

What happens when one server dies?

Why do companies put:

```text
CDN
Load Balancer
Reverse Proxy
API Gateway
```

in front of their backend?

And what's actually happening here?

```text
Client
  ↓
DNS
  ↓
Load Balancer
  ↓
Reverse Proxy
  ↓
API Servers
  ↓
Cache
  ↓
Database
```

That's where we start putting all our backend concepts together into an actual production architecture.

# Chapter 24 — System Design Basics