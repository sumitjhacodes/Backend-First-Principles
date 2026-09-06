# Chapter 20 — Docker & Containerization

If you have built backend applications for some time, you have probably seen this problem:

> "It works on my machine."

It works on your laptop.

You push the code.

Another developer clones it.

They install dependencies.

And suddenly:

```text
Node version is different.
PostgreSQL version is different.
Redis isn't installed.
Environment variables are missing.
Some package behaves differently.
```

Then someone says:

> "Can you send me your setup?"

This is one of the problems Docker tries to solve.

Docker gives us a consistent way to package an application together with the environment it needs to run.

But before learning commands like:

```bash
docker build
docker run
docker compose up
```

we need to understand what Docker actually is.

---

# 1. What Problem Does Docker Solve?

Imagine your backend application needs:

```text
Node.js 22
PostgreSQL 17
Redis
Environment variables
Specific npm dependencies
Specific OS libraries
```

Your laptop has all of this configured.

But production is a different machine.

A teammate has another machine.

Your CI server has another environment.

Without some kind of standardization, you can get:

```text
Developer machine
        ↓
"It works here"

CI server
        ↓
"Tests fail"

Production
        ↓
"Application crashes"
```

Docker changes the approach.

Instead of saying:

> "Please install these things and configure them exactly like I did."

we can package the application into a **container image**.

Then that image can be run in different environments.

```text
                 Docker Image
                      │
          ┌───────────┼───────────┐
          ↓           ↓           ↓
      Developer      CI       Production
       Container   Container   Container
```

The goal is consistency.

---

# 2. What Is Docker?

Docker is a platform for building, packaging, and running applications in isolated environments called **containers**.

A simple mental model:

```text
Application
    +
Dependencies
    +
Runtime
    +
Required system files
        ↓
    Docker Image
        ↓
     Container
        ↓
    Running App
```

For a Node.js backend, the container might contain:

```text
Node.js
npm dependencies
application code
configuration needed to start
system libraries
```

Then Docker runs that image as a container.

---

# 3. Container vs Virtual Machine

You may have heard of virtual machines before.

A virtual machine basically creates another computer inside your computer.

Conceptually:

```text
Physical Machine
│
├── Host OS
│
└── Hypervisor
      │
      ├── VM 1
      │     └── Guest OS
      │          └── Application
      │
      └── VM 2
            └── Guest OS
                 └── Application
```

Containers work differently.

```text
Physical Machine
│
├── Host OS
│
└── Container Runtime
      │
      ├── Container 1
      │     └── Application
      │
      ├── Container 2
      │     └── Application
      │
      └── Container 3
            └── Application
```

Containers share the host operating system's kernel while providing process and filesystem isolation.

That's one reason containers are generally lighter than full virtual machines.

### Important

Don't think:

> Container = tiny virtual machine

That's not quite correct.

A container is better understood as:

> An isolated process environment packaged with the files and dependencies the application needs.

---

# 4. Why Containers Are Useful for Backend Developers

Suppose you're building an API.

Your application requires:

```text
Node.js
PostgreSQL
Redis
```

Without Docker:

```text
Your Laptop
│
├── Node.js
├── PostgreSQL
└── Redis
```

Now another developer needs to install all of them.

With Docker:

```text
Docker
│
├── API container
├── PostgreSQL container
└── Redis container
```

Now the project can describe its infrastructure.

This becomes especially useful when a project has multiple services.

---

# 5. The Four Docker Concepts You Must Understand

Before going deeper, remember these four terms:

```text
Dockerfile
    ↓
Image
    ↓
Container

Docker Compose
    ↓
Multiple containers/services
```

Let's understand each one.

---

# 6. What Is a Dockerfile?

A **Dockerfile** is a text file containing instructions for building a Docker image.

For example:

```dockerfile
FROM node:22

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

This file basically says:

> Start with Node.js, create an application directory, install dependencies, copy the application, and start it.

Think of a Dockerfile as a **recipe**.

```text
Dockerfile
    ↓
Recipe
    ↓
Docker Image
```

---

# 7. Understanding a Dockerfile Line by Line

Let's break it down.

## FROM

```dockerfile
FROM node:22
```

This defines the base image.

We're saying:

> Start this image with Node.js 22.

You can think of it as the foundation.

```text
node:22
   ↓
Base
   ↓
Your application
```

---

# 8. WORKDIR

```dockerfile
WORKDIR /app
```

This sets the working directory inside the container.

Instead of doing everything from some random directory, our application will live in:

```text
/app
```

For example:

```text
/app
├── package.json
├── package-lock.json
├── src
└── ...
```

---

# 9. COPY

```dockerfile
COPY package*.json ./
```

This copies package files from your machine into the container.

For example:

```text
Your computer

package.json
package-lock.json

        ↓

Container

/app/package.json
/app/package-lock.json
```

Then:

```dockerfile
RUN npm ci
```

installs the dependencies.

After that:

```dockerfile
COPY . .
```

copies the rest of the application.

---

# 10. Why Copy package.json First?

You might wonder why we don't simply do:

```dockerfile
COPY . .
RUN npm install
```

We could.

But Docker images are built in layers, and Docker can reuse cached layers when earlier instructions haven't changed.

For example:

```dockerfile
COPY package*.json ./
RUN npm ci

COPY . .
```

If you only change:

```text
src/users.service.ts
```

your dependencies haven't changed.

Docker can reuse the dependency-installation layer.

This can make builds much faster.

This is an important practical Docker concept:

> Structure Dockerfiles so expensive steps can be cached.

---

# 11. RUN vs CMD

This confuses beginners a lot.

### RUN

```dockerfile
RUN npm ci
```

happens while **building the image**.

### CMD

```dockerfile
CMD ["npm", "start"]
```

defines the default command when the **container starts**.

Think:

```text
docker build
     ↓
RUN commands
     ↓
Image created
```

Then:

```text
docker run
     ↓
CMD executes
     ↓
Application starts
```

---

# 12. What Is a Docker Image?

An image is a packaged, read-only template used to create containers.

For example:

```text
backend-image
│
├── Node.js
├── dependencies
├── application code
└── startup configuration
```

You build an image:

```bash
docker build -t backend-app .
```

Now you have:

```text
backend-app
```

That image can be used to create containers.

---

# 13. What Is a Container?

A container is a running instance of an image.

Think:

```text
Image
  │
  ├── Container A
  ├── Container B
  └── Container C
```

One image can create multiple containers.

For example:

```bash
docker run backend-app
```

creates and starts a container from:

```text
backend-app
```

So:

> Image = blueprint
> Container = running instance

This is one of the most important Docker concepts.

---

# 14. Build vs Run

Another common confusion:

```bash
docker build
```

and

```bash
docker run
```

do different things.

### Build

Creates an image.

```text
Dockerfile
    ↓
docker build
    ↓
Image
```

### Run

Creates a container from an image.

```text
Image
   ↓
docker run
   ↓
Container
   ↓
Application
```

---

# 15. Running a Simple Backend Container

Suppose your application listens on:

```text
3000
```

You build the image:

```bash
docker build -t backend-app .
```

Then run:

```bash
docker run -p 3000:3000 backend-app
```

What does this mean?

```text
-p 3000:3000
```

means:

```text
Host port : Container port
```

So:

```text
Your computer
localhost:3000
      │
      ↓
Container:3000
      │
      ↓
Node.js application
```

---

# 16. Why Do We Need Port Mapping?

Containers have their own network namespace.

Your application may listen on:

```text
container:3000
```

But that doesn't automatically mean your laptop can access it through:

```text
localhost:3000
```

Port mapping connects them.

```bash
docker run -p 3000:3000 backend-app
```

means:

```text
Host
3000
 │
 │ port mapping
 ↓
Container
3000
```

You can also map different ports:

```bash
docker run -p 8080:3000 backend-app
```

Now:

```text
localhost:8080
      ↓
container:3000
```

---

# 17. What Does EXPOSE Actually Do?

You might see:

```dockerfile
EXPOSE 3000
```

Beginners sometimes think this publishes the port.

It doesn't.

`EXPOSE` documents the port the application expects to use.

You still need:

```bash
-p 3000:3000
```

to publish it from the container to the host.

So:

```text
EXPOSE
    ↓
Documentation / metadata

-p
    ↓
Actual host-to-container port publishing
```

---

# 18. Environment Variables in Docker

Our application probably has configuration such as:

```text
PORT
DATABASE_URL
REDIS_URL
JWT_SECRET
```

We shouldn't hardcode these inside the image.

Instead, provide configuration when the container runs.

For example:

```bash
docker run \
  -e PORT=3000 \
  -e DATABASE_URL="..." \
  backend-app
```

The application can access them through:

```typescript
process.env.PORT
```

This connects directly with Chapter 19.

Remember:

> Configuration belongs outside the application image.

---

# 19. Never Bake Secrets Into Images

Don't do this:

```dockerfile
ENV JWT_SECRET=my-super-secret-value
```

Especially not for real production secrets.

Why?

Because images can be:

```text
stored
shared
pushed to registries
cached
inspected
copied
```

Instead, inject secrets/configuration through your deployment environment or a proper secret-management system.

Docker doesn't magically make secrets secure.

---

# 20. What Is a Volume?

Containers are designed to be replaceable.

That creates a problem.

Suppose PostgreSQL stores data inside a container.

You remove the container.

The data may disappear with it.

We need persistent storage.

That's where **volumes** come in.

```text
Container
   │
   │ database writes
   ↓
Docker Volume
   │
   ↓
Persistent storage
```

The container can be deleted and recreated while the volume remains.

---

# 21. Example: PostgreSQL Volume

With Docker Compose, you might have:

```yaml
services:
  postgres:
    image: postgres:17
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

The important idea is:

```text
PostgreSQL Container
        │
        ↓
postgres_data volume
        │
        ↓
Persistent database files
```

This is very important for databases.

---

# 22. Bind Mount vs Volume

You will encounter both.

### Bind mount

Maps a specific host directory:

```text
./src
   ↓
/app/src
```

Useful during local development.

### Named volume

Managed by Docker:

```text
postgres_data
      ↓
PostgreSQL data
```

Useful for persistent application data.

A simple mental model:

```text
Bind Mount
→ "Use this folder from my machine."

Volume
→ "Docker, manage this persistent storage for me."
```

---

# 23. Why Containers Should Usually Be Disposable

A good container mindset is:

> Don't depend on manually modifying a running container.

For example, don't build a container and then SSH into it and manually install packages.

Instead:

```text
Change Dockerfile
      ↓
Build new image
      ↓
Replace container
```

This gives you repeatability.

The container should be something you can destroy and recreate.

---

# 24. Docker Networks

Now imagine we have:

```text
API
PostgreSQL
Redis
```

They are three separate containers.

How does the API talk to PostgreSQL?

Through a Docker network.

Conceptually:

```text
Docker Network
│
├── api
├── postgres
└── redis
```

Containers connected to the same Docker network can communicate with each other.

---

# 25. The localhost Problem

This is one of the most common Docker mistakes.

Suppose your Node.js API is running inside a container.

You write:

```text
DATABASE_URL=postgresql://localhost:5432/app
```

But PostgreSQL is running in another container.

This usually won't work.

Why?

Inside the API container:

```text
localhost
```

means:

> This API container itself.

Not the PostgreSQL container.

You might instead use the PostgreSQL service/container name.

For example:

```text
postgres:5432
```

Conceptually:

```text
API container
    │
    │ postgres:5432
    ↓
PostgreSQL container
```

Docker's internal DNS allows services to discover each other by service name on the same network.

This is a very important thing to understand.

---

# 26. Docker Compose

Now we have a problem.

Running everything manually could become annoying:

```bash
docker run ...
docker run ...
docker run ...
```

What if your project has:

```text
API
PostgreSQL
Redis
Worker
```

This is where **Docker Compose** becomes useful.

Docker Compose lets you define multiple services in one configuration file.

For example:

```yaml
services:

  api:
    build: .
    ports:
      - "3000:3000"
    environment:
      DATABASE_URL: postgresql://postgres:postgres@postgres:5432/app
      REDIS_URL: redis://redis:6379
    depends_on:
      - postgres
      - redis

  postgres:
    image: postgres:17
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: app
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7

volumes:
  postgres_data:
```

Now the project describes its infrastructure in one place.

---

# 27. What Does `docker compose up` Actually Do?

This is a great interview question.

Suppose you run:

```bash
docker compose up
```

At a high level Docker Compose:

```text
Read compose.yaml
       ↓
Understand services
       ↓
Build required images
       ↓
Pull required images
       ↓
Create network
       ↓
Create containers
       ↓
Attach volumes
       ↓
Configure environment
       ↓
Start containers
       ↓
Show container logs
```

For our example:

```text
             Compose
                │
       ┌────────┼────────┐
       ↓        ↓        ↓
      API    PostgreSQL  Redis
       │        │        │
       └────────┼────────┘
                ↓
           Docker Network
```

That's the mental model you should remember.

---

# 28. Service Names Become Important

In Compose:

```yaml
services:

  api:
    ...

  postgres:
    ...

  redis:
    ...
```

the service names become useful for internal communication.

For example:

```text
postgres:5432
```

and:

```text
redis:6379
```

The API doesn't need to know some random container IP address.

It can use the service name.

This is much more stable.

---

# 29. `depends_on` Does Not Mean "Ready"

You may write:

```yaml
depends_on:
  - postgres
```

This mainly expresses startup dependency/order.

It does **not automatically mean**:

> PostgreSQL is fully ready to accept connections.

For example:

```text
Postgres container starts
        ↓
Postgres process still initializing
        ↓
API starts
        ↓
API tries database connection
        ↓
Connection fails
```

This is why real applications often need:

* health checks
* retry logic
* application startup checks
* proper readiness handling

This connects directly with Chapter 17.

---

# 30. Health Checks

Docker can check whether a service is healthy.

Conceptually:

```text
Container
    ↓
Health check
    ↓
"Can this service actually respond?"
```

For example, a backend might expose:

```text
/health
```

A health check could call that endpoint.

For databases, a check might verify that the database accepts connections.

This is different from:

> Is the container process running?

A process can be running while the application is still unhealthy.

---

# 31. `.dockerignore`

Just like `.gitignore`, Docker has:

```text
.dockerignore
```

It tells Docker what should not be included in the build context.

For a Node.js project:

```text
node_modules
.git
.env
npm-debug.log
Dockerfile*
README.md
```

Be careful with `.env`.

If it contains secrets, you generally don't want to send it into the image build context.

---

# 32. Why Huge Docker Images Are a Problem

Imagine your image is:

```text
2.5 GB
```

You deploy it.

The server needs to download it.

CI needs to build it.

A registry needs to store it.

Every deployment becomes slower.

So we generally want reasonably small images.

Some common approaches:

* use an appropriate base image
* avoid unnecessary packages
* use `.dockerignore`
* use multi-stage builds
* install only production dependencies in production
* clean up unnecessary files

Don't optimize blindly though.

The goal isn't:

> Smallest possible image.

The goal is:

> A secure, reliable image containing exactly what the application needs.

---

# 33. Multi-Stage Builds

Multi-stage builds let you use different stages for building and running an application.

For example:

```dockerfile
FROM node:22 AS builder

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

RUN npm run build


FROM node:22-slim AS production

WORKDIR /app

COPY package*.json ./

RUN npm ci --omit=dev

COPY --from=builder /app/dist ./dist

CMD ["node", "dist/server.js"]
```

Conceptually:

```text
Builder stage
│
├── source code
├── dev dependencies
├── TypeScript
└── build tools
        │
        ↓
      build
        │
        ↓
Production stage
│
├── production dependencies
├── compiled application
└── runtime
```

The final image doesn't need all the build tools.

This can make production images cleaner and smaller.

---

# 34. Development vs Production Containers

Don't assume the Docker setup for development and production should be identical.

During development you might want:

```text
Hot reload
Source code mounts
Debugging
Dev dependencies
Verbose logs
```

Production may want:

```text
Compiled code
Production dependencies
No source mounts
Smaller image
Non-root user
Health checks
Controlled logging
```

For example:

```text
Development
    ↓
docker compose
    ↓
source mounted
    ↓
watch mode
```

Production:

```text
CI/CD
   ↓
build image
   ↓
push image
   ↓
deploy image
   ↓
container
```

---

# 35. Running as Root

Containers often run processes as root by default depending on the image.

That isn't ideal for production.

If the application is compromised, running as a non-root user can reduce the potential impact.

For example:

```dockerfile
USER node
```

when supported by the chosen base image.

Security is layered.

Running as non-root doesn't magically make your container secure, but it's a useful defense-in-depth practice.

---

# 36. Docker and Environment Configuration

Remember Chapter 19.

Docker doesn't replace configuration management.

You might have:

```text
Docker image
      +
Environment variables
      +
Secrets
      +
Runtime configuration
```

The same image can run in different environments.

```text
              Same Image
                  │
        ┌─────────┼─────────┐
        ↓         ↓         ↓
       Dev     Staging    Prod
        │         │         │
    Dev config  Stage     Prod
               config    config
```

This is exactly what we want.

---

# 37. Docker Registry

After building an image locally, you may want to store it somewhere.

A **container registry** stores container images.

Conceptually:

```text
Developer
    ↓
docker build
    ↓
Image
    ↓
docker push
    ↓
Container Registry
    ↓
Production server
    ↓
docker pull
    ↓
Container
```

Registries are used to distribute images.

Examples include public and private registries.

---

# 38. A Real Backend Architecture

Let's put everything together.

Imagine our backend has:

```text
Node.js API
PostgreSQL
Redis
Background Worker
```

We can model it like this:

```text
                         Internet
                            │
                            ↓
                     Reverse Proxy
                            │
                            ↓
                    ┌──────────────┐
                    │ API Container │
                    └──────┬───────┘
                           │
             ┌─────────────┼─────────────┐
             ↓             ↓             ↓
        PostgreSQL       Redis        Queue/Worker
        Container       Container       Container
             │             │
             ↓             ↓
          Volume        Cache/Data
```

Docker gives us a consistent way to package and run these services.

But Docker itself isn't the architecture.

Docker is the **packaging and runtime layer**.

---

# 39. Docker Does Not Solve Everything

This is important.

Docker does NOT automatically solve:

* authentication
* authorization
* database design
* scaling
* monitoring
* security
* CI/CD
* backups
* high availability
* load balancing

It helps with packaging and running applications consistently.

You still need to design the system correctly.

---

# 40. Common Docker Mistakes

Let's go through the mistakes I see beginners make most often.

## Mistake 1 — Using localhost Between Containers

Wrong:

```text
DATABASE_URL=postgres://localhost:5432/app
```

when PostgreSQL is another container.

Remember:

```text
localhost = current container
```

Use the service name on the Docker network.

---

## Mistake 2 — Putting Secrets in Dockerfiles

Don't do:

```dockerfile
ENV JWT_SECRET=secret123
```

for production secrets.

Inject secrets at runtime through proper configuration/secret management.

---

## Mistake 3 — Assuming Container Storage Is Permanent

Containers can be removed.

If data needs to survive:

```text
Use persistent storage.
```

For databases, use volumes or external managed storage.

---

## Mistake 4 — Installing Things Manually Inside Containers

Don't rely on:

```bash
docker exec -it container bash
```

and manually modifying the container as your normal workflow.

Update the Dockerfile and rebuild.

---

## Mistake 5 — Making One Giant Container

Don't put everything into one container:

```text
Node
Postgres
Redis
Nginx
Worker
...
```

Usually, services should have clear responsibilities.

For example:

```text
API → container
Worker → container
Postgres → container
Redis → container
```

Docker Compose can coordinate them locally.

---

## Mistake 6 — Thinking `depends_on` Means Ready

Startup order and service readiness aren't the same thing.

Your application should handle temporary dependency unavailability gracefully.

---

## Mistake 7 — Running Development Setup in Production

Things like:

```text
nodemon
source mounts
dev dependencies
debug tooling
```

usually don't belong in a production image.

---

## Mistake 8 — Ignoring Image Size

Huge images make builds and deployments slower.

Look at what you're actually putting into the image.

---

# 41. Useful Docker Commands

You don't need to memorize every Docker command.

Start with these.

### Check Docker

```bash
docker --version
```

### Build an image

```bash
docker build -t backend-app .
```

### List images

```bash
docker images
```

### Run a container

```bash
docker run backend-app
```

### Run with port mapping

```bash
docker run -p 3000:3000 backend-app
```

### List running containers

```bash
docker ps
```

### List all containers

```bash
docker ps -a
```

### Stop a container

```bash
docker stop <container>
```

### Remove a container

```bash
docker rm <container>
```

### Remove an image

```bash
docker rmi <image>
```

### View logs

```bash
docker logs <container>
```

### Execute a command inside a container

```bash
docker exec -it <container> sh
```

And for Compose:

```bash
docker compose up
```

```bash
docker compose up --build
```

```bash
docker compose down
```

```bash
docker compose ps
```

```bash
docker compose logs
```

```bash
docker compose logs -f api
```

---

# 42. `docker compose down` and Your Database

One thing to understand carefully.

When you run:

```bash
docker compose down
```

containers are removed.

But named volumes can remain unless you explicitly remove them.

For example:

```text
Postgres container
       ↓
postgres_data volume
```

Removing the container doesn't necessarily mean deleting the volume.

This is one reason volumes are useful.

Be careful with commands that explicitly remove volumes.

Deleting a database volume can mean deleting your local database data.

---

# 43. Mini Project — Containerize Our Todo API

Let's apply everything we've learned.

Suppose our project looks like:

```text
todo-api/
│
├── src/
│   ├── routes/
│   ├── controllers/
│   ├── services/
│   └── server.ts
│
├── package.json
├── package-lock.json
├── tsconfig.json
├── Dockerfile
├── compose.yaml
└── .dockerignore
```

Our architecture:

```text
             API
              │
       ┌──────┴──────┐
       ↓             ↓
  PostgreSQL       Redis
```

---

# 44. Step 1 — Create the Dockerfile

Start with:

```dockerfile
FROM node:22

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

Build:

```bash
docker build -t todo-api .
```

Run:

```bash
docker run -p 3000:3000 todo-api
```

Now the API should be accessible through the mapped port, assuming your application listens on the correct interface and port.

---

# 45. Important Node.js Docker Detail

This is another common beginner issue.

Your Node.js server should generally listen on:

```text
0.0.0.0
```

inside a container, not only:

```text
localhost
```

For example:

```typescript
app.listen(3000, "0.0.0.0");
```

Why?

If your application only listens on the container's loopback interface, traffic coming through the container's network interface may not reach it as expected.

Mental model:

```text
Correct:

Container network
       ↓
0.0.0.0:3000
       ↓
Node.js


Potential problem:

localhost:3000
       ↓
Only container loopback
```

---

# 46. Step 2 — Add PostgreSQL

Now create:

```text
compose.yaml
```

with:

```yaml
services:

  api:
    build: .
    ports:
      - "3000:3000"
    environment:
      DATABASE_URL: postgresql://postgres:postgres@postgres:5432/todos
    depends_on:
      - postgres

  postgres:
    image: postgres:17
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: todos
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

Now:

```bash
docker compose up --build
```

You have:

```text
API Container
     │
     │ postgres:5432
     ↓
PostgreSQL Container
     │
     ↓
postgres_data
```

---

# 47. Step 3 — Add Redis

Add:

```yaml
redis:
  image: redis:7
```

Then:

```yaml
environment:
  DATABASE_URL: postgresql://postgres:postgres@postgres:5432/todos
  REDIS_URL: redis://redis:6379
```

Now:

```text
             API
            /   \
           /     \
          ↓       ↓
    PostgreSQL    Redis
```

The important part isn't memorizing the YAML.

It's understanding:

```text
Service name
     ↓
Docker network
     ↓
Service-to-service communication
```

---

# 48. Step 4 — Add `.dockerignore`

Create:

```text
node_modules
.git
.env
npm-debug.log
dist
coverage
```

This prevents unnecessary files from being sent into the Docker build context.

---

# 49. Step 5 — Improve the Production Image

Once the basic setup works, improve it.

Think about:

```text
Multi-stage build
Production dependencies
Non-root user
Small base image
Health checks
Runtime configuration
Graceful shutdown
Logging
```

Don't start with optimization before understanding the basic container.

First:

```text
Make it work
```

Then:

```text
Make it reproducible
```

Then:

```text
Make it secure
```

Then:

```text
Make it efficient
```

---

# 50. What Happens During Deployment?

Let's imagine you're deploying your backend.

A simplified flow might look like:

```text
Developer pushes code
          ↓
CI pipeline starts
          ↓
Run tests
          ↓
Build Docker image
          ↓
Tag image
          ↓
Push image to registry
          ↓
Production server pulls image
          ↓
Start new container
          ↓
Health check
          ↓
Traffic goes to new version
```

This connects Docker with the CI/CD chapter that we'll cover later.

Docker is often the packaging unit moving through the deployment pipeline.

---

# 51. Docker and Scaling

Suppose one API container can't handle the traffic.

You could run multiple instances:

```text
             Load Balancer
            /      |       \
           ↓       ↓        ↓
        API 1    API 2    API 3
```

All three can be created from the same image.

```text
                 API Image
                     │
          ┌──────────┼──────────┐
          ↓          ↓          ↓
        API 1      API 2      API 3
```

This is one of the reasons immutable images are useful.

You don't manually configure every server differently.

You create one application image and run multiple instances.

Scaling itself is a larger topic, and we'll get into that in system design and production architecture.

---

# 52. Containerization vs Virtualization

Let's make the distinction simple.

### Virtual Machine

```text
Hardware
   ↓
Hypervisor
   ↓
Guest OS
   ↓
Application
```

### Container

```text
Host OS
   ↓
Container Runtime
   ↓
Container
   ↓
Application
```

Containers usually have lower overhead because they don't require a full guest operating system per application.

But remember:

> Containers still provide isolation; they are not simply "processes with no security boundary."

The exact isolation depends on the operating system and runtime.

---

# 53. Docker Mental Model

If you forget everything else from this chapter, remember this:

```text
                 Dockerfile
                     │
                     │ build
                     ↓
                  Image
                     │
                     │ run
                     ↓
                Container
                     │
           ┌─────────┼─────────┐
           ↓         ↓         ↓
        Network    Volume     Config
           │
           ↓
     Other Containers
```

And for multiple services:

```text
                    Compose
                       │
        ┌──────────────┼──────────────┐
        ↓              ↓              ↓
       API         PostgreSQL        Redis
        │              │              │
        └──────────────┼──────────────┘
                       ↓
                  Docker Network
```

That's the core mental model.

---

# 54. Interview Questions

## Q1. What is Docker?

Docker is a platform for packaging and running applications in isolated container environments.

It helps make application environments more consistent across development, CI, and production.

---

## Q2. What is a Docker image?

A Docker image is an immutable packaged template containing the application and the files/dependencies needed to run it.

Containers are created from images.

---

## Q3. What is a container?

A container is a running instance of a Docker image.

A simple way to remember it:

```text
Image = template
Container = running instance
```

---

## Q4. What is a Dockerfile?

A Dockerfile contains instructions used to build a Docker image.

For example:

```dockerfile
FROM node:22
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
CMD ["npm", "start"]
```

---

## Q5. What's the difference between `RUN` and `CMD`?

`RUN` executes commands during image construction.

`CMD` defines the default command executed when a container starts.

```text
RUN
↓
Build time

CMD
↓
Container runtime
```

---

## Q6. What's the difference between an image and a container?

An image is the packaged template.

A container is a running instance created from that image.

One image can be used to create multiple containers.

---

## Q7. Why do we use Docker volumes?

Volumes provide persistent storage outside the container's writable lifecycle.

They're especially important for stateful services such as databases.

---

## Q8. Why can't one container use `localhost` to access another container?

Because `localhost` refers to the current container's network namespace.

If PostgreSQL is running in another container, the API should communicate with that container through the Docker network, commonly using the service name.

---

## Q9. What is Docker Compose?

Docker Compose is a tool for defining and running multi-container applications using a Compose configuration file.

For example:

```text
API
PostgreSQL
Redis
Worker
```

can all be defined as services.

---

## Q10. What happens when you run `docker compose up`?

At a high level:

```text
Read configuration
      ↓
Build/pull images
      ↓
Create network
      ↓
Create containers
      ↓
Attach volumes/config
      ↓
Start services
      ↓
Show logs
```

The exact behavior depends on the Compose configuration and what's already present.

---

## Q11. Does `depends_on` guarantee that a database is ready?

No.

It can help control startup ordering, but service startup and service readiness are different things.

Applications should handle dependency readiness through health checks, retries, or startup checks as appropriate.

---

## Q12. Why are Docker images designed to be immutable?

Because treating images as immutable artifacts makes deployments more predictable.

Instead of modifying a running production container:

```text
Change code
    ↓
Build new image
    ↓
Deploy new container
```

This makes rollback and reproducibility easier.

---

## Q13. Why use multi-stage Docker builds?

Multi-stage builds allow build dependencies and tooling to stay in an earlier stage while the final image contains only what is required to run the application.

This can reduce image size and attack surface.

---

## Q14. Why should we use `.dockerignore`?

To prevent unnecessary or sensitive files from being sent into the Docker build context.

For example:

```text
node_modules
.git
.env
coverage
```

---

## Q15. How would you persist PostgreSQL data in Docker?

Use a persistent volume or external persistent storage.

For local Compose development:

```text
PostgreSQL container
       ↓
Named volume
       ↓
Persistent database data
```

For production, managed database services are often preferable depending on the architecture.

---

# 55. Scenario-Based Interview Questions

## Scenario 1

> "My Node.js API works locally but can't connect to PostgreSQL after moving both into Docker."

What would you check?

I'd check the database host first.

If PostgreSQL is another container, using:

```text
localhost
```

from the API container is probably wrong.

I'd use the PostgreSQL service name on the Docker network, for example:

```text
postgres:5432
```

Then I'd check:

```text
network
port
credentials
database name
environment variables
database readiness
```

---

## Scenario 2

> "Every time the PostgreSQL container is recreated, our local data disappears."

I'd check whether PostgreSQL is using persistent storage.

If the database is writing only to the container filesystem, recreating the container can lose the data.

I'd attach a persistent volume.

---

## Scenario 3

> "Our Docker image is 2 GB and deployments are slow."

I'd investigate:

```text
Base image
node_modules
Build artifacts
Unnecessary packages
Docker build context
Dev dependencies
Copied files
```

Then consider:

```text
.dockerignore
multi-stage builds
production-only dependencies
appropriate base image
```

But I wouldn't blindly optimize before understanding what is consuming the space.

---

## Scenario 4

> "The API container starts, but requests to port 3000 don't work."

I'd check:

```text
Does Node listen on 0.0.0.0?
Is the application actually running?
Is port 3000 correct?
Was the port published with -p?
Are firewall/network rules involved?
What do the container logs show?
```

For example:

```text
docker logs <container>
```

and:

```bash
docker ps
```

---

## Scenario 5

> "The API starts before PostgreSQL is ready and crashes."

I wouldn't assume:

```yaml
depends_on:
```

solves it.

I'd consider:

```text
health checks
startup retries
database connection retry logic
readiness handling
```

The key idea is:

> Container started != service ready.

---

# 56. Mini Project Challenge

Now containerize your Todo API properly.

Your goal:

```text
Todo API
   │
   ├── Node.js
   ├── PostgreSQL
   └── Redis
```

### Requirements

Create:

```text
Dockerfile
compose.yaml
.dockerignore
```

Your Compose setup should provide:

```text
API container
PostgreSQL container
Redis container
PostgreSQL persistent volume
Docker network
Environment configuration
```

The API should be able to communicate with:

```text
postgres:5432
redis:6379
```

without using `localhost` for those services.

---

# 57. Make the Project Production-Minded

After getting the basic setup working, improve it.

Try to add:

* multi-stage Docker build
* production dependencies only
* non-root user
* health endpoint
* database readiness handling
* graceful shutdown
* structured logs
* environment validation
* Docker image tagging
* CI build step

You don't need all of this on day one.

The point is to gradually understand how a development container becomes a production-ready artifact.

---

# 58. Things I Want You to Actually Understand

Don't finish this chapter thinking:

> "I know 20 Docker commands."

That's not the goal.

You should be able to explain:

```text
What is a container?
What is an image?
What is a Dockerfile?
Why do we need port mapping?
Why do we need volumes?
How do containers communicate?
Why doesn't localhost work between containers?
What does Compose solve?
What happens during docker compose up?
Why do we use multi-stage builds?
Why shouldn't secrets be baked into images?
Why should containers be disposable?
```

If you can explain those without memorizing a definition, you're doing fine.

---

# 59. The Bigger Backend Picture

Look at what we've learned so far.

We started with:

```text
HTTP
 ↓
Web Servers
 ↓
Routing
 ↓
APIs
 ↓
Databases
 ↓
Authentication
 ↓
Caching
 ↓
Background Jobs
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
```

Now we're moving from:

> "How do I build a backend?"

towards:

> "How do I run and operate a backend reliably?"

That's an important transition.

Docker is one of the tools that helps us make that transition.

---

# 60. Final Mental Model

Here's the model I'd keep in your head:

```text
                    Dockerfile
                        │
                     build
                        ↓
                  Docker Image
                        │
                      run
                        ↓
                   Container
                        │
          ┌─────────────┼─────────────┐
          ↓             ↓             ↓
       Network        Volume        Config
          │
          ↓
   Other Containers
```

And with Compose:

```text
                  compose.yaml
                       │
                       ↓
                 Docker Compose
                       │
       ┌───────────────┼───────────────┐
       ↓               ↓               ↓
      API          PostgreSQL         Redis
   container        container        container
       │               │               │
       └───────────────┼───────────────┘
                       ↓
                  Docker Network
                       │
                       ↓
                 Persistent Data
```

The key idea:

> **Docker packages your application. Containers run it. Images are the reusable artifacts. Volumes persist data. Networks connect services. Compose coordinates multiple containers.**

Once this mental model is clear, Docker commands become much easier to learn because you're no longer memorizing random commands.

---

# What's Next?

We've now covered how to package and run our backend.

But there is a bigger problem.

What happens when our database has:

```text
10,000 users?
1 million users?
100 million rows?
Thousands of queries per second?
```

At that point, simply putting PostgreSQL inside a container isn't enough.

We need to understand:

* Connection pooling
* Read replicas
* Database replication
* Primary vs replica
* Read/write splitting
* Database bottlenecks
* Indexing at scale
* Partitioning
* Sharding
* Horizontal scaling
* When databases become the bottleneck

That brings us to:

# Chapter 21 — Databases at Scale