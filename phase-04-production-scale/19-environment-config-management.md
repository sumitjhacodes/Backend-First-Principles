# Chapter 19 — Environment & Configuration Management

You've probably written something like this before:

```js
const PORT = 3000;
```

Or:

```js
const DATABASE_URL = "postgresql://localhost:5432/myapp";
```

Or maybe:

```js
const JWT_SECRET = "my-secret-key";
```

It works.

Your application starts.

You feel good.

Then you deploy.

And suddenly:

```text
Local database ≠ Production database
Local API key ≠ Production API key
Local port ≠ Production port
Development settings ≠ Production settings
```

Now you start changing code just to make the application run in a different environment.

That's a problem.

Your application code shouldn't need to know whether it's running on:

```text
Your laptop
Testing
Staging
Production
```

The environment should provide the configuration.

That's what this chapter is about.

---

# 1. What Is Configuration?

Configuration is information that tells your application:

> **"How should I run?"**

Examples:

```text
PORT=3000
DATABASE_URL=...
REDIS_URL=...
JWT_SECRET=...
NODE_ENV=development
```

These aren't really business logic.

They're settings.

Your code uses them.

For example:

```js
const port = process.env.PORT || 3000;
```

The application doesn't care where the value came from.

It just needs a port.

---

# 2. Why Configuration Should Be Separate From Code

Imagine this:

```js
const databaseUrl =
  "postgresql://user:password@production-db:5432/app";
```

You commit it to Git.

Now your source code contains:

```text
Production database credentials
```

That's dangerous.

Instead:

```text
Environment
   ↓
DATABASE_URL
   ↓
Application
```

The application reads the configuration at runtime.

---

# 3. Different Environments

Most real applications have multiple environments.

At minimum:

```text
development
test
production
```

You may also have:

```text
staging
preview
```

For example:

```text
Development
    ↓
your laptop

Test
    ↓
automated tests

Staging
    ↓
production-like environment

Production
    ↓
real users
```

Each environment may need different configuration.

---

# 4. Example

Development:

```text
DATABASE_URL=postgresql://localhost:5432/myapp_dev
```

Test:

```text
DATABASE_URL=postgresql://localhost:5432/myapp_test
```

Production:

```text
DATABASE_URL=postgresql://production-server/myapp
```

Same application code.

Different configuration.

That's exactly what we want.

---

# 5. Environment Variables

An environment variable is simply a value provided to a process by its environment.

In Node.js:

```js
process.env.PORT
```

reads an environment variable called:

```text
PORT
```

For example:

```text
PORT=3000
```

Then:

```js
console.log(process.env.PORT);
```

might output:

```text
3000
```

---

# 6. Important: Environment Variables Are Strings

This catches beginners often.

Suppose:

```text
PORT=3000
```

Then:

```js
process.env.PORT
```

is a string.

So:

```js
typeof process.env.PORT
```

is:

```text
"string"
```

Not:

```text
number
```

If you need a number:

```js
const port = Number(process.env.PORT);
```

But you should also validate it.

---

# 7. The `.env` File

During development, you might use:

```text
.env
```

Example:

```env
PORT=3000
DATABASE_URL=postgresql://localhost:5432/myapp
JWT_SECRET=development-secret
REDIS_URL=redis://localhost:6379
```

A library such as `dotenv` can load these values into `process.env`.

Then your application can access them:

```js
process.env.DATABASE_URL
```

---

# 8. Never Commit Secrets

This is extremely important.

Don't commit:

```text
.env
```

if it contains real secrets.

Add it to:

```text
.gitignore
```

For example:

```gitignore
.env
.env.local
.env.production
```

The exact files depend on your project.

---

# 9. Why This Matters

Imagine accidentally committing:

```env
DATABASE_PASSWORD=super-secret-password
JWT_SECRET=super-secret
AWS_SECRET_KEY=...
```

to a public GitHub repository.

Now those credentials may be exposed.

Even if you delete the file afterward, the secret may still exist in Git history.

So:

> **Never treat Git as a secret store.**

---

# 10. What Should Go Into Environment Variables?

Good candidates:

```text
Database URLs
API keys
Secrets
Tokens
Passwords
Service URLs
Ports
Feature flags
Environment names
Configuration values
```

For example:

```env
DATABASE_URL=...
REDIS_URL=...
STRIPE_SECRET_KEY=...
JWT_SECRET=...
PORT=3000
NODE_ENV=development
```

---

# 11. What Should NOT Go Into Environment Variables?

Don't turn your entire application into:

```text
ENV_VAR_1
ENV_VAR_2
ENV_VAR_3
ENV_VAR_4
ENV_VAR_5
...
```

Not every value needs to be configurable.

For example:

```js
const MAX_NAME_LENGTH = 100;
```

may simply be application logic.

Configuration should represent values that genuinely differ by environment or deployment.

---

# 12. Environment Variables Aren't Automatically Secure

This is another important point.

People sometimes think:

> "It's an environment variable, so it's secure."

Not necessarily.

Environment variables can be exposed through:

```text
Logs
Debugging
Misconfigured CI
Process inspection
Application errors
Deployment systems
```

So:

> **Environment variables are a way to provide secrets, not a magic security mechanism.**

You still need proper secret management.

---

# 13. Configuration Layer

Instead of accessing:

```js
process.env.DATABASE_URL
```

everywhere in your application, create a configuration module.

For example:

```text
src/
└── config/
    └── env.ts
```

Then:

```js
export const config = {
  port: Number(process.env.PORT || 3000),
  databaseUrl: process.env.DATABASE_URL,
  jwtSecret: process.env.JWT_SECRET
};
```

Your application can use:

```js
config.databaseUrl
```

instead of:

```js
process.env.DATABASE_URL
```

everywhere.

---

# 14. Why Have a Config Layer?

Suppose you access environment variables in 50 files.

Then someone changes:

```text
JWT_SECRET
```

or:

```text
DATABASE_URL
```

You now have configuration logic scattered throughout your application.

Instead:

```text
Environment
    ↓
Config module
    ↓
Application
```

Now configuration has one central entry point.

---

# 15. Validate Configuration

This is one of the most important improvements you can make.

Imagine your application requires:

```text
DATABASE_URL
JWT_SECRET
```

but someone forgets to configure them.

Without validation:

```text
Application starts
   ↓
Someone logs in
   ↓
JWT signing fails
   ↓
Production incident
```

A better approach:

```text
Application starts
   ↓
Validate configuration
   ↓
Missing JWT_SECRET
   ↓
Fail immediately
```

This is called:

> **Fail fast**

---

# 16. Configuration Validation

You can use libraries such as:

```text
Zod
Joi
envalid
```

or write your own validation.

For example:

```js
const port = Number(process.env.PORT);

if (!process.env.DATABASE_URL) {
  throw new Error("DATABASE_URL is required");
}

if (!process.env.JWT_SECRET) {
  throw new Error("JWT_SECRET is required");
}

if (!Number.isInteger(port)) {
  throw new Error("PORT must be a number");
}
```

Now your application doesn't start with broken configuration.

---

# 17. Why Fail Fast?

Suppose your application starts successfully:

```text
Server started
```

but:

```text
DATABASE_URL = undefined
```

Then five minutes later:

```text
GET /users
```

fails.

That's bad.

It's better to discover configuration problems during startup:

```text
Starting application...
Validating configuration...
ERROR: DATABASE_URL is missing
Application stopped
```

Now the problem is obvious.

---

# 18. Required vs Optional Configuration

Not every configuration value is required.

For example:

```text
DATABASE_URL
```

might be required.

But:

```text
LOG_LEVEL
```

could have a default.

Example:

```js
const logLevel = process.env.LOG_LEVEL || "info";
```

So configuration usually has:

```text
Required values
Optional values
Defaults
Validation rules
```

---

# 19. Don't Use Dangerous Defaults

This is okay:

```js
const logLevel = process.env.LOG_LEVEL || "info";
```

But be careful with:

```js
const jwtSecret =
  process.env.JWT_SECRET || "secret";
```

That's dangerous.

If production forgets the environment variable, your application silently uses:

```text
secret
```

That's a security problem.

For security-sensitive configuration:

> **Require it. Don't silently invent a fallback.**

---

# 20. Development Defaults

Some configuration can reasonably have development defaults.

For example:

```js
const port = Number(process.env.PORT || 3000);
```

That's usually fine.

But secrets are different.

For example:

```js
JWT_SECRET
DATABASE_PASSWORD
AWS_SECRET
```

should usually be explicitly provided.

---

# 21. `.env.example`

A useful pattern is:

```text
.env.example
```

It shows developers which variables they need.

Example:

```env
PORT=3000
NODE_ENV=development

DATABASE_URL=

REDIS_URL=

JWT_SECRET=

STRIPE_SECRET_KEY=
```

Notice:

> Don't put real secrets here.

It is a template.

---

# 22. `.env.example` vs `.env`

Think:

```text
.env
```

contains:

```text
Actual local values
```

while:

```text
.env.example
```

contains:

```text
Required variable names
```

So a new developer can:

```text
Clone repository
   ↓
Copy .env.example
   ↓
Create .env
   ↓
Fill values
   ↓
Run application
```

---

# 23. Secret Management in Production

For production, you generally don't want developers manually creating:

```text
.env
```

on production servers.

Cloud platforms and deployment systems usually provide secret/configuration management.

Conceptually:

```text
Secret Store
    ↓
Deployment
    ↓
Environment
    ↓
Application
```

Examples include cloud secret managers and CI/CD secret stores.

The exact tool varies.

The principle stays the same.

---

# 24. Secrets vs Configuration

These are related but not identical.

### Configuration

```text
PORT=3000
LOG_LEVEL=info
NODE_ENV=production
```

### Secret

```text
JWT_SECRET=...
DATABASE_PASSWORD=...
STRIPE_SECRET_KEY=...
```

Secrets require stricter access control.

You don't want every developer or service to have access to every production secret.

---

# 25. Principle of Least Privilege

Suppose your application needs:

```text
S3 access
```

Don't automatically give it:

```text
Full AWS account access
```

Give it only the permissions it actually needs.

The principle:

> **Give each system the minimum permissions necessary to do its job.**

This becomes extremely important as systems grow.

---

# 26. Configuration and Deployment

Suppose you deploy:

```text
Version 1
```

with:

```text
DATABASE_URL=A
```

Then:

```text
Version 2
```

should use:

```text
DATABASE_URL=A
```

unless you've intentionally changed it.

The application artifact and environment configuration can be treated separately.

Conceptually:

```text
Application artifact
        +
Environment configuration
        ↓
Running application
```

---

# 27. The Twelve-Factor App

You may hear:

> **12-factor app**

It's a methodology for building applications that are easier to deploy and operate.

One important principle is:

> **Store configuration in the environment.**

The bigger idea is to keep application code independent from environment-specific configuration.

You don't need to memorize all twelve factors right now.

Understand the philosophy.

---

# 28. Don't Create Separate Codebases for Environments

Bad approach:

```text
app-development/
app-staging/
app-production/
```

with slightly different code.

Now you have:

```text
Different bugs
Different behavior
Different deployments
```

Prefer:

```text
Same application
+
Different configuration
```

For example:

```text
Same code
   ↓
Development config

Same code
   ↓
Production config
```

---

# 29. Environment-Specific Behavior

Sometimes behavior really does need to differ.

For example:

```js
if (config.environment === "development") {
  enableDebugLogging();
}
```

That's okay when intentional.

But don't fill your application with:

```js
if (production) ...
if (development) ...
if (staging) ...
if (production) ...
```

everywhere.

Too much environment-specific branching makes the code difficult to reason about.

---

# 30. Feature Flags

Another concept you'll encounter:

> **Feature flags**

Suppose you've built:

```text
New Dashboard
```

but don't want to enable it for everyone yet.

You might have:

```text
NEW_DASHBOARD_ENABLED=true
```

Then:

```text
Feature flag
   ↓
Application
   ↓
Feature enabled?
```

Feature flags can be useful for:

```text
Gradual rollouts
Testing
Experiments
Emergency disabling
```

But don't let them accumulate forever.

Old flags become technical debt.

---

# 31. Configuration Is Not Business Data

This distinction is useful.

Configuration:

```text
DATABASE_URL
JWT_SECRET
PORT
LOG_LEVEL
```

Business data:

```text
users
orders
posts
payments
```

Don't use environment variables as your database.

If you have:

```text
50,000 users
```

you obviously don't put them into:

```text
USERS=...
```

Configuration tells the application how to operate.

The database stores application data.

---

# 32. Configuration Loading Order

Depending on your stack, configuration may come from:

```text
Environment variables
.env files
Command-line arguments
Config files
Cloud secret managers
Platform configuration
```

The exact precedence depends on your tooling.

The important thing is:

> **Know which source wins when multiple values exist.**

Otherwise you may think:

```text
DATABASE_URL = A
```

while the application is actually receiving:

```text
DATABASE_URL = B
```

---

# 33. Don't Print Your Entire Environment

This is dangerous:

```js
console.log(process.env);
```

Why?

Because it might print:

```text
JWT_SECRET
DATABASE_PASSWORD
API_KEYS
AWS_CREDENTIALS
```

Don't dump your entire environment into logs.

If you need to debug configuration:

```text
Log safe configuration names
Redact sensitive values
```

For example:

```text
DATABASE_URL = configured
JWT_SECRET = configured
REDIS_URL = configured
```

not:

```text
JWT_SECRET = actual-secret
```

---

# 34. Secret Rotation

What happens if a secret leaks?

You should be able to:

```text
Generate new secret
   ↓
Update secret store
   ↓
Deploy/restart
   ↓
Old secret becomes invalid
```

This is:

> **Secret rotation**

Don't design systems where changing a secret requires rewriting application code.

---

# 35. JWT Secret Rotation

Suppose your application signs JWTs with:

```text
SECRET_A
```

You discover:

> "SECRET_A was accidentally exposed."

You can't simply ignore it.

You need a rotation strategy.

Depending on your authentication design, you might:

```text
Rotate signing keys
Use key IDs
Support old/new keys during transition
Invalidate affected sessions/tokens
```

The exact strategy depends on the system.

The important lesson:

> Secrets can change.

Your architecture should handle that.

---

# 36. Database Configuration

Instead of:

```js
const db = new PrismaClient({
  url: "postgresql://localhost..."
});
```

use:

```text
DATABASE_URL
```

Then:

```text
Development
→ local database

Test
→ test database

Production
→ production database
```

Same code.

---

# 37. Redis Configuration

Same idea:

```text
REDIS_URL
```

Development:

```text
redis://localhost:6379
```

Production:

```text
redis://production-redis:6379
```

The application doesn't need separate Redis code.

It just connects using configuration.

---

# 38. External API Configuration

Suppose your application uses an email provider.

Don't write:

```js
const apiKey = "abc123";
```

Instead:

```text
EMAIL_API_KEY
EMAIL_API_URL
```

Then:

```text
Development
→ test provider

Production
→ production provider
```

Again:

```text
Same code
Different configuration
```

---

# 39. Configuration Validation With Zod

Since this project uses TypeScript, a schema validator such as Zod can make configuration safer.

Conceptually:

```ts
const envSchema = z.object({
  NODE_ENV: z.enum([
    "development",
    "test",
    "production"
  ]),

  PORT: z.coerce.number().default(3000),

  DATABASE_URL: z.string().url(),

  JWT_SECRET: z.string().min(32)
});
```

Then:

```ts
const env = envSchema.parse(process.env);
```

Now:

```text
Invalid configuration
        ↓
Application fails during startup
```

rather than:

```text
Application starts
        ↓
Random feature fails later
```

---

# 40. Why TypeScript Helps Here

Without validation:

```ts
process.env.DATABASE_URL
```

may be:

```text
string | undefined
```

You have to keep thinking:

> "Could this be undefined?"

After validation:

```ts
env.DATABASE_URL
```

can be treated as a validated value.

So you get:

```text
Environment variables
        ↓
Runtime validation
        ↓
Type-safe config
        ↓
Application
```

That's a much cleaner design.

---

# 41. Configuration Module Example

A simple structure:

```text
src/
├── config/
│   └── env.ts
├── lib/
│   ├── logger.ts
│   └── db.ts
├── routes/
├── services/
└── server.ts
```

`env.ts`:

```ts
import { z } from "zod";

const envSchema = z.object({
  NODE_ENV: z
    .enum(["development", "test", "production"])
    .default("development"),

  PORT: z.coerce.number().default(3000),

  DATABASE_URL: z.string().min(1),

  JWT_SECRET: z.string().min(32)
});

export const env = envSchema.parse(process.env);
```

Then:

```ts
import { env } from "./config/env";

console.log(env.PORT);
```

Now configuration is centralized.

---

# 42. Don't Validate Everything Everywhere

Bad:

```text
Controller validates environment
Service validates environment
Repository validates environment
Database validates environment
```

Configuration should generally be validated once at startup.

Then the rest of the application consumes validated configuration.

```text
Startup
   ↓
Validate
   ↓
Config object
   ↓
Application
```

---

# 43. Configuration Naming

Use clear names.

Good:

```text
DATABASE_URL
REDIS_URL
JWT_SECRET
PORT
LOG_LEVEL
NODE_ENV
```

Bad:

```text
DB1
THING
SECRET2
URL_A
VALUE_X
```

Future-you should understand what a configuration variable means.

---

# 44. Don't Put Secrets in Source Code

This includes:

```text
JavaScript
TypeScript
JSON
YAML
Dockerfiles
README files
Tests
```

Bad:

```ts
const stripeSecret = "sk_live_...";
```

Even if the repository is private, secrets shouldn't casually live in source control.

---

# 45. Don't Put Secrets in GitHub Issues

Another surprisingly common mistake.

Someone asks:

> "What's the production API key?"

And someone pastes it into:

```text
Issue
Pull Request
Slack
Commit
```

Don't.

Use proper secret-management channels.

---

# 46. Secret Scanning

Modern repositories can use secret scanning to detect accidentally committed credentials.

But:

> **Don't depend on scanners to save you.**

Prevent the leak first.

If a real secret is committed:

```text
1. Revoke/rotate it
2. Investigate exposure
3. Remove it from the codebase/history where appropriate
4. Replace it safely
```

Deleting the visible line isn't enough if the credential remains valid.

---

# 47. Configuration in Docker

From Chapter 20 we'll go deeper into Docker.

For now understand:

```text
Docker Image
+
Environment Variables
↓
Running Container
```

You generally don't want to bake production secrets into the Docker image.

Instead provide configuration when the container runs.

---

# 48. Configuration in CI/CD

Later, when we discuss CI/CD, you'll see:

```text
GitHub Actions
      ↓
Secrets
      ↓
Build / Test / Deploy
```

Again:

```text
Code
≠
Secrets
```

The deployment system injects the appropriate configuration.

---

# 49. Test Configuration

Your test environment should have its own configuration.

For example:

```text
NODE_ENV=test
DATABASE_URL=test_database
JWT_SECRET=test_secret
```

Your tests should not accidentally connect to production.

This is one of the most important safeguards in backend development.

---

# 50. Environment Configuration Example

Imagine your project:

```text
Backend API
```

Development:

```env
NODE_ENV=development
PORT=3000
DATABASE_URL=postgresql://localhost:5432/app_dev
REDIS_URL=redis://localhost:6379
JWT_SECRET=local-development-secret
LOG_LEVEL=debug
```

Production:

```env
NODE_ENV=production
PORT=8080
DATABASE_URL=<production database>
REDIS_URL=<production redis>
JWT_SECRET=<production secret>
LOG_LEVEL=info
```

Notice:

```text
Same application
Different values
```

---

# 51. What Happens During Startup?

A production backend can conceptually start like this:

```text
Application starts
       ↓
Load configuration
       ↓
Validate configuration
       ↓
Initialize logger
       ↓
Initialize database
       ↓
Initialize Redis
       ↓
Initialize other dependencies
       ↓
Start HTTP server
```

If required configuration is missing:

```text
Load configuration
       ↓
Validation fails
       ↓
Application stops
```

That's much safer.

---

# 52. Startup vs Runtime Errors

Configuration problems are often:

> **Startup errors**

For example:

```text
JWT_SECRET missing
```

You want to catch these immediately.

Runtime errors are different:

```text
Payment provider timeout
```

That happens while the application is serving requests.

So:

```text
Configuration problems
→ fail during startup

Operational problems
→ handle during runtime
```

This distinction is useful.

---

# 53. Configuration and Security

Good configuration management reduces risks such as:

```text
Hardcoded credentials
Accidental secret commits
Wrong database connections
Production/test confusion
Missing configuration
Uncontrolled permissions
```

Configuration management is not just developer convenience.

It's part of security.

---

# 54. Common Mistakes

## Mistake 1 — Hardcoding secrets

```js
const jwtSecret = "secret";
```

Don't.

---

## Mistake 2 — Committing `.env`

Don't commit real secrets.

---

## Mistake 3 — Using production database in tests

Extremely dangerous.

---

## Mistake 4 — Giving secrets safe-looking defaults

Bad:

```js
JWT_SECRET || "secret"
```

---

## Mistake 5 — Not validating configuration

Your application starts with broken configuration and fails later.

---

## Mistake 6 — Reading `process.env` everywhere

Centralize configuration.

---

## Mistake 7 — Logging secrets

Never dump:

```js
console.log(process.env);
```

---

## Mistake 8 — Giving every service every secret

Follow least privilege.

---

## Mistake 9 — Never rotating secrets

Assume credentials can eventually leak.

Design for rotation.

---

## Mistake 10 — Too many environment-specific branches

Keep application behavior consistent where possible.

---

# 55. Interview Questions

## 1. What is configuration?

Information that controls how an application runs, such as database URLs, ports, service URLs, and feature settings.

---

## 2. What are environment variables?

Values provided to a running process by its environment and commonly used for configuration.

---

## 3. Why shouldn't secrets be hardcoded?

Because source code can be exposed through repositories, logs, backups, developers, CI systems, or other channels.

---

## 4. What is a `.env` file?

A local file commonly used during development to define environment variables.

It should not contain production secrets in source control.

---

## 5. Should `.env` be committed?

Usually no when it contains secrets.

Instead commit a safe `.env.example` showing required variable names.

---

## 6. Why validate environment variables?

To catch missing or invalid configuration during application startup rather than discovering the problem later at runtime.

---

## 7. What does "fail fast" mean?

Detecting a problem as early as possible and stopping or rejecting invalid operation instead of allowing the system to continue in a broken state.

---

## 8. Why shouldn't you use a default JWT secret?

Because if the real secret is missing, the application could silently use a predictable or insecure value.

---

## 9. How would you manage secrets in production?

Use the secret/configuration management facilities provided by your deployment or cloud platform, restrict access, rotate secrets when necessary, and avoid storing them in source control.

---

## 10. What is the principle of least privilege?

Giving a user, service, or application only the permissions it needs to perform its job.

---

## 11. What is the difference between configuration and application data?

Configuration controls how the application operates.

Application data represents business information such as users, orders, and posts.

---

## 12. Why centralize configuration?

To avoid scattered environment-variable access, provide consistent validation/defaults, and give the application one clear configuration interface.

---

## 13. What is the 12-factor app?

A methodology for building applications that are easier to deploy, operate, and scale. One important principle is separating configuration from code by storing environment-specific configuration outside the application code.

---

# 56. Interview Scenario

> Your application works locally but crashes immediately after deployment. What would you check?

Don't immediately change application code.

Check:

```text
Environment variables
Database URL
Redis URL
Secrets
Required API keys
Port configuration
NODE_ENV
Configuration validation
Deployment configuration
```

Then check application startup logs.

Often:

```text
Local
→ .env exists

Production
→ variable missing
```

is the entire problem.

---

# 57. Interview Scenario

> Your test suite suddenly starts deleting real data. What's wrong?

Most likely the tests are using the wrong database configuration.

Check:

```text
DATABASE_URL
NODE_ENV
Test configuration
CI secrets
Database initialization
```

A test environment should use an isolated database.

Never assume:

```text
"Surely it can't be production."
```

Make it structurally difficult to connect tests to production.

---

# 58. Interview Scenario

> A developer accidentally committed a production API key. What should you do?

Don't just delete the line.

First:

```text
1. Revoke/rotate the credential
2. Determine what was exposed
3. Investigate usage if necessary
4. Remove it from source/history where appropriate
5. Replace it through proper secret management
6. Check for similar leaked credentials
```

The key point:

> **Once a real secret is exposed, treat it as compromised.**

---

# 59. Mini Project

Take your Todo API or another backend project.

Create:

```text
.env
.env.example
.gitignore
```

Then move configuration out of your code.

For example:

```text
PORT
NODE_ENV
DATABASE_URL
REDIS_URL
JWT_SECRET
LOG_LEVEL
```

---

# 60. Step 1 — Create `.env.example`

Example:

```env
NODE_ENV=development
PORT=3000

DATABASE_URL=

REDIS_URL=

JWT_SECRET=

LOG_LEVEL=debug
```

No real secrets.

---

# 61. Step 2 — Ignore `.env`

Your `.gitignore` should contain something like:

```gitignore
.env
.env.local
```

Then verify:

```bash
git status
```

Your local secret file shouldn't appear as a file you need to commit.

---

# 62. Step 3 — Create Configuration Module

Create:

```text
src/config/env.ts
```

Load and validate your environment.

For TypeScript:

```text
Environment
    ↓
Zod
    ↓
Validated configuration
    ↓
Application
```

---

# 63. Step 4 — Remove Hardcoded Values

Search your project for:

```text
localhost
password
secret
API keys
database URLs
```

and determine which values should actually come from configuration.

Don't blindly move every string into an environment variable.

Use judgment.

---

# 64. Step 5 — Test Missing Configuration

Temporarily remove:

```text
DATABASE_URL
```

Start the application.

Expected:

```text
Application refuses to start
```

Then restore it.

Do the same for:

```text
JWT_SECRET
```

This verifies your configuration validation actually works.

---

# 65. Step 6 — Create Separate Test Configuration

Make sure tests use:

```text
NODE_ENV=test
```

and:

```text
TEST DATABASE
```

not production.

Then deliberately check your test startup configuration.

---

# 66. Step 7 — Add Configuration Documentation

Your README should explain:

```text
Required environment variables
How to create .env
How to start the application
How to run tests
```

For example:

```text
cp .env.example .env
```

Then fill in the required values.

---

# 67. Your Configuration Architecture

By the end, your backend should conceptually look like:

```text
             Environment
                  │
                  ↓
          ┌───────────────┐
          │ Configuration │
          │   Validation  │
          └───────┬───────┘
                  │
                  ↓
             Application
                  │
       ┌──────────┼───────────┐
       ↓          ↓           ↓
   Database      Redis     External APIs
```

And:

```text
Development
Test
Staging
Production
```

can provide different configuration to the same application.

---

# 68. The Mental Model

Whenever you see:

```text
"Works on my machine."
```

think:

```text
What is different between environments?
```

Maybe:

```text
Node version
Environment variables
Database
Redis
External services
File system
Network
Permissions
Secrets
Configuration
```

A good backend engineer doesn't assume environments are identical.

They make differences explicit and controlled.

---

# 69. What You Should Know After Chapter 19

You should now understand:

```text
✓ Configuration
✓ Environment variables
✓ .env
✓ .env.example
✓ Development/test/production environments
✓ Secrets
✓ Secret management
✓ Configuration validation
✓ Fail fast
✓ Configuration modules
✓ Runtime vs startup errors
✓ Feature flags
✓ Secret rotation
✓ Least privilege
✓ 12-factor application principles
✓ Configuration in Docker
✓ Configuration in CI/CD
✓ Test environment configuration
✓ Environment-specific behavior
```

But the most important idea is simple:

```text
Code
≠
Configuration
≠
Secrets
≠
Data
```

Your application code should describe:

> **How the system works.**

Configuration should describe:

> **How this deployment should run.**

Secrets should be:

> **Provided securely at runtime.**

And data should live in:

> **The systems designed to store it.**

Once you understand that separation, deploying the same backend to different environments becomes much less painful.

---

# Next — Chapter 20: Docker & Containerization

So far we've talked about:

```text
Application
Database
Redis
Queues
Workers
Configuration
```

But there's another problem:

> **How do we make sure the application runs the same way on different machines?**

That's where Docker comes in.

We'll learn:

```text
Containers
Images
Dockerfiles
Docker Compose
Volumes
Networks
Ports
Environment variables
Container-to-container communication
Multi-stage builds
Production Dockerfiles
Container health checks
```

And we'll answer the question:

> **What exactly happens when you run `docker compose up`?**